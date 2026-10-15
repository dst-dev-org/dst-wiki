---
title: "kitcoon"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "小猫可的空标记组件，让 PET 动作命中后向实体推 on_petted 事件；自身不存任何状态。"
aliases: ["kitcoon", "小猫可宠物标记", "宠物动作标记", "on_petted 触发"]
tags: [component, creature]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/kitcoon.lua", "prefabs/kitcoon.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# kitcoon

## 快速判断

- 职责与适用：空组件，无字段、无方法逻辑，仅作为"该实体走 kitcoon 专属宠物路径"的服务端标记。[构造](../../../raw/current/scripts/components/kitcoon.lua#L1-L4)
- 优先使用：想让 PET 动作命中后触发小猫可专属反应时，由 kitcoon 预制体挂载本组件；ACTIONS.PET.fn 检查 `components.kitcoon ~= nil` 后向目标推 `on_petted`，预制体监听该事件唤醒并结为跟随者。[动作](../../../raw/current/scripts/actions.lua#L4243-L4245) [监听](../../../raw/current/scripts/prefabs/kitcoon.lua#L297)
- 重要副作用：PET 命中只推 `on_petted` 事件，交由监听者决定效果；组件本身不改状态、不发网络变量。[事件](../../../raw/current/scripts/prefabs/kitcoon.lua#L129-L134)
- 修改边界：狗窝寄存、RETURN_FOLLOWER 等归属逻辑在 `kitcoonden` 组件与 kitcoon 预制体内，不在本组件；模组不能靠挂本组件获得完整小猫可行为。[狗窝路径](../../../raw/current/scripts/actions.lua#L4250-L4258)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `Class(function(self, inst))` | 仅保存 `self.inst`，无任何字段初始化；非 netvar，不参与存读。[实现](../../../raw/current/scripts/components/kitcoon.lua#L1-L4) |
| `GetDebugString()` | `-> ""` | 调试信息固定返回空串，无业务消费者。[实现](../../../raw/current/scripts/components/kitcoon.lua#L6-L8) |

## 典型接入

### 判定 PET 动作的目标是否走小猫可路径

- 前提与选择：服务端；ACTIONS.PET 由 crittertraits 的 componentactions 条目对跟随者登记，本组件只在动作执行时区分"普通宠物"与"小猫可"。[动作登记](../../../raw/current/scripts/componentactions.lua#L327-L338)
- 生效与撤销：挂载后 PET 命中即推 `on_petted`（携带 `doer`）；原版监听者唤醒 sleeper 并在无跟随者时 `makefriend` + AddFollower。撤销即移除组件，动作回落到普通宠物分支。[监听实现](../../../raw/current/scripts/prefabs/kitcoon.lua#L129-L134)
- 原版依据：prefabs/kitcoon.lua 在服务端 `AddComponent("kitcoon")` 后注册监听。[挂载](../../../raw/current/scripts/prefabs/kitcoon.lua#L267)

## 常见陷阱与限制

- **把 kitcoonden 当成本组件的接口：** 狗窝的 `AddKitcoon`／`RemoveKitcoon`、`num_kitcoons` 归 kitcoonden 组件；本组件没有这些方法，跨组件混用会直接索引 nil。[狗窝组件](../../../raw/current/scripts/prefabs/kitcoonden.lua#L43)
- **以为挂上组件就能被 PET：** PET 动作的登记条件是 `replica.follower` 的跟随者关系（crittertraits 条目），组件只影响命中后的行为；无跟随关系的目标不会出现 PET。[登记条件](../../../raw/current/scripts/componentactions.lua#L327-L338)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 components/kitcoon.lua（10 行），核对 prefab 挂载、ACTIONS.PET 命中分支、on_petted 监听与 RETURN_FOLLOWER 的 kitcoonden 路径。未审阅小猫可任务、玩具玩耍等其余 prefab 逻辑。
- 结构检查：2026-10-28 validate_page.py 通过（kitcoon.md）；validate_source_links.py --strict 本页全部源码锚点通过；本页无完整 Lua 示例。
- 游戏运行：未运行；PET 事件链与自定义监听未实测。
