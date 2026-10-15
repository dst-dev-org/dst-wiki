---
title: "sanityaura"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "为附近人物提供精神光环，配置速率、距离衰减与同名去重，区分携带者专属效果。"
aliases: ["精神光环", "附近回精神", "光环衰减", "光环不叠加", "sanityaura"]
tags: [component, aura, mechanism, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/sanityaura.lua", "components/sanity.lua", "prefabs/glommer.lua", "prefabs/vault_sanity_adjuster.lua", "modutil.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sanityaura

## 快速判断

- 职责与适用：实体向附近具有 sanity 的接收者提供正／负精神速率；由接收者搜索并汇总，不自建周期回精神任务。[sanity.lua:546](../../../raw/current/scripts/components/sanity.lua#L546
- 优先使用：固定贡献 aura，观察者差异 aurafn，限定距离 max_distsq，控制衰减 fallofffn；仅持有者受益用 [[reference/components/sanity#装备随身物品与附近光环如何选择]]。
- 重要副作用：构造／移组件增删 sanityaura 标签；默认 aura=0，按距离平方衰减。同 baseauraname 在一次接收汇总只算一个来源。[sanityaura.lua:1](../../../raw/current/scripts/components/sanityaura.lua#L1)、[:20](../../../raw/current/scripts/components/sanityaura.lua#L20)
- 修改边界：服务端配置。普通周围搜索排除 INLIMBO／FX／NOCLICK／DECOR 和接收者自身；数值是点／秒贡献，实际净变化仍受人物规则影响。[sanity.lua:488](../../../raw/current/scripts/components/sanity.lua#L488)、[:590](../../../raw/current/scripts/components/sanity.lua#L590)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `aura`／`aurafn` | 默认 0；fn(inst, observer)->number | fn 非 nil 时替代 aura；返回必须是数值，nil 不自动回退。[sanityaura.lua:16](../../../raw/current/scripts/components/sanityaura.lua#L16)、[:30](../../../raw/current/scripts/components/sanityaura.lua#L30) |
| `GetBaseAura(observer)` | 无距离衰减的速率 | 直接读 aura／调用 aurafn，不检查范围；不是普通人物最终收到的速率。[sanityaura.lua:16](../../../raw/current/scripts/components/sanityaura.lua#L16 |
| `GetAura(observer)` | 距离处理后速率 | 范围内以 aura／aurafn 除以 fallofffn 返回值，或默认 max(1,distsq)；范围外 0。[sanityaura.lua:26](../../../raw/current/scripts/components/sanityaura.lua#L26 |
| `max_distsq` | 距离平方；省略用 10² | 须写 radius*radius；接收者还只搜 20 单位，单独调大此字段不能扩大搜索。[sanityaura.lua:25](../../../raw/current/scripts/components/sanityaura.lua#L25)、[sanity.lua:548](../../../raw/current/scripts/components/sanity.lua#L548)、[tuning.lua:2233](../../../raw/current/scripts/tuning.lua#L2233)、[:2234](../../../raw/current/scripts/tuning.lua#L2234) |
| `fallofffn` | fn(inst, observer, distsq)->非零数值 | 返回除数；返回 1 表示范围内无衰减，不是乘数或范围开关。[sanityaura.lua:30](../../../raw/current/scripts/components/sanityaura.lua#L30)、[vault_sanity_adjuster.lua:1](../../../raw/current/scripts/prefabs/vault_sanity_adjuster.lua#L1 |
| `SetBaseAuraName(name)` | 同类来源名称 | 同名只汇总首个通过检查的实体，假定幅值相同；没有择强／最近保证，nil 取消命名去重。[sanityaura.lua:20](../../../raw/current/scripts/components/sanityaura.lua#L20)、[sanity.lua:493](../../../raw/current/scripts/components/sanity.lua#L493 |

## 典型接入

### 让格罗姆在六单位内提供每分钟三点无衰减精神贡献

目标模组 `modmain.lua`；复用 glommer 原版组件与资源，不新增任务。主动替换其原版光环值；保留已有工厂／跟随行为。无衰减参照 vault_sanity_adjuster；本例取消命名去重，让各实体独立按六单位范围计算，避免范围外同名实体先占位。[glommer.lua:132](../../../raw/current/scripts/prefabs/glommer.lua#L132)、[:160](../../../raw/current/scripts/prefabs/glommer.lua#L160)、[vault_sanity_adjuster.lua:1](../../../raw/current/scripts/prefabs/vault_sanity_adjuster.lua#L1)、[modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("glommer", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local aura = inst.components.sanityaura
    if aura == nil then return end
    aura.aurafn = nil
    aura.aura = 3 / 60
    aura.max_distsq = 6 * 6
    aura.fallofffn = function(source, observer, distsq) return 1 end
    aura:SetBaseAuraName(nil)
end)
```

3/60 为每个实体的 0.05 点／秒贡献；接收者进入范围后在自身更新中计入，离开后贡献为零，多只范围内格罗姆可叠加。确需同类不叠加时再设稳定名字并核验下文去重限制，不能仅改名字就宣称始终择优。[sanity.lua:568](../../../raw/current/scripts/components/sanity.lua#L568)、[:598](../../../raw/current/scripts/components/sanity.lua#L598)

这不是人物净增三点／分钟：光照、湿度、装备、其他贡献及 rate_modifier 仍参与；光环免疫和 sanity 更新暂停也可能使其不生效。暂时关闭本光环可设 aura=0（本例已清 aurafn），需要永久移组件时 OnRemoveFromEntity 清标签；停用后新实例恢复原版定义。组件不存配置，初始化时重建。[sanity.lua:437](../../../raw/current/scripts/components/sanity.lua#L437)、[:546](../../../raw/current/scripts/components/sanity.lua#L546)、[sanityaura.lua:12](../../../raw/current/scripts/components/sanityaura.lua#L12

### 附近光环、装备与随身效果如何选择

- 世界实体惠及附近人物：用本组件。普通搜索过滤标签；负光环还受吸收、倍率和免疫处理，启蒙等状态规则由 [[reference/components/sanity]] 决定。[sanity.lua:546](../../../raw/current/scripts/components/sanity.lua#L546
- 仅装备者受益：用 [[reference/components/equippable#关键接口]] 的 dapperness；普通入栏不汇总装备值。[sanity.lua:514](../../../raw/current/scripts/components/sanity.lua#L514
- 仅携带者受益：按 [[reference/components/inventoryitem#生命周期]] 核验主栏／背包／手持及归属清理，再使用 [[reference/components/sanity#给人物增加每分钟三点精神贡献]] 的 externalmodifiers。入栏物品通常 INLIMBO，不能把本组件当成稳定的持有者效果。

## 常见陷阱与限制

- **fallofffn 返回 0 表示无衰减**：它是除数，0 会造成无效数值；无衰减返回 1，范围用 max_distsq。[sanityaura.lua:30](../../../raw/current/scripts/components/sanityaura.lua#L30
- **aura=3 就是每分钟三点**：消费者最后乘 dt，直接填 3 是三点／秒；先按实际秒数换算，不把额外贡献当净增长。[sanity.lua:590](../../../raw/current/scripts/components/sanity.lua#L590)、[:609](../../../raw/current/scripts/components/sanity.lua#L609)
- **同名来源自动选择最近／最强**：去重先于 GetAura，不按最终贡献排序；20 单位搜索内但在自身作用范围外的同名实体也可能先占位，让后续范围内来源被跳过。相同幅值不保证距离后的效果相同；独立范围用 nil 名称，需要不叠加时另核验选取策略。[sanity.lua:493](../../../raw/current/scripts/components/sanity.lua#L493)、[:568](../../../raw/current/scripts/components/sanity.lua#L568)
- **FX 光环／骑乘按普通周围搜索推断**：普通搜索排除 FX 等标签；坐骑另有直接 GetAura 汇总路径，不能把普通搜索过滤与光环免疫结论一概推广到坐骑。[sanity.lua:488](../../../raw/current/scripts/components/sanity.lua#L488)、[:578](../../../raw/current/scripts/components/sanity.lua#L578)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 全部对照 build-26148036 更新报告与新旧 tuning.lua diff 核验，仅 tuning.lua 变更——SANITY_EFFECT_RANGE／SANITY_AURA_SEACH_RANGE 值不变、锚点重定位（2223→2233；20 单位搜索键 2234），其余 sanity 键均为新增（CRAZINESS_TINY、BATBAT_*、SANITYMONSTERS_CHARLIEARENA_*）不涉本页；其余 5 个 sources 字节级未变；vault→virtualroom 房间系统迁移（P-122／P-123／P-124）不改本页结论，vault_sanity_adjuster 仍为注册 prefab 且在现行 vaultroom_defs 布局中生成；未覆盖：运行验证。

- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖发送 getter／距离／去重、sanity 接收／单位／过滤、格罗姆参照和无衰减接入；未全量审校人物特殊免疫、坐骑或所有光环实现。
- 结构检查：页面、导航、链接与源码行锚点检查通过；Lua 经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；范围进出、同名多实体选择、分钟贡献、暂停／免疫、多人和其他模组协作待实测。

相关：[[reference/components/sanity]]；[[reference/components/inventoryitem#生命周期]]；[[reference/components/equippable]]。
