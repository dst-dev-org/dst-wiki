---
title: "糖霜溅斑（icing_splat_fx）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "姜饼黏液弹（warg_gooicing）的糖霜溅斑：DoSplatFx 在黏液或目标位置落斑，命中可钉目标时经 pinnable:Stick 按 icing 专属列表接入四段量级变体。"
aliases: ["icing_splat_fx", "icing_splash_fx_full", "icing_splash_fx_med", "icing_splash_fx_low", "icing_splash_fx_melted", "糖霜溅斑", "糖霜特效", "姜饼溅斑", "姜饼黏液特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "prefabs/warg_gooicing.lua", "components/pinnable.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 糖霜溅斑（icing_splat_fx）

## 快速判断

- 职责与适用：姜饼黏液弹（warg_gooicing，冬季盛宴姜饼座狼内容）的糖霜落斑与量级溅斑，五注册分两组资源（`warg_gingerbread_splat`／`warg_gingerbread_splash`）。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L163-L191)
- 优先使用：查"糖霜溅斑／姜饼黏液"选本页；黏液弹的钉住与伤害读 warg_gooicing。
- 重要副作用：无音效、无 fn——糖霜声（gingerbread_vargr/splat）由 DoSplatFx 在**传入对象**的 SoundEmitter 播放：未钉住的可钉目标传目标（声在目标上），其余路径传弹体。[DoSplatFx](../../../raw/current/scripts/prefabs/warg_gooicing.lua#L25-L29)
- 修改边界：落斑 y=0 地面高度；pinnable 路径以 **AddChild 挂到被钉实体**；动画是否重复未核验。[落斑](../../../raw/current/scripts/prefabs/warg_gooicing.lua#L25-L29)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `icing_splat_fx` | 通用表条目 → 加载器代理＋一帧后本地实体，无 fn | bank／build=`warg_gingerbread_splat`，anim=`idle`。[条目](../../../raw/current/scripts/fx.lua#L163-L167) |
| `icing_splash_fx_full`／`_med`／`_low`／`_melted` | 同上协议 | bank／build=`warg_gingerbread_splash`，anim 逐名 full／med／low／melted；warg_gooicing 以专属列表 `splashfxlist = {full, med, low, melted}` 传给 pinnable。[四变体](../../../raw/current/scripts/fx.lua#L169-L191)、[列表](../../../raw/current/scripts/prefabs/warg_gooicing.lua#L17-L22) |

五注册在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，两组资源、无附加字段 | 同族同协议 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L163-L191) |
| 生成者：黏液弹命中 | `warg_gooicing` 的 `doprojectilehit`：可钉目标（有 pinnable 且未死）→ `DoSplatFx` 在黏液或目标位置落斑并 `pinnable:Stick("goo_icing", splashfxlist)`；非可钉目标 → `GetAttacked(WARG_GOO_DAMAGE)` 后同样 `DoSplatFx` | 服务端；两条命中分支都生成落斑 | [钉住分支](../../../raw/current/scripts/prefabs/warg_gooicing.lua#L34-L45)、[DoSplatFx](../../../raw/current/scripts/prefabs/warg_gooicing.lua#L25-L29) |
| 量级接入 | `pinnable:Stick` 把 icing 专属 `splashfxlist` 交给被钉实体，后续 SpawnShatterFX 按剩余比例在该列表取段 | 服务端；列表顺序 full→med→low→melted 为本族实际顺序 | [Stick](../../../raw/current/scripts/prefabs/warg_gooicing.lua#L41-L41)、[取段算法](../../../raw/current/scripts/components/pinnable.lua#L81-L89) |
| 依赖声明 | warg_gooicing 的 projectile_prefabs 声明全部五注册 | 仅声明 | [声明](../../../raw/current/scripts/prefabs/warg_gooicing.lua#L8-L15) |
| 资源 | 两组 bank：`warg_gingerbread_splat`／`warg_gingerbread_splash` | 资源命名不证明外观或玩法；动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L163-L191) |

## 典型接入

### 复用糖霜溅斑

- 前提与选择：落斑直接 Spawn（y=0 地面高度）；量级溅斑要配 pinnable：把本族列表传给 `pinnable:Stick`（不要沿用唾沫默认列表）。
- 生效与撤销：落斑 animover 自删；pinnable 路径 AddChild 到被钉实体、随其移动。
- 原版依据：[Stick 传表](../../../raw/current/scripts/prefabs/warg_gooicing.lua#L41-L41)。

```lua
local fx = SpawnPrefab("icing_splat_fx")
fx.Transform:SetPosition(x, 0, z)   -- 落斑固定地面高度；糖霜声由 DoSplatFx 的传入对象播放
```

- 调整与验证：量级由被钉实体剩余比例决定，调用侧无法指定具体变体；复刻声音时按分支选对象——未钉目标传目标、其余传弹体。

## 常见陷阱与限制

- **边界：列表顺序是本族事实**：icing 传给 pinnable 的顺序为 full／med／low／melted（[列表](../../../raw/current/scripts/prefabs/warg_gooicing.lua#L17-L22)），与唾沫默认列表（melted／low／med／full）不同；取段公式相同但量级映射随顺序变化，不能互套。[对照](../../../raw/current/scripts/prefabs/warg_gooicing.lua#L17-L22)
- **误区：糖霜声固定在弹体**：`DoSplatFx` 用**传入对象**的 SoundEmitter——未钉住的可钉目标分支传目标（声在目标上），已钉住与伤害分支传弹体（声在弹上）；分支不同播放对象不同。[DoSplatFx](../../../raw/current/scripts/prefabs/warg_gooicing.lua#L25-L29)
- **边界：两条命中分支都落斑**：钉住分支落点在"已钉则黏液、否则目标"，伤害分支落在黏液本体；复刻时按分支选位。[分支](../../../raw/current/scripts/prefabs/warg_gooicing.lua#L34-L45)
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。五注册配置、加载器协议、warg_gooicing 双命中分支（DoSplatFx 落斑、Stick 传 icing 列表、非可钉走伤害）、列表顺序核验。前轮"无生成点"结论系检索截断所致，本轮回填纠正。未覆盖：`warg_gingerbread_*` 动画资源内容、晚加入时序。
- 游戏运行：未运行；糖霜视觉与钉住表现未实测。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
