---
title: Claude Code + cc-switch + Skills 搭建出一个完整的 AI Agent 编程环境
description: 整理了 Claude Code + cc-switch + Skills 的入门教程，帮助你快速搭建完整的 AI 编程环境。三篇内容分别对应安装使用、API 切换工具、AI 技能扩展，建议按顺序学习。
published: 2026-03-10
tags: [AI编程, Claude Code, cc-switch]
category: 原创
draft: false
---

本文整理了 Claude Code + cc-switch + Skills 的入门教程，帮助你快速搭建完整的 AI 编程环境。下面三篇内容分别对应安装使用、API 切换工具、AI 技能扩展，建议按顺序学习。

## Claude Code 安装与使用

介绍了在国内使用 Claude Code 的两种安装方式（命令行直接安装和通过 IDE 插件安装），以及两种 API 配置方案（官方 Claude 账号和国内免费 LongCat 模型），重点讲解了使用美团开源的 LongCat 模型时的完整配置步骤，包括获取 API Key、修改配置文件、跳过登录验证等操作，最后演示了如何在项目目录中启动 Claude Code 并利用它进行代码开发。

<iframe src="//player.bilibili.com/player.html?bvid=BV1oPFDzQEG7&autoplay=0" scrolling="no" frameborder="no" framespacing="0" allowfullscreen="true" width="100%" height="500"></iframe>

**补充**：如果需要使用付费模型进行 AI 编程，比较推荐选择 MiniMax 的 Coding Plan Starter 套餐（29 元/月）。该套餐专门为开发者设计，每 5 小时提供 40 次 prompts 调用额度，支持 MiniMax M2 系列模型，适合 Claude Code、Cursor 等 AI 编程工具使用，能够满足大多数日常开发和代码辅助需求，性价比较高。

## cc-switch：快速切换 API 供应商

介绍了 cc-switch，一款用于 AI 编程时快速切换不同 API 服务资源的开源桌面工具（GitHub 4.2k 星），主要解决因 API 掉线、额度用完或需要更换模型（如从 GLM 切换到 Claude）时频繁修改配置的痛点。它通过图形化界面支持为 Claude Code 和 Codex 管理多个供应商配置，并具备一键切换、MCP 配置同步、WSL 环境自动穿透等实用功能，让用户在不同第三方 API、本地模型或官方接口之间自由组合。文章还特别提及一个第三方分支，通过代理模式实现了站点自动切换（故障转移）和无需重启服务的增强功能。

[在知乎阅读 cc-switch 完整介绍](https://zhuanlan.zhihu.com/p/1972328772964446521)

## Agent Skills：扩展 AI 能力

详细讲解目前非常火的 AI 技术 Agent Skills 是什么、怎么安装使用，并揭秘其内部原理以及如何开发自己的 Skills 技能包。从 Claude Code 官方技能市场到社区热门的 UI UX Pro MAX 前端工具，手把手教你告别千篇一律的 AI 审美和蓝紫渐变。视频还会对比 Agent Skills 和 MCP、斜杠命令的区别，解释"渐进式披露机制"如何节省上下文 tokens。无论你是刚接触 AI 编程的新手，还是想提升 Cursor / VS Code / Codex AI 编程效率的程序员，或者是对新技术感兴趣的 Vibe Coding 玩家，这期内容都能帮助你快速上手 Agent Skills，让 AI 真正成为你的得力助手。

<iframe src="//player.bilibili.com/player.html?bvid=BV1T7zzBQEaA&autoplay=0" scrolling="no" frameborder="no" framespacing="0" allowfullscreen="true" width="100%" height="500"></iframe>

## 推荐学习顺序

如果按学习顺序，推荐这样理解整个 AI 编程工具链：

1. 先安装并熟悉 **Claude Code**（核心 AI 编程工具）
2. 再使用 **cc-switch** 管理不同模型 API（解决 API 切换问题）
3. 最后通过 **Agent Skills** 扩展 AI 能力（让 AI 具备更多技能）

这样就可以搭建出一个完整的 AI Agent 编程环境。
