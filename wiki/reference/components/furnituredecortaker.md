---
title: "furnituredecortaker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "家具承物架的放物端：接收装饰物并维护其摆放状态，配合 furnituredecor 完成摆上与取下动作链。"
aliases: ["家具放物", "furnituredecortaker", "装饰物承放"]
tags: [component, item]
dst_build: "steam-buildid:26230496"
sources: ["components/furnituredecortaker.lua", "components/furnituredecor.lua", "prefabs/furniture_tables.lua", "actions.lua", "componentactions.lua", "components/pearldecorationscore.lua", "prefabs/vault_furniture.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# furnituredecortaker

## 快速判断

- 职责与适用：挂在桌子等家具上，作为 `DECOR` 动作的目标端；持有装饰物期间锁定自身（`enabled=false`）并监听装饰物被拾起/移除。
- 优先使用：结合 [[reference/components/furnituredecor]] 成对使用——物品端放物、家具端承物；动作校验在 [actions.lua](../../../raw/current/scripts/actions.lua#L2333-L2338)。
- 重要副作用：`enabled`/`decor_item` 字段变化经组件同步回调增删标签 `furnituredecortaker`/`hasfurnituredecoritem`（[同步](../../../raw/current/scripts/components/furnituredecortaker.lua#L1-L15)），客户端据此出动作按钮（[按钮](../../../raw/current/scripts/componentactions.lua#L1323)）。
- 修改边界：服务器逻辑；`OnRemoveFromEntity` 只摘自己的标签，装饰物释放靠监听回调。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetEnabled(enabled)` | bool | 停用后 `AbleToAcceptDecor` 恒 false（[实现](../../../raw/current/scripts/components/furnituredecortaker.lua#L59-L73)）。 |
| `AbleToAcceptDecor(item, giver)` | → `bool` | 有 `abletoaccepttest` 时交其裁决，默认可收（[实现](../../../raw/current/scripts/components/furnituredecortaker.lua#L65-L73)）。 |
| `AcceptDecor(item, giver)` | → `true` | 从堆叠取一件、监听 `onremove`/`onpickup`/`onputininventory`，调物品端 `PutOnFurniture`（[实现](../../../raw/current/scripts/components/furnituredecortaker.lua#L75-L100)）。 |
| `TakeItem()` | → 装饰物 or nil | 主动取下并回调 `ondecortaken`；原版桌子抓手用（[消费](../../../raw/current/scripts/prefabs/furniture_tables.lua#L31)）。 |
| `abletoaccepttest` / `ondecorgiven` / `ondecortaken` | 回调字段 | 使用方注入：能否收 / 已放上 / 已取下（[注入](../../../raw/current/scripts/prefabs/furniture_tables.lua#L156-L159)）。 |
| `OnSave` / `LoadPostPass` | 存读钩子 | 存装饰物 GUID 并在读档后重放 `AcceptDecor`（[存读](../../../raw/current/scripts/components/furnituredecortaker.lua#L124-L140)）。 |

## 典型接入

### 自定义家具接收装饰物

- 前提与选择：服务器 master 段注册；物品需带 [[reference/components/furnituredecor]] 才会触发 `PutOnFurniture` 联动。
- 生效与撤销：放上后家具锁定；装饰物被玩家拾起或移除时自动解锁并回调。
- 原版依据：桌子与保险库家具均按"可收测试 + 两个回调"注入（[桌子](../../../raw/current/scripts/prefabs/furniture_tables.lua#L156-L159)、[保险库](../../../raw/current/scripts/prefabs/vault_furniture.lua#L107-L110)）。

```lua
inst:AddComponent("furnituredecortaker")
inst.components.furnituredecortaker.abletoaccepttest = function(inst, item, giver)
    return item:HasTag("my_decor") -- 按需过滤可放物品
end
inst.components.furnituredecortaker.ondecorgiven = function(inst, item, giver)
    -- 放上后的表现/记分
end
inst.components.furnituredecortaker.ondecortaken = function(inst, item)
    -- 取下后的恢复
end
```

- 调整与验证：茶铺评分按 `decor_item` 计分（[计分](../../../raw/current/scripts/components/pearldecorationscore.lua#L171-L172)），自定义评分可仿此读字段。

## 常见陷阱与限制

- `ondecortaken` 的触发点有三处：装饰物移除、被拾起、主动 `TakeItem`；拾起路径会先 `TakeOffFurniture` 再回调（[拾起](../../../raw/current/scripts/components/furnituredecortaker.lua#L33-L47)），回调里不要再调用取下接口以免重复。
- `AcceptDecor` 直接吞掉传入的 `item` 实体（出堆叠/出背包）；动作层已保证传入的是可给物品，绕过动作直调时自行保证。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、furnituredecor 联动接口、actions/componentactions 动作链、桌子与保险库注入方式及茶铺计分；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
