---
title: "furnituredecor"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "家具装饰物标记：把物品标记为可摆上家具承物架并维护摆放状态与摆放/取下回调。"
aliases: ["家具装饰", "furnituredecor", "装饰物摆放"]
tags: [component, item]
dst_build: "steam-buildid:26230496"
sources: ["components/furnituredecor.lua", "components/furnituredecortaker.lua", "componentactions.lua", "prefabs/furniture_decor_items.lua", "prefabs/decor_flowervase.lua", "prefabs/w_radio.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# furnituredecor

## 快速判断

- 职责与适用：挂在可摆到承物家具上的装饰物品（花瓶、收音机、遗迹遗物等）；`enabled` 经 watcher 同步 `furnituredecor` 标签，`PutOnFurniture`/`TakeOffFurniture` 维护摆放状态并转发宿主回调（[组件全文](../../../raw/current/scripts/components/furnituredecor.lua#L5-L20)）。
- 优先使用：摆放/取下由家具端的 `furnituredecortaker` 驱动，本组件只被动响应（[取下调用](../../../raw/current/scripts/components/furnituredecortaker.lua#L111-L112)）；候选为 GIVE 动作，要求目标带 `furnituredecortaker` 标签（[候选](../../../raw/current/scripts/componentactions.lua#L1322-L1325)）。
- 重要副作用：`on_furniture` 只在内存中记录，不参与存档；宿主回调 `onputonfurniture`/`ontakeofffurniture` 由宿主直接赋值（[收音机挂接](../../../raw/current/scripts/prefabs/w_radio.lua#L288-L289)）。
- 修改边界：`OnRemoveFromEntity` 摘标签；组件源码注释建议标签加在 pristine 状态以优化网络（[注释](../../../raw/current/scripts/components/furnituredecor.lua#L14-L15)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `enabled` | `bool，默认 true（watcher）` | 写入即同步/摘除 `furnituredecor` 标签；`SetEnabled` 仅在变化时写入。[watcher](../../../raw/current/scripts/components/furnituredecor.lua#L1-L3)、[SetEnabled](../../../raw/current/scripts/components/furnituredecor.lua#L26-L30) |
| `IsOnFurniture()` | `-> bool` | 返回 `on_furniture == true`；原版收音机按此切换激活态。[实现](../../../raw/current/scripts/components/furnituredecor.lua#L32-L34)、[消费](../../../raw/current/scripts/prefabs/w_radio.lua#L154) |
| `PutOnFurniture(furniture)` | `furniture: entity -> 无` | 置 `on_furniture=true` 并调 `onputonfurniture(inst, furniture)`。[实现](../../../raw/current/scripts/components/furnituredecor.lua#L36-L41) |
| `TakeOffFurniture(furniture)` | `furniture: entity -> 无` | 清 `on_furniture` 并调 `ontakeofffurniture(inst, furniture)`；承物家具移除装饰时调用。[实现](../../../raw/current/scripts/components/furnituredecor.lua#L43-L48) |
| `decor_animation` | `string，默认 "idle"` | 全库未发现读取者，仅定义。[字段](../../../raw/current/scripts/components/furnituredecor.lua#L10) |

## 典型接入

### 让自定义物品可摆上承物家具

- 前提与选择：物品加本组件；`furnituredecor` 标签由组件构造自动加，原版装饰物品还在 pristine 前预加标签做优化（[通用装饰宿主](../../../raw/current/scripts/prefabs/furniture_decor_items.lua#L48)、[花瓶标签注释](../../../raw/current/scripts/prefabs/decor_flowervase.lua#L135-L136)）。
- 生效与撤销：玩家对承物家具用 GIVE 摆放，由家具端组件回填状态；装饰被取下/家具删除时 TakeOffFurniture 清状态（[承物端](../../../raw/current/scripts/components/furnituredecortaker.lua#L38-L39)）。

```lua
AddPrefabPostInit("my_decor_item", function(inst)
    inst:AddTag("furnituredecor") -- pristine 后补加，仅作兜底
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    local furnituredecor = inst:AddComponent("furnituredecor")
    furnituredecor.onputonfurniture = function(inst, furniture) end
    furnituredecor.ontakeofffurniture = function(inst, furniture) end
end)
```

- 调整与验证：摆放成功与否取决于家具端 `furnituredecortaker` 的槽位逻辑，本组件不判断适配性；珍珠装饰评分按带组件的实体统计（pearldecorationscore.lua:408）。

## 常见陷阱与限制

- **on_furniture 不存档**：两字段状态无 OnSave/OnLoad（[组件全文](../../../raw/current/scripts/components/furnituredecor.lua#L1-L50)）；读档后"是否在架上"由家具端加载流程重新 PutOnFurniture 建立，别直接依赖此字段做持久化判断。
- **回调字段未在构造时声明**：`onputonfurniture`/`ontakeofffurniture` 在构造里被注释（[注释](../../../raw/current/scripts/components/furnituredecor.lua#L12)），是"约定字段"——组件内以真值判断调用，宿主直接赋值即可。
- **标签双端可见、组件仅主机**：候选按标签生成，但状态与回调只在主机端；客户端镜像判断要用标签而非组件。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、GIVE 候选、furnituredecortaker 的摆放/取下链路、furniture_decor_items/decor_flowervase/w_radio/ruinsrelic/vaultrelic 宿主挂接、pearldecorationscore 消费、`decor_animation` 全库检索。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
