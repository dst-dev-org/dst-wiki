---
title: "气球爆裂（balloon_pop_body）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "气球爆裂两注册：DoPop 按装备槽选头部／身体变体，未装备选名为 nil 不生成表现；body 回退属于无有效持有者分支，范围攻击与仇恨在调用方。"
aliases: ["balloon_pop_body", "balloon_pop_head", "气球爆裂", "气球爆炸特效", "气球pop"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "prefabs/balloons_common.lua", "prefabs/boatrace_seastack.lua", "prefabs/hats.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 气球爆裂（balloon_pop_body）

## 快速判断

- 职责与适用：气球（balloon／气球堆等）爆裂时的爆裂表现，两注册仅动画高低不同（body=`pop_low`／head=`pop_high`）；爆裂伤害、仇恨引导与移除在 DoPop 调用方。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L119-L131)
- 优先使用：查"气球爆裂特效"选本页；气球玩法（环绕气球、气球堆）读 balloons_common／balloons_empty。
- 重要副作用：无音效；两注册都 `FinalOffset 1`。[条目](../../../raw/current/scripts/fx.lua#L119-L131)
- 修改边界：气球位置一次性落点（不父挂）；动画是否重复未核验。[落点](../../../raw/current/scripts/prefabs/balloons_common.lua#L71-L72)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `balloon_pop_body` | 通用表条目 → 加载器代理＋一帧后本地实体，fn=FinalOffset1 | bank／build=`balloon_pop`，anim=`pop_low`。[条目](../../../raw/current/scripts/fx.lua#L119-L124) |
| `balloon_pop_head` | 同上协议 | anim 换 `pop_high`。[条目](../../../raw/current/scripts/fx.lua#L126-L131) |

两注册在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，差异仅 pop_low／pop_high | 同族同协议 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L119-L131) |
| 生成者：DoPop 选槽 | 气球爆裂时（`DeactiveBalloon` 已执行）分三路：**有有效持有者且未骑乘**——按装备槽选名（头部→head、身体→body、未装备→选名 nil 不生成表现），气球 Remove；**有有效持有者但骑乘**——仅有 DeactiveBalloon，无表现、不 Remove、无范围攻击；**无有效持有者**——body 回退＋Hide＋延迟范围攻击 | 服务端；fx 生成在**气球**位置（非持有者） | [三路结构](../../../raw/current/scripts/prefabs/balloons_common.lua#L58-L84) |
| DoPop 伴随效果 | 选槽命中分支内：对半径 10 的 `balloonomancer` 引导仇恨（非玩家持有者）＋`doareaattack`；无主回退分支为延迟范围攻击 | 服务端；骑乘分支两者皆无 | [伴随](../../../raw/current/scripts/prefabs/balloons_common.lua#L65-L72)、[回退](../../../raw/current/scripts/prefabs/balloons_common.lua#L79-L84) |
| 生成者：赛船海堆 | `boatrace_seastack` 掉落后在落点直接生成 body 版并移除 | 服务端；不经 DoPop | [直落点](../../../raw/current/scripts/prefabs/boatrace_seastack.lua#L28-L29) |
| 依赖声明 | balloonvest／hats（气球帽）等各自声明所用变体 | 仅声明 | [hats](../../../raw/current/scripts/prefabs/hats.lua#L6853-L6853) |
| 资源 | bank／build=`balloon_pop` | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L119-L131) |

## 典型接入

### 复用爆裂表现

- 前提与选择：只需爆裂表现可直接 Spawn（body 为通用回退）；要复刻"按装备槽选名＋范围攻击＋仇恨"需走 DoPop 同构逻辑。
- 生效与撤销：落点一次性动画，animover 自删。
- 原版依据：[DoPop 选槽](../../../raw/current/scripts/prefabs/balloons_common.lua#L57-L71)。

```lua
local fx = SpawnPrefab("balloon_pop_body")
fx.Transform:SetPosition(balloon.Transform:GetWorldPosition())
```

- 调整与验证：head／body 只差动画高度，选名规则在调用方；范围攻击参数在 DoPop 内不在 fx。

## 常见陷阱与限制

- **误区：爆裂伤害属于 fx**：`doareaattack` 与 balloonomancer 仇恨引导都在 DoPop 内；fx 实体无 combat。[伴随](../../../raw/current/scripts/prefabs/balloons_common.lua#L66-L73)
- **边界：选槽只认装备位且无 body 兜底**：只有"当前装备在头／身"的气球才对应 head／body；有效持有者下手持或背包中的气球选名为 nil——不生成爆裂表现（气球仍移除），不会回退 body。[选槽](../../../raw/current/scripts/prefabs/balloons_common.lua#L61-L63)
- **边界：骑乘仅解除激活**：骑乘时 `DeactiveBalloon` 之后直接返回——无爆裂表现、不 Hide、不 Remove、无范围攻击；"骑乘落入回退"与本页前稿"整段跳过"都不成立。[缩进实证](../../../raw/current/scripts/prefabs/balloons_common.lua#L59-L77)
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。两注册配置、加载器协议、DoPop 全分支（选槽／骑乘豁免／仇恨／无主回退）、赛船海堆直落点、依赖声明、.po 名称键核验完毕。未覆盖：`balloon_pop` 动画资源内容、气球全玩法（balloons 页面范围）、晚加入时序。
- 游戏运行：未运行；爆裂视觉与范围攻击未实测。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
