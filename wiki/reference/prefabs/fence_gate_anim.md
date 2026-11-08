---
title: "宿主挂接的附属表现实体（fence_gate_anim 等）"
created: "2026-11-04"
updated: "2026-11-08"
type: prefab
summary: "六类宿主的 12 个表现子实体：服务端生成后 SetParent 跟随宿主、经 highlightchildren／highlightforward 转发高亮；SetParent 不登记 EntityScript.children，宿主删除时的引擎级联未核验（灯链两族除外：不设父、独立存档）。"
aliases: ["fence_gate_anim", "quagmire_park_gate_anim", "wood_chair_back", "stone_chair_back", "hermit_chair_rocking_back", "yoth_chair_rocking_back", "hermitcrab_teashop_front", "junk_pile_side", "junk_pile_blueprint", "yots_lantern_light_chain", "hermitcrab_lightpost_light_chain", "waterplant_base", "附属表现子实体", "椅子背面层", "灯柱灯链"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/fence.lua", "prefabs/furniture_chairs.lua", "prefabs/hermitcrab_teashop.lua", "prefabs/hermithouse.lua", "prefabs/junk_pile_big.lua", "prefabs/lantern_posts.lua", "prefabs/lantern_posts_defs.lua", "prefabs/waterplant.lua", "components/highlight.lua", "components/playercontroller.lua", "prefabskin.lua", "standardcomponents.lua", "entityscript.lua", "recipes.lua", "map/forest_map.lua", "map/maptags.lua", "map/static_layouts/junk_yard1.lua", "map/rooms/forest/terrain_ocean.lua", "map/bunches.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: complex
---

# 宿主挂接的附属表现实体（fence_gate_anim 等）

## 快速判断

- 职责与适用：本页是六类宿主（栅栏门／椅子／隐士茶铺／废料堆／灯柱／水草）在服务端生成、再 `SetParent` 挂到宿主名下的 12 个表现实体：生存模式栅栏门与 quagmire 公园门、椅子背／前层、茶铺正面、废料堆四侧与蓝本、两族灯链、水草底座。它们自身没有独立玩法入口，交互、生命、掉落与存档都在宿主；宿主在本 Wiki 均无独立 Prefab 页（待建），本页只给源码入口。
- 优先使用：要让外观独立于宿主被高亮／排序／播放动画，就复制"空壳子实体 + 宿主持有引用"的模式；要让它跟着宿主摆位，用 `entity:SetParent`；灯柱之间的摆动灯链不要用父子关系，改用 `light_chain` 的伙伴网络（下文对照表）。
- 重要副作用：这些子实体多数 `persists = false`，不单独存档、只在宿主存在期间使用；`junk_pile_blueprint` 还额外用 `Follower:FollowSymbol` 跟宿主符号；`waterplant_base` 额外在服务端挂 `freezable`；`yots_lantern_light_chain`／`hermitcrab_lightpost_light_chain` 相反，是独立保存、读档重挂的联网实体，不能按普通子实体处理。
- 修改边界：客户端在子实体 `fn` 里取 `entity:GetParent()` 时通常还没有父级，父级登记只能放在 `OnEntityReplicated`；本页不展开宿主玩法（门开关、椅子可坐、茶铺酿造、废料堆采集、灯柱点灯、水草战斗），也不核验动画资源是否存在。覆盖 6 个宿主族、7 个工厂函数与两种生命周期，故按 `size_class: complex` 保留整体。
- 名称：12 个注册名在 `chinese_s.po` 中均无 `STRINGS.NAMES` 键，标题使用已核验效果描述＋注册名，不是官方译名。

## 注册与构造

共同分界：宿主在服务端检查通过后 `SpawnPrefab` 子实体、`SetParent` 到宿主、把引用写回宿主字段；客户端收到复制空壳后在 `OnEntityReplicated` 里把自己补登到父实体的高亮列表。灯链的宿主生成点在 `OnBuilt`，子实体出生在两根宿主中点而不是宿主坐标。

宿主生成点（用于定位"谁创建了父实体"）：生存模式的门来自门物品部署（[fence.lua L823-L832](../../../raw/current/scripts/prefabs/fence.lua#L823-L832)），两只门本体在 [L942-L948](../../../raw/current/scripts/prefabs/fence.lua#L942-L948) 注册；三把有背椅走配方与 placer（[recipes.lua L1029-L1039](../../../raw/current/scripts/recipes.lua#L1029-L1039)），`yoth_chair_rocking` 另走可部署套件（L1100），`wood_stool` 无背；茶铺由自身 prefab 与 placer 登记（[hermitcrab_teashop.lua L484-L486](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L484-L486)）；废料堆由世界生成期的静态布局 `junk_yard`（junk_yard1）放置（[maptags.lua L123-L134](../../../raw/current/scripts/map/maptags.lua#L123-L134)、[junk_yard1.lua L124-L132](../../../raw/current/scripts/map/static_layouts/junk_yard1.lua#L124-L132)），本体在 [junk_pile_big.lua L711-L713](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L711-L713) 注册；水草由地形分布 `waterplant_spawner_rough` 经 bunches 映射生成（[terrain_ocean.lua L71](../../../raw/current/scripts/map/rooms/forest/terrain_ocean.lua#L71)、[bunches.lua L70-L80](../../../raw/current/scripts/map/bunches.lua#L70-L80)），本体与底座在 [waterplant.lua L584-L586](../../../raw/current/scripts/prefabs/waterplant.lua#L584-L586) 注册；灯柱分别由 `yots_lantern_post_item` 与 `hermitcrab_lightpost` 配方放置（[recipes.lua L974](../../../raw/current/scripts/recipes.lua#L974)、[L1103](../../../raw/current/scripts/recipes.lua#L1103)）。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `fence_gate_anim`、`quagmire_park_gate_anim` | `MakeWallAnim` 建空壳：Transform／AnimState／Network，八面、宽版 bank/build、`idle`、`FX` 标签；`isdoor` 时须在 `AddAnimState` 之前先加 `can_offset_sort_pos`（源码注释要求该 tag 为首个）再 `Hide("mouseover")`，服务端 `persists = false`（[fence.lua L763-L810](../../../raw/current/scripts/prefabs/fence.lua#L763-L810)）。宿主 `MakeWall` 把 `name.."_anim"` 加进依赖（[L580-L584](../../../raw/current/scripts/prefabs/fence.lua#L580-L584)），服务端出生后设父并写 `highlightforward`（[L676-L679](../../../raw/current/scripts/prefabs/fence.lua#L676-L679)） | 门的宿主 AnimState 只为鼠标悬停排序而加，自身不设 bank/build（[L606](../../../raw/current/scripts/prefabs/fence.lua#L606)），可见美术全在子实体；两只门的注册与动画集见 [L942-L948](../../../raw/current/scripts/prefabs/fence.lua#L942-L948)，活动门只有宽版动画 |
| `wood_chair_back`、`stone_chair_back`、`hermit_chair_rocking_back`、`yoth_chair_rocking_back` | `AddChair` 内 `backfn`：面数跟随宿主 `facings`（0／8／4），同 bank/build、`idle`、`SetFinalOffset(3)`、`Hide("parts")`，服务端 `persists = false`（[L195-L229](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L195-L229)）；`hasback` 时注册 `name.."_back"` 并加进依赖（[L231-L232](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L231-L232)），宿主出生后设父（[L295-L299](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L295-L299)） | 客户端校验父名等于去掉 `_back` 的子实体名（[L187-L193](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L187-L193)）；四把椅子 `hasback = true`（[L352-L357](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L352-L357)）。文件注释说明 "back" 是椅子背面而非独立层，朝上时叠到前面，摇椅用作前层扶手（[L347-L349](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L347-L349)） |
| `hermitcrab_teashop_front` | `fn_front`：同 bank/build、`idle`、`SetFinalOffset(3)`、隐藏 `snow`／`teashop_deco`／`teashop_wheel`／`teashop_shelf`／`teashop_main`／`teashop_mast`／`teashop_shadow`，`FX` 标签，服务端 `persists = false`（[L449-L482](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L449-L482)）；宿主出生后设父、归零局部坐标并登记（[L373-L377](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L373-L377)） | 宿主隐藏 `glow`、`teashop_front` 与 `teashop_straws` 并 `SetFinalOffset(1)`（[L350-L355](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L350-L355)）；宿主把 `PlayAnimation`／`PushAnimation` 同时转发到正面（[L37-L47](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L37-L47)），烧毁时正面播 `burnt`（[L289-L291](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L289-L291)） |
| `junk_pile_side`、`junk_pile_blueprint` | `side_fn`（[L629-L655](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L629-L655)）与 `blueprint_effect`（[L680-L709](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L680-L709)）；宿主 `spawn_sides` 在半径 2、每侧加 36° 偏移处生成 4 个 side 并设父（[L50-L68](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L50-L68)），`ResetFenceBP` 生成蓝本、跟随 `blueprint_follow` 符号后设父（[L231-L241](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L231-L241)） | side 带 `FX`＋`junk_pile` 标签（[L636-L637](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L636-L637)）、每侧 4bit 变化播 `side_idle1/2`（[L70-L82](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L70-L82)）；有 Daywalker 埋入对应侧时 `Hide`（[L84-L88](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L84-L88)），放出时 `Show`（[L275](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L275)）；蓝本自带 `Follower`，`blueprint_pst` 播完由 `animover` 自删（[L674-L678](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L674-L678)），`ClearFenceBP` 负责收尾（[L243-L250](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L243-L250)） |
| `yots_lantern_light_chain`、`hermitcrab_lightpost_light_chain` | `MakeLanternLightChain` 的 `lightchainfn`：Transform／Light／Network，`FX` 标签，带闪烁的 Light、`net_entity` 伙伴、`net_bool` 开灯（[L829-L906](../../../raw/current/scripts/prefabs/lantern_posts.lua#L829-L906)）；名字取 `data.overridelightchainprefabname` 或 `data.name.."_light_chain"`（[L737-L741](../../../raw/current/scripts/prefabs/lantern_posts.lua#L737-L741)）。宿主在 `OnBuilt` 找伙伴、在两点中点生成并 `SetPartners`（[L173-L190](../../../raw/current/scripts/prefabs/lantern_posts.lua#L173-L190)） | `yots_lantern_light_chain` 是旧档兼容用的覆盖名（[lantern_posts_defs.lua L94-L101](../../../raw/current/scripts/prefabs/lantern_posts_defs.lua#L94-L101)），`hermitcrab_lightpost_light_chain` 由 `hermitcrab_lightpost` 拼接而来（[L119-L138](../../../raw/current/scripts/prefabs/lantern_posts_defs.lua#L119-L138)）；此族**不** `SetParent`、没有 `persists = false`，读档走 `OnLoadPostPass` |
| `waterplant_base` | `basefn`：六面、bank `barnacle_plant`、build `barnacle_plant_colour_swaps`、`idle`、隐藏 `stage1/2/3`、`FX` 标签，服务端 `MakeLargeFreezableCharacter` 后 `persists = false`（[L540-L571](../../../raw/current/scripts/prefabs/waterplant.lua#L540-L571)）；宿主出生后设父、归零坐标、对齐随机帧并 `highlightchildren = { base }`（[L422-L431](../../../raw/current/scripts/prefabs/waterplant.lua#L422-L431)） | 宿主 `colouradder:AttachChild(inst.base)` 把花色染色带给底座（[L438-L439](../../../raw/current/scripts/prefabs/waterplant.lua#L438-L439)）；`syncanim`／`syncanimpush` 把动画同步到底座（[L68-L76](../../../raw/current/scripts/prefabs/waterplant.lua#L68-L76)）；`update_barnacle_layers` 按采集进度在底座上 Show／Hide `bud1..3`（[L92-L110](../../../raw/current/scripts/prefabs/waterplant.lua#L92-L110)）；变石头时先 `base:Remove()` 再删宿主（[L129-L134](../../../raw/current/scripts/prefabs/waterplant.lua#L129-L134)） |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 父级挂接：`entity:SetParent` | 门／椅／茶铺／废料堆两侧与蓝本／水草底座把子实体挂到宿主，位置用父局部坐标（茶铺、水草归零，垃圾堆 side 给半径偏移） | 只在服务端设父；客户端由复制关系恢复父级，并在 `OnEntityReplicated` 里补登记 | 门 [L677-L679](../../../raw/current/scripts/prefabs/fence.lua#L677-L679)、椅 [L296-L298](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L296-L298)、茶铺 [L373-L375](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L373-L375)、废料堆 [L61-L66](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L61-L66)、水草 [L422-L424](../../../raw/current/scripts/prefabs/waterplant.lua#L422-L424) |
| `highlightchildren` | 椅子背面、茶铺正面、废料堆两侧、蓝本、水草底座在客户端把自己插进父实体的着色列表；`highlight` 组件遍历该列表一并加色 | 客户端写父字段；服务端自己写的表不参与复制。茶铺客户端用追加，水草客户端直接覆盖为单元素表 | [highlight.lua L88-L90](../../../raw/current/scripts/components/highlight.lua#L88-L90)、[L135-L137](../../../raw/current/scripts/components/highlight.lua#L135-L137)；[茶铺 L441-L447](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L441-L447)、[水草 L533-L537](../../../raw/current/scripts/prefabs/waterplant.lua#L533-L537)、[废料堆 L615-L627](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L615-L627) |
| `highlightforward` | 门把悬停、攻击前闪光转发到门扇子实体，因为宿主自身没有可见美术 | 服务端出生时与客户端复制回调都写一次 | [fence.lua L91-L97](../../../raw/current/scripts/prefabs/fence.lua#L91-L97)；消费端 [playercontroller L2850](../../../raw/current/scripts/components/playercontroller.lua#L2850)、[L4610](../../../raw/current/scripts/components/playercontroller.lua#L4610)；机制见 [[reference/components/highlight]] |
| 排序：`SetFinalOffset`／`SetSortWorldOffset` | 椅背、茶铺正面显式 `SetFinalOffset(3)` 叠在宿主（`-1`／`1`）之前；废料堆 side 与水草底座没有显式偏移，层级只由宿主偏移差决定；门打开时客户端按铰链旋转给门扇补世界排序偏移 | 仅客户端计算；门扇一开一合会重算 | 椅 [L284-L285](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L284-L285) 与 [L215-L216](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L215-L216)、茶铺 [L459](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L459)、废料堆宿主 [L549](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L549)、水草宿主 [L407](../../../raw/current/scripts/prefabs/waterplant.lua#L407)、门 [L74-L88](../../../raw/current/scripts/prefabs/fence.lua#L74-L88) |
| `Follower` | 仅废料堆蓝本：先 `FollowSymbol(inst.GUID, "blueprint_follow")` 再 `SetParent`，既跟宿主符号又是子实体 | 服务端 `ResetFenceBP` 建立；客户端复制后只登记高亮 | [junk_pile_big.lua L235-L240](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L235-L240)；组件页 [[reference/components/follower]] |
| `freezable` | 仅水草底座：服务端 `MakeLargeFreezableCharacter` 添加 `freezable`，等级 4 破碎特效、抗性 3 | 只加在子实体上；宿主冻结路径本页未展开 | [waterplant.lua L566](../../../raw/current/scripts/prefabs/waterplant.lua#L566)、[standardcomponents.lua L353-L359](../../../raw/current/scripts/standardcomponents.lua#L353-L359) |
| `lightpostpartner` | 隐士茶铺／隐士屋挂该组件，把类型设为 `hermitcrab_lightpost`；`hermitcrab_lightpost` 灯柱据此把建筑当作可配对灯位。普通灯柱不挂组件，靠同名 prefab 直接配对 | 双端注册；茶铺未初始化槽位（`IsMultiShackled` 为假）即可配对，隐士屋初始化 3 个槽位、服务端占槽 | [lantern_posts.lua L12-L17](../../../raw/current/scripts/prefabs/lantern_posts.lua#L12-L17)、[L186-L189](../../../raw/current/scripts/prefabs/lantern_posts.lua#L186-L189)、[茶铺 L361-L362](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L361-L362)、[隐士屋 L804-L806](../../../raw/current/scripts/prefabs/hermithouse.lua#L804-L806)；[[reference/components/lightpostpartner]] |
| `entitytracker`、`net_entity`、`net_bool` | 灯链用两个 `net_entity` 同步伙伴、`net_bool` 同步是否亮灯，并在服务端用 `entitytracker` 记录伙伴以便读档重挂 | 伙伴变脏触发客户端重建链节；读档缺伙伴或烧毁则自删 | [lantern_posts.lua L845-L850](../../../raw/current/scripts/prefabs/lantern_posts.lua#L845-L850)、[L899-L901](../../../raw/current/scripts/prefabs/lantern_posts.lua#L899-L901)、[L726-L734](../../../raw/current/scripts/prefabs/lantern_posts.lua#L726-L734)；[[reference/components/entitytracker]] |
| `updatelooper` | 灯链在客户端与非专用服务器上每帧按玩家距离决定就近建链／离远删链，并在 `OnPostUpdate` 中摆位；专用服务器不添加 | 客户端到宿主服务端都跑一份；休眠时摘除更新函数 | [lantern_posts.lua L869-L876](../../../raw/current/scripts/prefabs/lantern_posts.lua#L869-L876)、[L597-L607](../../../raw/current/scripts/prefabs/lantern_posts.lua#L597-L607)、[L656-L668](../../../raw/current/scripts/prefabs/lantern_posts.lua#L656-L668)；[[reference/components/updatelooper]] |
| 关联 Prefab：`collapse_small`／`collapse_big`／`junk_break_fx`／`hermitcrab_fx_med`／`waterplant_rock` | 宿主被拆、放弃或变石头时生成的独立特效／产物，不是本页子实体；子实体仅 `SetParent` 跟随，宿主删除时的引擎级联未核验；烧毁走 `DefaultBurntStructureFn`，不在这些分支内 | 各宿主自己的清理分支 | 门 [L348-L356](../../../raw/current/scripts/prefabs/fence.lua#L348-L356)、茶铺 [L101-L108](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L101-L108) 与 [L263](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L263)、废料堆 [L409](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L409)、水草 [L129-L134](../../../raw/current/scripts/prefabs/waterplant.lua#L129-L134)；[[reference/prefabs/junk_break_fx]] |
| 资源与皮肤：`anim` bank／build | 门 `fence_gate`／`quagmire_park_gate`；椅子 `wood_chair`＋`wood_chair_chair`／`stone_chair`／`hermit_chair_rocking`／`yoth_chair_rocking`；茶铺 `hermitcrab_teashop`；废料堆 `scrappile`；灯 `yots_lantern_post`／`hermitcrab_lightpost`；水草 `barnacle_plant`＋`barnacle_plant_colour_swaps` | 皮肤只在服务端对 `inst.back` 覆盖部件符号：木椅 `chair01_parts`、石椅 `chair01_parts`、隐士摇椅 `chair_parts`；未见 `yoth_chair_rocking` 的覆盖分支 | [prefabskin.lua L250](../../../raw/current/scripts/prefabskin.lua#L250)、[L286](../../../raw/current/scripts/prefabskin.lua#L286)、[L3991](../../../raw/current/scripts/prefabskin.lua#L3991)；资源存在性未核验 |

## 典型接入

### 给自建宿主补一层可高亮的前景表现

- 前提与选择：宿主本体已有可见美术，只有一部分符号需要叠到最前（椅背、茶铺正面模式）；界面与高亮由 `highlightchildren` 统一处理，不要给子实体加交互组件。
- 生效与撤销：子实体只在服务端生成并设父，`persists = false` 使其不单独存档；`SetParent` 不登记 `EntityScript.children`，宿主移除时是否随父实体销毁未核验，自建宿主应在移除前显式 `Remove`。父实体的 `highlightchildren` 双端各写一次，客户端在复制回调里写。
- 原版依据：椅背的空壳＋父级登记与茶铺正面的动画转发（[椅 L195-L232](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L195-L232)、[茶铺 L373-L377](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L373-L377)）。

```lua
-- 宿主 prefab 的服务器段：生成子实体、设父、登记高亮
local function AttachFront(inst)
    local front = SpawnPrefab("my_host_front")
    front.entity:SetParent(inst.entity)
    front.Transform:SetPosition(0, 0, 0)
    front.persists = false
    inst.front = front
    inst.highlightchildren = { front }
end
```

```lua
-- 子实体 prefab：客户端复制完成后再登记到父实体
local function OnReplicated(inst)
    local parent = inst.entity:GetParent()
    if parent ~= nil and parent.prefab == "my_host" then
        parent.highlightchildren = parent.highlightchildren or {}
        table.insert(parent.highlightchildren, inst)
    end
end

local function fn()
    local inst = CreateEntity()
    inst.entity:AddTransform()
    inst.entity:AddAnimState()
    inst.entity:AddNetwork()

    inst.Transform:SetFourFaced() -- 必须与宿主一致
    inst.AnimState:SetBank("my_host")
    inst.AnimState:SetBuild("my_host")
    inst.AnimState:PlayAnimation("idle")
    inst.AnimState:SetFinalOffset(3)

    inst:AddTag("FX")
    inst.entity:SetPristine()

    if not TheWorld.ismastersim then
        inst.OnEntityReplicated = OnReplicated
        return inst
    end

    inst.persists = false
    return inst
end

return Prefab("my_host_front", fn)
```

- 调整与验证：`SetFinalOffset` 数值只要大于宿主的最终偏移即可压在前面；朝向面数、bank/build 必须与宿主一致，否则符号无法对齐；皮肤覆盖要在服务端对子实体单独调用 `OverrideItemSkinSymbol`（原版只对 `inst.back` 这么做）。示例把子实体保存在 `inst.front`；自建宿主应在移除前显式 `front:Remove()`，不要依赖 `SetParent` 的引擎级联。

## 常见陷阱与限制

- 客户端时序：子实体 `fn` 在客户端执行时通常还没有父级，`entity:GetParent()` 为 `nil`；父字段只能写在 `OnEntityReplicated`（[茶铺 L472-L474](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L472-L474)、[水草 L560-L562](../../../raw/current/scripts/prefabs/waterplant.lua#L560-L562)）。服务端不经这个复制回调，同一登记要在服务端出生处另写一次。
- `highlightchildren` 是覆盖还是追加要按子实体数量决定：椅背在客户端直接赋值单元素表（[椅 L188-L192](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L188-L192)），水草同样覆盖（[L533-L537](../../../raw/current/scripts/prefabs/waterplant.lua#L533-L537)）；茶铺用追加容纳正面与隐士蟹（[L441-L447](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L441-L447)），废料堆 side 与蓝本也追加（[L622-L627](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L622-L627)、[L667-L672](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L667-L672)）。多个子实体却用赋值会丢掉前一个。
- 子实体的存活边界：`persists = false` 只写在服务端分支，表示不单独存档；这些子实体多数只以 `SetParent` 跟随宿主，宿主移除路径不一定显式 `Remove`（蓝本等另有专属收尾），没有显式清理的分支在宿主删除时的引擎级联未核验，不能当已保证清理。不要给它挂 `health`、`workable` 等玩法组件。例外是水草底座的 `freezable`（[L566](../../../raw/current/scripts/prefabs/waterplant.lua#L566)），其受击与冻结是否作用在底座上本页未核验。
- 灯链不是普通子实体：它不 `SetParent`，坐标由两端宿主的符号位置与每帧摆动插值写入（[L173-L190](../../../raw/current/scripts/prefabs/lantern_posts.lua#L173-L190)、[lantern_posts.lua L476-L487](../../../raw/current/scripts/prefabs/lantern_posts.lua#L476-L487)）；按父级局部坐标驱动会错位。
- 灯链清理有三层：宿主 `onremove`／`onburnt` 调 `RemoveNeighbourLights` 把整条链移除（[L87-L104](../../../raw/current/scripts/prefabs/lantern_posts.lua#L87-L104)、[L341-L343](../../../raw/current/scripts/prefabs/lantern_posts.lua#L341-L343)）；链本体读档时伙伴缺失或已烧毁则自删（[L726-L734](../../../raw/current/scripts/prefabs/lantern_posts.lua#L726-L734)）；离屏只删链节、本体保留（[L597-L607](../../../raw/current/scripts/prefabs/lantern_posts.lua#L597-L607)）。
- 链节是每端各自建的非联网实体：`break` 动画播完由 `animover` 自删（[L688-L693](../../../raw/current/scripts/prefabs/lantern_posts.lua#L688-L693)），断链动画在 0.2 秒内按距离错峰（[L566-L595](../../../raw/current/scripts/prefabs/lantern_posts.lua#L566-L595)）；不要在网络层期待它们同步。
- 门的可见美术全在子实体上：宿主 `AnimState` 只为排序而建、不设 bank/build（[L606](../../../raw/current/scripts/prefabs/fence.lua#L606)），所有门动画经 `GetAnimState` 落在门扇子实体（[L55-L57](../../../raw/current/scripts/prefabs/fence.lua#L55-L57)、[L358-L361](../../../raw/current/scripts/prefabs/fence.lua#L358-L361)）；给门加自定义动画时改错对象会没有效果。
- 跨模式差异：`quagmire_park_gate` 在 quagmire 服务器模式下不加 `workable`／`combat`／`health`（[L694-L695](../../../raw/current/scripts/prefabs/fence.lua#L694-L695)），该模式里子实体只承担外观与开合，不能套用生存模式的敲击清理。
- 皮肤分支不完整：`yoth_chair_rocking` 有背面子实体但未找到对应 `inst.back` 皮肤覆盖（[prefabskin.lua L3991](../../../raw/current/scripts/prefabskin.lua#L3991) 只处理 `hermit_chair_rocking`），换肤需求要自行补。

## 验证记录

- 源码审阅：2026-11-04，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。覆盖 `fence.lua`、`furniture_chairs.lua`、`hermitcrab_teashop.lua`、`hermithouse.lua`、`junk_pile_big.lua`、`lantern_posts.lua`／`lantern_posts_defs.lua`、`waterplant.lua` 的 12 个注册名、7 个工厂函数、宿主生成点、父级挂接、高亮／排序转发与清理分支；另核验 `highlight.lua`、`playercontroller.lua`、`prefabskin.lua`、`standardcomponents.lua`、`recipes.lua`、`map/forest_map.lua` 的消费与宿主构造点。
- 名称核验：在 `languages/chinese_s.po` 以 `STRINGS.NAMES.<注册名>` 检索 12 个名字均为 0 命中，故不使用官方名；该 `.po` 已列入 `sources` 作为检索依赖，标题为已核验效果描述，非官方译名。
- 未覆盖：动画 zip、build 与具体符号是否存在；皮肤视觉差异；`waterplant_base` 上 `freezable` 的实际受击／冻结路径；多人端同步与运行时表现。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 通过；两段 Lua 示例经 `parser_probe.exe --syntax-only` 通过。

## 相关链接

- [[reference/components/highlight]]（`highlightchildren` 的着色实现）
- [[reference/components/lightpostpartner]]（灯柱伙伴与槽位）
- [[reference/components/sittable]]（椅子的占用与坐姿入口）
- [[reference/prefabs/junk_break_fx]]（废料堆与围栏的破裂特效）
- [[reference/prefabs/fx-world-index|特效 Prefab 索引·生物与环境]]

返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-03）。
