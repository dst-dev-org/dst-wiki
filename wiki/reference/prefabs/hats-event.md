---
title: "事件装扮与角色帽族（tophat、pumpkinhat 等 28 注册）"
created: "2026-11-07"
updated: "2026-11-08"
type: prefab
summary: "hats.lua 中 28 顶事件/角色/装扮帽（HEAD 槽）：角色技能帽（高礼帽魔术化、羽毛帽引鸟、松树先锋队帽、幽魂花冠）、年兽神龛舞装三件套、万圣节无面南瓜与九顶舞台剧面具、三顶远古面具、女王面容、月岛海盗猴帽四件、生物产物趣味帽（牛奶帽/气球帽/洞穴花环）；工厂分支、真实配方与获取链、官方名与消费端标签。"
aliases: ["tophat", "高礼帽", "featherhat", "羽毛帽", "walterhat", "松树先锋队帽子", "ghostflowerhat", "幽魂花冠", "dragonheadhat", "幸运兽脑袋", "dragonbodyhat", "幸运兽躯体", "dragontailhat", "幸运兽尾巴", "pumpkinhat", "无面南瓜", "南瓜头", "mask_dollhat", "玩偶面具", "mask_dollbrokenhat", "开裂玩偶面具", "mask_dollrepairedhat", "破碎玩偶面具", "mask_blacksmithhat", "铁匠面具", "mask_mirrorhat", "镜子面具", "mask_queenhat", "女王面具", "mask_kinghat", "国王面具", "mask_treehat", "树木面具", "mask_foolhat", "小丑面具", "mask_ancient_handmaidhat", "鞘翅面具", "mask_ancient_architecthat", "远见者面具", "mask_ancient_masonhat", "工匠面具", "mask_charlie_bosshat", "女王面容", "monkey_mediumhat", "船长的三角帽", "monkey_smallhat", "海盗头巾", "polly_rogershat", "波莉·罗杰的帽子", "salty_doghat", "咸狗帽", "batnosehat", "牛奶帽", "balloonhat", "气球帽", "rabbithat", "洞穴花环", "魔术师高礼帽", "事件帽", "装扮帽", "面具", "舞龙装", "海盗帽"]
tags: [prefab, item, equipment]
size_class: complex
dst_build: "steam-buildid:26474873"
sources: ["prefabs/hats.lua", "recipes.lua", "recipe.lua", "preparednonfoods.lua", "tuning.lua", "techtree.lua", "constants.lua", "languages/chinese_s.po", "components/equippable.lua", "components/inventory.lua", "components/magiciantool.lua", "components/hullhealth.lua", "components/ghostlyelixir.lua", "components/playervision.lua", "components/playerlightningtarget.lua", "components/anchor.lua", "prefabs/player_common_extensions.lua", "stategraphs/SGwilson.lua", "prefabs/ghost.lua", "brains/rabbitbrain.lua", "prefabs/rabbit.lua", "prefabs/player_common.lua", "prefabs/wathgrithr.lua", "prefabs/wes.lua", "prefabs/walter.lua", "prefabs/skilltree_wendy.lua", "prefabs/ghostflower.lua", "prefabs/molebat.lua", "actions.lua", "simutil.lua", "prefabs/statueharp_hedgespawner.lua", "prefabs/hound.lua", "play_the_doll.lua", "play_the_vault.lua", "prefabs/charlie_stage.lua", "prefabs/vaultroom_defs.lua", "prefabs/atrium_gate.lua", "components/piratespawner.lua", "prefabs/monkeyhut.lua", "prefabs/pirate_stash.lua", "scenarios/chest_abandonedboat.lua", "prefabs/blueprint.lua", "prefabs/perdshrine.lua", "prefabskins.lua", "standardcomponents.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 事件装扮与角色帽族（tophat、pumpkinhat 等 28 注册）

## 快速判断

- 职责与适用：`prefabs/hats.lua` 经 `MakeHat` 注册 85 顶玩法帽，本页收其中 **28 顶事件／角色／装扮帽**（全部 HEAD 槽）：角色技能帽 4（高礼帽、羽毛帽、松树先锋队帽、幽魂花冠）、年兽神龛舞装 3（幸运兽头/躯/尾）、万圣节系 14（无面南瓜＋玩偶面具族 9＋远古面具 3＋女王面容）、月岛海盗猴帽 4、生物产物趣味帽 3（牛奶帽、气球帽、洞穴花环）。工厂骨架、装备换装三路与 12 个 FX／技术实体注册归 [[reference/prefabs/hats|帽子跟随层与帽灯特效族]]；防护/战斗帽归 [[reference/prefabs/hats-armor|防护与战斗帽族]]；环境功能帽归 [[reference/prefabs/hats-environment|环境与功能帽族]]。
- 优先使用：查"某顶事件帽的组件/配方/获取"→[分族注册表](#注册与构造)；查"装备后身份/保护标签实际改了什么"→[身份与保护标签消费端](#身份与保护标签消费端)；查"从哪里获得"→[获取链](#获取链)；复刻"装备即身份"写法从[典型接入](#典型接入)进入。
- 重要副作用：多数帽的玩法是**给持有者加标签**（`master_crewman`、`boat_health_buffer`、`rabbitdisguise`、`ghost_ally`），卸下即移除；另有一类是**加在帽本体、靠装备扫描消费**的标签（`ancient_reader` 写面具本体、`spook_protection` 写南瓜帽、`elixir_drinker` 写幽魂花冠，消费端用 `EquipHasTag` 或把帽当动作目标），两者清理位置不同、别混写。无面南瓜的 equippable 组件**默认被移除**、雕出脸后才动态加回；幸运兽舞装仅在跳舞时消耗燃料；气球帽装备期受击即爆（只监听 `attacked`，先过 `ShouldProcOnAttackedOrBlocked`）。
- 修改边界：只覆盖帽本体工厂、组件组合、配方/获取与消费端标签；帽的跟随层/帽灯/地面脸 FX、皮肤 build 资源、动画包内容不在本页（见 [[reference/prefabs/hats|hats.md]] 与[验证记录](#验证记录)）。数值为快照源码推导值，未运行测量。

## 注册与构造

### 工厂与共同骨架

`MakeHat(name)` 的注册名是 `name.."hat"`、动画 build 是 `"hat_"..name`（[hats.lua#L16-L20](../../../raw/current/scripts/prefabs/hats.lua#L16-L20)；`prefabname` 全文件仅初始化与 [L6984-L6986](../../../raw/current/scripts/prefabs/hats.lua#L6984-L6986) 注册两处引用，无改名分支，故 28 注册名一律短名+`hat`）。`simple()` 骨架统一 Transform/AnimState/Network＋`hat` 标签＋snowmandecor，主模拟装 inventoryitem/inspectable/tradable/equippable（**equipslot 一律 `EQUIPSLOTS.HEAD`**，[hats.lua#L188-L235](../../../raw/current/scripts/prefabs/hats.lua#L188-L235)，[L227](../../../raw/current/scripts/prefabs/hats.lua#L227-L227)）；带 fueled 的帽装备回调经 `_base_onequip` 自动 StartConsuming、卸下自动 StopConsuming（[hats.lua#L40-L42](../../../raw/current/scripts/prefabs/hats.lua#L40-L42)、[L101-L103](../../../raw/current/scripts/prefabs/hats.lua#L101-L103)）。装备表现三路与皮肤协议见 [[reference/prefabs/hats|hats.md]] 骨架小节。面具系另有共享基座 `mask_common`：floater med＋`defaultanim`＋scrapbook `COSTUME`，默认带 fuel（SMALL_FUEL）＋可燃可蔓延，`noburn=true` 时全部省略（[hats.lua#L2704-L2725](../../../raw/current/scripts/prefabs/hats.lua#L2704-L2725)）。本页 28 顶均走 `simple()` 或其包装，**无 `default()` 裸帽**。逐帽分派分支：top [L6760-L6768](../../../raw/current/scripts/prefabs/hats.lua#L6760-L6768)、feather [L6769-L6770](../../../raw/current/scripts/prefabs/hats.lua#L6769-L6770)、walter [L6801-L6802](../../../raw/current/scripts/prefabs/hats.lua#L6801-L6802)、dragon×3 [L6824-L6829](../../../raw/current/scripts/prefabs/hats.lua#L6824-L6829)、batnose [L6844-L6846](../../../raw/current/scripts/prefabs/hats.lua#L6844-L6846)、balloon [L6851-L6854](../../../raw/current/scripts/prefabs/hats.lua#L6851-L6854)、monkey×2 [L6867-L6870](../../../raw/current/scripts/prefabs/hats.lua#L6867-L6870)、polly/salty [L6871-L6878](../../../raw/current/scripts/prefabs/hats.lua#L6871-L6878)、玩偶面具族 [L6887-L6904](../../../raw/current/scripts/prefabs/hats.lua#L6887-L6904)、远古面具 [L6912-L6916](../../../raw/current/scripts/prefabs/hats.lua#L6912-L6916)、ghostflower/rabbit/pumpkin [L6951-L6969](../../../raw/current/scripts/prefabs/hats.lua#L6951-L6969)、charlie [L6980-L6981](../../../raw/current/scripts/prefabs/hats.lua#L6980-L6981)。

单位换算（[tuning.lua#L2213-L2217](../../../raw/current/scripts/tuning.lua#L2213-L2217)、[L2225](../../../raw/current/scripts/tuning.lua#L2225-L2225)、[L2260-L2261](../../../raw/current/scripts/tuning.lua#L2260-L2261)、[L2949-L2950](../../../raw/current/scripts/tuning.lua#L2949-L2950)、[L2033-L2049](../../../raw/current/scripts/tuning.lua#L2033-L2049)）：`total_day_time`=480 秒；dapperness 单位为理智/天（TINY≈10.7／SMALL≈16／MED≈26.7／LARGE≈53.3；CRAZINESS_MED＝−100/`day_time`（day_time＝300 秒 → −0.333 理智/秒；全天 480 秒合计 −160，别按 `total_day_time` 换算））；insulator 保温值为秒（SMALL 60/MED 120）；waterproofer 效果为比例（SMALL 20%/SMALLMED 35%）；perish_warp 默认下 PERISH_FAST 6 天／FASTISH 8 天／MED 10 天／SLOW 15 天。

### 表A 角色与技能帽（4 注册）

| 注册名（官方名） | 组件与关键回调 | 获取 | 证据 |
|---|---|---|---|
| `tophat`（高礼帽） | DAPPERNESS_MED＋waterproof SMALL＋fueled USAGE 8 天耗尽自毁；运行时可变体「魔术师高礼帽」：`ConvertToMagician` 加 `shadow_item`/`nocrafting` 标签＋shadowlevel（默认 2 级）＋magiciantool 组件（组件自带 `magiciantool` 标签）＋影子 FX 跟随/落地展示（`tophat_shadow_fx`），装备改走 top_onequip/onunequip；触发：配方 `tophat_magician` 建造后（onPreBuilt）或存档 `data.magician`（OnSave/OnLoad） | 高礼帽配方蛛丝6（SCIENCE_ONE）；魔术化配方 tophat1＋噩梦燃料2（NONE，builder_tag shadowmagic，product=`tophat`） | [工厂](../../../raw/current/scripts/prefabs/hats.lua#L1044-L1071)、[魔术化](../../../raw/current/scripts/prefabs/hats.lua#L987-L1017)、[显示名](../../../raw/current/scripts/prefabs/hats.lua#L902-L904)、[onPreBuilt](../../../raw/current/scripts/prefabs/hats.lua#L1031-L1035)、[配方1](../../../raw/current/scripts/recipes.lua#L531)、[配方2](../../../raw/current/scripts/recipes.lua#L294)、[tuning](../../../raw/current/scripts/tuning.lua#L2864-L2864)、[magiciantool 标签](../../../raw/current/scripts/components/magiciantool.lua#L7-L7) |
| `featherhat`（羽毛帽） | DAPPERNESS_SMALL＋fueled USAGE 8 天；装备向持有者 birdattractor 写 spawnmodifier：maxbirds +3、刷新延迟 MIN −3/MAX −5 并 ToggleUpdate，卸下移除 | 配方乌鸦羽3＋红雀羽2＋触手斑点2（SCIENCE_TWO） | [装备/卸下](../../../raw/current/scripts/prefabs/hats.lua#L557-L584)、[工厂](../../../raw/current/scripts/prefabs/hats.lua#L586-L603)、[tuning](../../../raw/current/scripts/tuning.lua#L1082-L1083)、[配方](../../../raw/current/scripts/recipes.lua#L544) |
| `walterhat`（松树先锋队帽子） | DAPPERNESS_SMALL＋waterproof SMALL＋夏季保温 SMALL＋fueled USAGE 10 天；装备给持有者 `_sanity_damage_protection` 加 modifier（WALTERHAT_SANITY_DAMAGE_PROTECTION=0.5，Walter 把受伤折算理智时乘该系数），卸下移除；Walter 专属符号逻辑（`headbase_walter_hat`，skinname `walterhat_nature` 例外表）；技能 `walter_camp_walterhat` 激活时改为 waterproof SMALLMED＋保温 MED | 配方蛛丝4（NONE，builder_tag pinetreepioneer，即 Walter 自带标签） | [技能刷新](../../../raw/current/scripts/prefabs/hats.lua#L1542-L1572)、[装备](../../../raw/current/scripts/prefabs/hats.lua#L1590-L1629)、[工厂](../../../raw/current/scripts/prefabs/hats.lua#L1631-L1657)、[配方](../../../raw/current/scripts/recipes.lua#L212)、[tuning](../../../raw/current/scripts/tuning.lua#L4934-L4934)、[消费端](../../../raw/current/scripts/prefabs/walter.lua#L546-L546) |
| `ghostflowerhat`（幽魂花冠） | `open_top_hat` 标签（opentop 装备路）＋DAPPERNESS_MED＋perishable 10 天耗尽自毁＋forcecompostable（绿肥）＋rechargeable；装备给持有者 `ghost_ally` 标签、自身 `elixir_drinker` 标签，卸下反向移除、停掉 elixir_buff 并把 rechargeable 满量回落 | 配方 ghostflower（哀悼荣耀）×6（NONE，builder_skill wendy_ghostflower_hat，Wendy 技能树幽魂花分支） | [工厂](../../../raw/current/scripts/prefabs/hats.lua#L5219-L5245)、[装备/卸下](../../../raw/current/scripts/prefabs/hats.lua#L5186-L5205)、[配方](../../../raw/current/scripts/recipes.lua#L163)、[技能](../../../raw/current/scripts/prefabs/skilltree_wendy.lua#L194-L200) |

### 表B 年兽神龛舞装（3 注册）

| 注册名（官方名） | 组件与关键回调 | 获取 | 证据 |
|---|---|---|---|
| `dragonheadhat`（幸运兽脑袋）／`dragonbodyhat`（幸运兽躯体）／`dragontailhat`（幸运兽尾巴） | 共享工厂 `fns.dragon`：fueled USAGE 1 天、**仅跳舞时消耗**（startdancing 里 StartConsuming，stopdancing/onequiptomodel/unequip StopConsuming，覆盖骨架自动消耗）；`OnStartDancing`/`OnStopDancing` 进舞蹈状态给 `hatdance(_2)` 循环动画并打 `dragondance`＋部位标签（prefab 去尾 `hat`）；骑乘状态不触发；跳舞期每秒重数 2 距离内互相链式相连的舞者（head/body/tail 各计一件、上限 3），动态写 dapperness＝DAPPERNESS_LARGE×件数，停止归零 | 配方各幸运金币×8（PERDOFFERING_ONE，任一 YOT 供奉神龛；各神龛 prototyper 树含 PERDOFFERING=1） | [工厂](../../../raw/current/scripts/prefabs/hats.lua#L2183-L2206)、[跳舞](../../../raw/current/scripts/prefabs/hats.lua#L2135-L2150)、[计数](../../../raw/current/scripts/prefabs/hats.lua#L2092-L2133)、[tuning 燃料](../../../raw/current/scripts/tuning.lua#L3648-L3648)、[配方](../../../raw/current/scripts/recipes.lua#L1188-L1190)、[神龛树](../../../raw/current/scripts/tuning.lua#L1320-L1357)、[perdshrine](../../../raw/current/scripts/prefabs/perdshrine.lua#L101-L101) |

### 表C 万圣节：无面南瓜与面具（14 注册）

无面南瓜是本页唯一**注册时不可装备**的帽（见[陷阱](#常见陷阱与限制)）。

| 注册名（官方名） | 组件与关键回调 | 获取 | 证据 |
|---|---|---|---|
| `pumpkinhat`（无面南瓜；雕脸后显示名 `PUMPKINHAT_CARVED` 南瓜头） | 主模拟**移除 equippable**；carvable（pumpkinhatcarvable＋雕刻弹窗）、perishable（万圣节活动期 SLOW 15 天，平日 FASTISH 8 天，耗尽变 spoiled_food）、waxable（打蜡→waxed 永不腐并删 show_spoilage/icebox_valid）、waterproof SMALL＋forcecompostable 绿肥；净标签 show_spoilage/icebox_valid/`goggles`/`spook_protection`/`fullhelm_hat`/waterproofer/waxable，并移除骨架的 snowmandecor；雕出脸（face net_ushortint 5bit×3）后动态加回 equippable（HEAD，fullhelm 换装路），擦脸再移除；装备时理智组件获得光耗免疫＋玩家幽灵免疫＋ghost 光环免疫，生成 `pumpkinhat_fx` 双层跟随脸；皮肤 pumpkinhat_2/3 改 base 换贴图 | 配方南瓜1（TECH.HALLOWED_NIGHTS，仅活动期可配） | [工厂](../../../raw/current/scripts/prefabs/hats.lua#L6139-L6182)、[骨架定制](../../../raw/current/scripts/prefabs/hats.lua#L6099-L6137)、[装备组件动态化](../../../raw/current/scripts/prefabs/hats.lua#L5953-L5996)、[装备回调](../../../raw/current/scripts/prefabs/hats.lua#L5851-L5888)、[打蜡](../../../raw/current/scripts/prefabs/hats.lua#L6058-L6085)、[显示名](../../../raw/current/scripts/prefabs/hats.lua#L6091-L6093)、[配方](../../../raw/current/scripts/recipes.lua#L1414) |
| `mask_dollhat`（玩偶面具）／`mask_dollbrokenhat`（开裂玩偶面具）／`mask_dollrepairedhat`（破碎玩偶面具）／`mask_blacksmithhat`（铁匠面具）／`mask_mirrorhat`（镜子面具）／`mask_queenhat`（女王面具）／`mask_kinghat`（国王面具）／`mask_treehat`（树木面具）／`mask_foolhat`（小丑面具） | 共享工厂 `fns.mask`（=mask_common 默认参数）：fuel SMALL_FUEL＋可燃可蔓延＋scrapbook COSTUME＋floater med；无其他玩法组件，是纯装扮帽 | 无 Recipe2；灌木犬（hedgehound_bush）掉落——竖琴雕像被挖动时在四周刷 4 只携带随机 2 身体+2 面具的灌木犬，击杀掉出（YOTH 活动期追加公主套）；也是舞台剧《The Doll》戏服（演出中 swapmask 换装 doll→broken→repaired→queen）。均有 DeconstructRecipe（拆解法杖返还大理石2+蛛丝4，king/queen 多纸1，mirror 多金4，repaired 大理石1） | [mask_common](../../../raw/current/scripts/prefabs/hats.lua#L2704-L2728)、[分派](../../../raw/current/scripts/prefabs/hats.lua#L6887-L6904)、[掉落池](../../../raw/current/scripts/prefabs/statueharp_hedgespawner.lua#L44-L89)、[击杀掉出](../../../raw/current/scripts/prefabs/hound.lua#L860-L864)、[戏服表](../../../raw/current/scripts/play_the_doll.lua#L6-L14)、[换装](../../../raw/current/scripts/play_the_doll.lua#L240-L240)、[Deconstruct](../../../raw/current/scripts/recipes.lua#L1716-L1724) |
| `mask_ancient_handmaidhat`（鞘翅面具）／`mask_ancient_architecthat`（远见者面具）／`mask_ancient_masonhat`（工匠面具） | `fns.mask_ancient`＝mask_common(noburn)：**无燃料不可燃**；净标签 `ancient_reader`；`stalkerinspectable` 组件；scrapbook_subcat=costume | 生成于远古地窖（vault）特定房间地面（各 1 顶）；也是舞台剧《The Vault》戏服 | [工厂](../../../raw/current/scripts/prefabs/hats.lua#L2757-L2770)、[分派](../../../raw/current/scripts/prefabs/hats.lua#L6912-L6916)、[生成点](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L766-L768)、[戏服表](../../../raw/current/scripts/play_the_vault.lua#L6-L8) |
| `mask_charlie_bosshat`（女王面容） | **直接 `simple()` 不走 mask_common**：无燃料不可燃、无 COSTUME scrapbook；`_camerafocus` net_event＋客户端焦点相机协议（PushCameraFocus→TheFocalPoint StartFocusSource 10/28 渐变）；dappernessfn 仅对 `CHARACTER_KNOWS_CHARLIE`（winona/waxwell）给 CRAZINESS_MED，其余 0；OnSave/OnLoad 支持下次加载后延迟 1.2 秒推焦点 | Shrouden 仪式结算掉落组：仪式完成且 Charlie 未被击败时，在 Atrium Gate 旁生成（同组还有 king_cane、queen_torch） | [工厂](../../../raw/current/scripts/prefabs/hats.lua#L6733-L6749)、[焦点协议](../../../raw/current/scripts/prefabs/hats.lua#L6678-L6726)、[理智](../../../raw/current/scripts/prefabs/hats.lua#L6706-L6709)、[生成点](../../../raw/current/scripts/prefabs/atrium_gate.lua#L698-L710)、[CHARACTER_KNOWS_CHARLIE](../../../raw/current/scripts/constants.lua#L3124-L3128) |

### 表D 月岛海盗猴帽（4 注册）

| 注册名（官方名） | 组件与关键回调 | 获取 | 证据 |
|---|---|---|---|
| `monkey_mediumhat`（船长的三角帽） | fueled USAGE 6 天耗尽自毁＋waterproof SMALL；装备给持有者 `boat_health_buffer` 标签（装到模型也移除）；DAPPERNESS 无 | 海盗船长（primemate）生成即装备并带 `personal_possession`，死亡掉落；废弃船宝箱 66% 出其一（与 small 二选一）；是 polly_rogershat 的配方原料 | [工厂](../../../raw/current/scripts/prefabs/hats.lua#L2813-L2856)、[船长装备](../../../raw/current/scripts/components/piratespawner.lua#L311-L316)、[宝箱](../../../raw/current/scripts/scenarios/chest_abandonedboat.lua#L50-L54)、[tuning](../../../raw/current/scripts/tuning.lua#L6749-L6749) |
| `monkey_smallhat`（海盗头巾） | 同上 fueled/waterproof；装备给持有者 `master_crewman` 标签（收帆加速、起锚加速、船体物理与船桨判定加成） | 火药猴（powder_monkey）船员生成即装备；猴屋给幼猴 gear；废弃船宝箱 | [工厂](../../../raw/current/scripts/prefabs/hats.lua#L2773-L2810)、[船员装备](../../../raw/current/scripts/components/piratespawner.lua#L349-L352)、[猴屋](../../../raw/current/scripts/prefabs/monkeyhut.lua#L89-L89)、[标签消费端](#身份与保护标签消费端) |
| `polly_rogershat`（波莉·罗杰的帽子） | fueled USAGE 6 天，耗尽让鸟飞走并自毁；spawner（child=polly_rogers，SPAWN_TIME 1 天，平时 CancelSpawning）；装备 0 秒后放鸟并 `follower:SetLeader(owner)`，卸下/入模型鸟飞走；migrationpetowner（宠物随玩家迁移/despawn 撤走）；waterproof SMALLMED；鸟死亡时帽子换 `anim_dead` 动画＋图标名 `*hat2`＋持有者换 `swap_hat2` 符号（update_polly_hat_art 统一维护） | 配方猴帽中+金丝雀羽1+黑旗1（TECH.LOST，不可研究）；实际由蓝图学习：海盗船长 50% 掉 `polly_rogershat_blueprint`，海盗藏宝库（pirate_stash）列为重要蓝图 | [工厂](../../../raw/current/scripts/prefabs/hats.lua#L2613-L2654)、[放鸟/收鸟](../../../raw/current/scripts/prefabs/hats.lua#L2517-L2548)、[艺术切换](../../../raw/current/scripts/prefabs/hats.lua#L2450-L2474)、[配方](../../../raw/current/scripts/recipes.lua#L558)、[蓝图掉落](../../../raw/current/scripts/components/piratespawner.lua#L610-L611)、[蓝图教学](../../../raw/current/scripts/prefabs/blueprint.lua#L41-L41)、[藏宝库](../../../raw/current/scripts/prefabs/pirate_stash.lua#L13-L15) |
| `salty_doghat`（咸狗帽） | 与 polly 完全同构（同一组回调），仅 spawner child=`salty_dog`、装备放鸟延迟 0.7 秒 | 贝壳编织者（shellweaver）工坊：saltrock5＋polly_rogershat1（SHELLWEAVER_ONE，manufactured，actionstr SHELLWEAVER） | [工厂](../../../raw/current/scripts/prefabs/hats.lua#L2658-L2700)、[配方](../../../raw/current/scripts/recipes.lua#L987-L987) |

### 表E 生物产物趣味帽（3 注册）

| 注册名（官方名） | 组件与关键回调 | 获取 | 证据 |
|---|---|---|---|
| `batnosehat`（牛奶帽） | dapperness=−TINY＋`flipdapperonmerms`（持有者带 `merm` 标签时符号翻转→鱼人 +TINY）；`restrictedtag="usesvegetarianequipment"`＋refuse_on_restrict（玩家全有此标签、唯 Wigfrid 移除→只有 Wigfrid 装不上）；perishable 0.5 天且**仅装备期腐坏**（equip StartPerishing/unequip Stop）；装备给持有者挂 `hungerregenbuff` debuff＋推 learncookbookstats，卸下移除并让 foodmemory 记录；MakeHauntableLaunchAndPerish | 烹饪锅配方：batnose（蝙蝠鼻）＋kelp＋dairy≥1（priority 55；preparednonfoods 的 noprefab 条目，产物即本帽） | [装备/卸下](../../../raw/current/scripts/prefabs/hats.lua#L3006-L3037)、[工厂](../../../raw/current/scripts/prefabs/hats.lua#L3039-L3063)、[配方](../../../raw/current/scripts/preparednonfoods.lua#L7-L26)、[蝙蝠鼻掉落](../../../raw/current/scripts/prefabs/molebat.lua#L304-L308)、[tuning](../../../raw/current/scripts/tuning.lua#L5643-L5643) |
| `balloonhat`（气球帽） | DAPPERNESS_TINY＋waterproof SMALL＋**equippable.insulated=true（绝缘雷电，见消费端）**；fueled MAGIC 1 天，耗尽即爆（FueledDepletedPop）；poppable：只监听持有者 `attacked`（**没有 `blocked` 分支**），过 `ShouldProcOnAttackedOrBlocked` 后 `poppable:Pop()`；`DoPop` 先 `DeactiveBalloon`（撤碰撞体、加 `notarget`/`NOCLICK`、`persists=false`、不可拾取、关影子），再分支：持有者**骑乘中**则连 FX、AoE、Remove 都不做；已装在 HEAD/BODY 则喷 `balloon_pop_head`/`balloon_pop_body`＋`DoAreaAttack`（显式排除 `INLIMBO`/`notarget`/`noattack`/`flight`/`invisible`/`playerghost`）后 Remove；无主（放地上）走 `balloon_pop_body`＋延迟 AoE＋Remove | 配方气球0（仅要求手持气球类）＋理智×5（NONE，builder_tag balloonomancer——Wes 自带标签；sg_state makeballoon） | [工厂](../../../raw/current/scripts/prefabs/hats.lua#L1394-L1421)、[装备受击](../../../raw/current/scripts/prefabs/hats.lua#L1365-L1391)、[配方](../../../raw/current/scripts/recipes.lua#L377-L377)、[Wes 标签](../../../raw/current/scripts/prefabs/wes.lua#L23-L23)、[制作动作](../../../raw/current/scripts/actions.lua#L831-L840) |
| `rabbithat`（洞穴花环） | **活体帽**：eater（VEGGIE，handfed/fedbyall 标签）吃蔬菜回满耐久；perishable 5 天**始终腐坏**（MakeSmallPerishableCreatureAlwaysPerishing），腐坏时播死亡音、掉 smallmeat 后 ErodeAway；sleeper（夜里睡、光下醒）；DAPPERNESS_TINY；冬季自动换 `rabbit_winter_build`＋图标 rabbithat_winter，持有者精神低时客户端换 `beard_monster` build；装备给持有者 `rabbitdisguise` 标签并生成 `rabbithat_fx` 三层跟随（[[reference/prefabs/hats|hats.md]]）；idle 随机小动画；`small_livestock`/show_spoilage 标签（MakeFeedableSmallLivestockPristine）；float 缩放 {0.9,0.6,0.9} | 兔王商店：兔1＋草6（RABBITKINGSHOP_TWO，actionstr RABBITKINGSHOP，sg_state give）；DeconstructRecipe 同料 | [工厂](../../../raw/current/scripts/prefabs/hats.lua#L5425-L5471)、[季节/疯狂形态](../../../raw/current/scripts/prefabs/hats.lua#L5303-L5320)、[腐坏](../../../raw/current/scripts/prefabs/hats.lua#L5390-L5407)、[装备](../../../raw/current/scripts/prefabs/hats.lua#L5269-L5286)、[配方](../../../raw/current/scripts/recipes.lua#L993-L993)、[Deconstruct](../../../raw/current/scripts/recipes.lua#L1617-L1617)、[pristine 标签](../../../raw/current/scripts/standardcomponents.lua#L1018-L1021) |

## 关联机制导航

### 身份与保护标签消费端

标签效果全部查到消费端；`prefabs` 依赖数组不作证据。

| 标签（帽） | 消费端与实际效果 | 证据 |
|---|---|---|
| `master_crewman`（monkey_smallhat） | 收帆动作速度乘 MASTER_CREWMAN_MULT.LOWER_SAILT_BOOST；起锚速度同乘；船体物理（载重判定）与船桨功效放宽 | [actions.lua#L4851](../../../raw/current/scripts/actions.lua#L4851-L4851)、[anchor.lua#L196](../../../raw/current/scripts/components/anchor.lua#L196-L196)、[boatphysics.lua#L434](../../../raw/current/scripts/components/boatphysics.lua#L434-L434)、[oar.lua#L59](../../../raw/current/scripts/prefabs/oar.lua#L59-L59) |
| `boat_health_buffer`（monkey_mediumhat） | hullhealth 扫描船上带标签实体：船体衰退伤害间隔×2（衰退减半）、碰撞伤害×0.5 | [hullhealth.lua#L48-L65](../../../raw/current/scripts/components/hullhealth.lua#L48-L65)、[L206](../../../raw/current/scripts/components/hullhealth.lua#L206-L206)、[L242-L250](../../../raw/current/scripts/components/hullhealth.lua#L242-L250)、[tuning](../../../raw/current/scripts/tuning.lua#L6760-L6764) |
| `rabbitdisguise`（rabbithat） | 兔子 brain 的目标 notags（不再逃离玩家）；兔子被拾取判定改写（grabbableoverridetag） | [rabbitbrain.lua#L49](../../../raw/current/scripts/brains/rabbitbrain.lua#L49-L49)、[rabbit.lua#L390](../../../raw/current/scripts/prefabs/rabbit.lua#L390-L390) |
| `ghost_ally`＋`elixir_drinker`（ghostflowerhat） | 幽灵目标筛选排除（幽灵不锁定持有者）；ghostlyelixir 可对带标签者使用（喝下幽灵药剂的不再是 Abigail 限定） | [ghost.lua#L21](../../../raw/current/scripts/prefabs/ghost.lua#L21-L21)、[ghostlyelixir.lua#L9](../../../raw/current/scripts/components/ghostlyelixir.lua#L9-L9)、[SGwilson.lua#L1731](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1731-L1731) |
| `ancient_reader`（mask_ancient×3） | 远古铭文可读：阅读动作门禁与 vault 符文/腐蚀体/档案馆石碑/地窖迷雾的检查文本切换 | [actions.lua#L1220](../../../raw/current/scripts/actions.lua#L1220-L1220)、[vault_runes.lua#L7](../../../raw/current/scripts/prefabs/vault_runes.lua#L7-L7)、[atrium_overgrowth.lua#L27](../../../raw/current/scripts/prefabs/atrium_overgrowth.lua#L27-L27)、[archive_props.lua#L187](../../../raw/current/scripts/prefabs/archive_props.lua#L187-L187)、[nightmaregrowth.lua#L33](../../../raw/current/scripts/prefabs/nightmaregrowth.lua#L33-L33) |
| `spook_protection`＋`goggles`（pumpkinhat） | 万圣节 spooked 事件不再进状态、不再扣理智（spooked EventHandler 与 OnSpooked 均查装备标签）；`goggles` 走 playervision 沙尘暴视野 | [SGwilson.lua#L2429-L2440](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2429-L2440)、[player_common_extensions.lua#L899-L902](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L899-L902)、[playervision.lua#L50](../../../raw/current/scripts/components/playervision.lua#L50-L50)、[simutil.lua#L524](../../../raw/current/scripts/simutil.lua#L524-L524) |
| `equippable.insulated`（balloonhat） | IsInsulated（雷电绝缘，非温度）：命中时由默认 `DefaultOnStrike` 扫描装备槽决定免伤，改为推 `lightningdamageavoided`；**不是**"闪电不选你"——`DoStrike` 无论免伤与否都会推 `playerlightningtargeted`，且 `Inventory:ForceNoInsulated` 的门禁优先于装备 | [equippable.lua#L64-L65](../../../raw/current/scripts/components/equippable.lua#L64-L65)、[IsInsulated](../../../raw/current/scripts/components/inventory.lua#L761-L771)、[命中与选中](../../../raw/current/scripts/components/playerlightningtarget.lua#L3-L37)、[ForceNoInsulated](../../../raw/current/scripts/components/inventory.lua#L755-L757) |
| `restrictedtag="usesvegetarianequipment"`（batnosehat） | CanEquip 要求目标带标签；全体玩家在 player_common 加该标签、唯 Wigfrid 移除 | [equippable.lua#L159-L170](../../../raw/current/scripts/components/equippable.lua#L159-L170)、[player_common.lua#L2459](../../../raw/current/scripts/prefabs/player_common.lua#L2459-L2459)、[wathgrithr.lua#L136](../../../raw/current/scripts/prefabs/wathgrithr.lua#L136-L136) |

### 获取链

| 途径 | 本页注册名 | 证据 |
|---|---|---|
| 制作配方 | tophat（+魔术化）、featherhat、walterhat、ghostflowerhat、dragon×3、pumpkinhat、balloonhat、salty_doghat、rabbithat | 上文各表配方列 |
| 烹饪锅 | batnosehat | [preparednonfoods.lua#L7-L26](../../../raw/current/scripts/preparednonfoods.lua#L7-L26) |
| 灌木犬掉落（竖琴雕像/查理舞台） | 玩偶面具族 9 顶（YOTH 期另掉公主套，归 armor 页） | [statueharp_hedgespawner.lua#L56-L89](../../../raw/current/scripts/prefabs/statueharp_hedgespawner.lua#L56-L89)、[charlie_stage.lua#L141-L180](../../../raw/current/scripts/prefabs/charlie_stage.lua#L141-L180) |
| 地窖房间地面 | mask_ancient×3 | [vaultroom_defs.lua#L766-L768](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L766-L768) |
| 仪式结算（Atrium Gate） | mask_charlie_bosshat | [atrium_gate.lua#L698-L710](../../../raw/current/scripts/prefabs/atrium_gate.lua#L698-L710) |
| 海盗生态（船长装备/掉落、猴屋、废弃船宝箱、藏宝库蓝图） | monkey_mediumhat、monkey_smallhat、polly_rogershat（蓝图） | [piratespawner.lua#L300-L360](../../../raw/current/scripts/components/piratespawner.lua#L300-L360)、[monkeyhut.lua#L89](../../../raw/current/scripts/prefabs/monkeyhut.lua#L89-L89)、[chest_abandonedboat.lua#L50-L54](../../../raw/current/scripts/scenarios/chest_abandonedboat.lua#L50-L54)、[pirate_stash.lua#L13-L15](../../../raw/current/scripts/prefabs/pirate_stash.lua#L13-L15) |
| 蓝图学习 | polly_rogershat（TECH.LOST 配方由 blueprint teacher 组件教学） | [blueprint.lua#L12-L41](../../../raw/current/scripts/prefabs/blueprint.lua#L12-L41) |

### 皮肤

`prefabskins.lua` 中本页有皮肤槽的注册：featherhat 3 款、tophat 22 款（含 8 款宠物主题 costume 变体）、walterhat 16 款（`walterhat_nature` 触发 walter_onequip 例外符号表）、monkey_mediumhat 2 款、monkey_smallhat 2 款、pumpkinhat 2 款（pumpkinhat_2/3，帽本体 `base` net_tinybyte 切图标/贴图）；其余 22 注册无皮肤条目。皮肤仅换贴图/build（hats.md 皮肤协议），不换组件。证据：[prefabskins.lua#L758-L762](../../../raw/current/scripts/prefabskins.lua#L758-L762)、[L1303-L1311](../../../raw/current/scripts/prefabskins.lua#L1303-L1311)、[L1508-L1511](../../../raw/current/scripts/prefabskins.lua#L1508-L1511)、[L1914-L1937](../../../raw/current/scripts/prefabskins.lua#L1914-L1937)、[L2170-L2188](../../../raw/current/scripts/prefabskins.lua#L2170-L2188)、[皮肤例外消费](../../../raw/current/scripts/prefabs/hats.lua#L1595-L1607)。

## 典型接入

### 复刻"装备即身份标签"

- 前提与选择：需要"戴着帽子改变 AI 敌我/系统判定"时，原版统一模式是装备回调给**持有者**加标签、卸下回调移除，判定端只认标签。选此模式而非直接扫装备栏，是因为消费端（brain/combat/动作门禁）普遍只查 `HasTag`。
- 生效与撤销：onquip 加、onunequip 删；若帽可能被装到模型（OnEquipToModel），标签清理必须覆盖该路径（monkey_mediumhat 单独实现了 onequiptomodel 清理）。
- 原版依据：monkey_small（[equip/unequip](../../../raw/current/scripts/prefabs/hats.lua#L2778-L2786)）、rabbithat（[equip/unequip](../../../raw/current/scripts/prefabs/hats.lua#L5269-L5286)）、ghostflowerhat（[equip/unequip](../../../raw/current/scripts/prefabs/hats.lua#L5186-L5205)）。

```lua
-- scripts/prefabs/myhat.lua（服务端）：装备即身份的标签协议，复刻 monkey_mediumhat 的挂载位置
local function onequip(inst, owner)
    -- 换装符号照抄 hats 三路之一（_onequip / opentop_onequip / fullhelm_onequip）
    owner.AnimState:OverrideSymbol("swap_hat", inst.build, "swap_hat")
    owner:AddTag("my_disguise")          -- 消费端（brain 的 notags、combat 判定）只认持有者标签
end

local function onunequip(inst, owner)
    owner.AnimState:ClearOverrideSymbol("swap_hat")
    owner:RemoveTag("my_disguise")
end

local function onequiptomodel(inst, owner, from_ground)
    -- 装到模型（展示架/假人）时 Equip 仍会跑 onequip，标签被加到模型实体上，
    -- 模型没有玩法，必须在这里撤掉；换装符号同样要还原
    owner:RemoveTag("my_disguise")
end

local function fn()
    -- ... simple() 骨架同 hats.lua ...
    inst.components.equippable:SetOnEquip(onequip)
    inst.components.equippable:SetOnUnequip(onunequip)
    inst.components.equippable:SetOnEquipToModel(onequiptomodel)
end
```

- 调整与验证：替换点为标签名与消费端判定；挂载点是 `SetOnEquip`／`SetOnUnequip`／`SetOnEquipToModel` 三处（原版写法见 [monkey_medium 三件套](../../../raw/current/scripts/prefabs/hats.lua#L2818-L2831)），模型路径漏写会把标签留在展示架实体上。`Equippable:Equip` 在 `owner:HasTag("equipmentmodel")` 时于 onquip 之后再调 `onequiptomodel`（[equippable.lua#L92-L107](../../../raw/current/scripts/components/equippable.lua#L92-L107)）。示例为协议摘录（text 语义），未在游戏内执行。

## 常见陷阱与限制

- **误区：pumpkinhat 拿到就能戴。** 工厂在主模拟初始化时显式 `RemoveComponent("equippable")`（[hats.lua#L6157](../../../raw/current/scripts/prefabs/hats.lua#L6157-L6157)），只有雕刻出脸后才 `AddComponent("equippable")`（HEAD、fullhelm 路，[L5966-L5981](../../../raw/current/scripts/prefabs/hats.lua#L5966-L5981)）；擦脸会强制卸下并再删组件（[L5982-L5994](../../../raw/current/scripts/prefabs/hats.lua#L5982-L5994)）。给南瓜帽直接写 equippable 相关 mod 代码会空引用。
- **误区：mask_dollrepairedhat 是"修复完成"形态所以叫 Shattered。** 官方英文按 po：`MASK_DOLLBROKENHAT`="Cracked Doll Mask"（开裂）、`MASK_DOLLREPAIREDHAT`="Shattered Doll Mask"（破碎）——prefab 后缀与英文观感不一致，中文按 po 分别为开裂/破碎；DeconstructRecipe 的材料差（repaired 少 1 大理石）也随 prefab 而非随语义。
- **边界：幸运兽舞装燃料只在跳舞时消耗。** 装备回调经 `_base_onequip` 自动 StartConsuming 后，`dragon_equip` 紧接的 stopdancing 立即 StopConsuming 抵消（[dragon_equip](../../../raw/current/scripts/prefabs/hats.lua#L2161-L2164)）；tuning 原注释即"only consumes while dancing"（[tuning.lua#L3648](../../../raw/current/scripts/tuning.lua#L3648-L3648)）。复刻时若只抄 StartConsuming 会让帽子站着掉耐久。
- **边界：`TECH.LOST` 配方与 DeconstructRecipe 都不是"可研究配方"。** TECH.LOST={MAGIC/SCIENCE/ANCIENT=10} 永不满足（[constants.lua#L1324](../../../raw/current/scripts/constants.lua#L1324-L1324)），polly_rogershat 靠蓝图学习；DeconstructRecipe 注册项被 `GetValidRecipe` 显式排除（[recipe.lua#L211-L217](../../../raw/current/scripts/recipe.lua#L211-L217)），只供拆解法杖返还材料（[staff_common.lua#L639](../../../raw/current/scripts/prefabs/staff_common.lua#L639-L639)），不是获取途径。
- **边界：女王面容不走面具基座。** 它是本页唯一不经过 mask_common 的面具（直接 `simple()`），没有 fuel/可燃，也没有 COSTUME scrapbook；按"面具=可燃"写通用处理会漏掉它（mask_ancient×3 也因 noburn 不可燃，但仍有 COSTUME scrapbook）。
- **边界：牛奶帽的理智效果对鱼人反转。** `flipdapperonmerms` 在 GetDapperness 里对带 `merm` 标签持有者取反（[equippable.lua#L199-L200](../../../raw/current/scripts/components/equippable.lua#L199-L200)）——人类戴掉理智、Wurt 戴回理智；同类机制也用在环境页 mermhat 上。装备限制只拦 Wigfrid（其余玩家都有 usesvegetarianequipment）。
- **边界：气球帽的防雷击不在标签里，也不是"闪电不选你"。** 它通过 `equippable.insulated=true` 进 `Inventory:IsInsulated` 扫描（[inventory.lua#L761-L771](../../../raw/current/scripts/components/inventory.lua#L761-L771)），与 raincoat/wagpunkhat 同协议，查 `HasTag` 找不到它；作用是**命中时**由默认 `DefaultOnStrike` 改发 `lightningdamageavoided`，`DoStrike` 仍推 `playerlightningtargeted`，且 `ForceNoInsulated` 优先于装备。除 `attacked` 之外，气球的爆裂还由 fueled 耗尽（`FueledDepletedPop`）与落水免伤（背心 `preventdrowningdamage`）触发——**没有 `blocked` 入口**；且 `DoPop` 在持有者骑乘时只 `DeactiveBalloon`，既不放 FX、不打 AoE 也不立即 Remove。
- **边界：猴子会拾取并装备任何 HEAD 槽掉落物。** powder_monkey 的 OnPickup 对无帽猴子特化"拾起即装备"（[powdermonkey.lua#L83-L97](../../../raw/current/scripts/prefabs/powdermonkey.lua#L83-L97)）；把本页帽丢在月岛地上可能被猴子戴走，写掉落保护时要考虑。

## 验证记录

- 源码审阅：2026-11-07，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/hats.lua` 工厂骨架（L16-L235、L273-L275）与本页 28 注册各自的工厂/回调（feather L557-L603、top L902-L1071、balloon L1365-L1421、walter L1542-L1657、dragon L2092-L2206、polly/salty L2450-L2700、mask_common/mask/mask_ancient L2704-L2770、monkey L2773-L2856、batnose L3006-L3063、ghostflower L5181-L5245、rabbit L5249-L5471、pumpkin L5851-L6182、mask_charlie_boss L6678-L6749）与分派/注册区（L6727-L6986、return 清单 L7860-L7953）；逐帽配方实读 recipes.lua 对应行；消费端与获取链逐行实读（见各表证据列）。
- 穷举复核：`grep -o 'MakeHat("'` 实数 **85**，与 return 清单一致；差集＝85 − 排除清单一(25) − 排除清单二(32) ＝ **28**，本页 28 注册与差集逐名一致（脚本 comm 比对，多/少均无）。
- 名称核验：`languages/chinese_s.po` 以 python UTF-8 读取，28 个注册名全部有 `STRINGS.NAMES.*` 键（TOPHAT=高礼帽 L350470、FEATHERHAT=羽毛帽 L342660、WALTERHAT=松树先锋队帽子 L351725、GHOSTFLOWERHAT=幽魂花冠 L343205、DRAGONHEADHAT=幸运兽脑袋 L342240、DRAGONBODYHAT=幸运兽躯体 L342190、DRAGONTAILHAT=幸运兽尾巴 L342250、PUMPKINHAT=无面南瓜 L347325、PUMPKINHAT_CARVED=南瓜头 L347330、九顶玩偶族面具 L345085-L345150、MASK_ANCIENT×3 L345070-L345080、MASK_CHARLIE_BOSSHAT=女王面容 L345090、MONKEY_MEDIUMHAT=船长的三角帽 L345575、MONKEY_SMALLHAT=海盗头巾 L345580、POLLY_ROGERSHAT=波莉·罗杰的帽子 L347115、SALTY_DOGHAT=咸狗帽 L348945、BATNOSEHAT=牛奶帽 L339120、BALLOONHAT=气球帽 L338995、RABBITHAT=洞穴花环 L348420）；另有 TOPHAT_MAGICIAN=魔术师高礼帽 L350475。无"确无 NAMES 键"项。
- 分派核验结论：两份排除清单与源码逐名核对无讹。检查时发现的 8 注册覆盖缺口（spiderhat、mermhat、antlionhat、bat_bosscorpsehat、蘑菇×4）已由主代理裁决归环境页并补写完成（环境页现 32 注册），armor 页现 25 注册，25+32+28=85 对账吻合；本页按硬校验不收这 8 顶。排除清单中的帽子均属实归页：armor 页 25 顶含 armor/setbonus 或宿主玩法，environment 页 32 顶为保温/视觉/地形/伪装功能帽，与本页 28 顶无成分冲突（本页帽均无 armor/温控组件，唯一例外倾向是 pumpkinhat 的 waterproof SMALL 与 spook_protection——防护对象是"惊吓/幽灵"，非战斗减伤，归本页）。
- 分档说明：`size_class: complex`——28 注册横跨 5 个获取体系与 8 条消费端标签链；无面南瓜的 equippable 动态增删、幸运兽的多实体舞伴计数、波莉系的鸟随帽迁移（spawner+migrationpetowner+艺术切换）需要整体对照说明，超出 standard 单页容量。
- 未覆盖：帽 FX/跟随层内部（归 [[reference/prefabs/hats|hats.md]]）；tophat 魔术化后的储物间协议（归 [[reference/components/magiciantool|magiciantool 组件]]）；皮肤 build 实际资源与观感；BALLOONS.MakeBalloonMasterInit/poppable 的爆裂伤害数值（balloons_common 内部，本页只核到"受击/耗尽即爆"协议）；兔王商店与神龛的活动开关条件（活动调度归世界机制）；Wendy 幽魂花种植链细节。
- 结构检查：2026-11-07 第六轮修订后 `validate_page.py`（含 `--check-indexes`）通过（178 行／42611 字节，complex 档上限 480 行／65536 字节，0 issue）；`validate_source_links.py --strict` 通过（170 条引用、50 个源文件，0 error 0 warning）；本页 Lua 示例经 `parser_probe --syntax-only` 解析通过。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/hats|帽子跟随层与帽灯特效族]]（MakeHat 骨架、装备换装三路、本页帽的 `*_hat_fx` 跟随层与地面脸直构实体）
- [[reference/prefabs/hats-armor|防护与战斗帽族]]（armor/planar 帽与宿主面具）
- [[reference/prefabs/hats-environment|环境与功能帽族]]（保温/视觉/地形功能帽）
- [[reference/prefabs/tophat_shadow_fx|魔术师高礼帽影子特效]]（tophat 魔术化的影子 FX 主页）
- [[reference/components/magiciantool|magiciantool 组件]]（高礼帽魔术化后的储物间工具协议）
- [[reference/components/equippable|equippable 组件]]（equipslot、restrictedtag、insulated、dapperness 消费协议）

返回 [[reference/prefabs/wearables-index|穿戴防具与饰品]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 2 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
