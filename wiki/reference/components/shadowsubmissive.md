---
title: "shadowsubmissive"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "暗影生物臣服机制：持有 shadowdominance 压制标签的玩家不被敌对，攻击臣服生物会暂时失去压制力。"
aliases: ["shadowsubmissive", "暗影臣服", "暗影压制目标", "shadowdominance互作"]
tags: [component, ai]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/shadowsubmissive.lua", "components/shadowdominance.lua", "prefabs/nightmarecreature.lua", "components/playercontroller.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shadowsubmissive

## 快速判断

- 职责与适用：装在暗影生物上；宿主带 `shadowsubmissive` 标签，攻击者是否被"臣服"由其 `shadowdominance` 标签决定。([shadowsubmissive.lua:27-36](../../../raw/current/scripts/components/shadowsubmissive.lua#L27-L36) ([shadowsubmissive.lua:43-45](../../../raw/current/scripts/components/shadowsubmissive.lua#L43-L45)
- 优先使用：AI 用 `ShouldSubmitToTarget` 判当前目标、`TargetHasDominance` 判压制来源（装备标签或 `inherentshadowdominance` 天生标签）。[nightmarecreature.lua:23](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L23 ([shadowsubmissive.lua:47-56](../../../raw/current/scripts/components/shadowsubmissive.lua#L47-L56)
- 重要副作用：宿主被攻击时移除攻击者的 `shadowdominance` 标签，`forgetattackertime`（默认 12 秒）后按装备／天生恢复；与 shadowdominance 组件双向协作。([shadowsubmissive.lua:16-25](../../../raw/current/scripts/components/shadowsubmissive.lua#L16-L25) ([shadowdominance.lua:3-7](../../../raw/current/scripts/components/shadowdominance.lua#L3-L7)
- 修改边界：构造建议 pristine 加标签（原版生物如此）；监听宿主 `attacked` 事件。([shadowsubmissive.lua:32-35](../../../raw/current/scripts/components/shadowsubmissive.lua#L32-L35) ([nightmarecreature.lua:223-224](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L223-L224)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `ShouldSubmitToTarget(target)` | 布尔 | 目标有效且带 `shadowdominance` 标签；脑里用于中断攻击。([shadowsubmissive.lua:43-45](../../../raw/current/scripts/components/shadowsubmissive.lua#L43-L45) [nightmarecreaturebrain.lua:15](../../../raw/current/scripts/brains/nightmarecreaturebrain.lua#L15 |
| `TargetHasDominance(target)` | 布尔 | 装备槽有带标签物品或目标带 `inherentshadowdominance`；用于重选目标。([shadowsubmissive.lua:47-56](../../../raw/current/scripts/components/shadowsubmissive.lua#L47-L56) [nightmarecreature.lua:41](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L41 |
| `forgetattackertime` | 数值秒，默认 12 | 攻击者标签移除后等待恢复的时长；公开字段可覆写。[shadowsubmissive.lua:30](../../../raw/current/scripts/components/shadowsubmissive.lua#L30 [shadowsubmissive.lua:23](../../../raw/current/scripts/components/shadowsubmissive.lua#L23 |
| `OnReactivate`（内部） | DoTaskInTime 回调 | 背包装备仍带标签或天生标签时加回 `shadowdominance`。([shadowsubmissive.lua:3-14](../../../raw/current/scripts/components/shadowsubmissive.lua#L3-L14) |

## 典型接入

### 让自定义暗影生物被压制

- 前提与选择：宿主需 combat／attacked 事件链路；标签判定按玩家装备（shadowdominance 组件维护）或天生标签生效。[nightmarecreature.lua:86](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L86

```lua
inst:AddTag("shadowsubmissive") -- pristine 阶段
-- master postinit：
inst:AddComponent("shadowsubmissive")
```

- 生效与撤销：玩家控制器对带标签且 HostileToPlayerTest 通过的目标不再强制攻击；伤害加成系统也按 `shadowsubmissive` 注册。([playercontroller.lua:1759-1761](../../../raw/current/scripts/components/playercontroller.lua#L1759-L1761)

## 常见陷阱与限制

- **OnRemoveFromEntity 引用未定义全局 `inst`（应为 self.inst）**：调用 `RemoveComponent("shadowsubmissive")` 会报错；全库检索原版无此调用，模组使用时不要移除该组件。([shadowsubmissive.lua:38-41](../../../raw/current/scripts/components/shadowsubmissive.lua#L38-L41)
- 压制判定只看标签：外部直接给攻击者 AddTag(`shadowdominance`) 同样生效，但被攻击后会被本组件移除并按装备／天生规则恢复。([shadowsubmissive.lua:16-25](../../../raw/current/scripts/components/shadowsubmissive.lua#L16-L25)
- 挂起恢复任务期间（`_shadowdsubmissive_task` 非空）再次受击只重置计时，不再重复摘标签。([shadowsubmissive.lua:18-21](../../../raw/current/scripts/components/shadowsubmissive.lua#L18-L21)

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文、三类暗影生物安装与标签 pristine、脑与重选目标消费、玩家控制器分支。
- 游戏运行：未运行；未复现攻击后 12 秒恢复的时序。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。

相关：[[reference/components/shadowdominance]]
