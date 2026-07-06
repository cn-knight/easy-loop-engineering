# 1. Loop Engineering 概念学习

> 本文档是 loop engineering 的概念问答（基于原典：Addy Osmani + Karpathy《LOOPS.md》）。
> **形式**：按学习提纲分章节的 Q&A，结论沉淀于各章节下。
> **跨智能体**：文中调度/护栏/子代理语法以 Claude Code 为示例说明；Codex 等价做法见 `SKILL.md`「适配其他智能体」映射表。
> **复习指引**：每节若有"概念速览"块，先扫它快速回忆框架，再下钻 Q/A 看推演细节；第 8 节另含三角色端到端运行示例，非纯问答。

---

## 学习提纲（建议顺序）

> 顺序设计逻辑：先定位（这个概念处在什么层级）→ 再拆内循环（模型自带的最小循环）→ 再讲六块积木总览 → 逐块深入（先讲“调度/hook”两块，再讲其余）→ 最后讲风险护栏与落地。每节都对应一个可独立提问的学习单元。

### 0. 范式定位
- Prompt → Context → Harness → Loop 四层技术栈的层级关系与边界
- Loop Engineering 与 Harness Engineering 的区别（"跑道"vs"跑道上的调度系统"）
- 为什么说四层是"叠加"而非"替代"

### 1. Agent Loop 内循环（基础，模型自带）
- 感知 → 推理 → 行动 → 反思 的循环周期
- 这个循环什么时候结束（无工具调用的纯文本响应）
- 它和我们要设计的"外层 Loop"是什么关系

### 2. 六块积木总览
- Automations 调度 / Worktrees / Skills / Connectors / Sub-agents / Memory
- 各自的"职责一句话"
- 六块如何拼成一个完整 loop（Addy 的晨间分诊例子）

### 3. 调度类积木深入
- `/loop`：是什么、基本语法、间隔与 cron、管理命令
- `/goal`：是什么、停止条件、评估器模型
- `/loop` vs `/goal`：怎么选（有终点 vs 无终点）
- `/loop` 的关键限制（若干条，需自己想清楚为什么是限制）

### 4. hook 机制
- hook 是什么、与 `/loop`（按时间）的区别（按事件）
- hook 事件体系（有哪些事件、各在什么时机触发、哪些能阻止动作）
- hook 配置位置（三层优先级）与配置结构
- hook 命令怎么拿输入、怎么阻止一个工具调用
- 广告场景下 PreToolUse / PostToolUse 各能干什么

### 5. `/loop` + hook 协同（loop 的精髓）
- 单用 `/loop` 的风险、单用 hook 的局限
- 两者如何拼成可控的 loop
- hook 在其中扮演的"安全/审计横切层"角色

### 6. Memory / State 积木
- 为什么记忆必须落磁盘、不能放上下文窗口
- STATE.md / LOOP.md / 日志文件各记什么

### 7. Worktrees 积木
- 解决什么问题（多 Agent 并行写同一文件）
- git worktree 的原理
- subagent 的 `isolation: worktree` 怎么用

### 8. Sub-agents 积木
- maker/checker 分离为什么重要（自评偏差）
- 怎么配置独立子 Agent（不同指令/模型）
- implementer / verifier / orchestrator 的分工

### 9. Connectors 积木
- MCP 让 loop 触达真实工具的意义
- "只能看文件系统的 Loop 是小 Loop"
- 广告项目里 ApiClient 作为连接器的角色

### 10. 持久化调度（Automations 积木）
- 会话内 `/loop` 的生命周期限制
- 持久化的几条路（GitHub Actions / Claude Code Routines / Windows 任务计划）
- 各自适用场景

### 11. 风险与护栏（全程意识，非独立模块）
- 验证仍是人的事（"完成了"是声明不是证明）
- 理解债（comprehension debt）/ 认知投降（cognitive surrender）
- 停止条件设计、token 成本控制、人工门禁
- 动钱操作（广告调价）为何不能裸跑
- 护栏的实操 = harness 层约束（subagent/skill 指令禁令 + hook），散在前各节，非新东西

---

## 学习记录（问答结论沉淀于此，按章节追加）

> 以下各章节为概念问答的结论沉淀。未讨论的章节保持空缺。

### 0. 范式定位

**Q：四层技术栈（Prompt → Context → Harness → Loop）是行业共识吗？**

A：不是。三方出处分清：

| 来源                       | 怎么说                                                                                                                              | 性质                                 |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| **橙皮书（花叔）**              | §02"三级跳：从 Prompt 到 Context 到 Loop"，明确写"四层栈，一层管一件事""不是互相替代的，是叠起来的"，并列了四层表                                                         | **二次整理/理论化**，是花叔为讲清 loop 位置搭的脚手架   |
| **Addy Osmani（命名者）原文**   | 没有四层栈框架，全文甚至没出现"context engineering"一词；对 loop 与 harness 只说"Loop sits one floor above the harness"，称 harness 为 loop 的"cousin（表亲）" | **原作者表述松散**，只确认 loop 在 harness 上一层 |
| **Steinberger / Cherny** | 核心论点只有"别再手动 prompt 了，去设计 loop"，出发点是行为转变（操作者→设计者），未梳理四层栈                                                                          | **行为号召**，非分层理论                     |

**结论**：四层栈 + "叠加非替代"是花叔的加工，不是提出者的共识。读二手解读要分清"原作者的话"和"解读者的整理"。

**关于橙皮书的定位**：橙皮书可参考，但不作圣经。它有浓重 AI 写作风（花叔也是让 AI 总结的），AI 在其提示词下总结得是否准确未必然。原典以提出者原文为准。

---

**Q：抛开四层栈理论，loop engineering 的本质到底是什么？**

A（本质理解）：

Loop 的本质很直白——**把人从"人+机循环"里解放出来**。

- **现状（人+机循环）**：无论人给智能体的是一句话提示、文字输入，还是引用文件，本质都是"**人给一下，智能体动一下**"。即便人一句话能驱动智能体做很多事，**人的精力依然被拴在电脑前**——得在场盯着、续上下文、敲下一句。
- **Loop 要解决的**：改为让智能体能**自行进行更长时间和更多轮次的自我任务行进**，人不必每轮都介入。
- **终极方向**：AI 根据**任务目标自驱动**，这样才能有"**一个人管理一个团队**"的感觉——人从"逐句驱动的操作员"变成"定目标、设规则、验收结果的管理者"。

**与原典的呼应**：
- Steinberger："设计 loop，让 loop 去提示你的 agent" —— 即把"人给一下"这件事外包给 loop。
- Cherny："我有 loop 在跑，它们自己决定该干什么" —— 即智能体自行多轮行进。
- Addy："replacing yourself as the person who prompts the agent" —— 即把人从人+机循环里替换掉。

三方表述各异，但落到“把人从人+机循环中解放、让智能体自驱动多轮行进”的本质上，是一致的。**这个本质比"四层栈"更接近原典意图。**

---

**Q：harness 是 loop engineering 的基础和前提吗？harness 是不是厂家的事、我只是使用者？**

A：**核心结论对，一个前提需校准。** 已核实 Addy 的 agent harness engineering 原文。

**结论对的部分**：harness 确实是 loop 的基础和前提。没有 `/goal` 这类稳定的长时间/长任务自主执行能力，人离不开电脑、减不了提示词输入，loop 无从谈起。`/goal` 就是厂家核心 harness 提供的原语，让"智能体自行多轮行进"成为可能，loop 才有东西可包。Addy 原话："Loop sits one floor above the harness"——loop 是站在 harness 肩膀上的编排层。

**需校准的部分**：harness 不只是厂家的事。Addy 的定义比想象中宽得多：

- 公式：**Agent = Model + Harness**。Viv Trivedy："If you're not the model, you're the harness."（非模型本身的全是 harness。）
- Addy 明确列举 harness 包含：CLAUDE.md/AGENTS.md、技能文件、子智能体提示词、MCP 服务器、hooks、沙箱、编排逻辑、可观测性等。
- 且明确："it's *your* surface area, not the model provider's"——harness 是**用户的工程面**，不是厂家的。
- "A harness is a living system, not a config file you set up once."——harness 是随失败历史不断收紧的活系统。

所以 harness 实际分两层：

| 层 | 谁设计 | 例子 |
|----|--------|------|
| 核心 harness（厂家） | Anthropic 等 | agent 内循环、`/goal`、`/loop`、工具调用框架、沙箱 |
| 用户 harness（你） | 你自己 | CLAUDE.md、skills、MCP server、hooks、subagents、权限 |

**关键认知**：你在项目里写的 skill、配的 ApiClient 连接器、全局那个拦凭据的 hook——**这些全是 harness，你早就在做 harness engineering，只是没这么叫。** 只有 agent 内循环这种"核心 harness"才是厂家独占（除非手搓框架）。

**由此带来的更清晰图景**：
- Harness = 让单个 agent 能好好跑完一次的所有非模型部分（厂家核心 + 用户配置）
- Loop = ==在 harness 之上的"时间/调度/编排"层==，让 harness 能反复、自主朝目标跑
- 橙皮书六块积木里 skills/connectors/subagents/memory 大部分**本身就是 harness 组件**，loop engineering 是复用这些 harness 组件、上面加一层"调度+记忆接力+验证门禁"。这解释了六块积木与 harness 为何听起来重叠——**因为本就重叠，loop 是站在 harness 肩膀上的编排层。**

**校准后的一句话**：harness 是 loop 的基础和前提（对）；但 harness 不只是厂家的事——配置 CLAUDE.md/skills/hooks/MCP 的每一刻都已在做 harness engineering（需校准）。只有 agent 内循环这种==核心 harness 才是厂家独占==。

### 1. Agent Loop 内循环

**Q：在智能体里发一句"在桌面上创建一个 md 文件"，它创建完回"已完成"——这算一组内循环吗？loop engineering 提出者讲过内循环这个话题吗？**

A：**算一个完整内循环；但 loop engineering 提出者没把内循环当作议题。** 已核实 Addy 原文。

**Q1 答案——这正是一个内循环：**

内循环（agentic loop）流程：模型收到输入 → 推理 → **要么调工具，要么回纯文本**；若调工具，工具结果送回模型 → 再推理 → 再决定……**反复直到模型给出"无工具调用"的纯文本响应，内循环结束。**

该例拆解：
- 用户发"创建 md 文件"
- turn 1：模型推理 → 调 Write 工具创建文件
- 工具结果返回
- turn 2：模型推理 → 输出"已完成"（无工具调用）
- 内循环结束

→ **一次用户输入 = 一个内循环，内含 2 个 turn（2 次模型调用）。** 用户只敲一次键盘，模型内部自己转了多轮——这个"自己转"就是内循环，是"智能体自行多轮行进"的最小形态，但仍由用户这句话触发、跑完即停。

**关键术语区分**：
- **turn（轮）** = 一次模型调用
- **inner loop（内循环）** = 从一条消息开始、到模型给出无工具调用最终回复为止的整个"反复调工具"周期

**Q2 答案——提出者没讲内循环（核实 Addy 原文）：**

直接抓 Addy 的 loop engineering 原文，结论：
- 通篇讲**外层编排**（Automations/Worktrees/Skills/Connectors/Sub-agents），**未出现"inner loop"术语，未讨论"感知-推理-行动-反思"周期**。
- 唯一涉及"结束"的是 `/goal`："keeps going until a condition you wrote is actually true"——但这是**外层**退出条件（目标达成），非内循环退出条件（无工具调用）。
- Steinberger/Cherny 也是"写 loop 驱动 agent"，不涉及 agent 内部推理循环。

**"内循环"概念真正来源**：Anthropic 官方一般性 agent 文档（讲 agent 怎么工作：perceive→reason→act→reflect、无工具调用即退出），**早于且独立于 loop engineering**，是 agent 基础工作原理。

**问题出在哪**：中文解读（如 colddawn 文章）把两者缝合——先讲"Anthropic 官方画的感知/推理/行动/反思循环"，再讲 loop engineering，给人"提出者讲过内循环"的印象。**这是解读者的拼接，原典（Addy）并未把内循环当主题。** 与上一节对橙皮书的警惕同类：二手解读爱把不同来源概念缝在一起，缝得越顺滑越像原典体系。内循环是 Anthropic agent 文档的，loop engineering 是 Addy 的，二者有依赖（loop 包着内循环），但"内循环"非 loop engineering 提出者议题。

**内循环 vs 外层 loop 对照**：

| 维度 | 内循环（agentic loop） | 外层 loop（loop engineering） |
|------|---------------------|---------------------------|
| 谁讲 | Anthropic 官方 agent 文档（一般原理） | Addy/Steinberger/Cherny（编排方法论） |
| 触发 | 用户的一条消息 | 调度（时间/目标），无需用户每次输入 |
| 跑什么 | 模型反复调工具直到给纯文本 | 反复触发内循环、接力记忆、验证门禁 |
| 退出条件 | 无工具调用的文本响应 | 目标达成（`/goal`）或用户取消 |
| 该例对应 | 就是 1 个内循环 | 未涉及——用户仍亲自触发了它 |

**一句话**：用户那句话触发了一个内循环，但**还不是 loop engineering**——因为是用户亲自敲的、敲完即停。loop engineering 要做的是连"敲这一下"都由系统按节奏/目标替用户完成，让内循环能被反复、自主地触发。

### 2. 六块积木总览

**Q：六块积木到底是 Addy 原典还是二手解读？每块职责是什么？**

A：已核实 Addy 原文。**"五块积木 + 一层记忆"是 Addy 原话**，不是花叔编的。Addy 原句："A loop needs five things and then one place to remember stuff."（一个 loop 需要五样东西，外加一个记事的地方。）

**逐块职责（Addy 原话 + 一句话理解）**：

| 积木 | Addy 原话（职责） | 一句话理解 |
|------|------------------|-----------|
| 1. Automations（调度） | "go off on a schedule and do discovery and triage" / "discovery + triage on a schedule" | 让 loop 成为"循环"而非"跑一次"的心跳；按节奏自己发现工作、分诊 |
| 2. Worktrees（工作树） | "so two agents working in parallel dont step on each other" / "isolate parallel features" | 多 agent 并行时不踩脚，各自独立工作目录共享同一仓库历史 |
| 3. Skills（技能） | "write down the project knowledge the agent would otherwise just guess" / "codify project knowledge" | 把项目知识写下来，别让 agent 每次像金鱼一样重新猜 |
| 4. Plugins & Connectors（插件与连接器） | "plug the agent into the tools you already use" / "connect your tools" | 基于 MCP，让 agent 能动真工具（开 PR、查数据库、发通知） |
| 5. Sub-agents（子代理） | "so one of them has the idea and a different one checks it" / "ideate and verify" | maker/checker 分离，写代码的别给自己判卷 |
| 6. Memory / State（记忆） | "anything that lives outside the single conversation" / "track what's done" | 跨会话持久记忆，"agent 会忘，仓库不会" |

**三个二手解读常略过的原典细节**：

1. **Worktrees 的真相**：Addy 点明 worktree 只解决"机械冲突"，但"**YOU are still the ceiling**"——你审查的带宽才决定能并行多少个，工具不是瓶颈，你是。落到动钱场景：一个人盯，能并行的 loop 数量受限于你的验收精力。
2. **Skills vs Plugins 是内容与包装的关系**，非并列：Addy 原话"the skill is the authoring format and a plugin is how you ship it"。项目里写的 SKILL.md 是 skill；打包给队友一键装才叫 plugin。
3. **Connectors 的本质区别**：Addy 对照——"agent 告诉你该怎么修" vs "loop 自己开 PR、关联工单、CI 绿了 ping 频道"。项目里 ApiClient 能查数据是 connector 初级形态，能让 loop 自己执行调价+写日志+推送告警才是完整价值。

**关键认知（六块积木的真实构成）**：

六块积木不是六块"全新东西"，而是：
- **四块 harness 组件**（Skills / Connectors / Sub-agents / Memory）——即上一节确认的"用户工程面"，用户早就在配
- **一块并行隔离**（Worktrees）
- **一块调度**（Automations）——这才是 loop 层真正新增的，把 harness 反复触发、自主运转起来

→ **loop engineering 的新意主要在 Automations，其余是复用 harness。** 与上节"loop 站在 harness 肩膀上"完全吻合。看懂这点，就不会觉得六块积木是全新负担——大半是已用能力的重新编排。

**注脚：网上解读偏 Addy 是"传播路径依赖"，非行业共识；Karpathy 的实操哲学被淹没**

为什么中文圈讲 loop engineering 的文章几乎都在复述"五/六块积木 + 记忆"？不是因为这套框架是行业公认最对，而是**传播路径依赖**：

- 四位提出者里，**只有 Addy 给出了结构化、可列表、可复述的框架**（"A loop needs five things and then one place to remember stuff."）。他的 2026-06-07 长文是第一篇系统化命名+拆解 loop engineering 的文章，自然成了所有后续解读的母本。
- Steinberger / Cherny 只有一句口号，撑不起一篇文章；Karpathy《LOOPS.md》是九条实操规则（§I-§IX），不像"理论框架"，不便复述。
- 解读文章要写东西，自然全抄 Addy 的积木框架——最好转述、最像"知识体系"。

**代价**：Karpathy《LOOPS.md》的独到贡献被淹没——这些在 Addy 五块积木里**没有对应物**，抄 Addy 的人抄不到：
- §III 契约谈判（generator 与 evaluator 先就"完成标准"争吵达成 checklist）
- §V 允许 loop 推倒重来
- §VIII 删 harness（模型变强后旧 harness 一半变累赘）
- §IX 瓶颈永远在移动（coding→planning→verification→taste）

**快速判别法**：看到任何讲 loop engineering 的中文文章——通篇只讲"五/六块积木+记忆"→ 基本只读过 Addy；若还讲到"契约/推倒重来/删 harness/瓶颈移动"→ 读过 Karpathy 原文。大多数文章只满足前者。

→ **结论**：六块积木是 Addy 的整理，好用但非唯一体系。学 loop engineering 不能只看 Addy，Karpathy 的实操哲学是 Addy 框架之外的重要补充（两人观点互补，详见 `0_canonical-texts.md` 附录一、二）。这也印证本文一贯原则：以原典为准，警惕二手解读的单一信源偏差。

### 3. 调度类积木（/loop、/goal）

**Q：`/loop` 和 `/goal` 与六块积木是什么关系？属于 Automations 吗？**

A：已核实 Addy 原文。**两者都属于 Automations（调度）这块积木**，是其中最轻量的"会话内原语"。

**Automations 是 loop 的心跳**（Addy 小标题原话："Automations, this is the heartbeat"）：
> "Automations are what make a loop an actual loop and not just one run you did once."
>（Automations 让 loop 成为真正的循环，而不是跑一次就完的事。）

**Addy 把调度手段分两类**：

```
Automations（积木，loop 的心跳）
├── 持久化调度（关掉电脑也能跑）
│   ├── GitHub Actions
│   ├── cron 定时任务
│   └── Automations tab（Codex 那种）
│
└── 会话内原语（in-session，关掉终端就停）
    ├── /loop   —— 按节奏重跑（cadence 驱动）
    └── /goal   —— 跑到条件成立为止（condition 驱动）
```

==`/loop`、`/goal` 与 GitHub Actions、cron 是**并列同级**关系，都归 Automations 下==，区别在"持久化"还是"会话内"。

**`/loop` vs `/goal` 的本质区分**（Addy 原话）：
> "`/loop` re-runs on a cadence. `/goal` keeps going until a condition you wrote is actually true, and after every turn a separate small model checks whether you are done, so the agent that wrote the code isnt the one grading it."

- `/loop` = cadence 驱动（按==节奏重跑==，"每 N 分钟跑一次"）
- `/goal` = condition 驱动（跑到条件成立为止，"测试全过就停"）

**`/goal` 横跨两块积木**：它属于 Automations（调度方式），但其"独立小模型判定完成"机制用了 Sub-agents 的 maker/checker 思想——Addy 原话"the maker and checker split applied to the stop condition itself"。说明积木之间不隔离，会互相组合（呼应第 2 节"六块积木大半是 harness 组件的重新编排"）。

**`/goal` 的理解确认**：

理解：通过 `/goal` ==给智能体提交需求后，要明确**可验证的交付结果**，让智能体知道 goal 达成是可验证的==。—— **理解准确，与 Addy 原典吻合。** Addy 原话："keeps going until a condition you wrote is actually true"，例子 "all tests in test/auth pass and lint is clean"。

两个关键点（均与原典一致）：
1. "可验证的交付结果" = Addy 的 "verifiable stopping condition"——==必须给一个机器能判定真假的硬条件。==
2. ==判定达成者**独立于执行者**："==after every turn a separate small model checks"——写代码的模型不给自己判卷（maker/checker 应用到停止条件本身）。

**补充判别点（引子，详见风险护栏章节）**：`/goal` 的"可验证"必须是**机器可判定的硬条件**（测试通过 / lint 干净 / 类型检查通过）。"UI 好看吗""方案合理吗"这类需人判断的软条件，独立小模型也判不了——这是 loop 的验证盲区（"命令 exit 0 ≠ 产品没问题"）。

---

**Q：`/loop` 怎么用？Codex 有对应命令吗？**

A：已核实 Codex 官方文档。**`/loop` 是 Claude Code 内置命令，Codex 无同名 slash 命令。**

**Codex 的对应物**（核实 codex 官方文档导航）：Codex 的 Automation 分类下是 Non-interactive Mode / Codex SDK / App Server / GitHub Action + Automations tab（GUI 面板），无 `/loop` 命令。设计哲学差异：

| | Claude Code | Codex |
|---|---|---|
| 周期调度（cadence） | **`/loop` 内置 CLI 命令** | 外部调度：Automations tab（GUI）/ cron / GitHub Action 包住 non-interactive 模式 |
| 跑到目标达成（condition） | `/goal` | `/goal`（同名） |

本质区别：Claude Code 把"按节奏重跑"做进 CLI 本身（一条命令）；Codex 把周期调度当**外部编排**——CLI 只负责"跑一次"，循环靠 GUI 面板或外部 cron/GitHub Action 反复触发 non-interactive 模式。两边都覆盖 cadence 调度，Claude Code 多了个 CLI 内轻量入口。`/goal` 两边通用。

**`/loop` 本身**：Claude Code 内置 `/loop`（描述 "Run a prompt or slash command on a recurring interval (e.g. `/loop 5m /foo`). Omit the interval to let the model self-pace."）→ 可用。

基本语法：`/loop <间隔> <要跑的 prompt 或 slash 命令>`
- 间隔：`5m` / `1h` / `1d`；支持 cron 表达式
- 省略间隔 → self-pace（模型自己决定频率：活跃时勤、静默时疏）

**`/loop` 的四个关键限制**（想清楚"为什么是限制"）：

1. **会话内生命周期**：默认只活在当前会话，关掉 Claude Code 就没了。持久化需用 Routines/Desktop 计划任务/GitHub Actions（详见第 10 节）。→ *原因：纯内存调度，进程死则循环死。*
2. **只在 REPL 空闲时触发**：下一轮等当前对话空闲才注入，不打断你。→ *原因：避免抢占，但"准点性"不保证。*
3. **没有自动停止条件**：按节奏一直跑，不像 `/goal` 跑到条件成立就停；只能手动 `/loop cancel`。→ *原因：cadence 驱动，无"完成"概念。**这正是风险操作绝不能裸跑 `/loop` 的原因——它会一直跑一直执行，没人喊停。***
4. **每轮都是全新 agent 调用 + token 消耗**：每次触发是一次完整内循环，间隔越短任务越重烧 token 越快；无"接着上次"的天然记忆。→ *原因：每轮从零开始，必须配 STATE.md 让它在轮间接力（呼应 Memory 积木）。*

**`/loop` vs `/goal` 选用**：
- 有明确终点 → `/goal`（做完即停，安全）
- 无终点、周期性监控/巡检 → `/loop`
- **动钱/危险操作 → 优先 `/goal`**，绝不要裸放 `/loop`

→ 第 3 条（无自动停止）是 `/loop` 与 `/goal` 最本质的安全差异，也是 loop engineering 强调"验证门禁/停止条件"的原因——`/loop` 本身不提供这些，得靠 hook + STATE.md 补（见第 4、5 节）。

**误区澄清：==`/goal` 不能替代 `/loop`，二者驱动模型不同**==

常见误区：曾以为用 `/goal` 告诉智能体"每隔 X 小时抓一次数据"，和用 `/loop` 没本质差异，唯一差别是 `/goal` 会停止；`/loop` 只是在服务器做长时间任务部署，类似 cron 驱动 py 脚本。—— **"类似 cron"的类比基本对，但"`/goal` 能替代 `/loop`"和"没本质差异"需校准。**

用"每隔 X 小时抓一次数据"拆解：
- **`/goal` 跑会怎样**：`/goal` 机制是"跑到可验证条件成立就停"。给它"抓数据"，它跑完第一次就面临"停止条件是什么"——若写"抓一次"，抓完即停，**不会每隔 X 小时再抓**；若想让其反复抓，得给一个不可达成条件，但 `/goal` 设计是"达成即停"，要么跑到 token 上限熔断，要么在"是否 done"上反复纠结。**这不是 `/goal` 该干的事。**
- **`/loop 6h` 跑会怎样**：不需要停止条件，每 6 小时自动触发一次"抓数据"内循环，抓完本轮等下一次。天然咬合"周期性巡检"。

→ **该例中 `/loop` 能干、`/goal` 干不了，不是"没差异"。** `/goal` 是"做完即走"，`/loop` 是"按时打卡"。无终点的周期任务交给 `/goal` 是工具错配。

驱动模型根本不同：

| | `/goal` | `/loop` |
|---|---|---|
| 驱动 | 条件（condition） | 节奏（cadence） |
| 触发次数 | 1 次启动，内部反复跑到条件成立 | N 次启动，每次到点触发 |
| 何时结束 | 条件成立自动停 | 永不自动停，手动 cancel |
| 适合 | "做完这件事就走"（一次性长任务，内部可多轮） | "按时反复做这件事"（周期性短任务） |

感觉“差不多”的原因：把两者都想象成"agent 在干活"，看着像。但驱动模型完全不同。

"会停止"不是孤立唯一差异，是冰山一角，下面连着：触发模型（一次多轮 vs 多次一轮）→ token 消耗模式不同；状态延续（`/goal` 会话内连续 vs `/loop` 每轮独立需 STATE.md 接力）→ Memory 积木对 `/loop` 更关键。

**反向验证**：若 `/goal` 真能替代 `/loop`，Claude Code 不必同时内置两者，Addy 也不会并列列为两种 in-session 原语。并存正是因为"有终点的一次性长任务"与"无终点的周期性巡检"是两类真实不同需求。

**一句话校准**：`/loop` 与 `/goal` 非替代关系，是"周期巡检"与"一次性达标"两种驱动模型的并列工具。"定时抓数据"是 `/loop` 本职、`/goal` 干不了。

### 4. hook 机制

> **概念速览（复习用）**
>
> **hook 是什么**：预先写进 Claude Code 配置文件（settings.json）、在生命周期事件点**自动触发**的规则。不是对话里临时喊一声，而是"配好不用管、每次自动生效"。本质是"**when 事件 → do 动作**"的条件-动作线——把"当 X 发生时该干什么"从模型对话推理里抽出来，固化成 harness 层的确定性脚本（模型可能忘，脚本不会）。
>
> **配在哪**：settings.json 三层，作用域从大到小——全局「`~/.claude/settings.json`」（所有项目）/ 项目级「`<项目>/.claude/settings.json`」（仅本项目）/ 本地「`.claude/settings.local.json`」（仅本机+本项目，通常 gitignore）。配在哪一层决定规则的作用范围。
>
> **配置结构（when → do）**：
>
> ```json
> {
>   "hooks": {
>     "事件名": [            ← when：条件（如 PreToolUse）
>       {
>         "matcher": "Edit|Write",   ← 收窄影响范围：只匹配特定工具/场景
>         "hooks": [
>           {
>             "type": "command",     ← do：handler 形态（command/http/prompt）
>             "command": "脚本或命令"  ← 实际动作，由 harness 执行非模型
>           }
>         ]
>       }
>     ]
>   }
> }
> ```
>
> - **条件（when）** = 事件名 + matcher。30 个事件分三类 cadence（每会话/每 turn/每工具调用）；matcher 进一步缩窄（如 `Edit|Write` 只在编辑文件时触发），减少无谓触发。
> - **动作（do）** = handler，三种形态：`command`（跑 shell 命令/脚本，输入经 stdin JSON）、`http`（POST 到 URL）、`prompt`/`agent`（让模型处理）。多数是 `command`——你自己写的一段脚本。
> - **执行者是 harness，不是模型**：handler 是纯代码逻辑（grep、调 API、写日志），不经模型推理；跑完可返回 decision（如 `deny` 拦掉工具调用）喂回 harness。
> - **拦截/放行范式**：命令型 handler 用 exit code——exit 2 + stderr 拦截（原因回喂给 Claude），exit 0 放行；或返回 `decision:"block"` 的 JSON。
> - **与 `/loop` 正交**：`/loop` 调度模型（到时间跑一轮 agent），hook 调度确定性代码（到事件跑一段脚本）。
>
> 下面是逐项问答的结论沉淀。

**Q：Claude Code 和 Codex 都有 hook 机制吗？**

A：已核实。**两边都有 hook，但成熟度和形态差很多。**

- **Claude Code：有，且是一等公民。** 官方原生，配在 settings.json，事件体系完整（官方实际有 ~30 个事件，覆盖会话/turn/工具调用各层级）。很多人全局 settings.json 里可能已配过 PreToolUse hook（如拦含凭据文件的 Edit/Write）——**实战用过 hook 却没意识到那是 hook**，并非从零开始。
- **Codex：也有，但来得晚、形态更碎。** 2026 年才逐步完善（早于此时无 hook），用 `hooks.json` 或 `config.toml` 内联配置，非 Claude Code 的 settings.json 三层结构。生态更碎，出现 `hookstack-cli`（一行装 Claude Code hook，也支持 Codex/Copilot）、`@codex-control/hooks` 等第三方桥接包。

| | Claude Code | Codex |
|---|---|---|
| hook 原生支持 | 是，一等公民，settings.json 直接配 | 有，但来得晚，hooks.json/config.toml |
| 成熟度 | 事件体系完整，文档完善 | 逐步完善中，部分场景靠社区包 |
| Claude Code | 已可用 | 逐步完善中 |

**学习方向决定**：以 Claude Code 为准学 hook（最成熟、能直接落地），不并行讲 Codex 避免混淆。Codex hook 等需要时再单独查。

**衔接上一节**：`/loop` 没有自动停止条件（第 3 节第 3 条限制），`/goal` 跑完即停——"风险操作要拦截、操作完要记录"这类横切需求谁来管？就是 hook。hook 是按**事件**触发的横切层，与 `/loop`（按时间）正交。

**Q：全局已有的那个 PreToolUse hook 是谁配的？**

A：查了配置内容（拦 `.env/auth_code/password/secret`，中文报错"禁止编辑含凭据的敏感文件"）。**大概率是某次对话里 agent 应安全需求配的，非 Claude Code 官方自带。** 理由：官方出厂 settings.json 不预装业务 hook；中文报错+定制关键词+bash/grep 写法是 agent 定制痕迹。Claude Code 不会"偷偷自动写 hook"，是某次对话 agent 提议并执行、授权了但没意识到那是"配 hook"。

**关键认知：hook 的使用场景不是"对话里喊一声"**

==hook = 预先配进 settings.json、在生命周期事件点**自动触发**的规则，**不是对话中临时召唤**。类比烟雾报警器：预先装好、检测到烟自动响，不用每次做饭喊它。==

- 对话里说"帮我改文件"= 做饭；PreToolUse hook = 报警器，自动检查"文件含不含密码"，含就拦——全程不用喊它。
- "使用 hook"的真实动作 = 一次性把规则写进 settings.json（手写或让 agent 帮写），之后长期自动生效，跟你对不对话无关。
- 对比：对话里叮嘱 = 每次都得说、只对该轮有效；hook = 一次配好、不用再说、每次自动生效。hook 价值恰在"不用每次说"。

→ “不会用 hook”是错觉：hook 本就是“配好不用管”，很多人已被配过（凭据 hook 长期在跑）但没意识到。

**Q：hook 的事件体系是怎样的？**

A：已核实 Claude Code 官方文档（docs.claude.com/en/docs/claude-code/hooks，2026-07 抓取）。**官方正好 30 个 hook 事件**，按触发 cadence 分三类：每会话一次（SessionStart/SessionEnd 等）、每 turn 一次（UserPromptSubmit/Stop/StopFailure 等）、agentic loop 内每次工具调用（PreToolUse/PostToolUse 等）。

**全量事件表（30 个，官方原表内容，按事件名字母序排列）**：

| #   | 事件                    | 中文功能简述                                          | 何时触发                                                |
| --- | --------------------- | ----------------------------------------------- | --------------------------------------------------- |
| 1   | `ConfigChange`        | 配置文件在会话期间变更时                                    | 配置文件改动                                              |
| 2   | `CwdChanged`          | 工作目录改变时（如 Claude 执行 `cd`）                       | 工作目录切换（适配 direnv 等）                                 |
| 3   | `Elicitation`         | MCP server 在工具调用中请求用户输入时                        | MCP 请求输入                                            |
| 4   | `ElicitationResult`   | 用户回应 MCP elicitation 后、回执送回 server 前            | MCP 输入返回前                                           |
| 5   | `FileChanged`         | 被监视文件在磁盘上变化时                                    | 文件改动（matcher 指定文件名）                                 |
| 6   | `InstructionsLoaded`  | CLAUDE.md 或 `.claude/rules/*.md` 加载进上下文时        | 会话开始及懒加载时                                           |
| 7   | `MessageDisplay`      | assistant 消息文本显示时                               | 消息流式显示中                                             |
| 8   | `Notification`        | Claude Code 发系统通知时                              | 通知触发                                                |
| 9   | `PermissionDenied`    | auto 模式分类器拒绝工具调用时                               | 自动拒绝时（返回 retry:true 可让模型重试）                         |
| 10  | `PermissionRequest`   | 权限对话框弹出时                                        | 权限弹窗                                                |
| 11  | `PostCompact`         | 上下文压缩完成后                                        | 压缩后（仅观察）                                            |
| 12  | `PostToolBatch`       | 一整批并行工具调用完成后、下次模型调用前                            | 并行批次结束                                              |
| 13  | `PostToolUse`         | 工具调用成功后                                         | 工具成功后（仅观察）                                          |
| 14  | `PostToolUseFailure`  | 工具调用失败后                                         | 工具失败后（仅观察）                                          |
| 15  | `PreCompact`          | 上下文压缩前                                          | 压缩前（可阻止压缩）                                          |
| 16  | `PreToolUse`          | 工具调用执行前（可拦截）                                    | 每次工具调用前                                             |
| 17  | `SessionEnd`          | 会话终止时                                           | 会话结束                                                |
| 18  | `SessionStart`        | 会话开始/恢复时                                        | 会话开始或 resume                                        |
| 19  | `Setup`               | 仅初始化模式的一次性准备                                    | `--init-only`/`--init`/`--maintenance`（CI 或脚本一次性准备） |
| 20  | `Stop`                | Claude 响应完成时                                    | 一轮结束（可阻止结束）                                         |
| 21  | `StopFailure`         | 因 API 错误 turn 结束时                               | API 错误终止（输出/exit code 被忽略）                          |
| 22  | `SubagentStart`       | 子 Agent 被派生时                                    | 子 Agent 启动                                          |
| 23  | `SubagentStop`        | 子 Agent 完成时                                     | 子 Agent 结束（可阻止停止）                                   |
| 24  | `TaskCompleted`       | 任务被标记完成时                                        | 任务完成                                                |
| 25  | `TaskCreated`         | 通过 TaskCreate 创建任务时                             | 任务创建                                                |
| 26  | `TeammateIdle`        | agent team 队友即将空闲时                              | 团队队友 idle                                           |
| 27  | `UserPromptExpansion` | 用户输入的命令展开成 prompt、到达 Claude 前                   | slash 命令展开时（可阻止展开）                                  |
| 28  | `UserPromptSubmit`    | 用户提交 prompt、Claude 处理前                          | 每次提交消息                                              |
| 29  | `WorktreeCreate`      | 通过 `--worktree` 或 `isolation:"worktree"` 创建工作树时 | 工作树创建（替换默认 git 行为）                                  |
| 30  | `WorktreeRemove`      | 工作树被移除时（会话退出或子 Agent 结束）                        | 工作树移除                                               |

> 来源：docs.claude.com/en/docs/claude-code/hooks 的 "Event | When it fires" 原表，30 条逐一对应。日常最常用的是 `PreToolUse`（拦截危险操作）、`PostToolUse`（记日志/跑验证）、`UserPromptSubmit`（注入上下文）、`Stop`（结束时 commit STATE）、`SessionStart`（注入项目背景）这五个，其余按场景按需用。

**关键认知**：
- 只有带"decision control"的事件能**阻止**动作：`PreToolUse`/`UserPromptSubmit`/`Stop`/`SubagentStop`/`PreCompact`/`UserPromptExpansion` 等；`PostToolUse`/`Notification`/`MessageDisplay`/`SessionEnd` 等只能**观察记录**，不能拦。
- hook 输入：官方通过 **stdin 传 JSON**（含 tool_name、tool_input、session 信息等）；HTTP hook 走 POST body。
- 拦截方式：命令 exit 2，或输出 `BLOCK: <原因>`（PreToolUse 专用），或返回 `decision:"block"` 的 JSON；放行 exit 0。

→ 那个凭据 hook 就是 PreToolUse + matcher `Edit|Write` + grep 命中敏感词则 `echo BLOCK:`，是"可阻止事件 + stdin 拿输入 + BLOCK 拦截"的标准范式。

**Q：hook 配置的"条件-动作"结构是怎样的？执行动作的是模型还是 harness？**

A：理解校准。==hook 本质是"**when `事件`, do `handler`**"的条件-动作线：==

> when `UserPromptSubmit`（条件，由 Claude Code 在生命周期点自动检测）→ Claude Code 运行 你在 settings.json 里挂的 handler（动作）

- **条件**（when）= 事件名，Claude Code 在生命周期点自动检测，不用对话里说
- **动作**（do）= 你挂的 hook handler，==三种形态：==`command`（跑 shell 命令/脚本，输入经 stdin JSON）、`http`（POST 到你给的 URL）、`prompt`/`agent`（让模型处理）。多数是 `command`——==你自己写的一段脚本==

**关键校准：执行动作的是 harness（Claude Code），不是模型（Claude）。**

- ==handler 是**纯代码逻辑**（grep、调 API、写日志），不经模型推理==——这点和 `/loop`（调度模型跑一轮）正交
- handler 跑完可==返回一个 **decision**（如 PreToolUse 返回 `deny` 拦掉这次工具调用），==这个 decision ==喂回 harness 影响后续流程==；但 ==handler 本身的执行不依赖模型==
- 把"当 X 事件发生时==该干什么"从**模型对话推理**里抽出来，固化成 **harness 层的脚本规则**——==不用每次对模型说、==不用模型记住、且确定性执行==（模型可能忘，脚本不会）

用凭据 hook 落到该模型：

> when `PreToolUse`（工具是 Edit|Write）→ Claude Code 把 tool_input 经 stdin 传给 bash 脚本 → 脚本 grep 是否含 .env/password/... → 命中则 `echo "BLOCK: ..."` → Claude Code 拦掉这次 Edit/Write；没命中则 `exit 0` 放行

==全程拦截判断由脚本完成，无模型参与。==

**hook vs `/loop` 的正交关系**：`/loop` = 到时间点触发一次 agent（调度模型）；hook = 到事件点跑一段你的脚本（调度你的代码）。==一个调度模型推理，一个调度确定性代码。==

**Q：hook 配置位置（三层优先级）与配置结构？以一个实际案例说明。**

A：本节以本项目的真实案例收尾——既是配置位置与结构的示范，也是配 hook 的完整走查。

**背景（出了什么问题）**：写 markdown 文档时，若用了裸尖括号占位符（如 `<handler>`）。Obsidian 实时预览按 markdown 规则把它当**未闭合的 HTML 开标签**，从该行起进入"等待闭合标签"状态，把后续内容全吞进 raw HTML 区不渲染——表现为该行以下 Alt+L 高亮失效、围栏代码块等标记裸露。删掉围栏、改引用块都没用，因为元凶是尖括号本身。

**为什么用 hook 解决**：靠"我记住不写裸尖括号"不可靠（模型可能忘，对话压缩后尤其）。==hook 把"当写 .md 时检查裸尖括号"这根条件-动作线从模型推理里抽出来，固化成 harness 层的确定性脚本==——拦截判断由代码做，不依赖模型记忆。这正是 hook 与 `/loop` 的分工：一个调度模型，一个调度确定性代码。本案例就是把第 4 节学到的"条件-动作 + handler 是脚本非模型"立刻用上。

**配置位置（三层优先级）**：Claude Code 的 settings.json 有三层，优先级从低到高：

| 层 | 路径 | 作用域 |
|----|------|--------|
| 用户级（全局） | `~/.claude/settings.json` | 所有项目（已有的凭据 hook 在此） |
| 项目级 | `<项目>/.claude/settings.json` | 仅本项目 |
| 本地级 | `<项目>/.claude/settings.local.json` | 仅本项目+本机（通常 gitignore，不提交） |

==本案例选项目级==（`.claude/settings.json`），因为这条规则只针对本项目 Obsidian 文档，不该影响别的项目。

**配置结构**：

```
{
  "hooks": {
    "PreToolUse": [               ← 事件名（when）
      {
        "matcher": "Edit|Write", ← 缩窄到只匹配这两类工具
        "hooks": [
          {
            "type": "command",    ← handler 形态：跑 shell 命令
            "command": "bash -c 'python .../check_bare_angle.py'"
          }
        ]
      }
    ]
  }
}
```

整条条件-动作线：==when `PreToolUse`（工具是 Edit|Write）→ Claude Code 把 tool_input 经 stdin JSON 传给 Python 脚本 → 脚本判断 → 命中则 exit 2 拦截，否则 exit 0 放行。== 全程无模型参与。

**handler 脚本逻辑**（`check_bare_angle.py`，要点）：
1. 从 stdin 读 JSON，取 `tool_name`、`tool_input.file_path`、`tool_input.content`/`new_string`
2. 非 `.md` → exit 0 放行（只管 markdown 文档）
3. 先去掉围栏代码块 ``` 和行内代码 span `...`（里面的尖括号是代码，不该拦）
4. 在剩余文本里找 `<...>`，排除：闭合标签 `</x>`、自闭合 `<br/>`、注释 `<!--`、autolink `<https://...>`、带属性/空格的 `<a href>`、合法 HTML 标签（`<br>` 等）
5. 剩下的纯占位符（如 `<handler>`、`<事件>`）→ stderr 写原因 + exit 2（PreToolUse 拦截，原因回喂给 Claude）；否则 exit 0
6. **fail-open**：JSON 解析失败/内容为空一律 exit 0——lint 类护栏不能瘫痪编辑

**关键认知小结**：
- 配置在哪一层，决定了规则的**作用域**；项目级规则放项目 `.claude/settings.json`，不污染全局
- ==matcher 缩窄（`Edit|Write`）能减少无谓触发==，不每次工具调用都跑脚本
- ==拦截靠 exit 2 + stderr，放行靠 exit 0；这是命令型 hook 的通用范式==（与凭据 hook 同构，那个是 bash+grep，这个是 python+正则）
- fail-open 是护栏类 hook 的正确默认：宁可漏过也别卡死正常编辑
- 项目级 settings.json 通常需**重启会话**才加载

→ 本节收尾：从“不知道 hook 怎么用”到亲手配一个项目级 PreToolUse hook，把第 4 节全部要点（事件体系/条件-动作/三层配置/stdin 输入/exit 拦截/fail-open）一次走通。下一节 `/loop`+hook 协同将讨论如何把这种确定性护栏接入周期性 loop。

### 5. /loop + hook 协同

> **概念速览（复习用）**
>
> **为什么要合起来再学**：单独学 `/loop` 和 hook 时是"零件"，合起来才是"机器"——loop engineering 的精髓不在零件本身，而在它们怎么咬合。
>
> **两者正交**：`/loop` 在**时间轴**上触发（到点跑一轮 agent，提供"心跳"）；hook 在**事件轴**上触发（到事件跑一段脚本，提供"刹车与仪表"）。一个调度模型推理，一个调度确定性代码。
>
> **单用的局限**：单 `/loop` 跑风险操作 = 没刹车的车（无自动停止条件）；单 hook = 一堆静态护栏，不会主动发起任何事。==只有心跳没刹车是危险的，只有刹车没心跳是死的==。
>
> **合起来才是可控 loop**：把时间轴的循环 + 事件轴的护栏叠在一起——`/loop` 让 agent 反复自主跑，hook 在每次心跳的关节点（工具调用前/后、turn 结束）插入确定性检查、记录、拦截。
>
> 下面是逐项问答的结论沉淀。

**Q：为什么要把学过的 `/loop` 和 hook 合起来再学一遍？**

A：因为单独学时是"零件"，合起来才是"机器"。loop engineering 的精髓不在零件本身，而在它们怎么咬合。

**单独学的局限**：

- `/loop` 单学时已知==最致命的限制是"没有自动停止条件"==——按节奏一直跑、一直执行，没人喊停。单用 `/loop` 跑风险操作（如广告调价）= 放一辆没刹车的车。
- hook 单学时已知它是"==到事件跑一段脚本=="，但 hook ==自己不会主动发起任何事==——只能被动等事件。==单用 hook 没有循环==，只是一堆静态护栏。

**合起来才是 loop engineering**：

- `/loop` 提供"==心跳=="——让 agent 反复自主跑起来（loop 的"循环"性）。
- hook 提供"==刹车与仪表=="——在每次心跳的关节点（工具调用前/后、turn 结束）==插入确定性检查、记录、拦截==。
- 两者正交：==`/loop` 在**时间轴**上触发，hook 在**事件轴**上触发。==一个 loop 真正可控，是==把时间轴的循环 + 事件轴的护栏叠在一起==。

**动钱场景一句话**：`/loop 6h` ==让 agent 每 6 小时巡检数据（循环）==，但在 PreToolUse 上挂 hook 拦掉任何调价/扣款操作（护栏），在 PostToolUse 上挂 hook 记日志（审计）——这才是一个"敢放手让它跑"的 loop。缺任何一半都不成立：只有 loop 没护栏是危险的，只有护栏没 loop 是死的。

→ 这一节是 loop engineering 从"概念"走向"敢落地"的关键一跃，也是后面第 10 节风险护栏、落地的前置。

### 6. Memory / State

> **概念速览（复习用，原典支撑）**
>
> **是什么**：六块积木的"第六块"。Addy 原话："the sixth thing, the memory. A markdown file, or a Linear board, anything that lives outside the single conversation and holds what's done and what is next."——==活在**单次会话之外**、记录"已完成什么 + 下一步做什么"的载体==。一个 md 文件、一张 Linear 看板都算。
>
> **为什么必须落磁盘、不能放上下文窗口**：两个原典论点同向——
> - Addy："the model forgets everything between runs so the memory has to be on disk and not in the context. **The agent forgets, the repo doesnt.**"（模型在两次运行之间会忘，所以记忆必须在磁盘上、不在上下文里。==agent 会忘，仓库不会。==）
> - Karpathy《LOOPS.md》§IV 标题即立场："Write to disk, not to context."原话："Context windows lie. They compact, they rot, they hide what you said an hour ago behind a summary you did not write. A file on disk does not lie."（上下文窗口会撒谎——压缩、腐化、把一小时前的话藏在你没写过的摘要里；磁盘文件不会撒谎。）
>
> **核心要求**（Karpathy §IV）：模型应该能"崩了、丢了会话、读三个文件就接上次的班"（pick up where it left off by reading three files）。他给的清单：`feature_list.json`、`progress.md`、`contract.md`，外加一个 append-only 的 `log.md`。"==If you cannot describe your state in three files, your state is too complicated.=="（==三个文件描述不了的状态，就是太复杂了==。）
>
> **在 loop 里的角色**：Addy 晨间分诊例子原话——"The state file is the spine of the whole thing, it remembers what got tried, what passed, what is still open, so tomorrow morning the run picks up where today stopped."（state 文件是整个 loop 的脊柱，记住试过什么、什么过了、什么还开着，明早这次跑能从今天停的地方接上。）==Memory 是 loop 跨轮/跨会话接力的脊柱。==
>
> **与前面章节的咬合**：第 3 节 `/loop` 限制第 4 条——"每轮都是全新 agent 调用，无'接着上次'的天然记忆"。这条限制正是 Memory 积木要补的缺口：`/loop` 每轮从零开始，靠 STATE.md 把上一轮的进度喂给这一轮，loop 才能跨轮接力而非每次重头再来。所以 ==`/loop` 的"无记忆"限制 + Memory 的"磁盘脊柱" = 一个能跨轮延续的 loop==。
>
> **典型文件分工**（综合 Addy/Karpathy）：
> - `STATE.md` / `progress.md`——==当前进度：做过什么、正在做什么、下一步==
> - `contract.md`——==完成标准/验收 checklist==（呼应 Karpathy §III 契约）
> - `log.md`——append-only ==操作日志（试过什么、结果如何），供回溯==
> - `LOOP.md`——==loop 自己的运行规则/目标==（部分用法）
>
> 下面是逐项问答的结论沉淀。

**Q：长 loop 必然跨多个会话，是否用 memory 文件在会话切换时接力？切换前刷新记忆、下一轮读文件接上班？**

A：**核心对，三处校准（均与原典对齐）。**

**对的内核**：memory 文件跨会话接力、长 loop 必然跨多个会话、切换前刷新记忆、下一轮读文件接上班——全对，正是 Karpathy §IV"读三个文件就能接上班"+ Addy"state 是 loop 脊柱"的意思。

**校准 1："记忆混乱" → 更准确是"上下文被压缩/腐化"**

Karpathy §IV 原话不是"记忆变乱"，而是上下文窗口会"compact, rot, hide what you said an hour ago behind a summary you did not write"——==它会悄悄压缩、用你没写过的摘要覆盖你早先说过的话==。问题不是记忆变乱，是上下文被静默改写/丢失。磁盘文件"不撒谎"正因为它不会被这样压缩。

**校准 2：`/new` 只是其中一种手段，且要分两种 loop 场景**

`/new` 是 Claude Code 里强制清上下文的具体命令，但原典的抽象表述是"模型能崩溃/丢会话后读文件接上班"（§IV）——不规定用 `/new`。且要分清两种 loop：

| loop 形态 | 会话切换 | 需要 /new 吗 |
|---|---|---|
| `/loop`（按节奏重跑） | ==每轮本来就是全新 agent 调用==（第 3 节限制 4），天然就在轮间"换会话" | 不需要，内置 |
| 单个长会话（`/goal` 跑几小时，或手动盯着跑） | 上下文持续累积到腐烂 | 可能需要 `/new` 或重启 |

所以 memory 文件两种场景都要用，但"主动开新会话"的需求主要出现在**长单会话**场景；`/loop` 的轮间切换是自动的，memory 文件就是给这种自动切换接力用的。

**校准 3：写盘不应只在 `/new` 前，要贯穿全程**

"执行 /new 之前创建/刷新记忆文件"是最低要求。Karpathy §IV 标准更高："The model should be able to **crash**, lose its session, and pick up where it left off"——==随时崩溃都能接上==。这意味着写盘是连续的（append-only `log.md` 持续追加、`progress.md` 持续更新），不是攒到重启前才写。否则重启前那一刻崩溃，中间进度全丢。"刷新后再 /new"是计划内重启的可接受做法，但抗崩溃的最佳实践是边跑边写。

**一句话**：memory = 让 loop 跨会话/跨轮接力的脊柱；切换可由 `/loop` 自动完成或 `/new` 手动完成；==写盘要连续而非只在切换前，才能抗任意崩溃==。

**Q：以 `/loop` 跑广告报告 MCP 为例，"每轮全新 agent 调用"的"全新"到底指什么？**

A："全新 agent 调用"指**每次触发都是一次从零开始的内循环，没有上一轮对话的上下文残留**。

以 `/loop 6h` 跑广告报告为例的时间线：

```
T0    /loop 6h 启动 → 注册"每 6 小时触发"的调度
T0    第 1 轮触发 → agent A 开始全新内循环
        ↓ agent A 读 STATE.md → 调 MCP 拉广告报告 → 写结果 → 更新 STATE.md → 回纯文本（内循环结束）
        ↓ agent A 的上下文随之消亡（不保留）
... 6 小时 ...
T6h   第 2 轮触发 → agent B 开始全新内循环（不是 agent A 续命）
        ↓ agent B 读 STATE.md（看到 A 写的进度）→ 调 MCP 拉新报告 → 更新 STATE.md → 回纯文本
T12h  第 3 轮触发 → agent C ...
```

**"全新"的三层含义**：

1. **上下文是空的**：agent B 启动时，脑子里没有 agent A 那轮的对话历史。它不知道 A 当时怎么想的、试过什么、报过什么错——除非这些被写进了磁盘文件（STATE.md）。
2. **它不"记得"自己上轮干过**：每轮都是被 `/loop` 当作独立任务重新拉起，就像每次新开一个 Claude Code 窗口。模型在两次调用之间没有跨调用的记忆。
3. **只能靠文件接力**：B 唯一能知道"上次进行到哪"的途径，是 A 在上轮结束时把进度写进 STATE.md，B 开场读这个文件。

**对比：为什么这是 `/loop` 的特性而非缺陷**

| | `/goal` | `/loop` |
|---|---|---|
| 谁在跑 | 同一个 agent 在一个会话里反复跑 | 多个独立 agent 轮次 |
| 上下文 | 连续累积（直到腐烂或达标），==记得==自己几分钟前干过 | 隔断，==不记得==，靠 STATE.md 补 |
| Memory 依赖度 | 还能靠上下文撑一阵 | 没有文件就彻底断片 |

这就是第 3 节限制 4"每轮全新 agent 调用 + 无天然记忆"的本体：==`/loop` 的"循环"不是一只手接力跑，而是 N 个互不相识的人各跑一棒，靠交接棒（STATE.md）传递==。所以 Memory 积木对 `/loop` 比对 `/goal` 更关键。

**回到例子**：每 6 小时那次"MCP 拉报告"的 agent 是全新的——它开场第一件事必须是读 STATE.md，才知道"我这是第几次拉、上次拉到哪天、上次有什么异常没处理完"。若 A 没写盘，B 只能当第一次拉报告，前几轮的进度全丢。

### 7. Worktrees

> **概念速览（复习用，原典支撑）**
>
> **归属厘清**：==worktree 是 git 原生概念，不是 loop engineering 发明的==。loop engineering 只是把它借来用作多 agent 并行的隔离手段。这与第 0 节 harness 认知一致——六块积木大半是借已有能力重新编排，worktree 是典型一例：工具是 git 的，用法是 loop engineering 赋予的。
>
> **git 本身的 worktree**：`git worktree add` 是 git 自带命令，给同一仓库挂多个**工作目录**，每个 checkout 不同分支，但==共享同一份 .git 历史和对象库==。对比 `git clone`（整仓复制、历史也复制），worktree 是"多个工作目录共享一个仓库本体"——省空间、各目录改动物理隔离。
>
> **loop engineering 怎么借用**（Addy 原话）："A git worktree fixes it, its a separate working directory on its own branch sharing the same repo history, so one agent's edits literally can not touch the other one's checkout."——多个 agent 并行改文件会踩脚，worktree 让每个 agent 各占独立工作目录、各跑一个分支，物理上"一个 agent 的编辑碰不到另一个的 checkout"，共用同一仓库历史，最后再合并。
>
> **在六块积木里的位置**：六块积木真实构成（第 2 节）里，worktree 是"一块并行隔离"——非 harness 组件，也非调度，是 loop 层解决多 agent 并发写冲突的专用件。
>
> **一个重要提醒（Addy 原话，常被略过）**：worktree 只解决"机械冲突"（文件不踩脚），但 ==YOU are still the ceiling==——你审查的带宽才决定能并行多少个，工具不是瓶颈，你是。
>
> 下面是逐项问答的结论沉淀。

**Q：loop engineering 里的 worktree 和 git 的 worktree 是完全一致的概念吗？用 worktree 必须先建 .git 再用 git 命令吗？**

A：**不是完全一致——Addy 说的就是 git worktree，但 Claude Code 把它泛化了一层。** 分两层：

**Addy 原典层：就是 git worktree**

Addy 原话直接说 "A **git** worktree fixes it"——指的就是 git 的 `git worktree add`。按原典本意，loop engineering 里的 worktree = git worktree，必须先有 git 仓库（`.git`），再用 git 命令创建。"创建 .git 再用 git worktree"在原典语境下完全对。

**Claude Code 落地层：默认是 git worktree，但有 hook 逃逸口**

这是和 git 原版的关键差异。Claude Code 的 subagent `isolation: "worktree"`：

- **默认实现** = git worktree，==必须有 git 仓库==（工具说明原话："creates a temporary git worktree so the agent works on an isolated copy of the repo"；EnterWorktree 工具也要求 "Must be in a git repository"）。日常用法跟 Addy 一致：先 git init，再 worktree。
- **逃逸口** = 第 4 节学过的 hook 里有 `WorktreeCreate` / `WorktreeRemove` 两个事件，官方文档明确写 "==Replaces default git behavior=="。可挂 hook 用自定义逻辑替换掉 git worktree 的默认实现——理论上做出一个非 git 的隔离工作目录。

严格对照表：

| 层 | worktree 是什么 | 必须用 git 吗 |
|---|---|---|
| Addy 原典 | 就是 git worktree | 是 |
| Claude Code 默认 | git worktree | 是（需 git 仓库） |
| Claude Code + 自定义 hook | "隔离工作副本"的抽象，git worktree 是默认实现 | 否（hook 可替换成非 git 实现） |

**结论**：99% 场景下 loop engineering 说的 worktree 就是 git worktree，理解对——要 git 仓库 + git 命令。但概念上 Claude Code 比 Addy 多走一步：把 worktree 抽象成"隔离工作副本"这个能力位，git worktree 是其默认实现，可通过 `WorktreeCreate`/`WorktreeRemove` hook 替换。==这也是第 4 节 hook 的延伸——hook 不只拦危险操作，还能替换 harness 的内部行为（worktree 创建机制本身）==。

**Q：因为 Claude Code 有 WorktreeCreate/WorktreeRemove hook，即使不给项目建 .git，也能有 worktree 管理手段吗？**

A：**方向对，两处校准。**

**校准 1：明确支持无 git 的是 EnterWorktree 工具路径**

EnterWorktree 工具 Requirements 原话："Must be in a git repository, **OR** have WorktreeCreate/WorktreeRemove hooks configured"——OR 关系：==配了这两个 hook，就不需要 git 仓库==。此路径上理解成立。

但 subagent `isolation: "worktree"` 路径，工具描述只说 "creates a temporary **git** worktree"，默认仍假设 git 仓库。WorktreeCreate hook 理论上接管所有 worktree 创建行为，但此路径是否完全等价支持无 git，官方文档无明文，需实测确认。

**校准 2：hook 不是开关，是"你自己写隔离逻辑"**

最易误解的点。WorktreeCreate hook "Replaces default git behavior"——==默认的 git worktree 创建逻辑被你的 hook 脚本替换掉==。你得自己写脚本实现：建独立目录、隔离、清理。git worktree 白送的"共享历史 + 分支隔离 + 合并"，替换后没了，得自己搞或放弃。

所以"能不用 git 就有 worktree 管理"——能，但不是省事，而是把 git 免费提供的隔离能力换成自己手写。除非有特殊理由（项目本就不是 git 仓库、或要隔离到非 git 体系），否则直接 `git init` 用默认 git worktree 远比自写 hook 划算。

**一句话**：配 WorktreeCreate/WorktreeRemove hook 可在无 git 时接管 worktree（EnterWorktree 路径明确支持），但 hook 是==替换而非赠送==——你得自己实现隔离逻辑，git worktree 默认白送的能力要自己补。日常还是 `git init` 最省。

### 8. Sub-agents

> **背景：sub-agent 背景知识（综合公开资料校准）**

**1. 添加子代理的两种方式**

- **自动创建**：Claude Code 执行任务时，会根据任务需要自行创建 general-purpose 子代理。
- **自定义创建**：在全局或项目 `agents/` 文件夹中创建子代理定义文件。文件格式是 ==markdown 文件 + YAML frontmatter==（顶部 `---` 块放元数据，下方 markdown body 放指令），与 SKILL.md 同构。frontmatter 主要字段：`name`（名称）、`description`（触发描述）、`tools`（可用工具）、`model`（指定模型）。

> 已确认：① `color` 是有效 frontmatter 字段——填 `color: green` 后，会话里该子代理名（圆点后的 name）显示绿色，控制子代理在会话中的显示颜色。② `model` 可填模型名（如 `YOUR_MODEL`），子代理实际跑该模型（详见下文路径 A/B）。

**2. 用 skill 加速创建**

可创建一个全局 `agent_create` skill，把常用子代理风格写进 skill，随时通过该 skill 一键生成自定义子代理——把"重复配子代理"这件事本身也 skill 化。

**3. 子代理的功能**

- **并行执行多任务**：多个子代理同时干不同活。
- **保持各自会话空间 clean**：主代理/子代理上下文隔离，互不污染。

> 下面是逐项问答的结论沉淀。

**Q：subagent 能否使用非 Claude 模型？能否给不同子代理指定不同模型？**

**背景前提**：Claude Code 官方文档的 subagent `model` 字段支持 Claude alias（sonnet/opus/haiku）/ Claude 完整 ID / `inherit`。若你的 provider 暴露非 Claude 模型（如经兼容层接入），能否用于子代理、能否逐个差异化，取决于 provider 接入方式——需实测确认。模型解析顺序：`CLAUDE_CODE_SUBAGENT_MODEL` 全局 env > 调用参数 > frontmatter model > 主对话模型。

**认知退路**（Karpathy §II）：maker/checker 核心价值不只在"不同模型"，更在**不同上下文 + 不同指令**（三角色本就是"三上下文"非强制"三模型"）。==同模型 + 不同 subagent 定义仍能实现大部分 maker/checker 价值==——这是模型层万一不通时的兜底认知。

**两条路径均可行**：子代理能用与主对话不同的模型，且能逐个子代理指定不同模型。

---

**子代理使用非 Claude 模型**

**路径 A：`CLAUDE_CODE_SUBAGENT_MODEL` 全局子代理换模型 —— ✅ 可行**

背景：provider 官方文档的标准配置块含 `CLAUDE_CODE_SUBAGENT_MODEL` 字段，且注明"建议与主模型保持一致"——反证可不一致。这是官方支持路径，非 hack。

操作：在用户级 `settings.json` 加 env 块：
```json
"env": {
  "CLAUDE_CODE_SUBAGENT_MODEL": "YOUR_MODEL"
}
```
重启会话后派 general-purpose 子代理自报模型。

结果对照：

| | 主对话 | 子代理 |
|---|---|---|
| 配置前（基线） | 主模型 | 主模型（继承） |
| 配置后（路径 A） | 主模型 | ==指定模型== |

结论：==Claude Code 子代理能用与主对话不同的模型，`CLAUDE_CODE_SUBAGENT_MODEL` 路径可行==。loop engineering 模型分离理念可落地。

**路径 A 的局限**：`CLAUDE_CODE_SUBAGENT_MODEL` 是==全局字段==——所有子代理统一用一个模型，无法 maker 用 A、checker 用 B 逐个差异化。逐个差异化需靠 subagent frontmatter 的 `model` 字段（Test B）。

**模型清单**：具体可用模型由你的 provider 决定，见 `examples/providers/`。按角色选：代码向模型给 generator、推理向模型给 planner、轻量/异构模型给 evaluator。

**路径 B：frontmatter `model` 逐个子代理指定 —— ✅ 可行**

操作：① 清空全局 `CLAUDE_CODE_SUBAGENT_MODEL`（避免压住 frontmatter）② 在项目 `.claude/agents/model-probe.md` 建 custom subagent，frontmatter `model: YOUR_MODEL`，body 指令"只自报模型" ③ 重启加载该子代理 ④ 派它自报模型。

结果：子代理回报 frontmatter 指定的模型，与主对话不同。

结论：==frontmatter `model` 字段生效，能逐个子代理指定不同模型==。这意味着 loop engineering 模型分离理念能**完整落地**：maker 与 checker 子代理 frontmatter 填不同模型，两个子代理真正用不同模型——达成 Karpathy §II"不同模型降自评偏差"的理想形态。

**关键注意（模型解析顺序）**：Claude Code 官方顺序为 `CLAUDE_CODE_SUBAGENT_MODEL`(全局 env) > 调用参数 > frontmatter model > 主对话模型。==全局 env 优先级最高，会压住 frontmatter==。所以要逐个差异化，**不能设全局 `CLAUDE_CODE_SUBAGENT_MODEL`**，让每个子代理靠自己的 frontmatter 指定。路径 B 正是清空全局 env 才成功的。

**两测试合体的能力矩阵**：

| 机制 | 粒度 | 适用 |
|---|---|---|
| `CLAUDE_CODE_SUBAGENT_MODEL` 全局 env | 所有子代理统一一个模型 | 简单场景：所有子代理都换同一个模型 |
| frontmatter `model` 逐个指定 | 每个子代理可不同 | ==loop engineering 标准场景：maker/checker 用不同模型== |

→ sub-agent 积木==完全可用且支持模型差异化==，loop engineering 落地无模型层障碍。具体 provider 模型见 `examples/providers/`。

**Q：契约谈判是什么？要写"checker 怎么 challenge maker"的文档吗？**

A：部分对，但 Karpathy 的契约比"checker 怎么 challenge maker"更具体——==契约是"完成标准 checklist"，不是"挑战方式"==。（核实 Karpathy《LOOPS.md》§III "NEGOTIATE THE CONTRACT FIRST"原话）

**Karpathy §III 原话拆解**：

> "Before the generator writes a single line, it proposes what done looks like and the evaluator pushes back. The two argue via markdown files on disk until they agree on a checklist of testable assertions."

关键点：

1. **契约是"done 长什么样"的清单**，不是"怎么挑战"。是==一组可测试断言==（如"测试全过""lint 干净""某 API 返回 X"），不是"checker 要质疑 maker 哪些方面"。
2. **生成方（generator）先提**："我提议 done 是这些"。
3. **评估方（evaluator）反推**：对每条挑刺、加严、补漏。
4. **两边通过磁盘上的 markdown 文件争吵**（呼应第 6 节 Memory——契约也落盘，文件名 Karpathy 直接叫 `contract.md`），==吵到达成一致==才动手写码。
5. **规模参考**：小应用约 27 条断言合理；10 条太少 evaluator 会橡皮图章（rubber-stamp，没真审）。
6. **契约是被评分的东西**："The original spec from the planner is the boundary, but ==the contract is what gets graded=="——planner 给的原始规格是边界，但真正拿来打分的是契约。Karpathy 称这是"the single change that moved my own runs from broken demos to working products"（让他的 loop 从破演示变成可用产品的唯一改动）。

**所以设计 loop 时要写的文档是 `contract.md`**（或叫完成标准），内容是"做完了 = 满足这 N 条可测试断言"。它和 `/goal` 的可验证停止条件、第 6 节的 STATE 文件是同一脉络：

| 文档 | 记什么 | 谁用 |
|---|---|---|
| `contract.md` | done 的可测试断言 checklist | evaluator 拿来打分；goal 拿来判停 |
| `STATE.md`/`progress.md` | 进度（做到哪了） | 下一轮接力 |
| `log.md` | 试过什么、结果 | 回溯调试 |

**"checker 怎么 challenge maker"在 Karpathy 这里不是单独文档**——challenge 的方式就是"拿 contract.md 逐条验，不满足就打回"。挑战的"标准"是契约，挑战的"动作"是 evaluator 的工作流（读产出→对契约逐条判→不通过就回退）。

**Q：三角色分工（planner/generator/evaluator）和 maker/checker 是一回事吗？只是多分一个 planner？**

A：==逻辑一致但实质差异在"三上下文隔离"和"evaluator 预设敌意"==，不只是多一个角色。（核实 Karpathy《LOOPS.md》§II "SEPARATE THE ROLES"原话）

**Karpathy §II 原话**：

> "Three roles, three context windows, three system prompts. A planner that turns a vague human sentence into a sprint spec and never touches code. A generator that writes everything and is forbidden from grading its own work. An evaluator that reads diffs, launches playwright, plays the app, and is told from the first message that the code is broken and its job is to prove it."

**三角色各自的硬约束**（分工的实质，不只是"多分一个"）：

| 角色 | 干什么 | 硬约束（不许干什么） |
|---|---|---|
| **planner** | 把人的一句模糊话转成 sprint 规格 | ==永不碰代码== |
| **generator** | 写所有代码 | ==禁止给自己的作品打分== |
| **evaluator** | 读 diff、跑 playwright、玩 app | ==从第一条消息就被告诉"代码是坏的，你的任务是证明它坏"== |

**比 maker/checker 多出来的关键认知**：

1. **三上下文，不是三模型**：Karpathy 开篇强调 "Three roles, three ==context windows==, three system prompts"——核心是三个角色各用独立上下文窗口 + 不同系统提示，==上下文隔离比模型差异更重要==。呼应 Test B：靠 frontmatter 不同 prompt/tools 就能实现角色分离，模型是否不同次要。
2. **evaluator 的"预设敌意"**：从第一句就被设定"代码是坏的，任务是证明它坏"——不是中性检查，是==对抗式验证==，比 maker/checker 的"checker 审一下"更激进，逼 evaluator 主动找茬。
3. **planner 是"翻译层"**：把人的一句话模糊需求翻译成 generator 能执行的 sprint 规格。不写码、不验码，只做"需求→规格"转换。解决"maker 拿到的需求本身就不清楚"的问题——这是 maker/checker 二分里没有的环节。
4. **混角色是最常见失败**："Mixing the roles is the most common failure I see; the model becomes sycophantic the moment it grades itself, and the loop quietly converges on slop."（混角色是最常见失败；模型一给自己打分就谄媚，loop 悄悄收敛成垃圾。）

**与 maker/checker 的关系**：maker/checker 是二分（干+验），Karpathy 三角色在前面加 planner（翻译需求）。但==实质差异在"三上下文隔离"和"evaluator 预设敌意"==，不只是多一个角色。

**对应 Claude Code 落地**：三个 subagent 各写一份 frontmatter——planner（`tools: Read`，禁 Edit/Write）、generator（`tools: 全部`）、evaluator（系统提示里写"代码是坏的，证明它坏"）。模型可全 inherit 主模型（靠上下文隔离），也可给 evaluator 单独配异构模型做差异化（见上文 frontmatter 逐个指定）。

---

**示例：三角色子代理配置**

**子代理创建元 skill**：可把 frontmatter 模板、模型清单、三角色决策树、三条硬约束、契约配套提醒、创建流程固化进一个元 skill，业务 prompt 留空待具体任务补——元工具不包办业务。基于本节结论（model 可填模型名、color 只认 8 色名）。

**三角色子代理**（项目级 `.claude/agents/`）：

| 角色 | 文件 | model | color | tools | 硬约束 |
|---|---|---|---|---|---|
| planner | `planner.md` | 最强推理模型 | blue | Read,Grep,Glob（禁码） | 只做需求→规格翻译，永不碰码 |
| generator | `generator.md` | 最强代码模型 | green | 全部（含 Bash/Write/Edit） | 写码执行，禁自评 |
| evaluator | `evaluator.md` | 异构模型 | red | Read,Grep,Glob,Bash（只读+验证） | 预设敌意，对 contract.md 逐条验 |

**模型选择理由**（角色适配）：
- generator 用最强代码模型（执行层放最强代码能力）
- planner 用最强推理模型（搭架构、拆任务）
- evaluator 用与 generator 不同家族的模型 → 异构视角降自评共谋
- 三角色三模型，满足 maker/checker 分离原则

→ ==三角色差异化全部落地==，loop engineering 模型分离理念完整实现。子代理创建元 skill 可用、三角色子代理就位，为后续 loop 落地备好 sub-agent 积木。

---

**示例：三角色 loop 端到端运行（榜单数据 → HTML 邮件）**

> 用三角色子代理跑一次完整 loop，验证 Karpathy §II（角色分离）+ §III（契约谈判）+ §IX（瓶颈移动）。任务示例：抓当天某榜单数据，做成 HTML 页面作为邮件正文发出。产物全落盘。

**流程与产出**（orchestrator 居中 relay，因本轮 planner/evaluator 未配 Write）：

| 阶段 | 角色(模型) | 产出 |
|---|---|---|
| 规划 | planner | sprint_spec.md（步骤+验收+兜底） |
| 契约 v0.1 | generator | 断言草案 |
| 反推 | evaluator | 问题清单：太松/歧义/后门/缺失/可验证性/橡皮图章 |
| 契约 v0.2 | generator | 27条收敛 + 守住 loop 自主性（拒绝人工确认作硬断言） |
| 终审 v0.2 | evaluator | 通过 |
| 执行 | generator | 抓取→解析→HTML→发邮件 |
| 终验 v0.2 | evaluator | 多数通过，少数字面与客观不符 |
| 契约 v0.3 | orchestrator | 放宽3条匹配现实 |
| 终审 v0.3 | evaluator | ==全部通过== |

**实测验证的 loop engineering 原则**：
1. **三角色差异化落地**——planner/generator/evaluator 各跑指定模型，异构视角降共谋
2. **契约谈判收敛**——两轮争吵（v0.1→v0.2→v0.3），契约从"太松有后门"到"可客观验证"
3. **evaluator 预设敌意有效**——真找出后门（log自报可造假）、歧义（A3.2未闭合标签/A3.9排序字段）、橡皮图章（A0.3未询问收件人）
4. **§IX 瓶颈移动现场**——契约写时对数据源/工具能力的假设，执行才知达不到，==改契约匹配现实，非逼 generator 造假==
5. **loop 自主性守住**——拒绝人工确认作硬断言，全流程无人介入，邮件自主送达
6. **Memory 落盘**——progress.md/log.md + 中间产物全落盘，可跨轮接力

**暴露的两个设计问题（已修）**：
- ~~planner/evaluator 未配 Write，无法像 Karpathy 说的"通过磁盘 markdown 文件争吵"——本轮靠 orchestrator 居中 relay~~ → ==已修==：planner/evaluator 的 tools 加 Write（body 限定仅落盘 markdown，禁写代码/禁改产出），元 skill 决策树同步更新。下次 loop 争吵直接落盘，不再靠 orchestrator relay。
- ~~契约写时对数据源/工具能力的假设（≥20、250回执）应在契约阶段就核实，而非执行后才改~~ → ==已修==：planner body 加"契约可行性核查（提断言时做）"、evaluator body 加"可行性核查（反推时做）"、元 skill 加独立小节"契约可行性核查"含 trending 案例教训。把 §IX 瓶颈移动前置到契约阶段。

**关键认知**：契约是工具不是教条——断言字面与客观成功冲突时（如 A1.2 "sign in" 是页面 chrome 假阳性、A4.3 skill 不输出 250 但 exit 0），应判==意图达成==而非字面苛求；但 generator 用合理理由拒绝的（loop 自主性），evaluator 不该为找茬而找茬。契约 v0.1→v0.3 的演化本身就是 loop 的"活系统"特性（呼应第 0 节"harness 是随失败历史收紧的活系统"）。

### 9. Connectors

> **概念速览（复习用，原典支撑）**
>
> **是什么**：六块积木之一。==Connectors 本质是基于 MCP 的工具接入层==——把你的 API（广告 API、数据库、Slack、issue tracker 等）包装成 agent 能调用的工具。Addy 原话："Connectors, which are built on MCP, let the agent read your issue tracker, query a database, hit a staging api, drop a message in Slack."
>
> **为什么是 loop"能动真格"的关键**：Addy 原话对照——"an agent that says 'here is the fix'（只告诉你怎么修）vs a loop that opens the PR, links the ticket and pings the channel（自己开 PR、关联工单、CI 绿了 ping 频道）"。==没有 connector，loop 只能看文件系统、只能"说"；有 connector，loop 才能"做"==。所以 Addy 名句："A loop that can only see the filesystem is a tiny loop."（只能看文件系统的 loop 是小 loop。）
>
> **与第 2 节的呼应**：六块积木真实构成里，connectors 属"四块 harness 组件"之一——它本身就是 harness（让单个 agent 能动真工具的非模型部分）。loop engineering 复用它，让 loop 能在真实环境里行动。
>
> **落地示例**：项目里的 ApiClient（调外部 API 拉报告、调价）就是 connector——它让 loop 能真的进账户操作，而非只告诉你“建议调价”。能让 loop 自己执行调价+写日志+推送告警才是完整价值（呼应第 2 节概念速览）。
>
> 下面是逐项问答的结论沉淀。

### 10. 持久化调度（Automations 积木）

> **概念速览（复习用，原典支撑）**
>
> **持久化调度 = 让 loop 在关掉电脑/结束会话后仍能长时间持续跑**。属 Automations 积木的持久化子集（Addy 原话："push the whole thing to GitHub Actions if you want it to keep running after you close the laptop"）。
>
> 与会话内调度（`/loop`、`/goal`，第 3 节）相对：会话内只活在当前 Claude Code 进程，关掉就没了；持久化靠外部调度器反复触发，不依赖人盯着的终端。
>
> **持久化调度工具清单**：

| 类别             | 工具/方式                                            | 谁触发               | 适合场景                | 关键点                                 |
| -------------- | ------------------------------------------------ | ----------------- | ------------------- | ----------------------------------- |
| **会话内（非持久化）**  | `/loop`                                          | Claude Code 进程内调度 | 人在电脑前、临时周期任务        | 关掉 Claude Code 就停；第 3 节已讲其 4 限制     |
|                | `/goal`                                          | 同上，跑到条件成立         | 一次性长任务、人在场          | 同上非持久化                              |
| **持久化-云托管**    | **GitHub Actions**                               | GitHub 服务器定时触发    | 代码仓库、跨机器、免费额度内      | Addy 原典首推；cron 语法；仓库即配置             |
|                | **Codex Automations tab**                        | Codex 平台托管        | Codex 用户、GUI 配置     | Addy 提到；pick project+prompt+cadence |
|                | **云端 cron 服务**（如 cron-job.org / AWS EventBridge） | 云服务商              | 无仓库、需 HTTP 触发       | 比 GitHub Actions 更纯粹的定时             |
|                | **Claude Code Routines**（Cloud tasks）            | Anthropic 云基础设施   | Claude Code 用户、无人值守 | 官方持久化首选；最小间隔 1 小时                   |
| **持久化-==本机==** | **Windows 任务计划程序**                               | Windows 系统调度      | Windows 服务器、本地常驻    | Windows 适用；可设触发器+运行 claude -p            |
|                | **Linux cron / systemd timer**                   | 系统调度              | Linux 服务器常驻         | 同上，Linux 版                          |
|                | **launchd**（macOS）                               | macOS 系统调度        | Mac 常驻              | 同上，macOS 版                          |
|                | **PM2 / nohup 守护进程**                             | 进程守护              | 需常驻进程而非定时触发         | 守护而非调度，边界场景                         |

> Addy 原典明确列的：`/loop`、cron、GitHub Actions、Codex Automations tab。其余为补充。核心二分：==会话内（/loop、/goal，靠 Claude Code 进程）vs 持久化（靠外部调度器反复触发，不依赖人盯着的终端）==。
>
> **Windows 落地候选**：Windows 任务计划程序（本机常驻）或 GitHub Actions（云托管）——前者适合本地服务器跑 loop，后者适合有仓库的场景。
>
> 下面是逐项问答的结论沉淀。

**Q：GitHub Actions 是什么？收费吗？是不是上传到 github 仓库的项目能在 github 服务器上跑？**

A：GitHub Actions = GitHub 提供的 CI/CD 服务，在你 push 代码到仓库后、或按定时计划，==GitHub 的服务器自动跑你定义的脚本/任务==。

**核心机制**：
- 在仓库里放 `.github/workflows/xxx.yml`，声明"什么时候跑、跑什么"
- 触发方式：push/PR 触发、==定时触发（cron 语法，这是它当持久化调度的原理）==、手动触发等
- GitHub 给一台临时虚拟机（runner），跑完销毁

**收费**：
- 公开仓库完全免费（不限时长）
- 私有仓库有免费额度：免费账号每月 2000 分钟、Pro 账号 3000 分钟，超出才付费
- ==个人用、定时跑 loop 场景，免费额度基本够用，不用充值==

**精确理解**（校准"在 github 服务器上运行项目"）：
不是"把整个项目常驻在 github 跑"，而是"每次触发时 github 临时起一台虚拟机，跑你指定的步骤，跑完就没了"。==按触发跑一次，非常驻服务==。适合定时 loop（如每 6 小时拉一次数据），不适合常驻监听。

**对 loop engineering 的意义**（Addy 原话）："push the whole thing to GitHub Actions if you want it to keep running after you close the laptop"——==关掉电脑，github 服务器仍按 cron 定时触发 loop==。配置在仓库里（仓库即配置），跨机器、不用自己维护服务器。

**对本场景的提醒**：
- GitHub Actions 跑在 github 云虚拟机里，==外部 API 凭据、智能体凭据不能明文放仓库==（公开会泄露，私有也不安全）——要用 GitHub Secrets（加密环境变量）注入
- 云虚拟机访问某些服务可能有网络问题，落地前需测连通性

**Q：Codex Automations tab 入口在哪？收费吗？**

A：已查 Codex 官方文档。

**是什么**：Codex 的后台定期执行任务机制——设定时间表+任务描述，Codex 按计划在后台跑，结果落进收件箱（Triage）；无发现则自动归档。

**入口**：==Codex 应用（App，网页/桌面端）侧边栏的 "Automations" 面板==，点 "Create Automation" 创建。也可从普通 Codex 会话里直接描述任务+时间表创建/更新。注意：这是 Codex **App** 功能，非 CLI 命令——需用 Codex App（非 Codex CLI）。

**如何配置**：
1. 侧边栏 Automations 面板 → Create Automation
2. 填任务描述 + 时间表（支持 cron 语法自定义）
3. 选执行环境：本地项目 或 新 worktree（隔离改动）
4. 配沙盒设置：只读 / 工作区写入 / 完全访问（无人值守默认用你的默认沙盒设置）
5. 安排前先手动测试任务提示

**两种 automation 类型**：
- **常规自动化**：独立新运行，结果落 Triage 收件箱
- **会话自动化**：心跳式定期唤醒，==附加到当前会话==，保留上下文（类似 Claude Code 的 /loop 但持久化、且接回同一会话）——适合长运行命令状态检查、轮询外部源

**收费**：==文档未明说消耗什么额度==。Codex App 用 ChatGPT 账号体系，Automations 大概率消耗 ChatGPT Plus/Pro 订阅额度（非单独计费），但具体 Plus 是否含 Automations、每月限额多少，文档无明文——需登录 Codex App 后在 Automations 面板或账户设置确认。

**注意**：若你用 Codex CLI，Automations 是 **App 功能非 CLI**——要用需上 Codex App（chatgpt.com/codex 或桌面端）。CLI 本身无此命令（呼应第 3 节"Codex 无 /loop，靠外部调度"——Automations tab 就是 Codex 的持久化调度 GUI）。

> 需登录 Codex App 确认额度含不含 Automations、月限额多少。

**Q：durable cron（Claude Code 的 /loop durable 选项）是什么？**

A：==纠正：官方文档里没有 "durable cron"，也没有 /loop 的 "durable 选项"——这是此前表格的臆造错误，已改==。核实 Claude Code 官方 `scheduled-tasks.md` 文档：Claude Code 有**三种**调度方式，非"durable 选项"：

| 调度方式 | 跑在哪 | 需开机 | 需开 会话 | 持久 | 最小间隔 |
|---|---|---|---|---|---|
| **Cloud（Routines）** | Anthropic 云 | 否 | 否 | ==是== | 1 小时 |
| **Desktop 计划任务** | 你的机器 | 是 | 否 | 是 | 1 分钟 |
| **/loop** | 你的机器 | 是 | ==是== | 仅 --resume 恢复未过期任务 | 1 分钟 |

**/loop 的真实持久性**（校准第 3 节"会话内生命周期"限制）：
- /loop ==是会话级==，只活在当前对话，开新对话就停
- 但 `claude --resume`/`--continue` 恢复会话时，==未过期任务会回来==（7 天内重复任务 + 未到点一次性任务）
- 重复任务==7 天自动过期==（最后跑一次自删）
- 单会话最多 50 个计划任务；用 CronCreate/CronList/CronDelete 工具管理

**三种方式怎么选**（官方原话）：
- Cloud tasks（Routines）：需可靠运行、不依赖你机器的活
- Desktop tasks：需访问本地文件和工具
- /loop：会话内快速轮询

**关键结论**：==Claude Code 里没有 "durable cron"==——真正持久化是 **Routines**（Anthropic 云）和 **Desktop 计划任务**（本机），/loop 只是会话内、靠 --resume 续命 7 天。想要无人值守持久跑 loop：Routines（云）或 Desktop 计划任务（本机）或 GitHub Actions。

> Routines 是 Anthropic 管理的云基础设施上的调度，官方"持久调度"首选；Desktop 计划任务在你机器本地跑、需开机但不用开会话。两者都比 /loop 更适合长期 loop。

**Q：若有云服务器，怎么和 loop engineering 结合？需要在云端部署 Claude Code/Codex 吗？**

A：==是，云端部署智能体 + 系统定时驱动，是云服务器方案的典型用法==，且比 GitHub Actions 更适合某些场景。

**结合架构**（Linux 云服务器）：

```
云服务器（Linux，7×24 开机）
  ├── 装好 Claude Code CLI（配好智能体凭据 env）
  ├── 项目代码（含 .claude/ 配置：agents/skills/hooks）
  └── cron / systemd timer 定时触发
        └── 每到点执行：claude -p “跑巡检 loop” --allowedTools ...
            （non-interactive 模式，无人值守）
```

**三层机制**：
- **云服务器 = 持久化机器层**（7×24 在线，解决"关电脑就停"）
- **cron/systemd = 持久化调度层**（定时触发，不依赖人会话）
- **Claude Code `-p` 模式 = 无人值守执行**（non-interactive，跑完即退，不等用户输入）

三者合起来 = ==真正的持久化 loop==：到点 → cron 触发 → claude -p 跑一轮内循环 → 落盘 STATE/log → 退出 → 下周期再来。

**是否需云端部署 Claude Code/Codex**：==是==，loop 跑的就是智能体，云服务器上得有智能体可跑。本地装了 Claude Code，云服务器同样装一份 + 配智能体凭据 + 拉项目代码即可。

**几条持久化路径对比**：

| 路径 | 适合度 | 理由 |
|---|---|---|
| **云服务器 + cron + claude -p** | ==最适合== | 已有云服务器、熟悉部署；7×24 在线；凭据自己管不上传 github；网络可控 |
| GitHub Actions | 次选 | 需推 github + 凭据用 Secrets；云虚拟机访问国内服务可能有网络问题 |
| Claude Code Routines | 待观察 | Anthropic 云托管，凭据/网络/收费待确认 |
| 本机 Windows 任务计划 | 不适合长期 | 电脑不会 7×24 开机 |

**loop 推荐落地形态**：
- Linux 云服务器装 Claude Code + 项目代码 + 智能体凭据（env）
- systemd timer 或 cron 定时跑 `claude -p “巡检并写报告”`
- 配 hook 拦危险操作（调价/扣款）+ STATE.md 跨轮接力
- 报告邮件用 send_email 流程发送

**关键提醒**：
- 云服务器上 Claude Code 也要配 `.claude/`（agents/skills/hooks 同步上去），==harness 配置随项目走==
- 凭据（智能体 token、外部 API、邮箱授权码）放服务器环境变量或 .env，==不入 git==
- `claude -p` 是 non-interactive，要配好 `--allowedTools` 或权限模式避免卡在权限提示（无人值守没法点确认）

### 11. 风险与护栏（全程意识，非独立模块）

> **概念速览（复习用，原典支撑）**
>
> **"风险与护栏"不是六积木之一**，也非原典单列的框架。原典只散见"风险意识"：
> - **Addy 文末警告**："If I relied entirely on automated loops to fix it my product's quality would suffer. I'd likely end up stuck in a downward spiral, continuously digging myself into a deeper hole."（完全依赖自动化 loop 会陷 downward spiral）
> - **Addy 关键判断**："Two people can build the exact same loop and get completely opposite results. One uses it to move faster on work they understand deeply. The other uses it to avoid understanding the work at all. ==The loop doesn't know the difference. You do.=="（同样的 loop 两人用出相反结果——一个加速理解、一个逃避理解，loop 分不清，你分得清）
> - **Karpathy §IX 瓶颈永远在移动**：coding→planning→verification→taste——验证永远是瓶颈，最终落到人的品味判断。
>
> **护栏的实操 = harness 层约束（非新东西）**：风险管控不是独立模块，而是==散在前各节的 harness 约束==：
> - **subagent body 指令禁令**：planner"永不碰代码"、generator"禁自评"、evaluator"禁改产出/禁写代码"——在角色定义文件里明确"you can not do XXXX"
> - **skill 定义文件**：同样可写禁令（如子代理创建元 skill 限定 Write 用途）
> - **hook（第 4 节）**：PreToolUse 拦危险操作——==指令禁令模型可能忘，hook 脚本不会==，是更强的确定性护栏
> - **契约（第 8 节）**：evaluator 拿 contract.md 逐条验，本身也是一种验证护栏
>
> **核心认知**：风险意识全程相伴，不单独成节。落地时把禁令写进 subagent/skill 定义、用 hook 做确定性拦截、用契约做验证门禁——这些 harness 约束就是护栏的全部实操。==没有"护栏积木"，只有"在每个积木里写约束"==。
>
> 动钱场景特别提醒：动钱操作（调价/扣款）绝不能裸跑——靠 PreToolUse hook 拦 + subagent 指令禁令 + 必要时人工门禁（见 SKILL.md 落地流程）。
