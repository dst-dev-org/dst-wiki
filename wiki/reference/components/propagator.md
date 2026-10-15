---
title: "propagator"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "管理火势累计热、闪点、周围传播及火伤，区分温度输出、直接点燃和防火门禁。"
aliases: ["火势传播", "火焰蔓延", "燃烧火伤", "闪点", "阻止受热", "propagator"]
tags: [component, mechanism, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/propagator.lua", "components/burnable.lua", "components/health.lua", "standardcomponents.lua", "prefabs/log.lua", "prefabs/campfire.lua", "prefabs/fire.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# propagator

## 快速判断

- 职责与适用：累计传播热／闪点、向附近对象传播与火伤；`heatoutput` 不是摄氏度，改变人物供热输出应查 [[reference/components/heater]]。[propagator.lua:137](../../../raw/current/scripts/components/propagator.lua#L137 [propagator.lua:174](../../../raw/current/scripts/components/propagator.lua#L174 [fire.lua:69](../../../raw/current/scripts/prefabs/fire.lua#L69
- 优先使用：接收方配置 acceptsheat／flashpoint／SetOnFlashPoint；燃烧源由 burnable 启停传播，调整范围、输出和 damages，保留原启停链。[standardcomponents.lua:200](../../../raw/current/scripts/standardcomponents.lua#L200 [burnable.lua:384](../../../raw/current/scripts/components/burnable.lua#L384
- 重要副作用：传播也会解冻、发 firemelt；火伤需目标有 propagator＋health，并受 health 自身防护／限额影响。[propagator.lua:197](../../../raw/current/scripts/components/propagator.lua#L197 [health.lua:190](../../../raw/current/scripts/components/health.lua#L190
- 修改边界：服务端；组件默认 acceptsheat=false、damages=false，helper 会改它们。Delay 只挡传入热量，不阻止已开始的对外传播。[propagator.lua:8](../../../raw/current/scripts/components/propagator.lua#L8 [propagator.lua:73](../../../raw/current/scripts/components/propagator.lua#L73

## 关键接口

| 接口／字段 | 参数或默认 | 条件、作用与证据 |
|---|---|---|
| `acceptsheat`／`AcceptsHeat()` | 默认 false／boolean | 正常传播先查 flag 且未 pauseheating；直接 AddHeat 不查此 flag。[propagator.lua:133](../../../raw/current/scripts/components/propagator.lua#L133 [propagator.lua:197](../../../raw/current/scripts/components/propagator.lua#L197 |
| `flashpoint`／`decayrate` | 默认 100／1 | 累计热严格大于闪点才触发；每周期先按 dt 衰减，有随机入热上限及地皮倍率。[propagator.lua:33](../../../raw/current/scripts/components/propagator.lua#L33 [propagator.lua:155](../../../raw/current/scripts/components/propagator.lua#L155 |
| `SetOnFlashPoint(fn)` | `(inst)` | 单槽回调；原 helper 用 DefaultIgniteFn 进入冒烟，不等于立刻 Ignite；controlled burn 来源可抑制本次回调。[propagator.lua:65](../../../raw/current/scripts/components/propagator.lua#L65 [standardcomponents.lua:6](../../../raw/current/scripts/standardcomponents.lua#L6 |
| `AddHeat(amount, source?)` | 传播热量、来源实体 | Delay／fireimmune 阻断；达到上限时限制，超闪点设置 pauseheating，再判断回调。[propagator.lua:137](../../../raw/current/scripts/components/propagator.lua#L137 |
| `Delay(seconds)` | 秒 | 临时挡传入 AddHeat，重复调用重排延迟；不是停止传播或熄灭。[propagator.lua:73](../../../raw/current/scripts/components/propagator.lua#L73 |
| `StartSpreading(source?)`／`StopSpreading(reset?, heatpct?)` | 来源／boolean、比例 | 开始周期任务；停止清 spreading/source，reset=true 还清／重置累计热与暂停，后续更新清融化标记并热归零时停任务。[propagator.lua:98](../../../raw/current/scripts/components/propagator.lua#L98 [propagator.lua:238-246](../../../raw/current/scripts/components/propagator.lua#L238-L246) |
| `propagaterange`／`heatoutput` | 默认 3／5 | 搜索半径和基础传播输出；每 0.5 秒更新，入热按距离衰减，春季放大范围。[propagator.lua:92](../../../raw/current/scripts/components/propagator.lua#L92 [propagator.lua:186](../../../raw/current/scripts/components/propagator.lua#L186 |
| `damages`／`damagerange` | 默认 false／3 | 火伤额约 output×dt，最终由 health 处理；目标必须先在传播搜索范围内，再通过伤害半径／组件条件，玩家来源另有 PvP 倍率。[propagator.lua:219](../../../raw/current/scripts/components/propagator.lua#L219 |

## 典型接入

### 降低木头燃烧时的传播输出并关闭本组件火伤

目标模组 `modmain.lua`；log 已由 MakeSmallPropagator 配置接收与闪点回调。只替换三个字段，保留 burnable 的点燃／熄灭启停与原版资源，不手动启动任务。[log.lua:42](../../../raw/current/scripts/prefabs/log.lua#L42 [standardcomponents.lua:200](../../../raw/current/scripts/standardcomponents.lua#L200 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("log", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local propagator = inst.components.propagator
    if propagator ~= nil then
        propagator.propagaterange = 3
        propagator.heatoutput = 3
        propagator.damages = false
    end
end)
```

3 是基础世界距离／传播输出参数；不代表 3℃ 或每个目标每秒恰好收到 3。春季仍放大范围、距离仍衰减；damages=false 仅关闭该 propagator 的伤害分支，不保证角色免疫其他火源。解冻／融化与附近可燃目标受热仍保留，火焰子 FX 的 heater 供热另行配置。[propagator.lua:184](../../../raw/current/scripts/components/propagator.lua#L184 [propagator.lua:197](../../../raw/current/scripts/components/propagator.lua#L197 [fire.lua:69](../../../raw/current/scripts/prefabs/fire.lua#L69

停用后新／重建 log 恢复 helper 配置；本组件没有 OnSave／OnLoad。移除组件时 StopSpreading(true) 并执行清理更新；实体移除另处理 firemelt。其他原版源如营火在燃料分段变化时重写输出／范围，不能把这段初始赋值直接推广为永久营火配置。[propagator.lua:38](../../../raw/current/scripts/components/propagator.lua#L38 [propagator.lua:47](../../../raw/current/scripts/components/propagator.lua#L47 [campfire.lua:74](../../../raw/current/scripts/prefabs/campfire.lua#L74

## 常见陷阱与限制

- **acceptsheat=false 就全面防火**：正常传播会跳过受热，但直接 AddHeat 不检查；Ignite 也不依赖它。fireimmune 阻断入热／点燃，propagator 火伤分支却不检查 fireimmune；防火伤需核验 health 防护与目标资格。[propagator.lua:137](../../../raw/current/scripts/components/propagator.lua#L137 [propagator.lua:219](../../../raw/current/scripts/components/propagator.lua#L219 [burnable.lua:354](../../../raw/current/scripts/components/burnable.lua#L354
- **过闪点后冷却就恢复接受热**：pauseheating 不在自然衰减中重置；需要原熄灭／StopSpreading(true) 等复位路径。直接调 Flash 仍受随机上限和地皮抗热约束，不保证即刻点燃。[propagator.lua:104](../../../raw/current/scripts/components/propagator.lua#L104 [propagator.lua:167](../../../raw/current/scripts/components/propagator.lua#L167
- **伤害半径可独立设得更远**：实体候选先由 propagaterange 搜索；damagerange 再大也扩不了候选。输出设 0 仍可能发 firemelt，不等于关闭所有燃烧影响。[propagator.lua:186](../../../raw/current/scripts/components/propagator.lua#L186 [propagator.lua:213](../../../raw/current/scripts/components/propagator.lua#L213
- **CanSpreadHeat 自动禁止对外传播**：它是地皮查询函数，OnUpdate 没调用它；no_fire_spread 在 GetHeatResistance 阻断传入热。不能仅看函数名推广为统一对外门禁。[propagator.lua:114](../../../raw/current/scripts/components/propagator.lua#L114 [propagator.lua:128](../../../raw/current/scripts/components/propagator.lua#L128 [propagator.lua:174](../../../raw/current/scripts/components/propagator.lua#L174

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖累计热／闪点／地皮、传播与伤害资格、启停／移除、helper 与 log 案例；核验营火覆盖、燃烧启动和 fire FX 供热分工。全部冷冻／融化消费者、完整免伤叠加与游戏时序未覆盖。
- 结构检查：元数据、证据行、链接／章节、大小、登记通过；发布示例及木头燃烧＋传播组合经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；春季范围、受热延迟、多火源／PvP、熄灭／移除、FX 温度待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/burnable#生命周期]]；[[reference/components/heater#快速判断]]；[[reference/components/health#关键接口]]。
