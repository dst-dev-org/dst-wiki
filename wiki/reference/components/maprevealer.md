---
title: "maprevealer"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "周期性把实体周围地图区域揭示给玩家（经 player_classified.MapExplorer），支持全体广播或指定私人拥有者两种模式。"
aliases: ["地图揭示器", "地图探索", "maprevealer"]
tags: [component]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/maprevealer.lua", "components/maprevealable.lua", "prefabs/deerclopseyeball_sentryward.lua", "prefabs/sentryward.lua", "prefabs/wx78_drone_scout.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# maprevealer

## 快速判断

- 职责与适用：给建筑／哨戒类实体挂上"持续揭示周边地图"的能力，默认每 5 秒把自身位置区域揭示给目标玩家；目标可为全体或 `SetPrivateOwner` 指定的一人。[字段](../../../raw/current/scripts/components/maprevealer.lua#L1-L7)
- 优先使用：开关用 `Start(delay)`／`Stop()`；私人模式用 `SetPrivateOwner(owner)`／`GetPrivateOwner()`；私人揭示被"消费"后重置冷却用 `RestartPrivateRevealCooldown()`。[Start](../../../raw/current/scripts/components/maprevealer.lua#L60-L65)
- 重要副作用：`Start` 给实体加 `maprevealer` tag、`Stop` 摘除——maprevealable 靠该 tag 找揭示源；揭示写 `player_classified.MapExplorer:RevealArea`。[RevealMapToPlayer](../../../raw/current/scripts/components/maprevealer.lua#L79-L87)
- 修改边界：构造即自启（`self:Start()`），移除组件等价 `Stop`；揭示目标是玩家实体，组件本体不判服务端但调用链均为服务端。[L117](../../../raw/current/scripts/components/maprevealer.lua#L117)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `revealperiod` | 秒，默认 5 | 单个玩家的揭示间隔；多人公共模式下拆分为 `revealperiod/#players` 轮流揭示。[OnStart](../../../raw/current/scripts/components/maprevealer.lua#L50-L54) |
| `Start(delay)` | 无返回 | 已在跑则 no-op；加 tag 后延迟 `delay or random()*0.5` 启动首轮。[L60-L65](../../../raw/current/scripts/components/maprevealer.lua#L60-L65) |
| `Stop()` | 无返回 | 摘 tag、取消任务、清玩家缓存；`OnRemoveFromEntity` 即本方法。[L67-L77](../../../raw/current/scripts/components/maprevealer.lua#L67-L77) |
| `RevealMapToPlayer(player)` | 无返回 | 玩家握手未就绪（非 `POSTACTIVATEHANDSHAKE.READY`）时跳过；就绪则 `player_classified.MapExplorer:RevealArea(自身坐标)`。[L79-L87](../../../raw/current/scripts/components/maprevealer.lua#L79-L87) |
| `SetPrivateOwner(owner)`／`GetPrivateOwner()` | 实体或 nil | 指定后只揭示该玩家（无效拥有者会自停）；变更时重启循环。[L89-L102](../../../raw/current/scripts/components/maprevealer.lua#L89-L102) |
| `RestartPrivateRevealCooldown()` | 无返回 | 取消当前轮并延迟 `revealperiod` 重启；wx78 侦察无人机在追踪者读图后调用防连续刷图。[L104-L115](../../../raw/current/scripts/components/maprevealer.lua#L104-L115)、[调用](../../../raw/current/scripts/prefabs/wx78_drone_scout.lua#L193) |

## 典型接入

### 原版宿主：三种模式参照

- 公共广播：sentryward 与 deerclopseyeball_sentryward 构建时 `AddComponent("maprevealer")` 并在 pristine 态预加 `maprevealer` tag；眼球哨戒塔在装上眼球时 `Start()`、取下时 `Stop()`，烧毁也 `Stop()`。[挂载](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L473)、[启停](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L182)、[烧毁](../../../raw/current/scripts/prefabs/sentryward.lua#L36)
- 私人模式：wx78_drone_scout 无人机构造后 `SetPrivateOwner(inst)`（自己），被追踪时改 `SetPrivateOwner(tracker)`。[挂载](../../../raw/current/scripts/prefabs/wx78_drone_scout.lua#L319-L320)、[改主](../../../raw/current/scripts/prefabs/wx78_drone_scout.lua#L230)
- tag 消费：`maprevealable` 在 `PLAYER_REVEAL_RADIUS` 内按 `maprevealer` tag 找揭示源；有私人拥有者则按玩家记私有源，否则记公共源 `maprevealer`。[扫描](../../../raw/current/scripts/components/maprevealable.lua#L194-L209)

### 模组：可揭示地图的哨塔

- 前提与选择：服务端建筑实体；若希望地图上同时显示图标/被 maprevealable 感知，pristine 态加 `maprevealer` tag。
- 生效与撤销：`AddComponent` 后自动开始；`Stop()` 或移除组件即停。

```lua
-- 某建筑 prefab 的 master 段
inst:AddTag("maprevealer") -- pristine 段预加，供 maprevealable 扫描
-- master init：
inst:AddComponent("maprevealer")
inst.components.maprevealer.revealperiod = 10
-- 私人模式（可选）：
-- inst.components.maprevealer:SetPrivateOwner(owner)
```

- 调整与验证：示例仅语法检查未运行；`revealperiod` 单位为秒，揭示半径由 `MapExplorer:RevealArea` 内部决定，本组件不暴露半径参数。

## 常见陷阱与限制

- 揭示排队逐人进行：`OnRevealing` 每次取一名玩家、间隔 `delay` 后下一名；玩家列表缓存 `_players` 只在 Stop 清空，长运行下玩家离场由 `IsValid()` 跳过。[L21-L38](../../../raw/current/scripts/components/maprevealer.lua#L21-L38)
- tag 与组件可能脱节：预加 tag 的 prefab 若未加组件，maprevealable 会把无组件实体当公共揭示源（其注释明确提到 mod 错用 tag 的兼容分支）；反之 `Stop` 会摘 tag 导致 maprevealable 失去源。[兼容注释](../../../raw/current/scripts/components/maprevealable.lua#L196-L198)
- 私人拥有者失效不重启：拥有者无效时首轮 `OnStart` 直接 `Stop()`；之后即使拥有者恢复有效也需重新 `SetPrivateOwner` 触发。[L43-L49](../../../raw/current/scripts/components/maprevealer.lua#L43-L49)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖 maprevealer.lua 全文（含任务循环、私人模式、tag 管理）与三个宿主 prefab 的挂载/启停段、maprevealable 的 tag 扫描段；未覆盖 player_classified.MapExplorer 的揭示半径实现。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 通过（见批记录）。

相关：[[reference/components/mapspotrevealer]]。

## 相关链接

- [[reference/components/maprevealable]]：全图图标挂载端：公共/私人揭示来源。
