---
title: "Long-Horizon Agent 出现死循环时怎么处理？"
description: "为什么 max_steps 不足以解决 Long-Horizon Agent 的死循环问题。"
date: 2026-08-22
category: "Agent Runtime"
tags:
  - "Agent Runtime"
  - "Long Horizon"
  - "Reliability"
  - "Cost"
draft: false
featured: true
---

![Long-Horizon Agent Loop Detection](/img/long-horizon-agent-runtime.png)

**2026 年 6 月 15 日**发布的 DeerFlow 2.0 更新中，专门加入了可配置的 **loop detection**，
并支持针对不同 Tool 设置不同的频率阈值。DeerFlow 2.0 本身已经从 Deep Research workflow 重构成面向分
钟到小时级任务的 **Long-Horizon SuperAgent Harness**。DeerFlow 社区近期讨论也点出了一个很实际的问题：
长任务 Agent 经常真正死掉的不是 Planning，而是 Runtime——连续调用同一个工具、无限 retry、没有任何 forward progress，
却持续消耗 token 和时间。[ByteDance DeerFlow 2.0 Changelog](https://github.com/bytedance/deer-flow/blob/main/CHANGELOG.md?utm_source=chatgpt.com)

Long-Horizon Agent 的需求现在正逐渐变得越来越常见，例如真正让 Coding Agent 开始干活的时候，就可能出现下面的长流程：
```text
research → browse → analyze → code → test → fix → validate
```

但是 Long-Horizon Task 在几十甚至几百步以后，仅仅靠下面一个简单的loop是很不可靠的：
```python
for _ in range(50):
    agent.step()
```

因为**到 50 步时强杀**只能限制最坏成本，却不能判断： **Agent 到底是在合理迭代，还是第 12 步开始就已经卡死了？**

其实, `max_steps` 只是最终保险丝，不是 loop detection。 比如一个 Coding Agent 在修测试：
 ```text
pytest → 失败 → 修改代码 → pytest → 新错误 → 修改代码 → pytest
 ```

 即使连续执行了 10 次 `pytest`，也可能是正常进展, 但下面这种,跑 4 次基本就已经没有意义了,
 所以一般不会只统计 step number，而会判断 **progress**。

 ```text
 npm install → ECONNRESET
 npm install → ECONNRESET
 npm install → ECONNRESET
 npm install → ECONNRESET
 ```

---

 第一层是最简单的 **tool frequency detection**。 对每个 Tool 设置不同 policy, 
 这也是为什么 loop policy 最好做到 per-tool，而不是一个全局数字。

 ```text
 read_file   → 可以频繁执行
 search_code → 阈值较宽
 npm_install → 连续失败 2~3 次就应该干预
 git_push    → 默认不能自动连续 retry
 ```

 第二层是 **invocation fingerprint**。对于 tool_name, normalized_args, error_type, important_output, 去计算 fingerprint。
 如果 Agent 连续出现下面这种 tool error,  那基本可以判断 Agent 没有从失败中学习。

 ```text
 same tool + same args + same error
 ```

 第三层是 **state progress**。

 例如 Coding Agent 的状态可以包含：

 ```text
 failing_tests, modified_files, unresolved_errors, completed_subtasks
 ```

 即使调用序列不完全一样，如果连续 8 步之后：

 ```text
 failing_tests = 7
 modified_files = same
 unresolved_error = same
 ```

 那也属于 no-progress loop, 检测到 loop 后，不应该马上 terminate，而是设计分级恢复。

---

 第一阶段是： **reflection / failure compression**

 把之前几十步压成：
 ```text
 已尝试： A, B, C
 均因为 X 失败。
 不要再次重复这些策略。
 ```

 再让模型重新规划。
 如果依然失败，第二阶段可以：

 ```text
 change strategy → 换 Tool → 扩大搜索范围 → 读取新的 evidence
 ```

 第三阶段可以进行 **model escalation**：普通 worker 原本运行便宜模型，在连续 no-progress 后升级到更强模
 型进行诊断。但 escalation 本身也应该有预算，否则：

 ```text
 卡住 → 换贵模型 → 继续卡 → 更贵模型
 ```
 会变成成本黑洞。

 如果还是无法恢复，应该保存 checkpoint，并向用户请求必要信息，而不是继续假装工作。
 ```text
 agent → blocked
 ```

 同时还应该设置三类 budget：

 ```text
 step budget, token budget, wall-clock budget
 ```

 因为有些 loop 不一定步数很多，但单次 Tool 很贵。 例如浏览器工具每次运行 30 秒：

 ```text
 10 steps ≈ 5 min
 ```

 这时 step budget 就无法很好控制成本。 可观测性方面，就应该记录：

 ```text
 repeated_tool_count, repeated_error_count, no_progress_steps, recovery_trigger, 
 recovery_strategy, post_recovery_success, wasted_tokens, wasted_latency
 ```

 一个特别重要的指标是： **loop detector false-positive rate。**

 因为 Agent 有些合法行为本来就是重复的，比如分页抓取 20 个页面。如果 detector 太激进，会把正常任务杀掉。因此真正可靠的判断不是：

 ```text
 repeated action = loop
 ```

 而是：

 ```text
 repeated action + repeated outcome + no state progress
 ```

 最后仍然会保留 `max_steps`，但它只作为最外层 kill switch。 所以最终的 Runtime 策略大概是：

![Long-Horizon Agent Loop Detection 2](/img/long-horizon-agent-runtime-2.png)

所以我认为生产级 Long-Horizon Agent 的核心不是： **“最多让它跑多少步”** ,
而是 **“Runtime 能不能判断它是否仍然在朝目标推进。”**, **`max_steps` 控制的是损失上限, loop detection 判断的是执行质量。**
DeerFlow 2.0 将 loop detection 做成可配置、可按 Tool 设置不同频率阈值，本质上就是开始把这类问题从 Prompt 层下沉到 **Agent Harness / Runtime**。
