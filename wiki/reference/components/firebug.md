---
title: "firebug"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "核对定时脚下生成 Prefab 的 firebug 计时、理智门禁与睡眠启停，避免误作点火目标 AI。"
aliases: ["定时生火", "低理智生火", "脚下生成火源", "主动引燃 AI", "firebug"]
tags: [component, lifecycle, state-machine]
dst_build: "steam-buildid:26230496"
sources: ["components/firebug.lua", "stategraphs/SGwilson.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# firebug

## 快速判断

- 职责与适用：宿主更新期间倒计时，到期说 `ANNOUNCE_LIGHTFIRE`，可选在自身世界坐标生成 `prefab`。没有寻找可燃目标、追逐、LIGHT 动作或直接 Ignite；不提供主动找目标点火的 AI。[完整更新](../../../raw/current/scripts/components/firebug.lua#L26-L38)
- 优先使用：已有宿主的定时脚下生成机制可调整计时字段；手持工具点目标用 [[reference/components/lighter]]，目标燃烧与蔓延分别用 [[reference/components/burnable]]／[[reference/components/propagator]]。[默认配置](../../../raw/current/scripts/components/firebug.lua#L1-L10)
- 重要副作用：构造即 Enable；`prefab=nil` 默认只说话并重设计时，不会生火。非 nil 时直接使用 SpawnPrefab 返回值的 Transform，不做生成失败检查。[构造](../../../raw/current/scripts/components/firebug.lua#L1-L10) [到期行为](../../../raw/current/scripts/components/firebug.lua#L32-L36)
- 修改边界：本次定向查到 SGwilson 睡眠／醒来启停消费者，未定位原版 `AddComponent("firebug")` 构造宿主；不能据此称当前角色默认拥有它，也不能据文件名认定弃用。自定义宿主须自行负责服务端安装、依赖、生成 Prefab 注册及生命周期。[睡眠启停](../../../raw/current/scripts/stategraphs/SGwilson.lua#L300-L329)

## 关键接口

| 接口／字段 | 默认／参数 | 条件、作用与证据 |
|---|---|---|
| `Enable(enable)`／`Disable()` | enabled 构造后 true | Enable 的形参未使用，`Enable(false)` 仍启用；分别 StartUpdatingComponent／StopUpdatingComponent，重复调用依 enabled 幂等。[启停](../../../raw/current/scripts/components/firebug.lua#L12-L24) |
| `time_to_fire` | 60，秒 | `time_to_fire > dt` 时减 dt，否则触发一次；暂停不重置，恢复继续剩余时间。[构造](../../../raw/current/scripts/components/firebug.lua#L1-L10) [更新](../../../raw/current/scripts/components/firebug.lua#L26-L38) |
| `time_interval`／`time_variance` | 120／120，秒 | 触发后剩余时间改为 `interval + variance * math.random()`；不补算该帧多余 dt，不循环补触发；这些值未做范围校验。[重设计时](../../../raw/current/scripts/components/firebug.lua#L29-L36) |
| `sanity_threshold` | nil；比较 GetPercent 的比例值 | 非 nil 才访问 sanity；仅当 `IsInsanityMode()` 且比例 ≥ 阈值时直接 return，冻结计时。其他理智模式不受这一条件暂停。[门禁](../../../raw/current/scripts/components/firebug.lua#L26-L30) |
| `prefab` | nil／已注册 Prefab 名 | 到期先要求 talker 执行 Say，再可选 SpawnPrefab 并置于宿主坐标；组件不保证该 Prefab 有燃烧／传播功能。[触发](../../../raw/current/scripts/components/firebug.lua#L32-L36) |
| `GetDebugString()` | 字符串 | 返回 enabled 与剩余时间，不检查宿主依赖或 Prefab 有效性。[实现](../../../raw/current/scripts/components/firebug.lua#L40-L42) |

更新本身不判断 enabled；正常停用依赖宿主停止调度，不应在 Disable 后手动调用 OnUpdate 并期待被拦截。实体 StartUpdatingComponent 记录组件，StopUpdatingComponent 排队停止；不要把暂停当成撤回已生成实体。[更新实现](../../../raw/current/scripts/components/firebug.lua#L26-L38) [宿主调度](../../../raw/current/scripts/entityscript.lua#L439-L478)

## 典型接入

### 为已有 firebug 宿主延长定时间隔

完整 `modmain.lua`；仅适用于另一个已审阅宿主在服务端安装 firebug 的场景，无新增资源，也不替宿主选择火源。将首次计时设为 300 秒，后续采用 `300 + 300 * math.random()` 秒，保留宿主原来的理智阈值、prefab 与启停逻辑。AddComponentPostInit 在组件构造后执行；宿主在它之后重新赋值仍会覆盖本例。[注册入口](../../../raw/current/scripts/modutil.lua#L567-L573) [调用时序](../../../raw/current/scripts/entityscript.lua#L648-L665)

```lua
AddComponentPostInit("firebug", function(self, inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    self.time_to_fire = 300
    self.time_interval = 300
    self.time_variance = 300
end)
```

影响所有之后构造的服务端 firebug；不会给任何角色添加组件，默认无宿主时不会出现效果，也不会把 nil prefab 变成火源。运行中停用仍调用 Disable；本例不覆盖方法、回调或睡眠处理，不增加监听／任务。停用模组后新／重建组件恢复宿主初始化值，既有生成物由其自身管理。[组件完整实现](../../../raw/current/scripts/components/firebug.lua#L1-L44)

### 原版睡眠协作与自定义宿主边界

SGwilson 的 SetSleeperSleepState 在存在 firebug 时 Disable，SetSleeperAwakeState 则 Enable，不恢复进入睡眠前是否已经人为禁用；外部长期禁用规则需考虑醒来重新启用。它们只消费可选组件，不能证明角色工厂已经安装 firebug。[睡眠与醒来](../../../raw/current/scripts/stategraphs/SGwilson.lua#L300-L329)

自定义接入前要确保 talker 可用；设置 sanity_threshold 后必须有 sanity；非 nil prefab 必须能成功生成且有 Transform。组件没有生成实体引用、OnRemoveFromEntity、OnRemoveEntity、OnSave、OnLoad 或 LongUpdate。移除组件由 EntityScript 停止更新，删除宿主也会清更新表；两者均无此组件专用的已生成实体回收逻辑。保存系统仅调用有 OnSave 的组件，因此该组件不保存计时、enabled、阈值或 prefab；重建后按构造／宿主重新设置，不恢复剩余计时。[组件全文](../../../raw/current/scripts/components/firebug.lua#L1-L44) [移除组件](../../../raw/current/scripts/entityscript.lua#L670-L684) [删除更新清理](../../../raw/current/scripts/entityscript.lua#L1760-L1771) [组件存档入口](../../../raw/current/scripts/entityscript.lua#L1928-L1947)

## 常见陷阱与限制

- **“低理智才运行”**：阈值默认 nil，没有理智判断；设置阈值也只在 insanity 模式下按 ≥ 暂停，不能扩大为所有理智模式统一门禁。[门禁](../../../raw/current/scripts/components/firebug.lua#L26-L30)
- **“每隔固定秒数产生火焰”**：间隔有随机项，暂停冻结时间，prefab=nil 无实体；生成物到底是 FX、火源还是其他实体由具体 Prefab 决定，本页未核验任何默认火源。[到期处理](../../../raw/current/scripts/components/firebug.lua#L29-L36)
- **“禁用就会清掉火源”**：Disable 只停组件更新，不清剩余时间、不移除生成物；改 interval／variance 只影响下一次重设，立即改变本轮需明确改 time_to_fire。[启停](../../../raw/current/scripts/components/firebug.lua#L12-L24) [计时](../../../raw/current/scripts/components/firebug.lua#L29-L36)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 3 处（SGwilson SetSleeper 段 +12 两处、entityscript 更新清理／存档入口 +3）；无结论修订，仅锚点迁移——firebug.lua、modutil.lua 未变；SGwilson 本版改动不涉 SetSleeperSleepState／AwakeState 内容。依据：全局枢纽 diff 审阅＋变更清单命中 stategraphs/SGwilson.lua、entityscript.lua。
- 源码审阅：2026-10-22，Steam buildid 25643519；完整审阅 firebug，核对启停、门禁、依赖、生成与无存档／清理接口；核对 SGwilson 可选消费者、组件 PostInit、宿主更新／移除／存档入口。本次定向检索未定位正式构造宿主，未发布原版火源实例或角色默认行为结论。
- 结构检查：本页元数据、大小、本地链接与源码锚点经定向检查；完整 modmain 示例经 Lua 5.1 语法检查。
- 游戏运行：未运行；自定义宿主、理智模式切换、睡眠恢复、读档及生成物生命周期需按实际宿主实测。
