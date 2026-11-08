---
title: "犁地草膏（tillweedsalve）"
created: "2026-11-10"
updated: "2026-11-11"
type: prefab
summary: "犁地草膏用一次给目标挂 tillweedsalve_buff 持续回复：直接回 10 点、之后每 3 秒回 1 点共 20 点（60 秒）；重吃只刷新不叠加，死亡即停，本 build 注释掉了 perishable——不会腐坏。"
aliases: ["tillweedsalve", "tillweedsalve_buff", "犁地草膏", "荨麻药膏", "Tillweed Salve", "犁地草持续回复"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/tillweedsalve.lua", "prefablist.lua", "components/healer.lua", "components/debuff.lua", "components/debuffable.lua", "components/timer.lua", "components/health.lua", "components/health_replica.lua", "entityscript.lua", "scheduler.lua", "components/perishable.lua", "widgets/itemtile.lua", "standardcomponents.lua", "recipes.lua", "recipes_filter.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 犁地草膏（tillweedsalve）

## 快速判断

- 职责与适用：可拾取、可堆叠的**库存物品**治疗药膏，用一次同时做两件事：经 `healer` 直接回 10 点当前生命，再经 `SetOnHealFn` 给目标挂持续回复 buff（tillweedsalve_buff，本页关联注册、非物品）。物品形态归本页；`healer` 链通用规则从 [[reference/prefabs/healing-salves#healer-链共用规则族级主章节]] 接入，本页差异在[用药流程](#用药流程与恢复数值)。
- 优先使用：查回复节奏与总回复量看[用药流程](#用药流程与恢复数值)；查「为什么不用考虑新鲜度」看[常见陷阱](#常见陷阱与限制)。
- 重要副作用：buff 每 3 秒 `health:DoDelta(1, nil, "tillweedsalve")`，死亡／幽灵立即停；重吃同名 buff 只刷新时长与任务，不叠加速率。[OnTick](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L57-L65)
- 修改边界：不展开 `debuff`／`timer` 组件接口语义（归 [[reference/components/debuff]]／[[reference/components/timer]]）与 health 结算细节；物品侧田地产出（犁地草）归 [[reference/prefabs/weeds]]。

## 注册与构造

[注册行](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L121-L122)；[prefablist 登记](../../../raw/current/scripts/prefablist.lua#L1282)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `tillweedsalve`（犁地草膏） | `fn`：bank/build `tillweedsalve`、pristine 前 `AddTag("show_spoilage")` → 主模拟挂 `stackable`(40)／`inspectable`／`inventoryitem`／`healer`：`SetHealthAmount(10)`＋`SetOnHealFn(OnUse)` → `MakeHauntableLaunch` | 无 `healerbuffs` 标签；治疗量 10 是字面值，不走 TUNING 治疗档。[healer 接线](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L42-L44)、[OnUse](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L8-L10)、[堆叠](../../../raw/current/scripts/tuning.lua#L83)、[作祟 helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| `tillweedsalve_buff` | `buff_fn`：客户端生成即删；服务端 `Hide`＋`CLASSIFIED`＋`persists=false`＋`debuff` 三回调（`keepondespawn=true`）＋`timer`（启动 `regenover` 计时器） | 非物品、玩家不可见、无 NAMES 条目。[buff 实体](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L89-L118)、[计时器](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L114-L116) |

- 名称核验：简中实显「犁地草膏」（`STRINGS.NAMES.TILLWEEDSALVE`）；「荨麻药膏」是派发工作题名（tillweed 是犁地草不是荨麻，[[reference/prefabs/weeds]] 口径），仅作检索别名保留。[po](../../../raw/current/scripts/languages/chinese_s.po#L350410-L350412)

## 用药流程与恢复数值

```text
对目标用犁地草膏（ACTIONS.HEAL）
  ├─ Healer:Heal → health.canheal 时 DoDelta(10)         ← 直接部分
  ├─ OnHealFn：target:AddDebuff("tillweedsalve_buff", …) ← 新建或 Extend
  │    ├─ 新建：SpawnPrefab → buff_fn 起计时器 → AttachTo → OnAttached 起 3 秒周期任务
  │    └─ 已有：OnExtended → 计时器与周期任务都重置为满
  └─ 拆一件消耗（堆叠取末件，否则删除自身）
恢复节奏：t=3,6,…,60 共 20 次 × 1 点 = 20 点；计时器 60+1/30 秒后 Stop
```

| 环节 | 实际行为 | 证据 |
|---|---|---|
| 直接回复 | 10 点，乘执行者 `efficientuser` 的 HEAL 倍率；目标 `canheal=false` 时跳过本步（但本物品无 `healerbuffs` 标签，禁疗目标连动作都不会出现） | [healer 乘法与门禁](../../../raw/current/scripts/components/healer.lua#L36-L44)、[候选](../../../raw/current/scripts/componentactions.lua#L1346-L1362) |
| 周期回复 | `DoPeriodicTask(3, OnTick)`，`initialdelay=nil` 按 period 起拍 → 首次第 3 秒；每跳 `DoDelta(1, nil, "tillweedsalve")` | [OnAttached](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L70)、[起拍语义](../../../raw/current/scripts/scheduler.lua#L295-L297)、[常量](../../../raw/current/scripts/tuning.lua#L5833-L5834) |
| 时长 | `BUFF_DURATION = TILLWEEDSALVE_DURATION + FRAMES`＝60＋1/30 秒；多出的 1 帧保证最后一跳先于计时器到期（源码注释里「27/28 hp」是旧数值，按当前常量满程 20 点） | [L56](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L56)、[常量](../../../raw/current/scripts/tuning.lua#L5835)、[FRAMES](../../../raw/current/scripts/constants.lua#L16)、[seg_time](../../../raw/current/scripts/tuning.lua#L19-L20) |
| 提前停止 | 目标死亡（事件监听）或 OnTick 发现已死／幽灵 → `debuff:Stop()`；计时器 `regenover` 到期 → 同样 Stop | [death 监听](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L71-L73)、[OnTick 守卫](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L58-L64)、[timerdone](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L76-L80) |
| 重吃刷新 | `OnExtended` 先 `StopTimer` 再满时长重启，周期任务取消重建——不叠加，只续满 20 跳 | [OnExtended](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L82-L87) |
| 挂接门禁 | `AddDebuff` 自动补 `debuffable`，仅在 `DebuffsEnabled()` 且未死非幽灵时挂上 | [AddDebuff](../../../raw/current/scripts/entityscript.lua#L2140-L2154) |

- 总回复＝10（直接）＋20（持续）＝30 点；直接部分可被沃尔特技能倍率放大到 15，周期部分不受倍率影响（不走 healer）。[倍率接线](../../../raw/current/scripts/prefabs/skilltree_walter.lua#L409-L414)
- 存档：buff 实体不入档（`persists=false`），`debuffable` 把 `GetSaveRecord` 嵌进宿主存档，读档 `SpawnSaveRecord`（走 OnPreLoad→组件 OnLoad→实体 OnLoad 顺序）重生成后重新 attach；周期任务按 OnAttached 新建，剩余时间由 `timer` 组件的 `timeleft` 存取还原——所以读档后「还剩多久」是对的、下一跳回到 3 秒节拍。[debuffable 存取](../../../raw/current/scripts/components/debuffable.lua#L126-L137)、[L139-L151](../../../raw/current/scripts/components/debuffable.lua#L139-L150)、[载入顺序](../../../raw/current/scripts/entityscript.lua#L1988-L2013)、[timer 存取](../../../raw/current/scripts/components/timer.lua#L122-L142)

## 获得与消耗

- 配方：犁地草 1＋花瓣 1＋木炭 1，`TECH.SCIENCE_ONE`；登记在合成栏「恢复」（RESTORATION）分类。犁地草的产出归 [[reference/prefabs/weeds]]。[配方](../../../raw/current/scripts/recipes.lua#L418)、[分类](../../../raw/current/scripts/recipes_filter.lua#L1144-L1160)
- 全库按注册名检索无 `SpawnPrefab` 生成点与掉落表登记（范围同 [[reference/prefabs/healing-salves#验证记录]]）；消耗侧无其他配方引用。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`healer` | 直接 10 点＋OnHealFn 挂 buff | 仅主模拟；语义归组件页 | [[reference/components/healer#关键接口]]；[接线](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L42-L44) |
| 组件：`debuff`／`debuffable` | buff 三回调、`keepondespawn=true`、嵌套存档 | buff 仅服务端、非联网 | [[reference/components/debuff#关键接口]]、[[reference/components/debuffable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L108-L112) |
| 组件：`timer` | 计时器 `regenover` 控制总时长，`timerdone` 事件停 buff | buff 实体；`timeleft` 可存档 | [[reference/components/timer#关键接口]]；[启动](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L114-L116) |
| 组件：`health` | `DoDelta(1, nil, "tillweedsalve")`；死亡即停 | 目标需有 `health` | [[reference/components/health#关键接口]]；[OnTick](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L57-L65) |
| 数据：`TUNING.TILLWEEDSALVE_*` | 回复量 1／周期 3 秒／时长 60 秒 | 改常量即改节奏 | [常量](../../../raw/current/scripts/tuning.lua#L5833-L5835) |
| 标签：`show_spoilage` | pristine 前添加，但本件无 `perishable` 组件 | 见[陷阱](#常见陷阱与限制)；物品栏 UI 按标签读取（见 itemtile 判定） | [L27](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L27) |
| 资源：`tillweedsalve.zip` | 物品动画 | 只核验声明 | [assets](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L1-L4) |

## 常见陷阱与限制

- **误区：犁地草膏会腐坏**：perishable 整段（组件、时长、`StartPerishing`、腐坏替换）在本 build 被**注释掉**——`perishable` 组件的周期任务只在 `StartPerishing()` 里建，此处没有执行，物品没有新鲜度、不会变 `spoiled_food`。若在旧版本或未来版本看到该段未注释，本页结论需按新 build 复核。[被注释块](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L46-L49)、[StartPerishing](../../../raw/current/scripts/components/perishable.lua#L262-L273)
- **`show_spoilage` 标签是死的**：物品栏新鲜度条要求 `fresh`／`stale`／`spoiled` 标签之一，而这三个标签由 `perishable` 组件维护；本件没有该组件，永远凑不齐条件，标签不产生任何可见效果。别把它当「有新鲜度」的证据。[显示判定](../../../raw/current/scripts/widgets/itemtile.lua#L732-L749)、[标签维护](../../../raw/current/scripts/components/perishable.lua#L1-L20)
- **OnTick 不查 `canheal`**：周期回复直接 `DoDelta`，绕过 healer 的 `health.canheal` 门禁；但动作候选已把禁疗目标挡在门外（本件无 `healerbuffs`），原版触不到这条路径——mod 里放开候选时要补这个差异。[OnTick](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L57-L65)、[候选门禁](../../../raw/current/scripts/componentactions.lua#L1346-L1362)
- **重吃只刷新不叠加**：同名 debuff 走 `Extend`，计时器与周期任务重置为满，不存在双倍速率；想要叠加效果须换 buff 键名。[OnExtended](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L82-L87)、[Extend 分派](../../../raw/current/scripts/components/debuffable.lua#L95-L107)
- **源码注释数值过期**：L6 注释「heals 10 + 20 overtime」与当前常量一致，可采信；L56 注释里的「27 instead of 28 hp」与当前 20 点不符，是历史数值，以常量计算为准。[L56](../../../raw/current/scripts/prefabs/tillweedsalve.lua#L56)
- **作祟只抛飞**：`MakeHauntableLaunch` 无打碎／点燃分支。[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。核对 `tillweedsalve.lua` 全文件（物品工厂、buff 工厂、两行注册）、`healer.lua` 全文件、`debuff.lua` AttachTo、`debuffable.lua` AddDebuff／RegisterDebuff／存取段、`timer.lua` 计时与存取段、`health.lua` canheal 字段、`health_replica.lua` CanHeal、`entityscript.lua` AddDebuff／载入顺序段、`scheduler.lua` 周期起拍段、`perishable.lua` 标签维护与 StartPerishing 段、`itemtile.lua` 新鲜度显示判定、`standardcomponents.lua` 作祟 helper、`recipes.lua` 配方、`recipes_filter.lua` 分类、`tuning.lua`／`constants.lua` 常量、`prefablist.lua` 登记。
  名称核验：简中名按 `chinese_s.po` 的 `STRINGS.NAMES.TILLWEEDSALVE` msgctxt 实显核对（犁地草膏）；`tillweedsalve_buff` 无 NAMES 条目（非物品）。
  生成点否定断言范围：全 `scripts` 目录字面检索 `"tillweedsalve"`（排除本页文件、语言、配方、图鉴、登记表、分类表后无消费点）；动态拼名未检索。
  未覆盖：未实测读档节拍（「下一跳回到 3 秒」由 timer 存取与 OnAttached 代码推得）、未实测 `keepondespawn` 与跨裂隙搬运组合、未审校皮肤。
- 游戏运行：未运行。20 点满程与 60 秒时长由常量推导（20 跳 × 1 点）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 待主代理验收时统一运行（见交付回报）。

## 相关链接

- healer 链与同族药膏：[[reference/prefabs/healing-salves]]
- 组件语义：[[reference/components/healer]]、[[reference/components/debuff]]、[[reference/components/debuffable]]、[[reference/components/timer]]、[[reference/components/health]]
- 材料来源：[[reference/prefabs/weeds]]

返回 [[reference/prefabs/index|Prefab 索引]]。
