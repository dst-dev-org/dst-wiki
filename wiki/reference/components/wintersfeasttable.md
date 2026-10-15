---
title: "wintersfeasttable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "冬季盛宴餐桌的开餐开关：canfeast 同步字段驱动 readyforfeast 标签，开餐按人头消耗食物耐久并回调换菜。"
aliases: ["wintersfeasttable", "盛宴餐桌", "冬季盛宴开餐"]
tags: [component, server-client]
size_class: standard
dst_build: "steam-buildid:26230496"
sources: ["components/wintersfeasttable.lua", "prefabs/table_winters_feast.lua", "components/feasts.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wintersfeasttable

## 快速判断

- 职责与适用：挂在冬季盛宴餐桌上管理"能否开餐"。`canfeast` 是同步字段（netvar 回调注册），其变化自动增删实体 `readyforfeast` 标签——这是玩家端右键 WINTERSFEAST_FEAST 动作的唯一开关；置假同时打断正在开餐的玩家。[canfeast 回调](../../../raw/current/scripts/components/wintersfeasttable.lua#L10-L19)
- 优先使用：宿主 prefab（`table_winters_feast.lua:490`）负责挂组件、配 `ondepletefoodfn` 并按桌上有无食物等条件翻转 `canfeast`；批量消耗由 feasts 系统按桌组统一调 `DepleteFood`。[宿主挂载](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L490-L492)
- 重要副作用：开餐状态机在玩家侧——SGwilson 的 `winters_feast_eat` 状态把玩家写进 `current_feasters`，玩家被打断/退出时组件侧 `CancelFeasting` 对每人推 `feastinterrupted`。[玩家注册](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22595-L22596)
- 修改边界：组件消耗的是 1 号槽食物的 `finiteuses` 耐久而非删除物品；`DepleteFood(feasters)` 的参数实际未使用（人数信息由回调方在 ondepletefoodfn 里自行携带）。[消耗实现](../../../raw/current/scripts/components/wintersfeasttable.lua#L65-L81)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `canfeast` | boolean（同步） | 写真加 `readyforfeast` 标签、写假摘标签并 `CancelFeasting`；动作入口按标签过滤（`componentactions.lua:977-980`）。[回调](../../../raw/current/scripts/components/wintersfeasttable.lua#L10-L19) |
| `DepleteFood(feasters)` | feasters: 参数未使用 | 取 `inventory` 1 号槽物品，`finiteuses:Use()` 一次；剩余耐久 >0 调 `ondepletefoodfn`，归零调 `onfinishfoodfn`。依赖宿主有 `inventory` 组件。[实现](../../../raw/current/scripts/components/wintersfeasttable.lua#L65-L81) |
| `ondepletefoodfn` / `onfinishfoodfn` | `fn(inst)` | 食物被消耗一次/耗尽的回调；原版在宿主里换下一道菜与停止开餐（`table_winters_feast.lua:491` 起）。[配置点](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L490-L492) |
| `CancelFeasting()` | `-> nil` | 对 `current_feasters` 中每个有效玩家推 `feastinterrupted`（SGwilson 监听退出开餐态），清空名单。[实现](../../../raw/current/scripts/components/wintersfeasttable.lua#L55-L63) |
| `current_feasters` | table：玩家实体 → true | 由玩家侧状态机写入、组件侧清除；跨端边界属服务端运行期数据。[消费点](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22595-L22596) |
| `feastinterrupted` | 实体事件（推给玩家） | 开餐被打断的通知；`SGwilson.lua:2423` 监听。[监听方](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2423-L2425) |

## 典型接入

### 开餐条件与食物消耗链（原版参照）

- 前提与选择：餐桌需 `inventory`（放食物）与 `shelf`（GetDebugString 读 `shelf.itemonshelf`，仅调试用）；canfeast 翻转时机集中在 `table_winters_feast.lua`（有可用食物、未被占用等分支，如 `:292`、`:335`）。[宿主回调](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L490-L492)
- 生效与撤销：feasts 系统按 `wintersfeasttable` 标签把邻近餐桌分桌组，开餐检查任务统一对每桌 `DepleteFood(#feasters)`（`components/feasts.lua:92`）；食物耐久归零后回调换菜/停餐。

```lua
-- 服务端：自建餐桌的最小接入
inst:AddComponent("wintersfeasttable")
inst.components.wintersfeasttable.ondepletefoodfn = function(inst)
    -- 吃掉一口：换动画/音效；耐久耗尽会走 onfinishfoodfn
end
inst.components.wintersfeasttable.onfinishfoodfn = function(inst)
    inst.components.wintersfeasttable.canfeast = false  -- 摘标签并打断开餐者
end
```

- 调整与验证：消耗节奏由 feasts 的 `doFeastCheck` 周期决定（TICK_RATE 周期任务，`components/feasts.lua:137-138`）；组件自身不计时。

## 常见陷阱与限制

- **直接 AddComponent 不等于能开餐：** 动作可见性要求 `readyforfeast` 标签且实体无 `fire`/`burnt` 标签（`componentactions.lua:978`），标签由 canfeast netvar 驱动——宿主不翻转 canfeast 就永远开不了餐；原版还要求 `wintersfeasttable` 标签在 pristine 阶段预加（`table_winters_feast.lua:459-460`）。[动作条件](../../../raw/current/scripts/componentactions.lua#L977-L980)
- **没有 inventory 组件就调 DepleteFood：** `GetItemInSlot(1)` 会在 nil 上索引报错；组件不自带存储。[读取点](../../../raw/current/scripts/components/wintersfeasttable.lua#L66)
- **食物无 finiteuses：** DepleteFood 对无耐久食物静默跳过（整段包在 `finiteuses ~= nil` 判断内），既不回调也不消耗；放普通食物无效。[守卫条件](../../../raw/current/scripts/components/wintersfeasttable.lua#L68)
- **GetDebugString 依赖 shelf：** 调试字符串直接读 `shelf.itemonshelf`，无 shelf 组件的宿主在调试输出时会报错；仅影响调试不影响玩法。[读取点](../../../raw/current/scripts/components/wintersfeasttable.lua#L41)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（82 行），核对 table_winters_feast 的 canfeast 翻转与回调配置、feasts.lua:92 消耗调用与桌组注册（`feasts.lua:214-225`）、componentactions 动作条件、SGwilson 开餐状态与打断事件；`DepleteFood` 参数未使用经函数体确认。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
