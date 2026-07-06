---
title: "Claude Code--火山方舟-火山引擎"
source: "https://www.volcengine.com/docs/82379/1928262?lang=zh"
author:
published:
created: 2026-07-02
description: "火山引擎官方文档中心，产品文档、快速入门、用户指南等内容，你关心的都在这里，包含火山引擎主要产品的使用手册、API或SDK手册、常见问题等必备资料，我们会不断优化，为用户带来更好的使用体验"
tags:
  - "clippings"
---
接入 AI 工具

Claude Code

本文介绍如何在 Claude Code 中使用 方舟 Coding plan 。

说明

需要先访问 [方舟 Coding Plan活动](https://www.volcengine.com/activity/codingplan) ，按需订阅套餐后再配置使用编程工具。

核心配置

接入工具的核心参数如下：

模型配置

支持以下两种方式配置模型：

Base URL

不同的工具配置的 Base URL 根据兼容的协议会有不同：

- 兼容 Anthropic 接口协议工具：https://ark.cn-beijing.volces.com/api/coding

- 兼容 OpenAI 接口协议工具：https://ark.cn-beijing.volces.com/api/coding/v3

注意

请勿使用 https://ark.cn-beijing.volces.com/api/v3 ：该 Base URL 不会消耗您的 Coding Plan 额度，而是会产生额外费用。

API Key

[获取 API Key](https://console.volcengine.com/ark/region:ark+cn-beijing/apikey)

一、安装 Claude Code

前提条件：

- 安装 [Node.js 18 或更新版本环境](https://nodejs.org/en/download/) 。

- Windows 用户需安装 [Git for Windows](https://git-scm.com/download/win) 。

在命令行界面，执行以下命令安装 Claude Code。

复制

npm install -g @anthropic-ai/claude-code

安装结束后，执行以下命令查看安装结果，若显示版本号则安装成功。

复制

claude --version

二、配置 Claude Code

方式 1：自动化助手（推荐）

ArkCLI Helper 是一个编码工具助手，支持快速配置选择的工具接入 Coding Plan。安装并运行该助手，根据界面提示操作可自动完成工具配置，降低手动配置的时间成本和出错风险。

注意

- ArkCLI Helper 是基于 Ark Helper 升级演进的工具配置服务，相关能力将持续在 ArkCLI Helper 上迭代更新。建议优先使用 ArkCLI Helper 完成工具配置。

- ArkCLI Helper 支持 macOS、Linux、Windows 系统。

1. 安装 Ark CLI。

- 复制
	npm install -g @volcengine/ark-cli@latest
	arkcli --version

2. 登录 Ark CLI。

- ![image-20260624-144256-509.png](https://arkdoc.tos-cn-beijing.volces.com/images/CodingPlan/image-20260624-144256-509.png)

- 根据界面提示选择登录方式。首次登录时，按提示完成以下配置：

1. 选择项目（Project） ：可按需选择，或默认选择 账号全部资源 。

2. 选择消费模式（Type） ：选择 coding-plan。

- ![image-20260624-163746-727.png](https://arkdoc.tos-cn-beijing.volces.com/images/CodingPlan/image-20260624-163746-727.png)

3. 运行 ArkCLI Helper，根据界面提示完成 Claude Code 的配置。

1. 在命令行界面输入 arkcli helper 命令，启动 ArkCLI Helper。

2. 选择要配置的 Plan profile ：选择 coding-plan\_cn-beijing\_personal (Coding Plan)。

- ![image-20260624-163451-813.png](https://arkdoc.tos-cn-beijing.volces.com/images/CodingPlan/image-20260624-163451-813.png)

3. 选择默认 model ：按需选择。

4. 选择要配置的 AI Agent ：Claude Code。

- ![image-20260624-163912-096.png](https://arkdoc.tos-cn-beijing.volces.com/images/CodingPlan/image-20260624-163912-096.png)

完成上述操作后，将自动完成 Claude Code 的基础配置，无需手动安装。

方式 2：手动配置

完成Claude Code安装后，配置以下信息。

- ANTHROPIC\_BASE\_URL ：https://ark.cn-beijing.volces.com/api/coding

- ANTHROPIC\_AUTH\_TOKEN ： [获取API Key](https://console.volcengine.com/ark/region:ark+cn-beijing/apikey)

- ANTHROPIC\_MODEL: 支持配置 Model Name （实时切换模型）、配置ark-code-latest（控制台切换模型）两种方式，具体见 [模型配置](https://www.volcengine.com/docs/82379/1928262?lang=zh#3a775131) 。

注意

配置前注意清除以下环境变量，避免与旧配置冲突：

复制

unset ANTHROPIC\_AUTH\_TOKEN

unset ANTHROPIC\_BASE\_URL

若变量在 ~/.bashrc / ~/.zshrc 中，可以备份并同步删除对应变量。

配置步骤如下：

1. 创建并打开 settings.json 配置文件。文件路径因系统而异，具体操作如下：

2. 编辑配置文件，需要修改的配置信息如下：

- <ARK\_API\_KEY>：替换为您自己的 [API Key](https://console.volcengine.com/ark/region:ark+cn-beijing/apikey) 。

- <Model\_Name>：更新为需要使用的模型信息，如ark-code-latest。支持的模型信息参见 [模型配置](https://www.volcengine.com/docs/82379/1928262?lang=zh#3a775131) 。

- 复制
	{
	"env": {
	"ANTHROPIC\_AUTH\_TOKEN": "<ARK\_API\_KEY>",
	"ANTHROPIC\_BASE\_URL": "https://ark.cn-beijing.volces.com/api/coding",
	"ANTHROPIC\_MODEL": "<MODEL\_NAME>",
	"ANTHROPIC\_DEFAULT\_HAIKU\_MODEL": "<MODEL\_NAME>",
	"ANTHROPIC\_DEFAULT\_SONNET\_MODEL": "<MODEL\_NAME>",
	"ANTHROPIC\_DEFAULT\_OPUS\_MODEL": "<MODEL\_NAME>",
	"CLAUDE\_CODE\_SUBAGENT\_MODEL": "<MODEL\_NAME>"
	}
	}

说明

- 要开启 1M 上下文窗口，配置参见 [扩展上下文](https://www.volcengine.com/docs/82379/1928262?lang=zh#7a3e5c1b) 。

- 推荐使用完整模型配置，并按任务复杂度选择模型：Haiku（轻量）、Sonnet（日常）、Opus（复杂）。

- CLAUDE\_CODE\_SUBAGENT\_MODEL 建议与主模型保持一致。

- ANTHROPIC\_DEFAULT\_HAIKU\_MODEL 建议设置为小尺寸模型，通常不会影响整体使用效果。

3. 编辑或新增.claude.json 文件，修改或新增 hasCompletedOnboarding 字段值为 true。

1. 不同系统.claude.json配置文件路径如下：

- macOS/Linux：~/.claude.json

- Windows：C:\\Users\\<用户名>\\.claude.json

2. 修改或新增的字段信息如下：

- 复制
	{
	"hasCompletedOnboarding": true
	}

保存配置文件后，在新的终端窗口执行后续命令。

三、启动并使用 Claude Code

开始使用

1. 启动 Claude Code：进入项目目录，执行claude命令，即可开始使用 Claude Code。

- 复制
	cd my-project
	claude

- 启动后，选择信任此文件夹，允许 Claude Code 访问该文件夹中的文件。

- ![](https://p9-arcosite.byteimg.com/tos-cn-i-goo7wpa0wc/f621624ffa07485ea37f49ea96c8e8f4~tplv-goo7wpa0wc-image.image)

2. 模型状态验证：输入/status确认模型状态。

3. 在 Claude Code 中对话。

更多使用教程请参见 [Claude Code 官网](https://code.claude.com/docs/zh-CN/quickstart#%E6%AD%A5%E9%AA%A4-4%EF%BC%9A%E6%8F%90%E5%87%BA%E6%82%A8%E7%9A%84%E7%AC%AC%E4%B8%80%E4%B8%AA%E9%97%AE%E9%A2%98) 。

切换模型

请根据模型配置方式，选择对应的模型切换方案。

<iframe src="//about:blank" frameborder="0"></iframe>

| 配置方式 | 配置ark-code-latest | 配置 Model Name |
| --- | --- | --- |
| 切换模型操作 | 1. 在 [开通管理页面](https://console.volcengine.com/ark/region:ark+cn-beijing/openManagement?LLM=%7B%7D&advancedActiveKey=subscribe) 选择要使用的模型，切换后3-5分钟即可生效。  2. 在命令行窗口，启动 Claude Code 后，输入/status确认模型状态。 | - 启动时：执行claude --model <Model\_Name>，可指定对应的模型。  - 对话期间：执行/model <Model\_Name>切换模型。 |

扩展上下文

glm-5.2、deepseek-v4-flash、deepseek-v4-pro 支持 1M 上下文窗口，可用于包含大型代码库的长会话。以 glm-5.2 模型为例，配置信息如下：

复制

{

"env": {

"ANTHROPIC\_AUTH\_TOKEN": "<ARK\_API\_KEY>",

"ANTHROPIC\_BASE\_URL": "https://ark.cn-beijing.volces.com/api/coding",

"CLAUDE\_CODE\_AUTO\_COMPACT\_WINDOW": "1000000",

"ANTHROPIC\_MODEL": "glm-5.2\[1m\]",

"ANTHROPIC\_DEFAULT\_HAIKU\_MODEL": "glm-5.2\[1m\]",

"ANTHROPIC\_DEFAULT\_SONNET\_MODEL": "glm-5.2\[1m\]",

"ANTHROPIC\_DEFAULT\_OPUS\_MODEL": "glm-5.2\[1m\]"

}

}

注意

- 开启 1M 上下文时，需要在模型名称后增加 \[1m\] 后缀，如 glm-5.2\[1m\]，同时配置压缩窗口大小参数 "CLAUDE\_CODE\_AUTO\_COMPACT\_WINDOW": "1000000"。

- 若模型参数加上 \[1m\] 后缀后，Claude Code 识别模型不存在，需升级 Claude Code 到最新版本后重试。

使用 CC Switch

CC Switch 是一款跨平台桌面应用，专为使用 AI 编程工具的开发者设计。它帮助你统一管理 Claude Code、Claude Desktop、Codex、Gemini CLI、OpenCode、OpenClaw 和 Hermes 等受管应用的配置。

支持的平台

- Windows 10 及以上

- macOS 12 (Monterey) 及以上

- Linux Ubuntu 22.04+ / Debian 11+ / Fedora 34+（x64 / ARM64）

安装

更多安装方式和详细说明请参考 [CC Switch 官方文档](https://github.com/farion1231/cc-switch) 。

添加供应商

1. 在 CC Switch 中添加供应商：

1. 在主界面顶部图标栏选中 Claude Code 图标，点击右上角 + 进入添加新供应商。

2. 选择 自定义配置 ，填写以下信息：

- API Key ：替换为您自己的 [API Key](https://console.volcengine.com/ark/region:ark+cn-beijing/apikey)

- 请求地址 ：https://ark.cn-beijing.volces.com/api/coding

3. 展开高级选项，根据当前套餐支持的模型范围，分别配置 Sonnet、Opus、Fable、Haiku 模型，可配置的模型见 [模型配置](https://www.volcengine.com/docs/82379/1928262?lang=zh#3a775131) 。

注意

建议 Haiku 设置为小尺寸模型。

4. 选择右下角 添加 ，完成供应商配置。

2. 回到首页，点击右侧 启用 按钮，新开一个 Claude Code 会话使配置生效。

安装使用 Claude Code IDE 插件

1. 安装 Claude Code 并配置好环境变量，具体参考 [接入Claude Code](https://www.volcengine.com/docs/82379/1928262?lang=zh#7432bab1) 。

- Claude Code IDE 插件依赖 Claude Code CLI 工具，需先完成 Claude Code的安装及配置。

2. 安装并使用 IDE 插件。

因 IDE 插件会迭代演进，以下内容仅供参考，具体的安装及使用可参考 [Visual Studio Code](https://code.claude.com/docs/en/vs-code) 、 [JetBrains IDEs](https://code.claude.com/docs/en/jetbrains) 。

常见问题

- [Claude Code 如何开启深度思考模式？](https://www.volcengine.com/docs/82379/2165245?lang=zh#1c6446b6)

错误码

在编程工具中使用 Coding Plan 套餐时可能会遇到报错，您可以根据错误信息查看对应的报错场景并定位问题，具体可参考 [错误码](https://www.volcengine.com/docs/82379/1299023?lang=zh) 。

最近更新时间：2026.07.01 19:39:49

这个页面对您有帮助吗？

rangeDom

<iframe src="about:blank"></iframe>