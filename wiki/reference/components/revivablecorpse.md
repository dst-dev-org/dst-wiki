---
title: "revivablecorpse"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "可复活尸体组件：标记实体为可被复活的 corpse，提供复活资格判定与复活事件推送，双端可用。"
aliases: ["可复活尸体", "尸体复活", "revivablecorpse"]
tags: [component, creature, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/revivablecorpse.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "prefabs/lavaarena_bernie.lua", "actions.lua", "componentactions.lua", "gamemodes.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# revivablecorpse

## 快速判断

- 职责与适用：文件头注明双端运行（[注释](../../../raw/current/scripts/components/revivablecorpse.lua#L1)）；原版挂在玩家（游戏模式属性 `revivable_corpse` 时，([player_common.lua:2471-2473](../../../raw/current/scripts/prefabs/player_common.lua#L2471-L2473)）与熔炉伯尼（([lavaarena_bernie.lua:41-42](../../../raw/current/scripts/prefabs/lavaarena_bernie.lua#L41-L42)）上，支持"留下尸体、他人救助复活"玩法。
- 优先使用：资格判定 `CanBeRevivedBy(reviver)`（要求宿主有 `corpse` 标签），复活走 `Revive(reviver)`（[实现](../../../raw/current/scripts/components/revivablecorpse.lua#L24-L27)、[L75-L79](../../../raw/current/scripts/components/revivablecorpse.lua#L75-L79)）。
- 重要副作用：`Revive` 只推 `respawnfromcorpse` 事件（data 含 source/user），实际复活由玩家侧监听器完成（[监听](../../../raw/current/scripts/prefabs/player_common.lua#L2658)）。
- 修改边界：挂该组件的游戏模式强制禁用幽灵（[gamemodes.lua:291](../../../raw/current/scripts/gamemodes.lua#L291），两种复活体系互斥。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CanBeRevivedBy(reviver)` | `-> bool` | `corpse` 标签 + `canberevivedbyfn` 放行；客户端 playercontroller 也用它做目标校验（[实现](../../../raw/current/scripts/components/revivablecorpse.lua#L24-L27)、[调用](../../../raw/current/scripts/components/playercontroller.lua#L2001-L2002)）。 |
| `SetCanBeRevivedByFn(fn)` | `fn(inst, reviver) -> bool` | 宿主自定义资格（伯尼用它限制可复活状态）（[注册](../../../raw/current/scripts/prefabs/lavaarena_bernie.lua#L42)）。 |
| `Revive(reviver)` | 服务端 | 推 `respawnfromcorpse` 事件后不做其他事（[实现](../../../raw/current/scripts/components/revivablecorpse.lua#L75-L79)）。 |
| `SetCorpse(corpse)` | 服务端 | 增删 `corpse` 标签；玩家死亡／复活流程在 extensions 调用（[实现](../../../raw/current/scripts/components/revivablecorpse.lua#L65-L73)、[置位](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L786)、[清除](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L474)）。 |
| `SetReviveHealthPercent(p)` / `GetReviveHealthPercent()` | 服务端；默认 `0.5` | 复活血量系数；读侧用于重生回血（[读侧](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L448)）。 |
| `SetReviveSpeedMult(m)` / `SetReviveSpeedMultForTag(tag, m)` / `GetReviveSpeedMult(reviver)` | 服务端 | 复活速度倍率与按 reviver 标签的乘区；**本快照无外部调用者**（[实现](../../../raw/current/scripts/components/revivablecorpse.lua#L32-L63)）。 |

## 典型接入

### 复活动作链路（原版参照）

- 前提与选择：动作候选按组件存在性给出 REVIVE_CORPSE（([componentactions.lua:736-738](../../../raw/current/scripts/componentactions.lua#L736-L738)），动作执行静默失败也先判资格（([actions.lua:4131-4136](../../../raw/current/scripts/actions.lua#L4131-L4136)）。
- 生效与撤销：死亡时 `SetCorpse(true)` 打标，复活后 `SetCorpse(false)` 摘标（[extensions 两处](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L474)）。

```lua
-- 服务端：给自定义同伴接"尸体复活"
inst:AddComponent("revivablecorpse")
inst.components.revivablecorpse:SetCanBeRevivedByFn(function(inst, reviver)
    return reviver:HasTag("player") -- 自定义资格条件
end)
-- 触发复活（等价 REVIVE_CORPSE 动作）：
inst.components.revivablecorpse:Revive(doer) -- 内部推 respawnfromcorpse
```

- 调整与验证：组件本身不处理尸体外观／倒计时；自定义宿主需自建死亡到 corpse 标记的流程。

## 常见陷阱与限制

- **速度与血量 API 大多闲置**：`SetReviveSpeedMult`、`SetReviveSpeedMultForTag`、`GetReviveSpeedMult` 在本快照无调用者，属预留接口（全库 grep 复核）。
- **字段名拼写保留**：复活血量字段源码写作 `revive_health_percet`（percet），引用时按源码拼写（[字段](../../../raw/current/scripts/components/revivablecorpse.lua#L11)）。
- **Revive 不保证成功**：动作层静默吞掉资格失败；直接调用 Revive 前自行判 `CanBeRevivedBy`（[动作](../../../raw/current/scripts/actions.lua#L4132-L4135)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、两处挂接、动件候选与执行、playercontroller 客户端判定、extensions 置标／回血、gamemodes 幽灵互斥。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/spectatorcorpse]]：尸体观战相机组件。
