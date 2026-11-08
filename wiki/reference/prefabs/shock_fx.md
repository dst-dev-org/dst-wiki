---
title: "电击附着动画（shock_fx 族）"
created: "2026-10-30"
updated: "2026-11-08"
type: prefab
summary: "通用表六注册的电击附着表现：玩家电击态按变形形态四选一挂身跟随、WX78 电击模块反击中点弧光、月暴电花周期自附；纯表现，不结算电伤。"
aliases: ["shock_fx", "shock_arc_fx", "werebeaver_shock_fx", "weremoose_shock_fx", "weregoose_shock_fx", "moonstorm_spark_shock_fx", "电击附着", "电击动画", "电弧特效", "玩家电击动画", "海狸电击", "驼鹿电击", "鹅电击", "月暴电花电击"]
tags: [prefab, fx, network-sync]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/fx.lua", "fx.lua", "stategraphs/SGwilson.lua", "wx78_moduledefs.lua", "prefabs/moonstorm_spark.lua", "prefabs/electrocute_fx.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 电击附着动画（shock_fx 族）

## 快速判断

- 职责与适用：实体被电时的附着表现，共 6 注册共用 bank＝`shock_fx`；与 [[reference/prefabs/electrocute_fx|electrocute_fx]]（独立实体＋体型动画＋连锁）是两套注册两条路径。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L686-L730)
- 优先使用：查"被电的动画／电弧"选本页；连锁、体型动画与电伤结算链路选 [[reference/prefabs/electrocute_fx|electrocute_fx]]。
- 重要副作用：无；全族 `FinalOffset 1`，四名配音效 `dontstarve_DLC001/common/shocked`。[条目](../../../raw/current/scripts/fx.lua#L686-L730)
- 修改边界：玩家用例是**符号跟随**（代理挂玩家并跟随 `swap_shock_fx`）；电花用例父挂电花本体；动画是否循环未核验。[玩家挂接](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3419-L3430)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shock_fx` | 通用表条目 → 加载器代理＋一帧后本地实体，fn=FinalOffset1 | anim=`shock`，音效 shocked，autorotate，默认四向。[条目](../../../raw/current/scripts/fx.lua#L686-L693) |
| `werebeaver_shock_fx` | 同上协议 | anim=`werebeaver_shock`，四向＋autorotate。[条目](../../../raw/current/scripts/fx.lua#L695-L702) |
| `weremoose_shock_fx` | 同上协议 | anim=`weremoose_shock`，**八向**＋autorotate。[条目](../../../raw/current/scripts/fx.lua#L704-L712) |
| `weregoose_shock_fx` | 同上协议 | anim=`weregoose_shock`，八向＋autorotate；作为变形玩家**兜底**（鹅判定被注释，见陷阱）。[条目](../../../raw/current/scripts/fx.lua#L722-L730)、[选择链](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3419-L3424) |
| `shock_arc_fx` | 同上协议 | anim=`arc`，八向；**无音效、无 autorotate**，朝向靠生成者 `ForceFacePoint`。[条目](../../../raw/current/scripts/fx.lua#L714-L720) |
| `moonstorm_spark_shock_fx` | 同上协议 | 复用 anim=`weremoose_shock`，音效换 `moonstorm/common/moonstorm/spark_attack`，八向＋autorotate。[条目](../../../raw/current/scripts/fx.lua#L2436-L2444) |

定向检索简中 .po 未发现这 6 个注册名的 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目；差异仅 anim／音效／朝向位与 autorotate | 全族同协议 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L686-L730) |
| 生成者：玩家电击态 | SGwilson `electrocute` 态按形态标签四选一（非 wereplayer→`shock_fx`；beaver→海狸；weremoose→驼鹿；其余→鹅兜底），骑乘时代理 `SetSixFaced`，再 `SetParent` 挂玩家＋`AddFollower`＋`FollowSymbol` 跟随 `swap_shock_fx` | 服务端 onenter；符号跟随用例，本地表现对符号动画的持续贴合未核验 | [状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3408-L3430)、[[reference/prefabs/electrocute_fx|electrocute_fx]]（同链路的连锁层） |
| 生成者：WX78 电击模块反击 | `taser_onblockedorattacked`：WX78 受击／格挡时先经 `combat:GetAttacked` 结算反制电伤，再在 WX78 与攻击者**中点**落点生成 `shock_arc_fx` 并 `ForceFacePoint` 攻击者 | 服务端；表现与伤害结算分离 | [反击](../../../raw/current/scripts/wx78_moduledefs.lua#L641-L663) |
| 生成者：electrocute_fx 连锁 | `DoFork` 连锁电击时也生成 `shock_arc_fx` 居中指向新目标并推 `electrocute` 事件 | 服务端；与电击模块反击弧光共用同一注册 | [DoFork](../../../raw/current/scripts/prefabs/electrocute_fx.lua#L29-L92)、[[reference/prefabs/electrocute_fx|electrocute_fx]] |
| 生成者：月暴电花 | 电花（Moongleam／月熠）周期任务 `SpawnChild` 父挂自附 `moonstorm_spark_shock_fx`；初始 5～15 秒后首发，此后经 `dospark1` 衔接 | 服务端；父挂用例 | [电花](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L48-L58)、[译名](../../../raw/current/scripts/languages/chinese_s.po#L345095-L345097) |
| 资源 | bank／build=`shock_fx`，五条带音效、`shock_arc_fx` 无 | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L686-L730) |

## 典型接入

### 给实体加被电表现

- 前提与选择：玩家形态复刻按标签四选一并父挂＋符号跟随；普通实体用落点式 `SpawnPrefab("shock_fx")` 即可；弧光用 `shock_arc_fx` 并自行 `ForceFacePoint`。
- 生效与撤销：一次性附着动画，animover 自删；父挂式随宿主移动。
- 原版依据：[电击模块反击中点弧光](../../../raw/current/scripts/wx78_moduledefs.lua#L658-L663)。

```lua
local arc = SpawnPrefab("shock_arc_fx")
arc.Transform:SetPosition((x1 + x2) / 2, y, (z1 + z2) / 2)
arc:ForceFacePoint(tx, ty, tz)
```

- 调整与验证：四向／八向在数据里固定，骑乘六向是玩家态对**代理**的运行时覆盖（[覆盖](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3425-L3427)）；需要自定义形态动画时新增表项而非改运行时。

## 常见陷阱与限制

- **误区：本族就是 electrocute_fx**：两者共用 bank 名 `shock_fx` 但注册与路径不同——electrocute_fx 是独立实体、`SetFxTarget` 挂目标按体型播动画并负责连锁（[[reference/prefabs/electrocute_fx|其页 L42 已记本族玩家路径]]）；本族走通用工厂协议。[两套](../../../raw/current/scripts/fx.lua#L686-L693)
- **边界：weregoose_shock_fx 是兜底而非"鹅专属"**：选择链里 `HasTag("weregoose")` 被注释，weremoose 之后任何未匹配的变形玩家都会落到鹅动画。[选择链](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3419-L3424)
- **边界：shock_arc_fx 无音效且不自转**：与四名带音效＋autorotate 的成员不同；朝向完全由生成者决定（反击路径用 ForceFacePoint）。[条目](../../../raw/current/scripts/fx.lua#L714-L720)
- **边界：表现不等于电伤**：反击的伤害由 `GetAttacked` 在生成弧光**之前**结算（[结算](../../../raw/current/scripts/wx78_moduledefs.lua#L652-L656)）；玩家电击的伤害走 electrocute 事件链，本族只播动画。
- **边界：动画是否循环未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。六条目配置、加载器协议、玩家电击态选择链与挂接、反击结算与弧光落点、月暴电花周期自附逐条核验；.po 名称键定向核验无结果（月熠为消费者译名）。未覆盖：`swap_shock_fx` 符号所在动画资源、骑乘六向在本地表现上的实际生效、变形形态切换时序。
- 游戏运行：未运行；附着动画与符号跟随表现未实测。

返回 [[reference/prefabs/fx-status-index|增益、受击、护盾与状态反馈]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-03）。
