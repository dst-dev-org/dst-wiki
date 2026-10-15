---
title: "strongman"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "记录 Wolfgang 使用大力士健身房的状态并挂 ingym 标记，进入时暂停 mightiness、离开时恢复；仅 Wolfgang 挂载。"
aliases: ["strongman", "大力士健身房状态", "ingym 标记", "健身房会话"]
tags: [component, state-machine]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/strongman.lua", "prefabs/wolfgang.lua", "components/mightygym.lua", "components/mightiness.lua", "stategraphs/SGwilson.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# strongman

## 快速判断

- 职责与适用：会话状态组件，字段只有 `gym`（当前健身房实体引用）；由 mightygym 在开始/结束锻炼时调用，配合 `ingym` 标签供状态图与动作读取。[实现](../../../raw/current/scripts/components/strongman.lua#L6-L16)
- 优先使用：角色进入健身房流程走 `mightygym:StartWorkout(doer)` → `strongman:DoWorkout(gym)`；退出走 StopWorkout 对称路径，不要绕过 mightygym 直接调。[调用方](../../../raw/current/scripts/components/mightygym.lua#L283-L287)
- 重要副作用：DoWorkout 调 `mightiness:Pause()` 并加 `ingym` 标签；StopWorkout 移除标签、`mightiness:Resume()` 并清空 gym。ingym 标签被 SGwilson locomote、mightiness 等多处消费。[暂停](../../../raw/current/scripts/components/mightiness.lua#L160-L165) [标签消费](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1817-L1822)
- 修改边界：无存读、无网络变量；原版仅 Wolfgang 挂载（与 mightiness 同批）。其他角色挂本组件必须同时有 mightiness，否则 DoWorkout 直接索引 nil。[挂载](../../../raw/current/scripts/prefabs/wolfgang.lua#L667-L670)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `gym` | `entity|nil` | 当前使用的 mightygym 实体；StartWorkout 在调 DoWorkout 前先写 `self.strongman.gym = self.inst`，退出置 nil。[写入方](../../../raw/current/scripts/components/mightygym.lua#L284-L287) |
| `DoWorkout(gym)` | `gym: entity` -> `nil` | 保存 gym、`components.mightiness:Pause()`、`AddTag("ingym")`；无返回值、不校验重复进入（由 mightygym 的 `self.strongman == nil` 守卫）。[实现](../../../raw/current/scripts/components/strongman.lua#L6-L10) |
| `StopWorkout()` | `-> nil` | 移除 ingym、`mightiness:Resume()`、清 gym；顺序为先撤标签再恢复。[实现](../../../raw/current/scripts/components/strongman.lua#L12-L16) |

## 典型接入

### 复刻健身房占用流程（原版参照）

- 前提与选择：健身房侧组件 mightygym 用 `self.strongman` 单槽占用：`self.strongman == nil and doer.gym == nil` 才 StartWorkout，并把饥饿burnrate 修正、皮肤模式一并设置；strongman 组件只承担"角色侧状态"。[StartWorkout](../../../raw/current/scripts/components/mightygym.lua#L283-L296)
- 生效与撤销：锻炼结束由 mightygym:StopWorkout 统一恢复健身房外观、移除饥饿修正并调 `strongman:StopWorkout()`；玩家主动离开走 LEAVE_GYM 动作或 SGwilson locomote 里的 ingym 分支强制退出。[结束](../../../raw/current/scripts/components/mightygym.lua#L309-L320) [LEAVE_GYM](../../../raw/current/scripts/actions.lua#L5771-L5777)
- 原版依据：ingym 标签还参与 mightiness 的形态切换守卫与 Wolfgang 专属分支，移除组件前必须先走正常退出，否则标签与 mightiness 暂停状态残留。[消费](../../../raw/current/scripts/components/mightiness.lua#L359)

## 常见陷阱与限制

- **绕过 mightygym 直接 DoWorkout/StopWorkout：** gym 引用的写入与占用互斥都在 mightygym 里；直接调用会造成 `gym.components.mightygym.strongman` 与 `player.components.strongman.gym` 不一致，退出流程（trytoexitgym 读 `.gym`）会失配。[退出读取](../../../raw/current/scripts/components/mightygym.lua#L330-L334)
- **给无 mightiness 的角色挂组件：** DoWorkout/StopWorkout 都不判空 `components.mightiness`；非 Wolfgang 角色复用需自行保证该组件存在或改写调用链。[实现](../../../raw/current/scripts/components/strongman.lua#L6-L16)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（18 行），核对 Wolfgang 挂载、mightygym 开始/结束/退出路径、ingym 标签的全部消费点（SGwilson、SGwilson_client、mightiness、wolfgang）。未审阅健身房动作与重量升降的完整状态图。
- 结构检查：2026-10-28 validate_page.py 通过（strongman.md）；validate_source_links.py --strict 本页全部源码锚点通过；本页无完整 Lua 示例。
- 游戏运行：未运行；进出健身房与形态切换联动未实测。
