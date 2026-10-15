---
title: "羽毛扇与幸运扇（featherfan／perdfan）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "原版手扇参照：featherfan／perdfan 共用 fans.lua 的 common_fn 工厂，FAN 右键动作经 fan 组件回调在半径 7 内灭火（含传播器余热重置）并把实体体温向 2.5 度方向拉（单次最多 -50）；幸运扇另接吟唱回调，每次动作前摇内最多两次生成 2 秒龙卷风、每次 2 耐久；两态均无新鲜度、不可堆叠。"
aliases: ["featherfan", "perdfan", "羽毛扇", "幸运扇", "豪华扇", "降温扇", "Luxury Fan", "Lucky Fan"]
tags: [prefab, item]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fans.lua", "components/fan.lua", "components/finiteuses.lua", "components/burnable.lua", "components/temperature.lua", "componentutil.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "componentactions.lua", "bufferedaction.lua", "prefabs/staff_tornado.lua", "simutil.lua", "mainfunctions.lua", "recipes.lua", "prefabskins.lua", "prefabs/skinprefabs.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 羽毛扇与幸运扇（featherfan／perdfan）

## 快速判断

- 职责与适用：`prefabs/fans.lua` 共用工厂注册的两把手持降温扇（物品，不可装备）：羽毛扇 `featherfan`（15 次）只做范围灭火＋降温；幸运扇 `perdfan`（9 次）在同一效果上叠加"吟唱"——一次 FAN 动作的前摇里最多两次召唤 2 秒龙卷风，每次吟唱扣 2 耐久。两态均无新鲜度、不可堆叠（整文件无 perishable／stackable 挂载，见[验证记录](#验证记录)）。（[工厂](../../../raw/current/scripts/prefabs/fans.lua#L77-L130)、[注册](../../../raw/current/scripts/prefabs/fans.lua#L169-L170)）
- 优先使用：改扇风效果（灭火／降温数值、目标集）看[扇风结算](#扇风结算)；改龙卷风吟唱看[幸运扇吟唱](#幸运扇吟唱)；回调门禁契约（`canusefn`／空回调也会成功耗耐久）见 [[reference/components/fan]]。
- 重要副作用：扇风对半径 7 内带 `smolder`/`fire` 标签的目标 `Extinguish(true, 0)` 熄灭并重置传播器余热，同时对标签筛选（`smolder`/`fire`/`player`/`inventoryitemtemperature` 之一）且能读取到温度的目标按**当时**体温结算 `clamp(2.5 − 体温, −50, 0)` 的降温请求（玩家端还要过 `Temperature:DoDelta` 的隔热折算，[OnUse](../../../raw/current/scripts/prefabs/fans.lua#L18-L33)）；两扇耗尽均 `inst.Remove` 自毁（[common_fn](../../../raw/current/scripts/prefabs/fans.lua#L123-L125)）。
- 修改边界：本页覆盖两注册的工厂差异、动作链、扇风与吟唱结算及获取；龙卷风实体（tornado）内部机制、烧烧/温度系统只给入口；客户端预测仅到 `SGwilson_client` 的 `use_fan` 处理器。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `featherfan` | `feather_fn` → `common_fn()`（无覆盖符号、无吟唱回调）→ Pristine 前只有 `fan` 标签 → 主模拟 finiteuses＝15、floater `med` 尺寸 | 物品；普通扇：仅 `SetOnUseFn`；[工厂](../../../raw/current/scripts/prefabs/fans.lua#L132-L146) |
| `perdfan` | `perd_fn` → `common_fn("swap_fan_perd", OnChanneling)` → Pristine 前 `channelingfan` 标签 → 主模拟 finiteuses＝9、floater `large`＋漂浮换 `swap_fan_perd` 符号、`scrapbook_overridedata` 换图鉴外观 | 物品；吟唱扇：`SetOnChannelingFn`＋`SetOverrideSymbol`；仅声明 `tornado` 依赖（[prefabs](../../../raw/current/scripts/prefabs/fans.lua#L11-L14)）；[工厂](../../../raw/current/scripts/prefabs/fans.lua#L148-L167) |

共同初始化（[common_fn](../../../raw/current/scripts/prefabs/fans.lua#L77-L130)）：bank／build `fan`、`idle` 动画 → `fan` 标签（吟唱扇按回调另加 `channelingfan`，[fan 组件镜像注释](../../../raw/current/scripts/components/fan.lua#L22-L30)）→ `donotautopick` 标签 → floater（带 swap_data）→ Pristine → 主模拟 inspectable→inventoryitem→fan（OnUse／可选 OnChanneling／覆盖符号）→finiteuses（`SetOnFinished(inst.Remove)`＋`SetConsumption(FAN, 1)`）→`MakeHauntableLaunch`。

名称（简中 `.po` `msgctxt` 定向核验）：`STRINGS.NAMES.FEATHERFAN`＝羽毛扇（Luxury Fan）（[chinese_s.po#L342654-L342657](../../../raw/current/scripts/languages/chinese_s.po#L342654-L342657)）；`STRINGS.NAMES.PERDFAN`＝幸运扇（Lucky Fan）（[chinese_s.po#L346804-L346807](../../../raw/current/scripts/languages/chinese_s.po#L346804-L346807)）——perdfan 有独立 NAMES 键，不走派生命名。

获取：羽毛扇麋鹿鹅羽毛 5＋芦苇 2＋绳索 2，`TECH.SCIENCE_TWO`（[recipes.lua#L526](../../../raw/current/scripts/recipes.lua#L526)；原料名按 `STRINGS.NAMES.GOOSE_FEATHER`＝麋鹿鹅羽毛，[chinese_s.po#L343384-L343387](../../../raw/current/scripts/languages/chinese_s.po#L343384-L343387)）；幸运扇幸运金块 3，`TECH.PERDOFFERING_THREE`、`nounlock`＋`actionstr="PERDOFFERING"`，制作站为火鸡神龛 `perdshrine`（[recipes.lua#L1175](../../../raw/current/scripts/recipes.lua#L1175)、[PROTOTYPER_DEFS](../../../raw/current/scripts/recipes.lua#L44)；站名按 `STRINGS.NAMES.PERDSHRINE`＝火鸡神龛，[chinese_s.po#L346810-L346812](../../../raw/current/scripts/languages/chinese_s.po#L346810-L346812)）。皮肤：featherfan 有 `featherfan_exotic` 一枚注册（[prefabskins.lua#L754-L756](../../../raw/current/scripts/prefabskins.lua#L754-L756)、[skinprefabs.lua#L4802-L4808](../../../raw/current/scripts/prefabs/skinprefabs.lua#L4802-L4808)），perdfan 本 build 无皮肤。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：fan | `SetOnUseFn(OnUse)` 两扇同接；perdfan 另 `SetOnChannelingFn(OnChanneling)`＋`SetOverrideSymbol("swap_fan_perd")` | 主模拟；组件只分发回调，效果与耗耐久都在本文件 | [[reference/components/fan#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/fans.lua#L114-L121) |
| 组件：finiteuses | 上限＝当前＝15／9；`SetConsumption(ACTIONS.FAN, 1)` 由动作成功后结算；吟唱的 2 费由回调内直接 `Use(2)` | 主模拟；耗尽 `usesdepleted` 标签＋Remove | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/fans.lua#L123-L125)、[结算](../../../raw/current/scripts/bufferedaction.lua#L22-L37) |
| 组件：inventoryitem／floater／hauntable | 可拾取；featherfan `med`、perdfan `large`＋漂浮 bank swap；作祟弹飞 | 主模拟（hauntable）／公共端（floater） | [[reference/components/hauntable#按效果选工厂]]；[floater 差异](../../../raw/current/scripts/prefabs/fans.lua#L135-L136)、[perd floater](../../../raw/current/scripts/prefabs/fans.lua#L151-L155) |
| SG／Brain：`use_fan` | FAN 动作统一进扇风状态：吟唱扇在帧 30／50 各调一次 `Channel`，帧 70 `PerformBufferedAction`（再走 `Fan` 回调＋扣 1 费） | 服务端状态；客户端 `SGwilson_client` 有镜像 `use_fan` 预测（按 `channelingfan` 标签加 busy） | [路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1294)、[状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19452-L19500)、[客户端](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L3862-L3878) |
| API／动作：`ACTIONS.FAN` | 右键动作（`rmb=true`、可骑乘使用）；fn 做物品拟态检查后调 `fan:Fan(act.target or act.doer)`；动作由 `fan` 组件经 USEITEM／INVENTORY 收集器提供 | 主模拟结算；INVENTORY 收集器给右键、USEITEM 给对目标使用 | [定义](../../../raw/current/scripts/actions.lua#L472)、[fn](../../../raw/current/scripts/actions.lua#L3943-L3951)、[收集器](../../../raw/current/scripts/componentactions.lua#L2708-L2710) |
| API／数据：`OnUse` 目标集 | `FindEntities` 半径 7，排除 FX/NOCLICK/DECOR/INLIMBO/幽灵，须带 `smolder`/`fire`/`player`/`inventoryitemtemperature` 任一标签 | 主模拟；`GetEntityTemperature`／`DoDeltaTemperatureToEntity` 兼容 temperature 与物品温度两条路径，玩家负 delta 再经 `Temperature:DoDelta` 隔热折算 | [OnUse](../../../raw/current/scripts/prefabs/fans.lua#L16-L33)、[helper](../../../raw/current/scripts/componentutil.lua#L2416-L2438)、[隔热折算](../../../raw/current/scripts/components/temperature.lua#L71-L83) |
| 关联 Prefab：`tornado` | perdfan 吟唱生成，`SetDuration(2)` 缩命、写 `WINDSTAFF_CASTER` 供 PK 命名（复用风杖龙卷风实体） | 主模拟；依赖声明＋实际生成（吟唱回调内） | [生成](../../../raw/current/scripts/prefabs/fans.lua#L56-L71)、[SetDuration](../../../raw/current/scripts/prefabs/staff_tornado.lua#L125-L130) |
| 关联 Prefab：`collapse_small` 等 | perdfan 无；资源侧两扇共用 `anim/fan.zip`（bank／build `fan`），perdfan 以 `swap_fan_perd` 符号换色 | 静态声明 | [assets](../../../raw/current/scripts/prefabs/fans.lua#L1-L9) |
| 资源：anim `fan.zip` | `idle` 动画＋持握／扇风符号 `swap_fan`（perdfan 换 `swap_fan_perd`）；玩家持扇动画由 `use_fan` 状态覆盖 `fan01` 符号 | 静态声明 | [符号覆盖](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19466-L19474) |

## 扇风结算

一次 FAN 动作（两扇相同的效果部分，[use_fan 状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19452-L19500) 帧 70 → [FAN.fn](../../../raw/current/scripts/actions.lua#L3943-L3951) → [Fan](../../../raw/current/scripts/components/fan.lua#L47-L52) → [OnUse](../../../raw/current/scripts/prefabs/fans.lua#L18-L33)）：

1. 以动作目标（右键自身时为玩家）为中心取半径 7 实体（[FEATHERFAN_RADIUS](../../../raw/current/scripts/tuning.lua#L3036-L3039)）。
2. 有 burnable 的目标：`Extinguish(true, 0)`——停阴燃＋按 `resetpropagator` 停传播并重置余热为 0（[Extinguish](../../../raw/current/scripts/components/burnable.lua#L454-L476)）。
3. 有温度的目标（玩家经 temperature 组件、温变物品经 inventoryitem 温度）：回调按**当时**体温算请求值 `delta = clamp(2.5 − 体温, −50, 0)`——体温高于 2.5 度时向下拉、已 ≤2.5 度则为 0，−50 只是回调侧 clamp 的请求上限（[tuning](../../../raw/current/scripts/tuning.lua#L3036-L3039)）；玩家路径随后进 `Temperature:DoDelta`，负 delta 按冬季隔热折算（× `SEG_TIME/(SEG_TIME+冬季隔热)`，[temperature.lua#L71-L84](../../../raw/current/scripts/components/temperature.lua#L71-L83)），实际体温变化可小于请求值；温变物品走 `inventoryitem:AddTemperature` 无折算（[DoDeltaTemperatureToEntity](../../../raw/current/scripts/componentutil.lua#L2416-L2438)）。
4. 动作成功后 `OnUsedAsItem` 扣 1 耐久（[结算链](../../../raw/current/scripts/bufferedaction.lua#L22-L37)）；`canusefn` 未设置，回调必成功。

### 幸运扇吟唱

perdfan 的 `use_fan` 前摇内帧 30 与帧 50 各有一次 `Channel` 调用（[时间轴](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19478-L19499)），每次（[OnChanneling](../../../raw/current/scripts/prefabs/fans.lua#L39-L75)）：

- 门禁：`finiteuses:GetUses() > 3`——不足 4 次耐久不吟唱。
- 以目标（或持有者）位置 `FindWalkableOffset` 找落点，两次吟唱的角度按 `lasttornadoangle` 在 ±π 间交替。
- 扣 2 耐久、生成 `tornado` 并 `SetDuration(2)`（原版风杖龙卷风为 5 秒，[TORNADO_LIFETIME](../../../raw/current/scripts/tuning.lua#L3033)）、记录目标位置、写施法者身份供 PK 判定。

9 次耐久的示例轨迹（前提：每次吟唱都在 `FindWalkableOffset` 找到落点、每次普通扇风都成功执行）：第 1 次动作 9→7→5→4（2 龙卷＋1 动作费），第 2 次 4→2→1（1 龙卷＋1 费），第 3 次 1→0（只剩扇风）。每次吟唱的上限仍是门禁 `uses > 3`、一次 2 费、单次动作至多两次吟唱点；若某次吟唱找不到落点则该次不扣费也不生成（[OnChanneling](../../../raw/current/scripts/prefabs/fans.lua#L39-L74)），动作被物品拟态拦截时 `ACTIONS.FAN.fn` 提前返回 false、动作费也不扣（[FAN.fn](../../../raw/current/scripts/actions.lua#L3943-L3951)）。帧 70 的普通扇风效果（灭火＋降温）与吟唱独立结算，吟唱失败不影响扇风。

## 典型接入

### 调整扇风降温强度与目标

- 前提与选择：`modmain.lua` 用 `AddPrefabPostInit` 包装回调。降温数值是 `OnUse` 内现读的 `TUNING.FEATHERFAN_*`，但目标集与 clamp 逻辑写死在回调里，改行为选包装 `onusefn`；只改数值可直接换 tuning。
- 生效与撤销：postinit 只对新构造与读档重建的实例生效（读档重建 SpawnSaveRecord 同样先 `SpawnPrefab` 跑 postinit，[mainfunctions.lua#L464-L466](../../../raw/current/scripts/mainfunctions.lua#L464-L466)），模组加载时已存在的运行中实例不会重跑；回调不存档，`SetOnUseFn` 整体替换回调，原效果（灭火＋降温）需在新回调内复调。撤销即移除模组。
- 原版依据：原回调一行灭火一行降温（[OnUse](../../../raw/current/scripts/prefabs/fans.lua#L22-L31)）；组件分发忽略回调返回值、返回 true 即算动作成功并扣耐久（[Fan](../../../raw/current/scripts/components/fan.lua#L47-L52)）。

```lua
-- modmain.lua：在原扇风效果之外追加一段独立降温段（按结算时体温与隔热独立生效）
local function ExtendFan(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local fan = inst.components.fan
    local old_onuse = fan.onusefn
    fan:SetOnUseFn(function(inst, target)
        if old_onuse ~= nil then
            old_onuse(inst, target)
        end
        local x, y, z = target.Transform:GetWorldPosition()
        local ents = GLOBAL.TheSim:FindEntities(x, y, z, 7, nil,
            { "FX", "NOCLICK", "DECOR", "INLIMBO", "playerghost" }, { "player" })
        for _, v in pairs(ents) do
            local temp = GLOBAL.GetEntityTemperature(v)
            if temp ~= nil then
                GLOBAL.DoDeltaTemperatureToEntity(v, math.clamp(2.5 - temp, -80, 0))
            end
        end
    end)
end

AddPrefabPostInit("featherfan", ExtendFan)
AddPrefabPostInit("perdfan", ExtendFan)
```

- 调整与验证：示例是在原回调之后追加一段独立降温，两段**各自现读体温、各自 clamp、各自过** `Temperature:DoDelta` 的隔热折算（玩家负 delta × `SEG_TIME/(SEG_TIME+冬季隔热)`）——无隔热、初温 30 度例：原回调 `clamp(2.5−30,−50,0)`＝−27.5 把体温拉到 2.5 度，追加段 `clamp(2.5−2.5,−80,0)`＝0，无追加贡献；原回调被隔热折算得越小、追加段贡献越大。`−50`／`−80` 只是两段在 `DoDeltaTemperatureToEntity` 分发前的请求值上限（[componentutil.lua#L2416-L2438](../../../raw/current/scripts/componentutil.lua#L2416-L2438)），不是玩家实际体温变化量，两段不做合计。要改原回调的请求上限换 `TUNING.FEATHERFAN_COOLING`，它控制不了本例追加段。吟唱扇如需重复降温请自行去重（原回调半径内每目标只结算一次）。

## 常见陷阱与限制

- **边界：吟唱扇一次动作最多两发龙卷风**：`use_fan` 时间轴只有帧 30／50 两个 `Channel` 点，加帧 70 的普通扇风；吟唱门禁是 `uses > 3`，`Use(2)` 在回调内即时扣，与动作结算的 1 费分开。[时间轴](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19478-L19499)、[门禁](../../../raw/current/scripts/prefabs/fans.lua#L40)
- **边界：幸运扇保持羽毛扇全部效果**：perdfan 的 `onusefn` 与 featherfan 同一个 `OnUse`，灭火＋降温照常；龙卷风是叠加产出不是替代。[common_fn](../../../raw/current/scripts/prefabs/fans.lua#L114-L121)
- **边界：降温只向 2.5 度拉、不制冷到负温**：clamp 上限为 0，目标体温 ≤2.5 度时扇风对该目标无效；龙卷风实体本身不降温。[OnUse](../../../raw/current/scripts/prefabs/fans.lua#L28-L31)
- **边界：`donotautopick` 标签**：两扇都被标记为不参与自动拾取候选（simutil 的自动拾取过滤器直接排除，[simutil.lua#L406-L429](../../../raw/current/scripts/simutil.lua#L406-L429)），需要玩家手动拾取；此标签在 `common_fn` Pristine 前添加（[L100](../../../raw/current/scripts/prefabs/fans.lua#L100)）。
- **边界：perdfan 获取端在活动内容**：配方制作站 `perdshrine`（火鸡神龛）与 `TECH.PERDOFFERING_THREE` 接线经 recipes.lua／PROTOTYPER_DEFS 核验，神龛在当前世界的常规开放条件（活动开关／世界选项）未在本页核验。[PROTOTYPER_DEFS](../../../raw/current/scripts/recipes.lua#L44)
- **误区：羽毛扇是装备**：两扇均无 equippable，`fan`／`channelingfan` 标签 + FAN 右键动作即可用，无需装入手部槽。[工厂](../../../raw/current/scripts/prefabs/fans.lua#L77-L130)

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394（`raw/current/BUILD.txt`）。核验 `prefabs/fans.lua` 全文（common_fn 公共骨架、feather_fn／perd_fn 差异、OnUse 目标集与温度 clamp、OnChanneling 门禁／角度交替／2 费生成、无 perishable／stackable／equippable）；`components/fan.lua` 回调契约；`finiteuses.lua` 消耗链；`burnable.lua` 的 Extinguish（resetpropagator／heatpct）；`componentutil.lua` 两个温度 helper；SGwilson 的 FAN 路由与 `use_fan` 状态（30/50/70 帧）、SGwilson_client 镜像状态；`componentactions.lua` 两处 fan 收集器；`actions.lua` FAN 定义与 fn；`bufferedaction.lua` 结算链；`staff_tornado.lua` 的 SetDuration 与施法者字段；`simutil.lua` 的 donotautopick 过滤；recipes 两配方与 `PROTOTYPER_DEFS.perdshrine`；皮肤注册两文件；tuning 的 FEATHERFAN／PERDFAN／TORNADO_LIFETIME 键；chinese_s.po 两名称键定向核验；scrapbookdata 的 featherfan 条目（15 次）旁证。
- 游戏运行：未运行。未覆盖：扇风对燃烧实体的实际灭火范围表现、龙卷风 2 秒实际存在时长与 PK 命名、perdfan 神龛实际开放条件与合成流程、皮肤实装外观；−50 度／2.5 度／7 格／15／9 次为源码推导值。
- 结构检查：validate_page.py（standard 档）与 validate_source_links.py 实际执行通过（见交付说明）。
- 修订：2026-11-08（独立检视意见，主代理回源码复核采纳）：典型接入示例改为先定义 `ExtendFan` 再注册两处 `AddPrefabPostInit`（原顺序在注册时刻全局函数尚未定义，postinit 拿到 nil 会静默失效）；"调整与验证"改为如实描述两段独立 clamp 叠加（极端情况单次合计最多 −50+(−80)＝−130 度，中段温度 −55 而非 −27.5），不再称"等效把封顶放大到 −80／其余行为不变"；配方原料名按 po 键 `GOOSE_FEATHER` 改为麋鹿鹅羽毛（chinese_s.po:343384-343387）；删除正文 I09 内部台账编号引用，改为"整文件无 perishable／stackable 挂载"源码表述。
- 外部复检修订：2026-11-08（docs/reports/w05/prefab-inventory-review-b01-20261108，主代理逐项回源码核实采纳）：降温叠加表述纠正——两段回调各自现读体温、各自 clamp、各自过 `Temperature:DoDelta` 的隔热折算（temperature.lua:71-84），删除上一轮"合计 −55／−130"的说法（第二段读到的是第一段降温后的温度：无隔热、初温 30 度例中追加段 clamp(2.5−2.5,−80,0)＝0）；9 次耐久轨迹改为带前提的示例表述（每次吟唱需 `FindWalkableOffset` 找到落点才扣 2 费、普通 FAN 需通过物品拟态检查才进结算）；"postinit 对新旧实例都生效"收窄为新构造与读档重建实例（SpawnSaveRecord 先 SpawnPrefab，mainfunctions.lua:464-466）；制作站名按 `STRINGS.NAMES.PERDSHRINE` 改"火鸡神龛"（chinese_s.po:346810-346812）；frontmatter sources 补 components/temperature.lua 与 mainfunctions.lua。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
