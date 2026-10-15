---
title: "resistance"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "装备按攻击来源标签整次抵挡的门禁与通知；冷却、燃料和护盾效果由实际宿主负责。"
aliases: ["骨甲白天不触发抵挡", "装备整次挡伤", "resistance"]
tags: [component, equipment, callback, event, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/resistance.lua", "components/inventory.lua", "prefabs/armor_skeleton.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# resistance

## 快速判断

- 职责与适用：保存可抵挡的来源标签、是否允许抵挡回调及通知；实际库存命中后提前返回 0,nil，区别于 [[reference/components/damagetyperesist]] 的倍率。[组件](../../../raw/current/scripts/components/resistance.lua#L8-L46) [消费者](../../../raw/current/scripts/components/inventory.lua#L416-L420)
- 优先使用：改变资格包 shouldresistfn；既有骨甲的冷却／燃料／特效继续由宿主维护，不手动调用 ResistDamage 冒充一次完整挡伤。[门禁](../../../raw/current/scripts/components/resistance.lua#L34-L46) [骨甲协作](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L57-L92)
- 重要副作用：ResistDamage 调 onresistdamage(inst,damage_amount,attacker)，随后向装备发 damageresisted；该方法自身不扣血或清伤害。[通知](../../../raw/current/scripts/components/resistance.lua#L42-L47)
- 修改边界：服务端；已核验消费者是装备槽 ApplyDamage，单给任意生物挂组件不证明 combat 会替它抵挡。标签匹配还需要非空 attacker。[匹配](../../../raw/current/scripts/components/resistance.lua#L16-L24) [装备消费](../../../raw/current/scripts/components/inventory.lua#L408-L425)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `AddResistance(tag)`／`RemoveResistance(tag)` | 直接写 tags 集合，没有来源／key 或引用计数；不同效果共用标签时不能独立撤销。[实现](../../../raw/current/scripts/components/resistance.lua#L8-L14) |
| `HasResistance(attacker,weapon?)` | 有 attacker 时任一登记标签匹配 attacker 或 weapon 即 true；未匹配返回 nil。[实现](../../../raw/current/scripts/components/resistance.lua#L16-L24) |
| `HasResistanceToTag(tag)` | 只问标签是否登记，不等于当前攻击允许抵挡。[实现](../../../raw/current/scripts/components/resistance.lua#L26-L28) |
| `SetShouldResistFn(fn)`／`ShouldResistDamage()` | fn(item) 决定门禁；无回调默认 true，不接收伤害或攻击者参数。[实现](../../../raw/current/scripts/components/resistance.lua#L34-L40) |
| `SetOnResistDamageFn(fn)`／`ResistDamage(amount,attacker)` | 先效果回调，再装备事件；onresistdamage=nil 仍发事件，是否挡住由调用者控制。[实现](../../../raw/current/scripts/components/resistance.lua#L30-L46) |

库存匹配且门禁通过时直接返回 0,nil，跳过后续护甲和该次特殊伤害防御流程。不要把这个分支称为“仅挡普通伤害”，也不要推广为所有生命扣除免疫。[早退](../../../raw/current/scripts/components/inventory.lua#L408-L430) [后续特殊伤害](../../../raw/current/scripts/components/inventory.lua#L455-L487)

原骨甲先经冷却登记标签；抵挡消耗 fueled、启动下一轮冷却并短暂保留护盾，护盾结束移除标签；穿卸／模特展示另处理清理。组件本身不自带这些机制。[标签与冷却](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L49-L92) [穿卸](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L133-L178)

## 典型接入

### 骨甲仅黄昏／夜晚允许原抵挡，白天不触发

目标模组 `modmain.lua`，原注册名是 armorskeleton，文件名 armor_skeleton.lua 不等于注册名。复用资源与已有 resistance，不改冷却／燃料／标签集合；原资格还检查已装备且无 forcefield 装备。[注册](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L261) [资格](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L74-L82) [安装](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L229-L231)

```lua
AddPrefabPostInit("armorskeleton", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local resistance = inst.components.resistance
    if resistance == nil then return end
    local old = resistance.shouldresistfn
    resistance:SetShouldResistFn(function(item)
        return not GLOBAL.TheWorld.state.isday
            and (old == nil or old(item))
    end)
end)
```

门禁每次受击被查询，白天返回 false 时不进入该件骨甲的抵挡回调，也不由本组件消耗抵挡燃料；其他装备／伤害机制仍可生效。黄昏／夜晚只是额外允许，仍须原标签／充能／燃料链准备好，不能称夜间无条件无敌。[实际消费顺序](../../../raw/current/scripts/components/inventory.lua#L416-L425)

本例不暂停白天冷却，也不主动结束已有护盾特效；日夜切换无需新增轮询。移除本次门禁包装或停用并完整重载后恢复原资格，未实现热卸载。不要改包 onresistdamage 来代替资格：原宿主会临时置 nil 并在护盾／穿卸／充能时重设。[动态重设](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L49-L92)

## 常见陷阱与限制

- **ResistDamage 自己把伤害变零**：它只回调／发事件；实际清零是库存 return，直接调用可能只耗燃料／出特效。[方法](../../../raw/current/scripts/components/resistance.lua#L42-L46) [调用](../../../raw/current/scripts/components/inventory.lua#L416-L420)
- **AddResistance("_combat") 永久加成与原骨甲独立**：没有来源键，宿主冷却结束／卸装会增删同一集合；应保留宿主协调，不与其抢标签。[集合](../../../raw/current/scripts/components/resistance.lua#L8-L14) [清理](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L149-L178)
- **回调可按 attacker 决定是否挡**：shouldresistfn 只收到 item；新增按攻击者动态策略不能捏造签名，应另核验合适入口。[签名](../../../raw/current/scripts/components/resistance.lua#L38-L40)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖完整组件、库存早退与后续防御、骨甲标签／燃料／冷却／门禁、穿卸与展示清理、实际注册名。未全面审校其他抵挡装备、力场来源、所有日夜／区域环境或绕过库存的伤害。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整日夜资格例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；昼夜切换、护盾窗口、燃料耗尽、充能后受击、特殊伤害与其他力场协作待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。