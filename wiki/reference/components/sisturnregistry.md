---
title: "sisturnregistry"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级姐弟祭坛（sisturn）注册表：汇总所有祭坛的有花状态与'月树花'氛围，向 Wendy／Abigail 相关系统广播 onsisturnstatechanged。"
aliases: ["姐弟祭坛注册表", "sisturn注册", "sisturnregistry"]
tags: [component, server-client]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/sisturnregistry.lua", "prefabs/sisturn.lua", "prefabs/wendy.lua", "prefabs/abigail.lua", "prefabs/ghostly_elixirs.lua", "stategraphs/SGabigail.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sisturnregistry

## 快速判断

- 职责与适用：挂在 `TheWorld` 上的服务端注册表，追踪每座 sisturn 是否有花（is_active）以及是否为"月树花"氛围（is_blossom），状态变化时推世界事件；Wendy 姐妹系统是主要消费者。[状态聚合](../../../raw/current/scripts/components/sisturnregistry.lua#L25-L50)
- 优先使用：查询用 `IsActive()`／`IsBlossom()`；监听世界事件 `onsisturnstatechanged`（携带 `{is_active, is_blossom}`）做即时响应。[查询](../../../raw/current/scripts/components/sisturnregistry.lua#L100-L106)
- 重要副作用：向每座祭坛挂 `onremove`／`onburnt` 监听自动出册；祭坛侧推 `ms_updatesisturnstate` 驱动重算。[Register](../../../raw/current/scripts/components/sisturnregistry.lua#L89-L98)
- 修改边界：构造断言 mastersim，客户端不存在；懒加载——第一座祭坛建成时才 `TheWorld:AddComponent`；无存读钩子，状态由祭坛每帧事件重建。[挂载](../../../raw/current/scripts/prefabs/sisturn.lua#L343-L347)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `Register(sisturn)` | 无返回 | 登记祭坛（重复注册直接返回），初始记 false 并挂 onremove/onburnt 监听；实际有无花由祭坛推 `ms_updatesisturnstate` 更新。[L89-L98](../../../raw/current/scripts/components/sisturnregistry.lua#L89-L98) |
| `IsActive()` | 布尔 | 世界里存在"有花"的祭坛；Wendy 据此开姊妹花环范围增疗（25 范围找玩家）。[L100-L102](../../../raw/current/scripts/components/sisturnregistry.lua#L100-L102)、[wendy](../../../raw/current/scripts/prefabs/wendy.lua#L105-L108) |
| `IsBlossom()` | 布尔 | 任一祭坛容器内有 `moon_tree_blossom`（`getsisturnfeel() == "BLOSSOM"`）；影响 Abigail 外观、药水与技能判定。[L104-L106](../../../raw/current/scripts/components/sisturnregistry.lua#L104-L106)、[feel 定义](../../../raw/current/scripts/prefabs/sisturn.lua#L89-L95) |
| `onsisturnstatechanged`（事件） | `{is_active, is_blossom}` | 状态实际变化才推送；wendy.lua 监听后联动 ghostlybond 升级倍率等。[推送](../../../raw/current/scripts/components/sisturnregistry.lua#L45-L49)、[监听](../../../raw/current/scripts/prefabs/wendy.lua#L413) |
| `ms_updatesisturnstate`（输入事件） | `{inst, is_active}` | 祭坛在放花/取花/烧毁等时机推给世界，registry 据此重算并落库。[监听](../../../raw/current/scripts/components/sisturnregistry.lua#L75)、[祭坛侧](../../../raw/current/scripts/prefabs/sisturn.lua#L181) |

## 消费者一览

- Wendy 本体：`IsActive()` 控制精神光环增益与 ghostlybond 时间倍率（[L105](../../../raw/current/scripts/prefabs/wendy.lua#L105)、[L260-L262](../../../raw/current/scripts/prefabs/wendy.lua#L260-L262)）。
- Abigail：状态图按 `IsBlossom()` 选花冠形态（[SGabigail](../../../raw/current/scripts/stategraphs/SGabigail.lua#L6-L7)）；阿比盖尔花外观判定（[abigail.lua](../../../raw/current/scripts/prefabs/abigail.lua#L749-L750)）。
- 灵药与技能：ghostly_elixirs 配方提示、skilltree_wendy 的效果分支都读 `IsBlossom()`（[药水](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L101-L102)）。

## 常见陷阱与限制

- worldgen（`POPULATING`）期间收到状态更新会推迟到 0 秒任务后重算，避免布局阶段半初始化状态误广播。[L26-L31](../../../raw/current/scripts/components/sisturnregistry.lua#L26-L31)
- 组件懒加载：没有祭坛的世界没有该组件，所有读者都要判 nil（原版读者均如此写）；模组读取时同样不能假设存在。[懒加载](../../../raw/current/scripts/prefabs/sisturn.lua#L343-L345)
- is_blossom 只看"是否存在任一月树花祭坛"，多祭坛时无法区分是哪座；需要按祭坛粒度的状态得自己监听祭坛实体事件。[聚合逻辑](../../../raw/current/scripts/components/sisturnregistry.lua#L36-L43)
- 祭坛烧毁（`onburnt`）与移除都会出册，但"花被拿走"由祭坛自己推 `ms_updatesisturnstate` 更新，registry 不主动轮询容器内容。[出册](../../../raw/current/scripts/components/sisturnregistry.lua#L56-L64)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖 sisturnregistry.lua 全文（含 POPULATING 推迟、事件推送条件）与 sisturn.lua 的注册/推送/getsisturnfeel 段；核对 wendy、abigail、SGabigail、ghostly_elixirs、skilltree_wendy 的读取点；未覆盖 skilltree_wendy 各分支效果数值。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 通过（见批记录）。
