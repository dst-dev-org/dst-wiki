---
title: oceanfishable
created: 2026-10-21
updated: "2026-10-28"
type: component
summary: 海钓中"被钓对象"的钓竿互链、体力与挣扎、收线抵抗与可捕获判定
aliases: ["海钓鱼体力", "收线张力", "oceanfishing 被钓对象", "挣扎状态"]
tags: [component, state-machine, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/oceanfishable.lua", "components/oceanfishingrod.lua", "prefabs/oceanfish.lua", "prefabs/oceanfishingbobber.lua", "prefabs/oceanfishableflotsam.lua", "brains/oceanfishbrain.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# oceanfishable

## 快速判断

- 职责与适用：海钓里"被钓对象"一侧——与钓竿互链、体力／挣扎状态、反拉收线速度、可捕获标签；钓竿侧张力状态机见 [[reference/components/oceanfishingrod]]。
- 优先使用：由钓竿侧 `rod.components.oceanfishingrod:SetTarget(obj)` 建立关系；对象侧只读 `GetRod`、`IsStruggling`，别手写 `rod` 字段。
- 重要副作用：`SetRod` 会反向调 `rod:SetTarget`，并在对象上增删 `oceanfishing_catchable`；钩住期间每帧改 `locomotor.walkspeed`／`runspeed`。
- 修改边界：服务端组件；没挂竿时不 `OnUpdate`、不加可捕获标签；客户端只读标签／复制品，不能靠它驱动行为。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetRod(rod)` | 返回布尔（同一竿重复设置时无返回值） | 清旧竿（停更新、撤 `onremove` 监听、`prev_rod...:SetTarget(nil)`、`onsetrodfn(inst,nil)`），接新竿（监听 `onremove`、`rod...:SetTarget(inst)`、启动更新、`onsetrodfn(inst,rod)`）[SetRod](../../../raw/current/scripts/components/oceanfishable.lua#L67-L95) |
| `GetRod()` | 实体或 `nil` | 只读 [GetRod](../../../raw/current/scripts/components/oceanfishable.lua#L97-L99) |
| `StrugglingSetup(walk_speed, run_speed, stamina_def)` | 数值＋体力表 | 写 `max_walk_speed`／`max_run_speed`／`stamina_def`，`stamina=1`，清挣扎计时 [StrugglingSetup](../../../raw/current/scripts/components/oceanfishable.lua#L56-L65) |
| `catch_distance` | number，默认 `4` | "够近可捕获"的判定距离；原版鱼与浮标都用 2.5 [IsCloseEnoughToCatch](../../../raw/current/scripts/components/oceanfishable.lua#L125-L132) |
| `OnUpdate(dt)` | 有竿时逐帧 | 张力高扣体力／低回体力、挣扎计时翻转、刷新可捕获标签、重算速度 [OnUpdate](../../../raw/current/scripts/components/oceanfishable.lua#L134-L157) |
| `CalcStaminaDrainRate()` | 每秒（负值） | `-(stamina_def.drain_rate + rod:GetExtraStaminaDrain())` [CalcStaminaDrainRate](../../../raw/current/scripts/components/oceanfishable.lua#L120-L123) |
| `CalcLineUnreelRate(rod)` | 距离／秒 | 先给 `overrideunreelratefn`，否则仅在挣扎时按 `runspeed × 角度系数` 反拉 [CalcLineUnreelRate](../../../raw/current/scripts/components/oceanfishable.lua#L178-L189) |
| `ResetStruggling()`／`UpdateStruggleState()`／`IsStruggling()` | 无返回／无返回／布尔 | 置 `stamina=1` 并重排计时；AI 每 tick 调 `UpdateStruggleState` 后再读 `IsStruggling` [ResetStruggling](../../../raw/current/scripts/components/oceanfishable.lua#L159-L176) |
| `MakeProjectile()` | 实体 | `makeprojectilefn` 的返回值，缺省返回自身 [MakeProjectile](../../../raw/current/scripts/components/oceanfishable.lua#L52-L54) |
| `onsetrodfn`／`onreelinginfn`／`onreelinginpstfn`／`oneatenfn` | 函数字段 | 挂／摘竿、收线前后、被吞时回调；宿主按需赋值，组件只转发 [回调外壳](../../../raw/current/scripts/components/oceanfishable.lua#L32-L50) |

## 典型接入

### 调整原版鱼种的体力与捕获距离

- 前提与选择：目标文件 `modmain.lua`；`AddPrefabPostInit("oceanfish_small_1", fn)` 命中已装配宿主（[oceanfish 装配](../../../raw/current/scripts/prefabs/oceanfish.lua#L347-L352) 里已 `StrugglingSetup`）。组件只在服务端，先判 `TheWorld.ismastersim`。
- 单位与生效：`drain_rate`／`recover_rate` 是每秒体力比例；`catch_distance` 为世界单位，原版鱼 2.5（`FISHING_CATCH_DIST`）。改 `stamina_def` 必须换新表，否则改的是所有鱼共用的 `TUNING.OCEANFISH.FISHABLE_STAMINA`。
- 生效与撤销：新表在该鱼下一次 `ResetStruggling`（首次收线）后生效；撤销＝去掉 postinit 并让鱼重新进入战斗，已钩住的鱼保留旧表。
- 原版依据：[FISHABLE_STAMINA 默认值](../../../raw/current/scripts/tuning.lua#L5084-L5090)、[收线距离常量](../../../raw/current/scripts/tuning.lua#L243-L248)。

```lua
-- modmain.lua：单独放宽某鱼种的体力与捕获判定（服务端）
local function TweakFish(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local oceanfishable = inst.components.oceanfishable
    if oceanfishable == nil then
        return
    end
    oceanfishable.stamina_def = {
        drain_rate = 0.03,   -- 每秒扣体力
        recover_rate = 0.2,  -- 每秒回体力
        struggle_times = { low = 3, high = 8, r_low = 1, r_high = 1 },
        tired_times = { low = 4, high = 2, r_low = 1, r_high = 1 },
    }
    oceanfishable.catch_distance = GLOBAL.TUNING.OCEAN_FISHING.FISHING_CATCH_DIST + 1
end

AddPrefabPostInit("oceanfish_small_1", TweakFish)
```

- 调整与验证：诱饵的额外消耗走 `GetExtraStaminaDrain`（[oceanfishingrod](../../../raw/current/scripts/components/oceanfishingrod.lua#L381-L383)）；本例未运行，仅语法检查。

### 原版协作链

- 落水：`projectile` 生成 `_floater` 后 `SetRod(rod)`，钓竿反向 `SetTarget(floater)` [bobber 落水](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L52-L55)、[floater 装配](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L228-L235)。
- 咬钩交接：鱼 brain 发 `ACTIONS.EAT`，[EAT.fn](../../../raw/current/scripts/actions.lua#L806-L807) 把浮标的竿交给鱼 → `rod:SetTarget(fish)` → 旧目标 `WasEatenByA` 被移除（[SetTarget](../../../raw/current/scripts/components/oceanfishingrod.lua#L260-L301)）。
- 钩住：`onsetrodfn` 给鱼加 `partiallyhooked`（[oceanfish OnSetRod](../../../raw/current/scripts/prefabs/oceanfish.lua#L165-L173)）；首次收线 `OnReelingIn` 才 `ResetStruggling`（[oceanfish OnReelingIn](../../../raw/current/scripts/prefabs/oceanfish.lua#L151-L163)）。
- 收线：钓竿在 `line_tension >= START_UNREELING_TENSION` 时调 `CalcLineUnreelRate` 放线（[OnUpdate](../../../raw/current/scripts/components/oceanfishingrod.lua#L318-L323)）；捕获走 `MakeProjectile`＋`catch_distance`（[CatchFish](../../../raw/current/scripts/components/oceanfishingrod.lua#L337-L361)）。
- 非鱼宿主：残骸自己找 `fishinghook` 接管钓竿（[flotsam](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L68-L76)、[装配](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L265-L271)）。

## 常见陷阱与限制

- 竿对象必须有 `oceanfishingrod`：`SetRod` 直接调 `rod.components.oceanfishingrod:SetTarget(...)`（[SetRod](../../../raw/current/scripts/components/oceanfishable.lua#L67-L95)），缺组件会索引 `nil`。
- 没 `StrugglingSetup` 就设 `stamina`：张力高时 `CalcStaminaDrainRate` 索引 `stamina_def` 报错；它同时决定 `UpdateRunSpeed` 的速度上限基数，缺了则不改速度。
- 无 `locomotor`：`UpdateRunSpeed` 整体跳过、`CalcLineUnreelRate` 返回 0，鱼被直线拖回且不挣扎。
- `catch_distance` 与钓竿距离不匹配：可捕获标签按本值判定，`CatchFish` 落点也用它；调大能在更远处捕获，但钓竿仍在 `MAX_HOOK_DIST`(26) 与 `MAX_CAST_DIST`(16) 处停钓。
- 双向引用陷阱：`SetRod`↔`SetTarget` 互相调用，靠组件 `self.rod ~= rod` 与钓竿 `self.target ~= new_target` 终止；外部直接 `SetRod` 会把钓竿从原目标挤走，原目标被 `WasEatenByA` 移除。
- 清理：`OnRemoveFromEntity` 只做 `SetRod(nil)`；钓竿被删时靠 `onremove` 监听回到 `SetRod(nil)`（[监听](../../../raw/current/scripts/components/oceanfishable.lua#L82-L84)），所以不要自己先清 `rod.components`。
- 不挂竿就没有标签：`oceanfishing_catchable` 只在 `OnUpdate` 里刷新，客户端动作筛选依赖它（[componentactions](../../../raw/current/scripts/componentactions.lua#L86-L99)）。

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 3 处（tuning +14／+1、actions +3，区间文本逐段比对一致）；结论修订：无（OCEANFISH.FISHABLE_STAMINA／FISHING_CATCH_DIST 键值未变）。
- 依据：tuning 键级 hunk 地图＋变更清单命中 tuning.lua、actions.lua、componentactions.lua。

- 源码审阅：2026-10-21，steam-buildid:25643519，逐行读 `components/oceanfishable.lua`（209 行），并与 `oceanfishingrod.lua`、`prefabs/oceanfish.lua`、`prefabs/oceanfishingbobber.lua`、`prefabs/oceanfishableflotsam.lua`、`brains/oceanfishbrain.lua` 的调用点核对；覆盖双链、体力与挣扎、反拉速率、捕获距离、标签刷新，未覆盖鱿鱼／龙虾的 NPC 钓鱼与运行期行为。
- 游戏运行：未运行。
- 结构检查：validate_page.py 本页 0 error／0 warning（95 行，低于 180 行／14 KiB 上限）；validate_source_links.py --strict 0 error／0 warning，本页源码引用 27 处（唯一目标 26 个，覆盖 8 个源文件）。
