---
title: 基础精炼材料（boards、cutstone、rope、papyrus）
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: 四种由采集材料精炼得到的基础建造材料（木板、石砖、绳子、莎草纸）：注册与构造、配方产出（含 Woodie／Walter 专属替代）、堆叠／燃料／修补参数与进贡／使用方向的实际接线。
aliases: ["boards", "cutstone", "rope", "papyrus", "木板", "石砖", "绳子", "莎草纸", "精炼材料", "精制材料"]
tags: [prefab, item]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/boards.lua", "prefabs/cutstone.lua", "prefabs/rope.lua", "prefabs/papyrus.lua", "recipes.lua", "prefabs/woodie.lua", "prefabs/antlion.lua", "prefabs/pigking.lua", "prefabs/vault_lobby_exit.lua", "actions.lua", "tuning.lua", "constants.lua", "components/stackable.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 基础精炼材料（boards、cutstone、rope、papyrus）

## 快速判断

- 职责与适用：四个独立文件注册的 `inventoryitem` 建造材料（无共享工厂，同套标准 helper 骨架）：`boards`（木板）、`cutstone`（石砖）、`rope`（绳子）、`papyrus`（莎草纸）。可借鉴：精炼物品的标准接线与"配方产出→建造消费"链路。[boards 注册](../../../raw/current/scripts/prefabs/boards.lua#L57)、[cutstone 注册](../../../raw/current/scripts/prefabs/cutstone.lua#L50)、[rope 注册](../../../raw/current/scripts/prefabs/rope.lua#L81)、[papyrus 注册](../../../raw/current/scripts/prefabs/papyrus.lua#L51)
- 优先使用：查配方产出与专属替代见[获取途径](#获取途径与入栏路径)；调参数查[数值表](#数值与参数)；采集侧原料见 [[reference/prefabs/basic-gathered-materials]]；矿物原料见 [[reference/prefabs/basic-mineral-materials]]。
- 重要副作用：rope 挂 `useabletargeteditem`——对 `canrope` 目标使用时走 `ACTIONS.USEITEMON.fn` → `useabletargeteditem:StartUsingItem` → `target:OnUsedRope`（"TIE_ONTO" 只是 `GetUseItemOnVerb` 返回的界面动词文案键，不是动作名）；cutstone 设 `inventoryitem:SetSinks(true)` 并带 `rocktribute`（蚁狮进贡）；boards/papyrus/rope 挂 fuel＋可燃，cutstone 不可燃。[rope 使用接线](../../../raw/current/scripts/prefabs/rope.lua#L10-L27)、[USEITEMON 动作](../../../raw/current/scripts/actions.lua#L3351-L3363)、[cutstone sinks](../../../raw/current/scripts/prefabs/cutstone.lua#L33-L34)
- 修改边界：组件在主模拟（`SetPristine` 分支后）挂载；本页不覆盖 boards 的建造消费方清单与 woodie_boards／walter_rope 专属配方内部行为（仅核验产出入口）。

## 注册与构造

四个注册各自独立成文件；骨架与采集材料一致：公共端建实体／bank／build／`pickupsound`／差异标签／浮水 → `SetPristine` → 主模拟挂组件。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `boards` | 单文件单注册 `Prefab("boards", fn, assets)`；主模拟挂 edible、stackable、inspectable、burnable/propagator、inventoryitem、repairer、fuel、hauntable | 可食 `FOODTYPE.WOOD`（治疗／饥饿均 0）；浮水 `("med", .15, {1.15, .8, 1.15})`；repairer 带 `boatrepairsound`。 [组件段](../../../raw/current/scripts/prefabs/boards.lua#L29-L52) |
| `cutstone` | 同骨架；主模拟挂 stackable、inspectable、inventoryitem（`SetSinks(true)`）、tradable（`rocktribute=3`）、bait、repairer、hauntable（仅 `MakeHauntableLaunch`） | 公共端标签 `molebait`；**无** fuel／burnable（不可燃）、无浮水、不可食；作祟只弹飞不点燃。 [组件段](../../../raw/current/scripts/prefabs/cutstone.lua#L29-L45) |
| `rope` | 同骨架；文件顶部定义 `OnUsedRope`／`ResetInUse`／`UseableTargetedItem_ValidTarget`／`GetUseItemOnVerb`，主模拟挂 useabletargeteditem 等 | 公共端设 `inst.UseableTargetedItem_ValidTarget`（目标须 `canrope`）与 `inst.GetUseItemOnVerb`（"TIE_ONTO"）；标签 `cattoy`；浮水 `("small", 0.05)`。 [使用接线](../../../raw/current/scripts/prefabs/rope.lua#L6-L31)、[组件段](../../../raw/current/scripts/prefabs/rope.lua#L61-L76) |
| `papyrus` | 同骨架；主模拟挂 stackable、inspectable、fuel、tradable、burnable/propagator、hauntable、inventoryitem | 公共端标签 `cattoy`＋`bookcabinet_item`（书柜收纳）；不可食；浮水 `("med", nil, 0.75)`。 [组件段](../../../raw/current/scripts/prefabs/papyrus.lua#L32-L46) |

### 获取途径与入栏路径

四个注册都由配方产出（`TECH.SCIENCE_ONE`，科学仪器一档），原料为本族采集材料（[[reference/prefabs/basic-gathered-materials#获取途径与入栏路径|采集与入栏路径见彼页]]）：

| 注册名 | 主配方 | 专属替代（已核验） |
|---|---|---|
| `rope` | 采下的草×3，`TECH.SCIENCE_ONE`（[recipes.lua#L457](../../../raw/current/scripts/recipes.lua#L457)） | `walter_rope`：草×2、`TECH.NONE`、`product="rope"`，仅 `walter_camp_rope` 技能；主配方对持该技能者隐藏（`no_builder_skill`）。 [替代配方](../../../raw/current/scripts/recipes.lua#L458) |
| `boards` | 木头×4，`TECH.SCIENCE_ONE`（[recipes.lua#L459](../../../raw/current/scripts/recipes.lua#L459)） | `woodie_boards`：lucy×0＋木头×3、`TECH.NONE`、`product="boards"`，且要求 `builder_skill="woodie_human_lucy_1"`；主配方对带 `cancarveboards` 标签者隐藏——该标签只在 Woodie 激活同一技能后由 itemget/itemlose 监听维护（条件＝携带 lucy），无技能时被主动移除。 [技能门禁与监听维护](../../../raw/current/scripts/prefabs/woodie.lua#L1719-L1740)、[标签赋值](../../../raw/current/scripts/prefabs/woodie.lua#L1715-L1717)、[替代配方](../../../raw/current/scripts/recipes.lua#L173) |
| `cutstone` | 石头×3，`TECH.SCIENCE_ONE`（[recipes.lua#L460](../../../raw/current/scripts/recipes.lua#L460)） | 无 |
| `papyrus` | 采下的芦苇×4，`TECH.SCIENCE_ONE`（[recipes.lua#L461](../../../raw/current/scripts/recipes.lua#L461)） | 无 |

拾取与入栏链与采集材料相同（PICKUP→GiveItem）；本页四注册另可从精炼配方、掉落与容器转移获得，未逐一枚举生成点。

### 数值与参数

tuning 基准同采集页（`seg_time`=30、`STACK_SIZE_LARGEITEM`=10、`STACK_SIZE_MEDITEM`=20、`STACK_SIZE_SMALLITEM`=40，[tuning.lua#L19](../../../raw/current/scripts/tuning.lua#L19)、[L81-L83](../../../raw/current/scripts/tuning.lua#L81-L83)）。

| 注册名 | 堆叠上限 | 燃料值（秒） | 燃烧时长（秒） | 浮水 | 沉水 | 修补材料／回复值 |
|---|---|---|---|---|---|---|
| `boards` | `STACK_SIZE_LARGEITEM`=10 | `LARGE_FUEL`=180 | `LARGE_BURNTIME`=30 | 有 | 否 | WOOD／`REPAIR_BOARDS_HEALTH`=50 |
| `cutstone` | 10 | 无 fuel／burnable | — | 无 | `SetSinks(true)` | STONE／`REPAIR_CUTSTONE_HEALTH`=50 |
| `rope` | 未设→默认 20 | `MED_FUEL`=45 | `LARGE_BURNTIME`=30 | 有 | 否 | 无 repairer |
| `papyrus` | `STACK_SIZE_SMALLITEM`=40 | `SMALL_FUEL`=15 | `SMALL_BURNTIME`=7.5 | 有 | 否 | 无 repairer |

注意 rope 的燃烧时长（`LARGE_BURNTIME`=seg_time=30 秒）**小于**其燃料值（`MED_FUEL`=seg_time×1.5=45 秒喂出）：燃料值是喂给 `fueled` 对象的秒数、燃烧时长是自身燃尽秒数，两条数值各取不同 tuning 档、彼此独立。[rope fuel](../../../raw/current/scripts/prefabs/rope.lua#L66-L71)、[tuning L1674-L1683](../../../raw/current/scripts/tuning.lua#L1674-L1683)、[修补值](../../../raw/current/scripts/tuning.lua#L2104-L2117)

名称键经简中 .po `msgctxt` 核验：`NAMES.BOARDS`=木板（Boards）、`NAMES.CUTSTONE`=石砖（Cut Stone）、`NAMES.ROPE`=绳子（Rope）、`NAMES.PAPYRUS`=莎草纸（Papyrus）。[BOARDS](../../../raw/current/scripts/languages/chinese_s.po#L339585-L339587)、[CUTSTONE](../../../raw/current/scripts/languages/chinese_s.po#L341965-L341967)、[ROPE](../../../raw/current/scripts/languages/chinese_s.po#L348785-L348787)、[PAPYRUS](../../../raw/current/scripts/languages/chinese_s.po#L346745-L346747)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：inventoryitem | 拾取／入栏／转移；cutstone `SetSinks(true)` 掉水下沉可打捞 | 四注册；主模拟 | [[reference/components/inventoryitem#关键接口]]；[cutstone](../../../raw/current/scripts/prefabs/cutstone.lua#L33-L34) |
| 组件：stackable | 上限 10／20／40，见数值表；replica 同步 | 四注册；主模拟 | [[reference/components/stackable#关键接口]]；[boards](../../../raw/current/scripts/prefabs/boards.lua#L34-L35)、[默认](../../../raw/current/scripts/components/stackable.lua#L28) |
| 组件：fuel／burnable | 燃料值与燃尽时长独立（见数值表）；cutstone 完全不接燃料线 | boards／rope／papyrus；主模拟 | [[reference/components/fuel#关键接口]]、[[reference/components/burnable#关键接口]]；[boards](../../../raw/current/scripts/prefabs/boards.lua#L39-L40)、[L49-L50](../../../raw/current/scripts/prefabs/boards.lua#L49-L50) |
| 组件：edible | boards 可食 `FOODTYPE.WOOD`（0／0，无实际收益） | 仅 boards；主模拟 | [[reference/components/edible#关键接口]]；[boards](../../../raw/current/scripts/prefabs/boards.lua#L29-L32) |
| 组件：repairer | 修 WOOD／STONE 类可修物；boards 带 `boatrepairsound` | boards／cutstone | [[reference/components/repairer#关键接口]]；[boards](../../../raw/current/scripts/prefabs/boards.lua#L44-L47)、[cutstone](../../../raw/current/scripts/prefabs/cutstone.lua#L41-L43) |
| 组件：useabletargeteditem | rope 的对目标使用：`ValidTarget` 要求目标 `canrope`，触发 `target:OnUsedRope(inst, doer)`，动词 TIE_ONTO | 仅 rope；主模拟组件＋公共端回调 | [[reference/components/useabletargeteditem#关键接口]]；[rope 接线](../../../raw/current/scripts/prefabs/rope.lua#L6-L31)、[L75-L76](../../../raw/current/scripts/prefabs/rope.lua#L75-L76) |
| 组件：tradable／bait | cutstone `rocktribute=3`（蚁狮进贡值）；rope／papyrus 挂 tradable（未设 goldvalue，猪王不收）；cutstone 挂 bait | 各自；主模拟 | [[reference/components/tradable#关键接口]]、[[reference/components/bait#关键接口]]；[rocktribute 消费（蚁狮）](../../../raw/current/scripts/prefabs/antlion.lua#L116-L141)、[猪王 AcceptTest](../../../raw/current/scripts/prefabs/pigking.lua#L604-L605) |
| 组件：hauntable | `MakeHauntableLaunchAndIgnite`（boards／rope／papyrus）；`MakeHauntableLaunch`（cutstone，不点燃） | 主模拟 | [[reference/components/hauntable#按效果选工厂]]；[cutstone](../../../raw/current/scripts/prefabs/cutstone.lua#L45) |
| API／数据：配方标志位 | `no_builder_tag`／`no_builder_skill` 对持标签／技能者隐藏主配方，`forward_ingredients` 指向替代配方；专属配方的 `product` 都落回本页注册名 | 建造端判定；标签由 woodie.lua 按技能＋携带状态维护 | 配方定义 [recipes.lua#L457-L461](../../../raw/current/scripts/recipes.lua#L457-L461)、[cancarveboards 门禁与维护](../../../raw/current/scripts/prefabs/woodie.lua#L1719-L1740) |
| 关联 Prefab：cutgrass／log／cutreeds | 精炼原料（见采集页）；`rocks` 为 cutstone 原料（矿物页） | — | [[reference/prefabs/basic-gathered-materials]]、[[reference/prefabs/basic-mineral-materials]] |
| 关联 Prefab：vault_lobby_exit | `canrope` 标签的实际挂载者（Terra 宝库出口裂隙 rope 交互），当前快照仅此一处消费 | 宝库内；服务器 | [标签消费](../../../raw/current/scripts/prefabs/vault_lobby_exit.lua#L150)、[L295-L297](../../../raw/current/scripts/prefabs/vault_lobby_exit.lua#L295-L297) |
| 资源：anim 包 | boards.zip／cutstone.zip／rope.zip／papyrus.zip，bank／build 同名 | 公共端声明 | [boards assets](../../../raw/current/scripts/prefabs/boards.lua#L1-L4)、[cutstone assets](../../../raw/current/scripts/prefabs/cutstone.lua#L1-L4) |

## 典型接入

### 借鉴"对特定目标使用"的材料接线

- 前提与选择：服务端 `modmain.lua`；rope 的 `useabletargeteditem` 模式适合"手持材料对世界目标执行动作"的需求：目标端只需带 `canrope` 标签并实现 `OnUsedRope`，材料端不改。
- 生效与撤销：玩家对目标使用 rope 走 `ACTIONS.USEITEMON.fn` → `useabletargeteditem:StartUsingItem` → `OnUseFn`（即 `OnUsedRope`），成功与否由 `target:OnUsedRope` 返回值决定（`success, reason`）；rope 在成功路径不锁定 in-use 状态（`DoStaticTaskInTime(0, ResetInUse)`），失败返回原因并保持可再用。
- 原版依据：公共端 `UseableTargetedItem_ValidTarget` 过滤目标，`GetUseItemOnVerb` 返回的 "TIE_ONTO" 是界面动词文案键（动作本身是 USEITEMON）；当前快照唯一原版目标是宝库出口裂隙（`vault_lobby_exit`）。[rope 逻辑](../../../raw/current/scripts/prefabs/rope.lua#L6-L31)、[USEITEMON 动作](../../../raw/current/scripts/actions.lua#L3351-L3363)、[目标侧](../../../raw/current/scripts/prefabs/vault_lobby_exit.lua#L295-L297)

```lua
-- modmain.lua：让 rope 也能对自定义目标使用（目标端示例）
AddPrefabPostInit("my_rope_target", function(inst)  -- 自定义实体需先自行注册
    if GLOBAL.TheWorld.ismastersim then
        inst:AddTag("canrope")
        inst.OnUsedRope = function(inst, rope, doer)
            return true  -- 返回 false, "原因" 表示拒绝
        end
    end
end)
```

- 调整与验证：`OnUsedRope` 第一个参数是目标自身、第二个是绳、第三个是玩家；返回 `true` 视为成功（rope 不消耗，消耗逻辑需目标端自行处理）。示例的 `my_rope_target` 为占位注册名，须替换为已注册实体。

## 常见陷阱与限制

- **边界：cutstone 不接燃料线**：无 `fuel`／burnable，不可燃也不可作燃料；把它当普通"可燃材料"处理会拿到 nil 组件。同时作祟工厂是 `MakeHauntableLaunch`（不点燃）。[cutstone 组件段](../../../raw/current/scripts/prefabs/cutstone.lua#L29-L45)
- **边界：rope 的 `canrope` 目标极少**：`UseableTargetedItem_ValidTarget` 要求 `canrope` 标签，快照内仅宝库出口裂隙挂载；不要假设绳子能对任意物体"捆绑"。[消费点](../../../raw/current/scripts/prefabs/vault_lobby_exit.lua#L295-L297)
- **边界：Woodie／Walter 的主配方可见性与技能门禁**：`cancarveboards` 不是"携 lucy 即有"——Woodie 须先激活 `woodie_human_lucy_1` 技能（OnSkillSelectionChange 判定后才注册 itemget/itemlose 监听维护标签，无技能时主动 RemoveTag）。未激活该技能的 Woodie（默认出生即有 Lucy）主配方照常可见，专属 `woodie_boards` 也因 `builder_skill` 不可用——正常做木板走主配方，不会无路可做。Walter 同理按 `walter_camp_rope` 技能切换。做配方修改时三条（主配方＋两专属）都要覆盖。[技能门禁](../../../raw/current/scripts/prefabs/woodie.lua#L1719-L1740)、[标签赋值](../../../raw/current/scripts/prefabs/woodie.lua#L1715-L1717)、[替代配方](../../../raw/current/scripts/recipes.lua#L173)
- **边界：`tradable` 材料猪王不收**：rope／papyrus／boards（挂 tradable 或无）均未设 `goldvalue`，猪王 `AcceptTest` 要求 `goldvalue>0`；进贡对象是蚁狮（`rocktribute`）。带值范围只在本页四注册内表述：仅 cutstone 有值（`rocktribute=3`）；`rocktribute` 并非 cutstone 专属，检索到 antliontrinket／cotl_trinket／dumbbells／egg／heatrock／rocks（inv_rocks.lua）／refined_dust／vault_pillar_guard_pieces 等也带值，此处仅作背景、不在本页展开。[AcceptTest](../../../raw/current/scripts/prefabs/pigking.lua#L604-L605)、[蚁狮 AcceptTest](../../../raw/current/scripts/prefabs/antlion.lua#L116-L119)、[蚁狮进贡结算](../../../raw/current/scripts/prefabs/antlion.lua#L141)
- **边界：`boards` 可食但无收益**：`FOODTYPE.WOOD`、治疗／饥饿均为 0，是给"吃木头"类生物（如 `edible` WOOD 判定）准备的类别入口，不是玩家食物。[boards](../../../raw/current/scripts/prefabs/boards.lua#L29-L32)

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核验四份 prefab 文件全文（注册名、公共端与主模拟组件接线、浮水／sinks／hauntable 差异）、四条主配方与 woodie_boards／walter_rope 替代配方、`cancarveboards` 标签接线（woodie.lua）、`rocktribute` 消费（antlion.lua）、`canrope` 消费范围（rope.lua＋vault_lobby_exit.lua，`rg` 全 scripts 仅此两文件）、猪王 AcceptTest、tuning 实际取值、简中名称键。未覆盖：boards/cutstone/papyrus 作为建造配方的完整消费清单、woodie_boards／walter_rope 配方内部 sg_state 行为、蚁狮进贡完整结算。
- 游戏运行：未运行。燃料／堆叠等数值为 tuning 源码推导，非运行测量。
- 结构检查：validate_page.py 与 validate_source_links.py 通过。
- 修订：2026-11-08，主代理独立检视回源码复核后修订四处：①`cancarveboards` 门禁补全——woodie.lua#L1719-L1740 确认标签只在技能 `woodie_human_lucy_1` 激活后注册 itemget/itemlose 监听维护（条件＝携带 lucy）、无技能时主动 RemoveTag，且 `woodie_boards` 配方本身要求该 builder_skill（recipes.lua#L173），获取途径与陷阱两处补齐并明确未激活技能的 Woodie 走主配方；②"仅 cutstone／rocks 族有 rocktribute"全称限定为本页仅 cutstone 有值（=3），其余带值对象（antliontrinket／cotl_trinket／dumbbells／egg／heatrock／rocks／refined_dust／vault_pillar_guard_pieces）改记为检索背景，蚁狮锚点细化（antlion.lua#L116-L119／#L141）；③删除"配方标志位"行的占位链接并改写为实际门禁内容；④nav 表蚁狮锚点同步为 antlion.lua#L116-L141。
- 外部复检修订：2026-11-08，按外部复检报告（docs/reports/w05/prefab-inventory-review-b01-20261108.md R13，主代理回源码验证成立）修订三处：①rope 使用链口径改准——"TIE_ONTO"是 `GetUseItemOnVerb` 返回的界面动词文案键（rope.lua#L29-L31）而非动作，实际动作是 `ACTIONS.USEITEMON.fn` → `useabletargeteditem:StartUsingItem`（actions.lua#L3351-L3363）→ `OnUsedRope`，重要副作用与借鉴段两处同步；②rope 燃烧时长与燃料值方向改正——`LARGE_BURNTIME`=30 秒**小于** `MED_FUEL`=45 秒（tuning.lua#L1675／L1683），保留"两条数值独立"解释；③补记 actions.lua 入 sources。

返回 [[reference/prefabs/index|Prefab 索引]]。
