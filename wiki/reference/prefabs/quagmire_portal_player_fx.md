---
title: "暴食远古大门玩家出场与撒盐特效族（quagmire_portal_player_fx）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "暴食（The Gorge）活动的 5 个 fx.lua 通用表注册：远古大门玩家出场／滴水／出水水花与盘／碗撒盐特效；工厂行为面已逐项核验，脚本树内只有 3 个 quagmire 消费文件的依赖声明，实际生成点在 quagmire_event_server 事件数据包（本快照不含）。"
aliases: ["quagmire_portal_player_fx", "quagmire_portal_playerdrip_fx", "quagmire_portal_player_splash_fx", "quagmire_salting_plate_fx", "quagmire_salting_bowl_fx", "暴食传送门特效", "暴食撒盐特效", "盐盘特效", "盐碗特效", "远古大门特效", "The Gorge 特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/quagmire_portal.lua", "prefabs/quagmire_foods.lua", "prefabs/quagmire_food_burnt.lua", "entityscript.lua", "actions.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 暴食远古大门玩家出场与撒盐特效族（quagmire_portal_player_fx）

## 快速判断

- 职责与适用：暴食（The Gorge）活动表现 FX，`fx.lua` 通用表相邻 5 注册——远古大门玩家出场 `quagmire_portal_player_fx`（anim `idle`＋出生声）、玩家滴水 `quagmire_portal_playerdrip_fx`（无声）、出场水花 `quagmire_portal_player_splash_fx`（anim `exit`＋企鹅水花声）、撒盐 `quagmire_salting_plate_fx`／`quagmire_salting_bowl_fx`（盐 shake 声，盘／碗仅 anim 不同）。代理与本地实体分工、清理边界见 [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂·关键流程]]。[5 条目](../../../raw/current/scripts/fx.lua#L1715-L1750)
- 优先使用：查"暴食玩家出场／滴水／撒盐表现"读本页；生成时机与触发窗口在 quagmire 事件数据包，本页只保证工厂行为面与树内依赖关系。
- 重要副作用：5 条目均为纯表现条目（无 nameoverride／description，工厂只加 `FX` 标签，无玩法字段）；协议副作用（代理 1 秒自删、本地动画结束自删）归工厂页，不逐条重复。
- 修改边界：脚本树内无运行时生成点——in-tree 仅 3 个 quagmire 消费文件的依赖声明与 `event_server_data` 接线；专服不建本地表现。[事件数据加载](../../../raw/current/scripts/entityscript.lua#L77-L87)

## 注册与构造

5 条目同走通用工厂：本地实体播一次注册动画后按 animover 自删，朝向默认 `SetFourFaced`；树内无调用可核对，落点式／父挂式由（数据包内）生成者决定。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `quagmire_portal_player_fx` | 通用表条目：bank/build=`quagmire_portalspawn_fx`，anim=`idle`，声 `dontstarve/quagmire/common/portal/spawn`，fn=`FinalOffset1`（`AnimState:SetFinalOffset(1)`） | 玩家出场段。[条目](../../../raw/current/scripts/fx.lua#L1715-L1720)、[fn](../../../raw/current/scripts/fx.lua#L1-L3) |
| `quagmire_portal_playerdrip_fx` | 同协议：bank/build=`quagmire_portaldrip_fx`，anim=`idle` | 5 名中唯一无 sound、无 fn。[条目](../../../raw/current/scripts/fx.lua#L1723-L1726) |
| `quagmire_portal_player_splash_fx` | 同协议：bank/build=`quagmire_portalspawn_fx`（与出场共用），anim=`exit`，声 `dontstarve/creatures/pengull/splash`，fn=`FinalOffset1` | 出场资源分 idle／exit 两段；水花声借用企鹅音效。[条目](../../../raw/current/scripts/fx.lua#L1729-L1734) |
| `quagmire_salting_plate_fx` | 同协议：bank/build=`quagmire_salting_fx`，anim=`plate`，声 `dontstarve/quagmire/common/cooking/salt_shake`，fn=`FinalOffset1` | 撒盐·盘类菜品。[条目](../../../raw/current/scripts/fx.lua#L1737-L1742) |
| `quagmire_salting_bowl_fx` | 同协议：同 bank/build，anim=`bowl`，同声，fn=`FinalOffset1` | 撒盐·碗类菜品；与盘仅 anim 之差。[条目](../../../raw/current/scripts/fx.lua#L1745-L1750) |

命名证据：暴食为官方简中（"Gorge Event curios"→"暴食活动珍宝"，[po](../../../raw/current/scripts/languages/chinese_s.po#L380616-L380617)）；同注册名宿主的 STRINGS.NAMES.QUAGMIRE_PORTAL 键译作"远古大门"（[po](../../../raw/current/scripts/languages/chinese_s.po#L348079-L348082)），本页标题以该词指代 portal 宿主。5 个 FX 本身无 STRINGS.NAMES 键，"玩家出场／滴水／水花／撒盐"为已核验效果描述，非官方 FX 译名；撒盐语义另见盐渍菜品显示名 QUAGMIRE_SALTED_FOOD_FMT"咸的{food}"（[po](../../../raw/current/scripts/languages/chinese_s.po#L348206-L348207)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 5 名由 `prefabs/fx.lua` 加载器批量注册，ANIM 资产按 build 派生 `anim/<build>.zip` | 代理＋各端本地实体；专服只有代理 | [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂·关键流程]]、[资产派生](../../../raw/current/scripts/prefabs/fx.lua#L5-L16) |
| 消费者（依赖声明） | 出场 3 名声明于 quagmire_portal.lua（该文件另返回 activefx／bubblefx），撒盐 2 名声明于 quagmire_foods.lua 与 quagmire_food_burnt.lua | 装配期文件级声明，证明加载关系、不证明生成 | [portal 声明](../../../raw/current/scripts/prefabs/quagmire_portal.lua#L7-L13)、[foods 声明](../../../raw/current/scripts/prefabs/quagmire_foods.lua#L1-L6)、[burnt 声明](../../../raw/current/scripts/prefabs/quagmire_food_burnt.lua#L15-L22) |
| 消费者（事件数据包） | 3 文件 master_postinit 经 `event_server_data("quagmire", ...)` 接线到 quagmire_event_server 数据包；实际 SpawnPrefab 调用、触发窗口、父挂／落点与端均在包内，本快照不可见 | requireeventfile 为引擎级加载器，快照只含 scripts | [portal 接线](../../../raw/current/scripts/prefabs/quagmire_portal.lua#L95)、[foods 接线](../../../raw/current/scripts/prefabs/quagmire_foods.lua#L102)、[burnt 接线](../../../raw/current/scripts/prefabs/quagmire_food_burnt.lua#L71)、[加载实现](../../../raw/current/scripts/entityscript.lua#L77-L87) |
| 盐渍玩法（树内线索） | in-tree 的 REPLATE 动作重建菜品时以 `quagmire_saltable:Salt(0, true)` 保留盐渍态，全程不生成 FX | quagmire_saltable／quagmire_replatable 组件定义也在事件数据包 | [REPLATE 盐渍保留](../../../raw/current/scripts/actions.lua#L5300-L5310) |
| 同族对照 | 熔炉 `lavaarena_portal_player_fx` 同位注册：单条目、bloom、无 fn，注释标 anim 前 6 空帧对齐音效；暴食拆 idle／drip／exit 三名且无 bloom | 两活动模式的传送门玩家 FX 均树内无生成点、仅依赖声明 | [熔炉条目](../../../raw/current/scripts/fx.lua#L880-L886)、[[reference/prefabs/lavaarena_creature_teleport_small_fx|熔炉生物传送出生特效]] |
| 资源 | 3 个 build：quagmire_portalspawn_fx（出场＋水花）、quagmire_portaldrip_fx（滴水）、quagmire_salting_fx（撒盐两 anim）；音效事件 3 个（portal/spawn、pengull splash、cooking/salt_shake） | zip 由加载器按 build 声明，实体不在脚本快照，内容未解包 | [条目](../../../raw/current/scripts/fx.lua#L1715-L1750)、[资产派生](../../../raw/current/scripts/prefabs/fx.lua#L5-L16) |

## 典型接入

### 复用暴食出场／撒盐特效

- 前提与选择：模组内对注册名 `SpawnPrefab` 后摆位或父挂即可，无需自备动画与音效；落点式／父挂式语义见 [[reference/mechanisms/generic-fx-factory#典型接入|通用 FX 工厂·典型接入]]。
- 生效与撤销：本地表现一帧后出现、动画播完自删；代理 1 秒自清，宿主侧无需额外清理。
- 原版依据：树内无调用方（生成在事件数据包），以上为工厂行为面；撒盐盘／碗按菜品器形选名。

```lua
local fx = SpawnPrefab("quagmire_salting_bowl_fx")
fx.Transform:SetPosition(x, y, z)   -- 落点式；或 fx.entity:SetParent(host.entity) 父挂随宿主
```

- 调整与验证：两撒盐条目仅 anim 之差、出场与水花共用同一 bank 不同 anim；原版触发窗口须对齐事件数据包内调用，本快照不可见，接入后需自行运行验证。

## 常见陷阱与限制

- **边界：脚本树内查不到生成调用**：对 5 全名及拼名片段（`portal_player`／`salting`／`plate_fx`／`bowl_fx`／`playerdrip`／`player_splash`）在全 scripts 根（根级文件＋prefabs/＋components/＋stategraphs/）大小写不敏感检索，仅命中 fx.lua 数据与 3 处依赖声明；实际生成点在 `quagmire_event_server` 数据包（引擎 `requireeventfile` 加载，本快照不含）。树内可断言"无运行时生成点"，不能外推数据包外的全库情况。
- **边界：依赖声明不证明生成**：3 文件的 `prefabs` 数组只证明加载关系；触发窗口、父挂／落点与端在本快照不可核验，别从声明反推表现时机。
- **边界：滴水条目无声**：`quagmire_portal_playerdrip_fx` 是 5 名中唯一无 sound 的条目；若滴水需配音，由生成方自行播放。
- **边界：动画时长未核验**：是否循环、单段时长由 anim zip 数据决定，zip 不在脚本快照；不要假设 `idle` 段即持续表现。
- **边界：特效不是撒盐结算**：5 条目无玩法字段；盐渍状态在 `quagmire_saltable`（事件数据包）维护，播放特效不等于完成撒盐。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。5 条目逐项（bank/build/anim/sound/fn 及相邻条目边界）、3 消费文件依赖声明与 event_server_data 接线、in-tree REPLATE 盐渍保留路径、po 定向核验（暴食／远古大门／咸的{food}；5 FX 无 NAMES 键）已核验；负向检索为全 scripts 根大小写不敏感全名＋拼名片段（见上）。未覆盖：quagmire_event_server 数据包内实际生成点、触发窗口、父挂／落点与端；anim zip 内容与动画时长；音效实听。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/prefabs/index|Prefab 索引]]。
