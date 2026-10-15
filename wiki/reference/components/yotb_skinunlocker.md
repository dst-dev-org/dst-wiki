---
title: "yotb_skinunlocker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "在 YOTB 皮肤蓝图上记录可解锁的皮肤名，玩家使用后解锁对应皮肤集并消耗蓝图。"
aliases: ["yotb_skinunlocker", "YOTB 皮肤蓝图", "皮肤解锁物品"]
tags: [component, item]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/yotb_skinunlocker.lua", "prefabs/yotb_blueprints.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# yotb_skinunlocker

## 快速判断

- 职责与适用：极薄的皮肤名载体，只存 `skin` 字符串供 YOTB_UNLOCKSKIN 动作读取；作用对象是 YOTB 缝纫包蓝图。[构造](../../../raw/current/scripts/components/yotb_skinunlocker.lua#L1-L3)
- 优先使用：预制体挂载后立即 `SetSkin(data.skin_name)`；解锁逻辑全部在 ACTIONS.YOTB_UNLOCKSKIN.fn 内，组件不负责校验与消耗。[挂载](../../../raw/current/scripts/prefabs/yotb_blueprints.lua#L80-L81)
- 重要副作用：解锁成功会 `Remove()` 蓝图实体；重复解锁返回 `ALREADYKNOWN` 且同样移除蓝图。[动作](../../../raw/current/scripts/actions.lua#L5634-L5639)
- 修改边界：组件无存读、无网络变量；解锁集合操作依赖 doer 上的 `YOTB_unlockskinset`／`YOTB_issetunlocked` 函数（玩家侧安装），其他实体挂载组件也无法完成解锁。[动作条件](../../../raw/current/scripts/actions.lua#L5621-L5624)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetSkin(skin)` | `skin: string` | 记录蓝图对应的皮肤标识，写入 `self.skin`；预制体构造时以 `data.skin_name` 调用。[实现](../../../raw/current/scripts/components/yotb_skinunlocker.lua#L5-L7) [调用](../../../raw/current/scripts/prefabs/yotb_blueprints.lua#L81) |
| `GetSkin()` | `-> string|nil` | 返回记录值，未设置时为 nil；动作侧直接转交 `YOTB_unlockskinset`。[实现](../../../raw/current/scripts/components/yotb_skinunlocker.lua#L9-L11) |

## 典型接入

### 解锁 YOTB 皮肤蓝图对应的皮肤集

- 前提与选择：蓝图由 yotb_blueprints.lua 工厂批量生成，每个 prefab 携带自己的 `skin_name`；动作通过 componentactions 对玩家登记，无额外技能或场景条件。[工厂](../../../raw/current/scripts/prefabs/yotb_blueprints.lua#L92) [登记](../../../raw/current/scripts/componentactions.lua#L3103-L3108)
- 生效与撤销：使用蓝图时先取 skin；若 `makedoll` 存在先制作布偶，再查 `YOTB_issetunlocked(skin)`——已解锁返回 `ALREADYKNOWN` 并移除蓝图，未解锁则 `YOTB_unlockskinset(skin)` 后移除。[动作流程](../../../raw/current/scripts/actions.lua#L5621-L5639)
- 原版依据：解集合函数由玩家侧安装（与 luckuser 等同层的 player_common 挂载习惯）；组件仅是被动作读取的数据袋。

## 常见陷阱与限制

- **以为组件负责皮肤校验或存档：** 组件只有 SetSkin/GetSkin 两个方法，无 OnSave/OnLoad、无标签；皮肤集合的持久化在玩家侧数据，蓝图本身被消耗后不保留状态。[完整定义](../../../raw/current/scripts/components/yotb_skinunlocker.lua#L1-L11)
- **在非玩家目标上期待解锁：** componentactions 仅对 `doer:HasTag("player")` 登记 YOTB_UNLOCKSKIN，动作 fn 还要求 doer 存在 `YOTB_unlockskinset`；给 NPC 或怪物挂本组件不会产生可用动作。[登记](../../../raw/current/scripts/componentactions.lua#L3103-L3108) [动作条件](../../../raw/current/scripts/actions.lua#L5621-L5624)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（13 行），核对 yotb_blueprints 工厂挂载与 makedoll、YOTB_UNLOCKSKIN 全流程、componentactions 登记。未审阅 YOTB 事件本身的皮肤集数据结构与时装系统。
- 结构检查：2026-10-28 validate_page.py 通过（yotb_skinunlocker.md）；validate_source_links.py --strict 本页全部源码锚点通过；本页无完整 Lua 示例。
- 游戏运行：未运行；蓝图解锁与重复使用分支未实测。
