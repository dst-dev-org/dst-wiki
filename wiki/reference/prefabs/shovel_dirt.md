---
title: "铲土飞尘（shovel_dirt）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "铲／挖动作的飞尘表现：海狸啃咬态 DIG 分支在目标位置落点，冬牧兔人三处在自身位置落点；纯表现，挖掘结算在动作逻辑。"
aliases: ["shovel_dirt", "铲土飞尘", "挖土特效", "铲子尘土"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "stategraphs/SGwilson.lua", "stategraphs/SGcozy_bunnyman.lua", "prefabs/woodie.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 铲土飞尘（shovel_dirt）

## 快速判断

- 职责与适用：铲／挖类动作的飞尘表现，单注册；已核验入口为海狸啃咬态 DIG 分支（目标位置落点）与冬牧兔人三处（**自身位置**落点）。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L334-L338)
- 优先使用：查"铲土飞尘／挖土特效"选本页；已核验入口是海狸啃咬与冬牧兔人动作，普通铲子动作的对应表现未在本页核验。
- 重要副作用：无音效、无 fn。[条目](../../../raw/current/scripts/fx.lua#L334-L338)
- 修改边界：目标位置一次性落点（不父挂）；动画是否重复未核验。[落点](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5498-L5498)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shovel_dirt` | 通用表条目 → 加载器代理＋一帧后本地实体，无 fn | bank／build=`shovel_dirt`，anim=`anim`；无音效、无 tint、无 transform。[条目](../../../raw/current/scripts/fx.lua#L334-L338) |

注册名在简中 .po 无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，无附加字段 | 单注册 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L334-L338) |
| 生成者：玩家／海狸啃咬 | SGwilson `gnaw` 状态（海狸啃咬同时处理采矿与挖掘）的 **DIG 分支**在目标位置落点（mine 分支另走 PlayMiningFX，见 [[reference/prefabs/mining_fx|mining_fx]]）；挖掘结算在动作回调 | 服务端；入口是啃咬态的分支，非普通铲子动作 | [DIG 分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5490-L5498) |
| 生成者：冬牧兔人 | `SGcozy_bunnyman` 两处动作帧＋`spawn_loop` 态 onupdate 的**5% 概率分支**（每次更新掷骰）落点 | 服务端；第三处不是固定动作帧 | [动作帧](../../../raw/current/scripts/stategraphs/SGcozy_bunnyman.lua#L549-L552)、[概率分支](../../../raw/current/scripts/stategraphs/SGcozy_bunnyman.lua#L575-L578) |
| 依赖声明 | woodie（伍迪）声明本名（海狸挖木类动作） | 仅声明；快照内 woodie 无对应 SpawnPrefab | [声明](../../../raw/current/scripts/prefabs/woodie.lua#L58-L58) |
| 资源 | bank／build=`shovel_dirt` | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L334-L338) |

## 典型接入

### 复用飞尘表现

- 前提与选择：自定义挖／埋动作在对应动作帧对目标位置 Spawn；结算与可挖判定由动作逻辑自理。
- 生效与撤销：落点一次性动画，animover 自删。
- 原版依据：[玩家挖掘落点](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5498-L5498)。

```lua
local fx = SpawnPrefab("shovel_dirt")
fx.Transform:SetPosition(target.Transform:GetWorldPosition())
```

- 调整与验证：兔人路径在动作帧多点生成（同动作多次落点）时保持各自位置，fx 间无联动。

## 常见陷阱与限制

- **误区：fx 等于挖掘完成**：生成点挂在 gnaw／动作帧上，铲除结算与掉落在动作回调；先有结算安排后有表现，或同帧并存，因果不在 fx。[DIG 分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5490-L5498)
- **边界：woodie 仅声明**：依赖声明不等于生成；快照内 woodie.lua 未检索到对应 SpawnPrefab。[声明](../../../raw/current/scripts/prefabs/woodie.lua#L58-L58)
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。单注册配置、加载器协议、SGwilson gnaw DIG 分支、冬牧兔人动作帧与 spawn_loop 概率分支、woodie 声明辨析核验；快照内无 digger 组件（仅 gravedigger 属迁墓工具）。.po 名称键定向核验无结果。未覆盖：`shovel_dirt` 动画资源内容、普通铲子动作的表现路径、晚加入时序。
- 游戏运行：未运行；飞尘视觉未实测。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
