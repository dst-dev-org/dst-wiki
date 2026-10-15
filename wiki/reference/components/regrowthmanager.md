---
title: "regrowthmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级资源再生管理器：实体移除时登记队列，按种类时钟与季节倍率在原位附近重生 carrot、flower、蘑菇等资源，可注册自定义类型。"
aliases: ["regrowthmanager", "世界资源再生", "资源重生管理", "SetRegrowthForType"]
tags: [component, lifecycle, worldgen]
dst_build: "steam-buildid:26230496"
sources: ["components/regrowthmanager.lua", "standardcomponents.lua", "prefabs/cave.lua", "prefabs/forest.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# regrowthmanager

## 快速判断

- 职责与适用：主模拟世界（洞穴、森林都安装）的资源再生调度；“外部配置”型组件——原版初始化时注册 14 种资源（胡萝卜、花、兔洞、猫尾香蒲巢、洞穴花、蕨草、仙人掌、洞穴香蕉树、三色蘑菇、种植曼德拉草），实体按协议移除时入队，管理器按种类时钟重生产品。[断言](../../../raw/current/scripts/components/regrowthmanager.lua#L7) [安装](../../../raw/current/scripts/prefabs/cave.lua#L538) [森林安装](../../../raw/current/scripts/prefabs/forest.lua#L668)
- 优先使用：注册新类型用唯一公开方法 `SetRegrowthForType(prefab,regrowtime,product,timemult)`；修改原版节奏改对应 `TUNING.*_REGROWTH_*` 常量并在世界加载前生效。[注册接口](../../../raw/current/scripts/components/regrowthmanager.lua#L135-L138)
- 重要副作用：重生走 `SpawnPrefab(product)` 全新实体，不继承被移除实体的任何数据；重生点在原位置 20 半径内抖动，不是原位精确重生。[重生实现](../../../raw/current/scripts/components/regrowthmanager.lua#L83-L106)
- 修改边界：依赖 `map/terrain`、世界 Map 与 worldstate；组件监听世界 `beginregrowth` 事件，实体侧协议由 `standardcomponents.AddToRegrowthManager` 提供。无 OnRemoveFromEntity，移除组件不清理周期任务。[事件接线](../../../raw/current/scripts/components/regrowthmanager.lua#L147) [实体侧协议](../../../raw/current/scripts/standardcomponents.lua#L1843-L1847)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `SetRegrowthForType(prefab,regrowtime,product,timemult)` | 注册/覆盖一种类型的再生参数；`timemult` 是每次更新调用的函数，返回 0 完全暂停该类内部时钟。注册会清零该类 `_internaltimes`。[实现](../../../raw/current/scripts/components/regrowthmanager.lua#L135-L138) |
| `LongUpdate(dt)` | 每 29 秒周期调用（另有世界快进复用）；对 `timemult>0` 的种类累加内部时间 `dt*TUNING.REGROWTH_TIME_MULTIPLIER*timemult`（当前 1），再处理到期计时器。[入口](../../../raw/current/scripts/components/regrowthmanager.lua#L149) [实现](../../../raw/current/scripts/components/regrowthmanager.lua#L225-L270) [倍率](../../../raw/current/scripts/tuning.lua#L3219) |
| `beginregrowth` 事件（世界） | 由 `AddToRegrowthManager` 在实体 `onremove` 时推送，data 为被移除实体；管理器按 `target.prefab` 查配置入队，未注册只 print 不处理。[推送](../../../raw/current/scripts/standardcomponents.lua#L1834-L1837) [接收](../../../raw/current/scripts/components/regrowthmanager.lua#L114-L129) |
| `OnSave()`／`OnLoad(data)` | 保存各类型队列（product、相对剩余时间、位置）；加载时仅恢复已注册类型，未知 prefab 静默丢弃。[实现](../../../raw/current/scripts/components/regrowthmanager.lua#L276-L312) |
| `GetDebugString()` | 每种类型输出队列长度、当前 timemult 与队首剩余时间。[实现](../../../raw/current/scripts/components/regrowthmanager.lua#L318-L326) |

### 原版注册与其季节倍率

构造时注册的 14 类及各自条件倍率见 [注册段](../../../raw/current/scripts/components/regrowthmanager.lua#L151-L218)：胡萝卜夜间/冬季/积雪倍率 0；花要求白天、非冬季、湿润度>1 且非下雪，春季双速；兔洞夏季用夏季倍率；猫尾巢仅春季/秋季有倍率；仙人掌按季节与雨天相乘；三色蘑菇分别在白天/黄昏/夜晚快速。基准时长见 [TUNING](../../../raw/current/scripts/tuning.lua#L3169-L3186)。

### 落点判定与队列行为

到期计时器执行 `DoRegrowth`：先在原位置 20 半径抖动，玩家太近（`ENTITY_POPOUT_RADIUS`）返回 CACHE 留在队首下轮再试；否则 `TestForRegrow` 要求同地皮、可种植、不在路上、3 半径无实体、20 半径无 structure/wall/regrowth_blocker，全部通过才生成（[抖动与判定](../../../raw/current/scripts/components/regrowthmanager.lua#L83-L106)、[阻断标签](../../../raw/current/scripts/components/regrowthmanager.lua#L59-L81)）。FAILED 会把计时器重置为完整 regrowtime 后移到队尾（[失败重排](../../../raw/current/scripts/components/regrowthmanager.lua#L252-L259)）。

## 典型接入

### 为自定义资源注册世界再生

- 前提与选择：两部分协议缺一不可——实体 prefab 在构造时调用 `AddToRegrowthManager(inst)` 建立移除上报；世界侧在洞穴/森林 master_postinit 后注册同名类型。选本组件而非自建计时器可复用落点判定、玩家规避与存档恢复。[实体侧](../../../raw/current/scripts/standardcomponents.lua#L1843-L1847)
- 生效与撤销：注册即刻生效并清零该类时钟；重复调用覆盖旧配置。撤销没有官方接口，覆盖为 timemult 恒 0 可等效停用但不清队列。[覆盖语义](../../../raw/current/scripts/components/regrowthmanager.lua#L135-L138)

```lua
-- modmain.lua：在洞穴与森林世界注册自定义资源的再生
local function SetupRegrowth(inst)
    if TheWorld.ismastersim then
        inst.components.regrowthmanager:SetRegrowthForType(
            "my_harvestable",          -- 实体 prefab 名，须与其 AddToRegrowthManager 配套
            TUNING.TOTAL_DAY_TIME * 2, -- regrowtime：基准再生时长
            "my_harvestable",          -- 重生生成的 product prefab
            function() return 1 end)   -- timemult：按世界状态返回倍率，0 暂停
    end
end
AddPrefabPostInit("cave", SetupRegrowth)
AddPrefabPostInit("forest", SetupRegrowth)
```

- 调整与验证：实体验证要点——删掉实体后约 2 个游戏日、且玩家不在重生点附近时，在原位 20 半径内出现新实体；timemult 函数被每 29 秒调用一次，勿在其中做重活。

## 常见陷阱与限制

- **以为失败会快速重试：** `RETRY_MULT=0.25` 已定义但实际使用行被注释，FAILED 重排用的是完整 regrowtime 而非 1/4 时长。[常量](../../../raw/current/scripts/components/regrowthmanager.lua#L20) [注释掉的短重试](../../../raw/current/scripts/components/regrowthmanager.lua#L252-L258)
- **prefab 名不匹配即静默失效：** 实体的 `target.prefab` 必须与注册键完全一致（含 `carrot_planted` 这类变体名）；未注册只会打印日志，计时器被丢弃，也不进存档。[匹配点](../../../raw/current/scripts/components/regrowthmanager.lua#L115-L118) [加载过滤](../../../raw/current/scripts/components/regrowthmanager.lua#L301)
- **与 plantregrowth 是两套机制：** [[reference/components/plantregrowth]] 是植物实体自身的后代/恢复组件，按母株计算；本组件在世界侧按“实体被移除”再生，同种资源只能二选一，重复触发会叠加生成。选型前核对该 prefab 实际用了哪套。
- **队列按 FIFO 顺序隐式有序：** 实现假定同类型 regrowtime 相同（[注释](../../../raw/current/scripts/components/regrowthmanager.lua#L254-L255)）；若注册后用非常规手段改短某 timer，顺序不再保证。全部计时器都 CACHE 时本轮直接跳出防死循环（[跳出](../../../raw/current/scripts/components/regrowthmanager.lua#L263-L266)）。
- **位置存档有坐标为 0 的怪癖：** OnSave 把 x/y/z 为 0 的分量存成 nil，读回当 0 处理；依赖原点位置的用法需注意。[存档](../../../raw/current/scripts/components/regrowthmanager.lua#L286-L290)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 regrowthmanager.lua，核验注册表、beginregrowth 协议（standardcomponents 两侧）、落点判定、队列状态机、季节倍率与存读；核对洞穴/森林安装点与 TUNING 常量。未运行游戏；各资源实体的 AddToRegrowthManager 调用抽查了 carrot/cactus 等 11 个 prefab 文件存在性，未逐个审校。

相关：[[reference/components/plantregrowth]]、[[reference/components/forestresourcespawner]]。
