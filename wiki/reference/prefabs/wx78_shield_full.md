---
title: "WX78 护盾动画（wx78_shield_full）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "通用表六注册的 WX-78 电荷护盾表现：常驻 full／half、四条升破转场，全部父挂 WX-78；破盾转场 11 帧后脱离父级定在世界坐标，颜色与血条 UI 同步。"
aliases: ["wx78_shield_full", "wx78_shield_half", "wx78_shield_full_to_half", "wx78_shield_half_to_full", "wx78_shield_full_to_empty", "wx78_shield_half_to_empty", "WX78护盾动画", "护盾特效", "破盾特效", "电荷护盾动画", "WX78 盾特效"]
tags: [prefab, fx, network-sync]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "components/wx78_shield.lua", "wx78_moduledefs.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# WX78 护盾动画（wx78_shield_full）

## 快速判断

- 职责与适用：WX-78 电荷护盾的六种动画表现（常驻两态＋四条转场），全部父挂宿主；阈值切换、周期重播与读档重放的触发都在 [[reference/components/wx78_shield|wx78_shield 组件]]，本页只记动画与挂接。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[循环](../../../raw/current/scripts/fx.lua#L4159-L4197)
- 优先使用：查"护盾亮起／破盾动画"选本页；护盾充能、伤害拦截与穿透阈值选 [[reference/components/wx78_shield|wx78_shield]]。
- 重要副作用：无音效条目；fn 统一染 `243/187/6` 黄色（注释要求与血条 UI 的 `WX_SHIELD_COLOUR` 保持同步）＋`FinalOffset 1`。[fn](../../../raw/current/scripts/fx.lua#L4168-L4172)
- 修改边界：全部为父挂用例；破盾转场 11 帧后**主动脱挂**；动画是否循环未核验。[脱挂](../../../raw/current/scripts/fx.lua#L4174-L4185)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wx78_shield_full`／`wx78_shield_half` | 循环注册 → 加载器代理＋一帧后本地实体，`nofaced = true` | anim 同名常驻态；fn=`Wx78ShieldFn`（黄色＋FinalOffset 1）。[循环](../../../raw/current/scripts/fx.lua#L4187-L4197) |
| `wx78_shield_full_to_half`／`wx78_shield_half_to_full` | 同上协议 | 转场动画，同为 `Wx78ShieldFn`，无脱挂。[循环](../../../raw/current/scripts/fx.lua#L4187-L4197) |
| `wx78_shield_full_to_empty`／`wx78_shield_half_to_empty` | 同上协议，fn=`Wx78ShieldBreakFn` | 破盾转场：染黄后 **11×FRAMES 延时**执行 `Wx78ShieldSetNoParent`——有父级则按当前世界坐标 `SetParent(nil)`，之后不再跟随 WX-78。[破盾](../../../raw/current/scripts/fx.lua#L4181-L4185) |

定向检索简中 .po 未发现这 6 个注册名的 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 循环注册 6 名；数据内音效行被注释，无音效 | 全族同协议 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L4187-L4197) |
| 生成者：阈值切换 | 组件 `SetCurrent` 六分支：升到阈值上→`full`（原为 0）或 `half_to_full`（原在阈值下）；降到阈值下→破盾 `full_to_empty`／衰减 `full_to_half`（曾过阈值）、`half_to_empty`（直接清空）或 `half`（从 0 回升） | 服务端；穿透阈值默认 15，最大盾 100 | [状态机](../../../raw/current/scripts/components/wx78_shield.lua#L106-L126)、[默认值](../../../raw/current/scripts/components/wx78_shield.lua#L35-L37)、[[reference/components/wx78_shield|wx78_shield]] |
| 生成者：父挂 | `SpawnShieldEffect` 统一 `SpawnPrefab(名).entity:SetParent(inst.entity)`——六种全部挂到 WX-78 | 服务端组件调用；本地表现随宿主移动 | [父挂](../../../raw/current/scripts/components/wx78_shield.lua#L28-L30) |
| 生成者：周期重播 | `OnUpdate` 中 `effect_cooldown`（初始 5 秒）归零且仍有盾时重播 `full`／`half`（按阈值选），冷却重置为 10＋随机 8 秒；**前提是有充能源驱动组件更新**——无充能源时 `StopUpdatingComponent`，重播与回充都停 | 服务端；常驻态不是生成一次就永久显示 | [周期](../../../raw/current/scripts/components/wx78_shield.lua#L205-L218)、[更新启停](../../../raw/current/scripts/components/wx78_shield.lua#L56-L70) |
| 读档重放 | `OnLoad` 走 `SetCurrent`，读档有盾会再次触发对应特效 | 服务端；存档只存 current 值 | [读档](../../../raw/current/scripts/components/wx78_shield.lua#L226-L229) |
| 依赖声明 | 护盾模块电路 `wx78_moduledefs` 声明六名 | 仅声明 | [声明](../../../raw/current/scripts/wx78_moduledefs.lua#L1202-L1207) |
| 资源 | bank／build=`wx78_shield_fx`，六动画同名配置 | 动画时长与循环性未核验 | [循环](../../../raw/current/scripts/fx.lua#L4159-L4197) |

## 典型接入

### 复刻护盾表现切换

- 前提与选择：给自定义护盾接表现时，模仿"数值变化→按新旧值与阈值选注册名→父挂生成"的结构，而不是每帧生成常驻态。
- 生效与撤销：父挂动画随宿主；破盾转场 11 帧后脱挂定在世界坐标——期间移除宿主的表现归属未核验。
- 原版依据：[状态机](../../../raw/current/scripts/components/wx78_shield.lua#L106-L126)。

```lua
-- fx.lua 数据内（简摘）：破盾转场的延时脱挂
local function Wx78ShieldBreakFn(inst)
    Wx78ShieldFn(inst)  -- FinalOffset 1 + 黄色染色
    inst:DoTaskInTime(11 * FRAMES, Wx78ShieldSetNoParent)
end
```

- 调整与验证：改阈值只影响选名分支（`penetrationthreshold`），动画数据不变；颜色如需与 HUD 同步，改两处（数据 fn 与 healthbadge 常量）要一起动。

## 常见陷阱与限制

- **误区：存在 `wx78_shield_empty` 常驻注册**：枚举只有 full／half 与四条转场，"empty"只出现在 `*_to_empty` 名内；破盾后靠周期重播门禁（`currentshield > 0`）不再显示，没有空盾常驻条目。[枚举](../../../raw/current/scripts/fx.lua#L4159-L4167)
- **边界：常驻态靠周期重播维持，且有前提**：`full`／`half` 每次生成都是一次性动画实体，重播需要组件更新在跑（有充能源才 `StartUpdatingComponent`）且冷却到期时恰有盾。冷却在无盾期间降到非正后，`effect_cooldown > 0` 分支从此跳过——恢复护盾只会经 `SetCurrent` 触发数值切换表现，**周期重播不会自动恢复**（`SetCurrent` 与重新启动更新都不重置冷却）。[更新启停](../../../raw/current/scripts/components/wx78_shield.lua#L56-L70)、[状态机](../../../raw/current/scripts/components/wx78_shield.lua#L106-L126)、[冷却分支](../../../raw/current/scripts/components/wx78_shield.lua#L212-L218)
- **边界：转场选择看"曾过阈值"**：`full_to_half` 与 `half_to_empty` 的分界是上一帧是否 `>= penetrationthreshold`，不是旧值的绝对档位；复刻时先保存 `oldpercent` 对应关系。[状态机](../../../raw/current/scripts/components/wx78_shield.lua#L114-L124)
- **边界：破盾转场的 11 帧窗口**：脱挂任务是本地实体上的普通 `DoTaskInTime`（非静态任务）；转场期间仍随 WX-78 移动，之后固定在世界坐标，破盾瞬间的位移会带出半截动画。[脱挂](../../../raw/current/scripts/fx.lua#L4181-L4185)
- **边界：动画是否循环未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。六名循环配置（含 nofaced 与注释音效）、染色 fn 与血条同步注释、破盾延时脱挂、组件状态机六分支／周期重播／读档重放、模块依赖声明逐条核验；.po 名称键定向核验无结果。未覆盖：`wx78_shield_fx` 动画资源内容、`wx78_possessedbody` 宿主的组件接线条目、转场期间宿主移除的表现归属。
- 游戏运行：未运行；护盾亮灭节奏与破盾脱挂表现未实测。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
