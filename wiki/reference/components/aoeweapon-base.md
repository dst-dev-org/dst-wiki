---
title: "aoeweapon_base"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "范围武器共用的单目标命中分流与物品抛散；选择跳砸或突刺子类时配置破坏、采摘和战斗边界。"
aliases: ["aoeweapon_base", "AOE武器基类", "范围武器破坏树矿", "范围攻击采摘"]
tags: [component, actions, callback, equipment]
dst_build: "steam-buildid:26230496"
sources: [components/aoeweapon_base.lua, components/aoeweapon_leap.lua, components/aoeweapon_lunge.lua, components/combat.lua, components/weapon.lua, prefabs/spear_wathgrithr.lua]
reliability: verified
validation: source-reviewed
status: active
---

# aoeweapon_base

## 快速判断

- 职责与适用：供 [[reference/components/aoeweapon-leap]]／[[reference/components/aoeweapon-lunge]] 继承的公共逻辑；自身没有选区、位移或施法入口，不需要再给子类物品挂一份基类组件。[基类构造](../../../raw/current/scripts/components/aoeweapon_base.lua#L1)、[leap 继承](../../../raw/current/scripts/components/aoeweapon_leap.lua#L1)、[lunge 继承](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L1)
- 优先使用：在已接好技能的具体子类上设置筛选、伤害和命中回调；只攻击战斗目标的原版模式同时清空工作动作并 `SetTags("_combat")`。[闪电长矛配置](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L481-L490)
- 重要副作用：命中优先摧毁可工作对象，其次可选采摘，最后才走战斗；`onhitfn` 不能一概解释成“造成了伤害”。[OnHit](../../../raw/current/scripts/components/aoeweapon_base.lua#L86-L133)
- 修改边界：真实命中依赖服务端组件与 doer.combat；`SetDamage` 只存字段，具体子类才临时写到物品 weapon，战斗仍选 doer 当前手持武器。[SetDamage](../../../raw/current/scripts/components/aoeweapon_base.lua#L23)、[子类应用](../../../raw/current/scripts/components/aoeweapon_leap.lua#L42-L52)、[DoAttack](../../../raw/current/scripts/components/combat.lua#L1120-L1132)

## 关键接口

| 接口／字段 | 参数／默认 | 条件、作用与证据 |
|---|---|---|
| `SetDamage(dmg)`／`SetStimuli(stimuli)` | 默认 damage=10；stimuli 未赋值 | 保存技能伤害与攻击刺激；不是直接给目标扣血。默认与 setter 见[构造与配置](../../../raw/current/scripts/components/aoeweapon_base.lua#L1-L28)。 |
| `SetWorkActions(...)` | 默认 CHOP／HAMMER／MINE／DIG；传 ACTIONS 对象 | **替换**动作集合并重算 combinedtags；空参数清工作动作。不会清 tags，也不会禁止 NPC_workable 的无动作特殊分支。[配置](../../../raw/current/scripts/components/aoeweapon_base.lua#L31-L37)、[特殊分支](../../../raw/current/scripts/components/aoeweapon_base.lua#L91-L103) |
| `SetTags(...)` | 默认 `_combat`／`pickable`／`NPC_workable` | 替换 OR 候选标签，再合入动作的 `<id>_workable`；子类 FindEntities 用 combinedtags 作 one-of，不要求全部具备。[组合](../../../raw/current/scripts/components/aoeweapon_base.lua#L40-L68)、[消费者](../../../raw/current/scripts/components/aoeweapon_leap.lua#L55) |
| `SetNoTags(...)` | 默认 FX／DECOR／INLIMBO | 替换子类命中候选的排除集合；不控制独立的抛散查询。[配置](../../../raw/current/scripts/components/aoeweapon_base.lua#L70-L71)、[抛散筛选](../../../raw/current/scripts/components/aoeweapon_leap.lua#L28-L29) |
| `canpick` | 默认未赋值；直接设真开启 | 有 pickable、CanBePicked 且无 intense 时 `Pick(self.inst)`，不传玩家以避免自动入袋；排在工作分支之后。[采摘分支](../../../raw/current/scripts/components/aoeweapon_base.lua#L107-L118) |
| `SetOnPreHitFn`／`SetOnHitFn`／`SetOnMissFn` | 回调均 `(item, doer, target)` | pre 在分流前，不消费返回值；hit 为成功进入工作／采摘／攻击分支，miss 为这些分支都未进入。`OnHit` 本身没有布尔返回。[注册](../../../raw/current/scripts/components/aoeweapon_base.lua#L74-L84)、[调用](../../../raw/current/scripts/components/aoeweapon_base.lua#L86-L133) |
| `OnToss(doer,target,sourceposition,basespeed,startradius)` | sourceposition 可省；速度含随机量 | 先停用 mine，再对有物理且可反弹的 inventoryitem 尝试移至抛起点并设速度；不是丢弃背包接口。[OnToss](../../../raw/current/scripts/components/aoeweapon_base.lua#L141-L187) |

（26148036 核验：OnToss 的 mine 停用改经全局 `DeactivateInventoryItemBeforeLaunch(target)`——内部仍调 `mine:Deactivate`，mine 的包装照常命中，物品可另定义 `inventoryitem_DeactivateBeforeLaunch` 扩展点；`TryTeleportToLaunchPos` 失败时回退为直接 `Physics:Teleport` 并对 inventoryitem 调 `SetLanded(false, true)`——[OnToss](../../../raw/current/scripts/components/aoeweapon_base.lua#L141-L187)、[aoeweapon_base.lua:142](../../../raw/current/scripts/components/aoeweapon_base.lua#L142)、[:187](../../../raw/current/scripts/components/aoeweapon_base.lua#L187)）

### 命中分流与伤害边界

1. `onprehitfn` 先执行；可工作对象优先 `Destroy(doer)`，残存 stump 还会 Remove；DIG 不破坏带 spawner／childspawner 的对象。[工作条件](../../../raw/current/scripts/components/aoeweapon_base.lua#L86-L113)
2. 否则按 canpick 采摘；再否则仅在 CanTarget 且非 IsAlly 时 `doer.components.combat:DoAttack(target,nil,nil,stimuli)`。盟友检查只在战斗分支，不能据此承诺不拆友方建筑。[分流](../../../raw/current/scripts/components/aoeweapon_base.lua#L114-L122)
3. DoAttack 仍可因 CanHitTarget 失败而返回；基类却在调用后直接记 did_hit=true，因此 hit 回调不证明目标掉血、命中数不等于伤害次数。[基类判定](../../../raw/current/scripts/components/aoeweapon_base.lua#L119-L127)、[战斗拒绝](../../../raw/current/scripts/components/combat.lua#L1147-L1154)

## 典型接入

### 范围技能只打敌人，不顺带采摘或拆树矿

原版闪电长矛在服务端给 `aoeweapon_lunge` 配 `SetWorkActions()`、`SetTags("_combat")`，沿用默认未开启的 canpick；伤害、刺激、技能前后回调也在同一处设置。[完整配置入口](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L481-L490)

这是已有突刺技能的配置模式；完整 `modmain.lua` 调整例见 [[reference/components/aoeweapon-lunge#典型接入]]。普通近战、技能冷却、装备约束分别仍由 [[reference/components/weapon]]、[[reference/components/rechargeable]]、[[reference/components/aoespell]] 与宿主 SG 处理。

## 常见陷阱与限制

- **只把 SetWorkActions 清空**：默认 tags 仍含 NPC_workable，且无 action 的 NPC 工作对象不查动作表；需要限制候选，参照上文同时配置 SetTags。[候选与特殊分支](../../../raw/current/scripts/components/aoeweapon_base.lua#L17-L20)、[工作判断](../../../raw/current/scripts/components/aoeweapon_base.lua#L91-L103)
- **离手技能物品仍决定攻击伤害**：OnHit 传入 weapon=nil，Combat:GetWeapon 从 doer 的手部取物品；子类对自身 weapon 的临时修改不保证作用到另一件装备。维持原版装备／SG链再使用，不把此类当通用离体爆炸器。[实际传参](../../../raw/current/scripts/components/aoeweapon_base.lua#L119-L120)、[取武器](../../../raw/current/scripts/components/combat.lua#L853-L865)
- **自调 OnToss 到任意实体**：函数读取 `target.components.inventoryitem.nobounce`；原版由子类先用 `_inventoryitem` 且排除 locomotor／INLIMBO 筛选，不支持无 inventoryitem 对象的任意调用。[前置](../../../raw/current/scripts/components/aoeweapon_base.lua#L141-L142)、[原版筛选](../../../raw/current/scripts/components/aoeweapon_leap.lua#L28-L29)
- **命中筛选能阻止抛散**：子类另查 inventoryitem；SetTags／SetNoTags 只影响命中列表，不能拿它们关闭抛散。[两次查询](../../../raw/current/scripts/components/aoeweapon_leap.lua#L55)、[独立抛散](../../../raw/current/scripts/components/aoeweapon_leap.lua#L78-L87)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 4 处、结论修订 1 处（OnToss 前置与回退核验，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整基类、两个派生实现、Combat 选武器／攻击拒绝、Weapon 攻击磨损和闪电长矛实际配置。伤害公式及所有特殊目标不逐个审校。
- 结构检查：本页已执行页面格式／本地链接及严格源码引用检查；本页无独立完整 Lua 示例。
- 游戏运行：未运行；跨装备调用、工作／采摘与战斗交叠、抛散边缘物理效果未实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。