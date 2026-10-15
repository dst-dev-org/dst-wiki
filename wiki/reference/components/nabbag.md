---
title: "nabbag"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "抓取袋（Wortox 技能物品）：对主目标执行 PICKUP/NET 后把成功复制到前方锥形范围内的同类目标，受数量与耐久上限约束。"
aliases: ["抓取袋", "nabbag"]
tags: [component, item, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/nabbag.lua", "actions.lua", "tuning.lua", "prefabs/wortox_nabbag.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# nabbag

## 快速判断

- 职责与适用：挂在可使用抓取动作的物品上（原版 [wortox_nabbag](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L203)）；`DoNabFromAct` 承接 NABBAG 动作（连抓范围内物品），`ReplicateNetFromAct` 在 NET（捕虫网）动作成功后复制捕网。
- 优先使用：动作层已接好——`ACTIONS.NABBAG.fn` 调 [DoNabFromAct](../../../raw/current/scripts/actions.lua#L4558-L4563)，NET 命中后调 [ReplicateNetFromAct](../../../raw/current/scripts/actions.lua#L1872-L1875)；物品只需挂组件。
- 重要副作用：临时替换 `inventory.HandleLeftoversFn` 做合堆，结束后恢复（[L58-L95](../../../raw/current/scripts/components/nabbag.lua#L58-L95)、[L130](../../../raw/current/scripts/components/nabbag.lua#L130)）；NET 复制会真实执行 `ACTIONS.NET.fn` 并消耗工具耐久（[L28-L34](../../../raw/current/scripts/components/nabbag.lua#L28-L34)）。
- 修改边界：锥形范围与上限全由 `TUNING.SKILLS.WORTOX.*` 决定（CONEANGLE=150°、MAX_RADIUS=3、CIRCLE_RADIUS=1.25、MAX_ITEMS_PER_NAB=400、MAX_USES_PER_NAB_PERCENT=0.25，[tuning](../../../raw/current/scripts/tuning.lua#L7436-L7440)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `DoNabFromAct(act)` | `action data -> (success, reason)` | 主目标不得含 INLIMBO/FX/_container/heavy/fire 标签（[L44-L51](../../../raw/current/scripts/components/nabbag.lua#L44-L51)）；主目标 PICKUP 成功后遍历锥内 `_inventoryitem` 实体逐个拾取，受 `NABBAG_MAX_ITEMS_PER_NAB-1` 限制。[实现](../../../raw/current/scripts/components/nabbag.lua#L45-L133) |
| `ReplicateNetFromAct(act)` | `action data -> void` | 防重入标记 `replicatingnet`；锥内每个可被 NET 作用的目标执行 `ACTIONS.NET.fn` + `OnUsedAsItem`，工具用尽即停。[实现](../../../raw/current/scripts/components/nabbag.lua#L13-L41) |
| `finiteuses` 协作 | 可选组件 | 每多抓一个消耗 1 点耐久，总消耗不超过 `total × NABBAG_MAX_USES_PER_NAB_PERCENT`；无 finiteuses 时不限次。[判定](../../../raw/current/scripts/components/nabbag.lua#L103-L121) |
| 合堆接管 | HandleLeftoversFn 临时替换 | 拾取过量时把可堆叠物塞进手上/同名堆，放不下的掉在原地，避免中途打断。[实现](../../../raw/current/scripts/components/nabbag.lua#L56-L95) |

## 典型接入

### 给自定义物品接抓取能力

- 前提与选择：主机端物品挂组件即可进入原版动作判定；需要耐久限制时配 finiteuses（组件语义见 [[reference/components/finiteuses]]）。

```lua
inst:AddComponent("nabbag")
local finiteuses = inst:AddComponent("finiteuses")
finiteuses:SetMaxUses(200)
finiteuses:SetUses(200)
finiteuses:SetOnFinished(function(inst_) inst_:Remove() end)
finiteuses:SetConsumption(ACTIONS.NET, 4) -- 仿 wortox_nabbag：NET 路径的耐久折算
```

- 调整与验证：动作可用性由 prefab 的 `nabbag` 组件存在性决定（actions.lua 检查 `act.invobject.components.nabbag`，[L4558](../../../raw/current/scripts/actions.lua#L4558)）；范围参数改 TUNING 表即可全局生效。

## 常见陷阱与限制

- **上限偏移**：`max_items_per_nab = NABBAG_MAX_ITEMS_PER_NAB - 1` 已扣除主目标，且递减判断是 `> 1`（[L106](../../../raw/current/scripts/components/nabbag.lua#L106)、[L122-L126](../../../raw/current/scripts/components/nabbag.lua#L122-L126)），实际额外抓取数比名义上限少 2。
- **主目标标签会被二次过滤**：DoNabFromAct 先按自身 CANTTAGS（含 fire）拦截主目标，与 NET 路径的标签集（含 smolder）不同，两个动作不共用黑名单。
- **合堆接管是全局替换**：期间 doer 的 `HandleLeftoversFn` 被换成组件闭包，异常中断会破坏恢复；不要在 PICKUP 链中再改该函数。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、actions.lua 两个动作接入点、wortox_nabbag 挂载、tuning.lua 数值。
- 游戏运行：未运行。
