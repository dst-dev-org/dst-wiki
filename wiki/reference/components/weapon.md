---
title: "weapon"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "配置物品攻击伤害、攻击回调及远程入口，区分攻击伤害与工具工作。"
aliases: ["武器伤害", "攻击回调", "攻击扣耐久", "远程武器", "weapon"]
tags: ["component", "equipment", "callback", "mechanism"]
dst_build: "steam-buildid:26230496"
sources: ["components/combat.lua", "components/inventory.lua", "components/weapon.lua", "prefabs/spear.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# weapon

## 快速判断

- 职责与适用：武器侧伤害、攻击回调、弹体入口；攻击选择和伤害分发由 combat 等调用链负责。[components/combat.lua:889](../../../raw/current/scripts/components/combat.lua#L889
- 优先使用：SetDamage 配基础伤害，SetOnAttack 配攻击回调；工具效率另见 [[reference/components/tool]]。[components/weapon.lua:44](../../../raw/current/scripts/components/weapon.lua#L44
- 重要副作用：OnAttack 回调与物品 weapononattack 事件后，可能自动消耗 finiteuses；不要无条件重复扣次。[components/weapon.lua:105](../../../raw/current/scripts/components/weapon.lua#L105
- 修改边界：服务端改组件；GetDamage 不是最终掉血量，普通与位面等特殊伤害分开计算。[components/combat.lua:938](../../../raw/current/scripts/components/combat.lua#L938

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetDamage(dmg)` | number 或 `(inst, attacker, target)->number` | 默认 10；GetDamage 返回普通伤害及特殊伤害表。[components/weapon.lua:11](../../../raw/current/scripts/components/weapon.lua#L11 [components/weapon.lua:94](../../../raw/current/scripts/components/weapon.lua#L94 |
| `SetOnAttack(fn)` | `(inst, attacker, target)` | 不接 projectile 参数；回调发生于组件攻击入口，不承诺目标实际掉血。[components/weapon.lua:105](../../../raw/current/scripts/components/weapon.lua#L105 |
| `SetRange(attack, hit?)` | hit 省略按 attack | 武器攻击／命中范围；不等于所有 AI、动作、目标判定范围。[components/weapon.lua:48](../../../raw/current/scripts/components/weapon.lua#L48 |
| `SetProjectile(prefab)` | 弹体 Prefab 名称 | LaunchProjectile 按 projectile／complexprojectile 组件选择路径；不是给任意实体名字就能远程攻击。[components/weapon.lua:65](../../../raw/current/scripts/components/weapon.lua#L65 [components/weapon.lua:123](../../../raw/current/scripts/components/weapon.lua#L123 |

（26148036 核验：`weapononattack` 事件新增 `projectile` 字段，SetOnAttack 回调签名不变仍为 (inst, attacker, target)；LaunchProjectile 新增 `weapononprojectilelaunched` 事件，载荷为 attacker／target／projectile——均为可选新增，本页结论不变。）

## 典型接入

### 修改原版长矛基础攻击伤害

目标模组 `modmain.lua`，复用 spear 的资源、装备逻辑和有限次数。原版工厂配置见 [prefabs/spear.lua:55](../../../raw/current/scripts/prefabs/spear.lua#L55)，扣次链见 [components/weapon.lua:111](../../../raw/current/scripts/components/weapon.lua#L111)。

```lua
AddPrefabPostInit("spear", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.weapon ~= nil then
        inst.components.weapon:SetDamage(50)
    end
end)
```

50 是基础普通伤害；角色倍率、防御与其他修饰可改变最终值。保留攻击扣次；停用后新实例恢复原配置。没有新动画或弹体。

## 常见陷阱与限制

- **OnAttack 自己再 Use(1)**：默认武器入口已按 attackwear／修饰调用 finiteuses:Use，除非明确禁用该路径，否则重复扣次。[components/weapon.lua:111](../../../raw/current/scripts/components/weapon.lua#L111
- **直接 health:DoDelta 等于普通武器攻击**：绕开 combat／inventory 护甲和特殊伤害链；通过实际攻击入口选行为。[components/combat.lua:871](../../../raw/current/scripts/components/combat.lua#L871 [components/inventory.lua:408](../../../raw/current/scripts/components/inventory.lua#L408
- **远程只加 weapon 就完整**：还需弹体工厂、依赖资源与攻击动作／状态图；本页只给机制入口，不提供未核验的新远程工厂。[components/weapon.lua:123](../../../raw/current/scripts/components/weapon.lua#L123

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 3 处、结论修订 0 处（combat 锚点重定位与新增事件，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：GetDamage 普通／特殊伤害返回、OnAttack 回调／事件后扣次与长矛伤害示例，依据见上文对应锚点；未交付远程弹体工厂或核验全部角色／攻击状态。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/combat#伤害与事件顺序|战斗伤害链]]；[[reference/components/health|生命变化]]；[[reference/components/finiteuses]]；[[reference/components/planardamage]]；[[reference/components/armor]]。
