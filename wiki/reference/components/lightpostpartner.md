---
title: "lightpostpartner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "灯杆/茶铺类建筑的锁扣位管理：把灯笼等伙伴实体按编号槽位挂到建筑上，经网络变量同步并随存档恢复。"
aliases: ["灯杆挂点", "lightpostpartner", "灯笼锁扣"]
tags: [component, network]
dst_build: "steam-buildid:26230496"
sources: ["components/lightpostpartner.lua", "prefabs/lantern_posts.lua", "prefabs/hermitcrab_teashop.lua", "prefabs/hermithouse.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lightpostpartner

## 快速判断

- 职责与适用：为带挂点的建筑（`lantern_post`、隐士茶铺等）维护若干编号槽位，槽位存伙伴实体的 `net_entity`；组件自加标签 `lightpostpartner`（[标签](../../../raw/current/scripts/components/lightpostpartner.lua#L34)）。
- 优先使用：伙伴实体找空槽用 `ShacklePartnerToNextID`； lantern 系统按槽位号找回伙伴（[消费](../../../raw/current/scripts/prefabs/lantern_posts.lua#L418-L420)）。
- 重要副作用：建筑 `teleported`/`teleport_move`/`onremove`/`onburnt` 时清空整条链灯（[事件](../../../raw/current/scripts/components/lightpostpartner.lua#L36-L41)）。
- 修改边界：锁扣写操作仅服务器（各方法先查 `ismastersim`）；`OnRemoveFromEntity` 明确不支持（[注释](../../../raw/current/scripts/components/lightpostpartner.lua#L44-L45)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `InitializeNumShackles(num_entities)` | 双端必调 | 按槽位数建 `net_entity` 数组，脏事件 `shackledentitydirty`（[实现](../../../raw/current/scripts/components/lightpostpartner.lua#L62-L67)）。 |
| `SetType(prefab)` | 字符串 | 记录挂点类型供匹配（[实现](../../../raw/current/scripts/components/lightpostpartner.lua#L69-L71)）。 |
| `ShacklePartnerToNextID(partner)` / `ShacklePartnerToID(partner, id)` | 仅服务器 | 占槽并写 `partner.shackle_id`（[实现](../../../raw/current/scripts/components/lightpostpartner.lua#L85-L101)）。 |
| `UnshackleAll()` | 仅服务器 | 清空全部槽位并清伙伴的 `shackle_id`（[实现](../../../raw/current/scripts/components/lightpostpartner.lua#L103-L115)）。 |
| `GetNextAvailableShackleID()` / `GetShackleIdForPartner(partner)` | 双端可读 | 找空槽 / 反查伙伴所在槽（[实现](../../../raw/current/scripts/components/lightpostpartner.lua#L73-L81)、[反查](../../../raw/current/scripts/components/lightpostpartner.lua#L49-L56)）。 |
| `IsMultiShackled()` | → `shackled_entities or nil` | 是否已初始化槽位（[实现](../../../raw/current/scripts/components/lightpostpartner.lua#L58-L60)）。 |
| `OnSave` / `LoadPostPass` | 存读钩子 | 保存槽位实体 GUID；读档后重锁， burnt 建筑跳过（[读档](../../../raw/current/scripts/components/lightpostpartner.lua#L135-L144)）。 |

## 典型接入

### 给自定义建筑挂灯笼伙伴

- 前提与选择：`AddComponent` 与 `InitializeNumShackles` 必须在 prefab 双端构造段完成（net 实体每台机器都要建），原版茶铺即在 `SetPristine` 前注册（[注册](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L361-L362)）。
- 生效与撤销：服务器占槽后网络变量自动同步；伙伴离开或建筑事件触发时由原版灯笼流程 `UnshackleAll`。
- 原版依据：隐士屋同样在构造段注册并设置类型（[参照](../../../raw/current/scripts/prefabs/hermithouse.lua#L804)）。

```lua
-- 自定义灯杆 prefab（构造段，双端执行）
inst:AddComponent("lightpostpartner")
inst.components.lightpostpartner:InitializeNumShackles(2)
inst.components.lightpostpartner:SetType("lantern_post")
-- 服务器侧挂伙伴：
-- inst.components.lightpostpartner:ShacklePartnerToNextID(lantern)
```

- 调整与验证：伙伴按 `post_type` 匹配挂点的逻辑在灯笼 prefab 内（[匹配](../../../raw/current/scripts/prefabs/lantern_posts.lua#L13-L17)），换类型需同步。

## 常见陷阱与限制

- 传送/燃烧即断链：`RemoveChainLights` 会沿 `neighbour_lights` 递归摘除整条链上的灯并置空对端槽位（[断链](../../../raw/current/scripts/components/lightpostpartner.lua#L3-L24)），依赖链灯的建筑要自行重建。
- 槽位上限在 `InitializeNumShackles` 时定死；`ShacklePartnerToNextID` 在无空槽时不动作（[守卫](../../../raw/current/scripts/components/lightpostpartner.lua#L97-L100)）。
- `LoadPostPass` 只重锁存活的伙伴且跳过 `burnt` 建筑（[守卫](../../../raw/current/scripts/components/lightpostpartner.lua#L136)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 lantern_posts、hermitcrab_teashop、hermithouse 三处消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
