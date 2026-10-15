---
title: "冲滩浮渣伴生水洼与拾取碎裂（flotsam_puddle 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "海面漂浮物钓获冲滩后父挂在陆上浮渣物品下的贴地水洼（随机朝向缩放），玩家拾取浮渣时落点播放碎裂动画并掉落战利品；两注册纯表现。"
aliases: ["flotsam_puddle", "flotsam_break", "浮渣水洼", "冲滩水洼", "浮渣碎裂", "漂浮物冲滩"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/oceanfishableflotsam.lua", "components/flotsamgenerator.lua", "components/oceanfishable.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 冲滩浮渣伴生水洼与拾取碎裂（flotsam_puddle 族）

## 快速判断

- 职责与适用：`fx.lua` 通用表的两注册表现——`flotsam_puddle` 陆上浮渣物品（`oceanfishableflotsam`）脚下的贴地水洼，父挂随物品；`flotsam_break` 拾取浮渣时在原位的碎裂动画。纯表现，战利品由 pickable 发放。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L2106-L2120)
- 优先使用：查"浮渣冲滩水洼／浮渣拾取碎裂"选本页；世界刷点与沉没计时读 [[reference/components/flotsamgenerator|flotsamgenerator]]。
- 重要副作用：无玩法作用；`flotsam_puddle` 是全族唯一带 `sound` 的注册（`dontstarve/creatures/monkey/poopsplat`），父挂语义见工厂页。[条目](../../../raw/current/scripts/fx.lua#L2106-L2114)
- 修改边界：服务端生成；水洼只在"抛射物落陆"路径出现（父挂式），碎裂为落点式；专用服务器无本地表现。[落陆分支](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L130-L136)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `flotsam_puddle` | 通用表条目 → 加载器代理＋一帧后本地实体；fn 匿名：`SetLayer(LAYER_WORLD_BACKGROUND)`＋`SetOrientation(ANIM_ORIENTATION.OnGround)`；生成者再父挂、缩放 1.15、随机旋转 | bank=`flotsam`、build=`flotsam`、anim=`puddle`，音效 `dontstarve/creatures/monkey/poopsplat`。[条目](../../../raw/current/scripts/fx.lua#L2106-L2114) |
| `flotsam_break` | 通用表条目，无 fn、无 sound——普通朝向碎裂动画 | bank/build=`flotsam`、anim=`break`。[条目](../../../raw/current/scripts/fx.lua#L2117-L2120) |

两个注册名在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

注意 `flotsam_puddle` 的 fn 不走 `GroundOrientation` helper：layer 用 `LAYER_WORLD_BACKGROUND` 而非 helper 的 `LAYER_BACKGROUND`。[对照 helper](../../../raw/current/scripts/fx.lua#L24-L27)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目 | 父挂式（puddle）与落点式（break）并存 | [[reference/mechanisms/generic-fx-factory]]；[父挂语义](../../../raw/current/scripts/prefabs/fx.lua#L25-L28) |
| 生成者：冲滩伴生 | `playlandfx`：`SpawnPrefab("flotsam_puddle")` 后 `entity:SetParent(浮渣物品)`、`SetScale(1.15)`、`SetRotation(random*360)`；由 `OnLand` 陆地分支在物品落地 2 帧后调用 | 服务端；水洼成为物品子实体、随物品移动 | [playlandfx](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L108-L114)、[OnLand](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L116-L138) |
| 生成者：拾取碎裂 | `OnPicked`（pickable `onpickedfn`，hauntable `OnHaunt` 复用）：掉落战利品后 `SpawnPrefab("flotsam_break")` 落点；返回 true 使物品本体不给玩家，`remove_when_picked` 移除物品 | 服务端；碎裂与战利品同点 | [OnPicked](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L187-L202)、[pickable 配置](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L336-L340) |
| 上游：抛射物链 | 水中浮渣被钓钩捕获时 `oceanfishable:MakeProjectile` 调 `makeprojectilefn`（加 complexprojectile、`SetOnHit(OnLand)`）；落陆转生成陆上形态并伴生水洼 | 服务端 | [MakeProjectile](../../../raw/current/scripts/components/oceanfishable.lua#L52-L53)、[OnMakeProjectile](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L140-L157) |
| 上游：世界刷点 | [[reference/components/flotsamgenerator]] 加权表含 `oceanfishableflotsam_water`（水中游动形态，Wiki 待建，源码 prefabs/oceanfishableflotsam.lua `waterfn`） | 服务端；沉没覆写 `overrideflotsamsinkfn` 见其"沉没与清理"节 | [刷点表](../../../raw/current/scripts/components/flotsamgenerator.lua#L19-L28)、[[reference/components/flotsamgenerator]] |
| 依赖宿主：陆上形态 | `oceanfishableflotsam`（可拾取实体，Wiki 待建）挂 pickable／inventoryitem／hauntable／lootdropper 等，是水洼的父实体与碎裂的触发宿主 | 服务端组件；水洼随其移除 | [landfn](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L308-L355) |
| 资源 | bank/build=`flotsam`（同水中／陆上浮渣实体共用） | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L2106-L2120) |

## 典型接入

### 给世界物品配伴生贴地水洼

- 前提与选择：需要"物品脚下贴地水洼、随物品存在"的表现时，照原版 `playlandfx` 父挂三步走；父挂信息经代理网络同步到各端。
- 生效与撤销：本地表现成为宿主子实体随宿主移动；宿主被移除后表现不复存在（子层级随父清理），条目自身也受 animover 自删约束。
- 原版依据：[playlandfx](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L108-L114)。

```lua
local fx = SpawnPrefab("flotsam_puddle")
fx.entity:SetParent(inst.entity)                 -- 父挂宿主，表现随宿主
fx.Transform:SetScale(1.15, 1.15, 1.15)
fx.Transform:SetRotation(math.random() * 360)
```

- 调整与验证：`inst` 为服务端宿主实体；旋转单位为度、缩放为倍率（原版 1.15）。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：水洼只在落陆路径出现**：抛射物落在海面时走水中分支（继续游动＋`splash` 水花，无水洼）；绞盘打捞（`OnSalvage` 直接生成陆上形态）也不播 `playlandfx`——陆上浮渣可以没有水洼。[水中分支](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L120-L128)、[OnSalvage](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L213-L217)
- **边界：`flotsam_break` 不是水洼**：名字不带 puddle、无 fn，普通朝向的碎裂动画；不要给它按 `OnGround` 贴地预期排层。[条目](../../../raw/current/scripts/fx.lua#L2117-L2120)
- **边界：拾取后玩家拿不到浮渣本体**：`OnPicked` 返回 true 且 `remove_when_picked` 生效，玩家只得战利品；物品移除后父挂水洼随之消失，碎裂动画独自落点自删。[OnPicked](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L200-L201)
- **边界：动画是否重复未核验**：工厂只 `PlayAnimation` 一次，是否循环由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-11-01，steam-buildid:26335394。两注册配置、`playlandfx`／`OnLand`／`OnPicked`／`OnSalvage`／`OnHaunt` 全链、抛射物转换（oceanfishable:MakeProjectile）、世界刷点表与沉没覆写、`.po` 名称键核验完毕；对 `flotsam_puddle|flotsam_break` 在 scripts 全域 `rg -n`（根级、prefabs、components、stategraphs，含拼名模式）仅命中 fx.lua 条目与 oceanfishableflotsam.lua 两处，无其他生成点。未覆盖：动画资源内容与循环性、晚加入时序。
- 游戏运行：未运行；父挂水洼的分层与随动表现未实测。
- 结构检查：`validate_page.py` 通过（0 issue；78 行、8526 字节，低于 200 行／16 KiB 标准档）；`validate_source_links.py --strict` 通过（errors 0、warnings 0；22 处源码引用，初版 7 处锚点起于结构行，已按 name 行／语句行起点约定修正清零）。正文 Lua 示例另存临时文件经 `parser_probe --syntax-only` 通过（只查 Lua 5.1 语法，未执行）。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]、[[reference/prefabs/index|Prefab 索引]]。
