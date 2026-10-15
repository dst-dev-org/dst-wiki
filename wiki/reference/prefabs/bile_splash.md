---
title: "胆汁弹溅落（bile_splash）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "胆汁弹投射物命中时的溅落表现：命中点落点、按海陆生成胆汁洼、半径 2 内 AOE 伤害由投射物结算；溅落本身纯表现。"
aliases: ["bile_splash", "胆汁溅落", "胆汁弹特效", "胆汁命中表现"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "prefabs/waterprojectiles.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 胆汁弹溅落（bile_splash）

## 快速判断

- 职责与适用：水球投射物家族的胆汁弹（变异鸟弹）命中时的溅落表现，单注册；纯表现，洼地与 AOE 伤害在投射物 onhit。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L96-L102)
- 优先使用：查"胆汁弹命中／胆汁溅"选本页；伤害与洼地结算读 OnHitBile。
- 重要副作用：无；配音效 `moonstorm/creatures/mutated_robin/bile_shoot_splash`、`FinalOffset 1`。[条目](../../../raw/current/scripts/fx.lua#L96-L102)
- 修改边界：命中点一次性落点（不父挂）；动画是否重复未核验。[落点](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L60-L60)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `bile_splash` | 通用表条目 → 加载器代理＋一帧后本地实体，fn=FinalOffset1 | bank／build=`bird_bileshoot`，anim=`splash`，音效胆汁弹专属。[条目](../../../raw/current/scripts/fx.lua#L96-L102) |

注册名在简中 .po 无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，无附加字段 | 单注册 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L96-L102) |
| 生成者：胆汁弹命中 | `OnHitBile`：投射物当前位置落点生成；按 `IsOnOcean()` 生成 `bile_puddle_water`／`bile_puddle_land`；随后半径 2 内 combat 实体（排 INLIMBO／playerghost）`GetAttacked`，伤害 `TUNING.MUTANT_BIRD_SPLASH_DAMAGE` | 服务端；AOE 由投射物结算，溅落本身无伤害 | [OnHitBile](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L58-L74) |
| 依赖声明 | waterprojectiles 的 bile_prefabs 声明本名 | 仅声明；生成只在胆汁弹命中 | [声明](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L53-L56) |
| 资源 | bank／build=`bird_bileshoot`（变异鸟射击资源族） | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L96-L102) |

## 典型接入

### 复用溅落表现

- 前提与选择：只要溅落表现可任意位置 Spawn；完整胆汁链路（洼地＋AOE）在投射物 onhit 内，复用时应调 OnHitBile 同构逻辑而非只生成表现。
- 生效与撤销：落点一次性动画，animover 自删。
- 原版依据：[OnHitBile](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L58-L74)。

```lua
local fx = SpawnPrefab("bile_splash")
fx.Transform:SetPosition(x, y, z)   -- 命中点落点；animover 自删
```

- 调整与验证：AOE 排除表为 INLIMBO／playerghost 标签，含玩家——复刻时保留过滤条件。

## 常见陷阱与限制

- **误区：溅落带伤害**：伤害来自 OnHitBile 的 `GetAttacked` 遍历；`bile_splash` 实体无 combat／propagator。[AOE](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L67-L72)
- **边界：洼地按海陆分流**：`IsOnOcean()` 决定水洼／陆洼 prefab；溅落位置与洼地位置同为命中点。[分流](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L61-L65)
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。单注册配置、加载器协议、OnHitBile 全链（洼地分流与 AOE 过滤）、依赖声明、.po 名称键核验完毕。未覆盖：`bird_bileshoot` 动画资源内容、晚加入时序。
- 游戏运行：未运行；溅落视觉与 AOE 范围未实测。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
