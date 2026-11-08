---
title: "排箫（panflute）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "原版催眠乐器参照：panflute 经 PLAY 动作触发 instrument 组件半径 15 的听觉广播，按目标组件分流——sleeper 加困倦、grogginess 加昏沉、farmplanttendable 直接记一次照料；10 次耐久每吹一次耗 1；沃拓克斯技能树接入免费吹奏、吹奏招魂与目标遗忘仇恨。"
aliases: ["panflute", "排箫", "潘笛", "催眠乐器", "Pan Flute"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/panflute.lua", "components/instrument.lua", "components/sleeper.lua", "components/grogginess.lua", "components/farmplanttendable.lua", "components/finiteuses.lua", "components/tool.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "componentactions.lua", "bufferedaction.lua", "prefabs/skilltree_wortox.lua", "prefabs/wortox.lua", "prefabs/player_common.lua", "mainfunctions.lua", "actions.lua", "recipes.lua", "constants.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 排箫（panflute）

## 快速判断

- 职责与适用：`prefabs/panflute.lua` 单注册的可拾取乐器（物品，不可装备）：吹奏以音乐家为中心半径 15 广播，对带 `sleeper`／`player`／`tendable_farmplant` 任一标签的听众按组件分流——sleeper 加困倦入睡、grogginess 加昏沉、farmplanttendable 记一次照料；无新鲜度、不可堆叠的耐用品（10 次，每次 PLAY 耗 1；整文件无 perishable／stackable 挂载，见[验证记录](#验证记录)）。（[工厂](../../../raw/current/scripts/prefabs/panflute.lua#L101-L152)、[听觉分流](../../../raw/current/scripts/prefabs/panflute.lua#L29-L56)）
- 优先使用：做群体催眠／范围无伤效果参照 `instrument`＋`HearPanFlute` 模式，回调接线见 [[reference/components/instrument]]；改时长／半径读[典型接入](#典型接入)；沃拓克斯联动读[沃拓克斯技能联动](#沃拓克斯技能联动)。
- 重要副作用：吹奏不消耗资源、不产生冷却，效果即时结算；耐久耗尽 `inst.Remove` 整体移除（[finiteuses](../../../raw/current/scripts/prefabs/panflute.lua#L137-L142)）；骑乘听众在把 `ridersleep` 推给坐骑之外，自身组件分支照常结算——骑乘玩家仍会被吹出昏沉（[mount 与分流](../../../raw/current/scripts/prefabs/panflute.lua#L37-L55)）。
- 修改边界：本页覆盖注册、动作链、听觉分流与沃拓克斯三技能接线；配方里的芦苇（`cutreeds`）见 [[reference/prefabs/basic-gathered-materials]]；`wortox_soul_spawn`／`wortox_forget_debuff`／`wortox_panflute_buff` 各自的完整机制只给入口；客户预测只在 `SGwilson_client` 的 `play_flute` 处理器层提及，不展开预测实现。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `panflute` | `fn`：Transform／AnimState／Network → `MakeInventoryPhysics` → `flute`＋`tool` 标签（Pristine 前加，供客户端 SG 路由与优化）→ bank／build `pan_flute` → `MakeInventoryFloatable` → Pristine → 主模拟 inspectable→instrument→tool→finiteuses→inventoryitem→`MakeHauntableLaunch`＋floater 换动画事件 | 物品；无 SG／Brain、无 equippable、无 perishable／stackable；[工厂](../../../raw/current/scripts/prefabs/panflute.lua#L101-L152) |

名称：简中 `.po` `msgctxt "STRINGS.NAMES.PANFLUTE"`＝排箫（Pan Flute）（[chinese_s.po#L346739-L346742](../../../raw/current/scripts/languages/chinese_s.po#L346739-L346742)）。

获取：配方芦苇 5＋曼德拉草 1＋绳索 1，`TECH.MAGIC_TWO`（[recipes.lua#L508](../../../raw/current/scripts/recipes.lua#L508)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：instrument | `SetRange(15)`、`SetOnPlayedFn/OnHeardFn/OnFinishedPlayingFn` 三回调；`Play` 先吹奏回调、再逐听众广播、末尾完成回调 | 主模拟；range 为 `FindEntities` 半径 | [[reference/components/instrument#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/panflute.lua#L127-L132)、[Play](../../../raw/current/scripts/components/instrument.lua#L34-L51) |
| 组件：tool | `SetAction(ACTIONS.PLAY)` 加 `play_tool` 标签，使 PLAY 成为合法 tool 动作（`TOOLACTIONS.PLAY`） | 主模拟挂载；标签在 Pristine 前镜像（`tool`） | [[reference/components/tool#关键接口]]；[SetAction](../../../raw/current/scripts/components/tool.lua#L41-L45)、[TOOLACTIONS](../../../raw/current/scripts/constants.lua#L2187-L2197) |
| 组件：finiteuses | 上限＝当前＝10；`SetConsumption(PLAY, 1)`；`SetOnFinished(inst.Remove)`；`SetModifyUseConsumption(UseModifier)` 承接沃拓克斯免耗 | 主模拟；动作成功后经 `BufferedAction:Do` → `OnUsedAsItem` 结算 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/panflute.lua#L137-L142)、[结算](../../../raw/current/scripts/bufferedaction.lua#L22-L37) |
| 组件：inventoryitem／floater／hauntable | 可拾取；漂浮切 `float` 动画；作祟弹飞 | 主模拟（hauntable）／公共端（floater 标签） | [[reference/components/hauntable#按效果选工厂]]；[挂载](../../../raw/current/scripts/prefabs/panflute.lua#L144-L149) |
| SG／Brain：`play_flute` | PLAY 动作按 `flute` 标签路由到吹奏状态；帧 30 执行 `PerformBufferedAction` 并起笛声循环、帧 85 停声 | 服务端状态；客户端镜像状态在 `SGwilson_client`（`ActionHandler(ACTIONS.PLAY, …)` 同规则） | [路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1284-L1293)、[状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9284-L9334)、[客户端](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L566-L569) |
| API／动作：`ACTIONS.PLAY` | `fn` 要求 `invobject.components.instrument` 并调 `Play(act.doer)`；动作由 instrument 组件经 INVENTORY 收集器提供 | 主模拟结算；收集器同时给 `flute` 标签路由用 | [动作](../../../raw/current/scripts/actions.lua#L2857-L2861)、[收集器](../../../raw/current/scripts/componentactions.lua#L2755-L2757)、[定义](../../../raw/current/scripts/actions.lua#L410) |
| API／数据：`wortox_panflute_buff` | 沃拓克斯"灵感"免费吹奏 buff：吹奏回调发现即移除并还 5 点理智，本次耗用改 0 | 主模拟；buff 由技能 `wortox_panflute_playing` 的计时器授予 | [OnPlayed](../../../raw/current/scripts/prefabs/panflute.lua#L9-L19)、[UseModifier](../../../raw/current/scripts/prefabs/panflute.lua#L92-L99)、[授予](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L31-L37) |
| 关联 Prefab：`wortox_soul_spawn`／`_fx` | 仅 `prefabs` 依赖声明＋soulcaller 技能吹奏完延迟 52 帧召唤 6 魂（非本页展开机制） | 主模拟；依赖声明不等于普通吹奏生成 | [声明](../../../raw/current/scripts/prefabs/panflute.lua#L4-L7)、[召唤](../../../raw/current/scripts/prefabs/panflute.lua#L58-L90) |
| 关联 Prefab：`wortox_forget_debuff` | forget 技能下，被吹睡的目标挂遗忘 debuff：醒后 5 秒内 `combat:SetShouldAvoidAggro(音乐家)` | 主模拟；非联网实体、宿主死亡或超时自移除 | [挂载](../../../raw/current/scripts/prefabs/panflute.lua#L46-L49)、[实现](../../../raw/current/scripts/prefabs/wortox.lua#L1126-L1150) |
| 资源：anim `pan_flute.zip` | bank／build `pan_flute`，`idle`／`float` 动画；吹奏动画复用玩家 `flute` 状态并覆盖 `pan_flute01` 符号 | 静态声明 | [assets](../../../raw/current/scripts/prefabs/panflute.lua#L1-L3)、[符号覆盖](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9291) |

## 使用流程与听觉分流

一次吹奏的完整链路（服务端）：

1. 玩家对物品发起 PLAY 动作（inventory 收集器提供，[componentactions.lua#L2755-L2757](../../../raw/current/scripts/componentactions.lua#L2755-L2757)）→ SG 按 `flute` 标签进 `play_flute`（[路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1284-L1293)）。
2. 帧 30 `PerformBufferedAction` → `ACTIONS.PLAY.fn` → `Instrument:Play`：先 `OnPlayed`（缓存本次睡眠时长 20 秒、处理沃拓克斯 buff 与技能标记），再取半径 15 内全部实体逐个回调 `HearPanFlute`，最后 `OnFinishedPlaying`（[Play](../../../raw/current/scripts/components/instrument.lua#L34-L51)、[OnPlayed](../../../raw/current/scripts/prefabs/panflute.lua#L9-L27)）。
3. 动作成功后 `BufferedAction:Do` 调 `OnUsedAsItem(PLAY)`：`UseModifier` 先行（沃拓克斯免费吹奏返回 0），否则耗 1 耐久（[结算](../../../raw/current/scripts/bufferedaction.lua#L22-L37)、[OnUsedAsItem](../../../raw/current/scripts/components/finiteuses.lua#L101-L114)）。

`HearPanFlute` 对每个听众的门禁与分流（[HearPanFlute](../../../raw/current/scripts/prefabs/panflute.lua#L29-L56)）：

- 门禁：不是音乐家本人；非 PVP 时跳过其他玩家；冻结（freezable）、被钉（pinnable）、化石（fossilizable）状态听不见；且须带 `sleeper`／`player`／`tendable_farmplant` 任一标签。
- 骑乘中的听众：先把 `ridersleep`（sleepiness 10、时长同吹奏）推给坐骑实体，该分支无 return，听众自身的组件分流继续按序结算——骑乘玩家没有 sleeper 组件，仍会进 grogginess 支被吹出昏沉（[mount 与后续分支](../../../raw/current/scripts/prefabs/panflute.lua#L37-L55)）。
- `farmplanttendable` 优先：农田作物直接 `TendTo(音乐家)` 记一次照料（[TendTo](../../../raw/current/scripts/components/farmplanttendable.lua#L26-L31)），不入睡。
- `sleeper`：`AddSleepiness(10, 20)`——累积困倦**大于**抗性立即入睡，恰好**等于**抗性先排一个等长的延迟任务再入睡（[AddSleepiness](../../../raw/current/scripts/components/sleeper.lua#L248-L257)）；真正入睡还要求实体可见、未死亡，实际持续时长乘 `GetSleepTimeMultiplier`（累积额外抗性时最低 0.2 倍，[时长倍率与 GoToSleep](../../../raw/current/scripts/components/sleeper.lua#L275-L306)）。forget 技能激活时，还须此刻 `sleeper:IsAsleep()` 且目标带 combat 组件才加挂 `wortox_forget_debuff`（[挂载条件](../../../raw/current/scripts/prefabs/panflute.lua#L44-L49)）。
- `grogginess`（玩家走此支，玩家实体无 sleeper 组件，见 [player_common.lua#L2823](../../../raw/current/scripts/prefabs/player_common.lua#L2823)）：`AddGrogginess(10, 20)` 加昏沉值（[AddGrogginess](../../../raw/current/scripts/components/grogginess.lua#L163-L171)）。
- 都没有：推 `knockedout` 事件。

### 沃拓克斯技能联动

| 技能（skilltree_wortox） | 对吹奏的影响 | 证据 |
|---|---|---|
| `wortox_panflute_playing`（根技能） | 激活后启动计时器（1 游戏日±10%），到点授予 `wortox_panflute_buff`；带 buff 吹奏：buff 移除＋理智 +5（`SANITY_TINY`）＋本次 0 耗 | [计时](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L165-L193)、[授予](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L31-L37)、[tuning](../../../raw/current/scripts/tuning.lua#L7408-L7413) |
| `wortox_panflute_soulcaller` | 吹奏完成回调延迟 52 帧在周身环形带召唤 6 个 `wortox_soul_spawn`（半径 3.95–7.9：外圈＝`WORTOX_SOULSTEALER_RANGE`−0.1、内圈为外圈一半；TUNING 的 8 本身是"沃拓克斯走近时灵魂飞向其"的吸引范围调参基值，不是召唤半径，[tuning.lua#L4542](../../../raw/current/scripts/tuning.lua#L4542)） | [OnFinishedPlaying](../../../raw/current/scripts/prefabs/panflute.lua#L83-L90)、[DoSoulSummon](../../../raw/current/scripts/prefabs/panflute.lua#L66-L85) |
| `wortox_panflute_forget` | 吹奏时标记本次入睡的 sleeper 听众，挂遗忘 debuff（醒后 5 秒回避仇恨音乐家） | [OnPlayed](../../../raw/current/scripts/prefabs/panflute.lua#L21-L26)、[FORGET_DURATION](../../../raw/current/scripts/tuning.lua#L7413) |

## 典型接入

### 调整催眠时长与半径

- 前提与选择：`modmain.lua`。睡眠时长在每次吹奏的 `OnPlayed` 里现读 `TUNING.PANFLUTE_SLEEPTIME`（[OnPlayed](../../../raw/current/scripts/prefabs/panflute.lua#L11)），改 tuning 值即对后续每次吹奏生效，无需碰实例；半径存在组件 `range` 字段，构造期写入，用 postinit 重设。
- 生效与撤销：tuning 改动全局即时生效——`OnPlayed` 每次吹奏现读该值，已存在的运行中实例也随读取处生效；`SetRange` 只作用于新构造与读档重建的实例（读档重建 SpawnSaveRecord 同样先 `SpawnPrefab` 执行 postinit，[mainfunctions.lua#L464-L466](../../../raw/current/scripts/mainfunctions.lua#L464-L466)），模组加载时已存在的运行中实例不会重跑 postinit；range 不入存档（instrument 无 OnSave/OnLoad），重开世界按模组重新应用。撤销即移除模组。
- 原版依据：range 是 `Instrument:Play` 里 `FindEntities` 的直接实参（[Play](../../../raw/current/scripts/components/instrument.lua#L40)）；模组环境无 `__index=_G`，`TheWorld` 等引擎全局须 `GLOBAL.` 前缀，`TUNING` 由环境直供（[mods.lua#L258-L317](../../../raw/current/scripts/mods.lua#L258-L317)）。

```lua
-- modmain.lua：排箫催眠 20 秒 → 40 秒；半径 15 → 20
TUNING.PANFLUTE_SLEEPTIME = 40

AddPrefabPostInit("panflute", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.instrument:SetRange(20)
end)
```

- 调整与验证：`AddSleepiness(10, 时长)` 第二参即入睡时长（秒），`AddGrogginess` 第二参为昏倒时长；要加效果按 `HearPanFlute` 的分支顺序插入自有组件判断，避免被优先分支截走。

## 常见陷阱与限制

- **边界：非 PVP 不影响玩家**：门禁 `TheNet:GetPVPEnabled() or not inst:HasTag("player")`——普通合作局里排箫只催眠生物与照料作物，PVP 开启后才会把其他玩家吹昏（走 grogginess）。[门禁](../../../raw/current/scripts/prefabs/panflute.lua#L31-L36)
- **边界：目标组件决定分支**：sleeper／farmplanttendable／grogginess 按 `elseif` 顺序先到先得；骑乘只额外推一次 `ridersleep` 给坐骑，听众自身分支照常结算，二者不互斥。给新生物做"可被吹睡"需要挂 sleeper（或 groginess）并加对应标签。[分流](../../../raw/current/scripts/prefabs/panflute.lua#L37-L55)
- **边界：免费吹奏只免耐久不灭效果**：`UseModifier` 返回 0 只影响 `OnUsedAsItem` 的扣减，吹奏照常结算；buff 在吹奏回调里被移除，一次 buff 一次免费。[UseModifier](../../../raw/current/scripts/prefabs/panflute.lua#L92-L99)
- **边界：`panflute_sleeptime` 是实例上的临时约定**：`OnPlayed` 每次把 tuning 值缓存到 `inst.panflute_sleeptime` 供 `HearPanFlute` 读取（源码注释标明是留给模组的缓存位），模组改它只影响单次吹奏，改 tuning 才是持续方案。[缓存](../../../raw/current/scripts/prefabs/panflute.lua#L11)
- **误区：排箫是装备**：无 equippable 组件、无 HANDS 槽，`tool` 只是让 PLAY 进合法 tool 动作集；吹奏是物品动作，无需装备。[工厂](../../../raw/current/scripts/prefabs/panflute.lua#L101-L152)、[TOOLACTIONS](../../../raw/current/scripts/constants.lua#L2187-L2197)

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394（`raw/current/BUILD.txt`）。核验 `prefabs/panflute.lua` 全文（工厂、OnPlayed、HearPanFlute 门禁与四分支、召唤与完成回调、UseModifier、无 perishable／stackable／equippable）；`instrument.lua`（Play 广播序）；`sleeper.lua` 的 AddSleepiness；`grogginess.lua` 的 AddGrogginess；`farmplanttendable.lua` 的 TendTo；`finiteuses.lua` 消耗链与 `SetModifyUseConsumption`；`tool.lua` SetAction 与 `TOOLACTIONS.PLAY`；SGwilson 的 PLAY 路由与 `play_flute` 状态、客户端 ActionHandler；`componentactions.lua` instrument 收集器；`bufferedaction.lua` 的 OnUsedAsItem 结算；`skilltree_wortox.lua` 三技能节点、计时器与 buff 授予；`wortox.lua` 的 forget debuff 实现；recipes.lua 配方；tuning 的 PANFLUTE 三键、SANITY_TINY、WORTOX 技能数值与 `WORTOX_SOULSTEALER_RANGE`；player_common 确认玩家走 grogginess；chinese_s.po 名称键定向核验。
- 游戏运行：未运行。未覆盖：催眠对各类生物的实际抗性结算（sleepiness 10 对不同 `resistance` 的入睡轮数）、沃拓克斯 buff 图标与招魂实际表现、骑乘听众被吹奏的实际表现；20 秒／15 格／10 次为源码推导值。
- 结构检查：validate_page.py（standard 档）与 validate_source_links.py 实际执行通过（见交付说明）。
- 修订：2026-11-08（独立检视意见，主代理回源码复核采纳）：骑乘分流三处改为"`ridersleep` 推给坐骑后自身组件分支照常结算，骑乘玩家仍走 grogginess"（依据 panflute.lua:37-54 mount 分支后无 return）；客户端 PLAY 处理器锚点由 SGwilson_client.lua:582 改为 566-569（原行是 FAN 的 ActionHandler），修改边界同步去掉误写的 `use_fan`；frontmatter sources 补 prefabs/player_common.lua；删除正文 I08 内部台账编号引用，改为"整文件无 perishable／stackable 挂载"源码表述。
- 外部复检修订：2026-11-08（docs/reports/w05/prefab-inventory-review-b01-20261108，主代理逐项回源码核实采纳）：入睡条件改准——sleepiness 大于抗性立即入睡、等于抗性走等长延迟任务，实际入睡还要求实体可见／未死亡并乘 `GetSleepTimeMultiplier`（最低 0.2 倍，sleeper.lua:248-307）；forget debuff 补"此刻 IsAsleep 且带 combat 组件"前置（panflute.lua:44-49）；召唤环带半径改为 3.95–7.9，`TUNING.WORTOX_SOULSTEALER_RANGE`＝8 标注为灵魂吸引范围的调参基值而非召唤半径（tuning.lua:4542）；"postinit 对新旧实例都生效"收窄为新构造与读档重建实例（SpawnSaveRecord 先 SpawnPrefab 跑 postinit，mainfunctions.lua:464-466），并区分 tuning 全局即时生效与实例 postinit 的不同生效面。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 2 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
