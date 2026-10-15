---
title: "foodmemory"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "记录同类料理的重复食用次数与遗忘任务，配置沃利重复进食的三维倍率和记忆时长。"
aliases: ["食物记忆", "重复吃同一种料理", "沃利厌食", "沃利遗忘时间", "foodmemory"]
tags: [component, food, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/foodmemory.lua", "components/eater.lua", "prefabs/warly.lua", "spicedfoods.lua", "tuning.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# foodmemory

## 快速判断

- 职责与适用：挂在食用者身上，以基础料理 prefab 为键记录次数和一项遗忘任务。原版沃利在服务端 `master_postinit` 添加并配置，最后由 `MakePlayerCharacter("warly", ...)` 注册。[warly.lua:25-45](../../../raw/current/scripts/prefabs/warly.lua#L25-L45)
- 优先使用：改变记忆窗口用 `SetDuration`，改变重复食用倍率用 `SetMultipliers`；查询用 `GetMemoryCount`／`GetFoodMultiplier`，不必覆盖 eater 的 `Eat`。[foodmemory.lua:16-55](../../../raw/current/scripts/components/foodmemory.lua#L16-L55)
- 重要副作用：eater 在本次结算前读取倍率，成功进食末尾才记一次；倍率作用于基础生命、饥饿、精神计算，不限制为饥饿收益。[eater.lua:203-235](../../../raw/current/scripts/components/eater.lua#L203-L235)、[:270-284](../../../raw/current/scripts/components/eater.lua#L270-L284)
- 修改边界：原版入口在服务端；组件本身不改变能否吃、客户端动作或食物附加回调。沃利的料理门禁仍由 eater 偏好标签决定。[warly.lua:35-42](../../../raw/current/scripts/prefabs/warly.lua#L35-L42)；饮食与进食顺序见 [[reference/components/eater]]。

## 关键接口

| 接口／字段 | 参数与返回约定 | 条件、消费者与副作用 |
|---|---|---|
| `SetDuration(duration)` | 时间为秒；组件默认 `TUNING.TOTAL_DAY_TIME` | 仅赋值；后续新建／刷新任务才使用，已存在任务不会因此改期。[foodmemory.lua:3-18](../../../raw/current/scripts/components/foodmemory.lua#L3-L18) |
| `SetMultipliers(mults)` | 数值数组；默认 nil | 直接保存表引用；无复制、范围校验或自动存档。计数大于 0 时取 `mults[min(#mults, count)]`，无条目时回退 1。[foodmemory.lua:20-22](../../../raw/current/scripts/components/foodmemory.lua#L20-L22)、[:53-55](../../../raw/current/scripts/components/foodmemory.lua#L53-L55) |
| `RememberFood(prefab)` | 食物 prefab 字符串，无返回约定 | 合并基础料理键后次数加一，取消旧任务，再从此刻等待完整 duration；到期删除整条记录，不逐次减一。[foodmemory.lua:24-46](../../../raw/current/scripts/components/foodmemory.lua#L24-L46) |
| `GetBaseFood(prefab)` | 返回基础 prefab 或原字符串 | 只查 `spicedfoods[prefab].basename`；原版调味食品生成时登记 basename，不按名称后缀猜测。[foodmemory.lua:28-30](../../../raw/current/scripts/components/foodmemory.lua#L28-L30)、[spicedfoods.lua:25-45](../../../raw/current/scripts/spicedfoods.lua#L25-L45) |
| `GetMemoryCount(prefab)` | 无记录为 0 | 与 RememberFood 使用相同归一化键；原味与已登记调味版共享次数和遗忘时间。[foodmemory.lua:48-51](../../../raw/current/scripts/components/foodmemory.lua#L48-L51) |
| `GetFoodMultiplier(prefab)` | 无记录为 1，否则查倍率数组 | 读取不刷新计时。沃利数组为 `{.9, .8, .65, .5, .3}`；首次吃无记忆，第二次才取 .9，六次及以后取 .3，前提是期间未遗忘。[foodmemory.lua:53-55](../../../raw/current/scripts/components/foodmemory.lua#L53-L55)、[tuning.lua:4612-4613](../../../raw/current/scripts/tuning.lua#L4612-L4613) |

### 结算、保存与清理

eater 先读取已有记忆，把 `base_mult` 乘到 edible 的三维 getter 结果与各项 absorption；之后还可能被自定义三维函数、整堆倍率和各组件上限改变。`oneat`、人物回调、食物 OnEaten 和消耗之后才调用 RememberFood；整堆食用只调用一次，带 `potion` 标签的食物跳过这次记忆写入，但前面的倍率读取没有该标签门禁。[eater.lua:203-255](../../../raw/current/scripts/components/eater.lua#L203-L255)、[:270-284](../../../raw/current/scripts/components/eater.lua#L270-L284)

`OnSave` 只保存各键的 count 和任务剩余秒数 t；`OnLoad` 按 t 重建任务，缺 count／t 时分别回退 1／当前 duration。duration 和 mults 需由宿主构造重设；载入不会清空数据中未出现的现有键。移除组件时取消全部记忆任务。宿主对组件的存读与移除分发分别见 EntityScript。[foodmemory.lua:58-85](../../../raw/current/scripts/components/foodmemory.lua#L58-L85)、[:10-14](../../../raw/current/scripts/components/foodmemory.lua#L10-L14)、[entityscript.lua:1928-1935](../../../raw/current/scripts/entityscript.lua#L1928-L1935)、[:2005-2007](../../../raw/current/scripts/entityscript.lua#L2005-L2007)、[:670-680](../../../raw/current/scripts/entityscript.lua#L670-L680)

（26148036 核验：eater Eat 的倍率读取、三维乘算、oneat 事件与 RememberFood 顺序、potion 跳过，及 entityscript 存读／移除分发均无语义变化，仅锚点平移；eater 新增 ResetEdibleTagsCache 不触及本页）

## 典型接入

### 将沃利的重复料理记忆窗口缩短到一个游戏日

完整模组 `modmain.lua` 入口，普通生存模式；复用原版沃利、料理与资源。原版窗口为两个游戏日，本例仅改 duration，保留原倍率数组、饮食门禁、进食回调与记忆保存。注册／服务端门禁复用 [[reference/components/eater#典型接入]]。[warly.lua:35-45](../../../raw/current/scripts/prefabs/warly.lua#L35-L45)、[tuning.lua:4612-4613](../../../raw/current/scripts/tuning.lua#L4612-L4613)

```lua
AddPrefabPostInit("warly", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local memory = inst.components.foodmemory
    if memory ~= nil then
        memory:SetDuration(GLOBAL.TUNING.TOTAL_DAY_TIME)
    end
end)
```

值以秒计，使用游戏日常量避免把现实 24 小时当作游戏一天。之后每次记住该料理都会从当次进食重新计时；连续吃会不断延后遗忘。存档已有记录仍按已保存 t 恢复，直到下次吃同料理刷新，才使用新窗口，不会在加载时统一重置计时。

停用模组并重建人物后，duration 恢复原版；已经保存的剩余 t 仍可保留先前窗口的影响。若需运行期临时开关，应保存并恢复原 duration，并明确是否另行处理既有任务；不要用移除组件当成撤销配置，那会取消全部任务并失去该实例的记忆消费。示例无新增资源、监听或任务。

## 常见陷阱与限制

- **按首次吃下的时刻计算固定窗口**：重复进食会取消旧任务并重新等待 duration；这是从最后一次记录起的窗口。停止重复进食直至任务到期，才整项归零。[foodmemory.lua:32-46](../../../raw/current/scripts/components/foodmemory.lua#L32-L46)
- **在 oneat 再手动 RememberFood**：正常 eater 末尾还会记一次，造成双计数；事件阶段查询看到的也是本次写入前的次数。沿用原消费入口即可。[eater.lua:270-284](../../../raw/current/scripts/components/eater.lua#L270-L284)
- **更换调味料绕过记忆**：原味与 spicedfoods 已登记的调味版归同一基础键；模组食品若没有该映射就按自己的 prefab 独立记录。[foodmemory.lua:28-34](../../../raw/current/scripts/components/foodmemory.lua#L28-L34)
- **倍率只削减正收益或所有效果都被削减**：实际乘到通过 eater 门禁的三维 getter 结果，负值也可能被乘；食物 OnEaten 附加效果不由此统一缩放。应结合 [[reference/components/eater#进食顺序与恢复]] 与 [[reference/components/foodaffinity]] 判断。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：eater.lua／entityscript.lua／tuning.lua（食物记忆相关键）对照 build-26148036 更新报告 C-12／R-08；Eat 内倍率读取与三维乘算、oneat→宿主回调→OnEaten→RememberFood 顺序、potion 跳过、entityscript GetPersistData／SetPersistData／RemoveComponent 分发、WARLY_SAME_OLD_COOLDOWN／MULTIPLIERS 值均无语义变化；锚点重定位 9 处（eater 5、entityscript 2、tuning 2），RemoveComponent 648-658 不变；eater SetCanEat* 弃用与新增 ResetEdibleTagsCache 不触及本页结论。未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；读组件全文，核对沃利构造／注册、原版时长与数组、eater 前读后写与 potion 分支、调味基础映射、宿主存读／移除分发。饮食门禁和通用进食顺序复用同 build eater 页。
- 结构／语法：本页定向页面校验与源码链接 strict 校验通过；完整 modmain 示例通过已有 Full Moon 解析器 Lua 5.1 `--syntax-only` 检查。
- 游戏运行：未运行；同料理连吃、调味版合并、potion、自定义三维、存读剩余时间与多模组修改仍待实测。

相关：[[reference/components/eater]]；[[reference/components/foodaffinity]]；[[reference/components/edible]]。
