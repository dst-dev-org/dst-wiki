---
title: "winter_treeseed"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "冬日盛宴树种：记录种出的 winter_tree 系 prefab 名并可 SetTree 更换；种到 winter_treestand 后按字段值生成树。"
aliases: ["冬日盛宴树种", "冬之树种", "winter_treeseed"]
tags: [component, item, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/winter_treeseed.lua", "prefabs/acorn.lua", "prefabs/pinecone.lua", "prefabs/winter_treestand.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# winter_treeseed

## 快速判断

- 职责与适用：种子到树的映射数据——`winter_tree` 字段存要长出的 prefab 名，`SetTree` 更换；唯一消费场景是种到冬日盛宴树架 `winter_treestand`（[组件](../../../raw/current/scripts/components/winter_treeseed.lua#L1-L10)、[种植](../../../raw/current/scripts/prefabs/winter_treestand.lua#L22-L29)）。
- 优先使用：种子 prefab 主机侧加组件并 SetTree；原版 acorn 映射 winter_deciduoustree，三种松果映射 winter_tree／winter_twiggytree／winter_palmconetree（[acorn](../../../raw/current/scripts/prefabs/acorn.lua#L127-L128)、[松果](../../../raw/current/scripts/prefabs/pinecone.lua#L119-L122)、[映射](../../../raw/current/scripts/prefabs/pinecone.lua#L135-L137)）。
- 重要副作用：种植时树架按字段值 `SpawnPrefab` 并 `growable:StartGrowing()`，随后移除种子——字段值必须是带 growable 的已注册 prefab（[种植](../../../raw/current/scripts/prefabs/winter_treestand.lua#L25-L28)）。
- 修改边界：无更新逻辑、无存读钩子；PLANT 候选要求目标带 `winter_treestand` 标签且未起火／阴燃／烧焦（[候选](../../../raw/current/scripts/componentactions.lua#L2018-L2020)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `winter_tree` | 字符串，默认 `"winter_tree"` | 种植时按它 SpawnPrefab。[缺省](../../../raw/current/scripts/components/winter_treeseed.lua#L3) |
| `SetTree(tree)` | 参数字符串，无返回值 | 直接覆盖 `winter_tree`，无类型或注册校验。[方法](../../../raw/current/scripts/components/winter_treeseed.lua#L6-L8) |

## 典型接入

### 自定义树种（种到冬日盛宴树架）

- 前提与选择：自定义树 prefab 需已注册且带 growable 组件（种植流程直接调 StartGrowing，[种植](../../../raw/current/scripts/prefabs/winter_treestand.lua#L25-L28)）；树架由冬日盛宴活动提供，带 `winter_treestand` 标签（[标签](../../../raw/current/scripts/prefabs/winter_treestand.lua#L67)）。
- 生效与撤销：PLANT 动作执行走树架 `onplanted`，一次消耗一颗种子；字段值写错 prefab 名会在种植时才报错。

```lua
AddPrefabPostInit("my_seed", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("winter_treeseed")
    inst.components.winter_treeseed:SetTree("my_winter_tree")
end)
```

- 调整与验证：树架只负责生成与起长，树的成长阶段、产物由该树 prefab 自己的 growable 链定义。

## 常见陷阱与限制

- **组件与活动解耦**：三种松果始终挂组件（addcone 传入非 nil 即加，[挂接](../../../raw/current/scripts/prefabs/pinecone.lua#L119-L123)），但树架是冬日盛宴物件——非活动期没有可种目标，动作自然不出现。
- **普通种树不走本组件**：acorn／松果种进地里走 deployable 的 ondeploy，本组件只服务树架；两者并存互不干扰（[acorn](../../../raw/current/scripts/prefabs/acorn.lua#L127-L128)）。
- **预留未启用**：tree_rock_seed 的挂接整段注释（[注释](../../../raw/current/scripts/prefabs/tree_rock_seed.lua#L93-L94)），说明字段映射机制可复用但该宿主未上线。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、acorn 与三种松果挂接、树架 onplanted 流程与标签、PLANT 候选及 tree_rock_seed 注释例。未覆盖：wintersurprisespawner 等其他 winter_tree 标签消费者、树成长链、运行验证。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
