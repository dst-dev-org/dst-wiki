---
title: 小袋与专用袋族（seedpouch／spicepack／potatosack／beard_sack）
created: 2026-11-07
updated: 2026-11-07
type: prefab
summary: 原版专用袋族的注册与组装入口：BODY 槽容器袋（种子袋限定种子类收纳、厨师袋全收）、健身配重土豆袋与技能授予的胡须袋，含过滤、保鲜与皮肤接线。
aliases: [seedpouch, spicepack, potatosack, beard_sack_1, beard_sack_2, beard_sack_3, 种子袋, 厨师袋, 土豆袋, 胡须袋, 香料包, 专用袋, 小袋]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/seedpouch.lua", "prefabs/spicepack.lua", "prefabs/potatosack.lua", "prefabs/beard_sack.lua", "prefabs/wortox_nabbag.lua", "containers.lua", "components/container.lua", "recipes.lua", "languages/chinese_s.po", "components/beard.lua", "prefabs/wilson.lua", "prefabs/skilltree_wilson.lua", "prefabs/warly.lua", "components/perishable.lua", "prefabs/icepack.lua", "tuning.lua", "constants.lua", "components/mightygym.lua", "prefabs/mighty_gym.lua", "prefabskins.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 小袋与专用袋族（seedpouch／spicepack／potatosack／beard_sack）

## 快速判断

- 职责与适用：本页是按"袋"命名的原版实体族入口，对象类别各不相同——种子袋 `seedpouch`、厨师袋 `spicepack` 是 BODY 槽容器袋（种子袋限定收纳、厨师袋不过滤，借鉴"穿脱开包"的组装），土豆袋 `potatosack` 是可举起的重物（健身架配重，无容器），`beard_sack_1/2/3` 是威尔逊技能授予的 BEARD 槽食物袋。通用背包机制见 [[reference/prefabs/backpack]]，本页不重复其穿戴开包细节。
- 优先使用：给专用袋改收纳条件或布局 → [[reference/components/container#让原版背包只接收具有食物类型标签的物品]] 与 [containers 配置](../../../raw/current/scripts/containers.lua#L9-L17)；调保鲜 → [preserver 组件](../../../raw/current/scripts/prefabs/seedpouch.lua#L76-L77) 或 [foodpreserver 标签消费](../../../raw/current/scripts/components/perishable.lua#L81-L88)；新增重物参与健身 → [gymweight 消费](../../../raw/current/scripts/components/mightygym.lua#L54-L68)。
- 重要副作用：厨师袋点燃后锁开包、燃尽掉物并留灰（[燃烧回调](../../../raw/current/scripts/prefabs/spicepack.lua#L47-L68)）；胡须袋随胡须等级自动换袋转移内容、剃须或死亡时掉落移除（[UpdateBeardInventory](../../../raw/current/scripts/components/beard.lua#L253-L290)、[EmptyBeard](../../../raw/current/scripts/prefabs/wilson.lua#L133-L139)）。
- 修改边界：本页不覆盖 wortox_nabbag（强抢袋，机制见 [[reference/prefabs/wortox-nabbag]]）、krampus_sack／icepack／piggyback 等其余背包；皮肤仅核验种子袋与厨师袋的登记清单。`EQUIPSLOTS.BEARD`（[槽位定义](../../../raw/current/scripts/constants.lua#L654-L659)）的通用装备规则不在本页展开。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `seedpouch`（种子袋） | `fn` → 公共外观＋backpack 标签 → SetPristine → 主模拟容器／装备／保鲜 | BODY 槽、WidgetSetup("seedpouch")、preserver 组件、不可燃、可作祟。[公共构造](../../../raw/current/scripts/prefabs/seedpouch.lua#L35-L59)、[服务端组装](../../../raw/current/scripts/prefabs/seedpouch.lua#L62-L81)、[注册](../../../raw/current/scripts/prefabs/seedpouch.lua#L84) |
| `spicepack`（厨师袋） | `fn` → 公共外观＋标签（backpack／foodpreserver／nocool）→ SetPristine → 主模拟容器／装备／燃烧 | BODY 槽、WidgetSetup("spicepack")、无 preserver 组件、小地图图标、可燃烧。[公共构造](../../../raw/current/scripts/prefabs/spicepack.lua#L70-L99)、[服务端组装](../../../raw/current/scripts/prefabs/spicepack.lua#L101-L123)、[注册](../../../raw/current/scripts/prefabs/spicepack.lua#L126) |
| `potatosack`（土豆袋） | `fn` → 重物公共体（heavy 标签、gymweight=1）→ SetPristine → 主模拟装备／工作／燃烧 | BODY 槽但非容器；heavyobstaclephysics＋walkspeedmult=HEAVY_SPEED_MULT；可锤、可沉水。[公共构造](../../../raw/current/scripts/prefabs/potatosack.lua#L36-L59)、[服务端组装](../../../raw/current/scripts/prefabs/potatosack.lua#L61-L95)、[注册](../../../raw/current/scripts/prefabs/potatosack.lua#L98) |
| `beard_sack_1/2/3`（胡须袋，非官方译名） | `fn1/fn2/fn3` → 共用 `fn_common`（外观）→ SetPristine → 共用 `fn_mastersim`（BEARD 槽＋容器） | 三个注册名同文件返回；仅槽位 UI 与标签不同（1/2/3 格）。无 inspectable，无 .po 名称键。[三工厂](../../../raw/current/scripts/prefabs/beard_sack.lua#L58-L105)、[主模拟组装](../../../raw/current/scripts/prefabs/beard_sack.lua#L39-L56) |

名称核验（简中 `STRINGS.NAMES.*`，[chinese_s.po](../../../raw/current/scripts/languages/chinese_s.po#L349080-L349082)）：SEEDPOUCH="种子袋"、SPICEPACK="厨师袋"（L349775-L349777）、POTATOSACK="土豆袋"（L347230-L347232）；BEARD_SACK_* 在 .po 无条目，"胡须袋"为已核验效果描述而非官方译名。配方：`seedpouch`＝外壳碎片 2＋采下的草 4＋种子 2，TECH.SCIENCE_TWO（[recipes.lua:516](../../../raw/current/scripts/recipes.lua#L516)）；`spicepack`＝采下的草 4＋树枝 4＋硝石 2，TECH.NONE，builder_tag="masterchef"（仅沃利，[recipes.lua:100](../../../raw/current/scripts/recipes.lua#L100)、[warly 标签](../../../raw/current/scripts/prefabs/warly.lua#L18)）；`potatosack` 无合成配方，仅登记 `DeconstructRecipe` 产出草 2＋石头 3（拆解法杖用）；锤毁另走 `lootdropper:DropLoot()` 的默认 `droprecipeloot=true`，从 `AllRecipes["potatosack"]`（即那条 DeconstructRecipe）按 `HAMMER_LOOT_PERCENT=0.5` 向上取整取料＝草 1＋石头 2（[DeconstructRecipe](../../../raw/current/scripts/recipes.lua#L1650)），实际由健身架建造时生成（[onbuilt](../../../raw/current/scripts/prefabs/mighty_gym.lua#L36-L41)）；`beard_sack_1/2/3` 无配方，由 beard 组件按技能生成。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：equippable | seedpouch／spicepack／potatosack 为 BODY 槽并绑定穿脱外观；beard_sack 为 BEARD 槽且 SetPreventUnequipping(true) 禁止手动卸下 | 主模拟；seedpouch/spicepack 穿上 Open 容器、脱下 Close（与 backpack 同型） | [[reference/components/equippable]]；[seedpouch 绑定](../../../raw/current/scripts/prefabs/seedpouch.lua#L67-L71)、[beard_sack 槽位](../../../raw/current/scripts/prefabs/beard_sack.lua#L43-L47)、[穿脱回调](../../../raw/current/scripts/prefabs/seedpouch.lua#L12-L33) |
| 组件：container | WidgetSetup 各自布局；seedpouch/spicepack 为 type="pack" 侧边栏（openlimit=1），beard_sack 为 type="side_inv_behind"（acceptsstacks、lowpriorityselection） | 布局两端（组件＋Replica）都从 containers.params 读取 | [[reference/components/container#新容器的客户端布局]]；[seedpouch 布局](../../../raw/current/scripts/containers.lua#L1712-L1735)、[spicepack 布局](../../../raw/current/scripts/containers.lua#L49-L66)、[beard_sack 布局](../../../raw/current/scripts/containers.lua#L1353-L1454) |
| 组件：preserver | 仅 seedpouch 挂载，倍率 TUNING.SEEDPOUCH_PRESERVER_RATE=0.5 | 主模拟；perishable 优先走 owner 的 preserver 分支 | [[reference/components/preserver]]；[挂载](../../../raw/current/scripts/prefabs/seedpouch.lua#L76-L77)、[倍率](../../../raw/current/scripts/tuning.lua#L5158)、[分支顺序](../../../raw/current/scripts/components/perishable.lua#L81-L88) |
| 数据：foodpreserver／nocool 标签 | 仅 spicepack：foodpreserver 使袋内食物腐败倍率取 PERISH_FOOD_PRESERVER_MULT=0.75；nocool 使热食入袋不降温 | 消费在 perishable 更新时按 owner 标签判定；icepack 另有 fridge+nocool 组合 | [[reference/components/perishable]]；[标签挂载](../../../raw/current/scripts/prefabs/spicepack.lua#L87-L89)、[倍率值](../../../raw/current/scripts/tuning.lua#L2078-L2083)、[nocool 保热](../../../raw/current/scripts/components/perishable.lua#L139-L147)、[icepack 对照](../../../raw/current/scripts/prefabs/icepack.lua#L56-L58) |
| 数据：itemtestfn／priorityfn | seedpouch 只收 `seeds`／`*_seeds`／treeseed 标签，并把 itemtestfn 复用为 priorityfn（拾取时优先入袋）；beard_sack_1/2/3 只收 FOODGROUP.OMNI 六类 edible_ 标签食物；spicepack 不过滤 | itemtestfn 由 WidgetSetup 复制并转只读；priorityfn 消费在 Container:ShouldPrioritizeContainer | [[reference/components/container#关键接口]]；[seedpouch 过滤](../../../raw/current/scripts/containers.lua#L1731-L1735)、[OMNI 定义](../../../raw/current/scripts/constants.lua#L2051-L2062)、[priorityfn 消费](../../../raw/current/scripts/components/container.lua#L308-L320)、[WidgetSetup 复制](../../../raw/current/scripts/components/container.lua#L80-L97) |
| 组件：beard（消费端） | beard 组件在技能 wilson_beard_7 激活且胡须位数达 LEVEL1/2/3（1/3/9）时生成对应 beard_sack_N 并 Equip 到 BEARD 槽；换档时旧袋 `RemoveAllItems()`（按槽序收集、空槽跳过，得到压紧表）→ `Close` → `Remove`，新袋 `Equip`，再 `ipairs` 以**压紧后的序号**当目标槽 `GiveItem(item, slot, nil, true)`；缩容（3→2）时超出新袋容量的物品走 `drop_on_fail` 直接掉地，旧槽号并不保证保留 | 调用点：胡须增长（SetBits）、Shave、Reset、LoadPostPass、技能激活；无技能或位数不足时掉落并移除 | [[reference/components/beard]]；[UpdateBeardInventory](../../../raw/current/scripts/components/beard.lua#L253-L290)、[位数阈值](../../../raw/current/scripts/tuning.lua#L7613-L7617)、[技能定义](../../../raw/current/scripts/prefabs/skilltree_wilson.lua#L311-L321) |
| 组件：burnable／propagator | spicepack 小型燃烧＋三回调（点燃锁包／熄灭解锁／燃尽掉物生成 ash）；potatosack 中型燃烧；seedpouch 无燃烧 | 主模拟；beard_sack 仅可作祟（MakeHauntableLaunchAndDropFirstItem） | [[reference/components/burnable]]；[spicepack 回调](../../../raw/current/scripts/prefabs/spicepack.lua#L115-L119)、[potatosack](../../../raw/current/scripts/prefabs/potatosack.lua#L92-L93)、[作祟](../../../raw/current/scripts/prefabs/seedpouch.lua#L79) |
| 组件：workable／lootdropper／heavyobstaclephysics／submersible／symbolswapdata | potatosack 专属：锤满 2 点工作 → `onhammered` 先灭火、`DropLoot()`（默认 `droprecipeloot` → 草 1＋石头 2）、生成 `collapse_small`（材质 rock）并 `Remove`；沉重物理（半径 0.45）与减速举持；沉水；换装符号记录 | 主模拟；gymweight=1 供 mightygym CalcWeight 求和（缺省 2） | [[reference/components/workable]]；[锤毁](../../../raw/current/scripts/prefabs/potatosack.lua#L18-L29)、[重物字段](../../../raw/current/scripts/prefabs/potatosack.lua#L47-L48)、[CalcWeight](../../../raw/current/scripts/components/mightygym.lua#L54-L68)、[符号记录](../../../raw/current/scripts/prefabs/potatosack.lua#L71-L72) |
| 关联 Prefab：ash／collapse_small／mighty_gym | spicepack 燃尽生成 ash；potatosack 被锤生成 collapse_small（材质 rock）；mighty_gym 建造时生成 2 只 potatosack 作默认配重 | 生成点已核验；prefabs 声明不等于全部生成路径 | [ash](../../../raw/current/scripts/prefabs/spicepack.lua#L47-L56)、[collapse_small](../../../raw/current/scripts/prefabs/potatosack.lua#L25-L28)、[gym 生成](../../../raw/current/scripts/prefabs/mighty_gym.lua#L36-L41) |
| 资源：动画与 UI | seedpouch 用 bank/build `seedpouch`＋`ui_krampusbag_2x8`；spicepack 复用 bank `backpack1`、build `swap_chefpack`＋`ui_icepack_2x3`；beard_sack 复用 build `swap_krampus_sack`＋`ui_beard_1x1/2x1/3x1`；potatosack 用 `potato_sack` | 声明于各文件 assets；未验证压缩包内容 | [seedpouch assets](../../../raw/current/scripts/prefabs/seedpouch.lua#L1-L5)、[spicepack assets](../../../raw/current/scripts/prefabs/spicepack.lua#L1-L6)、[beard_sack assets](../../../raw/current/scripts/prefabs/beard_sack.lua#L1-L5) |

皮肤：`seedpouch_garden`／`seedpouch_rustic`（[登记](../../../raw/current/scripts/prefabskins.lua#L1688-L1691)）、`spicepack_porcelain/rustic/silver/survivor`（[登记](../../../raw/current/scripts/prefabskins.lua#L1794-L1799)）；potatosack 与 beard_sack 在 prefabskins 无皮肤。种子袋穿脱用 OverrideItemSkinSymbol 走 `seedpouch` 后备符号（[onequip](../../../raw/current/scripts/prefabs/seedpouch.lua#L12-L23)）；皮肤工厂与清理机制的完整链路见 [[reference/prefabs/backpack#皮肤差异与跟随实体]]，本页不复制。

## 典型接入

### 调整专用袋的保鲜

- 前提与选择：目标模组 `modmain.lua`，服务端 postinit。种子袋有 preserver 组件可直改倍率；厨师袋没有该组件，靠容器标签让 perishable 取 0.75 倍率——改它的保鲜走标签分支或补挂 preserver（组件分支优先于标签分支，[判定顺序](../../../raw/current/scripts/components/perishable.lua#L81-L88)）。
- 生效与撤销：改倍率影响之后入袋物品的腐败计算；卸载模组后新实例恢复 TUNING 默认。
- 原版依据：[seedpouch 挂载 preserver](../../../raw/current/scripts/prefabs/seedpouch.lua#L76-L77)、[倍率常量](../../../raw/current/scripts/tuning.lua#L5158)、[foodpreserver 分支](../../../raw/current/scripts/components/perishable.lua#L87-L88)。

```lua
-- modmain.lua
AddPrefabPostInit("seedpouch", function(inst)
    if GLOBAL.TheWorld.ismastersim and inst.components.preserver ~= nil then
        inst.components.preserver:SetPerishRateMultiplier(0.25) -- 原 0.5，越小越保鲜
    end
end)
```

- 调整与验证：倍率是无单位乘数（1＝原速）；preserver 判定按 owner（即袋子）执行，袋中袋不穿透判断，见 [[reference/components/preserver#常见陷阱与限制]]。

### 给专用袋登记收纳过滤或新布局

- 前提与选择：改既有袋的接收条件或新增袋布局，都在 `containers.params`（[模块导出](../../../raw/current/scripts/containers.lua#L7)）上以新键注册，两端（服务端 WidgetSetup 与客户端 Replica）都须能读到同键配置；完整可运行示例见 [[reference/components/container#让原版背包只接收具有食物类型标签的物品]]，此处只记本族差异。
- 生效与撤销：WidgetSetup 复制参数后转只读（[复制与只读](../../../raw/current/scripts/components/container.lua#L80-L97)），运行期改组件字段不可行；停用模组后新实例恢复原配置。
- 原版依据：种子袋把 itemtestfn 复用为 priorityfn 实现"拾取种子优先入袋"（[定义](../../../raw/current/scripts/containers.lua#L1731-L1735)，[消费](../../../raw/current/scripts/components/container.lua#L308-L320)）；胡须袋用 type="side_inv_behind" 把 UI 挂到人物侧后方；`type="pack"` 与 openlimit=1 决定侧边栏与同时打开数（[spicepack 布局](../../../raw/current/scripts/containers.lua#L49-L66)）。

## 常见陷阱与限制

- **边界：名字带 bag/sack 的不一定是容器袋**：`potatosack` 无 container 组件，是健身配重重物（[组件清单](../../../raw/current/scripts/prefabs/potatosack.lua#L61-L95)）；`wortox_nabbag`（强抢袋）也无 container，是技能授予的手持工具/武器（nabbag 组件＋weapon＋tool，[组装](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L203-L229)，官方名"强抢袋"见 [chinese_s.po](../../../raw/current/scripts/languages/chinese_s.po#L352495-L352497)），其锥形拾取机制复杂度足以独立成页，本页不展开。按文件名猜"袋＝容器"会同时误判这两例。
- **误区：厨师袋挂了保鲜组件**：spicepack 只有 foodpreserver/nocool 标签，没有 preserver 组件；给它 `SetPerishRateMultiplier` 前须先补组件。若同时存在 preserver 组件与标签，perishable 只走 preserver 分支。[分支](../../../raw/current/scripts/components/perishable.lua#L81-L88)
- **边界：三只袋的燃烧差异**：spicepack 可燃且点燃期间锁开包；potatosack 可燃但无容器可锁；seedpouch 完全没有 burnable/propagator，不会着火。复制 backpack 的燃烧接线时注意各袋差异。[spicepack](../../../raw/current/scripts/prefabs/spicepack.lua#L115-L119)、[seedpouch 无燃烧](../../../raw/current/scripts/prefabs/seedpouch.lua#L62-L81)
- **边界：胡须袋由 beard 组件全权管理**：SetPreventUnequipping(true) 禁止手动脱下；技能关闭或胡须位数不足时掉落内容并移除袋子；威尔逊死亡时 EmptyBeard 掉落移除（[监听 death](../../../raw/current/scripts/prefabs/wilson.lua#L160-L161)）。模组不应绕过 beard 组件直接增删该槽物品。[管理逻辑](../../../raw/current/scripts/components/beard.lua#L253-L290)
- **边界：胡须袋 UI 不在背包栏**：type="side_inv_behind" 渲染在人物侧后方，且 lowpriorityselection 会影响悬停选取优先级；照抄 pack 型布局得不到相同表现。[布局参数](../../../raw/current/scripts/containers.lua#L1370-L1373)

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394。完整读取 seedpouch.lua、spicepack.lua、potatosack.lua、beard_sack.lua、wortox_nabbag.lua；核验四个注册名（＋beard_sack 三变体）与工厂分界、containers.lua 对应布局与 itemtestfn/priorityfn、recipes.lua 配置与门禁、chinese_s.po 名称键（SEEDPOUCH/SPICEPACK/POTATOSACK/WORTOX_NABBAG，BEARD_SACK_* 无条目）、perishable 标签与 preserver 分支、beard 组件生成/转移/移除链与调用点、mighty_gym/mightygym 配重消费、TUNING（SEEDPOUCH_PRESERVER_RATE、WILSON_BEARD_BITS、PERISH_FOOD_PRESERVER_MULT）、EQUIPSLOTS 与 FOODGROUP.OMNI 常量、皮肤登记清单。
- 未覆盖：wortox_nabbag 的 nabbag 组件细节与 NET 复制路径仅作"非容器袋"反证，未成页；皮肤工厂实现未逐一审校；资源压缩包内容、多人开关细节、存档恢复未验证。
- 游戏运行：未运行。
- 结构检查：`validate_page.py`（含 `--check-indexes`）通过；standard 档上限 240 行 / 32768 字节；`validate_source_links.py --strict` 0 error 0 warning；`validate_source_hashes.py` 已记录。

相关：[[reference/prefabs/backpack]]；[[reference/prefabs/icepack]]；[[reference/prefabs/piggyback]]；[[reference/prefabs/krampus_sack]]；[[reference/components/container]]；[[reference/components/preserver]]；[[reference/components/beard]]；[[reference/components/perishable]]。

返回 [[reference/prefabs/storage-heavy-index|背包收纳与重物搬运]]、[[reference/prefabs/index|Prefab 索引]]。
