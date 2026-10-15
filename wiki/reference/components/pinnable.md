---
title: "pinnable"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "为玩家等已有受困状态图的宿主管理黏液受困、受击加速脱困和他人解救。"
aliases: ["pinnable", "黏液受困", "唾液定身", "解救受困玩家", "pinned", "UNPIN"]
tags: [component, lifecycle, state-machine, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/pinnable.lua", "prefabs/player_common.lua", "prefabs/spat.lua", "brains/spatbrain.lua", "stategraphs/SGwilson.lua", "componentactions.lua", "actions.lua", "components/rider.lua", "prefabs/woodie.lua", "entityscript.lua", "tuning.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# pinnable

## 快速判断

- 职责与适用：挂在**被黏住的目标**，保存受困状态、黏液符号与自然／受击磨损；玩家公共工厂已有组件，`SGwilson` 消费事件实现玩家操作限制。[player_common.lua:2815](../../../raw/current/scripts/prefabs/player_common.lua#L2815 [pinnable.lua:47](../../../raw/current/scripts/components/pinnable.lua#L47
- 优先使用：沿用玩家原有 `pinnable`／SG；施加前按需求检查 `IsValidPinTarget()`，施加用 `Stick()`，主动解除用 `Unstick()`。原版钢羊脑用资格查询选择吐痰，弹体命中直接 `Stick()`。[spatbrain.lua:57](../../../raw/current/scripts/brains/spatbrain.lua#L57 [spat.lua:295](../../../raw/current/scripts/prefabs/spat.lua#L295
- 重要副作用：受困增加 `pinned` 标签、按 `"pinned"` 原因停脑、清战斗目标和停止 locomotor；组件本身不持续锁定移动，玩家 SG 才负责关闭控制器、隐藏物品栏。[pinnable.lua:110](../../../raw/current/scripts/components/pinnable.lua#L110 [SGwilson.lua:19222](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19222
- 修改边界：在主模拟端修改；向任意生物补组件不能自动得到对应动画／SG。需要多来源禁移动但不采用黏液表现时参见 [[reference/components/rooted]]，伤害窗口触发事件参见 [[reference/components/stunnable]]。

## 关键接口

| 接口／字段 | 参数、返回或单位 | 条件、作用与证据 |
|---|---|---|
| `canbepinned` | 初始 `true` | `Stick` 的资格开关；骑乘、伍迪变身会修改它，不应在通用 PostInit 中强制覆盖。[pinnable.lua:50](../../../raw/current/scripts/components/pinnable.lua#L50 [rider.lua:147](../../../raw/current/scripts/components/rider.lua#L147 [woodie.lua:1264](../../../raw/current/scripts/prefabs/woodie.lua#L1264 |
| `IsValidPinTarget()` | 布尔值 | 要求开关开启、当前未受困、`GetTime() > last_unstuck_time + 1.5`；不是完整的可见／存活检查。[pinnable.lua:97](../../../raw/current/scripts/components/pinnable.lua#L97 [tuning.lua:437](../../../raw/current/scripts/tuning.lua#L437 |
| `Stick(goo_build, splashfxlist)` | 可省略的 build 名／FX Prefab 名列表；无返回值 | 内部只检查开关、实体可见、无 health 或未死亡；**不调用资格查询**。首次改变状态才记录起点、保存视觉配置并发 `pinned`；重复调用不刷新时长。[pinnable.lua:110](../../../raw/current/scripts/components/pinnable.lua#L110 |
| `SetDefaultWearOffTime(seconds)` | 正数，游戏秒；原默认 10 | 直接修改 `wearofftime`，应在施加前配置；0 会让比例计算除以零。受困后的修改也会影响当前比例。[pinnable.lua:77](../../../raw/current/scripts/components/pinnable.lua#L77 [pinnable.lua:156](../../../raw/current/scripts/components/pinnable.lua#L156 |
| `IsStuck()`／`RemainingRatio()` | 布尔值／未钳制比例 | 比例为 `(wearofftime - 已过秒数 - 受击次数 × 2) / wearofftime`；`attacked` 每次计一次，不按伤害数值。[pinnable.lua:35](../../../raw/current/scripts/components/pinnable.lua#L35 [pinnable.lua:156](../../../raw/current/scripts/components/pinnable.lua#L156 [tuning.lua:435](../../../raw/current/scripts/tuning.lua#L435 |
| `Unstick()` | 无返回值 | 仅已受困时清状态／符号、取消任务、记录解除时间、恢复对应停脑原因并发 `onunpin`；不恢复旧战斗目标。[pinnable.lua:162](../../../raw/current/scripts/components/pinnable.lua#L162 [entityscript.lua:1104](../../../raw/current/scripts/entityscript.lua#L1104 |

自然解除链为定时 `UpdateStuckStatus()` → 比例耗尽 → `Unstick()`；`attacked` 也更新比例。外部输入事件 **`unpinned`** 和 `death` 都调用 `Unstick()`；输出事件是 **`onunpin`**，二者不能混写。移除组件会注销三个监听并调用 `Unstick()`。[pinnable.lua:19](../../../raw/current/scripts/components/pinnable.lua#L19 [pinnable.lua:61](../../../raw/current/scripts/components/pinnable.lua#L61 [pinnable.lua:70](../../../raw/current/scripts/components/pinnable.lua#L70 [pinnable.lua:137](../../../raw/current/scripts/components/pinnable.lua#L137

玩家 `pinned` 事件进入 `pinned_pre`，再进循环状态；`onunpin` 进入 `breakfree`。`pinned_pre` 还会解冻、停止重物搬运并清缓冲动作；离开状态恢复物品栏／控制器。受击会走 `pinned_hit`。不能只模拟标签来复现这些行为。[SGwilson.lua:2346](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2346 [SGwilson.lua:19222](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19222 [SGwilson.lua:19343](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19343

（26148036 核验：受困三态（pinned_pre／pinned／pinned_hit）退出新增 `isstillpinned` 复查，未转往其他受困状态时兜底调用 `pinnable:Unstick()`，旧版仅恢复物品栏／控制器；`pinned_pre` 另按 `goo_vines` 受困 build 走 distress 表现——[SGwilson.lua:19222](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19222)、[:19243](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19243)）

## 典型接入

### 缩短已有玩家宿主的黏液自然受困时间

以下为已启用模组的完整 `modmain.lua` 内容，直接加载，不新增 Prefab、动画、贴图或 `PrefabFiles`；沿用玩家公共工厂与原钢羊黏液资源。`AddPlayerPostInit` 由模组环境提供，游戏全局通过 `GLOBAL` 读取。[modutil.lua:586](../../../raw/current/scripts/modutil.lua#L586 [player_common.lua:2815](../../../raw/current/scripts/prefabs/player_common.lua#L2815

```lua
local WEAR_OFF_SECONDS = 6

AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local pinnable = inst.components.pinnable
    if pinnable ~= nil then
        pinnable:SetDefaultWearOffTime(WEAR_OFF_SECONDS)
    end
end)
```

替换点只有正数秒值 `6`。它改变所有拥有该组件的玩家以后受困的磨损基数；受击仍额外扣除原版 2 秒，资格、骑乘／变身、命中和解救流程沿用原版。它不立即黏住玩家；实际触发可用钢羊弹体命中。卸载模组后新建实例恢复原默认，已运行实例需重新构造或另行恢复配置。[spat.lua:313](../../../raw/current/scripts/prefabs/spat.lua#L313 [pinnable.lua:156](../../../raw/current/scripts/components/pinnable.lua#L156

原版他人解救：组件动作发现施救者未 `pinned`、目标已 `pinned` 且非自身时提供 `UNPIN`；玩家用短动作执行，动作函数重新检查目标实际受困，再发送 `unpinned`。这不是受困者的自救动作。[componentactions.lua:650](../../../raw/current/scripts/componentactions.lua#L650 [SGwilson.lua:1431](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1431 [actions.lua:3725](../../../raw/current/scripts/actions.lua#L3725

## 常见陷阱与限制

- **把恢复宽限当作 `Stick` 内置免疫**：直接 `Stick()` 可越过 1.5 秒宽限；需要遵守宽限的自定义命中入口须先查 `IsValidPinTarget()`，并自行满足命中／阵营条件。查询本身也不检查死亡／可见性。[pinnable.lua:97](../../../raw/current/scripts/components/pinnable.lua#L97
- **重复 `Stick` 延长效果**：已有受困时不会重置起点、攻击计数、黏液 build 或 FX 列表，只再次停脑／停移动／清目标。[pinnable.lua:110](../../../raw/current/scripts/components/pinnable.lua#L110
- **只改 `canbepinned=false` 就期望立即脱困**：字段不是状态撤销接口；主动解除仍用 `Unstick()`。SG 对排队中的 `pinned` 另有开关复查，但不能当所有宿主的统一兜底。[SGwilson.lua:2346](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2346
- **移植到任意宿主或自定义黏液**：需有相应 SG／动画、`swap_goo` 和玩家 SG 使用的 `swap_goosplat` 符号及正确资源；自定义 `splashfxlist` 应为非空有效 Prefab 列表。默认资源名与覆盖点见实现，组件不会注册资源。[pinnable.lua:1](../../../raw/current/scripts/components/pinnable.lua#L1 [pinnable.lua:81](../../../raw/current/scripts/components/pinnable.lua#L81 [SGwilson.lua:19241](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19241
- **期待保存受困进度**：组件没有 `OnSave`／`OnLoad`，不保存受困时间、受击计数或配置；宿主其他机制的保存不能据此推导。组件移除清理与实体删除不同，实体删除还统一取消该实体任务。[pinnable.lua:70](../../../raw/current/scripts/components/pinnable.lua#L70 [entityscript.lua:1730](../../../raw/current/scripts/entityscript.lua#L1730

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 变更文件（components/pinnable.lua、player_common.lua、rider.lua、woodie.lua、stategraphs/SGwilson.lua、entityscript.lua、tuning.lua、componentactions.lua、actions.lua，更新报告 R-01/R-04/R-08）逐条核验，锚点重定位 25 处、结论修订 1 处（SGwilson 受困三态退出兜底 Unstick，见正文"26148036 核验"标注）；pinnable.lua 本体变化为 SpawnShatterFX 空列表保护与新增 GetTimeStuck()，rider.lua 变更为骑乘伤害重定向新增装备攻击者豁免，均不影响本页结论；未覆盖：运行验证。

- 源码审阅：2026-10-21，Steam buildid 25643519；完整阅读 `components/pinnable.lua`，核验默认常量、玩家安装与 SG 施加／受击／解除、钢羊脑／弹体、UNPIN 动作与动作收集、骑乘／伍迪资格和停脑／任务清理。原始脚本只读。
- 示例范围：仅配置原玩家宿主，不新增 SG 或控制协议；完整 `modmain.lua` 经 Lua 5.1 语法检查。页面元数据／大小／本地链接与源码行锚点检查通过。
- 游戏运行：未运行。钢羊命中、他人解救、死亡、骑乘／变身交界、客户端预测、保存重载与其他模组冲突仍需实测；未穷举其他宿主 SG。
