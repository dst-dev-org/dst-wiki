---
title: "墨水弹溅落（ink_splash）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "墨水弹投射物命中时的溅落表现：命中点落点、按海陆生成墨水洼、对 inkable 实体上墨；溅落本身纯表现，上墨与洼地在投射物链路。"
aliases: ["ink_splash", "墨水溅落", "墨水弹特效", "墨水命中表现"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "prefabs/waterprojectiles.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 墨水弹溅落（ink_splash）

## 快速判断

- 职责与适用：水球投射物家族的墨水弹（喷墨鸟弹）命中时的溅落表现，单注册；纯表现，上墨与洼地在投射物 onhit。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L88-L94)
- 优先使用：查"墨水弹命中／墨水溅"选本页；墨水洼与上墨范围选 [[reference/components/inkable|inkable]]。
- 重要副作用：无；配音效 `turnoftides/common/together/water/splash/bird`、`FinalOffset 1`。[条目](../../../raw/current/scripts/fx.lua#L88-L94)
- 修改边界：命中点一次性落点（不父挂）；动画是否重复未核验。[落点](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L79-L79)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `ink_splash` | 通用表条目 → 加载器代理＋一帧后本地实体，fn=FinalOffset1 | bank／build=`squid_watershoot`，anim=`splash`，音效 water/splash/bird。[条目](../../../raw/current/scripts/fx.lua#L88-L94) |

注册名在简中 .po 无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，无附加字段 | 单注册 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L88-L94) |
| 生成者：墨水弹命中 | `OnHitInk`：投射物当前位置落点生成；随后按 `IsOnOcean()` 生成 `ink_puddle_water`／`ink_puddle_land`，并对半径 1 内 `inkable` 实体调 `Ink()`；无伤害结算 | 服务端；投射物移除前调用 | [OnHitInk](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L77-L93) |
| 依赖声明 | waterprojectiles 的 ink_prefabs 声明本名 | 仅声明；生成只在墨水弹命中 | [声明](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L28-L30) |
| 资源 | bank／build=`squid_watershoot`（喷墨鸟射击资源族） | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L88-L94) |

## 典型接入

### 复用溅落表现

- 前提与选择：只要溅落表现可在任意位置 Spawn；要完整上墨链路应走投射物 onhit（洼地与 inkable 遍历都在其内）。
- 生效与撤销：落点一次性动画，animover 自删。
- 原版依据：[OnHitInk](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L77-L93)。

```lua
local fx = SpawnPrefab("ink_splash")
fx.Transform:SetPosition(x, y, z)   -- 命中点落点；animover 自删
```

- 调整与验证：上墨与洼地不会随 fx 自动发生；复刻完整效果需同时调用 puddle 与 inkable 遍历。

## 常见陷阱与限制

- **边界：溅落不等于上墨**：`Ink()` 与墨水洼由 OnHitInk 遍历半径 1 的 `inkable` 实体实现，溅落实体本身无组件。[上墨](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L85-L90)
- **边界：无伤害结算**：墨水弹命中不调 GetAttacked（同文件胆汁弹有 AOE 伤害，不要套用）。[对照](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L67-L72)
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。单注册配置、加载器协议、OnHitInk 全链（洼地分流与上墨遍历）、依赖声明、.po 名称键核验完毕。未覆盖：`squid_watershoot` 动画资源内容、晚加入时序。
- 游戏运行：未运行；溅落视觉未实测。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
