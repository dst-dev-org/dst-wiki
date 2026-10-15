---
title: "retrofitforestmap_anr"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "地表老档内容迁移组件：消费存档版本迁移写入的一次性旗标，用拓扑节点选点给旧森林世界补 ANR／Return of Them／跨界联动世代内容，改动地形或导航时强制存档重启。"
aliases: ["地表地图迁移", "老档补内容", "retrofitforestmap_anr"]
tags: [component, worldgen, data]
size_class: complex
dst_build: "steam-buildid:26230496"
sources: ["components/retrofitforestmap_anr.lua", "prefabs/forest.lua", "savefileupgrades.lua", "gamelogic.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# retrofitforestmap_anr

## 快速判断

- 职责与适用：地表世界（forest）专用世界组件，仅主端（[装配](../../../raw/current/scripts/prefabs/forest.lua#L670)、[mastersim 断言](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L9)）；只服务**旧存档升级**——旗标由 `savefileupgrades.lua` 按存档版本迁移写入 `map.persistdata.retrofitforestmap_anr`（[写入器](../../../raw/current/scripts/savefileupgrades.lua#L3-L14)），经 `world:SetPersistData(savedata.map.persistdata)` 送达组件 OnLoad（[gamelogic](../../../raw/current/scripts/gamelogic.lua#L579-L580)）。新世界与已迁移世界不触发。洞穴侧对应 [[reference/components/retrofitcavemap_anr]]。
- 优先使用：模组给老档补内容时仿其三段式（OnLoad 读旗标 → OnPostInit 执行 → 用后置 nil）并复用 50 次尝试选点思路（[RetrofitNewContentPrefab](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L48-L88)）；查"为什么我的老档没补到内容"先看旗标清单与消费点是否对得上。
- 重要副作用：直接生成／删除实体、改 `TheWorld.topology` 节点标签、改地块与导航网格；`requiresreset` 会在 30 秒倒数后存档并 `SendWorldRollbackRequestToServer(0)` 重启（[L1909-L1921](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1909-L1921)）。
- 修改边界：`OnSave` 返回空表（[L1929-L1931](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1929-L1931)）——旗标只活一次会话，重启后的存档不再携带；组件生命周期就是"读档 → OnPostInit 跑批"（[L1296-L1923](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1296-L1923)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnLoad(data)` 读旗标组 | bool，约 37 个 `retrofit_*`／`*_fix` 字段 | 与 `savefileupgrades` 写入的旗标名一一对应（`retrofit_part1` 为文件内 local 变量，其余在 self 上）（[L1933-L1974](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1933-L1974)）。 |
| `OnPostInit()` | 读档后跑批 | 依序判断各旗标执行迁移步骤；多数步骤自带"世界已有对应实体则跳过"的守卫（如 moonbase／antlion_spawner／deerspawningground）（[L1296-L1923](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1296-L1923)）。 |
| `RetrofitNewContentPrefab(inst, prefab, min_space, dist_from_structures, canplacefn, candidate_nodes, on_add_prefab)` | `-> bool` | 文件内 local：最多 50 次在拓扑节点（或候选节点表）多边形内 `GetRandomPointsForSite` 选点，`canplacefn` 或 `CanPlacePrefabFilteredAtPoint` 校验、四方 `min_space` 无实体、距 `structure` 达标才生成并回调（[L48-L88](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L48-L88)）。 |
| `RetrofitAgainstTheGrain(area)` | 内部函数 | `"Oasis"` 失败再 `"Badlands"`：给节点组打 `sandstorm` 标签、清点后放 `antlion_spawner` 与 `oasislake`、Oasis 区仙人掌换 `oasis_cactus`（[L770-L919](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L770-L919)）。 |
| `requiresreset` | bool | 任一改地形／导航步骤置真后统一走"倒数公告 → `ms_save` → 回滚重启"； cave 页同款机制（[L1909-L1921](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1909-L1921)）。 |
| `TILE/导航修复` | — | `nodeidtilemap_secondpass`（月岛 lunacyarea 节点重算 + reset，[L1555-L1564](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1555-L1564)）、`turnoftides_betaupdate1`（`Map:RetrofitNavGrid`，[L1477-L1481](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1477-L1481)）、`console_beard_turf_fix`（undertile 为空的 RIFT_MOON 地块改 BEARD_RUG，[L1627-L1645](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1627-L1645)）。 |

## 关键流程

### 旗标 → 步骤对照（OnPostInit 分组）

- **ANR 世代**：`retrofit_part1`（stagehand/moonbase/国际象棋雕件组，L1297-L1318）、`artsandcrafts`（缺哪种补哪种 `chesspiece_*_sketch`，L1320-L1357）、`artsandcrafts2`（推 `ms_unlockchesspiece` 解锁图纸，L1359-L1368）、`cutefuzzyanimals`（critterlab/beequeenhive，L1370-L1391）、`herdmentality`（8 只 deerspawningground，L1394-L1422）、`againstthegrain`（蚁狮+绿洲，L1424-L1443）、`penguinice`（冰岩标记 dryup 可除，L1445-L1465）。
- **Return of Them 海洋世代**：`turnoftides`（全海洋撒漂流木/海帶/海蚀柱 + reset，L1467-L1475）、`turnoftides_seastacks`（清旧海柱重播成批刷怪点，L1483-L1486）、`salty`（obstruction 鲹鱼群刷点，L1519-L1526）、`shesellsseashells`（wobster 巢 + reset，L1529-L1533）、`barnacles`（30% 海柱刷怪点换藤壶种植点，L1535-L1538）、`inaccessibleunderwaterobjects`（吊不起的沉物复位，L1540-L1543）、`moonfissures`（月裂缝旁配对补水，L1545-L1548）、`astralmarkers`（ExitPiece 节点放两个星体祭坛标记，L1550-L1553）、`removeextraaltarpieces`（清 (0,0,0) 错位祭坛件，L1580-L1583）、`rifts6_add_whirlpool`（危险海找 6×6 格放大漩涡，L1714-L1754）。
- **跨界与内容包**：`terraria_terrarium`（BGForest 森林地皮放 terrariumchest 并挂 chest_terrarium 场景，L581-L615）、`alittledrama_content`（charlie_stage_post＋statueharp_hedgespawner，L923-L1012）、`junkyard_content`／`junkyardv3_content`（junk_pile_big 为日舞者战斗必需，五档放宽强试，L1017-L1190）、`balatro_content`（balatro_machine，L1229-L1287）、`otterdens`（海岸撒 boat_otterden，L1192-L1225）、`daywalker_content`（删除地表旧 daywalker 内容，L1607-L1623）。
- **专项修复**：`fix_sculpture_pieces`（雕件头身复位，L1488-L1517）、`remove_rift_terraformers_fix`（清无主裂隙改造器，L1662-L1687）、`sharkboi_ice_hazard_fix`（清非海面冰块，L1698-L1712）、`fix_pearl_eating_everything`（珍珠吃了奖励的补偿，L1758-L1799）、`floating_heavyobstaclephysics_fix`（悬浮重物打补丁标记，L1803-L1811）、`missing_retrofits_generated_densities`／`enable_pearl_score`／`hermitcrab_relocation_change`（L1815-L1905）。

### 布点与海洋撒布

- 陆地内容统一走 `RetrofitNewContentPrefab`，常见两级放宽：先按地皮 `canplacefn`（如 FOREST/DECIDUOUS/目标瓦片白名单），失败去掉地皮限制或缩 `min_space` 再试（[L611-L613](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L611-L613)、[L1161-L1183](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1161-L1183)）。
- 海洋撒布按 `WORLD_TILES` 逐格以 `distributepercent` 加权随机（[populate_ocean](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L143-L161)）；成批刷怪借 `map/bunch_spawner`，并统一避让 `walkableplatform` 半径+4（[L222-L227](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L222-L227)）。

## 典型接入

### 模组借用该组件给老档补内容

- 前提与选择：主端；旗标来源要自管（原版 savefileupgrades 只写原版旗标）。消费侧用 `AddComponentPostInit` 包一层 OnLoad/OnPostInit，复用同一条 `map.persistdata` 通道。

```lua
AddComponentPostInit("retrofitforestmap_anr", function(self)
    local oldload, oldpostinit = self.OnLoad, self.OnPostInit
    self.OnLoad = function(s, data)
        oldload(s, data)
        if data ~= nil and data.my_mod_retrofit then
            s.my_mod_retrofit = true
        end
    end
    self.OnPostInit = function(s)
        oldpostinit(s)
        if s.my_mod_retrofit then
            s.my_mod_retrofit = nil
            print("My mod forest retrofit: placing content")
            -- 在此做选点放置；旗标写入需仿 savefileupgrades.lua:3-14
            -- 往 savedata.map.persistdata.retrofitforestmap_anr["my_mod_retrofit"] = true
        end
    end
end)
```

- 调整与验证：`my_mod_retrofit` 名字必须与写入侧一致；放完即清是关键——组件 OnSave 为空，不清旗标只会在同一次会话内多跑，但重启后旗标消失、语义不明。改地形类步骤需自置 `s.requiresreset`（复用原版倒数重启流程）。

## 常见陷阱与限制

- **catcoonden 分支永不可达**：OnPostInit 检查 `self.retrofit_catcoonden_deextinction`（[L1590-L1593](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1590-L1593)），但 OnLoad 清单没有这个字段（[L1933-L1974](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1933-L1974)），而 savefileupgrades 确实写入该旗标（[L1228](../../../raw/current/scripts/savefileupgrades.lua#L1228)）——当前版本补猫狸巢的旗标会被静默丢弃。
- **pearl 修复可截断后续步骤**：`fix_pearl_eating_everything` 在珍珠不在场时 `return`（[L1774-L1777](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1774-L1777)），直接跳出 OnPostInit，排在它后面的修复与 `requiresreset` 重启流程都被跳过；同批旗标会被下一次存档清掉。
- **部分旗标用后不清**：turnoftides_betaupdate1、salty、shesellsseashells、barnacles、alittledrama、junkyard 等不置 nil（对照 [L1477-L1486](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1477-L1486)）；因 OnSave 为空它们实际仍是一次性，模仿时别把"没清"当成可重复执行。
- **依赖英文布景 id 前缀**：`BGForest`/`BGDeciduous`/`ExitPiece`/`StaticLayoutIsland:HermitcrabIsland` 等都是对 `topology.ids` 的字符串匹配（[L590-L594](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L590-L594)、[L711](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L711)、[L1569](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1569)）；自建任务集改名会让迁移找不到宿主节点。
- **两个未接线的海洋助手**：`RetrofitNewOceanContentPrefab`（L90-L129）无调用点；`SaltyRetrofitting_PopulateBrinePools` 调用被注释（[L1524-L1525](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1524-L1525)），盐池批量补点在当前版本不会执行。
- **与 cavemap 侧差异**：本组件服务 forest 世界；洞穴老档走 [[reference/components/retrofitcavemap_anr]]，两页旗标名不通用。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。通读组件全部 1976 行；装配行（forest.lua:670）、旗标写入器与抽样写入点（savefileupgrades.lua:3-13、:672-675、:1228）、送达链两行（gamelogic.lua:579-580、:629）逐行核对；catcoonden 死分支与两个未接线助手经全文件调用点检索确认。页面为精选覆盖：37 个旗标按组归并叙述，未把每个迁移步骤的内部细节逐行展开。
- 结构检查：validate_page、validate_source_links --strict 通过（见批级记录）。
- 游戏运行：未运行；实际老档升级与 requiresreset 重启未实测。

相关：[[reference/components/retrofitcavemap_anr]]、[[reference/components/undertile]]、[[reference/components/terraformer]]
