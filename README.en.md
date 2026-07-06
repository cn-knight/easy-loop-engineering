# Easy Loop Engineering

English | [简体中文](./README.md)

> Configure loop engineering's six building blocks for any project: deterministic choices auto-applied, uncertain ones asked, plan → confirm → implement.

## Why this skill

Loop engineering is booming — industry leaders are endorsing it and even publishing playbooks like *LOOPS.md*. But it involves too many concepts: hard to remember after one pass, let alone apply. In real projects, many people don't know how to make their agent, from a loop-engineering angle, achieve what the leaders mean by "I don't want to keep prompting the agent endlessly — let the agent run itself."

So I gathered the original writings of the loop-engineering founders, plus my own process of learning and implementing it, into docs and packaged them as a skill. Inside any project, let the agent use this skill to set up a loop-engineering workflow, or guide me to implement loop engineering. No more rote-memorizing concepts — just targeted application.

## What it does

Say "configure loop engineering for this project" / "set up automation" / "make this project run itself" inside any project, and this skill decides which of the six loop-engineering building blocks apply and implements them:

1. **Sub-agents** — planner / generator / evaluator, heterogeneous maker/checker separation
2. **Memory/State** — `STATE.md` / `contract.md` / `log.md`, written as you go
3. **Automations** — Claude Code's `/goal` (condition-driven) or `/loop` (cadence); Codex uses Automations or cron
4. **Guards/Hooks** — mandatory guard rails for money / destructive / external actions
5. **Connectors** — reuse existing MCP / ApiClient to reach external systems
6. **Worktrees** — `git worktree` to isolate parallel agents

Methodology from Addy Osmani's loop-engineering essay and Karpathy's *LOOPS.md* (full text in `references/0_canonical-texts.md`).

## When to trigger

Express the intent "let the agent run itself" in natural language inside any project, e.g.:

- "set up loop engineering for this project"
- "configure automation" / "make this project run itself"
- "set up a self-running agent loop" / "help me build a loop"

The skill understands both English and Chinese.

The skill decides which building blocks apply, proposes a plan → you confirm → it implements.

### Two typical scenarios

Loop engineering falls into roughly two categories, mapping cleanly to the two scheduling primitives:

**Category 1 · Develop/optimize a code project (has endpoint, leans `/goal`)**
Run a dev/optimization loop around a code project: planner breaks down tasks, generator writes code, evaluator verifies; stops when the contract is met. Typical: build a product/feature, fix a batch of issues, refactor a module.

**Category 2 · Periodic automated operation (no endpoint, leans `/loop`)**
Periodically fetch data from an external API and process it — long-running patrol, no endpoint. Example: via MCP, regularly pull Amazon Ads reports and optimize ad campaigns (adjust bids, toggle campaigns).

> **Category 2 often touches money/external systems**: bid adjustment, payment, outbound calls are dangerous; the skill enforces PreToolUse guard rails (Claude Code) or equivalent approval/sandbox (Codex) — never run a no-stop-condition `/loop` bare.

> **Edge case**: event-driven automations (e.g., auto-triage on new issue) are more "automation" than "loop"; this skill's hook/connector blocks cover them, but they usually don't form a loop on their own.

## Install

### Claude Code

```bash
git clone https://github.com/cn-knight/easy-loop-engineering ~/.claude/skills/easy-loop-engineering
```

After install, run `/skills` to check easy-loop-engineering is in the skill list, or just ask Claude Code whether it has it (restart Claude Code if it doesn't show).

Then say "set up loop engineering for this project" in any project.

### Codex

Clone the repo where Codex can read it (project root or `~/.codex/`), open `AGENTS.md` as the Codex entry — reference it from your project's `AGENTS.md` or read it directly.

## Agent-neutral

The core methodology is decoupled from any specific agent: Claude Code is the primary runtime, Codex is a supported second runtime, with an "adapting to other agents" mapping table (see `SKILL.md`).

## Models

Author's setup: Claude Code + Volcengine Coding Plan (planner/generator/evaluator use GLM / MiniMax / Doubao — three heterogeneous families; see `examples/providers/volcengine/`). Model selection is **family-agnostic** — users on Claude, OpenAI, or any other family get the agent to pick appropriate per-role models following the same heterogeneous-separation principle.

## Structure

```
easy-loop-engineering/
├── SKILL.md              # Claude Code entry
├── AGENTS.md             # Codex entry
├── references/           # canonical texts + concept Q&A (shared, agent-neutral)
└── examples/providers/   # optional provider model-selection examples
```

## License

MIT
