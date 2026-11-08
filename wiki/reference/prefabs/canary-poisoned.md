---
title: "中毒金丝雀（canary_poisoned）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "prefabs/canary_poisoned.lua 的单一注册：鸟笼在毒菌蟾蜍放气的洞穴里按气量概率换出的病鸟——持有时才腐坏（总时长 BIRD_PERISH_TIME/.48、初始 48%，恰好 5 天）、洞穴分支 52 帧爆毒掉 1–2 根羽毛并把 poisonburst 推给半径 3 的帽子、地表分支复原成普通金丝雀，以及死亡／冻结禁拾取、掉 spoiled_food 与睡袋配方材料。"
aliases: ["canary_poisoned", "中毒金丝雀", "金丝雀（中毒）", "病鸟", "爆毒金丝雀", "sleepbomb 材料"]
tags: [prefab, creature]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/canary_poisoned.lua", "stategraphs/SGcanarypoisoned.lua", "prefabs/birds.lua", "prefabs/birdcage.lua", "prefabs/cave.lua", "prefabs/toadstool_cap.lua", "components/perishable.lua", "components/lootdropper.lua", "components/inventoryitem.lua", "components/occupiable.lua", "components/health.lua", "standardcomponents.lua", "entityscript.lua", "prefabs/player_common.lua", "recipes.lua", "prefabs/toadstool.lua", "tuning.lua", "constants.lua", "screens/redux/scrapbookdata.lua", "scrapbook_prefabs.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 中毒金丝雀（canary_poisoned）

## 快速判断

- 职责与适用：单一注册的"物品化病鸟"。它没有 Brain，只有 `SGcanarypoisoned` 一套状态机；被拿出来时按"爆毒／复原"二选一走完就自删，是毒菌蟾蜍帽吸毒的消耗品，也是睡袋的配方材料。[构造与注册](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L36-L119)、[默认状态](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L377-L377)
- 优先使用：查怎么拿到看[获得：鸟笼吸毒换鸟](#获得鸟笼吸毒换鸟)；查什么时候炸、什么时候复原看[状态机：爆毒与复原](#状态机爆毒与复原)；查鲜度与掉落看[数值与计时](#数值与计时)、[死亡、冻结与配方去向](#死亡冻结与配方去向)。
- 重要副作用：爆毒与复原都**不是死亡**——两者都把本体 `persists = false` 后删除；爆毒掉 1–2 根 `feather_canary`，复原掉 5–6 根，但复原出来的 `canary` 是全新鲜度，不继承病鸟剩余鲜度。[爆毒第 52 帧](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L256-L271)、[复原掉落](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L345-L351)、[换回金丝雀](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L14-L25)
- 修改边界：本页写到"鸟侧推送 `poisonburst`"为止。帽子侧的吸收、变暗与演出归 [[reference/prefabs/toadstool_cap_absorbfx]]；睡袋本体（投掷、睡眠云、蓝图门禁）归 [[reference/prefabs/throwables]]；普通金丝雀的鸟行为与吸呼气的宿主行为见[关联机制导航](#关联机制导航)。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `canary_poisoned` | 无包装工厂：`fn` 直构 → 物理／动画／动态阴影／音效／网络 → 七个标签 → `SetPristine` → 非主模拟返回 → 主模拟挂 `SGcanarypoisoned`、`lootdropper`、`inventoryitem`、`health`、`combat`、`inspectable`、可燃烧／可冻结、`hauntable`、`perishable`、三个拾取门禁监听 | 单注册、无 Brain、无 `stackable`（一只一格）、无 `eater`／`occupier`／`tradable`；物理组 `CHARACTERS`、质量 1、球半径 1。[物理](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L46-L54)、[主体](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L80-L116)、[注册](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L119-L119) |

固定细节：bank `canary`、build `canary_build`、开播 `struggle_idle_loop1`（循环）并 `SetTwoFaced`；依赖表里带 `canary`、`spoiled_food`、`feather_canary`。[动画与资源](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L1-L13)、[动画](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L64-L70)

### 名称核验

`inst.name = STRINGS.NAMES.CANARY`，所以简中实显是**金丝雀**，不是注册名对应的"金丝雀（中毒）"；`NAMES.CANARY_POISONED`＝"金丝雀（中毒）"这个键确实存在，但**没有任何实例名引用它**——检视台词走的是另一套键 `DESCRIBE.CANARY_POISONED`（按 `inst.prefab` 大写取），采集册里也没有 `canary_poisoned` 条目。[名称赋值](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L72-L72)、[CANARY](../../../raw/current/scripts/languages/chinese_s.po#L340085-L340087)、[CANARY_POISONED](../../../raw/current/scripts/languages/chinese_s.po#L340090-L340092)、[检视取键](../../../raw/current/scripts/prefabs/player_common.lua#L160-L165)

另有一个显示细节：`sickness` 标签让 `GetAdjective` 的 `small_livestock` 分支直接返回 `nil`，所以它在陈旧／濒死时也不会被加上"饥饿／濒死"前缀。[sickness 标签](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L56-L62)、[形容词分支](../../../raw/current/scripts/entityscript.lua#L2025-L2029)

## 获得：鸟笼吸毒换鸟

只有世界存在 `toadstoolspawner` 时，普通金丝雀才在 `makebird` 里接上吸气逻辑（地表没有这个组件，整段不执行）。[接线条件](../../../raw/current/scripts/prefabs/birds.lua#L401-L417)、[洞穴世界挂载](../../../raw/current/scripts/prefabs/cave.lua#L523-L523)

1. **装笼才开始吸气，离笼转排气。** 入笼由 `occupier.onoccupied`（被换成 `OnCanaryOccupied`）触发：笼子带 `cage` 标签就起周期任务，否则停；离笼／落地另有 `exitlimbo` 监听直接调 `StopInhalingGas` 停吸并转排气——`occupiable:Harvest` 只把 owner 置空、不会再调 `onoccupied`。[装笼回调](../../../raw/current/scripts/prefabs/birds.lua#L167-L173)、[exitlimbo 监听](../../../raw/current/scripts/prefabs/birds.lua#L401-L404)、[停吸转排气](../../../raw/current/scripts/prefabs/birds.lua#L150-L157)、[Harvest 只置空 owner](../../../raw/current/scripts/components/occupiable.lua#L104-L110)
2. **每段结算一次。** 周期按 `SEG_TIME`（30 秒）错峰：放气中 `_gaslevel + 1` 并立刻判毒；没放气则 `- 1`（不低于 0）。离笼后转为排气任务，逐步减回 0。[吸气](../../../raw/current/scripts/prefabs/birds.lua#L141-L148)、[排气](../../../raw/current/scripts/prefabs/birds.lua#L114-L127)
3. **过 12 级才可能中毒。** 判据 `gaslevel > 12 and math.random() * 12 < gaslevel - 12`，等效概率 `(gaslevel-12)/12`：13 级约 8.3%，24 级必中。[判毒](../../../raw/current/scripts/prefabs/birds.lua#L129-L139)
4. **鸟笼换鸟并继承鲜度。** 命中后向世界与笼子各推一次 `birdpoisoned`（数据是 `{ bird, poisoned_prefab = "canary_poisoned" }`）；`OnBirdPoisoned` 删原鸟、笼子进 SICK、生成病鸟往笼里塞，并把病鸟的鲜度按**原鸟剩余百分比**再缩一次（`病鸟48% × 原鸟百分比`）。[推事件](../../../raw/current/scripts/prefabs/birds.lua#L131-L138)、[换鸟](../../../raw/current/scripts/prefabs/birdcage.lua#L457-L477)
5. **气量存档。** 金丝雀自己写 `data.gaslevel`（向上取整），读档向下取整还原；另一只鸟中毒时，全世界金丝雀的气量会被压到 0–5。[存读](../../../raw/current/scripts/prefabs/birds.lua#L175-L183)、[连带清零](../../../raw/current/scripts/prefabs/birds.lua#L409-L413)

鸟笼换鸟时的笼子副作用（SICK 状态、`fall_sick` 动画、货架摆放、`SetBirdType` 修正）见第 4 步；鸟笼的喂食出蛋、腐烂结算与 SICK 动画细节不在本页展开。

## 状态机：爆毒与复原

`SGcanarypoisoned` 的初始状态是 `idle_loop`，**每轮动画结束**只做一次二选一判断：[默认状态](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L377-L377)、[判断点](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L90-L100)

- **复原门禁**：`TheWorld.components.birdspawner ~= nil and TheWorld.components.toadstoolspawner == nil`——即"有普通鸟刷新器、没有蟾蜍放气器"的地表世界。[CheckRecovery](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L9-L12)
- **爆毒门禁**：其余全部情况——原版洞穴两者只满足后者；两个组件都没有的自定义 shard 也会走爆毒，因为复原要求 `birdspawner` 存在。[同一判断点](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L92-L98)
- **爆毒节奏**：`loops` 从 0 起，`math.random(2) <= loops` 命中才进 `explode_pre`，否则 `loops + 1` 重播同一条循环动画；也就是最迟第三轮必进爆毒（第一轮必不进，第二轮 50%）。[自增与判定](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L94-L97)

### 爆毒分支

| 步骤 | 发生什么 | 证据 |
|---|---|---|
| `explode_pre` | 停物理、播 `struggle_idle_loop1`→`struggle_idle_pst`、禁拾取、加 `NOCLICK`；被打断（`onexit` 未置 `explode` 标记）会恢复拾取与点击 | [状态](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L203-L238) |
| `explode` 第 27／48 帧 | 播死亡音效与气球爆裂音效 | [时间轴](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L248-L255) |
| `explode` 第 52 帧（约 1.73 秒） | 关动态阴影、去碰撞、`persists = false`；在抬到 `y = 1` 的位置掉 `math.random(2)` 根 `feather_canary`（1–2 根）；对 `(x, 0, z)` 半径 3 内**带 `absorbpoison` 标签**的实体推 `poisonburst { source = inst }` | [第 52 帧](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L256-L271)、[过滤标签](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L3-L3) |
| 动画结束 | `inst:Remove()`，不留尸体 | [收尾](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L274-L281) |

接收端：全脚本只有毒菌蟾蜍帽（`toadstool_cap`）带 `absorbpoison` 标签，且只在长着蘑菇的状态监听 `poisonburst`，收到后 `setdark` 把自己变暗一整天并叠上吸收演出；帽子处于洞（状态 0）时不监听，所以 Boss 存活期间的爆毒对帽子无效。[标签](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L285-L285)、[监听窗口](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L181-L192)、[变暗与演出](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L69-L91)、[吸收演出注册](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L377-L379)

### 复原分支

| 步骤 | 发生什么 | 证据 |
|---|---|---|
| `recover_pre` | 播 `struggle_idle_pst`，掉一次扇翅音效后进 `recover` | [状态](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L284-L306) |
| `recover` | 播 `struggle_recovery`；第 17 帧无条件转 `recover_transform`（不等动画完） | [时间轴](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L308-L322) |
| `recover_transform` | 禁拾取、`persists = false`、掉 `4 + math.random(2)` 根 `feather_canary`（**5–6 根**） | [掉落](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L345-L351) |
| 动画结束 | `TransformCanary`：原地生成 `canary`，继承位置与朝向；若本体正在燃烧，把新鸟也点燃 | [换鸟](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L14-L25)、[调用](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L359-L366) |

**舍入差异**：任务口径常写成"复原掉 4–6 根羽毛"，源码是 `4 + math.random(2)`，而 Lua 的 `math.random(2)` 返回 1 或 2，所以实际只有 **5 或 6 根**；同一文件里爆毒用的是 `math.random(2)` 当"根数"，正好是 1–2 根。[复原掉落](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L348-L351)、[爆毒掉落](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L262-L266)

## 数值与计时

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒、`perish_warp`＝1，鲜度天数一律按 480 秒／天换算（S1）。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47-L47)

| 项目 | 值 | 推导与证据 |
|---|---|---|
| 生命 | 25（`BIRD_HEALTH = 25 × multiplayer_attack_modifier`，本版系数 1） | [常量](../../../raw/current/scripts/tuning.lua#L1791-L1791)、[赋值](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L91-L93) |
| 鲜度总时长 | `BIRD_PERISH_TIME / .48`＝2400／0.48＝**5000 秒**（约 10.42 天） | [常量](../../../raw/current/scripts/tuning.lua#L1792-L1792)、[赋值](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L106-L110) |
| 初始剩余 | `SetPercent(.48)` → 剩余 2400 秒＝**5 天**（正好等于 `BIRD_PERISH_TIME`） | 同上 |
| 计时开关 | 构造时 `StopPerishing`；进容器／背包 `StartPerishing`；丢到地上 `StopPerishing` | [回调](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L25-L34) |
| 容器倍率 | 鸟笼（owner 带 `cage`，本体带 `small_livestock`）用 `PERISH_CAGE_MULT`＝0.25，即 4 倍慢 | [分支](../../../raw/current/scripts/components/perishable.lua#L89-L91)、[常量](../../../raw/current/scripts/tuning.lua#L2085-L2085) |
| 腐坏产物 | `onperishreplacement = "spoiled_food"`；到点用替换品占原位（在容器里就塞回同格） | [赋值](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L108-L108)、[替换流程](../../../raw/current/scripts/components/perishable.lua#L293-L319) |

三条容易混的语义（S9）：

- `SetPerishTime` 同时改总时长与剩余时间（并把计时重挂）；`SetPercent` 只按新百分比改剩余。[SetPerishTime](../../../raw/current/scripts/components/perishable.lua#L188-L194)、[SetPercent](../../../raw/current/scripts/components/perishable.lua#L223-L233)
- 因为构造后立刻 `StopPerishing`，**站在地上不会腐坏**，`PERISH_GROUND_MULT`＝1.5 对它没有机会生效；只有被塞进容器（含鸟笼、背包）才开始走表。[地面倍率](../../../raw/current/scripts/components/perishable.lua#L97-L105)
- 换鸟时继承的是"百分比缩放"而不是"剩余秒数搬运"：病鸟先有 48%，再乘原鸟百分比，所以原鸟越新鲜，病鸟越接近 5 天。[继承](../../../raw/current/scripts/prefabs/birdcage.lua#L457-L470)

## 死亡、冻结与配方去向

| 交互 | 行为 | 证据 |
|---|---|---|
| 被打死（生命归零） | `death` 状态：停物理、去碰撞、禁拾取、`DropLoot`——`SetLoot({"spoiled_food"})` 是固定清单，稳定掉 1 份，不是概率表 | [死亡状态](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L190-L201)、[掉落配置](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L83-L84)、[固定清单语义](../../../raw/current/scripts/components/lootdropper.lua#L236-L240) |
| 烧死 | 本体可燃烧；已在燃烧／已带 `burnt` 时 `DropLoot` 会把 `spoiled_food` 改写成 `ash`（既无熟态变体，也不是树／巨石这类例外）。**例外**：燃烧是受控燃烧（`GetControlledBurn`）时保留原掉落、只把它点燃 | [可燃烧](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L100-L101)、[改写链](../../../raw/current/scripts/components/lootdropper.lua#L415-L445)、[受控燃烧分支](../../../raw/current/scripts/components/lootdropper.lua#L436-L437) |
| 冻结 | `freeze` → 禁拾取；`unfreeze` → **除非同时处于 `nopickup` 且已死**（`not (A and B)`），否则恢复拾取：活着但带 `nopickup` 也恢复、死了但不在 `nopickup` 状态也恢复 | [门禁](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L15-L23)、[监听](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L112-L114) |
| 杀死后的拾取 | 死亡状态带 `nopickup`，所以 `AllowPickup` 的两个条件同时成立时也不会放开 | 同上 |
| 电击 | 走 `CommonStates.AddElectrocuteStates`，没有专属分支 | [状态补齐](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L370-L375) |
| 陷阱 | 带 `untrappable`，不能进陷阱 | [标签](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L56-L62) |
| 配方材料 | 睡袋 `sleepbomb`：`shroom_skin` 1＋`canary_poisoned` 1，`TECH.LOST`、一次产 4；蓝图由悲惨的毒菌蟾蜍 100% 掉落 | [配方](../../../raw/current/scripts/recipes.lua#L484-L484)、[蓝图掉落](../../../raw/current/scripts/prefabs/toadstool.lua#L138-L139) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| SG：`SGcanarypoisoned` | 全套行为都在状态机里：`idle_loop` 判定、`explode`、`recover`、`death`、`dropped`、冻结／电击 | 主模拟；无 Brain、无 SG 切换 | [安装](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L81-L81)、[状态定义](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L47-L368) |
| 组件：perishable | 总时长 `BIRD_PERISH_TIME/.48`、初始 48%、进容器才计时、笼内 0.25 倍、腐坏换 `spoiled_food` | 主模拟；构造后处于暂停态 | [[reference/components/perishable]]；[赋值](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L106-L110) |
| 组件：inventoryitem | `SetSinks(true)`；挂 `OnPutInInventory`／`OnDropped` 两个回调；`canbepickedup` 由死亡／冻结事件改写 | 主模拟；客户端只读 replica | [[reference/components/inventoryitem]]；[挂载](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L86-L89) |
| 组件：lootdropper | 固定清单 `{"spoiled_food"}`；爆毒与复原的羽毛走 `SpawnLootPrefab` 而不是 `DropLoot`，因此**不吃**燃烧改写 | 主模拟 | [[reference/components/lootdropper]]；[清单](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L83-L84)、[羽毛](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L262-L266) |
| 组件：health | `BIRD_HEALTH`＝25；`murdersound` 换成打小动物的音效 | 主模拟 | [[reference/components/health]]；[赋值](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L91-L93) |
| 组件：combat | 只写 `hiteffectsymbol = "crow_body"`，伤害／射程全用默认 | 主模拟 | [[reference/components/combat]]；[挂载](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L95-L96) |
| 标签：`untrappable`／`small_livestock`／`sickness` | 陷阱拒收；容器腐坏分支与显示前缀都读 `small_livestock`／`sickness` | 两端（pristine 前加） | [标签](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L56-L62)、[笼内倍率](../../../raw/current/scripts/components/perishable.lua#L89-L91) |
| 关联 Prefab：`canary` | 复原分支的产物；本身是 `birds.lua` 的 `makebird("canary", "canary")`（快照内没有 `prefabs/canary.lua`），带 `MakeFeedableSmallLivestock` 与吸气逻辑 | 复原时生成 | [注册](../../../raw/current/scripts/prefabs/birds.lua#L439-L439)、[小牲畜计时](../../../raw/current/scripts/prefabs/birds.lua#L399-L399) |
| 关联 Prefab：`feather_canary`／`spoiled_food` | 羽毛是两条分支的掉落；`spoiled_food` 是打死与腐坏的产物 | 生成／替换 | [依赖表](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L8-L13) |
| 资源：`anim/canary.zip`＋`anim/canary_build.zip` | bank `canary`／build `canary_build`，与普通金丝雀共用；释放演出 `toadstool_cap_releasefx` 也复用这套资源 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L1-L6) |

### 共享规则接入（S1／S3／S7／S8／S9／S10 适用；S2／S4／S5／S6 不适用）

- **S1 时间基准**：本页天数按 `seg_time`＝30、`total_day_time`＝480 秒换算（[基准](../../../raw/current/scripts/tuning.lua#L19-L20)）。
- **S3 标签分层**：本页不是海鱼页，但按"逐注册列标签"的口径给了唯一的注册表与固定标签串；`small_livestock`＋`show_spoilage` 是它被当"小牲畜"处理的关键（[标签](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L56-L62)）。
- **S7 名称**：显示名是"金丝雀"，与注册名对应的"金丝雀（中毒）"不同，见[名称核验](#名称核验)。
- **S8 跨页归属**：`poisonburst` 的接收与帽子变暗演出归 [[reference/prefabs/toadstool_cap_absorbfx]]；睡袋的投掷链归 [[reference/prefabs/throwables]]；本页不展开。
- **S9 鲜度口径**：总时长与初始百分比是两个独立写入（`SetPerishTime` 后 `SetPercent(.48)`），不要按"10.4 天满鲜度"理解。[赋值](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L106-L110)
- **S10 待核验**：爆毒半径 3 与帽子的实战距离窗口、鸟笼 SICK 动画细节均未运行核验；本页只按源码推导（见[验证记录](#验证记录)）。
- **S2／S4／S5／S6 不适用**：不涉及两条钓鱼链、出水投射、重量归属或月亮转换；本页对象没有 `weighable`、`oceanfishable`、`halloweenmoonmutable`。

## 典型接入

### 让鸟笼自己换鸟（推原版 `birdpoisoned` 事件）

- 前提与选择：不要手写"生成病鸟 + GiveItem"——那条路只把鸟放进笼子的 inventory，不会触发笼子的 `OnGetShelfItem`（`birdcage.lua` L479-L486）：既不摘 `occupiable`、也不把 `shelf.cantakeitem` 置真、更不监听 `perished`，玩家正常交互拿不出来。原版入口是鸟笼自己监听的事件：`cage:PushEvent("birdpoisoned", { bird = bird, poisoned_prefab = "canary_poisoned" })`，由 `OnBirdPoisoned` 一次做完七件事——读原鸟鲜度、删原鸟、进 `SICK` 状态并播 `fall_sick`、生成病鸟并按百分比缩放、`GiveItem`、`PutItemOnShelf`（这才触发 `OnGetShelfItem`）、`SetBirdType` 修 build 并启动 SICK 动画任务。**因为处理器会直接 `data.bird:Remove()`，鸟必须从笼子当前的 `occupiable:GetOccupant()` 取**——自己另传一只鸟等于把它删掉。
- 生效范围：这个监听**只在有 `toadstoolspawner` 的世界接线**（`birdcage.lua` L642-L644），也就是原版洞穴；地表推同一事件不会有人处理。要在地表复刻，只能自己实现，并且拿不到 SICK 那套局部表现。
- 生效与撤销：病鸟生成即带 48% 初始值，缩放后立刻是"可用的剩余鲜度"；`PutItemOnShelf` 会把病鸟摆到货架、放出 `occupiable`（鸟不能再被直接塞进笼子），并挂 `perished` 回调：病鸟在货架上烂掉时笼子转成 `SKELETON`（骨架）状态、build 退回 crow。卸模组不影响已生成的病鸟。
- 原版依据：[换鸟实现](../../../raw/current/scripts/prefabs/birdcage.lua#L457-L477)、[货架接管](../../../raw/current/scripts/prefabs/birdcage.lua#L479-L486)、[监听条件](../../../raw/current/scripts/prefabs/birdcage.lua#L642-L644)、[事件源](../../../raw/current/scripts/prefabs/birds.lua#L131-L138)、[鲜度写入](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L106-L110)。

```lua
-- modmain.lua：把换鸟交给鸟笼自己的处理器（仅限有 toadstoolspawner 的世界）
-- 注意：OnBirdPoisoned 会直接 data.bird:Remove()，鸟必须是自己从笼子里取的当前住客
local function PoisonCagedBird(cage)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    if cage == nil or not cage:HasTag("cage") or cage.components.occupiable == nil then
        return
    end

    local bird = cage.components.occupiable:GetOccupant()
    if bird == nil or not bird:IsValid() or bird.components.perishable == nil then
        return
    end

    -- 数据形状与原版 birds.lua 推的一致；处理器会删这只鸟、生成病鸟并摆上货架
    cage:PushEvent("birdpoisoned", {
        bird = bird,
        poisoned_prefab = "canary_poisoned",
    })
end
```

- 调整与验证：验收看三件事——笼子进入 `SICK` 的咳嗽循环、病鸟出现在货架上且能被取出、把病鸟留在货架上让它烂掉时笼子转成 `SKELETON` 骨架态；另外确认传的是笼内当前住客（传空或传错鸟时函数应直接返回，不要删到别的鸟）。若在无 `toadstoolspawner` 的世界调用，函数正常返回但什么也不会发生（监听没接）；这时只能自行 `SpawnPrefab` + `GiveItem` + `shelf:PutItemOnShelf`，并接受没有 SICK 表现。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：病鸟在笼子里会"到点自爆"。** 爆毒要状态机走到 `explode_pre`；在笼子里它只是货架上的物品，原版没有任何"笼内引爆"路径，丢回场景（`OnDropped`）后才由 `dropped`／`idle_loop` 继续推进。[丢地入口](../../../raw/current/scripts/prefabs/canary_poisoned.lua#L29-L34)、[状态机](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L88-L101)。容器内 SG 是否推进未核验，本页不据此下结论。
- **边界：复原看的是世界组件，不是"在不在洞穴"。** `CheckRecovery` 要求 `birdspawner` 存在且 `toadstoolspawner` 为空；两个都没有的 shard 会走爆毒分支。[条件](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L9-L12)
- **边界：复原等于"换一只新鸟"。** `TransformCanary` 新建 `canary`，不会把剩余鲜度搬过去；想留住的鸟别让它复原，也别指望复原能续命。[换鸟](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L14-L25)
- **边界：爆毒只认 `absorbpoison`。** 没有该标签的实体即使站在半径 3 内也收不到 `poisonburst`；原版只有毒菌蟾蜍帽一个持有者，且帽子处于洞状态时不监听。[过滤](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L268-L270)、[帽子](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L181-L185)
- **边界：烧死只掉灰（受控燃烧例外）。** `spoiled_food` 没有熟态变体，燃烧改写会把它换成 `ash`；但受控燃烧（`GetControlledBurn`）走"保留原掉落"分支，不会再换成灰。羽毛走 `SpawnLootPrefab`，两条爆毒／复原分支的羽毛不受这条改写影响。[改写链](../../../raw/current/scripts/components/lootdropper.lua#L415-L445)、[受控燃烧分支](../../../raw/current/scripts/components/lootdropper.lua#L436-L437)
- **未核验**：爆毒半径 3 与帽子的实际距离窗口、鸟笼 SICK 状态的动画细节、`SEG_TIME` 吸气／排气节奏在实战中的观感、容器内 SG 是否推进——均只按源码推导，未运行游戏。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873（`raw/current/BUILD.txt`）。通读 `prefabs/canary_poisoned.lua`、`stategraphs/SGcanarypoisoned.lua`；核对 `prefabs/birds.lua` 的吸气／排气／判毒／存读／`makebird("canary")` 分支，`prefabs/birdcage.lua` 的 `OnBirdPoisoned`／`BUILD_OVERRIDES`／SICK 状态，`components/perishable.lua` 的倍率与替换链，`components/lootdropper.lua` 的固定清单与燃烧改写，`components/inventoryitem.lua` 的 `OnPutInInventory` 触发点，`entityscript.lua` 的形容词分支与 `SetPrefabName` 名称回填，`recipes.lua`／`prefabs/toadstool.lua` 的睡袋配方与蓝图，`tuning.lua` 的时间与鲜度常量。
- 名称核验：`languages/chinese_s.po` 的 `NAMES.CANARY`＝金丝雀、`NAMES.CANARY_POISONED`＝金丝雀（中毒）；实例名取前者，`NAMES.CANARY_POISONED` 只被检视台词那套 `DESCRIBE` 键对应（取键在 `prefabs/player_common.lua` L160-L165）。检索范围：`screens/redux/scrapbookdata.lua` 与 `scrapbook_prefabs.lua` 搜 `canary_poisoned` 均 0 命中（只有普通 `canary`／`feather_canary` 条目）。
- 源码清单勘误：任务口径列的 `prefabs/canary.lua` 在 build 26474873 的快照里不存在，普通金丝雀由 `prefabs/birds.lua` 的 `makebird("canary", "canary")` 注册。
- 未覆盖：鸟笼其它状态（喂食、出蛋、腐烂结算）、`toadstoolspawner` 的放气调度、`sleepbomb` 本体的投掷与睡眠云、`bird_mutant` 分支；采集册条目未逐字段核对。
- R2 独立复审修订（2026-11-09）：unfreeze 门禁改为 `not (nopickup and dead)`（P2-01）；补 `exitlimbo` 停吸转排气（P2-02）；典型接入拆成"核心四步＋笼子副作用"（P2-03）；`CANARY_POISONED` 改述为检视 `DESCRIBE` 键、采集册 0 命中（P2-04）；烧死补受控燃烧例外（P3）。
- 外部复检修订（2026-11-09）：典型接入改为推原版 `birdpoisoned` 事件（P1-3），补 `OnGetShelfItem` 的货架接管（`birdcage.lua` L479-L486）与"监听只在有 `toadstoolspawner` 的世界接线"（L642-L644），并写明手写 GiveItem 会取不出鸟。
- 二次复检修订（2026-11-09）：示例改为先从 `cage.components.occupiable:GetOccupant()` 取笼内当前住客、判空判有效后再推事件，避免 `OnBirdPoisoned` 的 `data.bird:Remove()` 删错鸟（P1-2）。
- 游戏运行：未运行；所有数值与概率为源码推导。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 前端换鸟：[[reference/prefabs/toadstool_cap_absorbfx]]、[[reference/prefabs/bird-egg]]
- 鲜度与拾取：[[reference/components/perishable]]、[[reference/components/inventoryitem]]
- 掉落与配方：[[reference/components/lootdropper]]、[[reference/prefabs/throwables]]

返回 [[reference/prefabs/index|Prefab 索引]]。
