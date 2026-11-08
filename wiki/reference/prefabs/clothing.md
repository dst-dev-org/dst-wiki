---
title: "BODY 槽衣物与背心族（raincoat 等 12 注册）"
created: "2026-11-07"
updated: "2026-11-08"
type: prefab
summary: "BODY 槽衣物与背心 12 注册（9 文件）的工厂、组件与消费端：雷电绝缘/防水、夏冬保温、dapperness、装备期 fueled 与常驻腐坏两条耐久链、气球背心挨打即爆、独奏乐器招募与影子等级；含配方、奖品亭兑换、官方名与皮肤。"
aliases: ["raincoat", "hawaiianshirt", "sweatervest", "reflectivevest", "trunkvest_summer", "trunkvest_winter", "beargervest", "balloonvest", "carnival_vest_a", "carnival_vest_b", "carnival_vest_c", "onemanband", "雨衣", "花衬衫", "犬牙背心", "清凉夏装", "透气背心", "松软背心", "熊皮背心", "充气背心", "叽叽喳喳围巾", "叽叽喳喳斗篷", "叽叽喳喳小披肩", "独奏乐器", "一人乐队", "背心"]
tags: [prefab, item, equipment]
size_class: standard
dst_build: "steam-buildid:26474873"
sources: ["prefabs/raincoat.lua", "prefabs/hawaiianshirt.lua", "prefabs/sweatervest.lua", "prefabs/reflectivevest.lua", "prefabs/trunkvest.lua", "prefabs/beargervest.lua", "prefabs/balloonvest.lua", "prefabs/carnival_vest.lua", "prefabs/onemanband.lua", "prefabs/balloons_common.lua", "prefabs/balloonspeed.lua", "prefabs/waxwell.lua", "prefabs/hutch.lua", "prefabs/koalefant.lua", "prefabs/carnival_prizebooth.lua", "prefabskins.lua", "recipes.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po", "componentutil.lua", "components/equippable.lua", "components/insulator.lua", "components/waterproofer.lua", "components/fueled.lua", "components/fuel.lua", "components/sanity.lua", "components/temperature.lua", "components/moisture.lua", "components/inventory.lua", "components/hunger.lua", "components/poppable.lua", "components/flotationdevice.lua", "components/leaderrollcall.lua", "components/shadowlevel.lua", "components/drownable.lua", "components/dynamicmusic.lua", "components/playerlightningtarget.lua", "components/burnable.lua", "standardcomponents.lua", "stategraphs/SGwilson.lua", "stategraphs/SGcatcoon.lua", "stategraphs/SGshadowwaxwell.lua", "brains/catcoonbrain.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# BODY 槽衣物与背心族（raincoat 等 12 注册）

## 快速判断

- 职责与适用：9 个独立文件注册的 **12 个 BODY 槽装备**：常规服饰 7（raincoat、hawaiianshirt、sweatervest、reflectivevest、trunkvest_summer/winter、beargervest）+奖品亭兑换 3（carnival_vest_a/b/c）+balloonvest+onemanband。可借鉴：雷电绝缘/防水/夏冬保温组合、两条耐久链、气球挨打即爆防御、`band` 标签演奏招募。
- 优先使用：查组件／耐久／配方用[注册表](#注册与构造)；查消费端用[保温防水与雷电绝缘](#保温防水与雷电绝缘)、[燃料与腐坏链](#燃料与腐坏链)；气球与乐队机制见对应小节。
- 重要副作用：fueled 衣物装备期耗耐久、耗尽自毁（balloonvest 耗尽即爆）；hawaiianshirt 常驻腐坏（背包内也计时）；balloonvest 被打/溺水即爆并攻击 2 格内实体（**骑乘时只停用不炸**）；onemanband 理智负收益随跟随数放大，每秒**招募猪／鱼人、照料作物**（作物不受随从上限）。
- 修改边界：只覆盖本体组件、配方、官方名、皮肤与消费端标签；动画资源、balloons_common 全量气球行为、Hutch／Maxwell 侧完整机制不在本页。数值均为源码推导值。

## 注册与构造

### 工厂与共同骨架

9 文件各自独立工厂、无共享 helper。12 注册全部 `equipslot = EQUIPSLOTS.BODY`（raincoat [L61](../../../raw/current/scripts/prefabs/raincoat.lua#L62)、hawaiianshirt [L52](../../../raw/current/scripts/prefabs/hawaiianshirt.lua#L53)、sweatervest [L46](../../../raw/current/scripts/prefabs/sweatervest.lua#L46)、reflectivevest [L63](../../../raw/current/scripts/prefabs/reflectivevest.lua#L64)、trunkvest [L77](../../../raw/current/scripts/prefabs/trunkvest.lua#L78)、beargervest [L72](../../../raw/current/scripts/prefabs/beargervest.lua#L73)、balloonvest [L82](../../../raw/current/scripts/prefabs/balloonvest.lua#L82)、carnival_vest [L49](../../../raw/current/scripts/prefabs/carnival_vest.lua#L49)、onemanband [L89](../../../raw/current/scripts/prefabs/onemanband.lua#L89)）。装备表现统一 `OverrideSymbol("swap_body", …)`（onemanband 用 `swap_body_tall`，[L13](../../../raw/current/scripts/prefabs/onemanband.lua#L13)；carnival 另盖 `backpack`，[carnival_vest.lua#L4-L5](../../../raw/current/scripts/prefabs/carnival_vest.lua#L4-L5)）；皮肤走 `GetSkinBuild`。**带 fueled 的装备回调 `StartConsuming`、卸下与入模型 `StopConsuming`**（raincoat [L14](../../../raw/current/scripts/prefabs/raincoat.lua#L14)、[L19](../../../raw/current/scripts/prefabs/raincoat.lua#L19)、[L28](../../../raw/current/scripts/prefabs/raincoat.lua#L28)），燃料只在装备期消耗。

单位换算（[tuning.lua#L2207](../../../raw/current/scripts/tuning.lua#L2207)、[L2214-L2216](../../../raw/current/scripts/tuning.lua#L2214-L2216)、[L2260-L2263](../../../raw/current/scripts/tuning.lua#L2260-L2263)、[L1674](../../../raw/current/scripts/tuning.lua#L1674)、[L1681](../../../raw/current/scripts/tuning.lua#L1681)）：`total_day_time`=480 秒；insulator 值是**秒**（SMALL 60/MED 120/LARGE 240）；dapperness 是**理智/秒**（SMALL ≈16/MED ≈26.7/MED_LARGE ≈35.5 每天合计）；SANITYAURA_SMALL ≈200/天；SMALL_FUEL=15 秒燃料值；SMALL_BURNTIME=7.5 秒。

### 本页注册表

**表A 常规服饰（7 注册）**

| 注册名（官方名） | 组件与关键数值 | 获取与证据 |
|---|---|---|
| `raincoat`（雨衣） | **雷电绝缘**（`equippable.insulated=true`）＋防水 SMALL＋冬保温 SMALL＋fueled USAGE 10 天耗尽自毁＋pristine `waterproofer`＋tradable | 配方 2 触手皮+2 绳+2 骨片 SCIENCE_ONE（[recipes.lua#L546](../../../raw/current/scripts/recipes.lua#L546)）；[raincoat.lua#L60-L77](../../../raw/current/scripts/prefabs/raincoat.lua#L61-L78) |
| `hawaiianshirt`（花衬衫） | dapperness MED＋**夏**保温 LARGE＋perishable 15 天**常驻腐坏**耗尽自毁＋pristine `show_spoilage`（物品格显示腐坏条） | 配方 3 莎草纸+3 蛛丝+5 仙人掌花 SCIENCE_TWO（[recipes.lua#L551](../../../raw/current/scripts/recipes.lua#L551)）；[hawaiianshirt.lua#L51-L64](../../../raw/current/scripts/prefabs/hawaiianshirt.lua#L52-L65) |
| `sweatervest`（犬牙背心） | dapperness MED＋冬保温 SMALL＋fueled USAGE 10 天＋tradable | 配方 8 猎犬牙+6 蛛丝 SCIENCE_TWO（[recipes.lua#L547](../../../raw/current/scripts/recipes.lua#L547)）；[sweatervest.lua#L45-L59](../../../raw/current/scripts/prefabs/sweatervest.lua#L45-L59) |
| `reflectivevest`（清凉夏装） | dapperness SMALL＋**夏**保温 MED＋防水 SMALL＋fueled USAGE 8 天＋pristine `waterproofer` | 配方 1 绳+3 红雀羽毛+2 猪皮 SCIENCE_ONE（[recipes.lua#L550](../../../raw/current/scripts/recipes.lua#L550)）；[reflectivevest.lua#L62-L79](../../../raw/current/scripts/prefabs/reflectivevest.lua#L63-L80) |
| `trunkvest_summer`（透气背心） | dapperness SMALL＋**夏**保温 SMALL＋防水 SMALL＋fueled USAGE 15 天＋pristine `waterproofer`＋tradable | 配方 1 trunk_summer+8 蛛丝 SCIENCE_TWO（[recipes.lua#L548](../../../raw/current/scripts/recipes.lua#L548)）；trunk 出自考拉象掉落（[koalefant.lua#L22-L23](../../../raw/current/scripts/prefabs/koalefant.lua#L22-L23)、[L36](../../../raw/current/scripts/prefabs/koalefant.lua#L36)）；[trunkvest.lua#L95-L110](../../../raw/current/scripts/prefabs/trunkvest.lua#L96-L111) |
| `trunkvest_winter`（松软背心） | dapperness SMALL＋**冬**保温 LARGE＋无防水＋fueled USAGE 15 天＋tradable | 配方 1 trunk_winter+8 蛛丝+2 牛毛 SCIENCE_TWO（[recipes.lua#L549](../../../raw/current/scripts/recipes.lua#L549)）；[trunkvest.lua#L112-L124](../../../raw/current/scripts/prefabs/trunkvest.lua#L113-L125) |
| `beargervest`（熊皮背心） | dapperness MED_LARGE＋冬保温 LARGE＋fueled USAGE 7 天＋**装备期持有者饥饿消耗 ×0.75**＋tradable | 配方 1 熊皮+1 犬牙背心+2 绳 SCIENCE_TWO（[recipes.lua#L553](../../../raw/current/scripts/recipes.lua#L553)）；饥饿修饰符 [beargervest.lua#L15-L17](../../../raw/current/scripts/prefabs/beargervest.lua#L15-L17)（消费端 [hunger.lua#L154](../../../raw/current/scripts/components/hunger.lua#L154)）；组件 [L71-L81](../../../raw/current/scripts/prefabs/beargervest.lua#L72-L82) |

**表B 特殊机制（5 注册）**

| 注册名（官方名） | 组件与关键数值 | 获取与证据 |
|---|---|---|
| `balloonvest`（充气背心） | `MakeBalloonMasterInit`（[balloons_common.lua#L134-L156](../../../raw/current/scripts/prefabs/balloons_common.lua#L134-L156)）：poppable＋combat 伤害 5＋health 1（死亡即 Pop）；装备即监听持有者 `attacked`（[balloonvest.lua#L20-L27](../../../raw/current/scripts/prefabs/balloonvest.lua#L20-L27)）；fueled **MAGIC** 1 天耗尽即爆；flotationdevice 防溺水一次；tags `cattoy`/`balloon`/`noepicmusic`；无保温无 dapperness | Wes 专属配方 0×balloons_empty+5 理智（`balloonomancer`，[recipes.lua#L376](../../../raw/current/scripts/recipes.lua#L376)）；[balloonvest.lua#L59-L85](../../../raw/current/scripts/prefabs/balloonvest.lua#L59-L85) |
| `carnival_vest_a`/`b`/`c`（叽叽喳喳围巾/斗篷/小披肩） | `MakeVest` 三连注册（[carnival_vest.lua#L75-L89](../../../raw/current/scripts/prefabs/carnival_vest.lua#L75-L89)）：dapperness SMALL＋**夏**保温 a=MED、b/c=LARGE＋fueled USAGE 5 天＋`fuel` 组件（SMALL_FUEL，可喂火）＋MakeSmallBurnable（**可点燃**）＋Propagator | 奖品亭兑换 24/48/48 奖票（[recipes.lua#L1196-L1198](../../../raw/current/scripts/recipes.lua#L1196-L1198)；科技由奖品亭 prototyper 授予 [carnival_prizebooth.lua#L135](../../../raw/current/scripts/prefabs/carnival_prizebooth.lua#L135)） |
| `onemanband`（独奏乐器） | **dapperfn 动态理智**：−DAPPERNESS_SMALL−净跟随数×SANITYAURA_SMALL（[onemanband.lua#L6-L10](../../../raw/current/scripts/prefabs/onemanband.lua#L6-L10)）＋fueled **ONEMANBAND** 6×seg_time=**180 秒**耗尽自毁＋leaderrollcall（半径 12、上限 10、可照料作物）＋leader＋shadowlevel 1＋pristine `band`/`shadowlevel`＋`SetSinks`＋hauntable | 配方 2 金+4 噩梦燃料+2 猪皮 MAGIC_TWO（[recipes.lua#L520](../../../raw/current/scripts/recipes.lua#L520)）；[onemanband.lua#L79-L108](../../../raw/current/scripts/prefabs/onemanband.lua#L79-L108) |

皮肤（[prefabskins.lua](../../../raw/current/scripts/prefabskins.lua) 逐项检索：[L213-L217](../../../raw/current/scripts/prefabskins.lua#L213-L217)、[L2026-L2032](../../../raw/current/scripts/prefabskins.lua#L2026-L2032)）：raincoat ×2（poncho/western，L1528-1531）、hawaiianshirt ×5（L984-990）、reflectivevest_cawnival（L1558-1560）、beargervest ×3（rose/shadow/yule）、trunkvest 双款 quilted；sweatervest/balloonvest/carnival_vest/onemanband 无皮肤。

## 关联机制导航

### 保温防水与雷电绝缘

| 装备链 | 消费端 | 本族成员 |
|---|---|---|
| `insulator`（值为秒；冬为默认 type，夏用 SetSummer，[insulator.lua#L7-L9](../../../raw/current/scripts/components/insulator.lua#L7-L9)、[L23-L29](../../../raw/current/scripts/components/insulator.lua#L23-L29)） | `Temperature:GetInsulation` 按 type 累加全部装备（[temperature.lua#L255-L272](../../../raw/current/scripts/components/temperature.lua#L255-L272)）；冬保温延缓冻、夏保温延缓过热 | 冬：sweatervest/trunkvest_winter/beargervest/raincoat；夏：hawaiianshirt/reflectivevest/trunkvest_summer/carnival ×3 |
| `waterproofer`（effectiveness 为比例，构造即打标签，[waterproofer.lua#L4-L11](../../../raw/current/scripts/components/waterproofer.lua#L4-L11)） | `Inventory:GetWaterproofness` 累加全部装备（[inventory.lua#L2719-L2731](../../../raw/current/scripts/components/inventory.lua#L2719-L2731)）→潮湿增速按 `waterproofmult≥1` 归零（[moisture.lua#L214-L231](../../../raw/current/scripts/components/moisture.lua#L214-L231)） | raincoat/reflectivevest/trunkvest_summer（均 SMALL=20%） |
| `equippable.insulated` | `IsInsulated`（**雷电绝缘，注释明确"非温度"**，[equippable.lua#L64-L66](../../../raw/current/scripts/components/equippable.lua#L64-L66)）→`Inventory:IsInsulated`（[inventory.lua#L759-L770](../../../raw/current/scripts/components/inventory.lua#L759-L770)）→闪电改推 `lightningdamageavoided`（[playerlightningtarget.lua#L4-L10](../../../raw/current/scripts/components/playerlightningtarget.lua#L4-L10)） | 仅 raincoat（HEAD 槽同链见 [[reference/prefabs/hats-environment]] 的 rainhat/eyebrella） |

### 燃料与腐坏链

- **fueled**：1 单位=1 秒（rate=1，[fueled.lua#L89](../../../raw/current/scripts/components/fueled.lua#L89)、[L325-L328](../../../raw/current/scripts/components/fueled.lua#L325-L328)）。USAGE 型打 `needssewing`→缝纫包 SEW（[fueled.lua#L10-L16](../../../raw/current/scripts/components/fueled.lua#L10-L16)、[componentactions.lua#L1708-L1713](../../../raw/current/scripts/componentactions.lua#L1708-L1713)）——本族除 hawaiianshirt/balloonvest/onemanband 外均可缝补。`accepting` 默认 false（[fueled.lua#L94](../../../raw/current/scripts/components/fueled.lua#L94)），fuel 物品默认 BURNABLE（[fuel.lua#L13-L21](../../../raw/current/scripts/components/fuel.lua#L13-L21)），MAGIC 注释即"不配燃料"（[constants.lua#L1995-L2007](../../../raw/current/scripts/constants.lua#L1995-L2007)）——**balloonvest/onemanband 不可补耐久**。耗尽本族均 Remove，balloonvest 例外 `FueledDepletedPop`（[balloons_common.lua#L189-L191](../../../raw/current/scripts/prefabs/balloons_common.lua#L189-L191)）。
- **perishable（仅 hawaiianshirt）**：创建即计时，背包内照常腐坏，耗尽 Remove（[hawaiianshirt.lua#L57-L60](../../../raw/current/scripts/prefabs/hawaiianshirt.lua#L58-L61)）。**两件考拉象背心不是 perishable**——`trunkvest_summer/winter` 是 `FUELTYPE.USAGE` 的 fueled 注册（装备期才耗），别把"衬衫／背心"整体当成腐坏链（[trunkvest 工厂](../../../raw/current/scripts/prefabs/trunkvest.lua#L77-L89)）。
- **dapperness 消费端**：`Sanity:Recalc` 累加装备 dapperness（[sanity.lua#L513-L522](../../../raw/current/scripts/components/sanity.lua#L513-L522)）；`GetDapperness` 中 **dapperfn 覆盖静态值**、浸湿追加 −0.1/秒（[equippable.lua#L196-L211](../../../raw/current/scripts/components/equippable.lua#L196-L211)、[tuning.lua#L2926](../../../raw/current/scripts/tuning.lua#L2926)）。

### 气球背心链

- 挨打即爆：持有者 `attacked` →`ShouldProcOnAttackedOrBlocked` 过滤（骑乘转移/自伤不触发，[componentutil.lua#L114-L121](../../../raw/current/scripts/componentutil.lua#L114-L121)）→`poppable:Pop`（[poppable.lua#L9-L17](../../../raw/current/scripts/components/poppable.lua#L9-L17)）→`DoPop` 先 `DeactiveBalloon`（撤碰撞体、加 `notarget`/`NOCLICK`、`persists=false`、不可拾取），再分支：非骑乘的装备态才生成 [[reference/prefabs/balloon_pop_body|balloon_pop_body]] FX 并 `DoAreaAttack` 2 格 5 点攻击（显式排除 INLIMBO/notarget/noattack/flight/invisible/playerghost）后 Remove；**持有者骑乘时这一支什么都不做**（无 FX、无 AoE、不 Remove，背心留在身上但已不可拾取）；无主（放地上）走隐藏＋延迟 AoE＋Remove（[balloons_common.lua#L19-L85](../../../raw/current/scripts/prefabs/balloons_common.lua#L19-L85)、[tuning.lua#L5922-L5923](../../../raw/current/scripts/tuning.lua#L5922-L5923)）。
- 防溺水：drownable 找到**已装备的** flotationdevice 时免一次溺水伤害并回调 Pop（[drownable.lua#L322-L359](../../../raw/current/scripts/components/drownable.lua#L322-L359)）。
- `cattoy`/`balloon`：浣猫把背心当玩具（[catcoonbrain.lua#L24](../../../raw/current/scripts/brains/catcoonbrain.lua#L24)），攻击气球类目标命中率加成（[SGcatcoon.lua#L505](../../../raw/current/scripts/stategraphs/SGcatcoon.lua#L505)）；`noepicmusic`：攻击者带此标签则被打不触发危险音乐（[dynamicmusic.lua#L717-L727](../../../raw/current/scripts/components/dynamicmusic.lua#L717-L727)）。
- balloonspeed（充气速度气球）同 require balloons_common 但自成注册、HAND 槽玩具（[balloonspeed.lua#L2](../../../raw/current/scripts/prefabs/balloonspeed.lua#L2)、[L224](../../../raw/current/scripts/prefabs/balloonspeed.lua#L224)），归杂项批次，不在本页。

### 独奏乐器链

- `band`（pristine）：玩家 idle 与 mounted_idle 态检测 BODY 槽 `band` 即切入 `enter_onemanband` 播乐态（[SGwilson.lua#L4229-L4232](../../../raw/current/scripts/stategraphs/SGwilson.lua#L4229-L4232)、[L4757-L4760](../../../raw/current/scripts/stategraphs/SGwilson.lua#L4757-L4760)、[L9188 起](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9188)）；Hutch 收 `band` 物品变音乐灯形态（[hutch.lua#L142-L143](../../../raw/current/scripts/prefabs/hutch.lua#L142-L143)）。
- 招募：装备 `leaderrollcall:Enable`，每秒 `DoRollCall` 在**同一次** 12 格扫描（排除 `werepig`/`player`）里分两支（[leaderrollcall.lua#L116-L149](../../../raw/current/scripts/components/leaderrollcall.lua#L116-L149)、[tuning.lua#L1730-L1732](../../../raw/current/scripts/tuning.lua#L1730-L1732)）：①带 `follower` 的猪／鱼人——要求无首领、未被收编、`CanBeRollCalled(leader)` 且 `leader.components.leader:GetNumFollowers() < 上限(10)` 才 `AddFollower`；②带 `farmplanttendable` 的作物——`TendTo(leader)`，**不受随从上限约束**（`can_tend_crop` 为真时循环不会 `break`，随从满了照样扫到实体列表结束继续照料）。扫描完再遍历现有跟随者，对带 `pig`/`merm`/`farm_plant` 标签且 `CanBeRollCalled` 的 `AddLoyaltyTime(5)`。
- `shadowlevel`（level 1）：Maxwell 装备影子等级按 `GetCurrentLevel` 读取（[waxwell.lua#L193](../../../raw/current/scripts/prefabs/waxwell.lua#L193)、[SGshadowwaxwell.lua#L153-L155](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L153-L155)）。
- 作祟：OnHaunt 开启 rollcall 并每 0.3 秒播 foley，入袋停止（[onemanband.lua#L29-L48](../../../raw/current/scripts/prefabs/onemanband.lua#L29-L48)）。

## 典型接入

### 调整原版衣物数值与新建 BODY 背心

- 前提与选择：只调数值用 modmain `AddPrefabPostInit`（mastersim 门禁）改组件字段；新建背心复刻 raincoat 最小集 `inventoryitem`＋`inspectable`＋`equippable BODY`（[raincoat.lua#L31-L79](../../../raw/current/scripts/prefabs/raincoat.lua#L31-L80)）＋按需组件，pristine 标签在 `SetPristine` 前加。
- 生效与撤销：装备期标签随网络同步生效，卸下自动失效；fueled 在装备回调内 Start/StopConsuming（入模型须另传只停耗的回调，[raincoat.lua#L27-L29](../../../raw/current/scripts/prefabs/raincoat.lua#L27-L29)）；postinit 改动随实体存档，卸 mod 后随实体重建消失。

```lua
-- modmain.lua：把犬牙背心冬保温提到 LARGE
AddPrefabPostInit("sweatervest", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.insulator:SetInsulation(GLOBAL.TUNING.INSULATION_LARGE)
end)
```

- 调整与验证：`TUNING` 在 mod 环境白名单内可直用，`TheWorld`/`CreateEntity` 等必须 `GLOBAL.` 前缀（mod 环境无 `__index=_G`，见 mods.lua `CreateEnvironment`）；自建背心的 bank/build 须与实际动画资源一致。示例仅语法自检，未在游戏内运行。

## 常见陷阱与限制

- **边界：注册名与官方名不能互推。** reflectivevest="清凉夏装"（非"反光背心"）、sweatervest="犬牙背心"、carnival_vest_a/b/c 是**围巾/斗篷/小披肩**（不是背心）、onemanband="独奏乐器"；trunk 是考拉象鼻干，与"象皮"无关。名称键见[验证记录](#验证记录)。
- **边界：`insulated` 是雷电绝缘不是保温。** raincoat 防雷靠 `equippable.insulated`（[equippable.lua#L64-L66](../../../raw/current/scripts/components/equippable.lua#L64-L66)）；防雨靠 waterproofer 20%，两者独立。
- **边界：两条耐久链计时窗口不同。** fueled 族只在装备期耗、可缝补；hawaiianshirt 创建即计时且不可缝补；balloonvest/onemanband 无补给途径。onemanband 的 180 秒是 6 时段（6×seg_time），**不是 6 天**（[tuning.lua#L1730](../../../raw/current/scripts/tuning.lua#L1730)）。
- **边界：balloonvest 是一次性消耗品，但骑乘时不炸。** 引爆入口四条：持有者 `attacked`（**只监听该事件，没有 `blocked`**）／溺水防护 `preventdrowningdamage`／fueled 耗尽／health 归零。前两条走 `DoPop`：先 `DeactiveBalloon`，**持有者骑乘中则既不播爆裂 FX、也不打 AoE、也不立即 Remove**；非骑乘才喷 `balloon_pop_body`＋2 格 5 点 AoE（排除列表写明 INLIMBO/notarget/noattack/flight/invisible/playerghost）后 Remove（[balloons_common.lua#L19-L85](../../../raw/current/scripts/prefabs/balloons_common.lua#L19-L85)）。
- **边界：onemanband 装备期是持续理智负担。** dapperfn 覆盖静态 dapperness：基线 −16/天起，跟随者多于宠物每多一个再 −200/天（[onemanband.lua#L6-L10](../../../raw/current/scripts/prefabs/onemanband.lua#L6-L10)）；招募本身不扣理智。
- **边界：beargervest 减饥饿只在装备期。** 修饰符 onequip 注册、onunequip/入模型移除（[beargervest.lua#L15-L25](../../../raw/current/scripts/prefabs/beargervest.lua#L15-L25)、[L34-L39](../../../raw/current/scripts/prefabs/beargervest.lua#L34-L39)）；×0.75 作用于总饥饿速率。
- **边界：carnival vest 可点燃。** MakeSmallBurnable 不覆写 `canlight`（[standardcomponents.lua#L164-L174](../../../raw/current/scripts/standardcomponents.lua#L164-L174)），burnable 默认 true 并镜像 `canlight` 标签（[burnable.lua#L6-L8](../../../raw/current/scripts/components/burnable.lua#L6-L8)、[L50](../../../raw/current/scripts/components/burnable.lua#L50)）；同族其余衣物无 burnable。

## 验证记录

- 源码审阅：2026-11-07，steam-buildid:26335394。实读 9 个 prefab 文件全文（12 注册逐行）；`recipes.lua` 12 条配方行、`tuning.lua` 全部引用常量行逐行实读；消费端按正文锚点实读：equippable/insulator/waterproofer/fueled/fuel/poppable/leaderrollcall/shadowlevel/flotationdevice 全文，sanity/temperature/moisture/inventory/hunger/drownable/dynamicmusic/playerlightningtarget/burnable/standardcomponents/componentutil/balloons_common 相关段，SGwilson（band 态）、waxwell/SGshadowwaxwell（shadowlevel）、catcoonbrain/SGcatcoon（玩具）、hutch（band 电池）、koalefant（trunk 掉落）、carnival_prizebooth（prototyper 树）、prefabskins（皮肤）、balloonspeed（辨析）。
- 穷举复核：9 源文件 `grep -i quagmire` 无命中（无 quagmire 分支）；equipslot 逐文件确认全部 BODY；`grep components.fuel.fueltype` 全 prefabs 枚举仅 BURNABLE/CAVE/CHEMICAL/NIGHTMARE/PIGTORCH/WORMLIGHT/LIGHTER 有实物，MAGIC/ONEMANBAND/USAGE 无 fuel 物品。
- 名称核验：`languages/chinese_s.po` msgctxt 逐键检索（来源 [chinese_s.po](../../../raw/current/scripts/languages/chinese_s.po)）：RAINCOAT 雨衣 L348477、HAWAIIANSHIRT 花衬衫 L343742、SWEATERVEST 犬牙背心 L350182、REFLECTIVEVEST 清凉夏装 L348577、TRUNKVEST_SUMMER 透气背心 L350837、TRUNKVEST_WINTER 松软背心 L350842、BEARGERVEST 熊皮背心 L339227、BALLOONVEST 充气背心 L339017、CARNIVAL_VEST_A 叽叽喳喳围巾 L340612、CARNIVAL_VEST_B 叽叽喳喳斗篷 L340617、CARNIVAL_VEST_C 叽叽喳喳小披肩 L340622、ONEMANBAND 独奏乐器 L346637。
- 分档说明：`size_class: standard`——12 注册结构同构（单槽、无状态机），机制链均为跨页组件，单页可承载。
- 未核验项：①balloons_common 气球系全量行为（漂浮/碰撞/生成细节，归杂项批次）；②hawaiianshirt 装备位的湿度加速细节；③Maxwell 影术对 shadowlevel 的完整消耗公式（仅核验读取点）；④动画/音效与皮肤 FX；⑤carnival vest 三款视觉差异（仅核验保温差）。
- 结构检查：2026-11-07 第六轮修订后 `validate_page.py`（含 `--check-indexes`）通过（134 行／25972 字节，standard 档上限 240 行／32768 字节，0 issue）；`validate_source_links.py --strict` 通过（103 条引用、43 个源文件，0 error 0 warning）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/hats-environment|环境与功能帽族]]（rainhat/eyebrella 的 HEAD 槽防雨防雷写法、insulator/fueled 同链）
- [[reference/prefabs/balloon_pop_body|balloon_pop_body]]、[[reference/prefabs/balloon_held_child|balloon_held_child]]（气球爆发 FX 与手持气球跟随层）
- [[reference/components/insulator|insulator]]、[[reference/components/waterproofer|waterproofer]]、[[reference/components/equippable|equippable]]（保温/防水/dapperness）
- [[reference/components/fueled|fueled]]、[[reference/components/perishable|perishable]]（两条耐久链）
- [[reference/components/leaderrollcall|leaderrollcall]]、[[reference/components/shadowlevel|shadowlevel]]、[[reference/components/poppable|poppable]]、[[reference/components/flotationdevice|flotationdevice]]（独奏乐器与气球背心专属组件）
- 返回 [[reference/prefabs/wearables-index|穿戴防具与饰品]]、[[reference/prefabs/index|原版 Prefab 索引]]
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 14 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-10）。
