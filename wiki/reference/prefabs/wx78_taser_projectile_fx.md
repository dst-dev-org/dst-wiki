---
title: "WX-78 电击蓄能爆发（wx78_taser_projectile_fx）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "WX-78 电击模块的常驻电壳与蓄满爆发：电壳父挂玩家、按蓄能百分比切三档动画并闪白；爆发时脱父挂定爆点，以玩家（或对友军时以 FX）为 attacker 直接结算电击范围伤害。"
aliases: ["wx78_taser_projectile_fx", "WX-78 电击爆发", "电击蓄能特效", "WX78 电壳", "电击模块爆发"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/wx78_taser_projectile.lua", "components/wx78_taserbuildup.lua", "wx78_moduledefs.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# WX-78 电击蓄能爆发（wx78_taser_projectile_fx）

## 快速判断

- 职责与适用：`prefabs/wx78_taser_projectile.lua` 只注册 1 个联网混合 FX `wx78_taser_projectile_fx`，服务 WX-78 的 `wx78_taserbuildup` 组件：组件创建时生成一只常驻电壳并父挂玩家，蓄能期间用 `DoFlash` 切 `shock_fx_low/med/high` 三档并闪烁玩家；蓄满或主动释放时 `Explode` 脱父挂、贴定爆点、按半径结算电击。[注册](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L312-L312)、[组件生成](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L120-L126)
- 优先使用：查"WX-78 电击模块的壳／爆发"看本页；需要复刻"常驻壳＋一次爆发"时，重点是 `SetFXOwner`／`DoFlash`／`Explode` 三个接口与 `showbase`／`baseradius` 两个 netvar。[接口](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L181-L187)
- 重要副作用：爆发伤害**不是** `owner.components.combat:DoAttack`；代码直接对每个受击者调 `v.components.combat:GetAttacked(attacker, damage, nil, "electric", inst.spdmg)`。attacker 通常是玩家；若受击者跟随玩家，则 attacker 换成 FX 自身（避免友军把账记到玩家头上）。[结算](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L96-L107)
- 修改边界：只覆盖该注册的工厂、壳／爆发状态、伤害循环与组件挂载；WX-78 技能树、模块数量、蓄能增长／衰减与熔炉事件不展开。
- 名称：`STRINGS.NAMES.WX78`＝"WX-78"（[po](../../../raw/current/scripts/languages/chinese_s.po#L352534-L352537)）；`wx78_taser_projectile_fx` 无 `STRINGS.NAMES` 键，标题中的"电击蓄能爆发"是已核验效果描述。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wx78_taser_projectile_fx` | `fn()`：Transform／AnimState／SoundEmitter／Network → bank/build `wx78_taser_blast_fx`、`shock_fx_low` 循环、bloom、光覆写 1、FinalOffset 1 → `showbase` net_bool＋`baseradius` net_float → `SetPrefabNameOverride("wx78")` → `FX`＋`NOCLICK`＋`notarget` → `SetPristine`。客户端监听 `showbasedirty` 并早退；主模拟 `Hide()`／`hidden=true`、挂 `updatelooper`、`combat`（默认 50、`ignorehitrange`、`KeepTargetFn=false`）、导出 `SetFXOwner`／`DoFlash`／`Explode`、监听 `animover`、`persists=false`、`targets={}`、`canhitplayers=true`。 | 单一注册；主模拟阶段默认 `Hide()` 的是隐身壳，`SetFXOwner` 才父挂玩家。伤害数值来自 `Explode` 参数，`combat` 组件主要用于 `CanTarget`／`IsAlly`。[工厂](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L257-L310) |

`wx78_taserbuildup` 组件由 WX-78 模块定义在 `taser_skill_activate` 里按模块数挂载（[挂载](../../../raw/current/scripts/wx78_moduledefs.lua#L690-L696)）；组件构造即 `SpawnFX()`，移除时 `OnRemoveFromEntity` 删 FX（[构造与移除](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L2-L31)）。

## 常驻壳：SetFXOwner 与 DoFlash

### SetFXOwner

`SetFXOwner(inst, owner)` 做四件事（[实现](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L181-L187)）：

- `entity:SetParent(owner.entity)`：壳跟随玩家；
- `inst.owner = owner`；
- `canhitplayers = not owner:HasTag("player") or TheNet:GetPVPEnabled()`；
- `targets[owner] = true`：爆发时不会打到自己。

### DoFlash

`DoFlash(duration, percent)` 按蓄能百分比选档并显示（[实现](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L210-L251)）：

| 百分比 | 动画 |
|---|---|
| `>= 0.75` | `shock_fx_high` |
| `>= 0.50` | `shock_fx_med` |
| 其余 | `shock_fx_low` |

- 显示后设 `duration`，在 `duration − 0.1 + random×0.2` 秒后播 `<anim>_pst`；`OnAnimOver` 在 pst 播完时 `Hide()`。[收尾](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L189-L197)
- 同时给 owner 加 `colouradder` 并每帧闪白／淡黄，`Flash_OnUpdate` 在 `flash` 用完后 `PopColour`。[闪白](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L130-L144)
- 重复 `DoFlash` 会取消上一只 pst 任务，重新计时；`Explode` 也会取消 pst 任务。[取消](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L153-L156)

### 组件的调用时机

- `FlashEffect` 在蓄能 ≥0.25 时按 `GetEffectDuration()` 调 `fx:DoFlash(duration, 0.75×percent)`（[实现](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L203-L208)）。
- 主动释放 `ReleaseBuildup` 只要求 ≥0.25：`fx:DoFlash(time, 1)` 并起 `detonate_time`（[释放](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L128-L142)）。
- `DoShockExplosion` 用**当前百分比**缩放伤害与半径：`fx:Explode(blast_damage × perc, blast_radius × perc)`。组件构造默认 `blast_damage=50`／`blast_radius=2`，但原版模块激活时 `taser_skill_updatebuildupstats` 会按模块数覆盖为 `n×50` 与 `2+n`；所以 1 模块满蓄能是 50／3、2 模块是 100／4，低蓄能再按比例缩小。[爆发](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L148-L150)、[模块覆盖](../../../raw/current/scripts/wx78_moduledefs.lua#L667-L674)

## 爆发：Explode 与伤害循环

`Explode` 把壳从玩家身上摘下并固定在爆点（[实现](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L146-L179)）：

```lua
local x, y, z = inst.entity:GetParent().Transform:GetWorldPosition()
inst.entity:SetParent(nil)
inst.Transform:SetPosition(x, y, z)
-- 取消 pst；按 radius 缩放；播 crackle_hit；18 帧后 DisableHits；动画长度+2 帧后 Remove
```

- `showbase=true`＋`baseradius=radius` 让客户端创建地面 `crackle_projection`；`OnShowBase` 用 `radius / CIRCLE_RADIUS_SCALE` 缩放本地子 FX，动画结束自删。[本地底座](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L18-L60)
- 伤害循环每帧扫 `_combat`＋`_health`，半径 `radius+3` 搜索、`radius+物理半径` 精确命中；`targets` 去重；墙上／limbo／noattack／playerghost 被排除。[扫描](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L67-L128)
- 电免疫目标 `damage_mult=1`；非免疫 `ELECTRIC_DAMAGE_MULT + ELECTRIC_WET_DAMAGE_MULT × 湿度`（1.5＋1×wet，[调值](../../../raw/current/scripts/tuning.lua#L659-L660)）。`damage = inst.damage × damage_mult`。
- owner 有效时：要求 `owner.combat:CanTarget(v)` 且非友军；`attacker = (v.follower:GetLeader() == owner and inst) or owner`；`GetAttacked(attacker, damage, nil, "electric", inst.spdmg)` 并推 `electrocute`。[owner 分支](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L96-L107)
- owner 为 nil（非佩戴场景）时：`v.combat:CanBeAttacked()`；非 PVP 下玩家跟随者视为友军（除非其 combat 目标本身是玩家）；然后 `GetAttacked(inst, ...)`。[无主分支](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L108-L124)
- `DisableHits` 在 18 帧后移除 `OnUpdate` 并关 `showbase`；`Remove` 在动画长度＋2 帧后执行。[窗口](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L62-L65)

## 关联机制导航

| 类型／对象 | 本注册如何使用 | 变体、端与条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：wx78_taserbuildup` | 生成／父挂／闪白／爆发／重生的驱动者 | 由 `wx78_moduledefs` 的 taser 技能按模块挂载 | [组件](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L2-L31)、[挂载](../../../raw/current/scripts/wx78_moduledefs.lua#L690-L696) |
| `组件：colouradder`（owner） | 闪白颜色叠加 | `DoFlash` 时按需添加 | [闪白](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L239-L250) |
| `组件：combat`（FX） | `CanTarget`／`IsAlly` 与默认伤害字段；实际伤害由 `Explode` 参数直接传入 | 只在主模拟挂 | [挂载](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L293-L296) |
| `TUNING.SKILLS.WX78.TASER_*` | 基础伤害 50、基础半径 2、单模块 +1 半径、上限 100 等 | 模块激活后按 `n×50` 与 `2+n` 覆盖；满蓄能才取满值，低蓄能再乘百分比 | [调值](../../../raw/current/scripts/tuning.lua#L7503-L7516)、[模块覆盖](../../../raw/current/scripts/wx78_moduledefs.lua#L667-L674) |
| `Prefab：wx78` | 名称覆写目标（死亡通告） | `SetPrefabNameOverride("wx78")` 只改显示名 | [覆写](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L275-L275) |

## 典型接入

### 手动创建一只电壳并爆发

- 前提与选择：只在主模拟调用；`SetFXOwner`／`DoFlash`／`Explode` 都只存在于主模拟实体。壳需要 owner 实体有 Transform。
- 生效与撤销：`SetFXOwner` 后壳跟随 owner；`DoFlash` 显示；`Explode` 脱父挂、造成一次范围伤害并自行删除。若不爆发，组件移除时必须 `fx:Remove()`。
- 原版依据：[组件生成](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L120-L126)、[爆发](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L148-L150)。

```lua
local fx = SpawnPrefab("wx78_taser_projectile_fx")
fx:SetFXOwner(owner)
fx:DoFlash(1.5, 1.0)          -- 满档高亮并显示
fx:Explode(50, 2)             -- 手动绝对参数：50 伤害、2 半径；原版 1 模块满蓄能是 50/3
```

- 调整与验证：`Explode` 的 damage／radius 是绝对值，不读组件百分比；要复刻蓄能缩放需像组件一样乘 `perc`。原版模块还会把默认 50／2 覆盖成 `n×50`／`(2+n)`，不要把手动 50／2 当模块满蓄能值。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：伤害 attacker 不总是玩家。** 正常目标记 owner（玩家）；若目标跟随 owner（友军），attacker 改成 FX 自身，避免友军仇恨记到玩家。按 attacker 做击杀归属时要考虑这条分支。[分支](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L102-L105)
- **边界：`combat` 的默认伤害不是实际伤害。** 工厂把 `combat:SetDefaultDamage(50)`，但 `OnUpdate` 用 `inst.damage`（`Explode` 参数）直接 `GetAttacked`；改默认伤害不会改爆发数值。[挂载](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L293-L296)、[结算](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L92-L105)
- **边界：电免疫不是"免伤"。** 电免疫目标取 `damage_mult=1`（吃基础伤害），非免疫取 1.5＋湿度倍率；两类都会推 `electrocute`。[倍率](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L92-L106)
- **边界：`Explode` 的伤害／半径由调用方定。** 组件传的是 `blast_damage×perc`／`blast_radius×perc`；默认 50／2，但原版 1 模块覆盖为 50／3、2 模块为 100／4。直接调 `Explode(50,2)` 是手动绝对参数，不等于原版模块满蓄能值，也不会自动按蓄能缩放。[组件](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L148-L150)、[模块覆盖](../../../raw/current/scripts/wx78_moduledefs.lua#L667-L674)
- **边界：pst 任务与闪白任务都挂在 FX 上。** FX 被提前删除时任务随实体取消；`Flash_OnUpdate` 直接索引 `inst.owner.components.colouradder`，owner 若在闪白期间失效没有 nil 保护（源码未处理该边界）。[闪白](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L130-L144)
- **误区：它是"投射物"。** 注册名带 `projectile`，但实际是父挂玩家的常驻壳，爆炸时脱父挂；没有飞行物或碰撞体。[工厂](../../../raw/current/scripts/prefabs/wx78_taser_projectile.lua#L257-L296)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/wx78_taser_projectile.lua`（311 行）与 `components/wx78_taserbuildup.lua`（246 行）；按需读取 `wx78_moduledefs.lua` 的 taser 技能挂载（L680-L700）、`tuning.lua` 的 `TASER_*`／电系倍率（L659-L660、L7503-L7516）。全域反查：`wx78_taser_projectile_fx` 仅 `components/wx78_taserbuildup.lua:121` 生成；`spellmastery` 无关。台账分诊结论（1 注册、生成组件 L121、SetFXOwner、showbase／baseradius、18 帧窗口、po 无 NAMES）复核属实；补充实核：伤害直接 `GetAttacked`、attacker 双分支、电免疫取 1 倍而非减半、实际伤害来自 `Explode` 参数而非 combat 默认值；原版模块按 n 覆盖 `blast_damage=n×50`、`blast_radius=2+n`，1 模块满蓄能 50／3、2 模块 100／4。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.WX78`＝"WX-78"（[L352534-L352537](../../../raw/current/scripts/languages/chinese_s.po#L352534-L352537)）；`wx78_taser_projectile_fx` 无 `STRINGS.NAMES` 键。
- 分档说明：`size_class: standard`——单注册，但需承载常驻壳／闪白／爆发三段、两种 attacker 分支与 `showbase` 本地底座；无多变体，未升 complex。
- 未覆盖：WX-78 技能树与模块数量、蓄能增长／衰减曲线、电击状态在玩家 SG 的表现、`colouradder` 工厂、动画包实际帧长；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/shadow_despawn|暗影消散、舞台影手与暗影憎恶光束]]（同为"混合 FX 直接结算伤害"的对照）
- 返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]（WX-78 电击模块）
