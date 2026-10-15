---
title: "龙舟竞赛旗帜、烟花与起跑浮标（boatrace_start_flag、boatrace_fireworks、boatrace_checkpoint_flag、boatrace_start_bobber）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "龙舟竞赛（起点塔／检查点玩法）外围的 3 个联网 FX 与 1 个标记实体：终点名次旗与夺冠烟花（跟随起点塔符号）、检查点缎带旗（跟随检查点符号）、起跑环 10 枚浮标（视觉环兼码头放置阻挡），以及起点塔／检查点两个竞赛控制器的触发链辨析。"
aliases: ["boatrace_start_flag", "boatrace_fireworks", "boatrace_checkpoint_flag", "boatrace_start_bobber", "龙舟竞赛旗帜", "竞赛烟花", "检查点缎带", "起跑浮标", "起点塔旗帜", "龙年竞速特效"]
tags: [prefab, fx, building, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/boatrace_start.lua", "prefabs/boatrace_checkpoint.lua", "prefabs/boatrace_common.lua", "stategraphs/SGboatrace_start.lua", "components/map.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 龙舟竞赛旗帜、烟花与起跑浮标（boatrace_start_flag、boatrace_fireworks、boatrace_checkpoint_flag、boatrace_start_bobber）

## 快速判断

- 职责与适用：龙舟竞赛（起点塔＋检查点的世界活动设施）外围的 **3 个纯 FX＋1 个标记实体**——①`boatrace_start_flag` 是终点塔上的名次旗（1～3 名三面，跟随 `flag1/2/3` 符号，图案按名次换）；②`boatrace_fireworks` 是夺冠烟花（一次性 `fireworks` 动画，火花符号按名次换）；③`boatrace_checkpoint_flag` 是检查点被触及后挂出的缎带旗（跟随 `ribbon_marker1..8` 符号，缎带编号按选手）；④`boatrace_start_bobber` 是起点塔周围半径 12 的 **10 枚起跑环浮标**——联网标记实体，视觉环之外带 `dockjammer` 玩法作用（阻挡码头放置）。[旗/烟花/浮标注册](../../../raw/current/scripts/prefabs/boatrace_start.lua#L1091-L1096)、[缎带旗注册](../../../raw/current/scripts/prefabs/boatrace_checkpoint.lua#L301-L304)
- 本体辨析：`boatrace_start`（官方名"起点塔"，[po](../../../raw/current/scripts/languages/chinese_s.po#L339674-L339677)）与 `boatrace_checkpoint`（官方名"比赛检查点"，[po](../../../raw/current/scripts/languages/chinese_s.po#L339634-L339637)）是竞赛的玩法控制器——SG 状态机、激活、计名次、发奖都在本体上，本页只辨析触发链；两个"套装"投掷物品（`boatrace_start_throwable_deploykit`／`boatrace_checkpoint_throwable_deploykit`）由 [boatrace_common 的套件原型](../../../raw/current/scripts/prefabs/boatrace_common.lua#L209-L301) 动态注册（名字＝产物名＋`_throwable_deploykit`），是部署道具不是 FX。
- 重要副作用：三个 FX 全部 `persists=false` 不存档，靠宿主符号跟随、由宿主事件驱动；浮标同样 `persists=false`，但其 `dockjammer` 标签会被码头部署检查读取（[map.lua 检查一](../../../raw/current/scripts/components/map.lua#L492-L494)、[检查二](../../../raw/current/scripts/components/map.lua#L534-L536)）——**浮标在场时竞赛区不能修码头**。
- 修改边界：只覆盖 4 注册的工厂、生成点、父挂／跟随与清理；起点塔计名次／影子船／奖品袋（`redpouch_yotd`）、观赛小龙蝇（`boatrace_spectator_dragonling`）、浮标指示器（`boatrace_checkpoint_indicator`，注册于其独立文件）、影子龙蝇船等竞赛玩法实体只作链条辨析。
- 名称：4 个覆盖注册名在简中 `chinese_s.po` 均无 `STRINGS.NAMES` 键（检索 `NAMES.BOATRACE` 仅命中起点塔／检查点／套装／浮标指示器等本体名），标题中文为已核验效果描述，非官方译名；宿主 `DRAGONBOAT_BODY`＝"龙蝇船"（[po](../../../raw/current/scripts/languages/chinese_s.po#L342174-L342176)）。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `boatrace_start_flag` | `flagfn`：Network／Transform／AnimState／Follower → 标签 `FX` → bank=`boatrace_start`、build=`boatrace_start`、循环 `flag_idle` → `highlightchild` 组件 → `animover` 监听（仅当前动画为 `flag_pst` 时自删）→ `persists=false`。 | **无 `SetPristine` 分界**：联网实体两端走同一工厂路径。旗帜图案来自 `boatrace_start_flag` 动画包的 `swapflag_1..3` 符号（生成方 OverrideSymbol 写入），实体自身 build 是起点塔。[工厂](../../../raw/current/scripts/prefabs/boatrace_start.lua#L1045-L1066)、[退场判断](../../../raw/current/scripts/prefabs/boatrace_start.lua#L627-L631)、[注册](../../../raw/current/scripts/prefabs/boatrace_start.lua#L1095) |
| `boatrace_fireworks` | `fireworksfn`：同旗工厂壳（Network／Transform／AnimState／Follower、`FX`、bank=build=`boatrace_start`、`highlightchild`），播一次性 `fireworks`，`animover` 直接自删，`persists=false`。 | 无 SetPristine 分界；火花贴图 `fx_spark_parts_00` 由生成方按名次 override 成 `fx_spark_parts_01..03`。[工厂](../../../raw/current/scripts/prefabs/boatrace_start.lua#L1068-L1089)、[注册](../../../raw/current/scripts/prefabs/boatrace_start.lua#L1096) |
| `boatrace_checkpoint_flag` | `flagfn`：同壳（bank=build=`boatrace_checkpoint`、循环 `ribbon_loop`、`highlightchild`），`animover` 仅在当前动画为 `ribbon_pst` 时自删，`persists=false`。 | 缎带图案来自 `boatrace_checkpoint_ribbon` 动画包的 `ribbon1..3` 符号（生成方 override）；同样无 SetPristine 分界。[工厂](../../../raw/current/scripts/prefabs/boatrace_checkpoint.lua#L258-L283)、[注册](../../../raw/current/scripts/prefabs/boatrace_checkpoint.lua#L304) |
| `boatrace_start_bobber` | `bobberfn`：Transform／AnimState／Network → 标签 `dockjammer`＋`NOCLICK` → 物理半径 override 0.5 → bank=build=`boatrace_start_bobber`、**地底层 `LAYER_BELOW_GROUND`**、FinalOffset -1、播 `place`＋`idle_loop`、随机起始帧 → `SetPristine` → 客户端早退 → 服务端 `persists=false`。 | 联网标记实体（**非纯 FX**）：无交互组件，`dockjammer` 有玩法作用；工厂有标准 SetPristine 分界（与三个旗／烟花 FX 不同）。[工厂](../../../raw/current/scripts/prefabs/boatrace_start.lua#L945-L973)、[注册](../../../raw/current/scripts/prefabs/boatrace_start.lua#L1094) |

## 关联机制导航

### 起点塔链：名次旗与烟花什么时候挂

起点塔激活（`activatable` → `OnActivated`）后收集 `MAX_BOATRACE_COMPONENT_DISTANCE`(30) 连通域内的检查点、进入 `fuse_on` 点火倒计时；倒计时结束 `do_event_start` 给每艘在 `BOATRACE_START_INCLUSION_PROXIMITY`(12) 内载有玩家的船 `AddChild` 一枚 `boatrace_checkpoint_indicator` 浮标指示器（[updateloop 收船](../../../raw/current/scripts/prefabs/boatrace_start.lua#L253-L330)、[开赛](../../../raw/current/scripts/prefabs/boatrace_start.lua#L398-L474)、[调值](../../../raw/current/scripts/tuning.lua#L8145-L8155)）。选手浮标经检查点时，检查点 `OnBeaconAtCheckpoint` 生成缎带旗并把 `beacon_reached_checkpoint` 推回起点塔（[检查点触及](../../../raw/current/scripts/prefabs/boatrace_checkpoint.lua#L98-L115)）；起点塔 `do_set_placing` 记名次后 `testforwinpresentation` 调 `setflag`——**每次名次产生都生成一面名次旗并把本体推进 `win` 状态**（[记名次](../../../raw/current/scripts/prefabs/boatrace_start.lua#L657-L671)、[挂旗](../../../raw/current/scripts/prefabs/boatrace_start.lua#L634-L655)）。`win` 状态动画播完回调 `winOver`：生成 `boatrace_fireworks` 于塔位、按 `winid`（名次）override 火花符号，并往选手落点投掷奖品袋（[win 回调](../../../raw/current/scripts/stategraphs/SGboatrace_start.lua#L77-L95)、[烟花与奖品](../../../raw/current/scripts/prefabs/boatrace_start.lua#L573-L625)）。

缎带旗侧（检查点）：`setflag` 从 `flag_positions`（1..8）随机取一个未用挂点，OverrideSymbol 写缎带编号（=选手浮标 `_index`）、播 `ribbon_pre`＋`ribbon_loop`、`SetParent`＋`FollowSymbol("ribbon_marker"..position)`、`highlightchild` 认主（[检查点挂旗](../../../raw/current/scripts/prefabs/boatrace_checkpoint.lua#L75-L95)）。竞赛结束 `boatrace_finish` 事件 → `OnRaceOver` 延迟 1~2 秒 → `ResetCheckpoint` 给所有已挂缎带播 `ribbon_pst`（[重置](../../../raw/current/scripts/prefabs/boatrace_checkpoint.lua#L126-L143)、[比赛结束](../../../raw/current/scripts/prefabs/boatrace_checkpoint.lua#L146-L149)）。

### 起跑环浮标：生成、预览与玩法作用

起点塔工厂第 0 帧任务 `CLIENT_OnInit`：两端把自身注册进 `TheWorld._boatrace_starts`（[注册](../../../raw/current/scripts/prefabs/boatrace_common.lua#L5-L13)），**主模拟**再调 `CreateBobberRing(inst, SERVER_DeployBobber, true)`——以 `BOATRACE_START_INCLUSION_PROXIMITY`(12) 为半径、每 36° 一枚共 10 枚，逐枚随机延迟落 `SpawnPrefab("boatrace_start_bobber")` 并 `AddChild` 到塔上（[环生成](../../../raw/current/scripts/prefabs/boatrace_start.lua#L799-L822)、[服务端落浮标](../../../raw/current/scripts/prefabs/boatrace_start.lua#L791-L797)）。放置预览（套装 placer）走另一条本地路径：`CLIENT_DeployBobber` 用 `CreateEntity("boatrace_start_bobber")` 造**非网络本地实体**（`CLASSIFIED/DECOR/NOBLOCK/NOCLICK` 标签），这些同名本地实体不是注册 prefab，仅作摆位预览（[本地预览](../../../raw/current/scripts/prefabs/boatrace_start.lua#L750-L789)）。浮标的 `dockjammer` 由 `Map` 的码头部署检查消费：码头目标瓦片上的实体带该标签即拒绝（[消费一](../../../raw/current/scripts/components/map.lua#L492-L494)、[消费二](../../../raw/current/scripts/components/map.lua#L534-L536)）。

起点塔自身布置时另有一个自动撒检查点玩法：建成 33 帧后向四周海面投掷 3 枚 `boatrace_checkpoint_throwable_deploykit`（抛物线、`checkpoint_throw` 状态），落点合法即变成检查点（[自动撒点](../../../raw/current/scripts/prefabs/boatrace_start.lua#L73-L112)、[套件落地](../../../raw/current/scripts/prefabs/boatrace_common.lua#L237-L274)）。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `boatrace_start_flag` | 无直接移除调用：竞赛重置 `reset_boatrace` 给所有旗播 `flag_pst`，旗自身的 `animover` 监听只在当前动画为 `flag_pst` 时 `Remove` | 旗在比赛全程 `flag_idle` 循环，只有重置这一条退场路径；`reset_boatrace` 同时是奖励发完／中途失败／无检查点的统一收尾（[重置播 pst](../../../raw/current/scripts/prefabs/boatrace_start.lua#L168-L174)、[判断](../../../raw/current/scripts/prefabs/boatrace_start.lua#L627-L631)） |
| `boatrace_fireworks` | 工厂 `animover` 自删（一次性动画播完即走） | 无宿主引用管理；`persists=false`。[自删](../../../raw/current/scripts/prefabs/boatrace_start.lua#L1084) |
| `boatrace_checkpoint_flag` | `ResetCheckpoint` 逐旗播 `ribbon_pst` → `animover` 判断自删 | 与起点旗同构：退场动画由宿主触发、工厂只认 `ribbon_pst`（[播 pst](../../../raw/current/scripts/prefabs/boatrace_checkpoint.lua#L130-L134)、[判断](../../../raw/current/scripts/prefabs/boatrace_checkpoint.lua#L274-L278)） |
| `boatrace_start_bobber` | 无任何显式移除点（全文件检索 `_bobbers` 只有登记与清表）；`persists=false` 不存档，靠宿主（起点塔）移除时的引擎父子级联消失；读档后由 `CLIENT_OnInit` 重新铺环 | 塔不拆则浮标常驻（意味着竞赛区持续禁修码头）；flag 的 `ListenForEvent("onremove")` 只是清理宿主的引用表，不删实体（[登记](../../../raw/current/scripts/prefabs/boatrace_start.lua#L795-L797)） |

## 典型接入

### 给联网宿主做"符号跟随＋宿主触发退场"的挂件 FX

- 前提与选择：宿主是联网实体且动画里有目标符号；FX 工厂不带 SetPristine 分界（两端同构）、加 `highlightchild` 让高亮随宿主；退场约定"宿主播 `*_pst` → FX 的 animover 判断当前动画后自删"。起点旗／检查点缎带共用这套约定。
- 生效与撤销：`SpawnPrefab` → OverrideSymbol 换图案 → `SetParent`＋`FollowSymbol(宿主GUID, 符号)` → `highlightchild:SetOwner(宿主)`；撤销由宿主在重置时统一播 `pst`（[起点塔挂旗](../../../raw/current/scripts/prefabs/boatrace_start.lua#L636-L642)、[检查点挂旗](../../../raw/current/scripts/prefabs/boatrace_checkpoint.lua#L86-L92)）。

```lua
-- 服务端：挂一面跟随宿主符号的旗（对应起点塔 setflag）
local flag = SpawnPrefab("boatrace_start_flag")
flag.AnimState:OverrideSymbol("swapflag_1", "boatrace_start_flag", "swapflag_" .. id)
flag.AnimState:PlayAnimation("flag_pre", false)
flag.AnimState:PushAnimation("flag_idle", true)
flag.entity:SetParent(inst.entity)
flag.Follower:FollowSymbol(inst.GUID, "flag" .. position, nil, nil, nil, true)
flag.components.highlightchild:SetOwner(inst)
```

- 调整与验证：替换点是注册名、符号名与图案 build；图案 OverrideSymbol 的第二参决定换装来源。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：`boatrace_start_bobber` 是纯装饰。** 它带 `dockjammer`，起点环内的瓦片不能修码头（消费端在 `Map` 的两个部署检查）；同时又 `NOCLICK`＋无组件——它"有玩法作用但不可交互"，页面按联网标记实体归类。[消费端](../../../raw/current/scripts/components/map.lua#L492-L494)
- **边界：浮标没有主动移除路径。** 想清浮标只能拆起点塔（锤 3 下）；模组若动态删环需自己持有引用并 `Remove`。[无移除点](../../../raw/current/scripts/prefabs/boatrace_start.lua#L791-L797)
- **边界：三个旗／烟花 FX 无 SetPristine 分界。** 联网实体但工厂两端全量执行；别在工厂里写依赖 `TheWorld.ismastersim` 分叉的逻辑（原版没有），浮标工厂反而有标准分界（[对比](../../../raw/current/scripts/prefabs/boatrace_start.lua#L1045-L1096)）。
- **边界：旗与缎带的图案都是"符号 override"而非独立 build。** 实体 build 恒为宿主动画包（`boatrace_start`／`boatrace_checkpoint`），图案来自另两个动画包的 swap／ribbon 符号；直接改 build 会丢宿主形状。[图案来源](../../../raw/current/scripts/prefabs/boatrace_start.lua#L637)
- **边界：名次旗每次名次生成一面，烟花每次颁奖放一发。** `setflag` 在 `win` 状态回调链里被名次推进反复调用（最多 3 面），`winOver` 每次颁奖都可能再放烟花；旗帜不去重靠 `#inst.flags < 3` 门禁（[门禁](../../../raw/current/scripts/prefabs/boatrace_start.lua#L181-L194)）。
- **边界：竞赛重置只清 FX，不清浮标。** `reset_boatrace` 撇清 beacons／flags／奖品状态，但起跑环浮标与塔共存亡；中途失败（无检查点／无船）也会走同一条重置链（[重置](../../../raw/current/scripts/prefabs/boatrace_start.lua#L127-L179)）。
- **边界：检查点缎带旗挂点随机且不复用。** `flag_positions` 初始 1..8，`setflag` 随机抽一个并移除；重置时才补回 8 个挂点（[挂点表](../../../raw/current/scripts/prefabs/boatrace_checkpoint.lua#L142)）。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/boatrace_start.lua`（六个注册、浮标族全部函数、竞赛控制器主体）、`prefabs/boatrace_checkpoint.lua` 全文、`prefabs/boatrace_common.lua`（套件原型／部署辅助／注册表）、`stategraphs/SGboatrace_start.lua` 全部状态；按需读取 `components/map.lua` 的两个 `dockjammer` 消费点、`tuning.lua` 的 BOATRACE 调值段。对 4 个注册名以完整名在 `raw/current/scripts` 全域反查：`boatrace_start_flag` 命中注册（boatrace_start.lua:1095）、工厂（:1045）、生成（:636 setflag）——**生成点唯一**；`boatrace_fireworks` 命中注册（:1096）、工厂（:1068）、生成（:575 winOver）——生成点唯一；`boatrace_checkpoint_flag` 命中注册（boatrace_checkpoint.lua:304）、依赖（:21）、工厂（:258）、生成（:86 setflag）——生成点唯一；`boatrace_start_bobber` 命中注册（boatrace_start.lua:1094）、工厂（:945）、生成（:792 SERVER_DeployBobber）与同名本地实体（:751 `CreateEntity`，非注册）；`_bobbers` 仅本文件内出现，无外部移除者。台账批 2"3 个纯 FX"复核属实，`boatrace_start_bobber` 按实归类为联网标记实体（`dockjammer` 玩法作用）。未运行游戏。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.BOATRACE_START`＝"起点塔"、`STRINGS.NAMES.BOATRACE_CHECKPOINT`＝"比赛检查点"（锚点见"快速判断"）；`NAMES.BOATRACE_*` 无 FLAG／FIREWORKS／BOBBER 键，4 个覆盖注册名标题使用效果描述。
- 分档说明：`standard`——4 注册两条宿主链（起点塔／检查点）＋一条浮标链；无多变体状态机，各链单层生成／清理。
- 未覆盖：竞赛计名次／影子船／奖品袋与 `yotd_raceprizemanager` 全链、浮标指示器与观赛小龙蝇本体、套装投掷物品的库存与漂浮细节、动画包内容与观感、客户端对无 SetPristine 联网 FX 的复制细节；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- 返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]
