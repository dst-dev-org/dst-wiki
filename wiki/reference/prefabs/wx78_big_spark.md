---
title: "WX-78 电火花、照明光柱与扫描底盘特效族（wx78_big_spark 等 4 注册）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "WX-78 技能与设备外围的 4 个 FX 注册：掉电/受击的大型电火花（联网代理壳＋本地发光动画双层，代理无 AnimState 使其闪白分支不可达）、照明电路的光柱管理实体（自身无动画、按净变量维护本地灯点链并带客户端移动预测）、扫描仪打在被扫生物身上的联网扫描束（AddChild 到目标）与备份底盘的机体渲染层（官方名即'备份底盘'的联网动画子实体）。"
aliases: ["wx78_big_spark", "wx78_lightbeam", "wx78_scanner_fx", "wx78_backupbody_inventory", "WX-78大型电火花", "WX-78照明光柱", "照明电路", "扫描仪扫描束", "备份底盘渲染层", "WX-78扫描仪", "生物扫描分析仪", "备份底盘"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/wx78_big_spark.lua", "prefabs/wx78_lightbeam.lua", "prefabs/wx78_scanner.lua", "prefabs/wx78_backupbody.lua", "prefabs/wx78.lua", "prefabs/wx78_possessedbody.lua", "wx78_moduledefs.lua", "prefabs/wx78_common.lua", "entityscript.lua", "mainfunctions.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# WX-78 电火花、照明光柱与扫描底盘特效族（wx78_big_spark 等 4 注册）

## 快速判断

- 职责与适用：4 个注册分三类载体——①`wx78_big_spark`（大型电火花：WX-78 掉电/受击时的联网代理壳＋非专服本地发光动画，3 个宿主文件共用一个包装函数）、②`wx78_lightbeam`（照明电路的光柱管理实体：自身无 AnimState，按 light_radius 净变量在宿主周围维护本地灯点链）、③`wx78_scanner_fx`（生物扫描分析仪打在被扫目标身上的联网扫描束，AddChild 挂到目标）、④`wx78_backupbody_inventory`（备份底盘的机体渲染层：官方名"备份底盘"的联网动画子实体，本体 wx78_backupbody 自身无 AnimState）。
- 优先使用：查"WX-78 冒电火花/掉电闪一下"用 big_spark 生成点表；查"照明电路的光圈移动/灯点数量"用 lightbeam；查"扫描仪锁定目标身上的扫描束"用 scanner_fx；查"备份底盘怎么显示/换肤/存档"用 backupbody_inventory。
- 重要副作用：`wx78_big_spark` 的代理壳**无 AnimState**，`AlignToTarget` 里的闪白循环在首次更新即自拆（[判定](../../../raw/current/scripts/prefabs/wx78_big_spark.lua#L86-L90)）——台账结论复核成立，只剩本地子体的动画＋灯光＋一次音效；`wx78_lightbeam` 的 AnimState 添加代码整段被注释（[注释](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L156-L158)），是纯灯点管理器；`wx78_scanner_fx` 未置 `persists=false`，靠引擎父级逃过存档循环节。
- 修改边界：只覆盖 4 注册的工厂、生成点、父子/跟随与清理；扫描仪玩法（proximityscan 节拍、SGwx78_scanner、brain）、备份底盘附身转移全流程、WX-78 技能树与模块体系、wx78_classified 同步不展开。
- 名称：官方名 `WX78_BACKUPBODY_INVENTORY`＝"备份底盘"（[po](../../../raw/current/scripts/languages/chinese_s.po#L352664-L352667)）、宿主 `WX78_SCANNER`＝"生物扫描分析仪"（[po](../../../raw/current/scripts/languages/chinese_s.po#L352724-L352727)）、light 模块 `WX78MODULE_LIGHT`＝"照明电路"（[po](../../../raw/current/scripts/languages/chinese_s.po#L352564-L352567)）；`WX78_BIG_SPARK`／`WX78_LIGHTBEAM`／`WX78_SCANNER_FX` 无 `STRINGS.NAMES.*` 键，标题中文为已核验效果描述，**非官方译名**。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wx78_big_spark` | 代理壳 `fn`：Transform／Network（**无 AnimState**）、标签 `FX`；非专服 `DoTaskInTime(0, StartFX)`；主模拟 `persists=false`＋`DoTaskInTime(1, Remove)` 1 秒硬寿命＋`AlignToTarget` 方法 | 本地子体由 `StartFX` 创建：只复制代理壳已有的引擎父级（壳无父级则保持世界坐标）、`SetFromProxy` 对位，AnimState `wx_fx/big_shock`＋泛光，Light 橙色强度 0.9 按 1/20 秒衰减 2/秒，非专服 SoundEmitter 播一次 `WX_rework/shock/big`。[代理壳](../../../raw/current/scripts/prefabs/wx78_big_spark.lua#L134-L157)、[本地子体](../../../raw/current/scripts/prefabs/wx78_big_spark.lua#L34-L72)、[注册](../../../raw/current/scripts/prefabs/wx78_big_spark.lua#L159) |
| `wx78_lightbeam` | `fn`：Transform／Network，标签 `FX`＋`NOCLICK`；`light_rotation`（net_shortint）＋`light_radius`（net_smallbyte）；主模拟 `persists=false`＋`AttachToOwner`／`SetLightRadius` 方法；客户端 `OnEntityReplicated` 从引擎父级取宿主 | 灯点链 `CreateLight`：本地实体仅 Transform＋Light（无 AnimState），标签含 `staysthroughvirtualrooms`；`CreateLight` 先把传入序号加 2 存为 `inst.i`，半径按 `sqrt(light_radius*0.08)*inst.i` 逐点放大；数量/间距由公式重算（见导航）。[工厂](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L153-L190)、[灯点](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L7-L39)、[注册](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L192) |
| `wx78_scanner_fx` | `scanfx_fn`：Transform／AnimState／Network；bank `scanner`、build `wx_scanner`、`scan_fx_pre`→`scan_fx_loop` 循环、`LAYER_BACKGROUND`；标签 `FX`；主模拟挂 `goAway` 方法（播 `scan_fx_pst` 后 `animover` 自删） | 未置 `persists=false`：生成后 `AddChild` 挂到被扫生物，存档循环节跳过带引擎父级的实体（[mainfunctions.lua:1114](../../../raw/current/scripts/mainfunctions.lua#L1114)），随目标存档语义成立。[工厂与注册](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L1183-L1222)、[goAway](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L1183-L1186) |
| `wx78_backupbody_inventory` | `fn_inventory`：Transform／SoundEmitter／AnimState／DynamicShadow／Network；标签 `equipmentmodel`＋`FX`；bank `wilson`＋build `wx78`＋`wx_chassis` 覆盖层、播 `wx_chassis_idle`；隐藏 mimic/trapper/gestalt 等符号；挂 `gestaltfxflags` 净变量（AddGestaltFx_Common）与 `steamfx` net_event（无朝向版热蒸汽）；主模拟加 inventory（maxslots=0，仅承载装备）＋skinner | **混合载体**：带 FX 标签但是备份底盘本体的渲染层——本体 `fn` 只有 Transform/SoundEmitter/Light/Network 无 AnimState，高亮与可 haunted 动画都借渲染层（[AnimStateGetterFn](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L793-L795)）。[工厂](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L1018-L1066)、[注册](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L1136) |

## 关联机制导航

### wx78_big_spark 的 6 处调用（3 个宿主文件各一个包装函数）

`SpawnPrefab("wx78_big_spark"):AlignToTarget(inst)` 在三个宿主文件里各自包成 `SpawnBigSpark`：[wx78.lua:54-56](../../../raw/current/scripts/prefabs/wx78.lua#L54-L56)、[wx78_possessedbody.lua:21-23](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L21-L23)、[wx78_backupbody.lua:27-29](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L27-L29)。

| 调用点 | 触发条件 | 证据 |
|---|---|---|
| wx78.lua:300 | 进水扣血 tick：非 invisible 且可见才闪，随后推 `wx78_spark` | [L299-L302](../../../raw/current/scripts/prefabs/wx78.lua#L299-L302) |
| wx78.lua:377 | `OnFrozen`：非冻结状态下被冻结，扣充电 | [L375-L383](../../../raw/current/scripts/prefabs/wx78.lua#L375-L383) |
| wx78.lua:432 | 玩家 `OnAllUpgradeModulesRemoved`（模块被弹光） | [L431-L434](../../../raw/current/scripts/prefabs/wx78.lua#L431-L434) |
| wx78.lua:499 | 饥饿掉电 tick：非死亡/无敌且可见才闪 | [L494-L503](../../../raw/current/scripts/prefabs/wx78.lua#L494-L503) |
| wx78_possessedbody.lua:380 | 附身底盘 `OnAllUpgradeModulesRemoved` | [L378-L381](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L378-L381) |
| wx78_backupbody.lua:720 | 备份底盘 `OnAllUpgradeModulesRemoved`，且仅当未被拆毁（workable 为空或还有剩余工作量） | [L718-L721](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L718-L721) |

`AlignToTarget` 把代理壳定位到宿主 `(x, 0, z)` 并挂 updatelooper 闪白循环（[L116-L132](../../../raw/current/scripts/prefabs/wx78_big_spark.lua#L116-L132)），但该循环依赖 AnimState（见陷阱）。原版链只在目标当时脚底生成世界位置爆点，不 `SetParent`，也不持续跟随目标；`StartFX` 只在壳已有父级时复用该父级，是条件能力，不是原版已跟随宿主。

### wx78_lightbeam 的灯点链与客户端预测

- 宿主：照明电路模块（`wx78module_light`）。`light_skill_update` 在 `_lightmodule_radius > 0` 且技能 `wx78_circuitry_betabuffs_2` 激活时生成并 `AttachToOwner`，否则移除（[moduledefs L793-L806](../../../raw/current/scripts/wx78_moduledefs.lua#L793-L806)）；模块装/卸经 `light_activate`／`light_deactivate` 增减半径（[L808-L816](../../../raw/current/scripts/wx78_moduledefs.lua#L808-L816)），每模块 [TUNING.WX78_LIGHT_RADIUS_PER_MODULE＝1.25](../../../raw/current/scripts/tuning.lua#L9418-L9418)。同时调宿主本体 Light 的半径/颜色，光柱是额外视觉层。
- 灯点数量与间距：`num_lights = ceil(sqrt(radius*2+32))`、`space_between = 0.5*radius^0.8`、第 i 点 `x_offset = 0.1 + i + space_between*i`（[UpdateLights](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L102-L123)）；灯半径另用 `CreateLight` 保存的 `i+2`（`inst.i`），即第 j 个灯点为 `sqrt(light_radius*0.08)*(j+2)`；每帧沿宿主旋转 0.75 插值摆动并按偏移落位（[OnUpdate](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L87-L100)）。
- 端差异：主模拟直接读宿主朝向并写 `light_rotation` 净变量；客户端走 `GetRemotePredictPositionExternal` 的移动预测（含停止后 2 秒惯性）再回读净变量（[GetTargetRotation](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L48-L85)）。灯点实体永远只存在于本地。

### wx78_scanner_fx 的 AddChild 目标语义

`StartScanFX` 在扫描仪锁定目标后生成扫描束并 `target:AddChild(inst.scan_fx)`，按目标物理半径 `Remap(0→5, 0.5→8)` 缩放（[L553-L563](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L553-L563)）。扫描束挂在**被扫生物**身上随其移动；扫描仪侧只用单槽引用 `inst.scan_fx` 管理：`OnUpdateScanCheck` 每帧启动（[L437](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L414-L449)），失败/成功/目标消失走 `StopScanFX`（[L565-L571](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L565-L571)）→ `goAway` 播 `scan_fx_pst` 后 `animover` 自删（[L1183-L1186](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L1183-L1186)）。同文件另有客户端本地扫描圈 `CreateRingFX`（由 `showringfx` 净变量驱动，[L625-L666](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L625-L666)），**不是注册 prefab**，勿与本注册混同。wx78_scanner.lua 同文件还注册 `wx78_scanner_item`／`wx78_scanner`／`wx78_scanner_succeeded` 等（本页不覆盖）。

### wx78_backupbody_inventory 的渲染层与存档替换

- 生成：本体 master 初始化 `SpawnPrefab("wx78_backupbody_inventory")` 后引擎 `SetParent`，登记进 `highlightchildren` 并回填 `steamfx` 引用（[L912-L917](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L912-L917)）；客户端 `OnEntityReplicated` 反向登记（[L1004-L1016](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L1004-L1016)），移除时自摘（[L995-L1002](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L995-L1002)）。
- 驱动：受击动画（`wx_chassis_hit`）由本体 `OnHit` 写渲染层 AnimState（[L51-L59](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L51-L59)），建造动画（`wx_chassis_place`）由 `OnBuiltFn` 写（[L208-L215](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L208-L215)）；附身状态的 gestalt 层开关经 `ConfigurePossessed` 调渲染层的净变量方法（[L740-L748](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L740-L748)、[wx78_common L553-L571](../../../raw/current/scripts/prefabs/wx78_common.lua#L553-L571)）。
- 存档辨析：本体 `OnSave` 把渲染层 `GetSaveRecord` 嵌进 `data.body_inventory`（[L831](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L831)）；`OnLoad` 时**先 Remove 再 `SpawnSaveRecord` 全量替换**并重新父挂（[L838-L849](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L838-L849)）——皮肤/装备状态随记录走，外部引用必须经 `inst.wx78_backupbody_inventory` 字段重取。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `wx78_big_spark` | 双层各自收尾：本地子体靠强度衰减到 0（`killfx` 时 Remove）＋`animover` 协同（[L6-L22](../../../raw/current/scripts/prefabs/wx78_big_spark.lua#L6-L22)、[L24-L31](../../../raw/current/scripts/prefabs/wx78_big_spark.lua#L24-L31)）；代理壳 1 秒硬寿命 `DoTaskInTime(1, Remove)` | 都 `persists=false`；`animover` 先到时置 `killfx` 让灯光淡完再删，反之直接删。[寿命](../../../raw/current/scripts/prefabs/wx78_big_spark.lua#L151-L152) |
| `wx78_lightbeam` | 灯点链由 `LightBeam_OnRemoveEntity` 全部 Remove（[L41-L45](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L41-L45)）；beam 实体由宿主 `light_skill_update` 显式移除，或宿主移除时是否随引擎级联未核验 | 灯点是 beam 的本地实体，由 beam 的 `OnRemoveEntity` 显式清；`persists=false`。[级联边界](../../../raw/current/scripts/entityscript.lua#L1779-L1783) |
| `wx78_scanner_fx` | 正常走 `goAway`（`animover` 自删）；目标实体 `Remove` 时由 `AddChild` 的 Lua `children` 删除链回收，仅死亡不触发；扫描仪 `OnRemove` 也兜底 `StopScanFX`（[L840-L843](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L840-L843)） | 无 `persists=false` 但恒有父级，存档循环节跳过（[mainfunctions.lua:1114](../../../raw/current/scripts/mainfunctions.lua#L1114)）。 |
| `wx78_backupbody_inventory` | 无自删：本体移除时是否随引擎父子回收未核验（砸毁 [OnWorked](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L31-L49)、附身转移 `inst:Remove()` [L203](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L162-L206)）；读档走 SpawnSaveRecord 替换重建 | `persists` 未置 false；本体存在时因引擎父级不入档，本体移除后的实际回收未核验。 |

## 典型接入

### 复刻"代理壳＋本地发光子体"的一次性爆点 FX

- 前提与选择：需要在主模拟定位、在客户端表现动画/灯光/音效的一次性 FX 时，用 wx78_big_spark 模式：联网代理壳（仅 Transform/Network）在非专服 `DoTaskInTime(0)` 造本地子体并 `SetFromProxy` 对位。
- 生效与撤销：代理壳自带硬寿命兜底，本地子体按动画/灯光衰减自删；宿主只需 `SpawnPrefab("...")` 后调用对位方法，无需管理引用。
- 原版依据：[代理壳 fn](../../../raw/current/scripts/prefabs/wx78_big_spark.lua#L134-L157)、[StartFX](../../../raw/current/scripts/prefabs/wx78_big_spark.lua#L34-L72)。

```lua
local function SpawnBigSpark(inst)
    SpawnPrefab("wx78_big_spark"):AlignToTarget(inst)
end
```

- 调整与验证：替换点是注册名与 `AlignToTarget` 的落点规则（宿主脚底 `y=0`）；若复刻者给代理壳加了 AnimState，`AlignToTarget` 的闪白分支会真正生效，注意行为分叉。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：wx78_big_spark 会把目标闪白。** 闪白循环（OnUpdateFlash/OnRemoveFlash，[L76-L114](../../../raw/current/scripts/prefabs/wx78_big_spark.lua#L76-L114)）写在代理壳上，而代理壳只有 Transform/Network；`AlignToTarget` 挂上 updatelooper 后首次更新即命中 `inst.AnimState == nil` 分支自拆，colouradder/freezable 染色路径都不会执行。
- **边界：电火花音效只在非专服、只播一次。** SoundEmitter 挂在本地子体上（专服没有），onupdate 首帧播 `WX_rework/shock/big` 后清标志（[L6-L10](../../../raw/current/scripts/prefabs/wx78_big_spark.lua#L6-L10)）；服务器端此 FX 完全无声。
- **边界：lightbeam 不是动画实体。** 工厂里 `SetBank/SetBuild/PlayAnimation` 全在注释里（[L156-L166](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L156-L166)）；对它调 SetBuild/SetTime 无效，视觉只能调灯点（数量、间距、半径公式固定）。
- **边界：lightbeam 的客户端灯位是预测出来的。** 客户端优先用 `playercontroller` 的远程预测位置推算朝向（停止行走后还保留 2 秒惯性，[PREDICT_FROM_LAST_WALK_DELAY](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L47-L47)），与服务器净变量结果可能短暂不同；代码里还有 `inst.sg` 的 `floating` 低阈值门禁（`0.0001` 对 `0.05`），但原版 beam 未安装 StateGraph，`inst.sg` 恒 nil，该分支未证实可达，实际回退到 0.05。[预测与门禁](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L48-L85)、[无 SG 装配](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L153-L190)
- **边界：扫描束生命周期挂在扫描仪单槽引用上。** `inst.scan_fx` 只有一个槽；同一时刻只有最新目标有扫描束。目标 `health:IsDead()` 时 `OnUpdateScanCheck` 先 `StopScanFX`（`goAway`）再 `OnScanFailed`；死亡本身不会 Remove 实体，`AddChild` 的 Lua `children` 删除链只在目标实体真正 `Remove` 时生效，不能把死亡直接当成“已删除引用”。复刻时仍应判 `IsValid`，但不要把该推演写成原版必然报错。[检查](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L419-L443)、[AddChild](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L553-L563)
- **边界：backupbody_inventory 不要单独 Spawn 使用。** 它的可见动画全由本体回调驱动（受击/建造/附身开关），单独生成只得到一个静止机体壳；读档时实体被 `SpawnSaveRecord` 全量替换，跨帧缓存该引用会悬空（走 `inst.wx78_backupbody_inventory` 字段重取）。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/wx78_big_spark.lua`、`prefabs/wx78_lightbeam.lua`、`prefabs/wx78_backupbody.lua` 全文，`prefabs/wx78_scanner.lua`（scanfx 工厂、Start/StopScanFX、扫描主循环、本地扫描圈），按需读取 `prefabs/wx78.lua` 与 `prefabs/wx78_possessedbody.lua` 生成段、`wx78_moduledefs.lua` light 模块段、`prefabs/wx78_common.lua` steam/gestalt 段、`entityscript.lua` 的 Remove 与 `mainfunctions.lua` 存档循环节。对 4 个注册名以 `grep -rn "注册名" --include=*.lua` 在 `raw/current/scripts` 全域反查：`wx78_big_spark` 命中注册（:159）、依赖（wx78_common.lua:11、prefablist.lua:1511）与 3 个 SpawnPrefab 包装（6 个调用点，见导航表）——**生成点与台账一致**；`wx78_lightbeam` 命中注册（:192）、依赖（wx78_moduledefs.lua:826、prefablist.lua:1520）、生成（:801）——**生成点唯一**；`wx78_scanner_fx` 命中注册（wx78_scanner.lua:1222）、依赖（:17）、生成（:557）——**生成点唯一**；`wx78_backupbody_inventory` 命中注册（wx78_backupbody.lua:1136）、依赖（:19）与生成两处（:912 SpawnPrefab、:842 SpawnSaveRecord），其余命中为常量表/皮肤/图鉴非生成。
- 名称核验：`languages/chinese_s.po` 定向检索——`WX78_BACKUPBODY_INVENTORY`＝"备份底盘"、`WX78_BACKUPBODY`＝"备份底盘"（[352654-352657](../../../raw/current/scripts/languages/chinese_s.po#L352654-L352657)）、`WX78_SCANNER`＝"生物扫描分析仪"、`WX78MODULE_LIGHT`＝"照明电路"（锚点见"快速判断"）；`WX78_BIG_SPARK`／`WX78_LIGHTBEAM`／`WX78_SCANNER_FX` 无 `STRINGS.NAMES.*` 键，标题使用效果描述。
- 分档说明：`size_class: standard`——4 注册三类载体，双层结构/灯点公式/AddChild 语义/存档替换各成小节，表格压缩生成点与清理；未选 complex 因无多状态机需要整体保留。
- 未覆盖：扫描仪完整玩法（proximityscan 节拍、`SGwx78_scanner`、`wx78_scannerbrain`、scandata 掉落）、`wx78_scanner_item`／`wx78_scanner`／`wx78_scanner_succeeded`／placer 等同文件其余注册、备份底盘附身转移与幽灵复活流程（`wx78_possessedbody` 主体）、upgrademodule 模块体系与 wx78_classified 同步、gestalt/热蒸汽 FX 的本地表现实体实现（netvar 协议已记，实体创建在 wx78_common.lua）、动画包内容与观感。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/electrocute_fx|电击表现与连锁闪电]]（通用电击表现；对照 WX-78 专属电火花的代理壳方案）
- [[reference/prefabs/winona_battery_high_shatterfx|薇诺娜宝石发电机设备特效族]]（同为"本体无 AnimState＋联网渲染子实体"的设备层结构）
- [[reference/prefabs/sparks|电击命中火花（sparks 族）]]（同名电火花语义的另一独立工厂）
- 返回 [[reference/prefabs/fx-character-index|角色技能、施法与读书]]（WX78 技能/装备）
