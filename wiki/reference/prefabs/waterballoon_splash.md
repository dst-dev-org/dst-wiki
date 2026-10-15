---
title: "水球溅落（waterballoon_splash）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "水球家族的水花表现：水球投射物命中（伴随加湿）与水球堆建成（建造者沾湿 20 点）两条生成路径；溅落本身纯表现无 fn。"
aliases: ["waterballoon_splash", "水球溅落", "水球水花", "水球命中表现"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "prefabs/waterprojectiles.lua", "prefabs/balloons_empty.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 水球溅落（waterballoon_splash）

## 快速判断

- 职责与适用：水球家族共用的水花表现，单注册；两条生成路径（投射物命中／水球堆建成），加湿与建造收益在各自调用方。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L112-L117)
- 优先使用：查"水球命中／水球堆建成的水花"选本页；加湿协议选 [[reference/components/wateryprotection|wateryprotection]]。
- 重要副作用：无；配音效 `dontstarve/creatures/pengull/splash`，**无 fn**（不设 FinalOffset，与同批 ink／bile 不同）。[条目](../../../raw/current/scripts/fx.lua#L112-L117)
- 修改边界：两条路径都是一次性落点（不父挂）；动画是否重复未核验。[投射物落点](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L102-L102)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `waterballoon_splash` | 通用表条目 → 加载器代理＋一帧后本地实体，无 fn | bank／build=`waterballoon`，anim=`used`，音效企鹅跳水声。[条目](../../../raw/current/scripts/fx.lua#L112-L117) |

注册名在简中 .po 无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，无附加字段 | 单注册 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L112-L117) |
| 生成者：水球弹命中 | `OnHitWater`：投射物位置落点生成，随后 `wateryprotection:SpreadProtection` 加湿并移除投射物 | 服务端；加湿在溅落之后 | [OnHitWater](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L101-L105) |
| 生成者：水球堆建成 | `balloons_empty` 的 `onbuilt`：建造完成在堆位置生成，并对建造者 `DoDeltaMoistureToEntity(builder, 20)` | 服务端；建造者被沾湿 20 点 | [onbuilt](../../../raw/current/scripts/prefabs/balloons_empty.lua#L57-L60) |
| 依赖声明 | waterprojectiles 的 waterballoon_prefabs 与 balloons_empty prefabs 各自声明本名 | 仅声明 | [投射物声明](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L17-L21) |
| 资源 | bank／build=`waterballoon`（与水球投射物同资源族） | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L112-L117) |

## 典型接入

### 复用溅落表现

- 前提与选择：只要水花可任意位置 Spawn；加湿需另行调 `wateryprotection:SpreadProtection` 或建造成就路径。
- 生效与撤销：落点一次性动画，animover 自删。
- 原版依据：[OnHitWater](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L101-L105)。

```lua
local fx = SpawnPrefab("waterballoon_splash")
fx.Transform:SetPosition(x, y, z)   -- 落点；animover 自删
```

- 调整与验证：两条原版路径的效果差异（加湿 vs 建造者沾湿）都在调用方，fx 不携带任何组件。

## 常见陷阱与限制

- **边界：同族三溅落配置不同**：ink／bile 有 `FinalOffset 1`，本条目无 fn——显示层级与兄弟注册不同，不要按同文件推断一致。[对照](../../../raw/current/scripts/fx.lua#L88-L117)
- **边界：水球堆路径会湿建造者**：onbuilt 的 20 点湿度施加给 builder（[DoDeltaMoisture](../../../raw/current/scripts/prefabs/balloons_empty.lua#L59-L59)），溅落本身无关。
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。单注册配置、加载器协议、OnHitWater 与 onbuilt 两条生成路径、依赖声明、.po 名称键核验完毕。未覆盖：`waterballoon` 动画资源内容、晚加入时序。
- 游戏运行：未运行；溅落视觉未实测。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
