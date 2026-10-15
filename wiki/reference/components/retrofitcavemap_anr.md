---
title: "retrofitcavemap_anr"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴老档内容迁移组件：按存档里的一次性 retrofit 旗标给旧世界补 A New Reign／Return of Them 世代内容（遗迹回响者、大堂迷宫、圣所等），完成即清旗标。"
aliases: ["洞穴地图迁移", "老档补内容", "retrofitcavemap_anr"]
tags: [component, worldgen, data]
dst_build: "steam-buildid:26230496"
sources: ["components/retrofitcavemap_anr.lua", "prefabs/cave.lua", "savefileupgrades.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# retrofitcavemap_anr

## 快速判断

- 职责与适用：世界级组件（仅主端），装配于洞穴世界（[cave](../../../raw/current/scripts/prefabs/cave.lua#L562)）；只服务**旧存档升级**——旗标由 `savefileupgrades.lua` 的存档版本迁移写入 `map.persistdata.retrofitcavemap_anr`，新世界与已迁移世界不会触发（[旗标写入](../../../raw/current/scripts/savefileupgrades.lua#L21-L24)）。
- 优先使用：模组给老档补内容时应仿照其一次性旗标模式（OnLoad 读旗标 → OnPostInit 执行 → 执行后置 nil），并复用 `RetrofitNewCaveContentPrefab` 的选点思路（[L52-L114](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L52-L114)）。
- 重要副作用：直接向世界添加实体、改 `TheWorld.topology`（新增 Atrium 节点）、改地块（胡须地毯修复）；部分步骤（Atrium 迷宫、导航网格）推 `requiresreset` 令服务器 45 秒后存档回滚重启（[L1083-L1096](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L1083-L1096)）。
- 修改边界：`OnSave` 返回空表，旗标只经 OnLoad 消费且每个旗标用一次（执行即清）；不存在运行时接口——组件生命周期就是"读档 → OnPostInit 跑批"（[L608-L1097](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L608-L1097)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `self.retrofit_*` 旗标组 | bool，OnLoad 读入 | 每个旗标对应一个迁移步骤：warts（蟾蜍王帽）、artsandcrafts（化石）、heartoftheruins（Atrium/遗迹）、respawnerfix、altars、caveholes、oldatriumfixup、statuechessrespawners、sacred_chest、acientarchives（原文拼写即如此）、dispencer_fixes、archives_navmesh、nodeidtilemap_atriummaze、daywalker_content、rifts6_add_fumarole、cave_mite_spawners、bat_boss_cave、add_one_vault_orb 等（[L1107-L1132](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L1107-L1132)）。 |
| `RetrofitNewCaveContentPrefab(inst, prefab, min_space, dist_from_structures, nightmare, searchnodes_override, ignore_terrain)` | → `bool, ent` | 50 次尝试在拓扑节点多边形内随机选点：默认只在 Cave 或 Ruins（Nightmare）节点（排除 Atrium／lunacyarea／RuinedGuarden），四方 `min_space` 无可放点、附近无实体、距 `structure` 达标才生成（[L52-L114](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L52-L114)）。 |
| `AddRuinsRespawner(prefab, spawnerprefab)` | → 数量 | 给现存的 `minotaur`／`monkeybarrel`／`slurper` 等遗迹实体配 `_ruinsrespawner_inst`（对象复活者，[L236-L257](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L236-L257)）；不足额再 `RepopNear`（靠 nightmarelight）/`RepopRandom` 补（[L259-L352](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L259-L352)）。 |
| `AddAtriumWorldTopolopy(left, top)` | 内部函数 | 向 `TheWorld.topology` 追加 32×32 格 Atrium 节点（tag `Atrium`,`Nightmare`），随后 `requiresreset=true`（[L120-L145](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L120-L145)）。 |
| `HeartOfTheRuinsAtriumRetrofitting` | — | 在地图边缘 7 个候选区找 41×41 全不可通行区块，用 object_layout 摆 4×4 大堂迷宫＋首尾房；wormhole_MARKER 位置生成 `tentacle_pillar` 触手门并配对（[L147-L233](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L147-L233)）。 |
| `ArchiveDispencerFixup` | — | 保证三类 archive_lockbox_dispencer（a/b/c 产品）齐备，缺失则改造现有分配器；无产品 lockbox 一并随机补产品（[L474-L578](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L474-L578)）。 |
| `BatBossCave_Retrofitting` | — | 在 `FernyBatCave` 节点内放 `batbosscave` 静态布景（先 12 距离后 6 距离放宽）（[L580-L602](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L580-L602)）。 |
| `requiresreset` 处理 | bool | 真：倒计时公告→`ms_save`→`TheNet:SendWorldRollbackRequestToServer(0)` 45 秒后回滚重启；OnLoad 可从存档继承该旗标（供 savefileupgrades 复用）（[L1083-L1096](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L1083-L1096)、[L1109](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L1109)）。 |

## 关键流程

### 一次迁移执行（OnPostInit）

1. OnLoad 把存档旗标铺到 self；OnPostInit 依序判断各旗标，执行对应步骤后立即置 nil（防重复执行）（[L608 起](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L608-L656)）。
2. 内容放置类步骤全部经由 `RetrofitNewCaveContentPrefab` 或按 topology 节点 id（`string.find` 匹配 `FernyBatCave`/`BGVentsRoom`/`LightPlantField` 等）定点处理（[L1016-L1031](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L1016-L1031)、[L860-L908](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L860-L908)）。
3. 特殊联动：archives 迁移委托给世界中预置的 `retrofit_archiveteleporter`／`retrofit_fumaroleteleporter` 实体的 `DoRetrofitting()`（失败时打印管理员手动指令）（[L776-L793](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L776-L793)、[L934-L951](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L934-L951)）。
4. 需要重开局的步骤（迷宫、导航网格、Atrium 瓦片修复）置 `requiresreset`，最终统一走保存回滚流程（[L834-L856](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L834-L856)）。

## 常见陷阱与限制

- **旗标一次性**：执行成功与否都会清旗标；模组加自己的迁移步骤要仿 `savefileupgrades` 写旗标＋组件加分支，不能依赖 OnSave（返回空表）（[L1103-L1105](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L1103-L1105)）。
- **无重试上限的兜底循环**：蟾蜍王帽与 daywalker spawningground 放置失败会无限循环重试（打印"Trying really hard"），理论上极端地图会卡 OnPostInit（[L615-L618](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L615-L618)、[L900-L903](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L900-L903)）。
- **依赖英文节点 id 字符串**：`RuinedGuarden`（拼写如此）、`AtriumMaze`、`FernyBatCave` 等都是 worldgen 布局 id 的 `string.find` 匹配，自建任务集改名会使迁移找不到宿主（[L69](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L69)、[L589](../../../raw/current/scripts/components/retrofitcavemap_anr.lua#L589)）。
- **地表不适用**：地表老档迁移是 `retrofitforestmap_*`（同目录 forest 组件）；本组件只在洞穴世界装配（[cave](../../../raw/current/scripts/prefabs/cave.lua#L562)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、cave 装配行、savefileupgrades 旗标写入模式；未覆盖 `_ruinsrespawner_inst` 系列与 archive teleporter prefab 的 `DoRetrofitting` 实现。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。
