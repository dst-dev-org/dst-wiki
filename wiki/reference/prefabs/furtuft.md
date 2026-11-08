---
title: "毛丛（furtuft）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "熊獾掉落的毛丛：活体每 45 秒掉一撮、震地 20% 概率掉 3 撮，尸体结算可把熊皮按剩余肉量折算成毛丛；计时 25–35 秒后进入消失序列；40 叠、45 秒燃料，90 撮换 3 张熊皮。"
aliases: ["furtuft", "毛丛", "Fur Tuft", "熊獾毛", "熊毛"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/furtuft.lua", "prefabs/bearger.lua", "brains/beargerbrain.lua", "behaviours/beargeroffscreen.lua", "components/shedder.lua", "components/disappears.lua", "components/fuel.lua", "components/fueled.lua", "components/lootdropper.lua", "prefabs/corpses.lua", "prefabs/corpses_defs.lua", "standardcomponents.lua", "recipes.lua", "tuning.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 毛丛（furtuft）

## 快速判断

- 职责与适用：`prefabs/furtuft.lua` 直接注册的单一物品（无变体、无皮肤），来源是熊獾的两条链——活体周期掉毛与尸体结算；资源用的是 `anim/bearger_tuft.zip`，与浣猫的猫尾（[[reference/prefabs/coontail]]）**不同源、不同表**。[注册](../../../raw/current/scripts/prefabs/furtuft.lua#L61)、[资源](../../../raw/current/scripts/prefabs/furtuft.lua#L1-L4)
- 优先使用：查怎么获得看[来源：熊獾掉毛](#来源熊獾掉毛)；查它能做什么看[配方与燃料](#配方与燃料)；查"为什么地上的毛丛没了"看[常见陷阱与限制](#常见陷阱与限制)。
- 重要副作用：生成时即 `PrepareDisappear`，从背包丢出（`ondropped`）会重排；`delay` 25 秒再加 0–10 秒随机，**计时到点先进入消失序列**——醒着时禁拾取、播 `disappear` 后删，休眠时直接删；进背包停表。[计时](../../../raw/current/scripts/components/disappears.lua#L3)、[排期](../../../raw/current/scripts/components/disappears.lua#L68-L77)、[消失](../../../raw/current/scripts/components/disappears.lua#L12-L49)
- 修改边界：熊獾本体（冬眠、震地、尸体）只写"产出毛丛"的必要分支；熊皮与熊獾完整行为归各自页面，见[关联机制导航](#关联机制导航)。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `furtuft` | 无包装工厂：`fn()` 直接构造；公共端（Transform／AnimState／SoundEmitter／Network／物理／动画／floater）→ `SetPristine` → 非主模拟返回 → 主模拟挂件 | 单注册、无皮肤、无 placer。[工厂](../../../raw/current/scripts/prefabs/furtuft.lua#L14-L59) |

- 主模拟挂件顺序：`disappears`（音效 `dust_blowaway`、动画 `disappear`）→ `inspectable` → `inventoryitem`（注册 `SetOnPutInInventoryFn`）→ `stackable`（`TUNING.STACK_SIZE_SMALLITEM`＝40）→ `fuel`（`TUNING.MED_FUEL`）→ 监听 `ondropped` 并立刻 `PrepareDisappear` → `MakeSmallBurnable(MED_BURNTIME)` → `MakeSmallPropagator` → `MakeHauntableLaunchAndIgnite`。[挂件](../../../raw/current/scripts/prefabs/furtuft.lua#L36-L56)
- **40 叠、45 秒燃料**：堆叠显式写 40；燃料值取 `seg_time*1.5`＝45 秒（`seg_time`＝30 秒）。[堆叠](../../../raw/current/scripts/prefabs/furtuft.lua#L44-L45)、[常量](../../../raw/current/scripts/tuning.lua#L81-L83)、[燃料常量](../../../raw/current/scripts/tuning.lua#L1673-L1678)、[图鉴](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L568)
- **可烧 15 秒**：`MakeSmallBurnable(inst, TUNING.MED_BURNTIME)`，`MED_BURNTIME`＝`seg_time*0.5`＝15 秒；`MakeSmallPropagator` 让它带点燃邻近物体的热扩散。[燃烧](../../../raw/current/scripts/prefabs/furtuft.lua#L53-L54)、[常量](../../../raw/current/scripts/tuning.lua#L1682)、[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L174)
- 作祟走 `MakeHauntableLaunchAndIgnite`，但本 build 里点燃分支整段被注释（`#HAUNTFIX`），实际只抛飞。[helper](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)

### 名称核验

`STRINGS.NAMES.FURTUFT`＝**毛丛**（Fur Tuft）；本文件不改 prefab 名、不用 nameoverride（S1）。[.po](../../../raw/current/scripts/languages/chinese_s.po#L343084-L343087)

## 来源：熊獾掉毛

熊獾与装甲熊獾都挂 `shedder`：`shedItemPrefab = "furtuft"`、`shedHeight = 6.5`，以 `TUNING.BEARGER_SHED_INTERVAL`＝45 秒启动。[挂接](../../../raw/current/scripts/prefabs/bearger.lua#L487-L490)、[常量](../../../raw/current/scripts/tuning.lua#L2662)、[两注册](../../../raw/current/scripts/prefabs/bearger.lua#L844-L846)

| 触发 | 产出与判定 | 条件／证据 |
|---|---|---|
| 周期掉毛（shedder） | 每 45 秒 1 件，落在熊獾位置 ±0.5 格、高度 6.5 处再落下 | `DoPeriodicTask(interval or 60)` 循环调用；`StartShedding` 先取消旧任务。[启动](../../../raw/current/scripts/components/shedder.lua#L12-L17)、[单次](../../../raw/current/scripts/components/shedder.lua#L26-L33) |
| 震地多掉 | `TUNING.BEARGER_MULTISHED_CHANCE`＝0.2 命中时 `DoMultiShed(3, false)`，**固定 3 件**（不是随机 1–3） | 震地回调里先掷 `math.random() <= 0.2`。[回调](../../../raw/current/scripts/prefabs/bearger.lua#L261-L265)、[常量](../../../raw/current/scripts/tuning.lua#L9349)、[多件](../../../raw/current/scripts/components/shedder.lua#L35-L45) |
| 冬眠 | 入睡时 `StopShedding`；醒来按 45 秒重新起表 | `ShouldSleep`／`ShouldWake`。[睡眠](../../../raw/current/scripts/prefabs/bearger.lua#L282-L307) |
| 死亡 | `OnDead` 只停掉周期掉毛任务；尸体产出走下一行的换算链 | [死亡](../../../raw/current/scripts/prefabs/bearger.lua#L335-L339) |
| 熊皮尸体换算（多件表） | 仅当 `meat_level >= 2` **且原战利品表有多件**、其中含 `bearger_fur` 时：基数 30 件／张，再按 `GetMeatPercent()` 向上取整（`ceil(30 × meat_perc)`，至少 1）；单件分支只掷是否清空，不做换算 | 结算时 `OverrideCorpseLoot` 把 `bearger_fur` 换成 `furtuft`。[覆写表](../../../raw/current/scripts/prefabs/corpses_defs.lua#L1467-L1472)、[门禁与数量](../../../raw/current/scripts/prefabs/corpses.lua#L465-L493)、[肉量](../../../raw/current/scripts/prefabs/corpses.lua#L208-L210) |

- **熊獾击杀表本身不掉毛丛**：普通熊獾的共享掉落表是 8 meat＋`bearger_fur`＋雕塑草图，没有 `furtuft`；尸体换算另计（见上表），装甲熊獾的表里也没有。[普通表](../../../raw/current/scripts/prefabs/bearger.lua#L76-L88)、[装甲表](../../../raw/current/scripts/prefabs/bearger.lua#L90-L99)
- **"离屏拖尾"是死代码**：`behaviours/beargeroffscreen.lua` 里有 `traildrop = "furtuft"` 的沿路掉落与 `Roam`，但 `beargerbrain.lua` 的 `require` 被注释掉，本 build 没有接线。[常量](../../../raw/current/scripts/behaviours/beargeroffscreen.lua#L6)、[掉落](../../../raw/current/scripts/behaviours/beargeroffscreen.lua#L91-L107)、[调用点](../../../raw/current/scripts/behaviours/beargeroffscreen.lua#L166-L179)、[注释的 require](../../../raw/current/scripts/brains/beargerbrain.lua#L8)

## 配方与燃料

| 环节 | 数值与规则 | 证据 |
|---|---|---|
| 配方 | `bearger_fur` 需 **90 furtuft**，一次产出 **3 张熊皮**（`numtogive = 3`）→ 折算 30 撮／张（推导） | `TECH.SCIENCE_TWO`。[配方](../../../raw/current/scripts/recipes.lua#L470)、[熊皮名](../../../raw/current/scripts/languages/chinese_s.po#L339229-L339232) |
| 烧火 | 燃料值 45 秒；结算时乘 `bonusmult × multfn × wetmult × masterymult` | 潮湿时 `wetmult` 取 `WET_FUEL_PENALTY`＝0.75，即 45×0.75＝33.75 秒（推导）；其余默认 1。[公式](../../../raw/current/scripts/components/fueled.lua#L197-L212)、[常量](../../../raw/current/scripts/tuning.lua#L1671) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`disappears` | `delay` 25、音效与消失动画；`ondropped` 重排、进背包停表 | 仅主模拟；地面 | [[reference/components/disappears]]；[挂接](../../../raw/current/scripts/prefabs/furtuft.lua#L36-L38)、[回调](../../../raw/current/scripts/prefabs/furtuft.lua#L6-L12) |
| 组件：`fuel` | `fuelvalue = TUNING.MED_FUEL`，默认 `fueltype = BURNABLE` | 主模拟 | [[reference/components/fuel]]；[挂接](../../../raw/current/scripts/prefabs/furtuft.lua#L47-L48) |
| 组件：`burnable`／`propagator` | `MakeSmallBurnable(MED_BURNTIME)`＋`MakeSmallPropagator` | 主模拟 | [[reference/components/burnable]]、[[reference/components/propagator]]；[调用](../../../raw/current/scripts/prefabs/furtuft.lua#L53-L54) |
| 组件：`stackable` | 显式 40 叠 | 主模拟 | [[reference/components/stackable]]；[赋值](../../../raw/current/scripts/prefabs/furtuft.lua#L44-L45) |
| 组件：`hauntable` | 作祟只弹飞（点燃分支被注释） | 主模拟 | [[reference/components/hauntable]]；[调用](../../../raw/current/scripts/prefabs/furtuft.lua#L56) |
| API：`shedder` | 熊獾每 45 秒 `DoSingleShed`；震地 `DoMultiShed(3,false)` | 熊獾／装甲熊獾服务端 | [[reference/components/shedder]]；[挂接](../../../raw/current/scripts/prefabs/bearger.lua#L487-L490) |
| 关联 Prefab：`bearger`／`mutatedbearger` | 唯一"活体"来源 | 两注册共用 `commonfn` | [注册](../../../raw/current/scripts/prefabs/bearger.lua#L844-L846) |
| 关联 Prefab：`bearger_fur` | 90 撮的配方产物；尸体换算的输入 | 制作／尸体结算 | [配方](../../../raw/current/scripts/recipes.lua#L470) |

## 典型接入

### 调整熊獾的掉毛间隔

- 前提与选择：服务端 `modmain.lua`。掉毛由 `shedder:StartShedding(interval)` 起周期任务，postinit 里重起一次即可覆盖构造时的 45 秒。
- 生效与撤销：只影响该实例之后的节奏；熊獾冬眠醒来时会用 `TUNING.BEARGER_SHED_INTERVAL` 再次 `StartShedding`，**postinit 的重设会被覆盖**，要长期生效需同时改 `TUNING` 或监听醒来事件。
- 原版依据：`StartShedding` 先 `Cancel` 旧任务再建新周期任务。[启动](../../../raw/current/scripts/components/shedder.lua#L12-L17)、[醒来重设](../../../raw/current/scripts/prefabs/bearger.lua#L298-L307)

```lua
-- modmain.lua：把熊獾的掉毛间隔改成 90 秒（冬眠醒来后会被原版重置）
AddPrefabPostInit("bearger", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local shedder = inst.components.shedder
    if shedder ~= nil then
        shedder:StartShedding(90)
    end
end)
```

- 调整与验证：间隔单位是秒；`DoSingleShed` 每次仍只出 1 件，震地的 3 件不受此间隔影响。未运行，只核对了调用顺序。

## 常见陷阱与限制

- **误区：毛丛和猫尾是一回事。** 两者都是"猫/熊身上的毛"，但注册、宿主、掉落表、堆叠（40 vs 20）、燃料与 `cattoy` 标签全不同；毛丛来自熊獾，浣猫掉的是 [[reference/prefabs/coontail]]。[对照](../../../raw/current/scripts/prefabs/furtuft.lua#L36-L56)
- **边界：地上的毛丛会自己消失。** 生成后 25–35 秒进入消失序列：醒着时先 `NOCLICK`＋禁拾取，再等 `disappear` 动画结束（另有“动画长度＋0.1”的兜底任务）才 `Remove`；休眠时 `Disappear` 开头直接 `Remove`。想留就捡起来（进背包 `StopDisappear`）。[消失](../../../raw/current/scripts/components/disappears.lua#L12-L49)、[停表](../../../raw/current/scripts/components/disappears.lua#L51-L61)
- **边界：烧掉就没有燃料价值。** 它自身可烧 15 秒，丢进火里当燃料才是 45 秒；被火点着走 `burnable` 而不是 `fuel`。[燃烧](../../../raw/current/scripts/prefabs/furtuft.lua#L53-L54)
- **边界：作祟不会点燃。** helper 名叫 `LaunchAndIgnite`，但点燃段被注释，只有弹飞。[helper](../../../raw/current/scripts/standardcomponents.lua#L1204-L1213)
- **未核验**：未运行游戏。45 秒间隔与 25–35 秒消失时间为源码推导；震地频率、尸体被啃到 `meat_level >= 2` 的实际路径未实测。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `furtuft.lua` 全文；`bearger.lua` 的 `commonfn`／`normalfn`／`mutatedfn`、shedder 挂接、震地回调、睡眠与死亡停表；`components/shedder.lua`、`components/disappears.lua`、`components/fueled.lua`；`behaviours/beargeroffscreen.lua` 与 `brains/beargerbrain.lua` 的注释 require；`corpses.lua`／`corpses_defs.lua` 的熊皮换算；`recipes.lua` 熊皮配方；`tuning` 的间隔、燃料、燃烧常量；`chinese_s.po` 名称。
- 游戏运行：未运行。秒数按 `seg_time`＝30 秒、`total_day_time`＝480 秒换算。
- 复审修订：2026-11-10，R1 条目：P3-6（shedder 挂接锚点）。
- 复审修订：2026-11-10，R09（尸体门禁与消失序列）、R13（摘要名称）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 同批对照：[[reference/prefabs/coontail]]
- 组件：[[reference/components/shedder]]、[[reference/components/disappears]]、[[reference/components/fueled]]、[[reference/components/burnable]]、[[reference/components/lootdropper]]

返回 [[reference/prefabs/index|Prefab 索引]]。
