---
title: "moontrader"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "单向献祭交易组件：校验 canaccept 后收走祭品并触发 onaccept，用于月重掷门这类喂了就没的供品入口。"
aliases: ["月亮献祭交易", "月重掷供品", "moontrader"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/moontrader.lua", "actions.lua", "componentactions.lua", "prefabs/multiplayer_portal.lua", "prefabs/moonrockidol.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# moontrader

## 快速判断

- 职责与适用：目标侧的供品接收组件：`AcceptOffering(giver, item)` 校验→取走→回调→移除祭品，返回 true；被拒绝时返回 `false, reason`。原版宿主是多人重掷门 `multiplayer_portal`（月石祭坛）。
- 优先使用：走原版 GIVE 动作链（玩家手持 `moonrelic` 物品对带 `moontrader` tag 的目标右键）；自定义祭品需要同时实现目标侧组件与物品侧动作注册。
- 重要副作用：构造即在实体上加 `moontrader` tag，注释建议放进 prefab 的 pristine 段（[L6-L7](../../../raw/current/scripts/components/moontrader.lua#L6-L7)）；`componentactions` 用该 tag 过滤（[L1511](../../../raw/current/scripts/componentactions.lua#L1511)）。
- 修改边界：纯服务端组件（宿主在 master 分支添加，[multiplayer_portal.lua#L400-L402](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L400-L402)）；无存读钩子，回调不入存档。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetCanAcceptFn(fn)` | `fn(inst, item, giver)` → success, reason | 可选前置校验；失败时 `AcceptOffering` 原样返回 `false, reason`，物品不动。[L10-L12](../../../raw/current/scripts/components/moontrader.lua#L10-L12)、[L19-L24](../../../raw/current/scripts/components/moontrader.lua#L19-L24) |
| `SetOnAcceptFn(fn)` | `fn(inst, giver, item)` | 校验通过、物品已脱离原主人后调用，随后 `item:Remove()`。[L14-L16](../../../raw/current/scripts/components/moontrader.lua#L14-L16)、[L32-L36](../../../raw/current/scripts/components/moontrader.lua#L32-L36) |
| `AcceptOffering(giver, item)` | boolean, reason | 堆叠物品取一个（`stackable:Get()`），否则 `inventoryitem:RemoveFromOwner(true)`。[L18-L39](../../../raw/current/scripts/components/moontrader.lua#L18-L39) |

## 典型接入

### 原版月重掷门

- 前提与选择：`multiplayer_portal` 加组件并设置两个回调：`moonrock_canaccept` 要求物品带 `moonportalkey` tag、洞穴里拒绝（reason `"NOMOON"`）；`moonrock_onaccept` 推 `ms_playerreroll` 并清空玩家及随从的库存／容器／插槽（[canaccept](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L355-L362)、[onaccept](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L364-L385)）。
- 动作链：GIVE 动作在 USEITEM 分支由物品侧 `moonrelic` 组件注册——目标带 tag 且玩家未骑乘才出现（[componentactions.lua#L1510-L1513](../../../raw/current/scripts/componentactions.lua#L1510-L1513)）；动作执行分流到 `AcceptOffering`（[actions.lua#L2330-L2331](../../../raw/current/scripts/actions.lua#L2330-L2331)）。祭品本体 `moonrockidol` 带 `moonportalkey` tag 与 `moonrelic` 组件（[moonrockidol.lua#L76](../../../raw/current/scripts/prefabs/moonrockidol.lua#L76)、[L89](../../../raw/current/scripts/prefabs/moonrockidol.lua#L89)）。

## 常见陷阱与限制

- 校验通过后必然移除祭品：`onaccept` 拿到的 item 已脱离主人，回调结束后 `item:Remove()`，没有退回路径；需要"验看再决定"的逻辑必须放进 `canaccept`。
- GIVE 动作的出现条件绑在物品组件 `moonrelic` 上：给不带该组件的物品接 `moontrader` 目标，需要在物品侧自建 componentaction，只靠目标 tag 不会出动作。
- 堆叠祭品只消费一个，剩余留在玩家身上（[L26-L27](../../../raw/current/scripts/components/moontrader.lua#L26-L27)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/moontrader.lua（41 行），核对 multiplayer_portal 回调全貌、GIVE 动作分流、USEITEM 物品侧注册与 moonrockidol 祭品配置。未覆盖重掷后角色创建流程。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；51 行、4658 字节，simple 档内）；validate_source_links.py --strict 通过（15 处源码引用，0 error／0 warning）；本页无完整 Lua 代码块。
