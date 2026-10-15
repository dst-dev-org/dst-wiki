---
title: "quagmire_recipeprices"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "熔炉（暴食）活动菜品估价广播队列：主模拟把估价结果排队写入网络字段，客户端解码后推 quagmire_recipeappraised 事件供菜谱书与通知栏消费。"
aliases: ["暴食估价", "quagmire_recipeprices", "菜品估价广播", "quagmire 估价"]
tags: [component, network, server-client]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/quagmire_recipeprices.lua", "prefabs/quagmire_network.lua", "quagmire_recipebook.lua", "widgets/quagmire_notificationwidget.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# quagmire_recipeprices

## 快速判断

- 职责与适用：挂在活动专用网络实体 `quagmire_network`（`persists = false`、CLASSIFIED 标签）上的单向广播组件：主模拟收 `ms_quagmire_recipeappraised` 入队，每 3 秒最多广播一条估价；客户端从网络字段解码并推世界事件。[实体](../../../raw/current/scripts/prefabs/quagmire_network.lua#L45-L48) [安装](../../../raw/current/scripts/prefabs/quagmire_network.lua#L55)
- 优先使用：客户端消费估价结果走世界事件 `quagmire_recipeappraised`（product/dish/silverdish/maxvalue/matchedcraving/snackpenalty/coins），不要直接读网络字段。[事件](../../../raw/current/scripts/components/quagmire_recipeprices.lua#L108-L116)
- 重要副作用：客户端解码时按 `QUAGMIRE_USE_KLUMP` 加载菜品动态贴图与字符串（klump 加密资源），并延迟 3 秒本地清空显示；不触发 dirty 不回传。[资源](../../../raw/current/scripts/components/quagmire_recipeprices.lua#L81-L88)
- 修改边界：无公开方法、无存读钩子（实体不持久化）；仅在暴食活动世界有意义。[安装段](../../../raw/current/scripts/prefabs/quagmire_network.lua#L53-L55)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 主模拟事件 `ms_quagmire_recipeappraised` | 监听 TheWorld | 入参含 product、dish（"plate"/"bowl"）、silverdish、maxvalue、matchedcraving（9 种 craving 之一）、snackpenalty、coins[1..4]；转 ID 后入队，队列空闲立即开跑。[处理](../../../raw/current/scripts/components/quagmire_recipeprices.lua#L147-L164) [常量](../../../raw/current/scripts/components/quagmire_recipeprices.lua#L11-L33) |
| 客户端世界事件 `quagmire_recipeappraised` | TheWorld 上 | `_recipename` dirty 后 3 秒排队节拍内解码推送；`matchedcraving` 已转回名称字符串。[推送](../../../raw/current/scripts/components/quagmire_recipeprices.lua#L108-L116) [节拍](../../../raw/current/scripts/components/quagmire_recipeprices.lua#L121-L141) |
| 网络字段组 | `net_string`/`net_tinybyte`/`net_bool`/`net_smallbyte` | `_recipename`（带 `recipedirty` 事件）、`_klumpkey`、`_dish`、`_silverdish`、`_maxvalue`、`_matchedcraving`、`_snackpenalty`、`_coins[1..4]`。[定义](../../../raw/current/scripts/components/quagmire_recipeprices.lua#L52-L62) |
| 消费者 | 原版两处 | 客户端菜谱书模块与通知栏挂件监听 `quagmire_recipeappraised`。[菜谱书](../../../raw/current/scripts/quagmire_recipebook.lua#L234-L235) [通知栏](../../../raw/current/scripts/widgets/quagmire_notificationwidget.lua#L30) |

## 常见陷阱与限制

- **广播是串行队列：** 每 3 秒一条，多条估价按到达顺序播放；主模拟在队列空时 `_recipename:set_local("")` 结束，客户端 `_task` 延迟清显示但不推空事件。[队列](../../../raw/current/scripts/components/quagmire_recipeprices.lua#L121-L141)
- **coins 恒 4 槽：** `NUM_COIN_TYPES = 4`，缺省补 0；解析金币数要按定长数组处理。[定长](../../../raw/current/scripts/components/quagmire_recipeprices.lua#L149-L151)
- **klumpkey 可能为空串：** 无 klump 密钥时跳过资源加载，事件照常推送；不要把资源加载失败当作估价失败。[分支](../../../raw/current/scripts/components/quagmire_recipeprices.lua#L81-L88)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（198 行），核验 quagmire_network 安装与客户端两处事件消费者。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。

相关：[[reference/components/quagmire_recipebook]]
