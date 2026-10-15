---
title: "grogginess"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "管理困倦累积、减速与昏睡请求；实际倒地和苏醒由状态图配合，区别生物 sleeper 与床具睡眠。"
aliases: ["困倦","昏睡","催眠抗性","昏睡免疫","grogginess"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/grogginess.lua","prefabs/player_common.lua","prefabs/player_common_extensions.lua","stategraphs/SGwilson.lua","util/sourcemodifierlist.lua","entityscript.lua","modutil.lua","tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# grogginess

## 快速判断

- 职责与适用：累积困倦、施加移动倍率、请求昏睡／苏醒；人物已有组件和相应状态图。生物日常睡眠用 [[reference/components/sleeper]]，床具用 [[reference/components/sleepingbag]]。见[接口](#关键接口)。
- 优先使用：AddGrogginess 增量；抗性与免疫按来源增删，效果结束成对撤销。见[接口与证据](#关键接口)。
- 重要副作用：达到门槛发 knockedout，不保证状态图接受；IsKnockedOut 读取 SG 的 knockout 标签，不读组件同名缓存。见[状态与清理](#状态与清理)。
- 修改边界：服务端；默认减速回调需要 locomotor，单独挂组件不是完整昏睡能力。人物死亡／复活会移除／重建组件，见[接入](#典型接入)。

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `AddGrogginess(amount, knockoutduration?)` | 正增量；最短昏睡时长为秒；无成功返回 | 非正数或有免疫来源时无动作。累加后执行 knockouttestfn；通过则取旧时长与本次时长较大值，并请求 KnockOut。[grogginess.lua:163](../../../raw/current/scripts/components/grogginess.lua#L163 |
| `SetResistance(value)`／`AddResistanceSource(source, value)`／`RemoveResistanceSource(source)` | 基础门槛＋加算来源 | 构造基础为 1，人物工厂设为 3；GetResistance 汇总。相同 source 覆盖，不提供额外 key 参数。[grogginess.lua:107](../../../raw/current/scripts/components/grogginess.lua#L107)、[:111](../../../raw/current/scripts/components/grogginess.lua#L111)、[:256](../../../raw/current/scripts/components/grogginess.lua#L256)、[player_common.lua:2823](../../../raw/current/scripts/prefabs/player_common.lua#L2823 |
| `AddImmunitySource(source)`／`RemoveImmunitySource(source)` | 来源对象 | 任一来源为真就免疫；Add／Subtract 被挡，下一次 OnUpdate 清空 grog_amount，但不直接退出昏睡 SG。[grogginess.lua:210](../../../raw/current/scripts/components/grogginess.lua#L210)、[:264](../../../raw/current/scripts/components/grogginess.lua#L264)、[:272](../../../raw/current/scripts/components/grogginess.lua#L272) |
| `SubtractGrogginess(amount)`／`ResetGrogginess()` | 正减量／无参数 | 减至至少 0，继续更新以完成减速恢复；Reset 复用 Subtract，也受免疫门禁，不是立即退出 SG。[grogginess.lua:210](../../../raw/current/scripts/components/grogginess.lua#L210)、[:223](../../../raw/current/scripts/components/grogginess.lua#L223) |
| `SetDecayRate(rate)`／`SetWearOffDuration(seconds)` | 每次更新扣量／恢复秒数 | OnUpdate 用 grog_amount - decayrate，**没有乘 dt**；昏睡计时与 wearofftime 才累加 dt。默认 decayrate 为 0.01，不能直接解读为每秒 0.01。[grogginess.lua:115](../../../raw/current/scripts/components/grogginess.lua#L115)、[:272](../../../raw/current/scripts/components/grogginess.lua#L272)、[tuning.lua:3158](../../../raw/current/scripts/tuning.lua#L3158 |
| `SetEnableSpeedMod(bool)`／`SetSpeedModMultiplier(mult)` | 开关／倍率 | 开关移除／恢复速度来源及 groggy 标签，不禁止累积和昏睡；倍率还乘已计算 speedmod，不是抗性。[grogginess.lua:42](../../../raw/current/scripts/components/grogginess.lua#L42)、[:123](../../../raw/current/scripts/components/grogginess.lua#L123) |
| `IsKnockedOut()`／`IsGroggy()`／`HasGrogginess()` | boolean | 首项查 SG；后两项还要求正量且启用减速，IsGroggy 排除已昏睡；不等于直接读取 grog_amount。[grogginess.lua:141](../../../raw/current/scripts/components/grogginess.lua#L141 |
| `SetKnockOutTest(fn)`／`SetComeToTest(fn)` | `(inst) -> truthy` | 单回调替换；默认昏睡测门槛与燃烧／火伤，默认苏醒需计时 **大于**时长且量低于抗性。人物另排除 yawn 状态。[grogginess.lua:49](../../../raw/current/scripts/components/grogginess.lua#L49)、[:99](../../../raw/current/scripts/components/grogginess.lua#L99)、[player_common_extensions.lua:12](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L12 |

## 状态与清理

KnockOut 对可见、未死亡实体发 `knockedout`；SGwilson 会因已有睡眠／起身／落水等状态忽略或排队。苏醒 ComeTo 仅在 SG 仍有 knockout 且未死亡时发 `cometo`，并先将量写为基础 resistance；SG 处理动画、骑乘解除与退出清理。因此 knockoutduration 是苏醒条件的一部分，不是准确的总动画时长。[grogginess.lua:241](../../../raw/current/scripts/components/grogginess.lua#L241)、[:248](../../../raw/current/scripts/components/grogginess.lua#L248)、[SGwilson.lua:2307](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2307)、[:12660](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12660)、[:12686](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12686)

归零后仍有 wearoff 恢复阶段，最后移除速度来源；移组件的 OnRemoveFromEntity 在 isgroggy 为真时清标签并调用 onwearofffn。默认回调负责撤销 locomotor 倍率，覆盖它须保留清理；宿主移组件会先停止更新。组件无 OnSave／OnLoad，TransferComponent 只向已有新组件转正量与部分计时，不代表所有来源都会迁移。[grogginess.lua:31](../../../raw/current/scripts/components/grogginess.lua#L31)、[:81](../../../raw/current/scripts/components/grogginess.lua#L81)、[:272](../../../raw/current/scripts/components/grogginess.lua#L272)、[:304](../../../raw/current/scripts/components/grogginess.lua#L304)、[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670

## 典型接入

### 角色困倦抗性增加两点并随组件重建恢复

目标模组 `modmain.lua`；复用原人物 locomotor／SG／资源。用 ComponentPostInit 而非只做 PlayerPostInit：死亡会移组件，复活会重新添加；人物在构造 grogginess 之前已有 player 标签。这里只登记独立加算来源，不覆盖后续工厂 SetResistance(3)。[modutil.lua:567](../../../raw/current/scripts/modutil.lua#L567)、[entityscript.lua:655](../../../raw/current/scripts/entityscript.lua#L655)、[player_common.lua:2452](../../../raw/current/scripts/prefabs/player_common.lua#L2452)、[player_common_extensions.lua:294](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L294)、[:632](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L632)

```lua
local RESIST_SOURCE = {}

AddComponentPostInit("grogginess", function(self, inst)
    if not GLOBAL.TheWorld.ismastersim or not inst:HasTag("player") then
        return
    end
    self:AddResistanceSource(RESIST_SOURCE, 2)
end)
```

原人物基础 3 时本来源贡献 +2；不是所有状态都保证必须累积到 5。改为特定角色时增加 Prefab 条件，来源 table 在本次模组加载内稳定复用；临时失效在当前组件上 `RemoveResistanceSource(RESIST_SOURCE)`，重新添加同源不叠加。本来源不存档，每次组件构造重施；停用后不再登记。实体来源删除可自动清理，table 来源须主动撤销。[grogginess.lua:111](../../../raw/current/scripts/components/grogginess.lua#L111)、[:256](../../../raw/current/scripts/components/grogginess.lua#L256)、[sourcemodifierlist.lua:64](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64

## 常见陷阱与限制

- **SetPercent(0) 清醒**：它算差量后仍调 AddGrogginess，负量会被拒绝；只会向上补量。要减量用 Subtract／Reset，退出 SG 仍按[状态流程](#状态与清理)。[grogginess.lua:196](../../../raw/current/scripts/components/grogginess.lua#L196
- **禁用减速等于昏睡免疫**：SetEnableSpeedMod 不挡 AddGrogginess 的 knockout 测试；真正免疫用来源接口，并接受已有昏睡要走苏醒条件。[grogginess.lua:123](../../../raw/current/scripts/components/grogginess.lua#L123)、[:180](../../../raw/current/scripts/components/grogginess.lua#L180)
- **提高门槛立即取消旧昏睡**：设置抗性不直接触发状态转换；CapToResistance 与 ComeTo 使用基础 resistance，不能把它们当完整来源汇总钳制。[grogginess.lua:229](../../../raw/current/scripts/components/grogginess.lua#L229)、[:248](../../../raw/current/scripts/components/grogginess.lua#L248)
- **ExtendKnockout 只会加长**：它把持续时长直接替换并重置已过时间，较小值也会写入；只在 SG 已昏睡时生效。[grogginess.lua:233](../../../raw/current/scripts/components/grogginess.lua#L233

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 4 处（player_common 组件构造／player 标签、tuning GROGGINESS_DECAY_RATE、SGwilson knockedout／cometo 三处）；无结论修订，仅锚点迁移——依据：全局枢纽 diff 审阅（mainfunctions 本版改动为错误界面迁入／落点保护／VAULT_KEY_FOUND，不涉实体睡醒分发）＋变更清单命中 player_common.lua、player_common_extensions.lua、SGwilson.lua、tuning.lua、entityscript.lua，grogginess 自身与 sourcemodifierlist 未变。
- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖累积／减量、来源门槛／免疫、更新单位、默认测试、SG 事件门禁与清理、人物死亡／复活重建及示例构造顺序。未审校全部困倦攻击来源、角色专用 SG、分片迁移或所有回调替换。
- 结构／语法：页面格式、来源、大小、链接与唯一登记通过，源码锚点 strict 检查无问题；本页完整 modmain 示例经 Full Moon 3.0.0／Lua 5.1 syntax-only 通过，未检查字节往返或运行效果。
- 游戏运行：未运行；抗性来源叠加、死亡／复活、减速恢复、多人显示与其他模组兼容待测。

相关：[[reference/components/locomotor#关键接口]]；[[reference/components/sleeper#关键接口]]。
