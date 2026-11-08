---
title: "显眼箱子微光与传送塔沙柱（terrariumchest_fx、townportalsandcoffin_fx）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "两个由玩法宿主管理生命期的联网表现层：显眼箱子在恐怖之眼世界选项开启且带 scenariorunner 时的背／前双层微光（开箱或烧毁即撤，宿主的 fx 引用同时是 SHIMMER 状态标记），以及懒人传送塔传送时包裹玩家的沙柱（进出两侧玩家状态机生成、KillFX 二段收尾）；附 treasurechest.lua 六种组合宝箱的宿主辨析。"
aliases: ["terrariumchest_fx", "townportalsandcoffin_fx", "显眼箱子微光特效", "宝箱闪烁层", "传送沙柱", "沙棺特效", "显眼箱子", "懒人传送塔"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/treasurechest.lua", "prefabs/townportal.lua", "stategraphs/SGwilson.lua", "actions.lua", "componentactions.lua", "components/teleporter.lua", "scenarios/chest_terrarium.lua", "map/static_layouts/terrarium_forest_spiders.lua", "components/retrofitforestmap_anr.lua", "mainfunctions.lua", "worldsettings_overrides.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 显眼箱子微光与传送塔沙柱（terrariumchest_fx、townportalsandcoffin_fx）

## 快速判断

- 职责与适用：两个挂在玩法宿主上的联网 FX。`terrariumchest_fx` 是显眼箱子未开启时包裹箱身的微光层（`idle_back` 联网壳＋非专用端本地 `idle_front` 子层）；`townportalsandcoffin_fx` 是玩家用懒人传送塔传送时从地面卷起再落下的沙柱（`portal_in`／`portal_out` 两段动画＋进出音效）。两者都由宿主代码生成并管理生命期，自身无玩法组件。[微光工厂](../../../raw/current/scripts/prefabs/treasurechest.lua#L617-L655)、[沙柱工厂](../../../raw/current/scripts/prefabs/townportal.lua#L254-L286)
- 本体辨析：`treasurechest.lua` 一文件返回 6 个组合宝箱注册（[返回块](../../../raw/current/scripts/prefabs/treasurechest.lua#L766-L773)），**只有 `terrariumchest`（官方名"显眼箱子"）的 master_postinit 生成 `terrariumchest_fx`**，其余五个宿主见[宿主辨析表](#生成点与宿主条件)；`townportal`（官方名"懒人传送塔"）本体是可频道化的传送建筑，不在本页展开。
- 重要副作用：`terrariumchest_fx` **无自删路径**，靠宿主 `onburnt`／`onopen` 事件显式移除；宿主实体被删除时是否随引擎父级回收未核验。宿主的 `inst.fx` 引用直接决定审查状态 SHIMMER（[状态读取](../../../raw/current/scripts/prefabs/treasurechest.lua#L562-L566)）；沙柱是本页唯一带碰撞的 FX（`ObstaclePhysics` 半径 0.5，[工厂](../../../raw/current/scripts/prefabs/townportal.lua#L262)）。两者均 `persists=false`。
- 修改边界：只覆盖 2 注册的工厂、生成点、宿主条件与清理；`terrarium` 物品与恐怖之眼召唤玩法、传送塔频道（channelable）与配对玩法、`chest_labyrinth` 等陷阱宝箱链不展开。
- 名称：`STRINGS.NAMES.TERRARIUMCHEST`＝"显眼箱子"（[po](../../../raw/current/scripts/languages/chinese_s.po#L350364-L350367)）、`STRINGS.NAMES.TOWNPORTAL`＝"懒人传送塔"（[po](../../../raw/current/scripts/languages/chinese_s.po#L350494-L350497)）；`TERRARIUMCHEST_FX`／`TOWNPORTALSANDCOFFIN_FX` 无 `STRINGS.NAMES.*` 键，标题中"微光""沙柱"为已核验效果描述，**非官方译名**。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `terrariumchest_fx` | Transform／AnimState／Network → `FX` 标签 → bank／build `terrariumchest_fx`、播 `idle_back` 循环、`FinalOffset(1)` → 非专用端造本地前层（`idle_front`、`FinalOffset(-3)`、标签 `FX`＋`CLASSIFIED`、引擎父挂 FX 本体）→ Pristine → 主模拟 `persists=false` | 前层是**非联网本地实体**，专服不存在；FX 自身无 SoundEmitter——`shimmer_chest_lp` 声音循环挂在宝箱宿主上（[宿主生成段](../../../raw/current/scripts/prefabs/treasurechest.lua#L602-L614)）。[工厂](../../../raw/current/scripts/prefabs/treasurechest.lua#L617-L655)、[前层](../../../raw/current/scripts/prefabs/treasurechest.lua#L631-L644)、[注册](../../../raw/current/scripts/prefabs/treasurechest.lua#L771) |
| `townportalsandcoffin_fx` | Transform／AnimState／SoundEmitter／Network → `ObstaclePhysics(0.5)` → bank／build `teleport_sand_fx`、`OverrideSymbol("sand_splash", "sand_splash_fx", "sand_splash")`、播 `portal_in`、`FinalOffset(7)` → 标签 `NOCLICK`／`FX` → Pristine → 主模拟播 `teleport_sand/in`、`persists=false`、挂 `KillFX` 方法并排 35 帧自动收尾任务 | `KillFX`（[实现](../../../raw/current/scripts/prefabs/townportal.lua#L243-L252)）幂等：取消 35 帧任务、停物理、播 `teleport_sand/out`＋`portal_out`，动画长度＋0.5 秒后 `Remove`；实例方法在工厂内挂（[L282-L283](../../../raw/current/scripts/prefabs/townportal.lua#L279-L283)）。[工厂](../../../raw/current/scripts/prefabs/townportal.lua#L254-L286)、[注册](../../../raw/current/scripts/prefabs/townportal.lua#L288-L290) |

## 关联机制导航

### 生成点与宿主条件

| 生成点 | 涉及注册 | 端、条件与调用 | 证据 |
|---|---|---|---|
| `terrarium_master_postinit` 的 `DoTaskInTime(0)` | `terrariumchest_fx` | 主模拟；**双重门禁**：`TUNING.SPAWN_EYEOFTERROR`（"Eye For An Eye"世界选项，默认 true，[tuning](../../../raw/current/scripts/tuning.lua#L6369)；世界设置 `eyeofterror=never` 时置 false，[overrides](../../../raw/current/scripts/worldsettings_overrides.lua#L451-L455)）且宝箱带 `scenariorunner` 组件；生成后 `SetParent(宝箱.entity)`、宿主监听 `onburnt`／`onopen`、播 `shimmer_chest_lp` 循环 | [生成](../../../raw/current/scripts/prefabs/treasurechest.lua#L602-L614) |
| 宝箱的 `scenariorunner` 从哪来 | — | 世界生成静态布局自带 `scenario = "chest_terrarium"` 属性（pigs／fire 变体同理）；`SpawnSaveRecord` 按存档 `scenario` 键挂组件并 `SetScript`；ANR 改造路径直接 `SetScript("chest_terrarium")`；该场景 `OnLoad` 留一个 dummy 陷阱函数，防止组件被立刻移除——这正是微光条件检查所依赖的状态 | [布局属性](../../../raw/current/scripts/map/static_layouts/terrarium_forest_spiders.lua#L88)、[scenario 挂载](../../../raw/current/scripts/mainfunctions.lua#L494-L502)、[retrofit](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L602-L603)、[dummy 保留](../../../raw/current/scripts/scenarios/chest_terrarium.lua#L55-L57) |
| `SGwilson` `entertownportal` `onenter` | `townportalsandcoffin_fx` | 主模拟（玩家状态机）；生成在玩家当前位置；入口是 `ACTIONS.TELEPORT.fn` 的 `GoToState("entertownportal")`，动作由 componentactions 对 `townportal` 标签实体的右键与 `teleporter` 场景节点提供 | [生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16040-L16043)、[动作入口](../../../raw/current/scripts/actions.lua#L3070-L3086)、[动作提供](../../../raw/current/scripts/componentactions.lua#L884-L885)、[场景节点](../../../raw/current/scripts/componentactions.lua#L3011) |
| `SGwilson` `exittownportal_pre` `onenter` | `townportalsandcoffin_fx` | 主模拟；到达侧传送塔处生成（玩家此时 Hide＋无敌）；转入方式是 `teleporter` 组件读 `statemem.teleportarrivestate`（进侧状态里写为 `exittownportal_pre`） | [生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16097-L16098)、[到达状态选择](../../../raw/current/scripts/components/teleporter.lua#L316-L319)、[进侧写入](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16037-L16038) |

`treasurechest` 与 `townportal` 各自的 `prefabs` 依赖表也声明本页注册（[treasurechest.lua:770](../../../raw/current/scripts/prefabs/treasurechest.lua#L770)、[townportal.lua:19](../../../raw/current/scripts/prefabs/townportal.lua#L15-L20)）——声明只作资源预载，不构成生成点。

### 宿主辨析：treasurechest.lua 六个组合宝箱

| 宿主注册 | 是否生成 `terrariumchest_fx` | 说明 |
|---|---|---|
| `terrariumchest` | 是 | 唯一宿主：master_postinit 双重门禁通过即生成（[L598-L615](../../../raw/current/scripts/prefabs/treasurechest.lua#L598-L615)） |
| `treasurechest` | 否 | `regular_master_postinit` 无 FX 逻辑（[注册](../../../raw/current/scripts/prefabs/treasurechest.lua#L766)） |
| `pandoraschest` | 否 | 厄运宝箱，无 FX 逻辑（[注册](../../../raw/current/scripts/prefabs/treasurechest.lua#L768)） |
| `minotaurchest` | 否 | 其 `scenariorunner` 是 `chest_labyrinth` 陷阱链，与微光无关（[L453-L463](../../../raw/current/scripts/prefabs/treasurechest.lua#L453-L463)） |
| `sunkenchest` | 否 | 沉水宝箱，可搬举／装备玩法（[注册](../../../raw/current/scripts/prefabs/treasurechest.lua#L772)） |
| `boat_ancient_container` | 否 | 古船容器，无 FX 逻辑（[注册](../../../raw/current/scripts/prefabs/treasurechest.lua#L773)） |

### 沙棺时序（传送全过程）

1. **进侧** `entertownportal`（[状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16028-L16087)）：onenter 生成沙柱于玩家位置并播 `townportal_enter_pre`；时间线帧 8 无敌＋禁用控制器、帧 18 `Hide` 玩家、帧 26 调 `teleporter:Activate(inst)`——成功则 `fx:KillFX()`（沙柱转出柱）并传送，失败转本地 `exittownportal` 退出；onexit **无条件** `KillFX`（[L16071-L16072](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16071-L16072)）。
2. **出侧** `exittownportal_pre`（[状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16089-L16128)）：在目标传送塔位置再生成一枚沙柱；玩家保持 Hide／无敌 32 帧后转 `exittownportal` 播 `townportal_exit_pst`；onexit `KillFX`（[L16114-L16115](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16114-L16115)）。
3. **收尾**：`KillFX` 幂等（`killtask` 空判），成功路径与 onexit 重复调用不重复播；即使宿主状态异常退出漏调，35 帧兜底任务也会自动转出柱并退场（[L283](../../../raw/current/scripts/prefabs/townportal.lua#L282-L283)）。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `terrariumchest_fx` | ①宝箱 `onburnt`／`onopen` → `terrarium_removefx`：`Remove` FX＋摘除两个监听＋`KillSound("shimmer")`；②宝箱实体被移除时是否随引擎父级回收未核验 | 无自删、`persists=false`；前层无自身清理，实际回收未核验；开过一次即永久消失，无重生路径。[removefx](../../../raw/current/scripts/prefabs/treasurechest.lua#L568-L576)、[父挂](../../../raw/current/scripts/prefabs/treasurechest.lua#L605-L606)、[persists](../../../raw/current/scripts/prefabs/treasurechest.lua#L652) |
| `townportalsandcoffin_fx` | 双路径：35 帧兜底任务或宿主状态 `KillFX` → `portal_out` 动画＋out 音效，动画长度＋0.5 秒后 `Remove` | `persists=false`；`Physics:SetActive(false)` 提前停碰撞；两次 `KillFX` 幂等。[KillFX](../../../raw/current/scripts/prefabs/townportal.lua#L243-L252) |

## 典型接入

### 复刻"宿主单槽引用管理"的常驻跟随 FX

- 前提与选择：想要"随宿主存在、特定事件后消失"的包裹层表现，复刻 `terrariumchest_fx` 模式——联网壳＋宿主事件驱动清理；想要"限时演出后自动退场"的传送表现，复刻沙柱的 `KillFX`＋兜底任务双路径。
- 生效与撤销：FX 生成后 `SetParent` 引擎父挂即可随宿主移动；清理责任在宿主，`inst.fx` 单槽引用同时充当状态标记（getstatus 读它）。
- 原版依据：[宿主生成段](../../../raw/current/scripts/prefabs/treasurechest.lua#L602-L614)、[removefx](../../../raw/current/scripts/prefabs/treasurechest.lua#L568-L576)。

```lua
local function AttachShimmerFX(chest)
    if not TheWorld.ismastersim then
        return
    end
    local fx = SpawnPrefab("terrariumchest_fx")
    fx.entity:SetParent(chest.entity)   -- 引擎父挂：随宿主移动；宿主删除时是否级联回收需另行核验
    chest.fx = fx                        -- 单槽引用：同时是 getstatus 的 SHIMMER 标记
    chest:ListenForEvent("onopen", function(h)   -- 事件驱动清理
        if h.fx ~= nil then
            h.fx:Remove()
            h.fx = nil
        end
    end)
end
```

- 调整与验证：替换点为注册名与监听事件；`shimmer` 音效循环须另挂宿主 SoundEmitter（FX 工厂不含声音）。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：微光层是"一次性"的。** 宿主监听的是 `onopen`（打开即触发）而非关闭——开箱一次 FX 与 shimmer 音效即永久移除；`SHIMMER` 审查状态也随之消失（getstatus 读 `inst.fx`）。想要"重新闭箱恢复微光"必须自行重建整条链。[状态读取](../../../raw/current/scripts/prefabs/treasurechest.lua#L562-L566)
- **边界：世界选项能整体关掉微光。** `eyeofterror=never` 时 `SPAWN_EYEOFTERROR=false`，生成段整个跳过，宝箱永远不显示微光也不播 shimmer 音效；此时宝箱内容物与陷阱玩法不受影响。[门禁](../../../raw/current/scripts/prefabs/treasurechest.lua#L602-L604)
- **边界：沙柱带碰撞。** 与多数 FX 不同，工厂挂 `ObstaclePhysics(0.5)`：传送演出期间沙柱会阻挡其他单位走位；`KillFX` 里 `Physics:SetActive(false)` 才解除。[物理](../../../raw/current/scripts/prefabs/townportal.lua#L262)、[停用](../../../raw/current/scripts/prefabs/townportal.lua#L247)
- **边界：到达侧生成点在玩家状态机。** `exittownportal_pre` 由 `teleporter` 组件按进侧写入的 `teleportarrivestate` 转入；修改到达演出要动 SG 状态而不是 teleporter 组件的固定字段（组件先读 `overrideteleportarrivestate` 再回落 statemem）。[到达选择](../../../raw/current/scripts/components/teleporter.lua#L316-L319)
- **边界：微光前层不在专服存在。** 前层在 `not TheNet:IsDedicated()` 分支创建且父挂 FX 本体；服务端逻辑不应引用它。[前层分支](../../../raw/current/scripts/prefabs/treasurechest.lua#L631-L644)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/treasurechest.lua`（MakeChest 工厂、terrarium 段全部函数、六注册返回块）与 `prefabs/townportal.lua` 全文（本体工厂、KillFX、FX 工厂）；按需读取 `stategraphs/SGwilson.lua` 三个 townportal 状态、`actions.lua` TELEPORT 动作、`componentactions.lua` 两处动作提供、`components/teleporter.lua` 到达状态选择、`scenarios/chest_terrarium.lua`、`map/static_layouts/terrarium_forest_spiders.lua`、`components/retrofitforestmap_anr.lua` 场景挂载、`mainfunctions.lua` SpawnSaveRecord 的 scenario 接线、`tuning.lua`／`worldsettings_overrides.lua` 的 SPAWN_EYEOFTERROR。对两个注册名以 `grep -rn "注册名" --include=*.lua` 在 `raw/current/scripts` 全域反查：`terrariumchest_fx` 命中注册（treasurechest.lua:771）、依赖表（:770）、生成点（:605）、工厂内字符串（:625-641）与 scenarios/chest_terrarium.lua:55 注释——**生成点唯一**；`townportalsandcoffin_fx` 命中注册（townportal.lua:290）、依赖表（:19）与 SGwilson.lua:16042／16097 两处生成——**无第三生成点**。`SPAWN_EYEOFTERROR=true`（tuning.lua:6368）、`EYEOFTERROR_SPAWNDELAY`＝15 天已核对。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.TERRARIUMCHEST`＝"显眼箱子"（[L350364-L350367](../../../raw/current/scripts/languages/chinese_s.po#L350364-L350367)）、`STRINGS.NAMES.TOWNPORTAL`＝"懒人传送塔"（[L350494-L350497](../../../raw/current/scripts/languages/chinese_s.po#L350494-L350497)）、`TERRARIUMCHEST.SHIMMER` 描述＝"那个看起来有点格格不入……"（[L20694-L20697](../../../raw/current/scripts/languages/chinese_s.po#L20694-L20697)）、`ACTIONS.TELEPORT.TOWNPORTAL`＝"触摸"（[L2139-L2142](../../../raw/current/scripts/languages/chinese_s.po#L2139-L2142)）；`TERRARIUMCHEST_FX`／`TOWNPORTALSANDCOFFIN_FX` 无 `STRINGS.NAMES.*` 键，正文"微光／沙柱"为效果描述。
- 分档说明：`size_class: standard`——2 注册同性质（宿主管理生命期），复杂度集中在组合宝箱宿主辨析与沙棺时序两个小节，标准档可承载，不需要整体拆页。
- 未覆盖：`terrarium` 物品激活与恐怖之眼（Eye of Terror）召唤玩法、`eyeofterror` 其余难度档（rare 等）数值、传送塔 channelable 频道玩法与 `linktownportals` 配对、`chest_terrarium_pigs`／`chest_terrarium_fire` 场景差异（共用 loot 表）、宝箱其余五个宿主的玩法、动画资源内容。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/fx_book_mount|书页覆盖与骑乘挂载特效族]]（同为双层前后覆盖结构；对照 front/back 分层写法）
- [[reference/prefabs/pocketwatch_warp_marker|怀表与传送门附属表现 FX 族]]（同为传送主题的进出两侧 FX 分工）
- 返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
