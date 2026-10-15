---
title: "electricattacks"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "管理攻击者电击来源，让 combat 在未指定刺激类型时选用电击，并区分湿度增伤、免疫与清理。"
aliases: ["electricattacks", "攻击带电", "电击来源", "电击湿度增伤"]
tags: [component, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/electricattacks.lua", "util/sourcemodifierlist.lua", "components/combat.lua", "components/weapon.lua", "componentutil.lua", "entityscript.lua", "tuning.lua", "prefabs/foodbuffs.lua", "prefabs/lightninggoat.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# electricattacks

## 快速判断

- 职责与适用：攻击者侧的布尔来源容器；不保存伤害数值。`combat:DoAttack` 在刺激类型仍为 nil 时，只看组件是否存在便选 `"electric"`。[electricattacks.lua:3](../../../raw/current/scripts/components/electricattacks.lua#L3)、[combat.lua:1133](../../../raw/current/scripts/components/combat.lua#L1133
- 优先使用：临时／多来源带电攻击成对调用 AddSource／RemoveSource；仅武器带电可用 `weapon:SetElectric`，不必给持有者加此组件。[weapon.lua:73](../../../raw/current/scripts/components/weapon.lua#L73
- 重要副作用：电击在实际 DoAttack 伤害分支参与普通伤害倍率；组件自身不产生火花、免疫、充能外观或计时。[combat.lua:1198](../../../raw/current/scripts/components/combat.lua#L1198)、[foodbuffs.lua:61](../../../raw/current/scripts/prefabs/foodbuffs.lua#L61
- 修改边界：服务端配置；来源列表没有复制／存档／自动到期接口。原版电羊在服务端充能路径添加来源，放电显式撤销，宿主另存 charged 状态后重建。[lightninggoat.lua:84](../../../raw/current/scripts/prefabs/lightninggoat.lua#L84)、[:109](../../../raw/current/scripts/prefabs/lightninggoat.lua#L109)、[:176](../../../raw/current/scripts/prefabs/lightninggoat.lua#L176)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `AddSource(source)` | 来源实体或名称；无返回约定 | 写 true；同一个 source 重复添加不会计数。包装层没有 key 参数，多个效果要用不同来源。[electricattacks.lua:8](../../../raw/current/scripts/components/electricattacks.lua#L8)、[sourcemodifierlist.lua:64](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64 |
| `RemoveSource(source)` | 与添加时相同的来源 | 删除来源后若聚合为 false，立即 RemoveComponent；其余来源仍在则保留。[electricattacks.lua:12](../../../raw/current/scripts/components/electricattacks.lua#L12 |
| `_sources` | 私有 SourceModifierList，初值 false，布尔 OR | 不是伤害倍率列表，也不应替代公开撤销方法。[electricattacks.lua:3](../../../raw/current/scripts/components/electricattacks.lua#L3)、[sourcemodifierlist.lua:33](../../../raw/current/scripts/util/sourcemodifierlist.lua#L33 |

### 刺激类型、湿度与伤害顺序

1. 未显式传入 stimuli 时，DoAttack 先问武器 overridestimulifn，再取武器的 electric 标记，最后才检查攻击者 electricattacks；已有非 nil 刺激不会被组件覆盖。[combat.lua:1133](../../../raw/current/scripts/components/combat.lua#L1133
2. 实际伤害分支只要最终 stimuli 或武器 stimuli 是 electric，且目标未满足 `IsEntityElectricImmune`，便以 `electric_damage_mult + electric_wet_damage_mult * target:GetWetMultiplier()` 作为 CalcDamage 的 multiplier。武器字段可覆盖默认值；默认分别为 1.5、1，即湿度系数 0／1 时为 1.5／2.5。[combat.lua:1198](../../../raw/current/scripts/components/combat.lua#L1198)、[tuning.lua:659-660](../../../raw/current/scripts/tuning.lua#L659-L660)
3. 湿度系数不是原始潮湿点数：wet 标签直接 1，其后 moistureimmunity 为 0，再考虑物品湿润、moisture 百分比或世界／游泳状态。[entityscript.lua:770](../../../raw/current/scripts/entityscript.lua#L770
4. 免疫判据为 electricdamageimmune 标签或 inventory:IsInsulated()；本分支免疫后保留倍率 1，不把整次普通攻击归零。倍率进入普通伤害计算，damagebonus 另加；特殊伤害没有统一乘此电击倍率。之后仍经过目标 GetAttacked／护甲和生命链。[componentutil.lua:22](../../../raw/current/scripts/componentutil.lua#L22)、[combat.lua:938](../../../raw/current/scripts/components/combat.lua#L938)、[:1218](../../../raw/current/scripts/components/combat.lua#L1218)

## 典型接入

### 让原版电羊持续具有额外的电击攻击来源

完整目标文件为已有模组的 `modmain.lua`，由模组入口加载；只改 `lightninggoat` 服务端实例，沿用原生战斗、状态图与资源，无新资源。原版充能用自身作 source，本例另用唯一字符串，避免放电移除原生来源时一并撤销本例。[lightninggoat.lua:109](../../../raw/current/scripts/prefabs/lightninggoat.lua#L109)、[:228](../../../raw/current/scripts/prefabs/lightninggoat.lua#L228)、[:306](../../../raw/current/scripts/prefabs/lightninggoat.lua#L306)

```lua
local SOURCE = "dstwiki_extra_goat_electric"

AddPrefabPostInit("lightninggoat", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.electricattacks == nil then
        inst:AddComponent("electricattacks")
    end
    inst.components.electricattacks:AddSource(SOURCE)
end)
```

生效为该实例的持续来源，无秒数或固定伤害配置；按上节倍率计算。本例不强制充能，也不赋予电击免疫／发光。运行期撤销应对同一实体调用 `inst.components.electricattacks:RemoveSource("dstwiki_extra_goat_electric")`（先判组件存在）；若仍有原生充能来源，组件保留。来源本身不存档，启用模组时新建／读档重建实例会重新添加；停用并重建实例后恢复原版配置。示例未进游戏运行。

原版临时方案：`buff_electricattack` 来自 MakeBuff("electricattack", ...)，attach 加 buff 实体来源，detach 显式 RemoveSource 并移除自己对目标 onattackother 的监听；火花和电荷特效由该宿主另外负责，不能由组件名推断。[foodbuffs.lua:61](../../../raw/current/scripts/prefabs/foodbuffs.lua#L61)、[:96](../../../raw/current/scripts/prefabs/foodbuffs.lua#L96)、[:223](../../../raw/current/scripts/prefabs/foodbuffs.lua#L223)

## 常见陷阱与限制

- **来源实体删除就完全撤销电击**：SourceModifierList 的 onremove 只删除列表项并重算，不调用 ElectricAttacks:RemoveSource；组件可能空着仍留在攻击者身上，而 combat 只检查存在性。应在效果卸载时显式 RemoveSource，再销毁来源；只加组件不加来源也会触发这项存在性检查。[sourcemodifierlist.lua:85](../../../raw/current/scripts/util/sourcemodifierlist.lua#L85)、[combat.lua:1142](../../../raw/current/scripts/components/combat.lua#L1142
- **直接 `GetAttacked(attacker, damage, nil, "electric")` 会自动乘湿度倍率**：该倍率计算在 DoAttack，不在 GetAttacked。电羊反击自行计算倍率后才送入受击入口；不要漏算或重复计算。[combat.lua:570](../../../raw/current/scripts/components/combat.lua#L570)、[lightninggoat.lua:137](../../../raw/current/scripts/prefabs/lightninggoat.lua#L137
- **组件保证所有远程攻击带电**：DoAttack 的投掷／弹体发射分支可提前返回，命中处理取决于具体弹体。本页只推荐已核验的电羊近战；远程与范围攻击的完整传播未覆盖。[combat.lua:1168](../../../raw/current/scripts/components/combat.lua#L1168
- 组件没有 OnSave／OnLoad、计时器或 OnRemoveFromEntity；持久状态和撤销由宿主维护。电羊的 charged 存档和食物 buff 的计时均不是本组件能力。[electricattacks.lua:3](../../../raw/current/scripts/components/electricattacks.lua#L3)、[lightninggoat.lua:169](../../../raw/current/scripts/prefabs/lightninggoat.lua#L169)、[foodbuffs.lua:206](../../../raw/current/scripts/prefabs/foodbuffs.lua#L206

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 10 处），未运行游戏。

- 源码审阅：2026-10-21，Steam buildid 25643519；完整审阅 electricattacks、SourceModifierList；定向核验 combat 刺激选择／电击倍率／CalcDamage／受击入口，weapon:SetElectric，免疫及湿度函数，默认常量，foodbuffs attach／detach／工厂与电羊充放电／反击／存档／注册。
- 结构与语法：2026-10-21，本页 validate_page.py 指定页检查通过（未检查导航登记）；validate_source_links.py --strict 通过；完整 modmain 示例经现有 parser_probe.exe --syntax-only 检查通过，仅验证 Lua 5.1 语法。
- 游戏运行：未运行；电羊近战的干湿目标、免疫／绝缘、充放电共存、读档及其他模组兼容待实测。未核验所有弹体、范围攻击或角色专属电击表现。

相关：[[reference/components/combat]]；[[reference/components/weapon]]。
