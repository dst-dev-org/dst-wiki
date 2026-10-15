---
title: "efficientuser"
created: "2026-10-16"
updated: "2026-10-28"
type: component
summary: "按执行者和动作叠加使用倍率；减少工具或武器耐久消耗，治疗倍率则由治疗消费者解释。"
aliases: ["降低工具耐久消耗", "砍树耐久减半", "攻击耐久倍率", "使用消耗倍率", "efficientuser"]
tags: [component, equipment, actions, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/efficientuser.lua", "util/sourcemodifierlist.lua", "components/finiteuses.lua", "components/weapon.lua", "components/healer.lua", "bufferedaction.lua", "entityscript.lua", "prefabs/wes.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# efficientuser

## 快速判断

- 职责与适用：挂在**执行者**，按 Action 保存多来源倍率；组件不主动扣耐久、加治疗或执行工作，实际含义由消费者决定。[efficientuser.lua:3](../../../raw/current/scripts/components/efficientuser.lua#L3
- 优先使用：降低 CHOP 次数消耗用 `AddMultiplier(ACTIONS.CHOP, 0.5, source)`；人物工作量加成查 [[reference/components/workmultiplier]]，工具基础效率查 [[reference/components/tool]]。[finiteuses.lua:101](../../../raw/current/scripts/components/finiteuses.lua#L101
- 重要副作用：不同来源相乘；小于 1 对已核验耐久路径是省耐久，对 HEAL 则是降低基础治疗贡献。MINE 还写入 REMOVELUNARBUILDUP，且不传 `key`。[efficientuser.lua:18](../../../raw/current/scripts/components/efficientuser.lua#L18)、[healer.lua:36](../../../raw/current/scripts/components/healer.lua#L36
- 修改边界：在服务端配置，仅对实际读取该倍率的调用链生效；不是全物品、全动作或时间耐久的统一折扣。[weapon.lua:111](../../../raw/current/scripts/components/weapon.lua#L111

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `GetMultiplier(action)` | Action → number；未设置为 1 | 不会根据动作名称自动换算含义。[efficientuser.lua:14](../../../raw/current/scripts/components/efficientuser.lua#L14 |
| `AddMultiplier(action, multiplier, source, key?)` | 数值倍率；来源为实体或名称 | 每动作一个 SourceModifierList；不同来源／key 相乘，同来源同 key 覆盖；无 key 用默认槽。source=nil 不添加，倍率为 nil 或 1 时移除该槽。[efficientuser.lua:18](../../../raw/current/scripts/components/efficientuser.lua#L18)、[sourcemodifierlist.lua:64](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64 |
| `RemoveMultiplier(action, source, key?)` | 无返回值 | 给 key 删除单槽；不传 key 删除该动作的整个来源。MINE 有额外映射，见限制。[efficientuser.lua:30](../../../raw/current/scripts/components/efficientuser.lua#L30)、[sourcemodifierlist.lua:103](../../../raw/current/scripts/util/sourcemodifierlist.lua#L103 |
| `OnRemoveFromEntity()` | 宿主移除组件时调用 | Reset 各列表，解绑来源 onremove；无需手动调用钩子。[efficientuser.lua:8](../../../raw/current/scripts/components/efficientuser.lua#L8)、[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670 |

### 倍率实际作用在哪里

| 消费者 | 结算位置与含义 | 不覆盖的边界 |
|---|---|---|
| `finiteuses:OnUsedAsItem` | 有该 action 的 consumption 才先乘执行者倍率，再调物品 modifyuseconsumption，最后 Use；BufferedAction 成功后通知仍有效的 invobject。[finiteuses.lua:101](../../../raw/current/scripts/components/finiteuses.lua#L101)、[bufferedaction.lua:22](../../../raw/current/scripts/bufferedaction.lua#L22)、[entityscript.lua:1811](../../../raw/current/scripts/entityscript.lua#L1811 | 物品修正回调可再次改值；直接 `finiteuses:Use(n)` 不主动读取 efficientuser |
| `weapon:OnAttack` | 在普通攻击耐久路径中，以 attackwear × 武器来源倍率 × 执行者 ATTACK 倍率计算后 Use。[weapon.lua:105](../../../raw/current/scripts/components/weapon.lua#L105 | 忽略战斗耐久或弹射投射物等分支跳过；不会调用 finiteuses 的动作消耗修正回调 |
| `healer:Heal` | 基础 health × **doer** 的 HEAL 倍率，再由目标 health 处理；治疗物品仍按原流程消耗。[healer.lua:21](../../../raw/current/scripts/components/healer.lua#L21 | 不是给被治疗者统一增益，也不代表按比例节约一份药品 |

只列本页核验的消费者，不推断其他 Action 或模组消费者。工作量与动画频率不由这个组件修改。

## 典型接入

### 所有玩家砍树耐久消耗减半，保留原版工作量

目标为已启用模组的 `modmain.lua`；以下可独立追加，无需 GLOBAL 代理，复用玩家和斧头资源，不新增 Prefab。`AddPlayerPostInit` 筛选 player，原版 Wes 也分别设置工作量与使用消耗倍率。[modutil.lua:586](../../../raw/current/scripts/modutil.lua#L586)、[wes.lua:63](../../../raw/current/scripts/prefabs/wes.lua#L63

```lua
local source = "my_mod_chop_durability"

AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.efficientuser == nil then
        inst:AddComponent("efficientuser")
    end
    inst.components.efficientuser:AddMultiplier(GLOBAL.ACTIONS.CHOP, 0.5, source)
end)
```

`0.5` 是耐久消费乘数：基础消费 1 且无其他来源／回调改值时，每次该路径扣 0.5。覆盖所有玩家（不只发起操作的人），不改 CHOP 工作量、攻击或 fueled 时间消耗；读档后由实例初始化重新设置。字符串来源用模组独有名称；本效果在整个实例寿命内有效。若另做临时开关，失效时在同一执行者调用 `RemoveMultiplier(GLOBAL.ACTIONS.CHOP, "my_mod_chop_durability")`，保留其他来源，不删除共享组件。停用模组后不再初始化此项，已消耗的物品次数不会返还。

## 常见陷阱与限制

- **“倍率更大就是更省”**：耐久消费者直接乘消费量；2 表示双倍扣除，0 才是该路径零消费。组件不夹取负数，负数传入 Use 会增加次数，不作为省耐久方案。[finiteuses.lua:85](../../../raw/current/scripts/components/finiteuses.lua#L85)、[efficientuser.lua:23](../../../raw/current/scripts/components/efficientuser.lua#L23
- **MINE 的 key 不会对称传给副动作**：Add(MINE,…,source,key) 在主动作按 key 存储，映射副动作时却用默认槽；Remove(MINE,source,key) 会删除副动作的整个来源。多 key 的 MINE 可相互覆盖／误撤副动作，优先给独立效果使用独立 source，并核对其他模组。[efficientuser.lua:25](../../../raw/current/scripts/components/efficientuser.lua#L25)、[:35](../../../raw/current/scripts/components/efficientuser.lua#L35)
- **临时效果只靠自动清理**：实体来源只在其 onremove 时自动撤销，卸装／Buff 失效仍需显式 Remove；名称来源没有实体删除事件。同来源不带 key 再次 Add 是覆盖，Remove 不带 key 是整源撤销。[sourcemodifierlist.lua:84](../../../raw/current/scripts/util/sourcemodifierlist.lua#L84)、[:103](../../../raw/current/scripts/util/sourcemodifierlist.lua#L103)
- **倍率自动存档**：本组件无 OnSave／OnLoad；重新施加由角色、装备或效果所有者负责。不要从 finiteuses 保存当前次数推断执行者倍率也会保存。[efficientuser.lua:3](../../../raw/current/scripts/components/efficientuser.lua#L3)、[finiteuses.lua:48](../../../raw/current/scripts/components/finiteuses.lua#L48

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 1 处），未运行游戏。

- 源码审阅：2026-10-16，Steam buildid 25643519；覆盖组件全部倍率方法、来源列表与移除、上述三类消费者、Wes 分工及玩家初始化案例；未全量审校所有动作或其他模组消费者。
- 结构检查：本批元数据、来源、源码行锚点、大小、链接与唯一登记通过；发布代码通过 Full Moon 3.0.0／Lua 5.1 语法、原 token 字节与精确往返检查，章节读取完整。
- 游戏运行：未运行；多人工作、 小数耐久显示、读档重新施加和其他模组来源叠加待实测。

相关：[[reference/components/finiteuses#关键接口|动作次数消费]]；[[reference/components/weapon#关键接口|攻击与耐久]]；[[reference/components/workmultiplier#关键流程|完整工作量结算]]。
