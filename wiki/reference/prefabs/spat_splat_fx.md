---
title: "唾沫溅斑（spat_splat_fx）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "唾沫怪主题的五注册溅斑：spat 弹命中直接生成 idle 落斑（y=0），pinnable 的默认碎裂列表覆盖 full／med／low／melted 四段量级（AddChild 到被钉实体）。"
aliases: ["spat_splat_fx", "spat_splash_fx_full", "spat_splash_fx_med", "spat_splash_fx_low", "spat_splash_fx_melted", "唾沫溅斑", "唾沫特效", "碎裂唾沫"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "components/pinnable.lua", "prefabs/spat.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 唾沫溅斑（spat_splat_fx）

## 快速判断

- 职责与适用：唾沫怪（spat）主题的落斑与溅斑表现，五注册分两组资源（`spat_splat`／`spat_splash`）；生成路径有二——spat 弹命中直接生成落斑、[[reference/components/pinnable|pinnable]] 的默认碎裂列表覆盖四个 splash 变体。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L133-L161)
- 优先使用：查"唾沫溅斑／被钉实体碎裂"选本页；pinnable 的钉住机制读其组件页。
- 重要副作用：无音效、无 fn——五注册都是纯资源条目。[条目](../../../raw/current/scripts/fx.lua#L133-L161)
- 修改边界：pinnable 路径以 **AddChild 挂到被钉实体**（随其移动）；动画是否重复未核验。[生成](../../../raw/current/scripts/components/pinnable.lua#L81-L89)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `spat_splat_fx` | 通用表条目 → 加载器代理＋一帧后本地实体，无 fn | bank／build=`spat_splat`，anim=`idle`；spat 弹命中时落点生成。[条目](../../../raw/current/scripts/fx.lua#L133-L137) |
| `spat_splash_fx_full`／`_med`／`_low`／`_melted` | 同上协议 | bank／build=`spat_splash`，anim 逐名 full／med／low／melted；构成 pinnable 默认碎裂列表（melted／low／med／full 顺序）。[四变体](../../../raw/current/scripts/fx.lua#L139-L161)、[默认列表](../../../raw/current/scripts/components/pinnable.lua#L11-L17) |

五注册在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，两组资源、无附加字段 | 同族同协议 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L133-L161) |
| 生成者：spat 弹命中 | `doprojectilehit`：播 spit_hit 声后在**弹体位置 y=0** 落点生成 `spat_splat_fx` | 服务端；attacker 失效会被置 nil 后续处理 | [命中](../../../raw/current/scripts/prefabs/spat.lua#L295-L298) |
| 生成者：pinnable 碎裂 | `SpawnShatterFX(ratio)`：按被钉物剩余比例 `clamp(floor(#list*ratio)+1)` 从默认列表选名，`SpawnPrefab` 后 **AddChild 到被钉实体**；入参 ratio 会立即被 `RemainingRatio()` 覆盖，调用侧不可控量级 | 服务端；`splashfxlist` 可被使用者覆盖换列表 | [算法](../../../raw/current/scripts/components/pinnable.lua#L81-L89)、[默认列表](../../../raw/current/scripts/components/pinnable.lua#L11-L17) |
| 依赖声明 | spat 弹投射物声明 full／med 两名 | 仅声明；快照内 spat.lua 无对应生成调用 | [声明](../../../raw/current/scripts/prefabs/spat.lua#L26-L28) |
| 资源 | 两组 bank：`spat_splat`（落斑）／`spat_splash`（量级溅斑） | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L133-L161) |

## 典型接入

### 复用碎裂溅斑

- 前提与选择：给自定义可钉实体接碎裂表现时，给 pinnable 传 `splashfxlist` 或沿用默认列表；量级由被钉实体剩余比例决定，**调用 `SpawnShatterFX` 时传入的 ratio 会被内部 `RemainingRatio()` 覆盖**，不能从外部控制量级。
- 生效与撤销：表现 AddChild 到被钉实体、随其移动、animover 自删；pinnable 侧无需清理。
- 原版依据：[SpawnShatterFX](../../../raw/current/scripts/components/pinnable.lua#L81-L89)。

```lua
-- 被钉实体（pinnable 组件）受击后由原版内部调用；量级取自 RemainingRatio
inst.components.pinnable:SpawnShatterFX(ratio)
```

- 调整与验证：`AddChild` 意味着表现坐标系是被钉实体的子级；直接 Spawn 落点则没有该父子关系。

## 常见陷阱与限制

- **边界：splat 落点固定地面高度**：spat 弹命中把落斑放在 `(x, 0, z)`——空中命中也压到地面；复刻时保留该 y 处理或自行调整。[落点](../../../raw/current/scripts/prefabs/spat.lua#L298-L298)
- **边界：默认列表顺序不是量级顺序**：pinnable 列表顺序为 melted／low／med／full，量级由"剩余比例×列表长度"取段决定，不是从 low 到 full 顺序播放。[算法](../../../raw/current/scripts/components/pinnable.lua#L83-L86)
- **边界：spat 弹的依赖声明**：spat.lua 的 prefabs 表声明部分注册（full／med）；实际生成经 doprojectilehit 只出落斑本名——声明与生成路径不同。[对照](../../../raw/current/scripts/prefabs/spat.lua#L26-L28)
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。五注册配置、加载器协议、spat 弹命中落斑、pinnable 默认列表与碎裂算法（含 ratio 覆盖行为）、spat 弹依赖声明核验。前轮"splat 本名无生成点"系检索截断（head 截断 spat.lua 后段）所致，本轮回填纠正。未覆盖：`spat_splat`／`spat_splash` 动画资源内容、pinnable 使用者的完整列表（shatterfx.md 已记辨析）、晚加入时序。
- 游戏运行：未运行；溅斑视觉与 AddChild 表现未实测。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
