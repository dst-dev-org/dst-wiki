---
title: hauntable
created: 2026-10-21
updated: "2026-10-28"
type: component
summary: 选择幽灵作祟 helper、保留宿主反应，并判断动作资格、冷却与恐慌清理边界
aliases: [幽灵作祟, 作祟冷却, MakeHauntable, AddHauntableCustomReaction, 恐慌反应]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/hauntable.lua, standardcomponents.lua, actions.lua, componentactions.lua, prefabs/flint.lua, brains/pigbrain.lua, stategraphs/SGwilsonghost.lua, tuning.lua, modutil.lua]
reliability: verified
validation: source-reviewed
status: active
---

# hauntable

## 快速判断

- 职责与适用：目标实体接受幽灵作祟，管理 `haunted` 标签、冷却和着色；具体效果由 `onhaunt(inst, haunter)` 决定。追加反应优先用下文组合 helper，保留宿主原回调。[组件构造](../../../raw/current/scripts/components/hauntable.lua#L9-L49)。
- 优先使用：只调间隔改现有 `cooldown`；新目标按“按效果选工厂”选择。**工厂名不保证对应效果仍启用**。
- 重要副作用：回调失败也可能进入 `haunted` 状态及冷却，但失败分支不再推送 `haunted` 事件（26148036 核验：C-17，事件仅在 onhaunt 成功后触发）；动作成功与效果成功不同，见下文执行链。
- 修改边界：燧石在主模拟端添加 helper；客户端动作收集只展示候选，不能替代服务端检查。模组修改也放主模拟端。[宿主](../../../raw/current/scripts/prefabs/flint.lua#L36-L57)、[动作收集](../../../raw/current/scripts/componentactions.lua#L410-L413)。

## 关键接口

### 动作资格与实际执行

`HAUNT` 声明 `ghost_valid=true`、`ghost_exclusive=true`；幽灵 SG 把它映射到 `haunt_pre`，在第 15 帧执行缓冲动作。服务端动作再次要求目标有效、非 limbo、有组件、未被持有，且没有 `haunted` 或 `catchable` 标签。`DoHaunt` 本身不重复这些资格检查；脚本直接调用会绕过动作门禁，不能据此判断玩家当前能否作祟。[动作声明](../../../raw/current/scripts/actions.lua#L462)、[SG 映射](../../../raw/current/scripts/stategraphs/SGwilsonghost.lua#L20)、[执行时机](../../../raw/current/scripts/stategraphs/SGwilsonghost.lua#L207-L221)、[服务端门禁](../../../raw/current/scripts/actions.lua#L3153-L3163)。

动作先向施作者推 `haunt`，数据 `{ target = target }`；`DoHaunt` 在 `onhaunt` 返回成功时才向目标推无数据的 `haunted`（26148036 核验：C-17，旧版无论成败都在函数末尾推送），不返回成功值；动作随后直接返回真。存在 `itemmimic` 时提前调用 `TurnEvil(doer)` 并返回，跳过普通回调与 `haunted` 事件，变身细节不在本页覆盖。[DoHaunt](../../../raw/current/scripts/components/hauntable.lua#L82-L115)。

| 接口／字段 | 接入约定与副作用 |
|---|---|
| `SetOnHauntFn(fn)` | 替换单一回调；返回真值走成功分支，`false/nil` 走失败分支。需要保留原效果时使用下述组合 helper。[setter](../../../raw/current/scripts/components/hauntable.lua#L35-L39) |
| `cooldown` | 秒；裸组件为 `nil`，成功默认 5 秒、失败默认 3 秒；工厂常预设为 3 秒，使两分支都用同一间隔。[计时赋值](../../../raw/current/scripts/components/hauntable.lua#L88-L111)、[常量](../../../raw/current/scripts/tuning.lua#L3043-L3047) |
| `SetHauntValue(val)` | 非假值才赋值，并置 `no_wipe_value=true`。普通成功且施作者非空时，未置该标志的值会清空。特殊值 `HAUNT_INSTANT_REZ` 会向 `playerghost` 推 `respawnfromghost`，不能把任意数值当作通用奖励。[赋值](../../../raw/current/scripts/components/hauntable.lua#L46-L50)、[消费](../../../raw/current/scripts/components/hauntable.lua#L88-L98) |
| `Panic(panictime)` | 默认 3 秒；以 `max` 延长现有恐慌和冷却，置 `haunted/panic` 并启动更新；它本身不调用 `onhaunt` 或 shader。需要 Brain 消费 `panic` 才会逃跑。[组件](../../../raw/current/scripts/components/hauntable.lua#L52-L59)、[猪人消费者](../../../raw/current/scripts/brains/pigbrain.lua#L382-L388) |
| `StopHaunt()` / `SetOnUnHauntFn(fn)` | 清冷却、`haunted` 与 shader，再调用停止更新判定；回调参数为宿主。不会立即清 `panic/panictimer`。`OnUpdate` 在冷却到期后调用它，恐慌另行倒计时。[停止与更新](../../../raw/current/scripts/components/hauntable.lua#L144-L178) |

### 按效果选工厂

以下参数顺序以源码为准，`inst` 均为目标，概率为 0–1；不是可任意叠加的效果列表，多数工厂会重新安装 `onhaunt`。

| 需求 | 推荐入口与本版本实际行为 |
|---|---|
| 仅接受作祟 | `MakeHauntable(inst, cooldown, haunt_value)` 只补组件、设置冷却和固定值，不改已有 `onhaunt`；裸组件默认回调返回真。[基础](../../../raw/current/scripts/standardcomponents.lua#L1047-L1051)、[默认回调](../../../raw/current/scripts/components/hauntable.lua#L1-L3) |
| 发射地面物品 | `MakeHauntableLaunch(inst, chance, speed, cooldown, haunt_value)` 成功发射后置值、清已落地标记、返回真。燧石实际用 `LaunchAndSmash`，其中 smash 分支已注释，保留发射。[Launch](../../../raw/current/scripts/standardcomponents.lua#L1053-L1068)、[LaunchAndSmash](../../../raw/current/scripts/standardcomponents.lua#L1071-L1104) |
| 冻结现有生物 | `MakeHauntableFreeze(inst, chance, cooldown, haunt_value)` 要求已有 `freezable` 且未冻住；成功增加寒冷并设置较长冷却。不会替目标创建 `freezable`。[Freeze](../../../raw/current/scripts/standardcomponents.lua#L1155-L1170) |
| 恐慌 | `MakeHauntablePanic(inst, panictime, chance, cooldown, haunt_value)` 尝试前唤醒 sleeper；成功置恐慌计时，Brain 决定移动。`PanicAndIgnite` 的点燃分支已注释。[Panic 家族](../../../raw/current/scripts/standardcomponents.lua#L1339-L1381)、[Brain](../../../raw/current/scripts/brains/pigbrain.lua#L386-L388) |
| 换 Prefab | `MakeHauntableChangePrefab(inst, newprefab, chance, haunt_value, nofx)` 接收名称或非空名称表，关闭成功后的普通冷却。替换成功会复制若干现有资源状态、推双向替换事件，隐藏旧实体并下一任务移除；不是原地改名。[替换流程](../../../raw/current/scripts/standardcomponents.lua#L1219-L1268) |
| 采集、点燃、腐败、掉物 | `Work`、`WorkAndIgnite`、`Ignite`、`Perish`、`DropFirstItem` 的相关效果被注释，当前回调返回假。不要用这些名字承诺对应功能。[Work](../../../raw/current/scripts/standardcomponents.lua#L1107-L1152)、[Ignite](../../../raw/current/scripts/standardcomponents.lua#L1173-L1188)、[Perish](../../../raw/current/scripts/standardcomponents.lua#L1294-L1308)、[DropFirstItem](../../../raw/current/scripts/standardcomponents.lua#L1455-L1483) |

组合入口 `AddHauntableCustomReaction(inst, fn, secondrxn, ignoreinitialresult, ignoresecondaryresult)` 会保存旧回调。`secondrxn=true` 先原版后自定义；默认只有前段成功才继续，`ignoreinitialresult=true` 可取消此门槛；`ignoresecondaryresult=true` 保留前段结果。若目标是“总追加副作用并保留原判定”，可选这三个标志都为真。它保留的是**回调结果**，不回滚自定义函数对字段的修改。[完整实现](../../../raw/current/scripts/standardcomponents.lua#L1528-L1551)。

## 典型接入

### 延长燧石作祟间隔，保留原发射反应

在已有服务端模组的 `modmain.lua` 加入以下完整代码；无额外资源，不替换 Prefab 或回调。默认模组入口见 [[cookbook/mod-starter#全局名称与加载环境]]。只调整燧石，`6` 的单位为秒；原版已安装 `MakeHauntableLaunchAndSmash`，因此直接改字段足够。[宿主](../../../raw/current/scripts/prefabs/flint.lua#L36-L57)、[注册入口](../../../raw/current/scripts/modutil.lua#L593-L599)。

```lua
AddPrefabPostInit("flint", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local hauntable = inst.components.hauntable
    if hauntable ~= nil then
        hauntable.cooldown = 6
    end
end)
```

生效于加载／生成并经过 post-init 的燧石；已经开始的冷却使用已写入 `cooldowntimer` 的值，下次作祟才读取新配置。退出并禁用模组后重载回原构造值，无须删组件或手工清标签；不提供热卸载。地面燧石、被持有燧石、冷却中再次作祟应分别验证，后两者由动作门禁拒绝。[读取与门禁](../../../raw/current/scripts/components/hauntable.lua#L99-L111)、[HAUNT.fn](../../../raw/current/scripts/actions.lua#L3153-L3163)。本例仅完成源码与语法核验，未在游戏中执行。

## 常见陷阱与限制

- **回调失败仍有冷却**：有效目标仍设 `haunted=true`、计时并着色，标签不代表效果成功；`haunted` 事件现只在成功分支推送（26148036 核验：C-17），监听它等价于"效果成功"信号。`LaunchAndIgnite` 会先发射，最终却返回 `false`，因为点燃与成功返回均被注释。[失败分支](../../../raw/current/scripts/components/hauntable.lua#L106-L111)、[该工厂](../../../raw/current/scripts/standardcomponents.lua#L1191-L1216)。
- **关闭成功冷却需自行收尾**：`cooldown_on_successful_haunt=false` 不会自动启动普通计时；成功回调置下的 `haunted` 仍在。换 Prefab helper 通过移除旧实体结束；持久宿主若照抄此字段，要另有停止路径。[成功分支](../../../raw/current/scripts/components/hauntable.lua#L88-L104)、[换物移除](../../../raw/current/scripts/standardcomponents.lua#L1247-L1266)。
- **清理不等于撤销自定义效果**：移除组件只关 FX、shader、移除标签，不调用 `onunhaunt`；`StopHaunt` 也未清恐慌计时。需要撤销自定义修改时自行设计幂等恢复，避免假定每个回调只执行一次。[更新与移除](../../../raw/current/scripts/components/hauntable.lua#L144-L185)。
- **关 `usefx` 不能保证关 shader**：`DoHaunt` 独立调用 `StartShaderFx`，需要可用 AnimState；无普通 AnimState 的宿主要核验 `SetAnimStateGetterFn` 的返回对象。[FX](../../../raw/current/scripts/components/hauntable.lua#L61-L64)、[shader getter](../../../raw/current/scripts/components/hauntable.lua#L117-L138)。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；漂移文件 components/hauntable.lua、actions.lua、componentactions.lua、tuning.lua 对照 R-01/R-04/C-17 逐条核验：DoHaunt 的 `haunted` 事件移入 onhaunt 成功分支（hauntable.lua:90，旧 :114 函数末尾推送），失败分支仍置状态／冷却／着色但不再发事件，正文"26148036 核验"标注 3 处；HAUNT 声明与 fn 门禁逐行未变，tuning 仅整体位移；锚点重定位 8 处、结论修订 3 处；未覆盖：运行验证。

- 2026-10-21，`steam-buildid:25643519`：已审阅组件全文、表中 helper 实效分支、组合回调、动作收集／执行与幽灵 SG、燧石构造和猪人恐慌消费者。结论范围以正文锚点为准，不覆盖全部宿主与复活完整流程。
- 示例仅调整原宿主冷却；完整 `modmain.lua` 片段已通过 Lua 5.1 语法检查；本页元数据、大小、链接及源码行锚点通过指定页检查。
- 未运行游戏；联机显示、冷却实际时序、换物存档及特殊组件交互未做运行验证。
