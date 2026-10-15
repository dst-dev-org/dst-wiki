---
title: "slingshotmodder"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "弹弓改装工具的入口组件，校验目标归属后调用目标弹弓 slingshotmods 的 Open/Close 打开或关闭改装界面。"
aliases: ["slingshotmodder", "弹弓改装工具", "弹弓模组套件", "MODSLINGSHOT"]
tags: [component, item]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/slingshotmodder.lua", "prefabs/slingshotmodkit.lua", "actions.lua", "componentactions.lua", "components/slingshotmods.lua", "components/linkeditem.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# slingshotmodder

## 快速判断

- 职责与适用：挂在使用工具（`slingshotmodkit`）上，把"改装弹弓"意图转发给目标弹弓的 `slingshotmods` 组件；组件自身不管理改装内容。[挂载](../../../raw/current/scripts/prefabs/slingshotmodkit.lua#L60)
- 优先使用：动作侧 MODSLINGSHOT 选定目标（先查手装弹弓，再查背包带 `slingshot` 标签的物品）后调 `StartModding(target, user)`；结束改装走 STOPMODSLINGSHOT → `StopModding`。[选定目标](../../../raw/current/scripts/actions.lua#L6420-L6435)
- 重要副作用：目标弹弓被 linkeditem 限定主人且 user 不是主人时返回 `false, "NOT_MINE"`；Open/Close 的实际条件（Walter 技能 `walter_slingshot_modding`、物品 GrandOwner）在 slingshotmods 侧。[归属拦截](../../../raw/current/scripts/components/slingshotmodder.lua#L5-L11) [打开条件](../../../raw/current/scripts/components/slingshotmods.lua#L12-L22)
- 修改边界：无字段、无存读；目标是"装有 slingshotmods 组件的弹弓"，对其他实体调用时表达式短路返回 false。[转发](../../../raw/current/scripts/components/slingshotmodder.lua#L12-L17)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartModding(target, user)` | `-> bool` 或 `bool, reason` | 目标有 linkeditem 且 `IsEquippableRestrictedToOwner()` 时校验 `GetOwnerUserID()` 与 `user.userid`，不符返回 `false,"NOT_MINE"`；否则返回 `target.components.slingshotmods:Open(user)`（无该组件时 false）。[实现](../../../raw/current/scripts/components/slingshotmodder.lua#L5-L13) |
| `StopModding(target, user)` | `-> bool` | 转发 `slingshotmods:Close(user)`；无 slingshotmods 时 false。[实现](../../../raw/current/scripts/components/slingshotmodder.lua#L15-L17) |

## 典型接入

### 原版改装会话的完整动作链（参照实现同类工具）

- 前提与选择：modkit 物品另挂 `useabletargeteditem` 并 `SetUseableMounted(true)`；componentactions 的 slingshotmodder 条目在"已打开 slingshotmodscontainer"时给 STOPMODSLINGSHOT，否则要求 Walter 技能激活才给 MODSLINGSHOT。[工具挂载](../../../raw/current/scripts/prefabs/slingshotmodkit.lua#L57-L60) [动作登记](../../../raw/current/scripts/componentactions.lua#L2972-L2986)
- 生效与撤销：MODSLINGSHOT 找目标 → StartModding 打开弹弓改装容器；停止时遍历 `inventory.opencontainers` 找 `prefab == "slingshotmodscontainer"` 且带 `install_target` 的容器，对其目标 StopModding。[停止](../../../raw/current/scripts/actions.lua#L6440-L6446)
- 原版依据：改装内容、技能门控与 GrandOwner 校验都在 slingshotmods:Open/Close 与 CanBeOpenedBy；本组件是动作与弹弓之间的薄中转。[打开条件](../../../raw/current/scripts/components/slingshotmods.lua#L12-L22)

## 常见陷阱与限制

- **以为组件负责技能或主人校验：** 组件只做 linkeditem 归属一道拦截（NOT_MINE）；Walter 技能、物品归属等在 slingshotmods:CanBeOpenedBy，未激活技能时 MODSLINGSHOT 根本不会登记。[登记条件](../../../raw/current/scripts/componentactions.lua#L2983-L2986) [CanBeOpenedBy](../../../raw/current/scripts/components/slingshotmods.lua#L12-L22)
- **把改装界面关闭算在工具身上：** StopModding 需要传入打开的 `install_target`；动作侧从 opencontainers 反查容器，找不到已打开的改装容器就直接失败，与工具在不在背包无关。[停止](../../../raw/current/scripts/actions.lua#L6440-L6446)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（19 行），核对 modkit 挂载、MODSLINGSHOT/STOPMODSLINGSHOT、componentactions 登记、slingshotmods:Open 关联的 CanBeOpenedBy 与 linkeditem 归属方法。未审阅改装件安装的容器与部件数据流。
- 结构检查：2026-10-28 validate_page.py 通过（slingshotmodder.md）；validate_source_links.py --strict 本页全部源码锚点通过；本页无完整 Lua 示例。
- 游戏运行：未运行；改装开启/关闭与 NOT_MINE 分支未实测。
