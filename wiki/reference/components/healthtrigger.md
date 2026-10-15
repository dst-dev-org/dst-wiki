---
title: "healthtrigger"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "按生命比例双向跨越阈值触发回调，用于阶段切换与血量节点通知。"
aliases: ["healthtrigger", "血量阈值", "生命比例跨线", "Boss血量阶段"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/healthtrigger.lua", "components/health.lua", "prefabs/dragonfly.lua", "prefabs/alterguardian_phase4_lunarrift.lua", "prefabs/wagboss_robot.lua", "entityscript.lua", "modutil.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# healthtrigger

## 快速判断

- 职责与适用：监听宿主 `healthdelta`，当生命比例跨过登记节点时调用 `fn(inst)`；扣血和回血都能触发。[healthtrigger.lua:5](../../../raw/current/scripts/components/healthtrigger.lua#L5)、[:25](../../../raw/current/scripts/components/healthtrigger.lua#L25)
- 优先使用：需求是“经过 50% 时做一次动作”用 `AddTrigger(0.5, fn)`；“只要低于 50% 就持续生效”还需自行维护当前状态及初始化。
- 重要副作用：一次变化可触发多个节点，降血按阈值从高到低、升血从低到高执行；登记不会自行执行或自动注销。[healthtrigger.lua:17](../../../raw/current/scripts/components/healthtrigger.lua#L17)、[:33](../../../raw/current/scripts/components/healthtrigger.lua#L33)
- 修改边界：在主模拟端已有 `health` 的实体接入；原版龙蝇在客户端提前返回后才添加两组件。组件不承担网络同步、存档阶段恢复或计时。[dragonfly.lua:492](../../../raw/current/scripts/prefabs/dragonfly.lua#L492)、[:506](../../../raw/current/scripts/prefabs/dragonfly.lua#L506)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `AddTrigger(amount, fn)` | `amount` 为生命比例数值，`fn(inst)` | 直接赋 `triggers[amount] = fn`；同值覆盖，传 nil 可移除该键，没有独立删除方法或返回值。[healthtrigger.lua:17](../../../raw/current/scripts/components/healthtrigger.lua#L17 |
| `triggers` | 阈值 → 函数的表，初始空表 | 常规比例用 0～1；实现不检查范围或回调类型。[healthtrigger.lua:8](../../../raw/current/scripts/components/healthtrigger.lua#L8 |
| `OnHealthDelta(data)` | 使用 `oldpercent`、`newpercent` | 触发条件为 `old > k and new <= k` 或 `old < k and new >= k`；旧值必须严格位于阈值一侧。[healthtrigger.lua:25](../../../raw/current/scripts/components/healthtrigger.lua#L25 |
| `OnRemoveFromEntity()` | 组件删除钩子 | 解绑自身 `healthdelta`；宿主 `RemoveComponent` 调用此钩子。[healthtrigger.lua:13](../../../raw/current/scripts/components/healthtrigger.lua#L13)、[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670 |

`health:DoDelta` 在 `SetVal` 后产生上述比例；分母是基础 `maxhealth`，不是惩罚后的可用上限。事件被无敌或 redirect 提前拦截时不发送。直接 `SetCurrentHealth` 不发事件；`SetPercent` 先改值再 `DoDelta(0)`，通常 old/new 已相同，不能用它模拟跨线。[health.lua:477](../../../raw/current/scripts/components/health.lua#L477)、[:499](../../../raw/current/scripts/components/health.lua#L499)、[:550](../../../raw/current/scripts/components/health.lua#L550)、[:613](../../../raw/current/scripts/components/health.lua#L613)

## 典型接入

### 将龙蝇 80% 血量节点移至 75%

以下为已启用普通服务器模组的完整 `modmain.lua` 内容；`AddPrefabPostInit` 在工厂创建后执行，使用龙蝇已有组件和资源。保留原 80% 节点回调，只移动其阈值；若其他模组已占用 75% 则跳过，50%、20% 节点不变。[modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)、[mainfunctions.lua:387](../../../raw/current/scripts/mainfunctions.lua#L387)、[dragonfly.lua:556](../../../raw/current/scripts/prefabs/dragonfly.lua#L556

```lua
AddPrefabPostInit("dragonfly", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local trigger = inst.components.healthtrigger
    if trigger == nil or trigger.triggers[0.75] ~= nil then return end
    local fn = trigger.triggers[0.8]
    if fn == nil then return end
    trigger:AddTrigger(0.8, nil)
    trigger:AddTrigger(0.75, fn)
end)
```

新建／读档重建的龙蝇以后跨过 75% 时执行原 80% 节点动作；0.75 是比例，不是 75 点生命，向上跨越也仍能触发。移动节点不会立即补触发或撤销已经执行的动作。停用模组并重载世界后恢复原版 80% 注册；只删除 75% 键会丢掉该节点，运行中撤销须将同一函数移回 80%。示例未在游戏运行。

原版龙蝇三个阈值共用 `OnHealthTrigger`，回调发 `transform` 并启动 rampingspawner，组件本身不判断降血方向。[dragonfly.lua:435](../../../raw/current/scripts/prefabs/dragonfly.lua#L435 月亮守护者第四阶段则在注册后显式执行初始阶段，并在宿主 `OnLoad` 按已恢复血量选择阶段；这说明跨线监听不能代替读档初始化。[alterguardian_phase4_lunarrift.lua:1031](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1031)、[:751](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L751)

## 常见陷阱与限制

- **等于阈值后继续离开也触发**：不成立。阈值 0.5 时，0.6→0.5 触发，0.5→0.4 不触发；0.4→0.5 也触发。反复跨越可重复回调，不是永久解锁一次。[healthtrigger.lua:28](../../../raw/current/scripts/components/healthtrigger.lua#L28
- **登记时已低血，期望立即进入阶段**：`AddTrigger` 只写表；自行按当前比例初始化。组件无 `OnSave/OnLoad`；health 读档先 `SetVal` 再刷新，不能补回历史跨线。[health.lua:163](../../../raw/current/scripts/components/health.lua#L163)、[entityscript.lua:1928](../../../raw/current/scripts/entityscript.lua#L1928
- **回调期间删除其他已排队阈值**：本次先收集全部键，再逐一从当前表取函数，没有存在性保护，删除尚未调用的键可导致错误；移除组件也不会中断正在执行的本批回调，应延后变更。[healthtrigger.lua:25](../../../raw/current/scripts/components/healthtrigger.lua#L25
- **把移除监听当作撤销阶段副作用**：删除组件只解绑，不恢复回调修改的攻击参数或外部任务。`wagboss_robot` 由宿主配对管理战斗组件与威胁任务，并移除 healthtrigger。[wagboss_robot.lua:1126](../../../raw/current/scripts/prefabs/wagboss_robot.lua#L1126)、[:1178](../../../raw/current/scripts/prefabs/wagboss_robot.lua#L1178)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 3 处（mainfunctions PostInit 段、entityscript GetPersistData、wagboss_robot 两处）；无结论修订，仅锚点迁移——依据：全局枢纽 diff 审阅＋变更清单命中 mainfunctions.lua、entityscript.lua、alterguardian_phase4_lunarrift.lua（本版仅改 CalcSanityAura 数值，OnLoad／healthtrigger 注册行号未变）、wagboss_robot.lua（RetargetFn 改动在前，SetCombatEnabled 等证据行 +7）；healthtrigger.lua、health.lua、dragonfly.lua 未变。
- 源码审阅：2026-10-21，Steam buildid 25643519；完整审阅组件，核验 health 事件生产／比例／直接设置／读档、EntityScript 保存与组件删除，以及龙蝇注册／回调、第四阶段宿主初始化／读档、机器人动态移除。未穷举所有阈值消费者。
- 游戏运行：未运行；双向多阈值顺序、回调内修改、读档与示例节点迁移仍待实测。
- 结构检查：页面元数据、大小、链接和源码行锚点及本页完整 Lua 示例语法检查通过；不代表游戏效果验证。

相关：[[reference/components/health]]；[[reference/components/damagetracker]]；[[reference/components/dpstracker]]。
