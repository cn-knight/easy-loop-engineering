# Easy Loop Engineering

[English](./README.en.md) | 简体中文

> 把 loop engineering 的六块积木按项目需求配置落地：确定的自动配，不确定的问用户，出方案 → 确认 → 实施。

## 为什么做这个

Loop engineering 概念如火如荼，行业大佬纷纷背书，甚至直接出《LOOPS.md》攻略。但牵涉概念太多，学一遍很难记住，更谈不到学以致用——具体到项目里，很多人不知道如何让智能体从 loop engineering 的角度，实现行业大佬们说的"我不想没完没了地给智能体写提示词，让智能体自己跑起来"。

所以干脆，我把提出 loop engineering 理论的大佬们的原文，以及我学习和实现 loop engineering 的过程，总结成文档，做成一个 skill。在项目里直接让智能体使用这个 skill，去创建 loop engineering 工作机制，或引导我实现 loop engineering。这样就拒绝了死记硬背概念，有的放矢。

## 它做什么

在任意项目里说"为这个项目配置 loop engineering""配置自动化""让项目自动跑起来"时，这个 skill 决定六块 loop engineering 积木中哪些适用并落地实现：

1. **Sub-agents** — planner / generator / evaluator，异构模型 maker/checker 分离
2. **Memory/State** — `STATE.md` / `contract.md` / `log.md`，边跑边写
3. **Automations** — Claude Code 的 `/goal`（条件驱动）或 `/loop`（定时）；Codex 用 Automations 或 cron
4. **Guards/Hooks** — 动钱 / 删数据 / 外发操作必配护栏
5. **Connectors** — 复用现有 MCP / ApiClient 触达外部系统
6. **Worktrees** — `git worktree` 隔离并行 agent

方法论来自 Addy Osmani 的 loop engineering 长文与 Karpathy《LOOPS.md》（全文见 `references/0_canonical-texts.md`）。

## 何时触发

在任意项目里，用自然语言表达“让智能体自主跑起来”的意图即可触发，例如：

- “为这个项目配置 loop engineering”
- “配置自动化” / “让这个项目自动跑起来”
- “给项目配 loop” / “帮我搭一个 loop”
- “set up loop engineering for this project”
- “make this project run itself” / “set up a self-running agent loop”

skill 会判断六块积木哪些适用，出方案 → 你确认 → 落地。

### 两类典型场景

loop engineering 的落地大致分两类，正好对应两种调度原语：

**第一类 · 开发/优化一个代码项目（有终点，偏 `/goal`）**
让智能体围绕代码项目跑“开发或优化”的自动化循环：planner 拆任务、generator 写码、evaluator 验码，契约达成即停。典型：做一个产品/功能、修一批 issue、重构模块。

**第二类 · 周期性自动化运行（无终点，偏 `/loop`）**
让智能体周期性访问外部 API 取数据并做后续加工，长期巡检、无终点。典型示例：通过 MCP 定期获取亚马逊广告报告，并优化广告活动（调价、开关 campaign）。

> **第二类常触钱/触外部**：调价、扣款、外发等属危险操作，skill 会强制配 PreToolUse 护栏（Claude Code）或等价审批/沙箱（Codex），绝不裸跑无停止条件的 `/loop`。

> **边界**：事件驱动型自动化（如新 issue 自动分诊）更像“automation”而非“loop”，本 skill 的 hook/connector 积木可覆盖，但通常不单独成 loop。

## 安装

### Claude Code

```bash
git clone https://github.com/cn-knight/easy-loop-engineering ~/.claude/skills/easy-loop-engineering
```

然后在任意项目里说"为这个项目配置 loop engineering"。

### Codex

把仓库 clone 到 Codex 能读到的位置（项目根或 `~/.codex/`），打开 `AGENTS.md` 作为 Codex 入口——从你项目的 `AGENTS.md` 引用它，或直接阅读。

## 跨智能体

核心方法论与具体智能体解耦：以 Claude Code 为主运行时，Codex 为支持的第二运行时，并附"适配其他智能体"映射表（见 `SKILL.md`）。

## 模型

作者用例：Claude Code + 火山引擎 Coding Plan（planner/generator/evaluator 用 GLM / MiniMax / Doubao 三家族异构，见 `examples/providers/volcengine/`）。skill 的模型选择**家族中立**——用 Claude 系列、OpenAI 系列、或其他家族的用户，agent 会按“异构分离”原则为本项目角色选配相应模型。

## 结构

```
easy-loop-engineering/
├── SKILL.md              # Claude Code 入口
├── AGENTS.md             # Codex 入口
├── references/           # 原典 + 概念问答（共享，agent 中立）
├── examples/providers/   # 可选 provider 模型选择示例
└── dev/loop/             # 本 skill 自身优化循环的设置示例
```

## License

MIT
