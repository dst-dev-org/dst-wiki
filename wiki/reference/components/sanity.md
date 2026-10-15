---
title: "sanity"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "修改精神值与上限、持续精神贡献和启蒙状态，区分装备贡献、额外速率与净增长。"
aliases: ["精神值", "精神上限", "回精神", "精神每分钟提升三点", "持续精神恢复", "启蒙", "sanity"]
tags: [component, lifecycle, mechanism, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/sanity.lua", "components/equippable.lua", "components/health.lua", "prefabs/player_common.lua", "prefabs/wintersfeastbuff.lua", "util/sourcemodifierlist.lua", "modutil.lua", "entityscript.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sanity

## 快速判断

- 职责与适用：人物精神当前值、上限／惩罚、清醒／疯狂／启蒙及持续变化；装备提供 dapperness，sanity 汇总为人物速率。[sanity.lua:377](../../../raw/current/scripts/components/sanity.lua#L377)、[:510](../../../raw/current/scripts/components/sanity.lua#L510)
- 优先使用：即时变化 DoDelta，上限 SetMax；独立持续贡献用 externalmodifiers；只有装备生效时优先 [[reference/components/equippable#修改已有手杖装备加速与精神基础贡献]]。[sanity.lua:312](../../../raw/current/scripts/components/sanity.lua#L312)、[:590](../../../raw/current/scripts/components/sanity.lua#L590)
- 重要副作用：DoDelta 封顶并更新 sanitydelta／清醒状态事件；SetMax 补满，GetPercent 可被强制疯狂／启蒙覆盖，不总等于 current/max。[sanity.lua:288](../../../raw/current/scripts/components/sanity.lua#L288)、[:377](../../../raw/current/scripts/components/sanity.lua#L377)
- 修改边界：服务端配置；速率按点／秒，最终受环境、装备、rate_modifier 与暂停条件影响。增加三点／分钟的贡献不保证净增三点。[sanity.lua:437](../../../raw/current/scripts/components/sanity.lua#L437)、[:590](../../../raw/current/scripts/components/sanity.lua#L590)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `SetMax(amount)`／`SetCurrent(amount)` | 正数上限／绝对值 | SetMax 同时补满；两者再 DoDelta(0)，仍有 redirect／ignore 限制。[sanity.lua:312](../../../raw/current/scripts/components/sanity.lua#L312 |
| `DoDelta(delta, overtime?)`／`SetPercent(per, overtime?)` | 相对量／基础 max 比例 | redirect 接管或 ignore 时返回；DoDelta 限制到惩罚后上限并更新状态，无实际增量返回约定。[sanity.lua:302](../../../raw/current/scripts/components/sanity.lua#L302)、[:377](../../../raw/current/scripts/components/sanity.lua#L377) |
| `GetRealPercent()`／`GetPercent()`／`GetPercentWithPenalty()` | 真值比例／有效状态比例／惩罚后比例 | 真实值用 current/max；后两项考虑 inducedinsanity／inducedlunacy，分母不同。[sanity.lua:284](../../../raw/current/scripts/components/sanity.lua#L284 |
| `externalmodifiers` | 加法 SourceModifierList，基础 0 | SetModifier(source, rate, key?)／RemoveModifier(source, key?)；rate 点／秒，不是倍率；实体 source 删除自动撤销。[sanity.lua:87](../../../raw/current/scripts/components/sanity.lua#L87)、[sourcemodifierlist.lua:64](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64 |
| `custom_rate_fn` | `(inst, dt) -> number` | 返回尚未乘 dt 的附加速率，单函数字段；独立效果优先列表，避免替换人物原函数。[sanity.lua:592](../../../raw/current/scripts/components/sanity.lua#L592 |
| `AddSanityPenalty(key, mod)`／`RemoveSanityPenalty(key)` | 惩罚比例贡献 | 同 key 替换，重算后刷新；可用上限最低保留 5 点。[sanity.lua:189](../../../raw/current/scripts/components/sanity.lua#L189 |
| `EnableLunacy(enable, source)` | 启用标志＋来源 key | 汇总来源后切换模式并发 sanitymodechanged；SetSanityMode 已是空的弃用接口。[sanity.lua:159](../../../raw/current/scripts/components/sanity.lua#L159)、[:184](../../../raw/current/scripts/components/sanity.lua#L184) |
| `SetInducedInsanity(src, val)`／`SetInducedLunacy(src, val)` | 来源＋开关 | 强制状态并刷新；不等同于耗尽／加满真实精神值。[sanity.lua:331](../../../raw/current/scripts/components/sanity.lua#L331 |

## 典型接入

### 给人物增加每分钟三点精神贡献

目标模组 `modmain.lua`，普通生存模式；已有人物组件，无新增资源。使用独立 key，不自建每秒 DoDelta 任务，不覆盖人物 custom_rate_fn。原版盛宴 buff 用同一加法列表增删贡献。[modutil.lua:586](../../../raw/current/scripts/modutil.lua#L586)、[wintersfeastbuff.lua:57](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L57)、[:64](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L64)

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.sanity ~= nil then
        inst.components.sanity.externalmodifiers:SetModifier(
            inst, 3 / 60, "dstwiki_sanity_passive")
    end
end)
```

按真实分钟换算成 0.05／秒；这是该实例常驻贡献，最终速率还乘 rate_modifier。临时 buff 结束时用同列表 `RemoveModifier(inst, "dstwiki_sanity_passive")`；独立 buff／物品可改用其实体作 source，状态失效时成对撤销，不能只等待来源删除。此页不实现物品栏持有检测，按 [[reference/components/inventoryitem]] 的 owner／移出边界组装。[sanity.lua:590](../../../raw/current/scripts/components/sanity.lua#L590)、[sourcemodifierlist.lua:103](../../../raw/current/scripts/util/sourcemodifierlist.lua#L103

### 装备、随身物品与附近光环如何选择

- 装备贡献：Recalc 只遍历 inventory.equipslots；普通物品栏里有 equippable.dapperness 不生效。卸下即不再汇总，无需自己撤销速率任务。[sanity.lua:510](../../../raw/current/scripts/components/sanity.lua#L510
- 仅持有者受益：用 inventoryitem 的实际持有范围决定给哪个人物挂 externalmodifiers，转移／掉落时移除旧人物项；机制链接 [[reference/components/inventoryitem#生命周期]]，来源删除只是兜底清理。
- 附近实体光环：Recalc 按 sanityaura 标签找周围实体并排除 INLIMBO；不要把入栏物品当稳定世界光环。发送值／范围／去重见 [[reference/components/sanityaura#关键接口]]。[sanity.lua:488](../../../raw/current/scripts/components/sanity.lua#L488)、[:546](../../../raw/current/scripts/components/sanity.lua#L546)

## 生命周期

- 构造启动组件更新；OnUpdate 在无敌、出生保护、睡眠、传送和相应 ignore 条件下暂停速率计算。直接 DoDelta 的限制仅 redirect／ignore，不能把持续更新的限制推广到所有入口。[sanity.lua:122](../../../raw/current/scripts/components/sanity.lua#L122)、[:377](../../../raw/current/scripts/components/sanity.lua#L377)、[:437](../../../raw/current/scripts/components/sanity.lua#L437)
- OnSave 保存 current／sane／mode，不保存持续修饰列表、上限或强制状态来源；OnLoad 恢复并刷新。组件没有自己的 OnRemoveFromEntity；宿主 RemoveComponent 会停止更新。[sanity.lua:258](../../../raw/current/scripts/components/sanity.lua#L258)、[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670)、[:1928](../../../raw/current/scripts/entityscript.lua#L1928)

## 常见陷阱与限制

- **把外部贡献写成倍率 1.2**：externalmodifiers 是加法速率，将变成额外 1.2／秒；与 hunger／combat 的乘法列表区分。[sanity.lua:87](../../../raw/current/scripts/components/sanity.lua#L87
- **custom_rate_fn 返回 rate * dt**：Recalc 最后还乘 dt，会重复乘时间；返回原速率或改用独立列表。[sanity.lua:592](../../../raw/current/scripts/components/sanity.lua#L592)、[:609](../../../raw/current/scripts/components/sanity.lua#L609)
- **直接 current 赋值代表清醒状态更新**：属性只同步当前网络值；状态阈值和事件在 DoDelta，优先用公开入口。[sanity.lua:30](../../../raw/current/scripts/components/sanity.lua#L30)、[:377](../../../raw/current/scripts/components/sanity.lua#L377)
- **低精神必定疯狂／高精神必定清醒**：lunacy 模式阈值方向不同，还有强制来源；按 IsInsane／IsEnlightened 判定。[sanity.lua:138](../../../raw/current/scripts/components/sanity.lua#L138)、[:391](../../../raw/current/scripts/components/sanity.lua#L391)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 2 处（entityscript／tuning 行号迁移）；结论修订：无，仅锚点迁移。
- 依据：全局枢纽 hunk 地图（R-08）＋变更清单命中 entityscript.lua、tuning.lua、prefabs/player_common.lua。

- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖上述数值／状态、速率、独立来源、装备／光环接收、保存恢复与宿主；未审校所有光环免疫、鬼魂与人物特殊规则。当前 SANITY_DAPPERNESS=1。[tuning.lua:2183](../../../raw/current/scripts/tuning.lua#L2183
- 结构检查：页面／索引、链接与源码行锚点检查通过；Lua 经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；分钟速率、上限、暂停／恢复、模式切换、重载及其他模组协作待实测。

相关：[[reference/components/hunger]]；[[reference/components/eater]]；[[reference/components/equippable]]。
