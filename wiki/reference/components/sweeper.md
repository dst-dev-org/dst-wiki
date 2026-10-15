---
title: "sweeper"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "清扫工具的空标记组件；右键候选与动作按钮据此把持有人识别为扫帚，目标端门禁与结算见 sweepable。"
aliases: ["扫帚工具标记", "自制扫帚清扫", "sweeper"]
tags: [component, item, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/sweeper.lua", "componentactions.lua", "components/playercontroller.lua", "components/playeractionpicker.lua", "components/rider_replica.lua", "entityscript.lua", "actions.lua", "prefabs/reskin_tool.lua", "prefabs/witchbroom.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# sweeper

## 快速判断

- 职责与适用：工具端空标记，构造只保存 inst，没有方法、字段、事件或耐久接口；持有它的实体在清扫动作里作为 invobject。可清扫目标端见 [[reference/components/sweepable]]。[完整定义](../../../raw/current/scripts/components/sweeper.lua#L1-L5)
- 优先使用：模组扫帚在服务端构造段 AddComponent("sweeper")，配 equippable 供手持；耐久消耗走 finiteuses:SetConsumption(ACTIONS.SWEEP, 次数)，消费协议复用 [[reference/components/finiteuses]]，不向本组件填造字段。[原版消耗配置](../../../raw/current/scripts/prefabs/witchbroom.lua#L180) [原版挂载](../../../raw/current/scripts/prefabs/witchbroom.lua#L188)
- 重要副作用：AddComponent 自动把组件登记为动作组件并经 actioncomponents 网络变量同步到客户端；USEITEM／EQUIPPED 两候选段随后在右键且目标带 sweepable 标签时插入 ACTIONS.SWEEP。[注册](../../../raw/current/scripts/entityscript.lua#L665) [USEITEM 候选](../../../raw/current/scripts/componentactions.lua#L1824-L1828) [EQUIPPED 候选](../../../raw/current/scripts/componentactions.lua#L2494-L2498)
- 修改边界：标记不含目标结算、玩家动画或消耗协议；骑乘不可用与不可长按重复来自 ACTIONS.SWEEP 定义，不在组件上配置。[动作定义](../../../raw/current/scripts/actions.lua#L745-L746)

## 关键接口

| 所在层 | 职责与证据 |
|---|---|
| `components.sweeper` | 只有 inst；完整文件没有公开方法或存读钩子。[定义](../../../raw/current/scripts/components/sweeper.lua#L1-L5) |
| USEITEM 候选 | 光标持物右键目标时经 GetUseItemActions 触发；`right and target:HasTag("sweepable")` 才插入 SWEEP。[触发](../../../raw/current/scripts/components/playeractionpicker.lua#L142-L148) [handler](../../../raw/current/scripts/componentactions.lua#L1824-L1828) |
| EQUIPPED 候选 | 手持装备右键目标时经 GetEquippedItemActions 触发，条件与 USEITEM 相同；两段 handler 逻辑一致。[触发](../../../raw/current/scripts/components/playeractionpicker.lua#L249-L255) [handler](../../../raw/current/scripts/componentactions.lua#L2494-L2498) |
| 动作按钮前进动作 | GetActionButtonAction 用含 "sweepable" 的 pickup_tags 白名单搜索附近目标，再经 GetPickupAction 在工具 HasActionComponent("sweeper") 且目标带标签时返回 SWEEP。[分支](../../../raw/current/scripts/components/playercontroller.lua#L2063-L2064) [白名单](../../../raw/current/scripts/components/playercontroller.lua#L2249) [搜索与调用](../../../raw/current/scripts/components/playercontroller.lua#L2264-L2277) |
| 动作定义 | SWEEP：distance 2.5、mount_valid=false、invalid_hold_action=true；fn 先揭示拟态工具，再调目标 sweepable:Sweep。[定义](../../../raw/current/scripts/actions.lua#L745-L746) [fn](../../../raw/current/scripts/actions.lua#L7126-L7133) |

## 典型接入

### 模组自制扫帚（手持清扫 sweepable 目标）

前提与选择：目标为自定义工具 Prefab 的服务端构造段（mastersim 分支内），无额外资源。sweeper 只负责让候选与动作按钮识别工具；目标的标签门禁与清扫结算由 [[reference/components/sweepable]] 承担，两侧都装才能成链。

```lua
local function master_postinit(inst)
    inst:AddComponent("inventoryitem")
    inst:AddComponent("equippable")
    inst:AddComponent("sweeper")
    inst:AddComponent("finiteuses")
    inst.components.finiteuses:SetMaxUses(50)
    inst.components.finiteuses:SetConsumption(ACTIONS.SWEEP, 1)
end
```

生效与撤销：AddComponent("sweeper") 在服务端登记动作组件并同步客户端，右键与动作按钮两条入口随即可用；仅在该 Prefab 构造内生效，移除组件即退回普通工具。耐久为可选配置，次数单位为次，消耗按 finiteuses 协议在动作成功后结算。

原版参照：witchbroom 常驻挂载 sweeper 并配 SWEEP 每次消耗 1；reskin_tool 仅在万圣夜活动期（IsSpecialEventActive(SPECIAL_EVENTS.HALLOWED_NIGHTS)）挂载，活动外没有此标记。[witchbroom](../../../raw/current/scripts/prefabs/witchbroom.lua#L188) [reskin_tool](../../../raw/current/scripts/prefabs/reskin_tool.lua#L403-L405)

## 常见陷阱与限制

- **只有右键和动作按钮两条入口**：两段候选 handler 都要求 right 为真，左键点击目标不产生 SWEEP；动作按钮路径还要求目标先进入 pickup_tags 白名单的 FindEntities 搜索半径（直接行走 3、否则 6）。[handler 条件](../../../raw/current/scripts/componentactions.lua#L1824-L1828) [搜索半径](../../../raw/current/scripts/components/playercontroller.lua#L2264-L2277)
- **骑乘与长按限制改不到组件上**：mount_valid=false 使骑乘动作过滤器直接排除 SWEEP，invalid_hold_action=true 使动作不进入长按重复缓存；两者都是动作定义属性，不是标记属性。[骑乘过滤](../../../raw/current/scripts/components/rider_replica.lua#L135-L137) [长按判定](../../../raw/current/scripts/components/playercontroller.lua#L4626-L4630)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖完整标记文件、USEITEM／EQUIPPED 两候选 handler 及触发入口、GetPickupAction 分支与动作按钮白名单路径、ACTIONS.SWEEP 定义与 fn、骑乘过滤与长按判定，以及 witchbroom／reskin_tool 两个原版挂载点；全库检索确认 "sweeper" 组件无其他消费者。未覆盖：玩家状态图执行与动作预测细节。
- 结构检查：格式、来源、链接／行号、大小档位通过 validate_page 与 validate_source_links 校验；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行；右键与动作按钮清扫、骑乘拦截、耐久消耗、活动期限定挂载待实测。

## 相关链接

- [[reference/components/sweepable]]：可清扫目标端组件与清扫结算。
