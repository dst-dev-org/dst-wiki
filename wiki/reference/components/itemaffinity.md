---
title: "itemaffinity"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "物品理智亲和：按优先级让角色因携带特定 prefab 或带特定标签的物品获得理智修正，同一时间只生效一条。"
aliases: ["物品亲和", "itemaffinity", "携带物品理智加成"]
tags: [component, item]
dst_build: "steam-buildid:26230496"
sources: ["components/itemaffinity.lua", "prefabs/wurt.lua", "components/sanity.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# itemaffinity

## 快速判断

- 职责与适用：挂在角色实体上；维护亲和条目表，物品进出背包（itemget/itemlose/dropitem）时自动重算，把命中条目的理智修正写入 `sanity.externalmodifiers`（[组件全文](../../../raw/current/scripts/components/itemaffinity.lua#L1-L10)）。
- 优先使用：角色初始化时 `AddAffinity(prefab, tag, sanity_bonus, priority)` 批量登记（[wurt](../../../raw/current/scripts/prefabs/wurt.lua#L655-L659)）；条目可只给 prefab、只给标签，或两者都给。
- 重要副作用：`RefreshAffinity` 每次先移除旧修正再按优先级重设（[刷新](../../../raw/current/scripts/components/itemaffinity.lua#L38-L51)）；条目按 `priority` 降序取第一个命中者，其余条目不生效。
- 修改边界：修正走 [[reference/components/sanity]] 的 `externalmodifiers`（additive SourceModifierList，[定义](../../../raw/current/scripts/components/sanity.lua#L87)），参与理智速率计算（[速率](../../../raw/current/scripts/components/sanity.lua#L590)）；组件无存读钩子，亲和表靠宿主每次初始化重建。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddAffinity(prefab, tag, sanity_bonus, priority)` | `prefab/tag: string\|nil；bonus: number；priority: number -> 无` | 追加条目并立即刷新；prefab 走 `inventory:Has(prefab, 1)`，tag 走 `HasItemWithTag(tag, 1)`。[实现](../../../raw/current/scripts/components/itemaffinity.lua#L16-L19)、[匹配](../../../raw/current/scripts/components/itemaffinity.lua#L42-L49) |
| `RemoveAffinity(prefab)` | `prefab: string -> 无` | 按 prefab 匹配移除第一条并刷新；tag-only 条目无法用此删除。[实现](../../../raw/current/scripts/components/itemaffinity.lua#L21-L36) |
| `RefreshAffinity()` | `-> 无` | 排序（priority 降序）→ 移除修正 → 第一个命中条目 `SetModifier(inst, bonus)` 后 break。[实现](../../../raw/current/scripts/components/itemaffinity.lua#L38-L51) |
| `affinities` | `条目表 { prefab, tag, sanity_bonus, priority }` | 可读；直接改表后需手动调 RefreshAffinity。[字段](../../../raw/current/scripts/components/itemaffinity.lua#L5) |

## 典型接入

### 给角色配"携带某物"的理智修正

- 前提与选择：角色需有 inventory 与 sanity 组件（刷新路径直接访问，[实现](../../../raw/current/scripts/components/itemaffinity.lua#L40-L43)）；原版 wurt 同时用正负条目（鱼骨加成、鱼肉/变质鱼减益，[宿主](../../../raw/current/scripts/prefabs/wurt.lua#L655-L659)）。

```lua
AddPrefabPostInit("my_character", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    local itemaffinity = inst:AddComponent("itemaffinity")
    itemaffinity:AddAffinity("hutch_fishbowl", nil, TUNING.DAPPERNESS_MED, 1)
    itemaffinity:AddAffinity(nil, "fish", TUNING.DAPPERNESS_MED, 1)
end)
```

- 调整与验证：`sanity_bonus` 为速率修正量（每秒理智），负值合法；`priority` 数值大者优先，同优先级按插入顺序稳定排序。

## 常见陷阱与限制

- **同一时间只有一条生效**：头注释与实现都明确不叠加（[头注释](../../../raw/current/scripts/components/itemaffinity.lua#L1-L2)、[break](../../../raw/current/scripts/components/itemaffinity.lua#L45-L50)）；想要多个物品同时加成需改用其他修正途径。
- **删除粒度是 prefab**：`RemoveAffinity` 只按 prefab 匹配第一条（[实现](../../../raw/current/scripts/components/itemaffinity.lua#L23-L33)），tag-only 条目或同 prefab 多条目需操作 `affinities` 后手动刷新。
- **监听不随组件移除**：itemget/itemlose/dropitem 监听挂在实体上且无卸载逻辑（[注册](../../../raw/current/scripts/components/itemaffinity.lua#L7-L9)），运行期移除组件后刷新闭包仍会访问 `self`，避免动态移除本组件。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、wurt 全部条目、sanity.externalmodifiers 的定义与消费点。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。

相关：[[reference/components/sanity]]
