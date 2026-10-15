---
title: "carnivalgamefeedable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "嘉年华喂鸡巢的可喂开关：enabled 赋值即同步 carnivalgame_canfeed 标签，OnFeed 回调承载喂食结果。"
aliases: ["嘉年华游戏可喂目标", "喂鸡巢可喂状态", "carnivalgamefeedable"]
tags: [component, callback, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/carnivalgamefeedable.lua", "prefabs/carnivalgame_feedchicks.lua", "stategraphs/SGcarnivalgame_feedchicks_nest.lua", "actions.lua", "class.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# carnivalgamefeedable

## 快速判断

- 职责与适用：给游戏目标提供"是否接受喂食"开关与喂食回调；宿主是嘉年华游戏巢（`carnivalgame_feedchicks_nest`）。`enabled` 赋值即增删 `carnivalgame_canfeed` 标签（Class onset 回调），客户端凭标签看到可喂候选。[字段回调](../../../raw/current/scripts/components/carnivalgamefeedable.lua#L9-L19)、[onset 机制](../../../raw/current/scripts/class.lua#L36-L43)
- 优先使用：主机侧设 `OnFeed`，用 `enabled` 开关喂食窗口——原版由巢状态机在 hungry_loop 进出时置位／复位（[置位](../../../raw/current/scripts/stategraphs/SGcarnivalgame_feedchicks_nest.lua#L100-L104)、[复位](../../../raw/current/scripts/stategraphs/SGcarnivalgame_feedchicks_nest.lua#L121-L123)）。
- 重要副作用：动作层在 `enabled=false` 时返回 false 与 "TOO_LATE"，喂食被拒绝；`DoFeed` 只转调 `OnFeed`，不做其他校验。[动作](../../../raw/current/scripts/actions.lua#L2459-L2464)
- 修改边界：组件加在 mastersim 判定之后，仅主机持有；无存读钩子，`enabled` 状态靠巢状态机每轮重建。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `enabled` | 布尔，默认 false；赋值触发 onset 回调 | true 加 `carnivalgame_canfeed` 标签、false 移除；标签是客户端候选依据。[定义](../../../raw/current/scripts/components/carnivalgamefeedable.lua#L1-L7)、[回调挂接](../../../raw/current/scripts/components/carnivalgamefeedable.lua#L14-L18) |
| `OnFeed` | `function(inst, doer, item) -> any`，默认 nil | 由 prefab 赋值；原版 `onfeed_nest` 在巢上发 `carnivalgame_feedchicks_feed` 事件并返回 true（[赋值](../../../raw/current/scripts/prefabs/carnivalgame_feedchicks.lua#L413-L414)、[回调](../../../raw/current/scripts/prefabs/carnivalgame_feedchicks.lua#L373-L376)） |
| `DoFeed(doer, item)` | 返回 OnFeed 结果；OnFeed 为 nil 时返回 false | 动作执行唯一入口；不做物品类型或次数校验。[方法](../../../raw/current/scripts/components/carnivalgamefeedable.lua#L21-L26) |

## 典型接入

### 自定义可喂游戏目标（原版同构）

- 前提与选择：目标 prefab 主机侧加组件并设 OnFeed；喂食窗口自行用 `enabled` 控制（原版交给巢状态机）。
- 生效与撤销：喂中走 `DoFeed` → OnFeed；返回真值动作成功。原版喂中链路继续：巢事件 → 车站监听 `ScorePoints` 记分（[记分](../../../raw/current/scripts/prefabs/carnivalgame_feedchicks.lua#L256-L259)）。

```lua
inst:AddComponent("carnivalgamefeedable")
inst.components.carnivalgamefeedable.OnFeed = function(inst, doer, item)
    inst:PushEvent("my_game_fed", { doer = doer, item = item })
    return true
end
inst.components.carnivalgamefeedable.enabled = true
```

- 调整与验证：OnFeed 返回值即动作成败；食物是否消耗、给什么奖励都由宿主自行处理，本组件不涉及。

## 常见陷阱与限制

- **直接调用 DoFeed 不查 enabled**：动作层才检查 `enabled`（[检查](../../../raw/current/scripts/actions.lua#L2461-L2462)）；服务端逻辑直接调 `DoFeed` 会绕开 TOO_LATE 判定。
- **enabled 不约束喂食对象**：OnFeed 原版实现忽略 `item` 参数种类，任何带 [[reference/components/carnivalgameitem]] 的物品都能喂中；要限制物品需在 OnFeed 自行校验。
- **候选依赖标签而非组件存在**：客户端没有本组件（主机侧私有），动作候选只看 `carnivalgame_canfeed` 标签；漏开 `enabled` 时动作根本不出现。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文与 onset 机制、巢 prefab 挂接、状态机置位／复位、CARNIVALGAME_FEED 执行与 TOO_LATE 分支。未覆盖：玩家动作状态内部、游戏计分全流程、运行验证。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
