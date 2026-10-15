---
title: "lunarthrall_plantspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月裂隙植物潮世界组件：月裂隙长成后周期投放 gestalt 寄生普通植物，目标取自植物群组或野生补丁，boss 占尸会削减波次。"
aliases: ["月裂隙植物潮", "月缚蔓藤寄生", "lunarthrall_plantspawner 组件"]
tags: [component, ai]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/lunarthrall_plantspawner.lua", "prefabs/forest.lua", "prefabs/lunarrift_portal.lua", "prefabs/corpses.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lunarthrall_plantspawner

## 快速判断

- 职责与适用：forest 世界组件（prefabs/forest.lua:706），全事件驱动：`ms_lunarrift_maxsize` 开启 3+random(1,3) 波寄生（:166-181），每波最多让 3 株植物被 `lunarthrall_plant` 寄生（:92-137）。
- 优先使用：给可种植目标接群组 → `setHerdsOnPlantable(plantable)`（:529-533，berrybush.lua:222-223、grass.lua:201-202、sapling.lua:100-101、plantables.lua:32-33 已用）；调试强植 → `SpawnPlant(target)`（debugcommands.lua:1964）。
- 重要副作用：寄生目标登记在 `targetedplants`（:133），被寄生后由 `lunarthrallplant_infested` 清除（lunarthrall_plant.lua:152 推事件，:217-222 消费）。
- 修改边界：波次节奏写死——单波启动延迟 `TIME_UNIT`±半个单位（`SEG_TIME*8`，:4、:153），波间 `PERIODIC_TIME`=2 天（:6）；裂隙消失（`ms_lunarportal_removed`）即取消全部任务（:199-215）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SpawnGestalt` | `(target, rift?) -> nil` | 有 rift 时从裂隙喷出 `lunarthrall_plant_gestalt`（:288-295）；否则找玩家视野外落点（`IsAnyPlayerInRangeSq` 循环外推，:298-319）。 |
| `InvadeTarget` | `(target) -> nil` | 目标可见时生成 gestalt，不可见直接 `SpawnPlant` 寄生（:424-437）。 |
| `SpawnPlant` | `(target) -> nil` | 生成 `lunarthrall_plant` 并 `infest(target)`（:323-327）。 |
| `FindHerd`／`FindWildPatch` | `() -> herd\|table` | 群组按可寄生成员数排序取前 5 随机（:370-401）；野生补丁在随机陆地 20 半径找 `plant`+`lunarplant_target` 标签实体，10 次取最优（:339-368）。 |
| `RemoveWave` | `() -> nil` | 削减一波；归零后 10s 计时关闭裂隙（:505-527）；`ms_gestalt_possession`（epiccorpse/wargcorpse，corpses.lua:245 推）触发。:234-239 |
| `OnSave`/`OnLoad`/`LoadPostPass` | 常规存取 | 存波次、两任务剩余时间与裂隙 GUID（:439-478）；`LongUpdate` 平移两任务（:489-503）。 |

## 关键流程：一波寄生的选点约束

1. 触发：裂隙达最大尺寸（lunarrift_portal.lua:316/:566 推 `ms_lunarrift_maxsize`）→ 记 `currentrift`、排 `PERIODIC_TIME` 周期任务与首个 `_nextspawn`（:166-181、:148-163）。
2. 选目标：先查 `plantherds`（`plantherdspawned` 事件登记，domesticplantherd.lua:11 推，:193-197），无群组退回 `FindWildPatch`；候选须未被标记（`targetedplants`）、30 距离内无已有 `lunarthrall_plant`（`EXISTING_PLANT_SPACE`，:8）、与已选目标间距 2.5~8（:87-128）。
3. 释放：每目标 0~2s 延迟 `releasethrall`（:139-142）——玩家在裂隙屏内则 `SpawnGestalt(target, rift)`，否则 `InvadeTarget`（:14-34）；一波放完即取消后续任务（:41-62）。
4. 收尾：最后一波后 `timer "endrift"` 10s → `finish_rift` 事件关裂隙（:224-232）。

## 典型接入

### 让新可种植植物成为寄生目标

- 前提与选择：植物须有 `lunarplant_target` 标签或加入 `domesticplantherd` 群组；组件按 `herd.components.herd.members` 枚举（:82-84、:380-388）。
- 生效与撤销：`setHerdsOnPlantable` 会给实体加 `knownlocations` 与 `herdmember` 组件并指定群组 prefab（:529-533），重复调用会重复加组件，只调一次。
- 原版依据：berrybush.lua:222-223、plantables.lua:32-33。

```lua
AddPrefabPostInit("my_plant", function(inst)
    if TheWorld.ismastersim and TheWorld.components.lunarthrall_plantspawner then
        TheWorld.components.lunarthrall_plantspawner:setHerdsOnPlantable(inst)
    end
end)
```

- 调整与验证：组件存在性必须判 nil（原版同款写法）；寄生表现由 `lunarthrall_plant` prefab 决定，本组件只负责选点与投放。

## 常见陷阱与限制

- 强依赖裂隙实体：`SpawnThralls` 直接 `self.currentrift.SoundEmitter`（:40），裂隙提前被移除而事件未到会报错；`OnLunarPortalRemoved` 是唯一清理路径（:199-215）。
- 群组来自 `domesticplantherd`：`plantherdspawned` 只认该事件，自建群组需自行推同名世界事件（domesticplantherd.lua:11）。
- 寄生判定用实例字段：`target.lunarthrall_plant` 为真即跳过（:331-334、:349、:384），不是组件检查。
- `OnSave` 引用 `refs` 返回 currentrift GUID（:441-451），LoadPostPass 靠 `newents` 还原（:460-463）；裂隙实体必须同存档保存。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 535 行；挂载 forest.lua:706、事件源 lunarrift_portal.lua:316/:566、lunarthrall_plant.lua:152、corpses.lua:245、domesticplantherd.lua:11，种植目标接线 berrybush/grass/sapling/plantables 逐行核对。
- 游戏运行：未运行。
