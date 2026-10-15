---
title: "shadowdominance"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "装备级暗影压制组件：装备时给持有者加 shadowdominance 标签，使暗影生物不再将其视为敌对目标。"
aliases: ["shadowdominance", "暗影压制", "骸骨帽压制", "shadowdominance标签"]
tags: [component, item]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/shadowdominance.lua", "prefabs/hats.lua", "prefabs/king_cane.lua", "prefabs/nightmarecreature.lua", "prefabs/woodie.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shadowdominance

## 快速判断

- 职责与适用：装在可装备物品上；自动维护持有者的 `shadowdominance` 标签（装备加、卸下/移除清），暗影生物按该标签放弃敌对。与 shadowsubmissive 组件协作。[shadowdominance.lua:1](../../../raw/current/scripts/components/shadowdominance.lua#L1 [nightmarecreature.lua:86](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L86
- 优先使用：物品 pristine 阶段加 `shadowdominance` 标签（原版骸骨帽如此），master 侧 AddComponent 即可。([hats.lua:2879-2880](../../../raw/current/scripts/prefabs/hats.lua#L2879-L2880) [hats.lua:2901](../../../raw/current/scripts/prefabs/hats.lua#L2901
- 重要副作用：攻击 shadowsubmissive 生物会让攻击者标签被移除并延迟恢复（对方组件的流程）；本组件在恢复任务挂起时不重复加标签。([shadowdominance.lua:3-7](../../../raw/current/scripts/components/shadowdominance.lua#L3-L7)
- 修改边界：卸下判断遍历 equipslots 排除自己（不能用 EquipHasTag——本物品此刻仍在槽内，源码注释说明）。([shadowdominance.lua:13-22](../../../raw/current/scripts/components/shadowdominance.lua#L13-L22)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造（无公开初始化方法） | Class(inst) | 加物品标签、监听 equipped／unequipped／onremove；添加时已装备则立即应用。([shadowdominance.lua:40-53](../../../raw/current/scripts/components/shadowdominance.lua#L40-L53) |
| `OnRemoveFromEntity` | 内部钩子 | 摘标签、移监听并按 OnRemove 复原持有者标签。([shadowdominance.lua:55-61](../../../raw/current/scripts/components/shadowdominance.lua#L55-L61) |
| `OnEquipped`（内部） | equipped 事件回调 | 持有者无挂起恢复任务时加 `shadowdominance` 标签。([shadowdominance.lua:3-7](../../../raw/current/scripts/components/shadowdominance.lua#L3-L7) |
| `OnRemove`（内部） | onremove 事件回调 | 物品在装备且被持有时按卸下处理；源码注释：OnRemoveEntity 时机太晚，故监听 onremove。([shadowdominance.lua:32-38](../../../raw/current/scripts/components/shadowdominance.lua#L32-L38) [shadowdominance.lua:48](../../../raw/current/scripts/components/shadowdominance.lua#L48 |

## 典型接入

### 装备使暗影生物不主动攻击

- 前提与选择：压制是标签判定——敌对测试在暗影生物侧读玩家标签；装备只需把标签维护对。([nightmarecreature.lua:86-88](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L86-L88)

```lua
-- pristine 阶段（SetPristine 之前）：
inst:AddTag("shadowdominance")

-- master postinit：
if not TheWorld.ismastersim then
    return
end
inst:AddComponent("shadowdominance")
```

- 生效与撤销：动态加移组件也可行（原版王杖堕落时加、复原时移除）。[king_cane.lua:196](../../../raw/current/scripts/prefabs/king_cane.lua#L196 [king_cane.lua:205](../../../raw/current/scripts/prefabs/king_cane.lua#L205

## 常见陷阱与限制

- 标签是判定核心，组件只是维护者：外部直接 AddTag 也生效（原版狼人形态对伍迪就这么做）；但不走组件时卸下清理需自理。([woodie.lua:1154-1168](../../../raw/current/scripts/prefabs/woodie.lua#L1154-L1168)
- 攻击一次臣服生物即失去压制约 12 秒，即使装备还穿着（标签由 shadowsubmissive 侧管理，见该页）。[shadowdominance.lua:4](../../../raw/current/scripts/components/shadowdominance.lua#L4
- 标签还被非生物系统消费：玩家状态图按装备上的该标签区分王杖堕落弹道音效，勿当作纯生物语义复用。([SGwilson.lua:58-63](../../../raw/current/scripts/stategraphs/SGwilson.lua#L58-L63)

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文、骸骨帽 pristine＋安装、王杖动态加移、暗影生物敌对测试与伍迪标签维护。
- 游戏运行：未运行；未复现攻击后压制失效的时序。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。

相关：[[reference/components/shadowsubmissive]] · [[reference/components/shadowlevel]]
