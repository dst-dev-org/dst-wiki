---
title: "miasmamanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴世界按米玛斯瓦片网格管理雾气（miasma）的生成、扩散、增强与消退排程，并提供查询、显式创建与消退标记接口。"
aliases: ["miasmamanager", "世界雾气", "米玛斯管理器", "阴影裂隙雾气", "CreateMiasmaAtPoint"]
tags: [component, lifecycle, data]
dst_build: "steam-buildid:26230496"
sources: ["components/miasmamanager.lua", "prefabs/cave.lua", "prefabs/cave_vents.lua", "prefabs/shadowrift_portal.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# miasmamanager

## 快速判断

- 职责与适用：洞穴世界的雾气主管理器，构造断言 `TheWorld.ismastersim`，原版在洞穴 master_postinit 安装；以 `TUNING.MIASMA_SPACING` 划分的瓦片坐标维护 DataGrid 强度数据，排程生成／扩散／消退，不是玩家中毒或 cloud 实体自身的逻辑。[断言](../../../raw/current/scripts/components/miasmamanager.lua#L16-L18) [安装](../../../raw/current/scripts/prefabs/cave.lua#L568-L570)
- 优先使用：显式造雾用 `CreateMiasmaAtPoint(x,y,z)`（内部校验陆地瓦片）；查询某点雾气数据用 `GetMiasmaAtPoint`，返回 `{strength, diminishing}` 或 nil；标记消退用 `SetMiasmaDiminishingAtPoint`。[创建](../../../raw/current/scripts/components/miasmamanager.lua#L192-L224) [查询](../../../raw/current/scripts/components/miasmamanager.lua#L156-L163)
- 重要副作用：每块新雾气会 `SpawnPrefab("miasma_cloud")` 并把世界状态置为活跃（`StartUpdatingComponent` + 世界事件 `miasma_setactive`）；消退到 0 时按击杀半径移除附近 miasma 标签实体。[云生成](../../../raw/current/scripts/components/miasmamanager.lua#L217-L223) [激活切换](../../../raw/current/scripts/components/miasmamanager.lua#L169-L179) [移除](../../../raw/current/scripts/components/miasmamanager.lua#L82-L87)
- 修改边界：创建资格依赖世界组件 `riftspawner`（阴影亲和裂隙）与 `miasma_venter` 标签实体；无阴影裂隙时既有云只消退不扩散。雾气网格与 enabled 状态随世界存档（ZipAndEncodeSaveData）。[创建资格](../../../raw/current/scripts/components/miasmamanager.lua#L226-L249) [存读](../../../raw/current/scripts/components/miasmamanager.lua#L116-L146)

## 关键接口

| 接口／配置 | 条件、作用与证据 |
|---|---|
| `GetMiasmaTileCoords(tx,ty)` | 把世界瓦片坐标向下取整对齐到雾气网格（间隔 `MIASMA_SPACING=1` 瓦），制造瓦片间空隙。[换算](../../../raw/current/scripts/components/miasmamanager.lua#L151-L155) [常量](../../../raw/current/scripts/tuning.lua#L7731-L7732) |
| `GetMiasmaAtPoint(x,y,z)`／`GetMiasmaAtTile(tx,ty)` | 返回该网格的雾气数据表（`strength`，可能带 `diminishing`）或 nil；云实体与玩家侧检查都用它做权威查询。[查询](../../../raw/current/scripts/components/miasmamanager.lua#L156-L163) [云消费者](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L427-L429) |
| `CreateMiasmaAtPoint(x,y,z)`／`CreateMiasmaAtTile(tx,ty)` | 校验瓦片存在、是陆地且有 tileinfo，网格已有数据则直接返回；成功则写入 strength=1、生成 `miasma_cloud` 并 `SetMiasmaActive(true)`。不校验裂隙／气孔距离（那是 Roll 的 Create 动作做的事）。[创建](../../../raw/current/scripts/components/miasmamanager.lua#L196-L224) [资格校验另在](../../../raw/current/scripts/components/miasmamanager.lua#L253-L271) |
| `IsValidForMiasmaCreationAt(mtx,mty)` | 返回该瓦片中心是否在阴影裂隙的 `[MIASMA_MIN_DISTSQ_FROM_RIFT, MIASMA_MAX_DISTSQ_FROM_RIFT]` 距离带内（1～12 瓦），或 3 瓦内有 `miasma_venter`。[校验](../../../raw/current/scripts/components/miasmamanager.lua#L226-L249) [常量](../../../raw/current/scripts/tuning.lua#L7738-L7740) |
| `SetMiasmaActive(active)` | 切换组件更新并在状态变化时向世界推 `miasma_setactive`；`IsMiasmaActive()` 读同源布尔。OnUpdate 里无云时自动置 false。[切换](../../../raw/current/scripts/components/miasmamanager.lua#L169-L179) [自动停止](../../../raw/current/scripts/components/miasmamanager.lua#L349-L354) |
| `SetMiasmaDiminishingAtPoint/AtTile` | 标记某雾气格为消退中；消退中的格子会在 Spread 轮里被额外削减，进入 `_diminishing_datas` 每秒结算。[标记](../../../raw/current/scripts/components/miasmamanager.lua#L93-L107) [结算](../../../raw/current/scripts/components/miasmamanager.lua#L333-L337) |
| `RollForMiasmaActionAt(mtx,mty,allowspread)` | 单格动作判定：世界无活跃阴影裂隙时只对已有云 Diminish；否则按概率 Create（`MIASMA_ODDS_CREATE=0.8`）、扩散（`MIASMA_ODDS_SPREAD=0.5`，向四邻递归 Roll）或 Enhance（strength 上限 `MIASMA_MAXSTRENGTH=2`）。[判定](../../../raw/current/scripts/components/miasmamanager.lua#L291-L329) |
| `DoRolls()` | 随机取一格执行 Roll；仅当云数 `< MIASMA_MAX_CLOUDS(500)` 时允许扩散。`DoDiminishes()` 对全部消退标记按 `MIASMA_DIMINISH_INTERVAL_SECONDS=1` 结算，扩散轮间隔 5 秒。[轮询](../../../raw/current/scripts/components/miasmamanager.lua#L338-L345) [更新循环](../../../raw/current/scripts/components/miasmamanager.lua#L356-L369) |
| `OnSave()`／`OnLoad(data)` | 存压缩的网格数据与 enabled；OnLoad 重建索引缓存并恢复 diminishing 标记，最后按存档恢复活跃状态。[存读](../../../raw/current/scripts/components/miasmamanager.lua#L116-L146) |

### 排程与裂隙条件

组件仅在有云时处于更新状态；`OnUpdate` 用两个独立的间隔计时器驱动消退与扩散轮，不随帧率漂移（while 累积补偿）。创建动作额外要求 `TheWorld.components.riftspawner` 存在且有阴影亲和裂隙（`GetRiftsOfAffinity(RIFTPORTAL_CONST.AFFINITY.SHADOW)`），`IsShadowPortalActive()` 为假时既有云随轮次衰减。[创建门禁](../../../raw/current/scripts/components/miasmamanager.lua#L253-L263) [衰减分支](../../../raw/current/scripts/components/miasmamanager.lua#L294-L303)

## 典型接入

### 让洞穴气孔或自定义实体产生雾气

- 前提与选择：模组端只应在主模拟世界（cave）拿到 `TheWorld.components.miasmamanager` 后调用；原版气孔在配置为喷雾时直接调用 `CreateMiasmaAtPoint`，并在实体初始化时给自己 `miasma_venter` 标签以参与 Roll 的资格校验。[气孔接线](../../../raw/current/scripts/prefabs/cave_vents.lua#L180-L184) [气孔标签](../../../raw/current/scripts/prefabs/cave_vents.lua#L215-L217)
- 生效与撤销：创建成功后组件自动进入更新并开始按轮次扩散／消退；删除世界组件不会自动移除已生成的 `miasma_cloud` 实体。无原版"撤销/清除"接口，清雾需自行按 `miasma` 标签移除实体并处理网格数据（无公开清理接口）。
- 原版依据：阴影裂隙 portal 在放置时向周围瓦片批量造雾；这也是裂隙存在时雾气自然恢复的来源之一。[裂隙造雾](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L81-L88)

```lua
-- 服务器端（mastersim）：在洞穴世界就绪后，于指定坐标显式创建雾气
AddPrefabPostInit("cave", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst:DoTaskInTime(0, function()
        local manager = inst.components.miasmamanager
        if manager == nil then
            return
        end
        manager:CreateMiasmaAtPoint(100, 0, 200) -- 非陆地瓦片会被内部直接拒绝
        print("miasma active:", manager:IsMiasmaActive())
    end)
end)
```

- 调整与验证：点位必须是陆地瓦片（`IsLandTile` 且有 tileinfo）；想自定义雾气强度／上限可改 `TUNING.MIASMA_*`，但组件构造不缓存 TUNING，改动即时生效。[瓦片校验](../../../raw/current/scripts/components/miasmamanager.lua#L196-L209)

## 常见陷阱与限制

- **在森林世界调用会取不到组件：** 原版只在 cave 安装；`TheWorld.components.miasmamanager` 在森林为 nil，需自行判空。[安装](../../../raw/current/scripts/prefabs/cave.lua#L568-L570)
- **把 `CreateMiasmaAtTile` 当完整资格检查：** 它只查陆地与占用，不查裂隙距离带与 venter；从随机点位造雾想走"合法来源"需先 `IsValidForMiasmaCreationAt`（其内部用瓦片中心点距离）。[两种校验](../../../raw/current/scripts/components/miasmamanager.lua#L226-L249)
- **移除云实体不影响网格数据：** `_Diminish` 是唯一同时清数据并移除实体的路径；外部 `ent:Remove()` 云后，网格仍在，轮次还会 Enhance 该格（云实体会被重新生成吗——不会，Enhance 只加强度；但数据残留会阻止该格再 Create）。[消退路径](../../../raw/current/scripts/components/miasmamanager.lua#L75-L91)
- **组件无 OnRemoveFromEntity：** 世界监听（worldmapsetsize）与更新状态不随移除组件清理；只有 `SetMiasmaActive(false)` 会停更新。[监听](../../../raw/current/scripts/components/miasmamanager.lua#L44
- **雾气数值只有 1～2 档：** 当前 `MIASMA_MAXSTRENGTH=2`，Enhance 到顶后 Roll 只返回 "Enhance" 而无实际变化；不要按连续强度场建模。[增强](../../../raw/current/scripts/components/miasmamanager.lua#L66-L71) [常量](../../../raw/current/scripts/tuning.lua#L7731-L7740)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 miasmamanager.lua，核验网格与索引缓存、Create／Spread／Enhance／Diminish 四类动作、Roll 门禁、OnUpdate 排程、存读、裂隙／气孔资格与 cloud 实体生成；核对 cave 安装、cave_vents 与 shadowrift_portal 消费者及 miasma_cloud_fx 的查询使用。未审校雾气对玩家的具体伤害机制（在 miasmawatcher／player 侧）。
- 结构检查：本页通过 validate_page.py（格式与来源）与 validate_source_links.py（严格链接）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；模组造雾、TUNING 调整与云实体残留行为未实测。

相关：[[reference/components/riftspawner]]、[[reference/components/miasmawatcher]]。

## 相关链接

- [[reference/components/riftspawner]]：阴影裂隙生命周期，决定雾气创建与衰减分支。
- [[reference/components/miasmawatcher]]：玩家侧雾气计数与受影响逻辑。
