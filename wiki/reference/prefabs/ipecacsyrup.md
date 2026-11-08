---
title: "泻根糖浆（ipecacsyrup）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "prefabs/ipecacsyrup.lua 双注册的沃姆木技能配方：泻根糖浆是 VEGGIE 食物（-20 血／+12.5 饥），吃到 SG 带 ipecacpoop 状态的动物（猪人／狼猪／月猪／皮弗娄牛家族）体内时挂非联网 debuff ipecacsyrup_buff（泻根中毒），按 3 秒一次共 15 次在其位置生成粪便并扣 5 血。"
aliases: ["ipecacsyrup", "ipecacsyrup_buff", "泻根糖浆", "泻根中毒", "泻药", "吐根糖浆", "Syrup of Ipecaca"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/ipecacsyrup.lua", "components/edible.lua", "components/debuff.lua", "components/debuffable.lua", "components/timer.lua", "components/eater.lua", "stategraphs/commonstates.lua", "prefabs/pigman.lua", "prefabs/beefalo.lua", "componentactions.lua", "prefabs/skilltree_wormwood.lua", "prefabs/player_common.lua", "brains/braincommon.lua", "brains/pigbrain.lua", "brains/pigguardbrain.lua", "brains/werepigbrain.lua", "brains/moonbeastbrain.lua", "brains/beefalobrain.lua", "prefabs/wormwood.lua", "screens/redux/scrapbookdata.lua", "entityscript.lua", "tuning.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 泻根糖浆（ipecacsyrup）

## 快速判断

- 职责与适用：`prefabs/ipecacsyrup.lua` 注册 2 个实体：可堆叠食物 `ipecacsyrup`（[fn_syrup L19-L63](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L19-L63)）与服务端专用 debuff `ipecacsyrup_buff`（[fn_buff L128-L162](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L128-L162)）。糖浆本体是普通 VEGGIE 食物；泻药反应（放屁拉屎掉血）只发生在 SG 带 `ipecacpoop` 状态的进食者身上——快照内只有猪人／狼猪／月变猪人 moonpig（po 无官方简中名，[SGpig.lua L648](../../../raw/current/scripts/stategraphs/SGpig.lua#L648)、[SGwerepig.lua L389](../../../raw/current/scripts/stategraphs/SGwerepig.lua#L389)、[SGmoonpig.lua L340](../../../raw/current/scripts/stategraphs/SGmoonpig.lua#L340)）与皮弗娄牛／小皮弗娄牛（[SGBeefalo.lua L1006](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L1006)）。
- 优先使用：给动物喂泻药或仿制"吃后挂 debuff"的食物读本页"泻药 buff 通道"；玩家食用与喂食入口的差异见下表。
- 重要副作用：每次 tick 都会 `SpawnPrefab("poop")` 并按 `IPECAC_DAMAGE_PER_TICK`＝5 扣进食者血（[ipecacsyrup.lua L93-L108](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L93-L108)）；带此 debuff 的猪人／猪守卫／狼猪／月兽／皮弗娄牛 Brain 会被强制插入 Panic 节点（[braincommon.lua L186-L193](../../../raw/current/scripts/brains/braincommon.lua#L186-L193)）；糖浆本身对玩家是 -20 血的食物（[L41](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L41)）。
- 修改边界：buff 实体 `persists=false`、CLASSIFIED、仅服务端（[L141-L145](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L141-L145)），客户端构造即自删（[L131-L135](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L131-L135)）；SG 状态与事件监听在 commonstates 的猪/牛共通段，改放屁动画或加新物种要动它们。未覆盖：进食动作 SG 细节（归 SG 页）、`poop` 实体页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `ipecacsyrup`（泻根糖浆） | `fn_syrup`：网络段（bank/build `ipecacsyrup`、浮水）→ `SetPristine` → master 段（edible VEGGIE、stackable LARGEITEM 10、tradable／inspectable／inventoryitem、小燃烧＋传播、hauntable） | 三维实测：`healthvalue = -TUNING.HEALING_MED`＝-20、`hungervalue = TUNING.CALORIES_TINY`＝12.5、`sanityvalue` 保持默认 0（源码注释明示，[ipecacsyrup.lua L39-L44](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L39-L44)、[tuning.lua L2025](../../../raw/current/scripts/tuning.lua#L2025)、[L2058](../../../raw/current/scripts/tuning.lua#L2058)）；edible 默认值另见 [[reference/components/edible]] |
| `ipecacsyrup_buff`（泻根中毒） | `fn_buff`：客户端自删 → 服务端：Transform、`_tick_count` 初始 15、隐藏非联网实体、CLASSIFIED、debuff＋timer 组件、OnSave/OnLoad | 不进 `assets`；依赖声明在主实体的 `prefabs` 数组（[ipecacsyrup.lua L7-L11](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L7-L11)） |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | `foodtype=VEGGIE`、三维覆写、`SetOnEatenFn` 挂 buff | 服务端；玩家吃无反应（SG 门禁） | [[reference/components/edible]]；[ipecacsyrup.lua L39-L44](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L39-L44) |
| 组件：debuff＋timer（仅 buff） | 计时驱动 tick、`Stop`／`Extend` 由 debuffable 分派 | 仅服务端；无 Replica | [ipecacsyrup.lua L148-L155](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L148-L155)、[[reference/components/debuffable]] |
| 组件：stackable／tradable／burnable 等 | LARGEITEM 10 堆叠、可交易、小燃烧＋传播、hauntable | 两注册只在主实体 | [ipecacsyrup.lua L47-L60](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L47-L60) |
| SG／事件：`ipecacpoop` | `CommonHandlers.OnIpecacPoop` 监听事件转状态；`AddIpecacPoopState` 定义放屁状态 | SGpig／SGwerepig／SGmoonpig／SGBeefalo 四图 | [commonstates.lua L2479-L2481](../../../raw/current/scripts/stategraphs/commonstates.lua#L2479-L2481)、[L2483-L2498](../../../raw/current/scripts/stategraphs/commonstates.lua#L2483-L2498)；挂载点 SGpig.lua L648、SGwerepig.lua L389、SGmoonpig.lua L340、SGBeefalo.lua L1006 |
| Brain：`IpecacsyrupPanicTrigger` | debuff 存续期间强制 `Panic` 乱窜 | pigbrain／pigguardbrain／werepigbrain／moonbeastbrain／beefalobrain | [braincommon.lua L186-L193](../../../raw/current/scripts/brains/braincommon.lua#L186-L193)；消费者 pigbrain.lua L395、pigguardbrain.lua L79、werepigbrain.lua L43、moonbeastbrain.lua L88、beefalobrain.lua L202 |
| 关联 Prefab：`poop` | 每 tick 生成在目标位置并走 `periodicspawner.onspawn` 钩子 | 服务端 tick 期间 | [ipecacsyrup.lua L93-L102](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L93-L102)；poop 页 Wiki 待建 |
| 数据／依赖 | wormwood 角色 prefabs 数组与图鉴注册为声明，不构成生成点 | 快照内无 `SpawnPrefab("ipecacsyrup")` | [wormwood.lua L31](../../../raw/current/scripts/prefabs/wormwood.lua#L31)、[scrapbookdata.lua L733](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L733) |

## 食用与喂食入口

- 糖浆挂 `edible_VEGGIE` 标签（edible 组件按 foodtype 镜像，[edible.lua L14-L22](../../../raw/current/scripts/components/edible.lua#L14-L22)）。
- **玩家自己吃**（右键食用）：SGwilson 无 `ipecacpoop` 状态，`syrup_OnEaten` 的状态检查不通过，只承受食物三维（-20 血／+12.5 饥），无泻药反应（[ipecacsyrup.lua L13-L17](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L13-L17)）。
- **喂给动物**：猪人走 trader 通道（GIVE 动作，[pigman.lua L877-L881](../../../raw/current/scripts/prefabs/pigman.lua#L877-L881)，接受判定 `eater:CanEat`（[L62-L65](../../../raw/current/scripts/prefabs/pigman.lua#L62-L65)），饮食含 VEGGIE（[L746](../../../raw/current/scripts/prefabs/pigman.lua#L746)））；皮弗娄牛同（[beefalo.lua L1203-L1206](../../../raw/current/scripts/prefabs/beefalo.lua#L1203-L1206)、饮食 VEGGIE＋ROUGHAGE（[L1151-L1152](../../../raw/current/scripts/prefabs/beefalo.lua#L1151-L1152)））；跟随宠物／小牲畜走 FEED（[componentactions.lua L1146-L1162](../../../raw/current/scripts/componentactions.lua#L1146-L1162)，handfed／小牲畜分支延伸至 L1203）。进食统一经 `eater` 组件回调 `Edible:OnEaten`（[eater.lua L275](../../../raw/current/scripts/components/eater.lua#L275)）→ `SetOnEatenFn` 检查 SG 状态后 `AddDebuff`。
- 同为 veggie 食客但 SG 无该状态者（如兔人）吃了同样无反应——buff 门禁只看 SG，不看饮食。

## 泻药 buff 通道（ipecacsyrup_buff）

按时间顺序的实际分支：

1. **挂接**：`AddDebuff("ipecacsyrup_buff", "ipecacsyrup_buff")` → 已有同名 debuff 则转 `Extend`，否则生成实体并 `AttachTo`（[debuffable.lua L95-L108](../../../raw/current/scripts/components/debuffable.lua#L95-L108)）。`OnAttached`：跟随目标、启动 `pooptick` 计时 3 秒（`TUNING.IPECAC_TICK_TIME`）、监听目标 `death`／`onremove` 即停（[ipecacsyrup.lua L68-L76](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L68-L76)、[tuning.lua L7853-L7855](../../../raw/current/scripts/tuning.lua#L7853-L7855)）。
2. **叠加语义**：再次喂食走 `OnExtended`，只把 `_tick_count` 重置回 15，不叠加、不重置计时（[ipecacsyrup.lua L78-L81](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L78-L81)）。
3. **每次 tick**（`timerdone`）：`_tick_count <= 0` 则 `debuff:Stop()`；否则计数减 1 并重启 3 秒计时。随后只要 `debuff.target` 仍非空，就生成 1 个 `poop`、转发目标 `periodicspawner.onspawn`、推 `ipecacpoop` 事件（SG 转放屁状态，busy 或死亡时跳过，[commonstates.lua L2473-L2477](../../../raw/current/scripts/stategraphs/commonstates.lua#L2473-L2477)、状态本体 [L2483-L2498](../../../raw/current/scripts/stategraphs/commonstates.lua#L2483-L2498)）、对目标 `health:DoDelta(-5)`（[ipecacsyrup.lua L83-L110](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L83-L110)、[L112-L116](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L112-L116)）。
4. **停止语义**：`Stop` 调 `target:RemoveDebuff`，`OnDetach` 先清空 `target` 再删实体（[debuff.lua L28-L32](../../../raw/current/scripts/components/debuff.lua#L28-L32)、[L44-L51](../../../raw/current/scripts/components/debuff.lua#L44-L51)）；因此计数耗尽后的最后一次计时到期只停 buff、不再产出。净效果：**15 次 tick（约 45 秒）、15 个粪便、共 -75 血**；目标死亡或被移除立即中断。
5. **玩家下线／换人**：玩家 despawn 时 `RemoveOnDespawn` 会移除未设 `keepondespawn` 的 debuff（[debuffable.lua L39-L49](../../../raw/current/scripts/components/debuffable.lua#L39-L49)、[player_common.lua L1434](../../../raw/current/scripts/prefabs/player_common.lua#L1434)）——但本 buff 只可能挂在动物身上，玩家不触发。

## 存读档时序

- 保存：目标实体的 `debuffable:OnSave` 直接取每个 debuff 的 `GetSaveRecord` 写进组件数据（[debuffable.lua L126-L137](../../../raw/current/scripts/components/debuffable.lua#L126-L137)）；buff 的 `OnSave` 记 `_tick_count`（[ipecacsyrup.lua L118-L120](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L118-L120)）。`RegisterDebuff` 设置的 `persists=false` 只影响世界存档散存，不影响此处显式保存。
- 载入：`SpawnSaveRecord` 重建 buff 实体，按实体内顺序先 `OnPreLoad`→组件 `OnLoad`（timer 恢复 `pooptick` 剩余时间，[timer.lua L135-L142](../../../raw/current/scripts/components/timer.lua#L135-L142)）→实体 `OnLoad` 恢复计数（[entityscript.lua L1988-L2013](../../../raw/current/scripts/entityscript.lua#L1988-L2013)、[ipecacsyrup.lua L122-L126](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L122-L126)）；随后 `debuffable:OnLoad`→`RegisterDebuff`→`AttachTo` 重跑 `OnAttached`——此时 `StartTimer` 发现同名计时已存在仅打印告警返回，**不重置剩余时间**（[timer.lua L36-L40](../../../raw/current/scripts/components/timer.lua#L36-L40)）。读完档泻药流程无感续跑。

## 获得路径

- **合成**：红蘑菇（红）×1＋蜂蜜×1＋变质食物×1，`TECH.NONE`，需沃姆木技能节点 `wormwood_syrupcrafting`（[recipes.lua L342](../../../raw/current/scripts/recipes.lua#L342)、[skilltree_wormwood.lua L148-L151](../../../raw/current/scripts/prefabs/skilltree_wormwood.lua#L148-L151)）。
- 无其他生成点：快照内除配方与图鉴注册外无 `SpawnPrefab("ipecacsyrup")` 消费者。

## 常见陷阱与限制

- **误区：玩家吃泻根糖浆会拉肚子。** buff 挂接门禁是进食者 SG 的 `ipecacpoop` 状态（[ipecacsyrup.lua L13-L17](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L13-L17)）；玩家 SG 没有该状态，只有食物三维惩罚。
- **边界：buff 不联网。** `ipecacsyrup_buff` 是服务端专用隐藏实体，客户端读不到其存在，模组想在客户端 UI 显示"泻根中毒"需要自己同步。
- **边界：掉血不走战斗。** 每 tick 的扣血用 `health:DoDelta` 直接结算（来源记为 buff 实体），不触发受击事件链、不可被护甲减免。
- **边界：`poop` 的归属。** 粪便生成在目标脚下但不进其背包；若目标带 `periodicspawner`（如养殖互动）会走其 `onspawn` 钩子（[ipecacsyrup.lua L97-L100](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L97-L100)）。

## 验证记录

- 源码审阅：2026-11-10，steam-buildid:26474873。`prefabs/ipecacsyrup.lua` 全文（双注册、buff 全部回调）；`edible`（默认值与标签镜像）、`debuff`／`debuffable`（挂接／Extend／移除／存读档／RemoveOnDespawn）、`timer`（StartTimer 冲突行为与存读档）、`eater`（OnEaten 回调点）相关段；`commonstates` 的 IpecacPoop 定义与 4 个 SG 挂载点反查；pigman／beefalo 的饮食与 trader 喂食段、FEED 候选反查；Brain Panic 触发器与五个消费 Brain；`entityscript.lua` 载入顺序段；tuning 三常量、edible 默认值对照；配方与技能节点；`chinese_s.po` 两个 `NAMES.*` 键实显（泻根糖浆／泻根中毒）。
- 未覆盖：`poop`／`flies` 等产出实体页、SG `ipecacpoop` 状态在全部皮肤/变体下的表现、`poopcloud` 无关。
- 游戏运行：未运行。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
