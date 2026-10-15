---
title: "damagetypebonus"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "按目标标签和动态回调计算伤害倍率；武器／攻击者贡献可叠加，来源键控制替换与撤销。"
aliases: ["武器对暗影阵营额外伤害", "按目标标签增伤", "damagetypebonus"]
tags: [component, equipment, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/damagetypebonus.lua", "util/sourcemodifierlist.lua", "components/weapon.lua", "components/combat.lua", "components/explosive.lua", "prefabs/glasscutter.lua", "prefabs/sword_lunarplant.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# damagetypebonus

## 快速判断

- 职责与适用：按受击目标 HasTag 匹配数值来源，并乘动态回调，返回倍率；不直接扣血或给目标加标签。[计算](../../../raw/current/scripts/components/damagetypebonus.lua#L42-L57)
- 优先使用：固定类别 AddBonus(tag,source,mult,key)，撤销用同 tag／source／key；确需动态条件才加回调。基础武器伤害仍归 [[reference/components/weapon]]。[来源入口](../../../raw/current/scripts/components/damagetypebonus.lua#L10-L27)
- 重要副作用：同标签的多来源／多键相乘，多条匹配标签也相乘；原 combat 还叠加武器和攻击者各自贡献。[标签](../../../raw/current/scripts/components/damagetypebonus.lua#L42-L53) [武器与人物](../../../raw/current/scripts/components/combat.lua#L889-L899)
- 修改边界：服务端；只在实际调用 GetBonus 的伤害路径生效，不保证直接 health:DoDelta 或所有自定义伤害都经过它。[武器消费者](../../../raw/current/scripts/components/weapon.lua#L94-L102) [爆炸消费者](../../../raw/current/scripts/components/explosive.lua#L66)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `AddBonus(tag,src,pct,key?)` | pct 是相乘倍率，1.25 表示该贡献乘 1.25，不是加 1.25 点。每个 tag 使用默认基值 1 的 SourceModifierList。[实现](../../../raw/current/scripts/components/damagetypebonus.lua#L10-L17) [默认合并](../../../raw/current/scripts/util/sourcemodifierlist.lua#L8-L26) |
| `RemoveBonus(tag,src,key?)` | 指定 key 只撤该键；省略 key 撤该来源的所有键，不能随手清掉原有贡献。[组件](../../../raw/current/scripts/components/damagetypebonus.lua#L19-L27) [来源清理](../../../raw/current/scripts/util/sourcemodifierlist.lua#L103-L121) |
| `AddBonusCallback(fn)`／`RemoveBonusCallback(fn)` | fn(item,target) 返回倍率，nil 按 1；添加同一函数对象前先移除，不会重复登记该对象。不同闭包仍是不同回调。[管理](../../../raw/current/scripts/components/damagetypebonus.lua#L29-L39) [调用](../../../raw/current/scripts/components/damagetypebonus.lua#L50-L53) |
| `GetBonus(target)` | target=nil 返回 1 且不调回调；非空时匹配所有标签，随后乘所有回调结果，不内置限幅。[实现](../../../raw/current/scripts/components/damagetypebonus.lua#L42-L57) |

### 来源与消费边界

source=nil 不设置；同 source／key 覆盖而非累加；省略 key 使用 "key"；倍率 nil 或默认 1 会移除该键。实体 source 删除时自动清贡献，字符串来源需要调用者主动撤销。组件本身无保存钩子，静态配置在宿主构造时重施。[设置协议](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64-L98) [删除协议](../../../raw/current/scripts/util/sourcemodifierlist.lua#L103-L121)

weapon:GetDamage 把武器贡献同时用于普通／特殊伤害；combat 的攻击者贡献在对应计算中也进入普通和特殊伤害。不能只凭名字称其“仅普通伤害增幅”。原爆炸还把源实体倍率用于工作量；修改爆炸源时需一并考虑。[武器](../../../raw/current/scripts/components/weapon.lua#L94-L102) [人物计算](../../../raw/current/scripts/components/combat.lua#L938-L963) [爆炸工作量](../../../raw/current/scripts/components/explosive.lua#L66-L83)

## 典型接入

### 原玻璃刀对 shadow_aligned 目标额外乘 1.25，保留原暗影加成

目标模组 `modmain.lua`；复用 glasscutter 资源和原 damagetypebonus，不改耐久／攻击回调。原玻璃刀匹配 shadowsubmissive；shadow_aligned 是另一标签，原月亮植物剑也使用它，不能视为同义词。[原刀](../../../raw/current/scripts/prefabs/glasscutter.lua#L64-L76) [原阵营标签调用](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L262)

```lua
AddPrefabPostInit("glasscutter", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local bonus = inst.components.damagetypebonus
    if bonus ~= nil then
        bonus:AddBonus("shadow_aligned", inst, 1.25, "my_mod_shadow")
    end
end)
```

目标兼有两种标签时，新旧贡献相乘；只匹配 shadowsubmissive 的目标不因本例改变。最终生命损失还受 combat、护甲和受伤方机制影响；不保证“所有暗影生物都多掉 25% 生命”。

要在当前实例撤销，仅用 RemoveBonus("shadow_aligned",inst,"my_mod_shadow")；移除物品会清实体来源，停用并完整重载后不再重施。此处不新增攻击路径或全局人物倍率，实际效果仅沿原武器消费链。

## 常见陷阱与限制

- **以攻击者标签作增伤门禁**：GetBonus 查 target 的标签；根据攻击来源减伤应看 [[reference/components/damagetyperesist]]。[查询对象](../../../raw/current/scripts/components/damagetypebonus.lua#L42-L53)
- **多次 AddBonus 会不断叠值**：同来源同键是替换，不同键／来源才乘算；用自定义键避免改掉原默认键。[设置](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64-L98)
- **回调 nil 表示拒绝攻击**：这里只按 1 参与倍率；回调不得用来代替攻击资格／返回值协议。[调用](../../../raw/current/scripts/components/damagetypebonus.lua#L50-L53)
- **挂组件就会增强一切伤害**：必须有消费者调用；爆炸源还影响工作量，直接生命增量不由此组件接管。[爆炸](../../../raw/current/scripts/components/explosive.lua#L66-L83)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 4 处），未运行游戏。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖完整组件、来源合并／撤销、武器和 combat 所引分支、爆炸倍率调用、玻璃刀配置及原月亮植物剑标签调用。未审校全部自定义伤害、目标标签分布、骑乘分支或整套特殊伤害系统。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整玻璃刀例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；双标签叠加、无匹配目标、额外人物贡献、多人伤害和撤销待实测。
