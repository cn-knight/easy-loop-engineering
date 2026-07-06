# easy-loop-engineering 共享化契约（done 定义）

本文件是优化循环的契约：评估器每轮对照下列断言，全部满足 = 共享化完成（loop 停止）。

> 注：为避免泄漏原作者个人 token，本契约用抽象描述指代被清除项；评估时由执行者代入具体清单。

## 验收断言

1. **无个人路径/标识**：grep 不到原作者的用户名 / 个人路径段 / 项目编号 / 个人家目录路径
2. **无个人 skill 依赖**：不依赖任何个人私有 skill（自包含或泛化）
3. **去个人昵称**：原个人昵称 → `原典 / Canonical Texts` 全量替换（含文件名）
4. **模型去硬编码**：不把任何具体模型名当默认；改为异构分离原则 + provider 作 optional example
5. **跨智能体**：CC 专属概念（条件/定时调度命令、事件 hook、subagent env、SKILL.md frontmatter）均标注为 CC 并给 Codex 等价物或抽象表述
6. **学习笔记公文化**：`1_概念学习.md` 去第一人称/个人语气，改为面向读者的知识问答
7. **GitHub 就绪**：README（双语）+ MIT LICENSE + 安装/使用说明
8. **provider 专属文档移出核心**：provider 文档移至 `examples/providers/`
9. **自包含**：clone 即可用，不依赖私人环境

## 断言状态（每轮由 evaluator 更新）

| # | 断言 | Round 3 后 | 备注 |
|---|---|---|---|
| 1 | 无个人路径/标识 | ✅ | 全 shipped 文件清零；examples/providers/volcengine 的命中为有意保留的 provider 示例 |
| 2 | 无个人 skill 依赖 | ✅ | SKILL.md 内联 frontmatter 模板，不依赖外部私有 skill |
| 3 | 个人昵称→原典 | ✅ | fathers 全清 + 文件改名 0_canonical-texts.md |
| 4 | 模型去硬编码 | ✅ | 改为异构分离原则；火山仅作 optional example（examples/providers/） |
| 5 | 跨智能体标注 | ✅ | SKILL.md+AGENTS.md 映射表 + 1_概念学习.md header 跨智能体注 |
| 6 | 学习笔记公文化 | ✅ | 1_概念学习.md 去第一人称/个人路径/个人邮箱/个人场景，改为面向读者的 Q&A |
| 7 | GitHub 就绪 | ✅ | README 双语 + MIT LICENSE（cn-knight）+ 安装/使用 |
| 8 | provider 文档移出 | ✅ | 已移至 examples/providers/volcengine/ |
| 9 | 自包含 | ✅ | clone 即用，无外部私有依赖 |

**状态**：全部 ✅——发布就绪，待用户终检。
