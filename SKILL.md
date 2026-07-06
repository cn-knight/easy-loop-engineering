---
name: easy-loop-engineering
description: Use when the user wants to set up loop engineering or automation for a project — e.g. "为这个项目配置 loop engineering"、"配置自动化"、"让这个项目自动跑起来"、"给项目配 loop"、"set up a self-running agent loop". Decides which of the six building blocks apply (sub-agents, state files, scheduling, event guards/hooks, persistence, contract) and implements them. Agent-neutral with Claude Code as the primary runtime and Codex as a supported alternative. Creates sub-agent files, state files, guard rails, scheduling scripts. Asks about uncertain choices, auto-does deterministic ones, confirms the plan before implementing.
---

# Easy Loop Engineering — 项目配置器

## 这个 skill 做什么

在任意项目目录下，用户说"配 loop engineering / 配自动化 / 让项目自动跑"时，帮用户把 loop engineering 的六块积木按项目需求配置落地。用户记不住怎么配——本 skill 把配置知识固化进来：**确定的自动配，不确定的问用户，出方案 → 确认 → 实施**。

本 skill **agent 中立**：核心是 loop engineering 的六积木方法论（来自 Addy Osmani 与 Karpathy 原典），与具体智能体解耦。落地时以 **Claude Code 为主运行时**，**Codex 为支持的第二运行时**，并附"适配其他智能体"指引。

## 何时触发

用户在任何项目里表达"要给这个项目上 loop engineering / 自动化 / 让 agent 自主跑 / set up a self-running loop"等意图时。

## 核心原则（不可违背）

1. **以原典为准，警惕二手缝合**：不确定的概念/做法查 `references/0_canonical-texts.md`（Addy 全文 + Karpathy §I-§IX），别凭印象瞎编。配置细节查 `references/1_loop-engineering概念学习.md`。
2. **动钱/危险操作必须配护栏，绝不裸跑无人值守循环**：调价、扣款、删数据、外发等操作，事件护栏（Claude Code 为 PreToolUse hook；Codex 为等价审批/沙箱机制）必拦。无停止条件的循环裸跑风险操作 = 没刹车的车。
3. **不确定问用户，确定直接做**：见下方"问 vs 直接做"清单。
4. **方案确认是唯一人工闸门，确认后自主运行**：风险（动钱 / 外部 / 停止条件 / 失败兜底）前置到方案里说清；用户确认（Step 4）后才落地（Step 5），落地后 loop 自主运行，中途不临时问用户、不乱停（用户可能已离场）。安全靠确定性护栏（hook 拦截 / 契约停止条件 / triage 收件箱），不靠打断。
5. **贯彻模型分离**：generator 与 evaluator 用不同模型（maker/checker 降自评共谋）。

## 配置工作流（5 步）

### Step 1 — Intake（搞清项目要自动化什么）
信息够就直接进 Step 3；不够就逐项问：
- **自动化目标（两类典型）**：① 开发/优化代码项目（有终点 → 偏条件驱动 `/goal`）；② 周期性自动化运行，如定期取 API 数据并加工（无终点 → 偏定时 `/loop`）
- **是否触达外部/动钱**：要接 connector？有调价扣款等危险操作？（有 → 必配护栏）
- **是否多 agent 并行写同一仓库**：（是 → 配 worktree）
- **跑在哪**：会话内临时？关机也要跑？（后者 → 持久化方案）
- **用哪个运行时**：Claude Code？Codex？其他？（影响落地语法）

### Step 2 — Diagnose（映射六块积木）
把项目需求映射到下方决策表，逐块标"适用 / 不适用 / 待定"。

### Step 3 — Propose（出配置方案）
生成方案，每项标 `[自动配置]` 或 `[需确认]`，含：子智能体角色与模型、state 文件、调度（条件/定时 + 间隔/停止条件）、护栏、持久化方案、契约思路、connector、worktree。

### Step 4 — Confirm
把方案给用户，逐项确认 `[需确认]` 部分。

### Step 5 — Implement（落地，见下方产出清单）

## 六块积木配置决策表

| 积木 | 何时配 | 怎么配（Claude Code / Codex） | 查 reference |
|---|---|---|---|
| **Sub-agents** | 任务需 maker/checker 分离（写码+验码），或需规划/执行/验证多角色 | CC：写 `.claude/agents/*.md`（frontmatter 模板见下方）；Codex：写 `.codex/agents/*.md` 或 AGENTS.md 约定的子智能体 | `1_...概念学习.md` 第8节 |
| **Memory/State** | 总是配（loop 跨轮接力必需） | 建 `STATE.md`(进度) / `contract.md`(完成标准) / `log.md`(append-only 日志)；**边跑边写**抗崩溃，不只切换前写。两端通用 | `0_canonical-texts.md` Karpathy §IV |
| **Automations** | 总是配（loop 的心跳） | 有终点→`/goal`(CC)；周期巡检→`/loop`(CC)；关机也跑→持久化（GitHub Actions / 云服务器+cron / Windows任务计划）。Codex 无 /loop//goal，用 Codex Automations 或 cron+`codex exec` | `1_...概念学习.md` 第3、10节 |
| **Guards/Hooks** | 有危险操作（动钱/删数据/外发）必配 | CC：PreToolUse + matcher，exit 2 拦 / exit 0 放行，fail-open；Codex：等价审批/沙箱机制 | `1_...概念学习.md` 第4节（CC 30事件表） |
| **Connectors** | loop 要触达外部系统（广告API/数据库/邮件/Slack） | 复用项目现有 MCP/ApiClient；没有则提示用户配。两端通用 | `1_...概念学习.md` 第9节 |
| **Worktrees** | 多 agent 并行改同一仓库文件 | 默认 `git worktree`（需 `git init`）；CC 有 WorktreeCreate/WorktreeRemove hook；Codex 用原生 git worktree | `1_...概念学习.md` 第7节 |

### Sub-agents：角色、模型与创建

**三角色**：
- **planner**（需求→规格翻译，永不碰码）
- **generator**（写码执行，禁自评）
- **evaluator**（预设敌意，对 contract 逐条验）

**模型分配原则（异构分离）**：planner 用最强推理模型；generator 用最强代码模型；evaluator 用与 generator **不同家族**的模型（降共谋）。

- **作者用例**：Claude Code + 火山 Coding Plan——planner=`MiniMax-M2.7` / generator=`GLM-5.2` / evaluator=`Doubao-Seed-2.0-Code`（三家族异构，见 `examples/providers/volcengine/`）。
- **其他家族同样适用**，由 agent 按原则为本项目角色选配：Claude 系列（opus/sonnet/haiku）、OpenAI 系列（o-series / gpt-4o 等）、或任何 provider。
- **单家族兜底**：不同家族最佳；同家族不同 tier 次之；同模型 + 不同 subagent 定义（不同上下文+指令）仍可实现大部分 maker/checker 价值（Karpathy §II）。

**逐个差异化就别设全局 env，靠各自 frontmatter 指定。**具体 provider 示例见 `examples/providers/`。

**Claude Code subagent frontmatter 模板**（自包含，无需外部 skill）：
```
---
name: generator
description: 写码执行角色，禁止自评
tools: Read, Write, Edit, Bash, Grep, Glob
model: YOUR_CODE_MODEL
color: green
---
<角色职责与约束正文>
```
- `color` 仅 8 色：red/blue/green/yellow/purple/orange/pink/cyan
- 模型解析顺序（CC）：`CLAUDE_CODE_SUBAGENT_MODEL` 全局 env > 调用参数 > frontmatter model > 主对话模型

**Codex 等价**：在 `.codex/agents/` 下建对应子智能体，模型分配遵循同一异构原则；Codex 无 color 字段。

### 契约配置（Karpathy §III）
- generator 先提"done 长什么样"的**可测试断言**清单，evaluator 反推加严，吵到达成一致 → 落 `contract.md`
- 小应用 ~20-30 条断言；太少 evaluator 橡皮图章
- "the contract is what gets graded"——契约是被评分的东西
- 提断言时同步做**契约可行性核查**（数据源/工具能力是否真能满足断言），别等执行后才改

## 何时问用户 vs 直接做

**直接做（确定的）**：
- state 文件模板（STATE.md / contract.md / log.md）
- 标准护栏（动钱操作拦）
- 调度建议（条件/定时 + 间隔）
- contract.md 模板
- subagent frontmatter（CC）

**问用户（不确定的）**：
- 三角色模型选择偏好（原则已给，确认具体模型即可）
- 持久化方案选哪个（云服务器 / GitHub Actions / Windows任务计划）
- 危险操作的边界（哪些操作算"动钱"必拦）
- 项目特定 connector（现有 MCP 够不够、要不要新接）
- state 文件放项目根还是配置目录下
- 运行时选 Claude Code 还是 Codex

## 适配其他智能体

本 skill 的六积木方法论与具体智能体解耦。已知映射：

| 概念 | Claude Code | Codex | 其他智能体 |
|---|---|---|---|
| 调度-条件驱动 | `/goal` | Codex Automations / 脚本判断 | 条件循环 |
| 调度-定时 | `/loop` | cron + `codex exec` | cron + CLI |
| 事件护栏 | PreToolUse/PostToolUse hook | 审批/沙箱 | 拦截器中间件 |
| 子智能体 | `.claude/agents/*.md` | `.codex/agents/` | 按该智能体约定 |
| 技能封装 | `SKILL.md` + `references/` | `AGENTS.md` + `references/` | 按该智能体约定 |
| 模型指定 | frontmatter `model:` | 按 Codex 约定 | 按 provider 约定 |

运行时不在表内时，按"功能等价"映射，核心是六积木本身。

## references 索引（查原典用）

| reference 文件 | 查什么 |
|---|---|
| `references/0_背景知识.md` | 概念背景与范式定位（loop 本质、harness、四层栈、六积木总览） |
| `references/1_loop-engineering概念学习.md` | 完整学习问答——六积木逐节、调度对比、护栏事件表、Memory、Worktree、Sub-agents 实测、持久化、风险护栏。**配置遇细节先查这里** |
| `references/0_canonical-texts.md` | 原典：Addy 全文 + Karpathy §I-§IX。**不确定时以这里为准** |
| `examples/providers/` | 可选 provider 的模型选择示例（如火山引擎 Coding Plan） |

> 注：references 中的调度/护栏/子智能体语法以 Claude Code 为示例说明；Codex 等价物见上方映射表。

## Step 5 Implement 产出清单（落地后在目标项目创建）

- [ ] `.claude/agents/*.md`（CC）或 `.codex/agents/`（Codex）— 子智能体
- [ ] `STATE.md` / `contract.md` / `log.md` — state 文件
- [ ] `.claude/settings.json`（CC hook 护栏）或 Codex 等价 — 若有危险操作
- [ ] 调度方案 — `/loop` 或 `/goal`（CC）；Codex 用 Automations/cron；持久化则给 cron 脚本 / GitHub Actions workflow / Windows 任务计划 / 云服务器部署说明
- [ ] 一份"配置清单 + 如何启动 loop"说明交给用户

## 重要：本 skill 不包办业务

本 skill 配的是 loop engineering 的**骨架**（六积木配置）。具体业务逻辑（广告怎么调价、产品代码写什么）由用户在子智能体 body / skill / contract 里填。本 skill 落完骨架后，要告诉用户"下一步把业务规则写进哪些文件"。
