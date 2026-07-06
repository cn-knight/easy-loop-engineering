# 0. Loop Engineering 提出者原话索引（一手资料库）

> 用途：把 loop engineering 几位关键人物的原话集中存档，避免反复联网查证、避免二手解读失真。
> 原则：以原典为准；抓不到一手的，标注存疑，不假装找到。
> 整理日期：2026-06-28

## 四人原话速查

### 1. Addy Osmani（Google Cloud AI 总监，概念命名者）
- **原文**：2026-06-07 博客《Loop Engineering》→ 本文下方附录全文。
- **核心定义（原话）**："Loop engineering is replacing yourself as the person who prompts the agent. You design the system that does it instead."
- **loop 定义**："a recursive goal where you define a purpose and the AI iterates until complete"
- **与 harness 关系**："Loop engineering sits one floor above the harness."
- **六块积木原话**："A loop needs five things and then one place to remember stuff."
- 一手可靠，全文已落盘。

### 2. Peter Steinberger（OpenClaw / PSPDFKit 创始人）
- **原话**（经 Addy 引用）："You shouldn’t be prompting coding agents anymore. You should be designing loops that prompt your agents."
- **出处**：X 推文 https://x.com/steipete/status/2063697162748260627
- 注：X 反爬，原推未直接抓到；上述文字为 Addy 原文引用，链接已存。一手转引可靠。

### 3. Boris Cherny（Anthropic Claude Code 负责人）
- **原话**（经 Addy 引用）："I don’t prompt Claude anymore. I have loops running that prompt Claude and figuring out what to do. My job is to write loops."
- **出处**：X 引用帖 https://x.com/rohanpaul_ai/status/2063289804708835412 （源自 Cherny 在 WorkOS Unplugged 活动的发言）
- 注：X 反爬，原帖未直接抓到；上述文字为 Addy 原文引用，链接已存。一手转引可靠。

### 4. Andrej Karpathy（OpenAI 联合创始人、前 Tesla AI 总监，Vibe Coding 提出者）—— ✅ 已找到一手

- **一手原文**：《LOOPS.md: Field Notes on Agents That Run for Days》（loops.md, v060726），作者署名 "Andrej Karpathy, Independent Researcher"。会议体格式重排的工作笔记，全文已落盘 → 本文末尾附录二。
- **关于用户引用的那句**"把你自己从 loop 的执行过程中移除"：Karpathy 确实深度论述了 loop，但**没有这句字面原话**。其本意由 §I 这几句承载（这才是源头）：
  - "A prompt is a thing you type once and forget. **A loop is a thing that runs while you sleep.**"
  - "The unit of leverage stopped being the prompt... what matters now is the procedure."
  - "If you find yourself iterating on a single message at three in the morning, you are still in the prompting era. **Close the tab. Write the loop.**"
  - 二手解读那句是这层的转译，方向对，但非 Karpathy 字面原话。
- **关键原话索引**（详见附录二全文）：
  - **§I 写 loop 不写 prompt**：loop 五动词 "gather, reason, act, verify, repeat"——"Everything in this document is a footnote on those five verbs."
  - **§II 角色分离**：planner / generator / evaluator 三角色三上下文；"the model becomes sycophantic the moment it grades itself"——≈ Addy maker/checker，但 Karpathy 拆成三个。
  - **§III 先谈契约**：generator 与 evaluator 在写码前先就"完成标准"争吵达成 checklist——"the single change that moved my own runs from broken demos to working products"。
  - **§IV 写磁盘不写上下文**："Context windows lie... A file on disk does not lie."——≈ Addy Memory。
  - **§V 允许 loop 推倒重来**：新模型会主动 delete 项目重来，"The restart is the loop working correctly."
  - **§VIII 删除 harness**：模型变强后旧 harness 一半变累赘，"delete anything the model now does for free"——harness 应不断删，单调增长的 harness 是你不再读它了。
  - **§IX 瓶颈永远在移动**：coding→planning→verification→taste，"You do not finish; you find the next thing to fix."
- **与 Addy 的关系**：高度呼应（loop 替你跑 / maker-checker / 记忆落盘 / 内循环五动词），但 Karpathy 更偏实操、更有个人色彩（契约谈判、允许推倒重来、删 harness、瓶颈移动是 Addy 未强调的独观点）。两人观点互补，Karpathy 是 loop engineering 的核心提出者之一，不只"Vibe Coding 之父"。

## 待补充（后续按需抓取）
- [ ] Steinberger / Cherny 的 X 原推全文（需登录或绕反爬）
- [ ] Boris Cherny WorkOS Unplugged 访谈转录
- [x] ~~Karpathy "自移除" 原话~~ → 已找到《LOOPS.md》一手原文（见附录二）

---

# 附录：Addy Osmani《Loop Engineering》全文（2026-06-07）

> 以下为 Addy 原文抓取存档（含导航栏等页面元素，未做精修，保证原始性）。

[ Home ](/) [GitHub ](https://github.com/addyosmani) [Press ](/press) [Biography ](/bio) [LinkedIn](https://www.linkedin.com/in/addyosmani/) [Twitter ](https://twitter.com/addyosmani) [Newsletter](https://addyo.substack.com) [Blog ](/blog)

# Loop Engineering

## June 7, 2026

**Loop engineering is replacing yourself as the person who prompts the agent. You design the system that does it instead.** A loop here can be thought of a recursive goal where you define a purpose and the AI iterates until complete. I believe this may be the future of how we work with coding agents. However, its still early, I’m skeptical and you absolutely _have_ to be [careful](https://x.com/weswinder/status/2063700289710964906) about token costs (usage patterns can vary wildly if you are token rich or poor), so I want to unpack what it is and what it means.

* * *

Peter Steinberger recently [said](https://x.com/steipete/status/2063697162748260627): “You shouldn’t be prompting coding agents anymore. You should be designing loops that prompt your agents.” Similarly, Boris Cherny, head of Claude Code at Anthropic, [said](https://x.com/rohanpaul_ai/status/2063289804708835412) “I don’t prompt Claude anymore. I have loops running that prompt Claude and figuring out what to do. My job is to write loops”.

Okay, so what does any of that mean?

For like two years the way you got something out of a coding agent was you wrote a good prompt and shared enough context. You type a thing, you read what came back, you type the next thing. The agent is a tool and you are holding it the entire time, one turn after the other. That part is kind of over, or at least some think it’s going to be.

Now you build a small system that finds the work, hands it out, checks it, writes down what is done and then decides the next thing, and you let that system poke the agents instead of you. I wrote before about the cousin of this, [agent harness engineering](https://addyosmani.com/blog/agent-harness-engineering/), which is making the environment one single agent runs inside and the [factory model](https://addyosmani.com/blog/factory-model/) \- the system that builds the software. Loop engineering sits one floor above the harness. The harness but it runs on a timer, it spawns little helpers, and it feeds itself.

The thing that surprised me is this is not really a tool thing anymore. A year ago if you wanted a loop you wrote a pile of bash and you maintained that pile forever and it was yours and only yours. Now the pieces just ship inside the products. Steinberger’s list maps almost exactly onto the Codex app, and then almost the same onto Claude Code. And once you notice the shape is the same you stop arguing about which tool, you just design a loop that still works no matter which one you happen to be sitting in.

## The five pieces, and then notes

A [loop](https://x.com/reach_vb/status/2063713960495558940) needs five things and then one place to remember stuff. Let me list it first and then map it.

  1. **Automations** that go off on a schedule and do discovery and triage by themselves.
  2. **Worktrees** so two agents working in paralell dont step on each other.
  3. **Skills** to write down the project knowledge the agent would otherwise just guess.
  4. **Plugins and connectors** to plug the agent into the tools you already use.
  5. **Sub-agents** so one of them has the idea and a different one checks it.



Then the sixth thing, the memory. A markdown file, or a Linear board, anything that lives outside the single conversation and holds what’s done and what is next. Sounds too dumb to matter. But it’s the same trick every long running agent depends on and I went into it in [long-running agents](https://addyosmani.com/blog/long-running-agents/), the model forgets everything between runs so the memory has to be on disk and not in the context. The agent forgets, the repo doesnt.

Both products have all five now.

Primitive | Job in the loop | Codex app | Claude Code  
---|---|---|---  
**Automations** | discovery + triage on a schedule | [Automations tab](https://developers.openai.com/codex/app/automations): pick project, prompt, cadence, environment; results land in a Triage inbox; `/goal` for run-until-done | Scheduled tasks and cron, `/loop`, `/goal`, hooks, GitHub Actions  
**Worktrees** | isolate parallel features | Built-in worktree per thread | `git worktree`, `--worktree`, `isolation: worktree` on a subagent  
**Skills** | codify project knowledge | [Agent Skills](https://developers.openai.com/codex/skills) (`SKILL.md`), invoked with `$name` or implicitly | [Agent Skills](https://addyosmani.com/blog/agent-skills/) (`SKILL.md`)  
**Plugins / connectors** | connect your tools | Connectors (MCP) plus plugins for distribution | MCP servers plus plugins  
**Sub-agents** | ideate and verify | [Subagents](https://developers.openai.com/codex/subagents) defined as TOML in `.codex/agents/` | Task subagents in `.claude/agents/`, agent teams  
**State** | track what’s done | Markdown or Linear via a connector | Markdown (`AGENTS.md`, progress files) or Linear via MCP  
  
The names are a bit different here and there but the capability is the same thing. Let me go one by one because honestly the details are where a loop either holds together or quietly leaks everywhere.

## Automations, this is the heartbeat

Automations are what make a loop an actual loop and not just one run you did once. In the Codex app you make one in the Automations tab and you pick the project, the prompt it will run, how often, and if it runs on your local checkout or on a background worktree. The runs that find something go to a Triage inbox, and the runs that find nothing just archive themselves wich is nice. OpenAI uses them internally for boring stuff like daily issue triage, summarising CI failures, writing commit briefings, hunting bugs somebody added last week. And an automation can call a skill, so you keep the recurring thing maintainable, you fire `$skill-name` instead of pasting a giant wall of instructions into a schedule that nobody will ever update.

Claude Code gets to the same place but through scheduling and hooks. You can run a prompt or a command on a interval with `/loop`, you can schedule a cron task, you can fire shell commands at certain points in the agent lifecycle with hooks, or you push the whole thing to GitHub Actions if you want it to keep running after you close the laptop. Same idea exactly, you define an autonomous task, you give it a cadence, and the findings come to you so you are not the one going around checking.

There is a second in-session primitive worth knowing, and it’s the one closer to what this whole post is about. `/loop` re-runs on a cadence. `/goal` keeps going until a condition you wrote is actually true, and after every turn a separate small model checks whether you are done, so the agent that wrote the code isnt the one grading it. You give it something like “all tests in test/auth pass and lint is clean” and walk away. Codex has the same thing, also called `/goal`, it keeps working across turns until a verifiable stopping condition holds, with pause and resume and clear. Same primitive, both tools, wich is kind of the pattern for this whole article.

So this is the part that surfaces the work. The rest of the loop is what acts on it.

## Worktrees so paralell doesnt turn into chaos

The second you run more than one agent the files start colliding, that becomes the failure. Two agents writing the same file is the exact same headache as two engineers committing to the same lines and nobody talked to each other first. A git worktree fixes it, its a separate working directory on its own branch sharing the same repo history, so one agent’s edits literally can not touch the other one’s checkout.

Codex builds the worktree support right in so several threads hit the same repo at once and dont bump into each other. Claude Code gives you the same isolation with `git worktree`, a `--worktree` flag to open a session in its own checkout, and a `isolation: worktree` setting you stick on a subagent so each helper gets a fresh checkout that cleans itself up after. I wrote about the human side of all this in [the orchestration tax](https://addyosmani.com/blog/orchestration-tax/), the worktrees take away the mechanical collision but YOU are still the ceiling, your review bandwith decides how many you can actually run, not the tool.

## Skills, so you stop explaining your project every single time

A skill is how you stop re-explaining the same project context every session like a goldfish. Both tools use the same format, a folder with a `SKILL.md` inside holding instructions and metadata, and then optional scripts, references, assets. Codex runs a skill when you call it with `$` or `/skills`, or by itself when your task matches the skill description, wich is the reason a tight boring description beats a clever one. Claude Code does it the same way and I wrote the pattern up in [agent skills](https://addyosmani.com/blog/agent-skills/).

Skills are also where intent stops costing you over and over. I argued in [the intent debt](https://addyosmani.com/blog/intent-debt/) that an agent starts every session cold and it will fill any hole in your intent with a confident guess. A skill is that intent written down on the outside, the conventions, the build steps, the “we dont do it like this because of that one incident”, written one time where the agent reads it every run. Without skills the loop re-derives your whole project from zero every cycle, with skills it kind of compounds.

One thing to keep straight, the skill is the authoring format and a plugin is how you ship it. When you want to share a skill across repos or bundle a few together you package them as a plugin. True in Codex, true in Claude Code.

## Plugins and connectors, the loop touches your real tools

A loop that can only see the filesystem is a tiny loop. Connectors, wich are built on MCP, let the agent read your issue tracker, query a database, hit a staging api, drop a message in Slack. Codex and Claude Code both speak MCP so the connector you wrote for one usually just works in the other. And plugins bundle connectors and skills together so your teammate installs your setup in one go instead of rebuilding the whole thing from memory.

This is the difference between an agent that says “here is the fix” and a loop that opens the PR, links the Linear ticket and pings the channel once CI is green by itself. The connectors are the reason the loop can act inside your actual environment instead of just telling you what it would do if it could.

## Sub-agents, keep the maker away from the checker

The most useful structural thing in a loop, by far, is splitting the one who writes from the one who checks. The model that wrote the code is way too nice grading its own homework. A second agent with different instructions and sometimes a different model catches the stuff the first one talked itself into.

Codex only spawns subagents when you ask, runs them at the same time and then folds the results back into one answer. You define your own agents as TOML files in `.codex/agents/`, each with a name, a description, instructions and optional model and reasoning effort, so your security reviewer can be a strong model on high effort while your explorer is some fast read-only thing. Claude Code does the same with subagents in `.claude/agents/` and agent teams that pass work between them. The usual split in both is one agent explores, one implements, one verifies against the spec.

I made this case twice already, once as [the code agent orchestra](https://addyosmani.com/blog/code-agent-orchestra/) and once as [adversarial code review](https://addyosmani.com/blog/adversarial-code-review/). The reason it matters specifically inside a loop is the loop runs while you are not watching, so a verifier you actually trust is the only reason you can walk away. Subagents do burn more tokens since each one does its own model and tool work, so spend them where a second opinion is worth paying for. This is also basically what Claude Code’s `/goal` does under the hood, a fresh model decides if the loop is done instead of the one that did the work, the maker and checker split applied to the stop condition itself.

## What one loop looks like

Stick it together and a single thread turns into a little control panel. Here is one shape I keep using.

An automation runs every morning on the repo. Its prompt calls a triage skill that reads yesterdays CI failures, the open issues, the recent commits, and writes the findings into a markdown file or a Linear board. For each finding that is worth doing the thread opens an isolated worktree and sends a sub-agent to draft the fix, and a second sub-agent reviews that draft against the project skills and the existing tests.

Connectors let the loop open the PR and update the ticket. Anything the loop can not handle lands in the triage inbox for me. The state file is the spine of the whole thing, it remembers what got tried, what passed, what is still open, so tomorrow morning the run picks up where today stopped.

And look at what you actually did there. You designed it one time. You did not prompt any of those steps. Thats Steinberger’s whole point made real, and its the same loop in Codex or in Claude Code because the pieces are the same pieces.

## What the loop still does not do for you

The loop changes the work, it does not delete you from it. And three problems actually get sharper as the loop gets better, not easier.

Verification is still on you. A loop running unattended is also a loop making mistakes unattended. The whole reason you split the verifier sub-agent from the maker is to make the loop’s “its done” mean something, and even then “done” is a claim and not a proof. I keep saying the same line from [code review in the age of AI](https://addyosmani.com/blog/code-review-ai/), your job is to ship code you confirmed works.

Your understanding still rots if you allow it. The faster the loop ships code you did not write, the bigger the gap between what exists and what you actually get. Thats [comprehension debt](https://addyosmani.com/blog/comprehension-debt/) and a smooth loop just makes it grow faster unless you read what the loop made.

And the comfortable posture is the dangerous one. When the loop runs itself its very tempting to stop having an opinion and just take whatever it gives back. I called that [cognitive surrender](https://addyosmani.com/blog/cognitive-surrender/). Designing the loop is the cure when you do it with judgement and the accelerant when you do it to avoid thinking, same action, opposite result.

## Build the loop. Stay the engineer.

I think this is a preview of how our work is going to evolve. That said, If I weren’t reviewing the code myself or if I relied entirely on automated loops to fix it my product’s quality would suffer. I’d likely end up stuck in a downward spiral, continuously digging myself into a deeper hole.

That said, go ahead and set up your loops, but don’t forget that prompting your agents directly is also effective. It’s all about finding the right balance.

Loops can also result in different outcomes depending on you. Two people can build the exact same loop and get completely opposite results. One uses it to move faster on work they understand deeply. The other uses it to avoid understanding the work at all. The loop doesn’t know the difference. You do.

That’s what makes loop design harder than prompt engineering, not easier. Cherny’s point isn’t that the work got easier. It’s that the leverage point moved.

Build the loop. But build it like someone who intends to stay the engineer, not just the person who presses go.

[ ![Beyond Vibe Coding book cover](https://addyosmani.com/assets/images/beyond.webp)](https://beyond.addy.ie)

Enjoyed this?

### Go deeper in _Beyond Vibe Coding_

My O'Reilly book on AI-assisted and agentic engineering: specs, harnesses, evals, context, and shipping production-grade software with AI.

[Read the book](https://beyond.addy.ie)

[ ![](/assets/images/addy_2022.jpg) Addy Osmani is an engineering and evangelism leader who spent over 14 years at Google leading developer experience across Chrome and, in recent years, AI (Gemini, coding agents, and agentic engineering), most recently as a Director at Google Cloud AI. ](http://twitter.com/addyosmani) [ Tweet](https://twitter.com/intent/tweet?text=https://addyosmani.com/blog/loop-engineering/%20-%20Loop%20Engineering%20by%20@addyosmani) [ Bluesky](https://bsky.app/intent/compose?text=Loop%20Engineering%20-%20https://addyosmani.com/blog/loop-engineering/) [ Mastodon](https://mastodon.social/share?text=Loop%20Engineering%0Ahttps://addyosmani.com/blog/loop-engineering/) [ Threads](https://www.threads.net/intent/post?text=Loop%20Engineering%0Ahttps://addyosmani.com/blog/loop-engineering/) [ LinkedIn](https://www.linkedin.com/sharing/share-offsite/?url=https://addyosmani.com/blog/loop-engineering/) Share

Want more? Subscribe to my free newsletter:

**Disclaimer:** The views and opinions expressed on this site are my own and do not necessarily reflect the views, positions, or strategies of Google or any of its affiliates. 

© Copyright 2026 Addy Osmani


---

# 附录二：Andrej Karpathy《LOOPS.md: Field Notes on Agents That Run for Days》全文

> 来源：loops.md v060726，作者署名 "Andrej Karpathy, Independent Researcher"。
> 由用户提供截图（Andrej Karpathy.jpg），经视觉模型逐字转录存档。会议体格式重排的工作笔记。

## Abstract

This file exists because most agent systems die not from a weak model but from a weak harness. The model can write code; the model can review code; the model can verify its own output against a rubric it agreed to ten minutes ago. What it cannot do, on its own, is decide when to stop, when to restart, and where to write the result. That is the work of the loop. The pattern in this note treats the loop as a first-class object: roles are separated, state lives on disk, contracts are negotiated between agents before the first line of code is written, and the harness is read like a stack trace whenever something goes wrong. Short loops, simple state, clean contracts. Everything else is decoration.

Index Terms. agentic loops, Claude Code, harness design, generator-evaluator pattern, sprint planning, file-system state, contract negotiation, trace reading, deletable scaffolding.

## I. WRITE THE LOOP, NOT THE PROMPT

A prompt is a thing you type once and forget. A loop is a thing that runs while you sleep. The unit of leverage stopped being the prompt the moment models became good enough to follow a procedure without supervision; what matters now is the procedure. If you find yourself iterating on a single message at three in the morning, you are still in the prompting era. Close the tab. Write the loop. The loop is short: gather, reason, act, verify, repeat. Everything in this document is a footnote on those five verbs.

## II. SEPARATE THE ROLES

Three roles, three context windows, three system prompts. A planner that turns a vague human sentence into a sprint spec and never touches code. A generator that writes everything and is forbidden from grading its own work. An evaluator that reads diffs, launches playwright, plays the app, and is told from the first message that the code is broken and its job is to prove it. Mixing the roles is the most common failure I see; the model becomes sycophantic the moment it grades itself, and the loop quietly converges on slop.

## III. NEGOTIATE THE CONTRACT FIRST

Before the generator writes a single line, it proposes what done looks like and the evaluator pushes back. The two argue via markdown files on disk until they agree on a checklist of testable assertions. Twenty-seven criteria is a reasonable size for a small app; ten is usually too few and the evaluator rubber-stamps. The original spec from the planner is the boundary, but the contract is what gets graded. This is the single change that moved my own runs from broken demos to working products.

## IV. WRITE TO DISK, NOT TO CONTEXT

Context windows lie. They compact, they rot, they hide what you said an hour ago behind a summary you did not write. A file on disk does not lie. Keep feature_list.json, progress.md, contract.md, and an append-only log.md with ## [YYYY-MM-DD] op | title entries. The model should be able to crash, lose its session, and pick up where it left off by reading three files. If you cannot describe your state in three files, your state is too complicated.

## V. LET THE LOOP RESTART

Counter-intuitively, the best behavior I see from current frontier models is the willingness to throw everything away and start over when a run goes sideways. Older models patched and patched until the codebase resembled archaeology; newer ones, given a clean evaluator and a contract on disk, will delete the project at iteration nine and ship a working version at iteration eleven. Do not interrupt this. The restart is the loop working correctly. Insert a human only when the contract itself is wrong, not when the build is.

## VI. SCORE THE SUBJECTIVE

Taste is gradable if you write it down. Four axes, weighted: design, originality, craft, functionality. Calibrate on three reference sites the evaluator is told are good and three it is told are slop. The output is a number between zero and one and a paragraph explaining the gap. The model will not invent taste; it will only converge toward the taste you described. The whole game is writing the rubric carefully enough that converging toward it is what you actually wanted.

## VII. READ THE TRACES

Every debugging insight I have about agent loops came from reading the raw transcript, not from running another experiment. Pipe the agent's output into a file, grep for the moment its judgment diverged from yours, edit the prompt for that exact moment, run again. This is the same muscle as reading a stack trace; the difference is that the trace is written in English and most of it is the model talking to itself. Skip this step and you are tuning by vibe.

## VIII. DELETE THE HARNESS

The harness exists to compensate for the model. As the model improves, half of what you wrote last quarter becomes overhead. Context resetting between sessions was load-bearing for one model generation and dead weight for the next; sprint decomposition was the only thing keeping a four-hour build coherent and is now a constraint on a model that holds two hours in one head. Re-read your harness against each new release and delete anything the model now does for free. The harness that grows monotonically is a harness you have stopped reading.

## IX. THE BOTTLENECK ALWAYS MOVES

When coding stops being the bottleneck, planning becomes the bottleneck. When planning is solved, verification becomes the bottleneck. When verification is automated, taste becomes the bottleneck. You do not finish; you find the next thing to fix. The whole point of the loop is to make the bottleneck visible. If everything is going smoothly, you are not looking carefully enough. Find the new bottleneck, fix it, ship a smaller harness, repeat.

---

© 2026 A. Karpathy. Personal use of this material is permitted. This is an independent reformatting of working notes on long-running agent loops (loops.md, v060726) into a conference-style document. Freely available; ideas subject to revision as the models change. [AK]
