---
title: "弹弓弹药命中表现（slingshotammo_hitfx_rock）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "通用表 18 个 slingshotammo_hitfx_* 注册：投射物命中目标时的落点表现，按弹药覆符号／换特定动画／叠光照；火药条目仅暴击时抑制常规表现，slingshot_aoe_fx 不在本组。"
aliases: ["slingshotammo_hitfx_rock", "slingshotammo_hitfx_gold", "slingshotammo_hitfx_marble", "slingshotammo_hitfx_thulecite", "slingshotammo_hitfx_honey", "slingshotammo_hitfx_freeze", "slingshotammo_hitfx_slow", "slingshotammo_hitfx_poop", "slingshotammo_hitfx_moonglass", "slingshotammo_hitfx_dreadstone", "slingshotammo_hitfx_gunpowder", "slingshotammo_hitfx_lunarplanthusk", "slingshotammo_hitfx_purebrilliance", "slingshotammo_hitfx_horrorfuel", "slingshotammo_hitfx_gelblob", "slingshotammo_hitfx_scrapfeather", "slingshotammo_hitfx_stinger", "slingshotammo_hitfx_trinket_1", "弹弓命中表现", "弹弓命中特效", "沃尔特弹弓特效", "弹弓弹药命中"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "prefabs/slingshotammo.lua", "prefabs/slingshotaoefx.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 弹弓弹药命中表现（slingshotammo_hitfx_rock）

## 快速判断

- 职责与适用：沃尔特弹弓各弹药投射物命中目标时的伴随表现，共 18 注册（SHOT_TYPES 循环注册），纯表现；伤害由弹药与武器链结算。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[循环](../../../raw/current/scripts/fx.lua#L4081-L4134)
- 优先使用：查"弹弓命中特效／某种弹药打中什么效果"选本页；弹丸本身、AOE 范围圈（slingshot_aoe_fx，见 [[reference/prefabs/slingshot_ice_aoe_fx|弹弓 AOE 命中圈]]）与弹药 debuff 是其他注册，不在本页。
- 重要副作用：无；全员 `SetFinalOffset(3)`、配音效 `dontstarve/characters/walter/slingshot/<type>`。[fn](../../../raw/current/scripts/fx.lua#L4110-L4134)
- 修改边界：命中时在**目标位置**落点生成；动画是否循环未核验。[ImpactFx](../../../raw/current/scripts/prefabs/slingshotammo.lua#L72-L77)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `slingshotammo_hitfx_rock` | 循环 `table.insert` 进通用表 → 加载器代理＋一帧后本地实体 | anim=`used`，**不覆符号**（保留 rock 显示石弹）。[fn 分支](../../../raw/current/scripts/fx.lua#L4117-L4132) |
| gold／marble／thulecite／honey／freeze／slow／poop／dreadstone／lunarplanthusk／gelblob／scrapfeather／trinket_1（12 名） | 同上协议 | anim=`used`＋`OverrideSymbol("rock", "slingshotammo", <type>)` 换成对应弹药符号。[循环](../../../raw/current/scripts/fx.lua#L4110-L4134) |
| `slingshotammo_hitfx_stinger` | 同上协议 | anim 换 `used_stinger`＋覆符号。[特定动画表](../../../raw/current/scripts/fx.lua#L4103-L4108) |
| `slingshotammo_hitfx_moonglass` | 同上协议 | anim 换 `used_moonglass`＋覆符号。[特定动画表](../../../raw/current/scripts/fx.lua#L4103-L4108) |
| `slingshotammo_hitfx_gunpowder` | 同上协议 | anim 换 `used_gunpowder`＋覆符号＋泛光＋`SetLightOverride(.1)`；**仅暴击时抑制常规表现**（见陷阱）。[fn 分支](../../../raw/current/scripts/fx.lua#L4126-L4128) |
| `slingshotammo_hitfx_purebrilliance` | 同上协议 | anim=`used`＋覆符号＋泛光＋`SetLightOverride(.1)`。[fn 分支](../../../raw/current/scripts/fx.lua#L4123-L4125) |
| `slingshotammo_hitfx_horrorfuel` | 同上协议 | anim=`used`＋覆符号＋`SetLightOverride(1)`。[fn 分支](../../../raw/current/scripts/fx.lua#L4121-L4122) |

定向检索简中 .po 未发现这 18 个注册名的 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 循环注册 18 名，差异仅动画／符号／光照 | 全族同协议 | [[reference/mechanisms/generic-fx-factory]]；[循环](../../../raw/current/scripts/fx.lua#L4110-L4134) |
| 数据装配 | 每条弹药数据 `data.impactfx = "slingshotammo_hitfx_" .. (data.symbol or "rock")`，并作为对应 `_proj` 投射物的依赖声明 | 服务端装配期决定映射，改 symbol 即换命中表现 | [装配](../../../raw/current/scripts/prefabs/slingshotammo.lua#L1573-L1585) |
| 生成者：投射物命中 | `OnHit` 先执行弹药自带 onhit，再 `ImpactFx`：目标仍有效且攻击者有效时，在**目标位置**落点生成；`inst.noimpactfx` 可抑制 | 服务端；顺序敏感（见陷阱） | [OnHit](../../../raw/current/scripts/prefabs/slingshotammo.lua#L89-L109)、[ImpactFx](../../../raw/current/scripts/prefabs/slingshotammo.lua#L72-L77) |
| 火药特例 | `OnHit_Gunpowder` 的**暴击分支**（`inst._crithit`）置 `noimpactfx = true` → 暴击命中不显示 `hitfx_gunpowder`，另生成 `slingshotammo_gunpowder_explode` 并带尘暴 AOE；非暴击仍正常生成 | 服务端；抑制只作用于暴击路径 | [火药命中](../../../raw/current/scripts/prefabs/slingshotammo.lua#L810-L831)、[绑定](../../../raw/current/scripts/prefabs/slingshotammo.lua#L1460-L1461) |
| 组外注册 | `slingshot_aoe_fx`（freeze／slow／purebrilliance／horrorfuel 等弹药的 AOE 表现）独立注册于 slingshotaoefx.lua，不随本组 bank 并入，见 [[reference/prefabs/slingshot_ice_aoe_fx|弹弓 AOE 命中圈]] | 服务端 | [独立注册](../../../raw/current/scripts/prefabs/slingshotaoefx.lua#L258-L258) |
| 资源 | bank／build=`slingshotammo`，覆符号把 `rock` 换成弹药符号 | 动画时长与循环性未核验 | [fn](../../../raw/current/scripts/fx.lua#L4117-L4132) |

## 典型接入

### 新弹药复用命中表现

- 前提与选择：在 slingshotammo 的弹药数据里设 `symbol`，装配即得 `slingshotammo_hitfx_<symbol>`；符号需已存在于 `slingshotammo` 动画 build，否则覆符号无效果。
- 生效与撤销：命中目标位置一次性动画，animover 自删；投射物移除不影响表现。
- 原版依据：[装配](../../../raw/current/scripts/prefabs/slingshotammo.lua#L1573-L1585)。

```lua
-- 弹药数据内
local ammo_data = {
    name = "myammo",
    symbol = "gold",        -- 命中表现复用 gold 的符号与 hitfx 名
    damage = TUNING.SLINGSHOT_AMMO_DAMAGE_GOLD,
}
```

- 调整与验证：想关闭命中表现可在投射物上置 `noimpactfx = true`（火药即此做法）；抑制只作用于 `ImpactFx`，不影响弹药自身 onhit 特效。

## 常见陷阱与限制

- **边界：hitfx_gunpowder 暴击时被抑制**：`OnHit_Gunpowder` 的 `_crithit` 分支置 `noimpactfx`，暴击命中的常规表现由 `slingshotammo_gunpowder_explode` 替代；非暴击命中仍正常显示该注册。[分支](../../../raw/current/scripts/prefabs/slingshotammo.lua#L810-L831)
- **误区：命中表现等于命中判定**：表现生成前已有目标有效性判断，伤害与 debuff 由弹药 onhit／武器链结算；没有 `ImpactFx` 不代表没打中（火药即反例）。[顺序](../../../raw/current/scripts/prefabs/slingshotammo.lua#L89-L109)
- **边界：rock 不覆符号**：`OverrideSymbol` 只在非 rock 弹药执行；rock 命中表现就是动画自带的石弹符号，想换显示需自行覆符号。[分支](../../../raw/current/scripts/fx.lua#L4118-L4119)
- **边界：目标中途失效不生成**：弹药 onhit 执行后目标若已失效（移除等），`ImpactFx` 跳过；失效判断先于生成。[判断](../../../raw/current/scripts/prefabs/slingshotammo.lua#L96-L99)
- **边界：动画是否循环未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。18 名循环注册与三种动画／两类光照特例、弹药数据装配、OnHit 顺序、火药抑制路径与 slingshot_aoe_fx 组外边界逐条核验；.po 名称键定向核验无结果。未覆盖：18 种弹药的完整伤害与 debuff 配置（归弹药页范围）、used 系动画资源内容。
- 游戏运行：未运行；覆符号显示与光照表现未实测。

返回 [[reference/prefabs/fx-status-index|增益、受击、护盾与状态反馈]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
