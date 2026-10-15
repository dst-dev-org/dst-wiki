---
title: "damagereflect"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "配置宿主或已装备物品的反射伤害，理解 combat 的预计算、攻击后结算与反伤事件。"
aliases: ["damagereflect", "伤害反射", "装备反伤", "onreflectdamage"]
tags: [component, callback, equipment, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/damagereflect.lua", "components/combat.lua", "prefabs/armor_lunarplant.lua", "prefabs/hutch.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# damagereflect

## 快速判断

- 职责与适用：给 combat 的反伤计算提供普通／特殊伤害；组件自身不监听 attacked，也不自动伤害攻击者。默认普通反伤为固定 10。[damagereflect.lua:3](../../../raw/current/scripts/components/damagereflect.lua#L3)、[combat.lua:966](../../../raw/current/scripts/components/combat.lua#L966
- 优先使用：固定普通反伤用 SetDefaultDamage，条件反伤或位面反伤用 SetReflectDamageFn；原版亮茄盔甲返回 0 普通伤害和 planar 表。[armor_lunarplant.lua:154](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L154
- 重要副作用：反伤在目标护甲损坏之前预计算，攻击完成后返还攻击者；反伤量不等于目标实际损失生命，也不要求目标受击成功。[combat.lua:1216](../../../raw/current/scripts/components/combat.lua#L1216)、[:1235](../../../raw/current/scripts/components/combat.lua#L1235)
- 修改边界：服务端；正常路径由攻击者 DoAttack 驱动。仅直接调用目标 GetAttacked 不会调起这一预计算／返还链，组件无自己的保存与计时。[combat.lua:570](../../../raw/current/scripts/components/combat.lua#L570)、[damagereflect.lua:18](../../../raw/current/scripts/components/damagereflect.lua#L18

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `SetDefaultDamage(value)` | 普通伤害数值；无范围校验 | 初值 10；不是比例或减伤值。不设置回调时直接返回该值。[damagereflect.lua:7](../../../raw/current/scripts/components/damagereflect.lua#L7)、[:14](../../../raw/current/scripts/components/damagereflect.lua#L14) |
| `SetReflectDamageFn(fn)` | `(inst, attacker, damage, weapon, stimuli, spdamage) -> normal, special` | inst 为带组件的实体，装备时是物品；damage／spdamage 为攻击侧计算量，尚未经过目标防御。单槽覆盖，nil 恢复默认路径。[damagereflect.lua:10](../../../raw/current/scripts/components/damagereflect.lua#L10)、[combat.lua:975](../../../raw/current/scripts/components/combat.lua#L975 |
| `GetReflectedDamage(attacker, damage, weapon, stimuli, spdamage)` | 返回普通数值，以及可选特殊伤害表 | 回调首返回 nil／false 会退回 defaultdamage，0 在 Lua 为真可明确返回 0；无回调时仅返回默认普通值。建议普通量非负；消费者直接比较 `> 0`，不做类型校验。[damagereflect.lua:18](../../../raw/current/scripts/components/damagereflect.lua#L18)、[combat.lua:976](../../../raw/current/scripts/components/combat.lua#L976 |
| `onreflectdamage` | 事件 data：`inst, attacker, reflected_dmg, reflected_spdmg` | inst 是贡献反伤的宿主／装备，attacker 是收到反伤的攻击者；只有实际进入返还分支、贡献实体仍有效才发送，并不证明最终掉血。[combat.lua:979](../../../raw/current/scripts/components/combat.lua#L979)、[:1238](../../../raw/current/scripts/components/combat.lua#L1238) |

### 反伤计算与结算顺序

1. 攻击者 combat 的 `ignoredamagereflect` 为真时，CalcReflectedDamage 直接返回 0、nil。一般目标聚合自身组件及所有 EQUIPSLOTS 中已装备物品的组件；骑乘目标则改聚合坐骑和鞍，不走骑手自身／骑手装备的这条普通聚合。[combat.lua:966](../../../raw/current/scripts/components/combat.lua#L966)、[:1000](../../../raw/current/scripts/components/combat.lua#L1000)
2. 每个来源使用同一份传入伤害作计算，普通量相加，特殊表经 MergeSpDamage 合并；不是逐来源扣减原攻击伤害。单来源普通量大于 0 或特殊表非 nil 才记入贡献列表。[combat.lua:974](../../../raw/current/scripts/components/combat.lua#L974
3. DoAttack 在 CalcDamage 和 instancemult 后、目标 GetAttacked_Internal 前计算反伤，仅此处 projectile 为 nil 时进入；目标无 combat 的对应分支还可能以 0 输入计算反伤。随后目标受击、武器 OnAttack 与后续范围攻击完成，才结算反伤。[combat.lua:1216](../../../raw/current/scripts/components/combat.lua#L1216)、[:1218](../../../raw/current/scripts/components/combat.lua#L1218)
4. 合计普通量大于 0 或特殊表非 nil，且攻击者有 health 且尚存活，才对攻击者调用 `self:GetAttacked_Internal(targ, reflected_dmg, nil, nil, reflected_spdmg, true)`，再向有效贡献实体发事件。此受击仍能走攻击者防御／无敌等限制；未再调用 DoAttack，因此这条链不会自动递归反射。[combat.lua:1234](../../../raw/current/scripts/components/combat.lua#L1234)、[:1235](../../../raw/current/scripts/components/combat.lua#L1235)

（26148036 核验：反伤计算与返还现走 `GetAttacked_Internal(..., from_doattack=true)`，外部 `GetAttacked` 已不返回结果；亮茄甲的藤蔓／husk 反伤火花 proc 另被 `ShouldProcOnAttackedOrBlocked` 门控（重定向与"自己装备打自己"来源不再触发）——[combat.lua:1218](../../../raw/current/scripts/components/combat.lua#L1218)、[:1235](../../../raw/current/scripts/components/combat.lua#L1235)、[armor_lunarplant.lua:27](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L27）

## 典型接入

### 把原版亮茄盔甲改为固定普通反伤 20

完整目标文件为已有模组的 `modmain.lua`，由模组入口加载；仅修改 `armor_lunarplant` 服务端实例。原工厂经过 commonfn 添加组件及原回调，再注册此 Prefab；沿用装备、修理和反伤火花资源，无新增资源。[armor_lunarplant.lua:199](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L199)、[:214](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L214)、[:245](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L245)、[:436](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L436)

```lua
AddPrefabPostInit("armor_lunarplant", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.damagereflect ~= nil then
        inst.components.damagereflect:SetReflectDamageFn(function()
            return 20
        end)
    end
end)
```

20 是每次被纳入反伤聚合时提供的普通伤害点数，替换原来的位面反伤与暗影目标加成，不是最终生命损失。只改 SetDefaultDamage 不会覆盖原版返回 0 的回调。本例在物品被装备、且攻击走上述 DoAttack 链时贡献反伤；卸下即退出已装备物品扫描，破损装备流程由原版处理。组件参数不存档，启用模组时重建实例重新配置；停用后重建实例恢复原版。运行期若需要撤销，安装回调前应保存原 `reflectdamagefn` 并用 SetReflectDamageFn 恢复，不能以 nil 恢复原版位面回调。示例未进游戏运行。

另一个原版模式为 `hutch`：CheckBattery 只在 FUGU 形态且找到 pointy 物品时加组件／默认伤害／事件监听；失去条件时成对移除。反伤事件还会消耗容器内相应物品的 finiteuses，组件自身没有此扣次行为。[hutch.lua:138](../../../raw/current/scripts/prefabs/hutch.lua#L138)、[:191](../../../raw/current/scripts/prefabs/hutch.lua#L191)、[:150](../../../raw/current/scripts/prefabs/hutch.lua#L150)

## 常见陷阱与限制

- **返回 nil 禁用反伤**：会退回默认伤害；需要零普通、无特殊伤害时返回 `0, nil`。若只调用 SetDefaultDamage(0)，还需检查已有回调是否仍返回伤害。[damagereflect.lua:18](../../../raw/current/scripts/components/damagereflect.lua#L18
- **从伤害百分比或掉血判断触发**：默认量为固定值；计算发生在防御之前，目标格挡、躲避或护甲损坏不撤销已计算结果。若攻击者在自身攻击后已死，则不返还也不发送贡献事件。[combat.lua:1215](../../../raw/current/scripts/components/combat.lua#L1215)、[:1238](../../../raw/current/scripts/components/combat.lua#L1238)
- **有组件便反射一切伤害／所有远程和范围攻击**：实际入口有 projectile 条件及发射提前返回；GetAttacked 本身不收集反伤。不要把本页近战路径推广到独立弹体／AOE。[combat.lua:1169](../../../raw/current/scripts/components/combat.lua#L1169)、[:1216](../../../raw/current/scripts/components/combat.lua#L1216)
- **调用 `combat:GetDamageReflect` 或 `damagereflect:OnAttacked`**：前者源码标注 unused，仍调用后者，但本快照组件并无 OnAttacked 方法。使用实际 DoAttack／CalcReflectedDamage 链；本页不推荐旧入口。[combat.lua:1295](../../../raw/current/scripts/components/combat.lua#L1295)、[damagereflect.lua:18](../../../raw/current/scripts/components/damagereflect.lua#L18
- 组件没有计时、OnSave／OnLoad、OnRemoveFromEntity 或免疫标签判断。动态启停及事件监听由宿主管理；反伤能否真正掉血取决于攻击者的受击链。[damagereflect.lua:3](../../../raw/current/scripts/components/damagereflect.lua#L3)、[hutch.lua:213](../../../raw/current/scripts/prefabs/hutch.lua#L213

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 17 处、结论修订 2 处（反伤走 GetAttacked_Internal，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码审阅：2026-10-21，Steam buildid 25643519；完整审阅 damagereflect，定向核验 combat 聚合／骑乘分支／DoAttack 顺序／GetAttacked 防御及旧入口、亮茄盔甲回调／装备破损／工厂注册、哈奇电池切换／反伤事件。基础护甲与生命链复用 [[reference/components/combat#伤害与事件顺序]]。
- 结构与语法：2026-10-21，本页 validate_page.py 指定页检查通过（未检查导航登记）；validate_source_links.py --strict 通过；完整 modmain 示例经现有 parser_probe.exe --syntax-only 检查通过，仅验证 Lua 5.1 语法。
- 游戏运行：未运行；普通／位面反伤、多个装备来源、骑乘、护甲当次损坏、攻击者死亡与事件时机待实测。未审阅所有弹体、独立 AOE、其他角色／宿主的反伤表现。

相关：[[reference/components/combat]]；[[reference/components/weapon]]；[[reference/components/armor]]。
