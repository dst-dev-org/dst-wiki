---
title: oceanfishingrod
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "海钓竿的服务端抛竿、目标切换、线长张力、收线与捕获，含 tackle／lure 装配与客户端预测。"
aliases: ["海钓竿", "海钓", "抛竿", "收线", "鱼线张力", "浮标", "鱼饵"]
tags: [component, item, equipment, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/oceanfishingrod.lua", "components/oceanfishingrod_replica.lua", "components/oceanfishable.lua", "components/oceanfishinghook.lua", "components/oceanfishingtackle.lua", "prefabs/oceanfishingrod.lua", "prefabs/oceanfishingbobber.lua", "prefabs/oceanfishinglure.lua", "prefabs/oceanfish.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua", "tuning.lua", "stats.lua", "class.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# oceanfishingrod

## 快速判断

- 职责与适用：挂在 `oceanfishingrod` 物品上的服务端组件，负责抛竿投射物、浮标与鱼目标切换、线长／张力、收线与捕获；装配数据来自容器 slot 1 的 bobber 与 slot 2 的 lure，客户端 replica 只同步目标、张力量化与最大抛距。[oceanfishingrod.lua:12](../../../raw/current/scripts/components/oceanfishingrod.lua#L12)、[prefabs/oceanfishingrod.lua:55](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L55)、[oceanfishingrod_replica.lua:4](../../../raw/current/scripts/components/oceanfishingrod_replica.lua#L4
- 优先使用：`OCEAN_FISHING_CAST` → `Cast(fisher, pos)`；`OCEAN_FISHING_REEL` → `Reel()`；`OCEAN_FISHING_CATCH` → `CatchFish()`；中止 → `StopFishing(reason)`；这些方法和字段都在服务端。[actions.lua:1398](../../../raw/current/scripts/actions.lua#L1398)、[actions.lua:1415](../../../raw/current/scripts/actions.lua#L1415)、[actions.lua:1432](../../../raw/current/scripts/actions.lua#L1432
- 重要副作用：`SetTarget` 维护目标与 `oceanfishable` 的双向引用；`StopFishing` 向钓鱼者和目标推 `oceanfishing_stoppedfishing` 并触发 `ondonefishing`（可能销毁容器内单次鱼饵）；成功捕获不推 stopped，见常见陷阱。[oceanfishingrod.lua:260](../../../raw/current/scripts/components/oceanfishingrod.lua#L260)、[oceanfishingrod.lua:363](../../../raw/current/scripts/components/oceanfishingrod.lua#L363
- 修改边界：服务端组件；客户端 `line_tension` 被量化成 `0/1/2`；无存档钩子与移除清理；客户端最大抛距不是精确落点（见抛竿一节）。[oceanfishingrod_replica.lua:18](../../../raw/current/scripts/components/oceanfishingrod_replica.lua#L18)、[oceanfishingrod.lua:153](../../../raw/current/scripts/components/oceanfishingrod.lua#L153

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetDefaults(projectile_prefab, casting_tuning, lure_tuning, lure_setup)` | 四项配置；无返回值 | 复制 casting 配置、设默认 projectile／lure、刷新客户端最大抛距；使用前必须调用。[oceanfishingrod.lua:108](../../../raw/current/scripts/components/oceanfishingrod.lua#L108 |
| `gettackledatafn` | `fun(rod) -> {bobber=ent\|nil, lure=ent\|nil}` | 原版 `GetTackle` 读容器 slot 1／2；缺省时 `Cast` 会因调用 nil 失败。[prefabs/oceanfishingrod.lua:55](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L55 |
| `Cast(fisher, targetpos)` | 返回 `boolean` | 缓存 tackle、开始统计、计算落点、生成投射物并 `SetTarget`，再启动更新和 `oncastfn`。[oceanfishingrod.lua:168](../../../raw/current/scripts/components/oceanfishingrod.lua#L168 |
| `Reel()` | 返回 `boolean`（未断线） | 要求目标有 `oceanfishable` 且钓鱼者有效；收线、更新张力，满足断线条件时 `StopFishing("linesnapped", true)`。[oceanfishingrod.lua:230](../../../raw/current/scripts/components/oceanfishingrod.lua#L230 |
| `CatchFish()` | 无返回值 | 目标有 `oceanfishable` 时把鱼抛向钓鱼者、推 `fishcaught`、`SetTarget(nil)`；不调用 `StopFishing`。[oceanfishingrod.lua:337](../../../raw/current/scripts/components/oceanfishingrod.lua#L337 |
| `StopFishing(reason, lost_tackle)` | `reason` string；`lost_tackle` 可选 truthy | 停更新、调用 `ondonefishing`、记录统计、向钓鱼者与目标推 stopped、清空 `fisher` 与目标。[oceanfishingrod.lua:363](../../../raw/current/scripts/components/oceanfishingrod.lua#L363 |
| `SetTarget(new_target)` | entity／nil | 切换目标；非 projectile 目标才计算 `line_dist`；旧目标被吃或解绑、调用 `onnewtargetfn`、`oceanfishable:SetRod`，并推 `newfishingtarget`。[oceanfishingrod.lua:260](../../../raw/current/scripts/components/oceanfishingrod.lua#L260 |
| `line_dist` / `line_tension` / `line_slack` | number；张力与松弛范围 `0..1` | 张力由“目标距离 - 线长”计算；松弛是目标比线更近时的反向量。[oceanfishingrod.lua:206](../../../raw/current/scripts/components/oceanfishingrod.lua#L206 |
| `GetTensionRating()` / `IsLineTensionHigh/Good/Low()` | 浮点或 boolean | 服务端阈值：高 `>0.80`、好 `>0.10` 且 `<=0.80`、低 `<=0.10`。[oceanfishingrod.lua:190](../../../raw/current/scripts/components/oceanfishingrod.lua#L190)、[tuning.lua:250-251](../../../raw/current/scripts/tuning.lua#L250-L251) |
| `UpdateClientMaxCastDistance()` | 无返回值 | 基础 `dist_max` 加 bobber／lure 的 `dist_max` 写入 replica；`SetDefaults` 与容器 `itemget`／`itemlose` 时调用。[oceanfishingrod.lua:129](../../../raw/current/scripts/components/oceanfishingrod.lua#L129 |
| `GetExtraStaminaDrain()` | number；缺省 `0` | 读 `lure_data.stamina_drain`，供 `oceanfishable` 的体力消耗使用。[oceanfishingrod.lua:381](../../../raw/current/scripts/components/oceanfishingrod.lua#L381)、[oceanfishable.lua:120](../../../raw/current/scripts/components/oceanfishable.lua#L120 |

### tackle 与 lure 数据

- 装配来源：容器 slot 1 为 bobber、slot 2 为 lure；`itemget`／`itemlose` 只刷新客户端最大抛距。原版 hermitcrab 会自动装入 `oceanfishingbobber_ball` 与一条 lure。[prefabs/oceanfishingrod.lua:192](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L192)、[prefabs/hermitcrab.lua:1596](../../../raw/current/scripts/prefabs/hermitcrab.lua#L1596
- bobber 的 `casting_data`：`dist_max`、`dist_min_accuracy`、`dist_max_accuracy`、`max_angle_offset` 与 `projectile_prefab`；基础值 `5 / 0.70 / 1.30 / 40`，bobber 的数在基础值上相加；原版 lure 数据中与 casting 同名的键只有 `dist_max`。[tuning.lua:263-267](../../../raw/current/scripts/tuning.lua#L263-L267)、[oceanfishingrod.lua:139](../../../raw/current/scripts/components/oceanfishingrod.lua#L139
- lure 的 `lure_data`：`charm`、`reel_charm`、`radius`、`style`、`timeofday`、`weather`、`dist_max`、`stamina_drain`；`lure_setup` 另带 `build`／`symbol`／`single_use`／`fns.charm_mod_fn`。[tuning.lua:284](../../../raw/current/scripts/tuning.lua#L284)、[oceanfishinghook.lua:31](../../../raw/current/scripts/components/oceanfishinghook.lua#L31)、[oceanfishingtackle.lua:24](../../../raw/current/scripts/components/oceanfishingtackle.lua#L24

### 抛竿、收线与客户端预测

- 服务端落点：`min(输入距离, dist_max)` 再乘随机精度，最低夹到 `2`；角度在 `max_angle_offset` 内三次方随机偏移；投射物速度 `Lerp(8,16,t)`、重力 `Lerp(-30,-15,t)`，`t` 由 `MAX_CAST_DIST = 16` 归一。[oceanfishingrod.lua:153](../../../raw/current/scripts/components/oceanfishingrod.lua#L153)、[oceanfishingrod.lua:77](../../../raw/current/scripts/components/oceanfishingrod.lua#L77)、[tuning.lua:232](../../../raw/current/scripts/tuning.lua#L232
- 收线与放线：无 `locomotor` 的目标每次固定减少 `0.75`，有 `locomotor` 的鱼按 `0.75 * (1 - line_tension)` 减少；张力 `>=0.92` 且收线前已高张力则断线。张力 `>=0.70` 时鱼会把线拉长（按 `CalcLineUnreelRate` 的 90%），目标超过 `MAX_HOOK_DIST = 26` 直接停止并丢失 tackle。[oceanfishingrod.lua:230](../../../raw/current/scripts/components/oceanfishingrod.lua#L230)、[oceanfishingrod.lua:311](../../../raw/current/scripts/components/oceanfishingrod.lua#L311)、[oceanfishingrod.lua:318](../../../raw/current/scripts/components/oceanfishingrod.lua#L318
- 客户端值：replica 的 `_max_cast_dist` 是 `net_smallbyte`，只由“基础 `dist_max` + bobber `dist_max` + lure `dist_max`”取整得到，不含随机精度；服务端实际落点还会乘 `dist_min_accuracy..dist_max_accuracy`，显示半径只能当瞄准上限。[oceanfishingrod_replica.lua:36](../../../raw/current/scripts/components/oceanfishingrod_replica.lua#L36)、[oceanfishingrod.lua:129](../../../raw/current/scripts/components/oceanfishingrod.lua#L129
- 客户端张力：replica 按 `0.10`／`0.80` 量化成 `0/1/2`，SG 只用它切换动作与音效；精确张力只在服务端。[oceanfishingrod_replica.lua:18](../../../raw/current/scripts/components/oceanfishingrod_replica.lua#L18)、[SGwilson.lua:14055](../../../raw/current/scripts/stategraphs/SGwilson.lua#L14055
- 统计：开始记录 bobber／lure prefab 与 `cast_time`，上钩时记录 phase 与等待时长，结束时补 `result`、目标 prefab、`isafish`、重量与 `catch_time`，由 `Stats.PushMetricsEvent("fishing", ...)` 上报。[oceanfishingrod.lua:45](../../../raw/current/scripts/components/oceanfishingrod.lua#L45)、[oceanfishingrod.lua:61](../../../raw/current/scripts/components/oceanfishingrod.lua#L61)、[stats.lua:298](../../../raw/current/scripts/stats.lua#L298

## 典型接入

### 提高原版海钓竿的基础抛距（prefab postinit）

- 前提与选择：写入目标模组 `modmain.lua`，仅服务端执行；复用原版 `oceanfishingrod` prefab、容器、投射物与资源，不新增 prefab。[prefabs/oceanfishingrod.lua:138](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L138
- 生效与撤销：新生成及读档重建的海钓竿都会经过 `fn`，postinit 重新 `SetDefaults`；停用模组后恢复基础 `dist_max=5`、精度 `0.70..1.30`，tackle 加成仍按实际 bobber／lure 叠加。[prefabs/oceanfishingrod.lua:182](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L182
- 原版依据：prefab 用 `OCEANFISHING_TACKLE.BASE` 与 `OCEANFISHING_LURE.HOOK` 调用 `SetDefaults`。[tuning.lua:263-267](../../../raw/current/scripts/tuning.lua#L263-L267)、[oceanfishingrod.lua:108](../../../raw/current/scripts/components/oceanfishingrod.lua#L108

```lua
-- modmain.lua（服务端逻辑）
AddPrefabPostInit("oceanfishingrod", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local rod = inst.components.oceanfishingrod
    if rod == nil then
        return
    end
    rod:SetDefaults(
        rod.default_projectile_prefab,
        {
            dist_max = 7,
            dist_min_accuracy = 0.80,
            dist_max_accuracy = 1.20,
            max_angle_offset = 40,
        },
        GLOBAL.TUNING.OCEANFISHING_LURE.HOOK,
        rod.default_lure_setup
    )
end)
```

- 调整与验证：`dist_max` 为世界单位，`dist_*_accuracy` 是乘数，`max_angle_offset` 为角度；`SetDefaults` 会重置 casting／lure 数据，因此要传完整四参。示例仅做 Lua 语法检查，未运行游戏；客户端最大抛距由 `SetDefaults` 内的 `UpdateClientMaxCastDistance` 刷新，若在容器装入前调用则只含基础值。

### 原版接入点与分工

- 程序化捕获：sharkboimanager 对 `oceanfishing_catchable` 且非 `fishinghook` 的目标直接 `CatchFish()`；落点失败时 bobber 投射物调用 `StopFishing("badcast")`。[components/sharkboimanager.lua:405](../../../raw/current/scripts/components/sharkboimanager.lua#L405)、[prefabs/oceanfishingbobber.lua:60](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L60
- 与池塘钓的分工：池塘钓以 `fishable` 池塘为目标、按鱼量等待；本页以浮标和鱼实体为目标、靠容器 tackle 与线长张力。两者状态机不共享，见 [[reference/components/fishingrod]]；不要把 `target`／`hookedfish` 与 `line_dist`／`line_tension` 混用。

## 常见陷阱与限制

- **不调用 `SetDefaults` 就使用**：`casting_data`、`lure_data`、`projectile_prefab` 都由它初始化；未调用时 `Cast`、`GetExtraStaminaDrain` 会读 nil，自建宿主还要提供 `gettackledatafn`。[oceanfishingrod.lua:108](../../../raw/current/scripts/components/oceanfishingrod.lua#L108)、[oceanfishingrod.lua:381](../../../raw/current/scripts/components/oceanfishingrod.lua#L381
- **成功捕获没有 stopped 事件**：`CatchFish` 只推 `fishcaught` 并清目标，不调用 `StopFishing`；等待 stopped 的消费者收不到成功结果，`fisher` 字段会保留到下次 `Cast`／`StopFishing`。[oceanfishingrod.lua:337](../../../raw/current/scripts/components/oceanfishingrod.lua#L337
- **reason 决定脱钩损失**：宿主 `OnDoneFishing` 在 `lost_tackle` 为真时销毁容器内容；`linesnapped`／`toofaraway`／带鱼时的 `interrupted` 传 `true`，`reeledin` 与 `badcast` 不传。[prefabs/oceanfishingrod.lua:108](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L108)、[oceanfishingrod.lua:303](../../../raw/current/scripts/components/oceanfishingrod.lua#L303
- **客户端最大抛距不是落点**：replica 只累加 `dist_max`，显示半径只当瞄准上限，见上文「抛竿、收线与客户端预测」。
- **张力方向不对称**：目标朝钓鱼者游来时最高张力被压到 `0.80`，不会进入高张力；目标比线更近时产生 `line_slack`，非浮标目标松弛满会以 `linetooloose` 停止。[oceanfishingrod.lua:206](../../../raw/current/scripts/components/oceanfishingrod.lua#L206)、[tuning.lua:250-251](../../../raw/current/scripts/tuning.lua#L250-L251)
- **单次鱼饵在“鱼接手”时消耗**：`OnHookedSomething` 在目标不是 projectile 且不是 `oceanfishinghook` 时移除 `single_use` tackle；浮标落水与浮标被咬阶段不消耗。[prefabs/oceanfishingrod.lua:118](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L118)、[oceanfishingtackle.lua:29](../../../raw/current/scripts/components/oceanfishingtackle.lua#L29
- **浮标被吃掉会删除目标**：`SetTarget` 在新目标上钩时对旧 `oceanfishable` 调 `WasEatenByA`，旧实体随即 `Remove()`；不要把 `target` 缓存到停止之后。[oceanfishingrod.lua:274](../../../raw/current/scripts/components/oceanfishingrod.lua#L274)、[oceanfishable.lua:44](../../../raw/current/scripts/components/oceanfishable.lua#L44
- **移除组件与存档**：本组件没有 `OnRemoveFromEntity`，`RemoveComponent` 不会调用 `ondonefishing` 或 `SetTarget(nil)`；组件也不保存线长／张力，浮标实体 `persists=false`，读档后不会恢复进行中的海钓。[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670)、[prefabs/oceanfishingbobber.lua:223](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L223

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 8 处（actions.lua OCEAN_FISHING 三入口、tuning.lua 张力／tackle／lure 数值、SGwilson 张力表现）；结论修订：无，仅锚点迁移。依据：枢纽 diff 审阅＋变更清单命中 actions.lua、SGwilson.lua、tuning.lua、entityscript.lua、prefabs/hermitcrab.lua（命中区域逐行核对均未漂移）。
- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖组件构造与宿主、抛竿／收线／捕获、tackle 与 lure 数据、客户端 replica 预测、海钓事件与统计、失败与中断路径。未运行游戏。
- 结构检查：2026-10-21，`validate_page.py` 通过；源码链接 65 处 0 error／0 warning（`--strict`）；示例 `parser_probe --syntax-only` 通过。
- 游戏运行：未运行；多人同步、断线／重连、实际落点分布与统计上报未覆盖。

相关：[[reference/components/fishingrod|池塘钓竿]]；[[reference/components/complexprojectile|抛射物]]；[[reference/components/reticule|瞄准圈]]；[[reference/components/container|容器槽位]]。
