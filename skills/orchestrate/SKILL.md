---
name: orchestrate
description: Decide whether multiple Codex agents will shorten the critical path, then coordinate only independent, non-overlapping subtasks with clear ownership and minimal overhead. Use when the user asks for subagents, parallel work, task decomposition, collaboration, or invokes $orchestrate. Invoking it does not require spawning agents.
---

# Agent 协同编排

目标是更快完成任务，不是使用更多代理。主 Agent 负责范围、关键路径、整合和最终结果；用户明确指定的数量、模型、权限和分工优先。

## 1. 先判断是否值得并行

先读取项目指令和工作区状态，再记录范围卡：目标、DoD、允许改动、非目标、必要验证、baseline。

只有以下条件全部成立才启动子代理：

- 至少有两个属于 DoD 的独立子任务；
- 文件、接口或调查范围不重叠；
- 主 Agent 有不重复的工作可同步推进；
- 预计“最慢子任务 + 协调整合”比单 Agent 更快。

否则使用单 Agent。调用 `$orchestrate` 不等于必须 spawn；单故障、单模块、小范围和只读任务默认不拆。

## 2. 委派而不重复

- 当前下一步依赖的阻塞任务留给主 Agent，只委派可并行的 sidecar。
- 每个文件和公共接口只有一个 owner；主 Agent 不重做已委派范围，子代理不得继续委派。
- 默认同时活跃两个子代理；若有三个以上独立、低耦合且未完成的 DoD 子任务，可扩展至平台上限，但不为凑满而创建。已完成或关闭的代理不占并发名额，无固定累计总数。未指定时使用 `model=gpt-5.6-luna`；一般任务使用 `reasoning_effort=high`，仅在复杂且质量优先时使用 `max`。用户指定优先。
- Prompt 只包含目标、ownership、边界、交付物、DoD、必要验证和输出预算。
- 新发现只分为阻塞或非阻塞。首个被证实的根因出现后立即收敛，请其余代理提交 handoff，不再扩展调查。

- 每波结束后，只有仍有未完成的独立 DoD 任务才启动下一波；关闭代理不会自动补建代理。

## 3. 正确等待

- 主 Agent 先推进不重叠工作，只在下一步确实被阻塞时等待；同一目标只保留一条等待链。
- 若外层 `functions.exec` 返回 `Script running with cell ID ...`，先用该 `cell_id` 调用 `functions.wait` 直到完成，再操作相关代理。
- timeout、`active` 或 compaction 都不等于失败；不要因此重启、重复实现或中断代理。
- 仅在用户取消、明确循环、越权或不可恢复失败时结束代理；先取得 final/partial handoff。`interrupt` 不用于催促。
- 关闭运行中的代理后，先确认其进入终态，再启动下一代理。

## 4. 整合与验收

- 检查 handoff 和改动，复用已通过的验证，不重复运行无关测试。
- 以用户的真实 DoD 验收；线上任务修复必须重试原任务并确认最终状态和关键产物。
- 只运行受影响范围的必要检查，最后执行一次差异检查；不要追加审计、优化或新代理。
- 汇总父子代理的结果、耗时、验证、未完成项和风险；成本统计包含所有代理。
