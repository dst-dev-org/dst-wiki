---
title: "hermitcrab_relocation_manager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级隐士蟹（珍珠）整家搬家管理器：按猴女王到猴岛传送门的方位旋转变换 setpiece 布局，并把珍珠的家当分批传送到位。"
aliases: ["隐士蟹搬家", "珍珠搬迁", "hermitcrab_relocation_manager 组件"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/hermitcrab_relocation_manager.lua", "prefabs/forest.lua", "prefabs/hermitcrab_relocation_kit.lua", "components/wagpunk_arena_manager.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hermitcrab_relocation_manager

## 快速判断

- 职责与适用：仅主机的世界级组件（components/hermitcrab_relocation_manager.lua:73 断言），管理珍珠（hermitcrab）及其房屋、肉架、蜂箱、标记等实体的整体传送；布局硬编码为 `PEARLSETPIECE_MONKEYISLAND`（:10-64，注释注明强依赖 monkeyisland_01 静态布局与 hermitcrab_01 实体）。
- 优先使用：想整体搬珍珠家当 → `SetupTeleportingPearlToSetPieceData(setpiecedata, centerx, centerz)`（:315）；查询入口 `GetPearl`／`GetPearlsHouse`／`GetPearlsFishingMarkers`（:354、:358、:367）。
- 重要副作用：传送期间锁 `pearlmovingdata`；全部到位后在 TheWorld 推 `ms_hermitcrab_relocated`（:210），hermitcrab、hermithouse、pearldecorationscore、wagpunk_arena_manager 均监听该事件。
- 修改边界：传送目标布局由调用方给出（格式同 `PEARLSETPIECE_MONKEYISLAND`）；组件只按 prefab 名配对实体与坐标，不自行选址；`CanPearlMove()`（:282-284）为false 期间拒绝新搬家用例（hermitcrab_relocation_kit.lua:91-93 部署门）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `RegisterHermitCrab`／`RegisterPearlEntity` | `(ent) -> nil` | 由世界事件驱动挂接：监听 TheWorld 的 `ms_register_hermitcrab`／`ms_register_pearl_entity`（:476-477）；注册来源见 prefabs/hermitcrab.lua:2107。 |
| `RegisterMonkeyQueen`／`RegisterMonkeyPortal` | `(ent) -> nil` | 监听 `ms_register_monkeyqueen`／`ms_register_monkeyisland_portal`（:479-480）；重复注册置 `self.failed=true`（:434-437、:451-454）。 |
| `SetupTeleportingPearlToSetPieceData` | `(setpiecedata, centerx, centerz) -> nil` | 按 prefab 配对实体并生成 `pearlmovingdata`（坐标=中心+偏移，错峰 delay=counter*0.25+random*0.25，:296-298）；hermitcrab 在场且清醒时不立即启动（:334-347）。 |
| `SetupMovingPearlToMonkeyIsland` | `() -> nil` | 用存储的猴岛中心坐标搬回默认布局（:350-352）；唯一调用方 wagpunk_arena_manager.lua:991-993（珍珠寻宝任务完成时）。 |
| `CanPearlMove` | `() -> boolean` | `pearlmovingdata == nil` 即空闲；hermitcrab_relocation_kit 部署校验用它拒绝搬家中的部署。:282-284 |
| `GetPearl`／`GetPearlsHouse` | `() -> entity\|nil` | 珍珠本体；房屋按 hermithouse2/hermithouse/construction1-3 顺序返回首个命中。:354-365 |
| `GetPearlsFishingMarkers` | `() -> table` | 全部 `hermitcrab_marker_fishing` 实体列表，pearldecorationscore.lua:572 消费。:367-375 |
| `OnSave`/`OnLoad`/`LoadPostPass` | 常规存取 | 存方位角与中心坐标；传送中存 `movingents`（GUID+数据），载入后 `InitiatePearlTeleport` 续传（:381-424）。 |

## 关键流程：旋转与传送管线

1. 方位角：`TryToApplyRotationTransformation`（:146-181）用 monkeyqueen→monkeyportal 的 `math.atan2` 求角（portal 所在 tile 中心，:171-177）；八角扇区映射到布局的 8 种翻转/旋转（:81-127），结果直接原地改写 `PEARLSETPIECE_MONKEYISLAND`。
2. 传送步骤：每实体 `Disappear → Teleport → Appear → Arrive`（:251、:237、:227、:216）——出场景、落位、播放 `hermitcrab_fx_tall/med/small` 特效 12*FRAMES（:197）后回场景；对有特效的实体到点前用 `ClearSpotForRequiredPrefabAtXZ` 清障（:183-186、:221-224）。
3. 完成回调：最后一个实体完成即清 `pearlmovingdata` 并推 `ms_hermitcrab_relocated`（:206-211）；移动中若实体被移除按 deleted 路径收尾（:213-215）。
4. 请求触发：SGhermitcrab.lua:999 推 `ms_hermitcrab_wants_to_teleport`；组件仅在"已布好但未启动"时响应（:482-486，逻辑故意取反）。

## 典型接入

### 查询珍珠及其家当（装饰评分、周边判定）

- 前提与选择：世界为 forest 才有本组件（prefabs/forest.lua:715）；查询方须容忍 nil（如 standardcomponents.lua:1974-1976 的 `IsWithinHermitCrabArea` 先判组件与 house 存在）。
- 生效与撤销：注册依赖原版实体的 `ms_register_*` 世界事件，mod 新增实体若不推事件则不会进入 `pearlsentities`，也就不参与传送。

```lua
local function GetPearlsHouseSafe()
    local mgr = TheWorld.components.hermitcrab_relocation_manager
    return mgr ~= nil and mgr:GetPearlsHouse() or nil
end
```

- 调整与验证：仅读取，无副作用；原版消费方见 hermitcrab_shell.lua:103-104、hermitcrab_teashop.lua:75、shellweaver.lua:129。

## 常见陷阱与限制

- 布局表强绑定 monkeyisland_01：坐标是相对猴岛中心的偏移，旋转又依赖猴女王与传送门实体；任一缺失会打印 ERROR 并停止（:163-169），mod 自建世界须自行保证这些实体。
- `failed` 后不可恢复：重复注册猴女王/传送门（多见于 mod）会置失败并打印错误（:434-437、:451-454），此后 `TryToApplyRotationTransformation` 永远返回 false（:147-153）。
- 同名 prefab 超出布局槽位时静默丢弃并仅在 staging 分支 `c_announce`（:307-309），普通客户端不会收到提示。
- 传送期间 hermitcrab 的 `sg.mem.teleporting` 置 true（:334-335），状态图行为受此标记约束；GOHOME 缓冲动作会被 locomotor 清除（:339-343）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 490 行；挂载点 forest.lua:715、kit 部署门 hermitcrab_relocation_kit.lua:91-93/:148、任务触发 wagpunk_arena_manager.lua:991-993/:1854、事件源 hermitcrab.lua:2107 与 SGhermitcrab.lua:999 逐行核对。
- 游戏运行：未运行。
