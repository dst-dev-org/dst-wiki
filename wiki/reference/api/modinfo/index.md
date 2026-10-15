---
title: modinfo 主题导航
created: 2026-10-11
updated: 2026-10-11
type: index
tags: [overview]
---

# modinfo

按需求直达页面或章节；以下三页独立读取，不要求依次加载。建立两个入口文件见 [[cookbook/mod-starter#最小-demo]]。

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/api/modinfo/fields|原生字段清单]] | 23 个顶层字段及优先级／依赖／图标／版本示例、默认行为与限制 | priority、mod_dependencies、forcemanifest；active／source-reviewed |
| [[reference/api/modinfo/configuration|配置项与名称]] | 配置字段／标题／数值／客户端选项、读取配置、本地名称与多语言 | configuration_options、GetModConfigData；active／source-reviewed |
| [[reference/api/modinfo/game-modes|自定义游戏模式]] | game_modes 条目、settings 初始化、模式与世界设置边界 | game_modes、AddGameMode；active／source-reviewed |

返回 [[reference/api/index|API]]。
