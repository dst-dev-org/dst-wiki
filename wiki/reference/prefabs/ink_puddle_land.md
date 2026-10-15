---
title: "墨水与胆汁洼地（ink_puddle、bile_puddle 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "墨水弹与胆汁弹命中落点的贴地水洼动画：按投射物是否在海面选 _water／_land 两档，四注册共用 squid_puddle bank；洼地纯表现，上墨与 AOE 伤害在投射物 onhit。"
aliases: ["ink_puddle_land", "ink_puddle_water", "bile_puddle_land", "bile_puddle_water", "墨水洼", "胆汁洼", "墨水水洼", "胆汁水洼", "命中洼地"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/waterprojectiles.lua", "prefabs/squid.lua", "prefabs/birds_mutant.lua", "prefabs/oceanfishableflotsam.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 墨水与胆汁洼地（ink_puddle、bile_puddle 族）

## 快速判断

- 职责与适用：`fx.lua` 通用表的贴地水洼动画，四注册——墨水弹命中洼地 `ink_puddle_land`／`ink_puddle_water`、胆汁弹命中洼地 `bile_puddle_land`／`bile_puddle_water`；纯表现实体，无玩法作用。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L2076-L2102)
- 优先使用：查"墨水／胆汁命中后的地面水洼"选本页；溅起动画选 [[reference/prefabs/ink_splash|墨水弹溅落]]／[[reference/prefabs/bile_splash|胆汁弹溅落]]，上墨与 AOE 结算也在其投射物链路。
- 重要副作用：无玩法作用；`fn` 走 `GroundOrientation`——`ANIM_ORIENTATION.OnGround` 贴地并置于 `LAYER_BACKGROUND` 层。[helper](../../../raw/current/scripts/fx.lua#L24-L27)
- 修改边界：服务端生成、命中点一次性落点（不父挂）；海陆选名由生成方按 `IsOnOcean()` 决定，条目本身无条件逻辑。[ink 分流](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L80-L84)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `ink_puddle_land` | 通用表条目 → 加载器代理＋一帧后本地实体，fn=GroundOrientation | bank=`squid_puddle`、build=`squid_puddle`、anim=`puddle_dry`，无声、无染色。[条目](../../../raw/current/scripts/fx.lua#L2076-L2080) |
| `ink_puddle_water` | 同上 | 同资源，anim=`puddle_wet`。[条目](../../../raw/current/scripts/fx.lua#L2083-L2087) |
| `bile_puddle_land` | 同上 | bank=`squid_puddle` 但 build=`bird_puddle`、anim=`puddle_dry`。[条目](../../../raw/current/scripts/fx.lua#L2091-L2095) |
| `bile_puddle_water` | 同上 | 同上，anim=`puddle_wet`。[条目](../../../raw/current/scripts/fx.lua#L2098-L2102) |

四个注册名在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

选名条件：海陆两档由生成方按投射物命中点 `inst:IsOnOcean()` 二选一（ink 与 bile 各自同构），不在条目内处理，也无动态拼名。[bile 分流](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L61-L65)、[ink 分流](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L80-L84)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，无附加字段 | 四注册同构，仅 build／anim 差异 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L2076-L2102) |
| 生成者：墨水弹 onhit | `OnHitInk`：投射物落点先溅落（`ink_splash`）、再按海陆生成本族洼地；随后半径 1 内 `inkable` 实体 `Ink()`；投射物 `Remove` | 服务端；溅落与洼地同帧同点各自生成 | [OnHitInk](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L77-L93)、[[reference/prefabs/ink_splash]] |
| 生成者：胆汁弹 onhit | `OnHitBile`：溅落＋按海陆生成本族洼地；随后半径 2 内 combat 实体（排 INLIMBO／playerghost）受 `TUNING.MUTANT_BIRD_SPLASH_DAMAGE` | 服务端；伤害由投射物结算，洼地不参与 | [OnHitBile](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L58-L74)、[[reference/prefabs/bile_splash]] |
| 上游：投射物 | `inksplat`／`bilesplat` 复杂抛射物由鱿鱼与变异鸟的 `LaunchProjectile` 发射（注释标明从 stategraph 调用），命中回调触发洼地 | 服务端发射与命中 | [squid](../../../raw/current/scripts/prefabs/squid.lua#L52-L56)、[birds_mutant](../../../raw/current/scripts/prefabs/birds_mutant.lua#L30-L33) |
| 依赖声明 | waterprojectiles 的 `ink_prefabs`／`bile_prefabs` 声明本族四名 | 仅声明；生成只在弹体 onhit | [ink](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L28-L33)、[bile](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L51-L56) |
| 资源 | bank 统一 `squid_puddle`；墨水 build=`squid_puddle`、胆汁 build=`bird_puddle` | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L2076-L2102) |

## 典型接入

### 在任意点位复现命中洼地

- 前提与选择：洼地表现可在任意位置 `SpawnPrefab`；选名按目标点是否海洋。原版以投射物实体 `inst:IsOnOcean()` 判，任意点位可用 `Map:IsOceanAtPoint`（原版 `OnLanded` 同款用法）。
- 生效与撤销：落点一次性动画，animover 自删；无玩法作用，复刻完整命中效果需另做 inkable 遍历或 AOE（见上文生成者行）。
- 原版依据：[OnHitInk 分流](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L80-L84)、[IsOceanAtPoint 用例](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L301-L306)。

```lua
local on_ocean = TheWorld.Map:IsOceanAtPoint(x, y, z, false)
local fx = SpawnPrefab(on_ocean and "ink_puddle_water" or "ink_puddle_land")
fx.Transform:SetPosition(x, y, z)   -- 贴地洼底动画；animover 自删
```

- 调整与验证：`x, y, z` 为世界坐标；胆汁族把注册名换成 `bile_puddle_*` 即可。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：溅落与洼地是两个实体**：同一 onhit 内先 `SpawnPrefab("ink_splash")`（溅起）再生成 `ink_puddle_*`（地面洼底），不是同一实体的两态，生成溅落不会自带洼地。[OnHitInk](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L79-L84)
- **边界：洼地不是持续实体**：通用表条目只有动画表现；本族四名连音效字段都没有，地面不留玩法残留。同为"puddle"的 `ice_puddle` 是带独立文件的实体 prefab（raw/current/scripts/prefabs/ice_puddle.lua），与本族机制不同。
- **边界：胆汁洼 bank 与墨水同族**：四名 bank 都是 `squid_puddle`，区分靠 build（`squid_puddle`／`bird_puddle`）；按 bank 猜资源归属会错。[条目](../../../raw/current/scripts/fx.lua#L2091-L2095)
- **边界：动画是否重复未核验**：工厂只 `PlayAnimation` 一次，是否循环由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-31，steam-buildid:26335394。四条目配置与 `GroundOrientation` helper、`OnHitInk`／`OnHitBile` 全链、上游 `LaunchProjectile`（squid／birds_mutant）、依赖声明、`.po` 名称键核验完毕；对 `ink_puddle|bile_puddle` 在 scripts 全域 `rg -n`（根级、prefabs、components、stategraphs，含拼名模式）仅命中 fx.lua 条目与 waterprojectiles.lua，无其他生成点。未覆盖：动画资源内容与循环性、晚加入时序。
- 游戏运行：未运行；洼地视觉与分层未实测。
- 结构检查：`validate_page.py` 通过（0 issue；78 行、8220 字节，低于 200 行／16 KiB 标准档）；`validate_source_links.py --strict` 通过（errors 0、warnings 0；22 处源码引用，初版 8 处锚点起于 fx.lua 条目 `{` 行，已按 name 行起点约定修正清零）。正文 Lua 示例另存临时文件经 `parser_probe --syntax-only` 通过（只查 Lua 5.1 语法，未执行）。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]、[[reference/prefabs/index|Prefab 索引]]。
