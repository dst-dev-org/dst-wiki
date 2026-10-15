---
title: "highlightchild"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "让 FX 子实体登记到宿主的高亮列表并随宿主一起被高亮着色，属主可经网络变量同步到客户端。"
aliases: ["highlightchild", "子实体高亮", "跟随高亮", "高亮属主同步"]
tags: [component, network]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/highlightchild.lua", "components/highlight.lua", "prefabs/hats.lua", "prefabs/backpack.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# highlightchild

## 快速判断

- 职责与适用：装在 FX／子实体上，把自身登记进 `owner.highlightchildren` 表；宿主 highlight 组件着色时遍历该表同步 `SetHighlightColour`。([highlight.lua:88-92](../../../raw/current/scripts/components/highlight.lua#L88-L92)
- 优先使用：装备时 `SetOwner(持有者)`、卸下时 `SetOwner(nil)`；自定义变更逻辑用 `SetOnChangeOwnerFn(fn)`。([backpack.lua:75-100](../../../raw/current/scripts/prefabs/backpack.lua#L75-L100) ([hats.lua:5915-5916](../../../raw/current/scripts/prefabs/hats.lua#L5915-L5916)
- 重要副作用：带 Network 的宿主建 net_entity `highlightchild.syncowner`（事件 `syncownerdirty`）；主模拟 SetOwner 时写网络变量，客户端监听后本地换属主。([highlightchild.lua:10-15](../../../raw/current/scripts/components/highlightchild.lua#L10-L15) ([highlightchild.lua:24-29](../../../raw/current/scripts/components/highlightchild.lua#L24-L29)
- 修改边界：专用服务器跳过高亮簿记（只跑回调）；组件只定义 `OnRemoveEntity`，`RemoveComponent` 不会清理属主表。([highlightchild.lua:36-37](../../../raw/current/scripts/components/highlightchild.lua#L36-L37) ([highlightchild.lua:18-22](../../../raw/current/scripts/components/highlightchild.lua#L18-L22)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetOwner(owner)` | 实体或 nil | 主模拟先 `syncowner:set(owner)` 再本地换属主；nil 表示脱离。([highlightchild.lua:24-29](../../../raw/current/scripts/components/highlightchild.lua#L24-L29) |
| `SetOnChangeOwnerFn(fn)` | fn(inst, owner) | 每次属主变化（含网络同步触发）后调用，双端生效。([highlightchild.lua:31-33](../../../raw/current/scripts/components/highlightchild.lua#L31-L33) ([highlightchild.lua:52-54](../../../raw/current/scripts/components/highlightchild.lua#L52-L54) |
| `OnChangeOwner(owner)` | 实体或 nil | 非专用服务器维护旧属主移除＋新属主表插入，并清旧高亮色。([highlightchild.lua:35-50](../../../raw/current/scripts/components/highlightchild.lua#L35-L50) |
| `owner.highlightchildren` | 宿主实体上的普通表 | 无则创建 `{self.inst}`；highlight 组件 ApplyColour／移除时遍历同步与清色。([highlightchild.lua:44-48](../../../raw/current/scripts/components/highlightchild.lua#L44-L48) ([highlight.lua:135-141](../../../raw/current/scripts/components/highlight.lua#L135-L141) |

## 典型接入

### 帽子／背包 FX 随持有者高亮

- 前提与选择：FX prefab 自身装 highlightchild（原版帽子 FX 在 `MakeFX` 里添加）；宿主需有 highlight 组件才有实际着色。[hats.lua:7054](../../../raw/current/scripts/prefabs/hats.lua#L7054

```lua
local function onequip(inst, owner)
    local fx = SpawnPrefab("myhat_fx")
    fx.entity:SetParent(inst.entity)
    fx.components.highlightchild:SetOwner(owner)
    inst._fx = fx
end

local function onunequip(inst, owner)
    if inst._fx ~= nil then
        inst._fx.components.highlightchild:SetOwner(nil)
        inst._fx:Remove()
        inst._fx = nil
    end
end
```

- 生效与撤销：换属主时旧属主列表移除并清高亮色；FX 实体移除时 OnRemoveEntity 兜底出列表。([highlightchild.lua:18-22](../../../raw/current/scripts/components/highlightchild.lua#L18-L22)

## 常见陷阱与限制

- `RemoveComponent("highlightchild")` 不触发清理（无 OnRemoveFromEntity）：卸下组件前先 `SetOwner(nil)`。([highlightchild.lua:18-22](../../../raw/current/scripts/components/highlightchild.lua#L18-L22)
- 专用服务器上属主表不维护：依赖 `highlightchildren` 做服务端逻辑不可靠。([highlightchild.lua:36-37](../../../raw/current/scripts/components/highlightchild.lua#L36-L37)
- 高亮着色实际由宿主 highlight 组件驱动；本组件只登记表与同步属主，本身不改颜色。([highlight.lua:88-92](../../../raw/current/scripts/components/highlight.lua#L88-L92)

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文、highlight 侧列表消费、帽子 FX 与背包两条典型挂载／SetOwner 路径。
- 游戏运行：未运行；未核对客户端同步的时序表现。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。
