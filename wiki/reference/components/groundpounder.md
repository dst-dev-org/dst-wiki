---
title: "groundpounder"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "以逐环震地执行范围战斗、工作破坏、点燃和物品／船只推开，并由宿主 SG 决定触发时机。"
aliases: ["groundpounder", "震地攻击", "震地环", "范围砸树矿", "熊獾砸地", "地面冲击波"]
tags: [component, actions, callback, state-machine, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/groundpounder.lua, components/combat.lua, prefabs/bearger.lua, stategraphs/SGbearger.lua, prefabs/minotaur.lua, prefabs/woodie.lua, entityscript.lua]
reliability: verified
validation: source-reviewed
status: active
---

# groundpounder

## 快速判断

- 职责与适用：宿主在指定中心依次产生震地环，按圈数决定伤害、工作／点燃、推船、弹射物品与地面特效；不负责选择敌人、播放动作或决定何时攻击。[逐环调度](../../../raw/current/scripts/components/groundpounder.lua#L307-L357) [原版 SG 触发](../../../raw/current/scripts/stategraphs/SGbearger.lua#L1151-L1170)
- 优先使用：新增接入显式 `UseRingMode()`，配置环数与作用圈，再从宿主攻击状态调用 `GroundPound()`；默认仍是为旧模组保留的 PointMode。[默认与模式差异](../../../raw/current/scripts/components/groundpounder.lua#L25-L34) [熊獾配置](../../../raw/current/scripts/prefabs/bearger.lua#L504-L513)
- 重要副作用：RingMode 暂时关闭 Combat 附带范围伤害并开启 `ignorehitrange`；共享命中表跨环阻止重复战斗攻击，不阻止同一对象再次被工作或弹射。[战斗门禁](../../../raw/current/scripts/components/groundpounder.lua#L170-L178) [各处理分支](../../../raw/current/scripts/components/groundpounder.lua#L193-L244) [结束赋值](../../../raw/current/scripts/components/groundpounder.lua#L292-L294)
- 修改边界：原版在服务端挂载、配置 Combat 并由服务端 SG 调用；本组件没有 replica、存档或移除清理钩子。不能靠客户端调用同步伤害，也不能把移除组件当作取消已排队震地。[端分界](../../../raw/current/scripts/prefabs/bearger.lua#L458-L460) [Combat](../../../raw/current/scripts/prefabs/bearger.lua#L476) [SG](../../../raw/current/scripts/prefabs/bearger.lua#L549-L550) [任务调度](../../../raw/current/scripts/components/groundpounder.lua#L344-L354) [组件移除](../../../raw/current/scripts/entityscript.lua#L670-L683)

## 关键接口

| 接口／配置 | 参数、默认值或条件 | 作用与证据 |
|---|---|---|
| `UseRingMode()` | 无参；设置 `usePointMode=nil` | 用同心环搜索实体，跨环共享战斗／船只命中集合；新接入优先此模式。[实现](../../../raw/current/scripts/components/groundpounder.lua#L25-L34) |
| `GroundPound(pt, ents_hit)` | pt 默认宿主当前位置；ents_hit 可省略或传实体→真值表 | 先生成中心环特效，再排队逐环任务；没有攻击完成返回值。传入表用于排除已尝试战斗命中的实体，只在 RingMode 传递到各环。[入口](../../../raw/current/scripts/components/groundpounder.lua#L307-L330) [调度](../../../raw/current/scripts/components/groundpounder.lua#L331-L357) |
| `numRings / ringDelay` | 默认 `4 / 0.2`；延迟单位秒 | 第 1 环延迟 0，第 i 环延迟 `(i-1)*ringDelay`；任务仍通过 `DoTaskInTime` 执行。[默认](../../../raw/current/scripts/components/groundpounder.lua#L4-L5) [调度](../../../raw/current/scripts/components/groundpounder.lua#L324-L357) |
| `initialRadius / radiusStepDistance / ringWidth` | 默认 `1 / 4 / 3`；世界距离 | 第 i 环半径 `initialRadius+(i-1)*radiusStepDistance`；实体搜索覆盖半径上下各 ringWidth 的环带，内边界不小于 0。[默认](../../../raw/current/scripts/components/groundpounder.lua#L6-L8) [环带](../../../raw/current/scripts/components/groundpounder.lua#L181-L189) |
| `damageRings / groundpounddamagemult` | 默认 `2 / 1` | 前 damageRings 环尝试 `combat:DoAttack(target,nil,nil,nil,mult)`；要求存活 Health、CanTarget，玩家宿主额外排除 Combat:IsAlly。未配置 Combat 不能执行伤害环。[默认](../../../raw/current/scripts/components/groundpounder.lua#L10) [倍率](../../../raw/current/scripts/components/groundpounder.lua#L22) [命中](../../../raw/current/scripts/components/groundpounder.lua#L221-L233) |
| `destructionRings / destroyer / workefficiency / burner` | 默认 `3 / false / nil / false` | 前指定环先工作、再尝试点燃；workefficiency 非 nil 用 WorkedBy，否则需 destroyer 才 Destroy。排除捕虫 NET；点燃还排除 Fueled、已燃烧／烧焦对象。[默认](../../../raw/current/scripts/components/groundpounder.lua#L11-L19) [处理](../../../raw/current/scripts/components/groundpounder.lua#L193-L216) |
| `platformPushingRings / inventoryPushingRings` | 默认 `2 / 0` | 前指定环推有 boatphysics 的平台，或停用并弹射 inventoryitem；RingMode 船只按整次震地去重，PointMode 仅在单环内去重。[默认](../../../raw/current/scripts/components/groundpounder.lua#L12-L15) [物品](../../../raw/current/scripts/components/groundpounder.lua#L237-L244) [平台](../../../raw/current/scripts/components/groundpounder.lua#L252-L277) |
| `groundpoundfx / groundpoundringfx / fxRings` | 默认两种原版 FX；fxRings 默认取 numRings | 中心环 FX 每次调用都会生成；点状地面 FX 才受 fxRings、陆地且非码头门禁。自定义 prefab 必须可生成，不可用 nil 当关闭开关。[默认](../../../raw/current/scripts/components/groundpounder.lua#L13-L21) [地面 FX](../../../raw/current/scripts/components/groundpounder.lua#L284-L287) [中心 FX](../../../raw/current/scripts/components/groundpounder.lua#L310-L320) [圈数](../../../raw/current/scripts/components/groundpounder.lua#L347-L351) |
| `pointDensity / fxRadiusOffset` | 默认 `.25`；offset 未赋值、RingMode 才支持 | GetPoints 生成环上 FX 点；RingMode 的实体伤害环仍使用原始半径，调点密度／FX 偏移不等于调伤害范围。[生成点](../../../raw/current/scripts/components/groundpounder.lua#L37-L66) [分开传参](../../../raw/current/scripts/components/groundpounder.lua#L344-L346) |
| `noTags / groundpoundFn` | 默认排除 FX、NOCLICK、DECOR、INLIMBO、groundpound_immune；回调初始 nil | noTags 用于实体／船只搜索。groundpoundFn(inst) 在所有环任务排队后立即调用，不是逐目标或所有环完成通知。[默认](../../../raw/current/scripts/components/groundpounder.lua#L16-L23) [实体查询](../../../raw/current/scripts/components/groundpounder.lua#L181) [回调](../../../raw/current/scripts/components/groundpounder.lua#L360-L361) |

## 典型接入

### 生物攻击：工厂配置作用圈，SG 在动作落点触发

- **位置与前提：** 原版熊獾工厂先过服务端门禁，挂 Combat 与 groundpounder，设 RingMode、3 圈、间隔半径 2、宽度 1.5、damage／destruction／platform 圈均为 3；`destroyer=true` 允许摧毁可工作对象。普通熊獾的 Combat 伤害由其独立工厂设置，震地倍率不是最终固定伤害。[组件配置](../../../raw/current/scripts/prefabs/bearger.lua#L504-L513) [普通形态伤害](../../../raw/current/scripts/prefabs/bearger.lua#L575)
- **触发与资源：** `SGbearger` 的 pound 状态播放 `ground_pound`，第 20 帧调用 GroundPound、重设冷却、播放声音和摇屏；组件自身不补这些动作。原版工厂依赖中声明两种震地 FX；自制生物须在自身 prefab／SG 接好资源与调用，不能只 AddComponent 就获得攻击。[落点与冷却](../../../raw/current/scripts/stategraphs/SGbearger.lua#L1151-L1170) [FX 依赖](../../../raw/current/scripts/prefabs/bearger.lua#L24-L25)
- **扩展回调：** 熊獾 groundpoundFn 用于掉毛，只有宿主参数；增加行为时保留既有回调，且不要误把它当伤害完成回调。[原版回调](../../../raw/current/scripts/prefabs/bearger.lua#L261-L263)

### 控制破坏强度与同次攻击的重复伤害

| 模式 | 选择理由、入口与边界 |
|---|---|
| 分圈控制伤害／工作／弹射 | 伍迪海狸配置仅第 1 圈伤害、3 圈工作、2 圈推船、1 圈弹射；workefficiency=7 是每次 WorkedBy 的工作量，不是伤害倍率。[配置](../../../raw/current/scripts/prefabs/woodie.lua#L818-L830) |
| 有破坏圈但不主动砸毁工作对象 | 远古守护者配置 destructionRings=2，却未设 destroyer 或 workefficiency；这两个值默认 false／nil，不能只看破坏圈数断言会砸树矿。[配置](../../../raw/current/scripts/prefabs/minotaur.lua#L548-L563) [真正门禁](../../../raw/current/scripts/components/groundpounder.lua#L198-L206) |
| 震地与后续 AOE 共用命中记录 | 变异熊獾 SG 新建 targets 表传给 GroundPound，下一帧给自有 DoAOEAttack 复用；需保留该宿主协作协议，单独调用 GroundPound 不附带后续击退／塌陷。[协作](../../../raw/current/scripts/stategraphs/SGbearger.lua#L1381-L1405) |

## 常见陷阱与限制

- **“一个波次肯定只影响一次”**：默认 PointMode 的命中表在每次 DestroyPoints 内新建，跨环可能重复战斗；RingMode 只共享战斗与船只表，工作／点燃／物品弹射仍逐环执行。需要整次战斗去重先启用 RingMode，不把该保证扩展到工作。[旧表](../../../raw/current/scripts/components/groundpounder.lua#L75-L85) [共享表](../../../raw/current/scripts/components/groundpounder.lua#L327-L353) [工作／战斗分流](../../../raw/current/scripts/components/groundpounder.lua#L193-L244)
- **伤害记录不等于实际扣血**：先写 ents_hit，再调用 Combat:DoAttack；后者仍可走 CanHitTarget 失败／onmissother。因此共享表表示已处理攻击尝试，不能当伤害结算证明；玩家友军过滤也只在宿主 `isplayer` 为真时额外启用。[写表时机](../../../raw/current/scripts/components/groundpounder.lua#L224-L233) [Combat 拒绝](../../../raw/current/scripts/components/combat.lua#L1147-L1153)
- **自定义 Combat 门禁被复原覆盖**：结束时固定 EnableAreaDamage(true)、ignorehitrange=false，不保存旧值；与原本禁用范围伤害／忽略距离的其他机制组合时需审查并协调，不能认为组件自动恢复任意旧状态。[入口](../../../raw/current/scripts/components/groundpounder.lua#L173-L175) [出口](../../../raw/current/scripts/components/groundpounder.lua#L292-L294) [EnableAreaDamage 语义](../../../raw/current/scripts/components/combat.lua#L171-L172)
- **只移除组件／退出 SG 就中止波次**：GroundPound 不保存任务句柄，任务闭包仍持有 self；RemoveComponent 只停止组件更新并调用其可选清理，而此组件未实现该钩子。不要运行中临时增删来取消余波；宿主真正移除才由实体生命周期取消其挂载任务。[调度](../../../raw/current/scripts/components/groundpounder.lua#L303-L304) [排队](../../../raw/current/scripts/components/groundpounder.lua#L344-L354) [移除组件](../../../raw/current/scripts/entityscript.lua#L670-L683) [实体任务清理](../../../raw/current/scripts/entityscript.lua#L1746)
- **复用屏外接口**：GroundPound_Offscreen 源码明确标为废弃且未使用，并有“不符合预期”注释；不要以其名义承诺休眠区伤害，新增接入使用已审阅的 GroundPound 链。[警告](../../../raw/current/scripts/components/groundpounder.lua#L365-L368)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 2 处），未运行游戏。

- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖 RingMode／PointMode 分流、环范围与门禁、工作／战斗／弹射／推船／FX、任务与回调时机，以及熊獾 SG、守护者和海狸三种配置；Combat 仅核对范围开关、选武器入口与攻击拒绝，不宣称所有伤害公式已审校。
- 结构检查：本页格式／本地链接及严格源码引用检查通过；本页采用原版组装参照，无独立完整 Lua 示例。
- 游戏运行：未运行；跨环物理表现、不同阵营与武器组合、死亡／移除时的剩余波次未实测。

相关：[[reference/components/combat]]、[[reference/components/workable]]、[[reference/components/inventoryitem]]。
