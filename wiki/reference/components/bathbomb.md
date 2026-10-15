---
title: "bathbomb"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "浴弹标记：bathbomb 标签让物品对 bathbombable 温泉生成 BATHBOMB；执行成功时温泉生效且浴弹被移除销毁。"
aliases: ["浴弹", "温泉浴弹", "bathbomb"]
tags: [component, item, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/bathbomb.lua", "prefabs/bathbomb.lua", "componentactions.lua", "actions.lua", "components/bathbombable.lua", "recipes.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# bathbomb

## 快速判断

- 职责与适用：标记可用浴弹；构造加 `bathbomb` 标签，`OnRemoveFromEntity` 摘除标签，此外无字段无方法。[组件](../../../raw/current/scripts/components/bathbomb.lua#L1-L11)
- 优先使用：目标侧用 bathbombable 组件（温泉宿主）；USEITEM 候选要求双标签——物品 `bathbomb`、目标 `bathbombable`（[候选](../../../raw/current/scripts/componentactions.lua#L1052-L1055)）。
- 重要副作用：执行成功时调 `bathbombable:OnBathBombed(item, doer)` 并把浴弹从库存移除后 `Remove()`——浴弹被消耗、不返还（[执行](../../../raw/current/scripts/actions.lua#L4818-L4825)、[目标侧](../../../raw/current/scripts/components/bathbombable.lua#L30-L35)）。
- 修改边界：组件加在 mastersim 判定之后（[宿主](../../../raw/current/scripts/prefabs/bathbomb.lua#L60-L62)），仅主机持有；物品可合成（moon_tree_blossom + nitre，TECH.CELESTIAL_ONE，[配方](../../../raw/current/scripts/recipes.lua#L835)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnRemoveFromEntity()` | 引擎移除组件时调用 | 仅摘 `bathbomb` 标签；无其他清理逻辑。[方法](../../../raw/current/scripts/components/bathbomb.lua#L7-L8) |

## 典型接入

### 让自定义浴弹可用

- 前提与选择：物品加标签（pristine 前，双端）与组件（主机）；目标温泉由 bathbombable 的 `can_be_bathbombed` 决定是否接受（[目标侧状态](../../../raw/current/scripts/components/bathbombable.lua#L30-L32)）。
- 生效与撤销：动作生成靠标签，执行要求物品与目标都持有对应组件且 `can_be_bathbombed` 为真；条件不满足时执行函数无返回值，动作失败、浴弹保留（[执行条件](../../../raw/current/scripts/actions.lua#L4822-L4823)）。

```lua
AddPrefabPostInit("my_bathbomb", function(inst)
    inst:AddTag("bathbomb")
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("bathbomb")
end)
```

- 调整与验证：浴弹命中后的效果（玻璃化、计时器等）全部在 bathbombable 宿主的 onbathbombedfn 内；本组件不携带效果数据。

## 常见陷阱与限制

- **执行即销毁**：动作函数显式 `RemoveItem(...):Remove()`（[销毁](../../../raw/current/scripts/actions.lua#L4824)），没有投掷物中间态；想保留物品需自建动作而非复用 BATHBOMB。
- **候选靠标签、执行靠组件**：只加标签时动作仍出现，但执行函数查 `act.invobject.components.bathbomb`，缺组件直接走失败路径（[执行检查](../../../raw/current/scripts/actions.lua#L4819-L4820)）；两个都要加。
- **组件无效果逻辑**：温泉侧 hotspring／hermithotspring 的 bathbombed 状态、计时与外观切换在 bathbombable 与宿主 prefab 内，本页不覆盖其内部流程。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、bathbomb prefab 挂接、USEITEM 候选、BATHBOMB 执行与消耗、bathbombable 的 OnBathBombed 入口、合成配方行。未覆盖：温泉宿主效果全流程、bathbombable 其余方法、运行验证。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
