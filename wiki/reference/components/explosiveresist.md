---
title: "explosiveresist"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "爆炸受击累积的临时抗性与延迟衰减；输入为护甲前已乘本次抗性的普通爆伤，不是实际掉血。"
aliases: ["熊獾连续爆炸减伤衰减", "爆炸抗性积累", "explosiveresist"]
tags: [component, creature, lifecycle, tuning, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/explosiveresist.lua", "components/explosive.lua", "prefabs/bearger.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# explosiveresist

## 快速判断

- 职责与适用：保存 0～1 临时抗性，爆炸命中后累积、延迟后随组件更新衰减；不是按攻击标签配置的 [[reference/components/damagetyperesist]]。[组件](../../../raw/current/scripts/components/explosiveresist.lua#L1-L46)
- 优先使用：改变衰减速度调 decaytime；显式调整当前值用 SetResistance／DoDelta，不直接写 resistance 跳过更新启停。[设置](../../../raw/current/scripts/components/explosiveresist.lua#L26-L46)
- 重要副作用：原 explosive 先把普通爆伤乘 (1-resistance)，再把该数值传 OnExplosiveDamage，之后才 GetAttacked；累计依据不是最终生命损失。[调用顺序](../../../raw/current/scripts/components/explosive.lua#L98-L117)
- 修改边界：服务端；只有实际消费该组件的爆炸链有效，此处特殊伤害不乘该抗性，直接 combat／health 调用不自动识别“爆炸”。26230496 起抗性消费仍在 notpvp 门禁之前——PVP 被炸玩家不掉血但仍会积累抗性。[消费者](../../../raw/current/scripts/components/explosive.lua#L98-L118)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `maxresistdamage`／`decaytime`／`decaydelay` | 原默认 8000／8 秒／2 秒；分别控制累积尺度、每秒衰减 1/decaytime 与延迟。组件直接读字段，不校验正值，避免零／负时间或分母。[构造](../../../raw/current/scripts/components/explosiveresist.lua#L1-L9) [当前调参](../../../raw/current/scripts/tuning.lua#L2358-L2360) |
| `Enable(boolval)` | 布尔开关 | 禁用后 `OnExplosiveDamage` 不再积累抗性，`GetResistance()` 恒返回 0；enabled 默认 true 且不存档，构造时复位。[实现](../../../raw/current/scripts/components/explosiveresist.lua#L11-L13) |
| `OnExplosiveDamage(damage,src)` | damage>0 且组件未禁用才重设 delayremaining=decaydelay，再加 damage/maxresistdamage；src 当前未使用。零伤害或禁用时不刷新延迟。[实现](../../../raw/current/scripts/components/explosiveresist.lua#L15-L20) |
| `GetResistance()`／`DoDelta(delta)` | 读取当前值（禁用时返回 0）／经 setter 增减；delta 是抗性小数，不是伤害点数。[实现](../../../raw/current/scripts/components/explosiveresist.lua#L22-L28) |
| `SetResistance(n)` | 夹在 0～1；正值启动组件更新，零停止并清剩余延迟。单独 setter 不会重置为 decaydelay。[实现](../../../raw/current/scripts/components/explosiveresist.lua#L30-L38) |
| `OnUpdate(dt)` | dt<delayremaining 时只减延迟；否则按整个 dt/decaytime 扣抗性，未按跨延迟的剩余帧时长精确拆算，不保证精确秒级边界。[实现](../../../raw/current/scripts/components/explosiveresist.lua#L40-L46) |

保存仅在抗性至少 0.01 时输出向下取整的百分数；加载乘 0.01 还原并启更新，不保存 delayremaining、enabled 或自定义参数，没有自身 LongUpdate。读档不能承诺完整恢复爆炸后的等待窗口。[存读](../../../raw/current/scripts/components/explosiveresist.lua#L48-L58)

（26148036 核验：新增 `Enable(boolval)` 开关——禁用时 `OnExplosiveDamage` 不积累、`GetResistance()` 返回 0；原累积／衰减链与消费顺序不变。）
（26230496 核验：消费链锚点随 explosive 的 SetKnockbackData 改动重定位——抗性消费块仍在 notpvp 伤害门禁之前，PVP 下伤害被跳过时抗性照常累积；累积／衰减逻辑本身未变。）

## 典型接入

### 熊獾保持原爆炸抗性累积，衰减速度改为每秒 1/20

目标模组 `modmain.lua`，复用原 bearger 和资源；原工厂已有 combat／explosiveresist，不重新挂组件或主动制造爆炸。[原安装](../../../raw/current/scripts/prefabs/bearger.lua#L476-L483)

```lua
AddPrefabPostInit("bearger", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local resist = inst.components.explosiveresist
    if resist ~= nil then
        resist.decaytime = 20
    end
end)
```

20 是衰减速度的秒尺度：进入扣值分支时每秒扣约 0.05 抗性，不是“每次爆炸后固定免伤 20 秒”，也不是设置初始 20% 抗性。原 maxresistdamage／decaydelay 和累计逻辑保留；无新的爆伤时抗性最终回零停止更新。[衰减／启停](../../../raw/current/scripts/components/explosiveresist.lua#L30-L46)

配置在构造时重施，组件保存当前抗性而不保存 decaytime；停用并完整重载后恢复原速度，但存档中的抗性仍按原 OnLoad 恢复，不等于清零。当前实例热撤需要恢复本次改写前值，本例不提供热卸载。

### 原爆炸消费顺序

原 explosive 在目标可被攻击时，普通总伤害先乘源 damagetypebonus，再乘 (1-当前抗性)，把结果用于增抗，然后单独收集特殊伤害，最后交 combat。即使之后护甲抵掉伤害，抗性也可能已累积；特殊伤害不在此处乘 (1-resistance)。更完整爆炸门禁／工作量见 [[reference/components/explosive]]。[原链](../../../raw/current/scripts/components/explosive.lua#L98-L117)

## 常见陷阱与限制

- **按未减伤原爆伤积累**：消费处实际传入已扣本次爆炸抗性的 dmg；抗性越高，相同后续原爆伤带来的新增抗性可能越少。[顺序](../../../raw/current/scripts/components/explosive.lua#L98-L102)
- **任何爆炸表现都触发**：组件不会监听视觉爆炸或 attacked 自动识别来源，必须有真实调用；自定义爆炸另核验其伤害入口。
- **SetResistance(1) 永久免疫**：正值会启动衰减，而且这里只消费普通爆伤；该 setter 不建立持续 Buff。[设置](../../../raw/current/scripts/components/explosiveresist.lua#L30-L46) [特殊伤害](../../../raw/current/scripts/components/explosive.lua#L104-L117)
- **存读精确保留小数与等待**：百分数向下取整，低于 0.01 不保存，延迟未存；不要据此实现精确护盾时序。[保存](../../../raw/current/scripts/components/explosiveresist.lua#L48-L58)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 15 处、结论修订 3 处（Enable 开关，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖完整组件、爆炸目标门禁／普通与特殊伤害调用顺序、熊獾安装及默认调参；未审校全部爆炸宿主、熊獾变体／阶段、所有后续 combat 事件或离线补时机制。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整熊獾配置例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；连续／同帧爆炸、抗性满值、延迟跨帧、护甲／整次挡伤、存读和变体待实测。
