---
title: "南瓜灯（pumpkin_lantern）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
summary: "南瓜＋萤火虫合成的可拾取灯具 pumpkin_lantern：非白天（黄昏与夜晚）触发淡入、入昼延迟约 2 秒淡出、进背包立刻灭并暂停腐坏、落地恢复腐坏，平时 10 天／万圣节 40 天，腐坏或被打死后掉一只萤火虫。"
aliases: ["pumpkin_lantern", "南瓜灯", "南瓜灯笼"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/pumpkin_lantern.lua", "recipes.lua", "map/boons.lua", "map/layouts.lua", "map/rooms/forest/pigs.lua", "components/perishable.lua", "components/inventoryitem.lua", "components/health.lua", "components/lootdropper.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "actions.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 南瓜灯（pumpkin_lantern）

## 快速判断

- 职责与适用：`pumpkin_lantern` 是可拾取物品（`item`）而不是建筑；它靠自己的 `Light` 在夜间照明，白天或被拿起来就熄灭。玩法上是一条“落地计时→腐坏或被打死→掉一只萤火虫”的消耗链。[工厂](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L192-L262)
- 优先使用：配方与获得查[名称与获得](#名称与获得)；天数与鲜度查[数值与结算](#数值与结算)；昼夜与发光条件查[昼夜淡入淡出](#昼夜淡入淡出)；进包／落地行为查[落地亮灯与进包暂停](#落地亮灯与进包暂停)。
- 重要副作用：`health` 上限只有 1，死亡时 `lootdropper` 生成一只 `fireflies` 并把动画换成 `broken`（腐坏路径先播 `rotten`，并在死亡回调里跳过 `broken`）；进背包会 `StopPerishing`，**只有落在地面才继续腐坏**；作祟只会把它弹飞，不加速腐坏。[死亡与腐坏回调](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L113-L130)、[入包](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L169-L176)、[作祟](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337)
- 修改边界：萤火虫本体、燃料与光照归其自身注册（Wiki 待建）；提灯的开关与容量归 [[reference/prefabs/lantern]]；本页只写南瓜灯的配方、昼夜、鲜度与死亡掉落。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `pumpkin_lantern` | 单工厂、无变体：实体＋Transform／AnimState／SoundEmitter／**AddLight**／Network → 标签 `veggie`／`light` → `MakeInventoryPhysics` → 光源参数与 `Light:Enable(false)` → bank `pumpkin`、build `pumpkin_lantern`、播 `idle_day` → 网络量 `_fade`（初值 11＝熄灭）→ `MakeInventoryFloatable("med", 0.1, 0.78)` → `SetPristine` → 客户端监听 `fadedirty` 后返回 → 主模拟挂组件 | `Light` 是实体侧接口；客户端靠 `_fade` 同步亮度包络，主模拟才真正 `Light:Enable`；`scrapbook_anim` 固定 `idle_day`。[工厂前半](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L192-L231)、[主模拟段](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L233-L261) |

光源参数：falloff 0.5、intensity 0.8、radius 1.5、颜色 `(200, 100, 170)/255`，并开了 `EnableClientModulation(true)` 让客户端按 `_fade` 自己算亮度。[光源参数](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L206-L211)

### 名称与获得

简中名 `STRINGS.NAMES.PUMPKIN_LANTERN`＝“南瓜灯”（S2）。[NAMES.PUMPKIN_LANTERN](../../../raw/current/scripts/languages/chinese_s.po#L347350-L347352)

主配方（S4）：`pumpkin` 1＋`fireflies` 1，科技档 `TECH.SCIENCE_ONE`。[配方行](../../../raw/current/scripts/recipes.lua#L430)

另有三条不靠合成台的获得路径，但都只在万圣节活动期间出现：猪人村落布点、起始礼物（`welcomitem`）、世界生成小礼物点（boon 的物品区）。[起始礼物](../../../raw/current/scripts/map/layouts.lua#L387)、[战利品组合](../../../raw/current/scripts/map/boons.lua#L30)、[猪村布点](../../../raw/current/scripts/map/rooms/forest/pigs.lua#L12)

## 数值与结算

时间基准：`total_day_time`＝480 秒，鲜度天数按 480 秒/天折算（S1）。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 项 | 定义处 | 结算处与实际值 |
|---|---|---|
| 鲜度 | `SetPerishTime(万圣节 ? PERISH_SUPERSLOW : PERISH_MED)`＋`SetOnPerishFn(onperish)`＋`StartPerishing()` [perishable 定义](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L244-L247) | 定义值：平时 10 天＝4800 秒，`IsSpecialEventActive(SPECIAL_EVENTS.HALLOWED_NIGHTS)` 为真时换 40 天＝19200 秒。腐坏只在世界地面流逝，而地面分支另乘 `PERISH_GROUND_MULT`＝1.5，所以地面实际约 6.67 天＝3200 秒（万圣节约 26.67 天＝12800 秒）；进背包时计时已被暂停，不涉及该倍率 [两个常量](../../../raw/current/scripts/tuning.lua#L2048-L2051)、[事件分支](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L244-L247)、[地面倍率](../../../raw/current/scripts/tuning.lua#L2083-L2083) |
| 生命 | `health:SetMaxHealth(1)`、`canmurder=false`，并挂 `combat` [生命与战斗](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L237-L242) | 上限 1 点；不能被“杀死”动作处理，但会吃伤害；归零触发 `death` [health 组件](../../../raw/current/scripts/components/health.lua#L503-L546) |
| 腐坏结果 | `onperish` 只做两件事：播 `rotten`、`health:Kill()` [onperish](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L127-L130) | 没有 `onperishreplacement`，所以不会变成腐烂物，而是走死亡链 [无替换则只回调](../../../raw/current/scripts/components/perishable.lua#L277-L320) |
| 死亡掉落 | `lootdropper:SpawnLootPrefab("fireflies")` [ondeath](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L113-L125) | 固定 1 只萤火虫，动画换 `broken` 并播 `vegi_smash`；腐坏路径因为当前动画已是 `rotten` 而跳过换动画 [动画条件](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L121-L124) |
| 落地淡入 | `FADE_INTENSITY`／`FADE_RADIUS`／`FADE_FALLOFF`／`FADE_FRAMES` | 0.8／1.5／0.5／5 帧；闪烁把半径在 1.5–1.6 间抖动 [常量与闪烁](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L11-L21) |

数值口径：`_fade` 的取值 0–11 是“亮度包络”，5 是满亮，11 是熄灭端点；主模拟在 `0 < _fade <= 10` 时 `Light:Enable(true)`，所以 11 已经落在开灯区间之外。[开关条件](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L37-L39)、[包络推进](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L23-L54)

## 昼夜淡入淡出

`WatchWorldState("isday", OnIsDay)` 是昼夜切换的主入口（构造结束时还有一次首帧点亮）：白天到达时先延迟 `2 + math.random()` 秒再 `FadeOut`，进入 `not isday` 时同样延迟后 `FadeIn`；延迟由 `_daytask` 承担，重复触发会取消旧任务。注意 `not isday` 同时覆盖黄昏与夜晚，所以黄昏就会亮；天亮后也不会立刻灭，仍会亮 `2 + math.random()` 秒才开始淡出。[OnIsDay](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L136-L152)、[接线与首帧](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L252-L255)

发光条件被 `CanFade` 收口：只要“被持有”或“已死”就不做淡入淡出，所以**只有落在地面、没死、且处于 `not isday`（黄昏或夜晚）**的南瓜灯才会进入淡入；入昼后要先等 `2 + math.random()` 秒才淡出。[CanFade](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L132-L134)、[昼夜延迟](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L136-L152)

淡入淡出是逐帧推进的：`_fade` 是联网量，变化时 `OnFadeDirty` 起一个 `FRAMES` 周期的任务，`OnUpdateFade` 每帧把 `_fade` 往 5 推（淡入）或往 11 推（淡出），同时按比例设 intensity／radius／falloff；到 5 或超过 10 时停任务。淡入到位后另起 0.1 秒的闪烁任务。[包络与 dirty](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L23-L61)

动画与光同步：淡入用 `idle_night_pre` → `idle_night_loop`，淡出用 `idle_night_pst` → `idle_day`；读档或立即切换时直接跳 `idle_night_loop`／`idle_day`。[两条路径](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L63-L111)

## 落地亮灯与进包暂停

`inventoryitem` 的两个回调是这条链的开关（S5）：`OnPutInInventory` 先 `StopPerishing()`、取消昼夜任务、再 `FadeOut(inst, true)` 立即熄灭；`OnDropped` 反过来——没死就 `StartPerishing()`，取消旧任务，然后按“非白天且可淡入”决定 `FadeIn` 还是立即 `FadeOut`。[两个回调](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L154-L176)

结果是南瓜灯的鲜度只在世界地面上流逝：拿在手上、放进背包或箱子都停表；重新落地又从原剩余值继续。这条规则与多数可腐坏物品相反——它们放进背包后仍照常计时，是本页最容易读反的一处。[停表](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L169-L176)、[恢复](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L154-L157)

读档只处理动画与光：`OnLoad` 取消旧任务后按当前 `isday` 直接 `FadeOut(inst, true)` 或 `FadeIn(inst, true)`，不重设 `_fade` 之外的状态。[OnLoad](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L178-L190)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：perishable | 10／40 天两档，`onperish` 只回调不替换；进包停表、落地恢复 | 仅主模拟 | [[reference/components/perishable]]；[接线](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L244-L247) |
| 组件：health／combat | 上限 1、`canmurder=false`，死亡触发 `ondeath` | 仅主模拟；吃伤害但不可“杀死” | [接线](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L237-L242) |
| 组件：lootdropper | 死亡时生成 `fireflies` | 仅主模拟 | [[reference/components/lootdropper]]；[掉落](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L120) |
| 组件：inventoryitem | 落地／入包两个回调驱动计时与光照；`nobounce=true` | 回调只在主模拟设置 | [[reference/components/inventoryitem]]；[接线](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L234-L250) |
| 组件：inspectable | 常规查看 | 仅主模拟 | [[reference/components/inspectable]]；[挂载](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L233) |
| 网络：`_fade`（net_smallbyte） | 客户端按 `fadedirty` 重算光与动画 | 客户端只监听该量 | [注册](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L217-L218)、[客户端分支](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L227-L231) |
| 关联 Prefab：`fireflies` | 配方材料与死亡掉落 | 掉落一次 | Wiki 待建；[依赖声明](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L6-L9) |
| 标签：`veggie`／`light` | 参与蔬菜与光照类判定 | 两端都加 | [标签](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L201-L202) |
| 动作：DROP 的 `PLACELANTERN` | 放下时动作名显示为“放置”，但没有 `deployable`，实际仍是普通掉落 | 仅字符串分支 | [字符串分支](../../../raw/current/scripts/actions.lua#L1192-L1198) |
| 资源：`anim/pumpkin_lantern.zip` | 昼夜两套动画，bank 复用的是 `pumpkin` | 原版资源，不随页分发 | [资产](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L1-L4) |

## 典型接入

### 把南瓜灯的保质期统一改成 20 天

- 前提与选择：服务端 `modmain.lua` 的 postinit。构造时已经按活动选过 10／40 天并启动计时，因此用 `SetNewMaxPerishTime` 改上限可以保留当前新鲜度比例；直接写 `SetPerishTime` 会把剩余时间一起补满。[原值](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L244-L247)、[两个接口的差别](../../../raw/current/scripts/components/perishable.lua#L188-L215)
- 生效与撤销：只影响新生成的南瓜灯；已在存档里的实例在重新加载并构造时同样会跑到本 postinit，因此也会被改写。单位是秒，`TUNING.TOTAL_DAY_TIME`＝480。[TOTAL_DAY_TIME](../../../raw/current/scripts/tuning.lua#L63)
- 原版依据：`perishable` 在“已经启动计时”的情况下也能改上限，`SetNewMaxPerishTime` 会按旧百分比换算剩余时间。[保留比例](../../../raw/current/scripts/components/perishable.lua#L204-L215)

```lua
-- modmain.lua：南瓜灯统一 20 天保质（不再区分万圣节 40 天）
AddPrefabPostInit("pumpkin_lantern", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local perishable = inst.components.perishable
    if perishable ~= nil then
        perishable:SetNewMaxPerishTime(20 * GLOBAL.TUNING.TOTAL_DAY_TIME)
    end
end)
```

- 调整与验证：验证时看新生成南瓜灯的 `perishable.perishtime` 是否为 9600 秒，并确认进背包仍然停表、落地继续倒数。

## 常见陷阱与限制

- **误区：南瓜灯是建筑或可燃灯。** 它挂的是 `inspectable`／`inventoryitem`／`combat`／`health`／`lootdropper`／`perishable`／`hauntable` 与引擎侧 `Light`，但没有 `burnable`、没有 `deployable`、也不是 placer；放下的“放置”只是 DROP 动作的显示字符串。[无燃烧与部署件](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L192-L262)、[显示字符串](../../../raw/current/scripts/actions.lua#L1192-L1198)
- **边界：只有地面上的南瓜灯会腐坏。** `OnPutInInventory` 明确 `StopPerishing`，所以背包、箱子里的南瓜灯不会烂；这条与多数食物相反。[停表](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L169-L176)
- **边界：非白天触发淡入，入昼延迟淡出；手持不亮。** `CanFade` 把“被持有”和“已死”排除；昼夜切换走 `2 + math.random()` 秒的延迟任务，所以天亮后还会亮一小段才淡出。进包时不等延迟，立即 `FadeOut(inst, true)`。[条件](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L132-L176)
- **边界：腐坏不会变成腐烂物。** `onperish` 只播 `rotten` 再 `health:Kill()`；因为没有 `onperishreplacement`，最终留下的是“死掉的南瓜灯”而不是替换物，萤火虫在同一 `ondeath` 里掉出。[两条回调](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L113-L130)
- **边界：作祟只会弹飞。** `MakeHauntableLaunchAndPerish` 的“作祟加速腐坏”分支在本版整段被注释，只剩 `Launch`。[注释分支](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337)、[挂载](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L259)
- **边界：读档分支有拼写错误。** `OnLoad` 里写的是 `inst._daytask:cancel()`（其余位置都是 `:Cancel()`）；正常存档不写 `_daytask`，所以该分支通常不执行，一旦外部置入该字段会直接报错（未运行验证）。[小写 cancel](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L178-L186)
- **未核验**：昼夜切换的 2 秒级延迟观感、万圣节活动判定与游戏内掉落（S7）。

## 验证记录

- 源码审阅：2026-11-09，steam-buildid:26474873（`raw/current/BUILD.txt` 实读）。`pumpkin_lantern.lua` 全文核对光源参数、`_fade` 包络、昼夜任务、两个 `inventoryitem` 回调、死亡与腐坏两条回调、组件与存读；并核 `perishable` 的 `SetNewMaxPerishTime` 与无替换物分支、`lootdropper:SpawnLootPrefab`、`recipes.lua` 配方行、`map/` 三处世界生成布点、`actions.lua` 的 `PLACELANTERN` 字符串与 `standardcomponents` 的作祟 helper。
- 游戏运行：未运行；昼夜与腐坏均为源码审阅（S7）。未覆盖：多人下光包络的同步、萤火虫掉落的拾取时序。
- 检视修订（2026-11-09，同一 build）：①三条非合成获得路径统一加“仅万圣节活动期间”，`welcomitem` 按源码拼写（`map/layouts.lua` L387、`map/boons.lua` L26-L30、`map/rooms/forest/pigs.lua` L12）；②`_fade` 口径改为“5＝满亮、11＝熄灭端点，开灯区间为 0<`_fade`≤10”；③“与多数可食装备相反”改为“与多数可腐坏物品相反”（南瓜灯本身不可食、非装备）；④“它只有…”的组件清单改为实际挂载集合（补 `combat`／`lootdropper`／`perishable`／`hauntable`），并写明没有 `burnable`／`deployable`／placer；⑤“腐烂食物”“实战”改为规范措辞；⑥`快速判断`与陷阱补“作祟只弹飞”（`MakeHauntableLaunchAndPerish` 的 perish 分支已注释）；⑦`sources` 去掉无对应断言的 `prefabs/fireflies.lua` 与 `components/stackable.lua`。
- 复验修订（2026-11-10）：“不会变成腐烂食物”改为“不会变成腐烂物”，与陷阱段用词统一。
- 外部检视修订（2026-11-10，W05 报告 R09）：①昼夜口径改为 `not isday`（含黄昏与夜晚），并写明天亮后仍先亮 `2 + math.random()` 秒才淡出；②发光条件与陷阱同步该延迟（`pumpkin_lantern.lua` L132-L166）。两条死亡动画维持原表述：受击路径播 `broken`、腐坏路径先播 `rotten` 并在死亡回调跳过 `broken`。复验补充：摘要改为“非白天触发淡入、入昼延迟淡出”，快速判断与陷阱同步“腐坏路径跳过 `broken`”与入昼延迟口径。
- 结构检查：`validate_page.py` 通过；`validate_source_links.py --strict` 0 error／0 warning。本页 Lua 示例为完整片段，未在游戏内执行。

## 相关链接

- 提灯／矿工帽的照明与燃料：[[reference/prefabs/lantern]]
- 鲜度与替换：[[reference/components/perishable]]
- 同批的冰物品：[[reference/prefabs/ice]]

返回 [[reference/prefabs/index|Prefab 索引]]。
