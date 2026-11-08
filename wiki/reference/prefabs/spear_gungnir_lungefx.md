---
title: "突刺与装备跟随电光（spear_gungnir_lungefx、spear_wathgrithr_lightning_lunge_fx 等 3 注册）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "两把突刺长矛文件的 3 个 FX 注册：尖齿矛（熔炉事件包本体）的突刺烟（快照内 0 生成点的诚实缺口，FastForward 快进接口在）、奔雷矛突刺的一次性八向落点光（aoeweapon_lunge → SGwilson combat_lunge 链生成）与充能奔雷矛的装备跟随光层（本体持有、SetFxOwner 换绑）。"
aliases: ["spear_gungnir_lungefx", "spear_wathgrithr_lightning_lunge_fx", "spear_wathgrithr_lightning_fx", "尖齿矛", "奔雷矛", "充能奔雷矛", "突刺特效", "突刺烟", "装备跟随电光", "沃特格里斯长矛", "战场女武神长矛"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/spear_gungnir.lua", "prefabs/spear_wathgrithr.lua", "prefabs/lavaarena.lua", "components/aoeweapon_lunge.lua", "stategraphs/SGwilson.lua", "componentutil.lua", "entityscript.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 突刺与装备跟随电光（spear_gungnir_lungefx、spear_wathgrithr_lightning_lunge_fx 等 3 注册）

## 快速判断

- 职责与适用：两份长矛源文件的 **3 个 FX 注册**：`spear_gungnir_lungefx`（尖齿矛突刺烟，[注册](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L150-L151)）、`spear_wathgrithr_lightning_lunge_fx`（奔雷矛八向突刺落点光，[注册](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L691-L696)）、`spear_wathgrithr_lightning_fx`（充能奔雷矛的装备跟随电光层，[注册](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L691-L696)）。前两个是一次性世界落点表现，第三个**没有自身生命周期**——由充能矛本体持有并跟随装备。
- 载体模式：①**一次性自删落点**（gungnir lungefx 按动画长度＋2 帧定时自删；wathgrithr lunge fx 等动画结束与灯光归零两条件都完成后退场；都 `persists=false`）；②**宿主持有的常驻跟随层**（wathgrithr fx——不设自删，通过 `SetFxOwner` 在"矛本体"与"持有者"之间换绑父级与跟随符号）。
- **诚实缺口**：`spear_gungnir_lungefx` 在本快照 `raw/current/scripts` 内 **0 生成点**——尖齿矛本体的主模拟装配整段来自熔炉事件包 `event_server_data("lavaarena", "prefabs/spear_gungnir").master_postinit`（[L95](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L95-L95)），事件包文件不在快照内；事件包可能是消费方，具体生成调用未核验。检索模式与范围见[验证记录](#验证记录)。
- 修改边界：只覆盖 3 个 FX 注册的工厂、生成链、端模型与清理；两把长矛本体（武器数值、奔雷矛升级链、充能玩法）只作链路背景不展开。对照页：瞄准圈（reticuleline 系列）与电击火花（`electrichitsparks`）各有页面。
- 名称：官方名 `STRINGS.NAMES.SPEAR_GUNGNIR`＝"尖齿矛"（[po](../../../raw/current/scripts/languages/chinese_s.po#L349749-L349752)）、`SPEAR_WATHGRITHR`＝"战斗长矛"（[po](../../../raw/current/scripts/languages/chinese_s.po#L349759-L349762)）、`SPEAR_WATHGRITHR_LIGHTNING`＝"奔雷矛"（[po](../../../raw/current/scripts/languages/chinese_s.po#L349764-L349767)）、`SPEAR_WATHGRITHR_LIGHTNING_CHARGED`＝"充能奔雷矛"（[po](../../../raw/current/scripts/languages/chinese_s.po#L349769-L349772)）；3 个 FX 注册名均无 `STRINGS.NAMES.*` 键，标题与正文效果描述**非官方译名**。

## 注册与构造

### spear_gungnir_lungefx（突刺烟；快照内无生成点）

工厂 `fxfn`（[L114-L148](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L114-L148)）：Transform／AnimState／Network（物理整段注释掉），标签 `FX`／`NOCLICK`；bank／build 借用 `lavaarena_staff_smoke_fx`、播 `idle`，`SetAddColour(1, 1, 0, 0)` 叠加增色＋泛光 shader。主模拟 `persists=false`＋动画长度＋2 帧定时自删（[L141-L142](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L141-L142)）。唯一实例方法是 `FastForward(pct)`（[L100-L108](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L100-L108)）：取消自删任务、`SetTime(长度×pct)` 快进动画、按剩余时长重排自删——快进接口留在快照内；事件包可能是消费方，具体调用未核验。

尖齿矛本体（[fn](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L52-L98)）在快照内可见的部分：客户端侧 `aoetargeting` 直线准星（reticuleline 系，[L77-L87](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L77-L87)）＋pristine 标签 `aoeweapon_lunge`／`rechargeable`（[L71-L75](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L71-L75)）；主模拟一切装配（含 aoeweapon_lunge 组件与突刺回调）都在 `master_postinit` 即事件包内。本体进场参照：熔炉活动角色开局配装（[tuning.lua#L3484](../../../raw/current/scripts/tuning.lua#L3484-L3484)）与 lavaarena 本体依赖表（[lavaarena.lua#L46](../../../raw/current/scripts/prefabs/lavaarena.lua#L46-L46)）——均为依赖声明，不是生成点。

### spear_wathgrithr_lightning_lunge_fx（八向突刺光）

工厂 `LungueTrailFxFn`（[L600-L645](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L600-L645)）：Transform／AnimState／Network／**Light**（半径 3.5、衰减 1.5、初始强度 0.9、色 237,237,209），标签 `FX`／`NOCLICK`，`SetEightFaced()` 八向（[L617](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L617-L617)）；bank／build `elec_lunge_fx`、播 `fx`、泛光＋LightOverride 1。主模拟随机缩放 1.4–1.7、`persists=false`，并以 1/20 秒周期跑 `FX_OnUpdate`（[L578-L589](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L578-L589)）：光强按 `i = i - dt*2` 线性衰减到 0。退场是**双路协同**（[FX_OnAnimOver](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L591-L598)）：`animover` 先到时若更新任务还在则只置 `killfx=true`，由更新循环在光强归零后 `Remove`；更新先结束则由 `animover` 直接删。

生成链（[Lightning_OnLunged](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L174-L182)）：`SpawnPrefab` 落在突刺目标点 `targetpos`、`SetRotation(施放者朝向)`、`rechargeable:Discharge` 进冷却——挂接在 `aoeweapon_lunge` 组件的 `SetOnLungedFn`（[L486](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L481-L490)），普通／充能两把奔雷矛共用。冷却 3 秒／充能 1.5 秒（[tuning](../../../raw/current/scripts/tuning.lua#L8058-L8071)）。

### spear_wathgrithr_lightning_fx（充能矛的装备跟随电光层）

工厂 `FxFn`（[L649-L686](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L649-L686)）：Transform／AnimState／Follower／Network，标签 `FX`；bank／build `spear_wathgrithr_lightning`、循环播 `swap_loop`；`bolt_b/c/f` 三符号泛光＋LightOverride 0.5、`glow` 0.25、整体 LightOverride 0.1，**`HideSymbol("swap_spear_wathgrithr_lightning")` 隐藏矛身只留电弧**；带 `highlightchild` 组件（客户端也构建）；主模拟加 `colouradder`、`persists=false`。**无任何自删钩子**——寿命完全由充能矛本体管理。

持有与换绑（[LightningCharged_SetFxOwner](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L282-L316)）：

- 装备时（[OnEquip L111-L117](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L87-L119)）：`SetFxOwner(owner)` → FX 引擎父挂到持有者、`Follower:FollowSymbol(owner, "swap_object")`、`highlightchild:SetOwner(owner)`、colouradder 附着、持有者播充能音循环；
- 未装备／掉落时：`SetFxOwner(nil)` → FX 父挂回矛本体、跟随矛自己的 `swap_spear` 符号（漂浮展示，[L310-L314](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L310-L315)）；
- 皮肤按 `FX_OFFSETS`（[L274-L280](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L274-L280)）微调 y 偏移并覆写皮肤符号。
- 生成即对帧：充能矛 postinit 生成 FX 时取同一随机帧、`SetFrame` 对齐（[L521-L525](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L511-L529)），随后 `SetFxOwner(nil)` 挂回本体。

## 关联机制导航

### 生成点与宿主调用

| 注册 | 生成语句（file:line） | 消费文件与条件 |
|---|---|---|
| `spear_gungnir_lungefx` | **无**（诚实缺口） | 全 `raw/current/scripts` 反查仅命中：本文件 prefabs 依赖（[L16](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L12-L19)）与注册（L151）、prefablist.lua:1188（文件清单）。检索模式：`grep -rn "spear_gungnir_lungefx" --include=*.lua` 加 `_lungefx` 后缀动态拼名检索（仅命中注册自身）。事件包（`event_server_data` 装载的 `lavaarena_event_server/prefabs/spear_gungnir.lua`，[装载入口](../../../raw/current/scripts/entityscript.lua#L77-L87)）可能是消费方，具体生成调用未核验；保留注册与 `FastForward` 接口的正向证据，不判死注册 |
| `spear_wathgrithr_lightning_lunge_fx` | [spear_wathgrithr.lua#L175](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L174-L182) | 唯一生成点：奔雷矛 `aoeweapon_lunge` 组件的 `onlungedfn`。触发链：CASTAOE 动作 → SGwilson `combat_lunge_start`（[ActionHandler L1324](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1316-L1331)、[状态 L17397](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17397-L17428)）→ 施法回调推 `combat_lunge` 事件（[Lightning_SpellFn](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L157-L159)）→ [combat_lunge 状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17430-L17449)调 `DoLunge`（[aoeweapon_lunge.lua#L44-L50](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L44-L50)、[onlungedfn 回调 L228-L229](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L228-L229)）→ 本表生成。突刺前的 `OnPreLunge` 只做 2 帧后重置电击判定窗口（[L165-L172](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L165-L172)） |
| `spear_wathgrithr_lightning_fx` | [spear_wathgrithr.lua#L524](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L511-L529) | 唯一生成点：充能奔雷矛 `LightningSpearPostInitFn_Charged` 装配时（普通奔雷矛 `prefabs_lightning` 依赖表不含它，[L28-L34](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L28-L34)；只有 charged 的依赖表含，[L36-L42](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L36-L42)）。换绑消费：OnEquip／OnUnequip（[L111-L117](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L87-L119)、[L134-L140](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L121-L141)）与浮起／停止漂浮的 `OnStopFloating` 恢复循环动画（[L322-L325](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L322-L325)） |

充能矛本体由普通矛升级而来（[Lightning_OnUpgraded](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L248-L270)：换 slot 生成充能矛并删除普通矛）；突刺命中按上限修复耐久（[Lightning_OnLungedHit](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L184-L195)，每次突刺最多修 2 次×4 点）。

### 层、父子与端模型

| 对象 | 本组注册如何使用 | 边界 | 证据 |
|---|---|---|---|
| 落点对齐 | wathgrithr lunge fx 落 `targetpos`（突刺终点而非施放者）并复制施放者旋转；八向贴图自动取最近方向 | gungnir lungefx 的落点/朝向逻辑在快照外，无法核验是否同构 | [OnLunged](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L174-L182) |
| 引擎父子 | wathgrithr fx 在"矛本体 ↔ 持有者"之间换绑父级，无 Lua children 登记；lunge fx 不父挂，世界落点 | SetFxOwner 换绑同时处理 colouradder 附着/脱离与 highlightchild 归属，三者必须同步 | [SetFxOwner](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L282-L316) |
| 端模型 | 三注册都是联网壳；wathgrithr fx 在客户端也建（highlightchild 在 Pristine 前加入，[L673](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L649-L686)），随矛的网络实体复制；lunge fx 的光强衰减只在主模拟跑 | 联网壳上的动画主模拟驱动；gungnir lungefx 的 FastForward 是主模拟端方法 | [工厂](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L600-L645)、[Pristine 分界](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L673-L675) |

### 清理路径

| 注册 | 退场 | 边界与证据 |
|---|---|---|
| `spear_gungnir_lungefx` | 动画长度＋2 帧定时自删（`FastForward` 可压缩剩余时长）；`persists=false` | [自删](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L141-L142)、[快进](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L100-L108) |
| `spear_wathgrithr_lightning_lunge_fx` | 光强归零与动画结束两条件都完成后才 `Remove`；都 `persists=false` | [双路](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L578-L598) |
| `spear_wathgrithr_lightning_fx` | **无自身清理**；充能矛 `finiteuses` 用尽 `inst.Remove`（[CommonFn L394](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L391-L394)）后实际是否随引擎父子回收未核验 | 唯一显式路径＝随持有者／本体移除；换绑只是改父级不改寿命 |

## 典型接入

### 复刻"装备跟随光层"模式

- 前提与选择：需要"跟随装备、脱手即回到物品展示"的常驻光层时，复刻 SetFxOwner 模式——FX 无自删、寿命绑本体，换绑一次处理父级＋Follower 符号＋highlight＋colour 四件事。
- 生效与撤销：主模拟生成后先挂本体（`SetFxOwner(nil)`），装备回调里换到持有者；皮肤偏移在换绑时按 skin_build 查表。
- 原版依据：[换绑](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L282-L316)、[装配对帧](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L511-L529)。

```lua
local function SetGlowOwner(fx, holder)
    if holder ~= nil then
        fx.entity:SetParent(holder.entity)
        fx.Follower:FollowSymbol(holder.GUID, "swap_object", 0, 0, 0, true)
    else
        fx.entity:SetParent(nil)
    end
end
```

- 调整与验证：原版实现还同步 highlightchild／colouradder／皮肤符号，复刻时这三件不可省略否则高亮与染色会残留前任持有者。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：`spear_gungnir_lungefx` 是死注册。** 快照内 0 生成点属实，但生成方可能在 `event_server_data` 装载的熔炉事件包里（尖齿矛本体的主模拟装配整段都在那里）；本页只能核验"快照内无生成调用＋快进接口存在"，不能核验事件包侧接线，也不判死注册。[缺口](#生成点与宿主调用)
- **边界：gungnir lungefx 与 wathgrithr lunge fx 不共用工厂。** 两者 bank／动画／自删时刻／实例方法（FastForward 仅 gungnir 有）都不同；名字相似不代表同族。[对照](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L114-L148)、[对照](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L600-L645)
- **边界：wathgrithr fx 不随"丢在地上"消失。** 换绑只改父级：矛落地漂浮时 FX 跟随矛的 `swap_spear` 符号继续显示电弧；唯一显式路径是矛本体被移除（耐久耗尽、升级替换），是否随引擎父子回收未核验。[漂浮](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L310-L315)
- **边界：突刺光是"落点"不是"轨迹"。** `spear_wathgrithr_lightning_lunge_fx` 生成在突刺终点、原地播一次，不沿路径拉出光带；轨迹观感来自八向贴图＋随机缩放＋点光衰减。[落点](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L175-L177)
- **边界：突刺命中还另有电击火花。** 奔雷矛普通攻击命中带电目标时走 `SpawnElectricHitSparks`（[L151](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L149-L153)，[实现](../../../raw/current/scripts/componentutil.lua#L1022-L1030)生成 `electrichitsparks` 系注册），与本页两个注册无关；突刺本身的电击判定窗口由 `OnPreLunge` 的 2 帧延迟Hack撑开（[L165-L172](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L165-L172)）。
- **边界：`.po` 名称按本体注册键。** 三 FX 注册无官方名；"尖齿矛"等官方名挂在长矛本体注册名上（武器物品名），页面借本体名描述宿主——对照索引时按本体名检索武器、按效果描述检索 FX。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/spear_gungnir.lua`、`prefabs/spear_wathgrithr.lua` 全文（本体与 3 FX 工厂、换绑、突刺回调链）；按需读取 `components/aoeweapon_lunge.lua` 的 DoLunge 与两个回调注册、`stategraphs/SGwilson.lua` 的 CASTAOE ActionHandler 与 combat_lunge_start／combat_lunge 状态、`componentutil.lua` 的 SpawnElectricHitSparks、`entityscript.lua` 的 event_server_data、`tuning.lua` 的 `SPEAR_WATHGRITHR_LIGHTNING_*` 与熔炉配装表。全 `raw/current/scripts` 反查（`grep -rn "spear_gungnir" --include=*.lua`）：命中 spear_gungnir.lua 全部自身行（依赖 L16、注册 L151、event_server_data L95）、prefablist.lua:1188（文件清单）、tuning.lua:3484（熔炉配装表）、lavaarena.lua:46（依赖表）——无任何 SpawnPrefab／动态拼名命中；`grep -rn "_lungefx"` 全库仅命中 gungnir 两行。`grep -rn "spear_wathgrithr_lightning_lunge_fx\|spear_wathgrithr_lightning_fx"`：lunge_fx 命中依赖表 L32/L40、生成 L175、注册 L695；fx 命中依赖表 L41、生成 L524、注册 L696——各唯一生成点。
- 名称核验：`languages/chinese_s.po` 定向检索——`SPEAR_GUNGNIR`＝"尖齿矛"（msgid "Pith Pike"，[L349749-L349752](../../../raw/current/scripts/languages/chinese_s.po#L349749-L349752)）、`SPEAR_WATHGRITHR`＝"战斗长矛"（[L349759-L349762](../../../raw/current/scripts/languages/chinese_s.po#L349759-L349762)）、`SPEAR_WATHGRITHR_LIGHTNING`＝"奔雷矛"（msgid "Elding Spear"，[L349764-L349767](../../../raw/current/scripts/languages/chinese_s.po#L349764-L349767)）、`SPEAR_WATHGRITHR_LIGHTNING_CHARGED`＝"充能奔雷矛"（[L349769-L349772](../../../raw/current/scripts/languages/chinese_s.po#L349769-L349772)）；`SPEAR_WATHGRITHR_LIGHTNING_LUNGE_FX`／`_FX`／`SPEAR_GUNGNIR_LUNGEFX` 均无 `STRINGS.NAMES.*` 键，标题与正文 FX 描述为已核验效果描述。
- 分档说明：`size_class: standard`——3 注册、两条清晰的宿主链（gungnir 缺口判定＋wathgrithr 突刺/换绑）；无多状态多生命周期，一页可整体承载。
- 未覆盖：熔炉事件包内容（`lavaarena_event_server/prefabs/spear_gungnir.lua` 不在快照，lungefx 生成与本体战斗数值无法核验）、奔雷矛技能树解锁与 aoetargeting 准星细节（[[reference/prefabs/reticuleaoe|瞄准圈与 AoE 目标指示]]）、`electrichitsparks` 系注册（电击表现页）、皮肤资产内容、动画观感。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/reticuleaoe|瞄准圈与 AoE 目标指示]]（两把矛的 CASTAOE 直线准星 reticuleline 系）
- [[reference/prefabs/electrocute_fx|电击表现与连锁闪电]]（奔雷矛命中触发的 electrichitsparks 与 electrocute 表现）
- [[reference/prefabs/hitsparks_fx|命中火花特效]]（同属战斗反馈的命中类 FX）
- 返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
