---
title: "carnivalgameitem"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "嘉年华喂鸡游戏食物标记：让物品对处于可喂状态的 carnivalgame_feedchicks_nest 生成 CARNIVALGAME_FEED。"
aliases: ["嘉年华游戏食物", "喂鸡游戏蛆虫道具", "carnivalgameitem"]
tags: [component, item, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/carnivalgameitem.lua", "componentactions.lua", "actions.lua", "prefabs/carnivalgame_feedchicks.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# carnivalgameitem

## 快速判断

- 职责与适用：空组件，仅 `self.inst`；作用对象是嘉年华喂鸡游戏的 Grub 食物。两处候选（USEITEM 手持、EQUIPPED 装备）都要求目标带 `carnivalgame_canfeed` 标签且 `target.prefab == "carnivalgame_feedchicks_nest"`——目标 prefab 名硬编码。[候选](../../../raw/current/scripts/componentactions.lua#L1088-L1094)、[装备候选](../../../raw/current/scripts/componentactions.lua#L2252-L2258)
- 优先使用：食物 prefab 主机侧加组件（原版唯一宿主 `carnivalgame_feedchicks_food`，另配 inventoryitem／equippable，[宿主](../../../raw/current/scripts/prefabs/carnivalgame_feedchicks.lua#L471-L479)）；目标可喂窗口由 [[reference/components/carnivalgamefeedable]] 的 `enabled` 控制。
- 重要副作用：组件本体无逻辑；执行调 `target.components.carnivalgamefeedable:DoFeed(invobject, doer)`，巢未 hungry 时返回 false 与 "TOO_LATE"。[执行](../../../raw/current/scripts/actions.lua#L2459-L2464)
- 修改边界：组件加在 mastersim 判定之后，仅主机持有；候选经 actionreplica 组件 id 列表在客户端生成，与 [[reference/components/vasedecoration]] 同一机制（[注册](../../../raw/current/scripts/componentactions.lua#L3182-L3190)）。

## 典型接入

### 让自定义物品可以喂嘉年华巢

- 前提与选择：物品主机侧加组件即可；巢侧 OnFeed 不检查物品种类，任意带本组件的物品都能喂中（[原版回调](../../../raw/current/scripts/prefabs/carnivalgame_feedchicks.lua#L373-L376)）。
- 生效与撤销：玩家侧动作路由到 `doequippedaction` 状态（[SG 路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1134-L1137)）；喂中后巢发 `carnivalgame_feedchicks_feed` 事件，车站监听记分（[记分链](../../../raw/current/scripts/prefabs/carnivalgame_feedchicks.lua#L256-L267)）。

```lua
AddPrefabPostInit("my_grub", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("carnivalgameitem")
end)
```

- 调整与验证：手持（USEITEM）与装备（EQUIPPED）两条候选路径内容一致；物品是否被消耗不在本组件与动作函数内处理。

## 常见陷阱与限制

- **目标写死为喂鸡巢**：其他嘉年华游戏（打靶、赶猪等）不走本组件，复用前先确认目标 prefab 与标签链路（[候选](../../../raw/current/scripts/componentactions.lua#L1089-L1090)）。
- **可喂≠ hungry 窗口内**：巢在 `carnivalgame_canfeed` 标签存在时才生成动作；标签由 feedable 的 `enabled` 增删，标签不在但玩家仍对巢使用时无动作可发。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、两处候选、CARNIVALGAME_FEED 执行、food／nest 两个 prefab 的挂接与 SG 路由。未覆盖：玩家 SG doequippedaction 状态内部、喂中后食物去向、运行验证。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
