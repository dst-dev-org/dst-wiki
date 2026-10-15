---
title: "klaussackloot"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级克劳斯袋战利品抽取器：预 roll 四组奖励与冬日盛宴追加，供开袋动作一次性取走并自动重掷。"
aliases: ["克劳斯袋战利品", "克劳斯战利品抽取", "klaussackloot"]
tags: [component, loot]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/klaussackloot.lua", "prefabs/klaus_sack.lua", "prefabs/world.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# klaussackloot

## 快速判断

- 职责与适用：挂在 TheWorld 上的战利品抽取器（prefabs/world.lua:576）；`klaus_sack` 被真钥匙打开时按组取走奖励（prefabs/klaus_sack.lua:66）。构造时即 roll 一次（components/klaussackloot.lua:53）。
- 优先使用：取一份成组战利品→`GetLoot()`；自定义发放复用同样走 `GetLoot()`，元素格式见下表。
- 重要副作用：`GetLoot()` 返回前重掷全部奖励表（components/klaussackloot.lua:139），每次调用结果不同。
- 修改边界：纯服务器数据组件，无网络字段；两张 loot 表整体随实体存读（:144-157）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddGiantLootPrefabs` | `global (prefabs: table) -> nil` | require 本组件即注入的全局函数；把三张 giant loot 表的 prefab 补进依赖列表（components/klaussackloot.lua:36-48，消费 prefabs/klaus_sack.lua:27-28）。 |
| `GetLoot` | `() -> table` | 冬日盛宴激活时先追加 `wintersfeast_loot` 两组，再附 `loot` 四组；随后重掷。:127-142 |
| `RollKlausLoot` | `() -> nil` | 生成四组：`amulet+goldnugget+charcoal 补足 3-4 件`、50% amulet 变体、10% `krampus_sack` 变体、giant loot 组（1 个 loot1、50% loot2、2 个不重复 loot3）。:70-125 |
| `loot` 元素格式 | `string \| {prefab, count}` | 字符串直接生成；`{"winter_food1", 4}` 由消费方按堆叠数量处理（prefabs/klaus_sack.lua:33-38）。 |

## 典型接入

### 复用克劳斯袋战利品发放（服务器）

- 前提与选择：世界 prefab 上组件常驻，直接读 `TheWorld.components.klaussackloot`，无需自建实例。
- 生效与撤销：`GetLoot()` 每次调用重掷并返回新表；组件无清理边界。

```lua
-- modmain.lua（服务器）：把一份克劳斯袋战利品发给玩家
local function GiveKlausLoot(doer)
    local mgr = TheWorld.components.klaussackloot
    if mgr == nil or doer.components.inventory == nil then
        return
    end
    for _, items in ipairs(mgr:GetLoot()) do
        for _, entry in ipairs(items) do
            local prefab = type(entry) == "table" and entry[1] or entry
            local count = type(entry) == "table" and entry[2] or 1
            local item = SpawnPrefab(prefab)
            if item ~= nil then
                if count > 1 and item.components.stackable ~= nil then
                    item.components.stackable:SetStackSize(count)
                end
                doer.components.inventory:GiveItem(item)
            end
        end
    end
end
```

- 调整与验证：冬盛宴判定用 `IsSpecialEventActive(SPECIAL_EVENTS.WINTERS_FEAST)`（:129），测试时可临时改为始终追加。

## 常见陷阱与限制

- `GetLoot()` 是"取走即重掷"语义；已返回的表不受下次重掷影响，但同一实例上二次调用会拿到不同奖励。
- 钥匙判定、锁组件移除与袋子自毁流程在 klaus_sack 与 [[reference/components/klaussackkey]]、[[reference/components/klaussacklock]] 侧维护，本组件不处理钥匙逻辑。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 158 行；核验挂载（prefabs/world.lua:576）、消费（prefabs/klaus_sack.lua:27-28、:66）与 loot 元素双格式（prefabs/klaus_sack.lua:33-38）。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/klaussackkey]]、[[reference/components/klaussacklock]]。
