---
title: "playeractionpicker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家动作拾取器：把鼠标/准星输入翻译成按优先级排序的 BufferedAction 候选，经动作过滤栈与左右键覆盖点实现角色定制交互。"
aliases: ["动作拾取", "左键右键动作", "playeractionpicker 组件"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/playeractionpicker.lua", "prefabs/player_common.lua", "components/playercontroller.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# playeractionpicker

## 快速判断

- 职责与适用：`inst == ThePlayer 或主机` 时随 playercontroller 一起挂载（prefabs/player_common.lua:1023-1026）；只产出候选动作，执行权在 playercontroller。
- 优先使用：改默认点击行为 → `leftclickoverride`/`rightclickoverride`（:15-16，wolfgang.lua:549、woodie.lua:385 用例）；临时屏蔽某类动作 → `PushActionFilter`/`PopActionFilter`（:58-76）。
- 重要副作用：容器注册表（`RegisterContainer`，:22-29）让右键优先走容器 widget 动作（:420-423）；`GetInventoryActions` 按 `CONTROL_FORCE_TRADE` 把点击变 DROP（:260-275）。
- 修改边界：动作本体与优先级在 componentactions.lua 的 `CollectActions` 各场景注册；本组件只排序、过滤与组装。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `DoGetMouseActions` | `(position?, target?, spellbook?) -> lmb, rmb` | 总入口（playercontroller.lua:2830/:4913/:5103）；黑暗中仅保留 DROP(<4距离)/SET_HEADING/BOAT_CANNON_SHOOT 与 STOP_STEERING_BOAT/BOAT_CANNON_STOP_AIMING（:530-555）；同动作去重、CLOSESPELLBOOK 不作提示（:561-577）。 |
| `GetLeftClickActions`/`GetRightClickActions` | `(position, target, …) -> table` | 覆盖→掌舵/炮击/高尔夫意图→持有物/装备/场景/点动作分支（:301-386、:388-496）；右键 AOE 武器在瞄准时剥离其他动作（:447-449）。 |
| `SortActionList` | `(actions, target, useitem?) -> BufferedAction[]` | 按 `action.priority` 降序，逐个过 `actionfilter`，target 为实体/Vector3 分别组装 BufferedAction，CASTAOE 附带 aoe 范围（:84-107）。 |
| `PushActionFilter`/`PopActionFilter` | `(filterfn, priority?) -> nil` | 过滤栈只取最高优先级生效（:42-56）；全局档位 `ACTION_FILTER_PRIORITIES`（:1-9：paused 999/ghost 99/mounted 20/floaterheld 15/heavylifting 10/default -99），默认过滤在 player_common.lua:1027。 |
| `GetInventoryActions` | `(useitem, right?) -> table` | CONTROL_FORCE_TRADE 且未锁定槽 → 强制 DROP（堆叠再按 CONTROL_FORCE_STACK），否则 CollectActions("INVENTORY")（:257-289）。 |
| `GetPointActions`/`GetPointSpecialActions`/`GetDoubleClickActions` | `(pos, …) -> table` | POINT 场景 + `pointspecialactionsfn`（:225-234，支持准星点位 pos2 返回）与 `doubleclickactionsfn`（:240-247）。 |
| `RegisterContainer`/`UnregisterContainer`/`HasContainerWidgetAction` | `(container) -> nil` | 容器移除自动注销；右键点已注册容器且非只读走 `GetSceneActions(target, true)`（:420-423）。:22-40 |

## 关键流程：一次左键点击的决策链

1. `leftclickoverride` 先行：返回非空即用，`usedefault=false` 同样短路（:302-307）。
2. 意图类短路：掌舵（SET_HEADING）、船炮瞄准、高尔夫瞄准（:315-329）。
3. 持有 active item → USEITEM on target（平台/忽略悬停标签走 POINT，:332-343）；否则场景目标按 CONTROL_FORCE_INSPECT→LOOKAT、敌方/CONTROL_FORCE_ATTACK→ATTACK、装备→EQUIPPED 的顺序，最后回退 SCENE（:345-363）。
4. 空地点击：装备的 POINT 动作（剥掉 DROP 防误丢，:366-379）→ pointspecialactionsfn（:380-383）。

## 典型接入

### 给角色自定义左键（wolfgang 模式复现）

- 前提与选择：override 返回 `{BufferedAction…}` 或 `nil`；第二返回值 `usedefault` 为 true 且动作空时继续默认链（:303-306）。
- 生效与撤销：切形态/状态时恢复 nil（woodie.lua:374、wolfgang.lua:560 同款）；override 在主机与本地玩家都生效（组件两端挂载）。
- 原版依据：prefabs/wolfgang.lua:549、prefabs/woodie.lua:385。

```lua
local function MyLeftClickPicker(inst, target, position)
    if target ~= nil and target:HasTag("my_tag") then
        return inst.components.playeractionpicker:SortActionList({ ACTIONS.MY_ACTION }, target)
    end
    return nil, true -- 继续默认逻辑
end
inst.components.playeractionpicker.leftclickoverride = MyLeftClickPicker
```

- 调整与验证：过滤栈成对 Push/Pop（playercontroller.lua:5513 骑乘用 mounted 档）；`SortActionList` 不会补 priority，自定义 action 表需带 priority 字段。

## 常见陷阱与限制

- 过滤栈只看最高优先级：低优先级过滤在高位存在时完全不生效（:42-56），恢复顺序错乱会漏掉过滤。
- `GetLeftClickActions` 有副作用：会读 `replica.inventory` 并改 `disable_right_click`（:313），在非点击上下文调用需知晓。
- 黑暗中候选大削：不可见时只放行个别动作（:531-555），依赖"看得见"的 mod 逻辑在此分支拿不到常规候选。
- 右键容器优先于持物：已注册容器的目标不看 active item（:420-424），自定义对容器的持物交互要走 USEITEM 注册而非 override。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 582 行；挂载 player_common.lua:1023-1027，playercontroller.lua:2830/:4913-4916/:5103/:5243/:5513，override 用例 wolfgang.lua:549-560、woodie.lua:374-385 逐行核对。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/aoetargeting]]
