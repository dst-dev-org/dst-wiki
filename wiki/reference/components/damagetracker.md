---
title: "damagetracker"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "在启用期间累计宿主 healthdelta.amount 的绝对值，跨过总量阈值时回调。"
aliases: ["damagetracker", "累计生命变化", "龙蝇掉鳞阈值", "分时累计受伤"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/damagetracker.lua", "components/health.lua", "prefabs/dragonfly.lua", "stategraphs/SGdragonfly.lua", "entityscript.lua", "modutil.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# damagetracker

## 快速判断

- 职责与适用：监听宿主自己的 `healthdelta`；启用时累加 `math.abs(data.amount)`，用于一段或多段计数期合并后的总量门槛。[damagetracker.lua:4](../../../raw/current/scripts/components/damagetracker.lua#L4)、[:24](../../../raw/current/scripts/components/damagetracker.lua#L24)
- 优先使用：允许回血也计入、只需启停累计的场景；严格“实际受到伤害”需自行过滤负 amount 并明确封顶／过量伤害语义，不能直接依赖此组件名。
- 重要副作用：`Start`／`Stop` 只切开关，不清零；回调只在旧累计严格小于、此次累计大于等于阈值时执行，跨过后不会每次继续触发。[damagetracker.lua:16](../../../raw/current/scripts/components/damagetracker.lua#L16)、[:29](../../../raw/current/scripts/components/damagetracker.lua#L29)
- 修改边界：主模拟端使用。原版龙蝇在服务端添加组件，SG 管理倒地期间的启停；该组件不是攻击者输出统计，也不按 attacker 归因。[dragonfly.lua:492](../../../raw/current/scripts/prefabs/dragonfly.lua#L492)、[SGdragonfly.lua:279](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L279

## 关键接口

| 接口／字段 | 签名或默认值 | 条件、作用与证据 |
|---|---|---|
| `Start()`／`Stop()` | 无参数、无返回值 | 分别将 `enabled` 设 true／false；不解绑监听，不清 `damage_done`。[damagetracker.lua:16](../../../raw/current/scripts/components/damagetracker.lua#L16 |
| `damage_done` | 数值，初始 0 | 每个启用时收到的事件增加 amount 的绝对值；重开计数仍沿用原值。[damagetracker.lua:7](../../../raw/current/scripts/components/damagetracker.lua#L7)、[:24](../../../raw/current/scripts/components/damagetracker.lua#L24) |
| `damage_threshold` | 数值，默认 2500 | 配置字段，没有 setter 或自动重算；设为不大于已有累计的值，不会补发过去的跨线。[damagetracker.lua:8](../../../raw/current/scripts/components/damagetracker.lua#L8)、[:29](../../../raw/current/scripts/components/damagetracker.lua#L29) |
| `damage_threshold_fn` | nil 或 `fn(inst)` | 达到门槛时调用；回调不接收增量、攻击者或总量，需自行读取字段。[damagetracker.lua:30](../../../raw/current/scripts/components/damagetracker.lua#L30 |
| `enabled` | 初始 false | 只有启用时才累计；构造时已经注册匿名监听。[damagetracker.lua:11](../../../raw/current/scripts/components/damagetracker.lua#L11 |

原版 `health:DoDelta` 先处理拦截／吸收／单次上限，再 `SetVal` 封顶当前生命，最后把处理量作为 `healthdelta.amount` 发送。事件量可能不同于实际前后生命差：过量治疗、过量伤害也可能被累计；正数治疗同样取绝对值。`DoDelta(0)` 不增加总量，提前拦截不发事件。[health.lua:565](../../../raw/current/scripts/components/health.lua#L565)、[:613](../../../raw/current/scripts/components/health.lua#L613)

## 典型接入

### 调低龙蝇倒地掉鳞的累计门槛

以下为已启用服务器模组的完整 `modmain.lua` 内容；Prefab 工厂后执行，只修改龙蝇已有计数器的阈值。沿用原版 SG 启停、`DoBreakOff` 回调和 `dragon_scales` 资源，不新增动画、贴图或 Prefab。[modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)、[mainfunctions.lua:387](../../../raw/current/scripts/mainfunctions.lua#L387)、[dragonfly.lua:381](../../../raw/current/scripts/prefabs/dragonfly.lua#L381)、[:534](../../../raw/current/scripts/prefabs/dragonfly.lua#L534)

```lua
AddPrefabPostInit("dragonfly", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local tracker = inst.components.damagetracker
    if tracker ~= nil then
        tracker.damage_threshold = 1000
    end
end)
```

1000 的单位是累计事件量的绝对值，不是秒或命中次数。新建／读档重建的龙蝇采用此阈值；原版 `knockdown` 进入时 `Start`，倒地链退出／起身及相关电击中断路径 `Stop`，期间累计可跨多次倒地保留。[SGdragonfly.lua:279](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L279)、[:348](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L348)、[:387](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L387)、[:948](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L948)

跨线后沿原版流程发出一片鳞片；本例不清总量、不改回调，因而不是“每倒地一次重新达到 1000 就掉一次”。停用模组并重载恢复原版阈值；只停计应调用 `Stop()`。运行中恢复阈值不会撤销已掉落物或补触发历史跨线。示例未在游戏运行。

## 常见陷阱与限制

- **把文件头的 damage done 理解为输出伤害**：监听对象是宿主本身，累计规则也没有 amount 符号或来源筛选；受治疗也累计。要统计玩家输出，应另选战斗归因入口。[damagetracker.lua:13](../../../raw/current/scripts/components/damagetracker.lua#L13)、[:27](../../../raw/current/scripts/components/damagetracker.lua#L27)
- **Start 当作重开一轮**：Stop 后再次 Start 继续原累计；需要新一轮时由自有生命周期显式将 `damage_done = 0`。回调为 nil 时跨线同样已经发生，之后才设置回调不会补发；阈值应为正数。[damagetracker.lua:16](../../../raw/current/scripts/components/damagetracker.lua#L16)、[:29](../../../raw/current/scripts/components/damagetracker.lua#L29)
- **RemoveComponent 当作完整停用**：组件没有 `OnRemoveFromEntity`，匿名监听捕获旧 self；`RemoveComponent` 不替组件自动解绑。先 Stop 可使旧闭包停止累计，但反复删加仍留监听；常规需求保留组件、成对启停，实体删除才清理全部事件。[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670)、[:1258](../../../raw/current/scripts/entityscript.lua#L1258)、[:1731](../../../raw/current/scripts/entityscript.lua#L1731)
- **累计自动存档**：组件没有保存／加载钩子，重建时回到 0／false；宿主若要延续，需自行保存并恢复。原版龙蝇只存交战标记，加载时处理战斗重置，不保存此总量。[entityscript.lua:1928](../../../raw/current/scripts/entityscript.lua#L1928)、[dragonfly.lua:386](../../../raw/current/scripts/prefabs/dragonfly.lua#L386)、[:401](../../../raw/current/scripts/prefabs/dragonfly.lua#L401)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 4 处（mainfunctions PostInit +26、entityscript RemoveAllEventCallbacks +4／OnRemoveEntity 段 +3／GetPersistData +3）；无结论修订，仅锚点迁移——damagetracker.lua、health.lua、dragonfly.lua、SGdragonfly.lua 本版未变。依据：全局枢纽 diff 审阅（R-08 距离函数改动不涉事件回调清理路径）＋变更清单命中 entityscript.lua、mainfunctions.lua。
- 源码审阅：2026-10-21，Steam buildid 25643519；完整审阅组件，核验 health 事件量与生命封顶、组件／实体删除和保存分派；龙蝇服务端注册、阈值与掉鳞回调、宿主存读档及 SG 全部 damagetracker 启停位置。
- 游戏运行：未运行；过量治疗、跨次倒地累计、掉鳞、读档与其他模组覆盖阈值均待实测。
- 结构检查：页面元数据、大小、链接和源码行锚点及本页完整 Lua 示例语法检查通过；不代表游戏效果验证。

相关：[[reference/components/health]]；[[reference/components/healthtrigger]]；[[reference/components/dpstracker]]。
