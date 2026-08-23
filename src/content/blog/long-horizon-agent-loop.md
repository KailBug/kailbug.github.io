---
title: "Long-Horizon Agent 出现死循环时怎么处理？"
description: "为什么 max_steps 不足以解决 Long-Horizon Agent 的死循环问题。"
date: 2026-08-22
category: "Agent Runtime"
tags:
  - "Agent"
  - "Long Horizon"
  - "Runtime"
  - "Reliability"
draft: false
featured: true
---

Long-Horizon Agent 的运行时间越长，越容易出现重复调用工具、反复修改同一文件，或者在多个相似状态之间来回切换的问题。

## 问题不只是步数过多

`max_steps` 可以限制一次运行最多执行多少步，但它只能在达到上限后终止任务，无法判断 Agent 是否仍在取得有效进展。

> 步数限制是一种最终保险，但不是完整的死循环检测机制。

## 可以观察哪些信号？

| 信号 | 可能的问题 |
| --- | --- |
| 连续执行相同工具和参数 | 重复动作 |
| 多轮之后状态没有变化 | 无进展循环 |
| 在少量状态之间反复切换 | 状态振荡 |
| Token 持续消耗但没有新产物 | 推理停滞 |

## 一个简单的停止原因模型

```ts
type StopReason =
  | 'completed'
  | 'step_limit'
  | 'no_progress'
  | 'repeated_action';
```

更可靠的 Agent Runtime 应同时记录步数、状态变化、工具调用历史和任务产物，再根据这些信号决定继续、重规划或停止。