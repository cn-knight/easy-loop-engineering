# Easy Loop Engineering (Codex 入口)

本文件是 easy-loop-engineering 在 **Codex** 运行时下的入口。方法论与 Claude Code 入口（`SKILL.md`）完全一致，仅落地语法不同。

## 触发

用户说"配 loop engineering / 配自动化 / 让项目自动跑 / set up a self-running loop"时。

## 工作流

同 `SKILL.md` 的 5 步（Intake → Diagnose → Propose → Confirm → Implement）。核心原则、六积木决策表、契约配置、"问 vs 直接做"清单均见 `SKILL.md`，此处不重复——只给 Codex 侧的落地语法差异。

## Codex 落地语法映射

| 积木 | Codex 做法 |
|---|---|
| Sub-agents | `.codex/agents/*.md`，模型分配遵循异构分离原则（planner=最强推理，generator=最强代码，evaluator=异构家族） |
| Memory/State | `STATE.md` / `contract.md` / `log.md`，边跑边写（与运行时无关） |
| Automations | 无 `/loop`/`/goal`；用 Codex Automations 或 cron + `codex exec`；关机也跑则持久化（GitHub Actions / 云服务器 / Windows任务计划） |
| Guards | Codex 审批/沙箱机制替代 PreToolUse hook；动钱/删数据/外发必拦 |
| Connectors | 复用项目现有 MCP/ApiClient |
| Worktrees | `git worktree` |

## 模型分配原则

planner 用最强推理模型；generator 用最强代码模型；evaluator 用与 generator 不同家族的模型。具体模型由你的 provider 决定——可选 provider 示例见 `examples/providers/`。

## references

- `references/0_canonical-texts.md` — 原典（Addy + Karpathy），不确定时以这里为准
- `references/1_loop-engineering概念学习.md` — 配置细节先查这里
- `references/0_背景知识.md` — 概念背景
- `examples/providers/` — 可选 provider 模型选择示例

> references 中的调度/护栏/子智能体语法以 Claude Code 为示例说明，Codex 等价做法见上表。

## 适配其他智能体

见 `SKILL.md` 的"适配其他智能体"映射表。
