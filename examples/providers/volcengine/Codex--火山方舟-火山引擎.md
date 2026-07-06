---
title: "Codex--火山方舟-火山引擎"
source: "https://www.volcengine.com/docs/82379/2556056?lang=zh"
author:
published:
created: 2026-07-02
description: "火山引擎官方文档中心，产品文档、快速入门、用户指南等内容，你关心的都在这里，包含火山引擎主要产品的使用手册、API或SDK手册、常见问题等必备资料，我们会不断优化，为用户带来更好的使用体验"
tags:
  - "clippings"
---
接入 AI 工具

Codex

本文介绍如何配置方舟 Coding Plan 个人版到 Codex CLI 中使用。

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

Codex CLI

一、安装 Codex CLI

前提条件：安装 [Node.js 18 或更新版本](https://nodejs.org/en/download/) 。

在命令行界面，执行以下命令安装 Codex CLI。

复制

npm i -g @openai/codex

安装结束后，执行以下命令检查版本。

复制

codex --version

二、配置 Codex CLI

1. 创建并打开 Codex 的配置文件。文件路径因系统而异，具体操作如下：

2. 编辑 config.toml，需关注的配置信息如下：

- model：配置方式参见 [模型配置](https://www.volcengine.com/docs/82379/2556056?lang=zh#3a775131) 。

- env\_key：设置的是环境变量名称，请不要直接修改 ARK\_API\_KEY，您需要在下一步设置该环境变量的值。

- 复制
	model = "<Model\_Name>"
	model\_provider = "volcengine-coding-plan"
	model\_supports\_reasoning\_summaries = true
	\[model\_providers.volcengine-coding-plan\]
	name = "volcengine-coding-plan"
	base\_url = "https://ark.cn-beijing.volces.com/api/coding/v3"
	env\_key = "ARK\_API\_KEY"

注意

- model\_supports\_reasoning\_summaries = true：开启推理能力。

- model\_reasoning\_effort：控制思考长度，可以设置为 low、medium、high。

- minimax-m2.7、kimi-k2.6、kimi-k2.7-code 不支持设置 model\_supports\_reasoning\_summaries = true。

3. 配置环境变量，需要将 ARK\_API\_KEY 环境变量设置为 [API Key](https://console.volcengine.com/ark/region:ark+cn-beijing/apikey) 。

三、开始使用

执行以下命令启动 Codex CLI。

复制

codex

Codex 桌面客户端

一、安装 Codex 桌面客户端

从 [Codex app](https://developers.openai.com/codex/app) 下载并安装 Codex 桌面客户端。

二、配置 Codex 桌面客户端

登录 Codex 桌面客户端后，根据以下步骤进行配置。

1. 单击左下角的 设置 - 设置 。

- ![image-20260630-172328-897.png](https://arkdoc.tos-cn-beijing.volces.com/images/CodingPlan/image-20260630-172328-897.png)

2. 在 配置 页面的 自定义 config.toml 设置 区域，单击 打开 config.toml 。

- ![image-20260630-172525-618.png](https://arkdoc.tos-cn-beijing.volces.com/images/CodingPlan/image-20260630-172525-618.png)

3. 编辑 config.toml，需关注的配置信息如下。

- model：配置方式参见 [模型配置](https://www.volcengine.com/docs/82379/2556056?lang=zh#3a775131) 。

- env\_key：设置的是环境变量名称，请不要直接修改 ARK\_API\_KEY，您需要在下一步设置该环境变量的值。

- 复制
	model = "<Model\_Name>"
	model\_provider = "volcengine-coding-plan"
	model\_supports\_reasoning\_summaries = true
	\[model\_providers.volcengine-coding-plan\]
	name = "volcengine-coding-plan"
	base\_url = "https://ark.cn-beijing.volces.com/api/coding/v3"
	env\_key = "ARK\_API\_KEY"

注意

- model\_supports\_reasoning\_summaries = true：开启推理能力。

- model\_reasoning\_effort：控制思考长度，可以设置为 low、medium、high。

- minimax-m2.7、kimi-k2.6、kimi-k2.7-code 不支持设置 model\_supports\_reasoning\_summaries = true。

4. 配置环境变量，需要将 ARK\_API\_KEY 环境变量设置为 [API Key](https://console.volcengine.com/ark/region:ark+cn-beijing/apikey) ，具体操作可参见 [配置 Codex CLI](https://www.volcengine.com/docs/82379/2556056?lang=zh#6d2d0676) 。

三、开始使用

配置完成后，建议退出并重新启动 Codex 桌面客户端以使配置生效。重新启动后，即可在对话界面中使用。

错误码

在编程工具中使用 Coding Plan 套餐时可能会遇到报错，您可以根据错误信息查看对应的报错场景并定位问题，具体可参考 [错误码](https://www.volcengine.com/docs/82379/1299023?lang=zh) 。

最近更新时间：2026.06.30 21:15:26

这个页面对您有帮助吗？

rangeDom

<iframe src="about:blank"></iframe>