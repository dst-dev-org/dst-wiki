---
title: 强抢袋（wortox_nabbag）
created: 2026-11-07
updated: 2026-11-07
type: prefab
summary: 沃拓克斯技能授予的手持装备原版参照：右键强抢锥形范围物品、左键当捕网并复制捕捉，伤害随背包满度三档并受灵魂加成，可借鉴技能门禁＋受限装备＋NAB/NET 双动作组装。
aliases: ["wortox_nabbag", "强抢袋", "Knabsack", "强抢贼"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/wortox_nabbag.lua", "components/nabbag.lua", "prefabs/skilltree_wortox.lua", "prefabs/wortox.lua", "prefabs/wortox_souljar.lua", "recipes.lua", "tuning.lua", "constants.lua", "components/equippable.lua", "components/equippable_replica.lua", "prefabs/inventoryitem_classified.lua", "components/inventory.lua", "components/weapon.lua", "components/finiteuses.lua", "components/tool.lua", "entityscript.lua", "components/builder.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "prefabs/butterfly.lua", "screens/redux/scrapbookdata.lua", "prefabskins.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 强抢袋（wortox_nabbag）

## 快速判断

- 职责与适用：`wortox_nabbag`（官方名"强抢袋"，英文 Knabsack）是沃拓克斯技能树 `wortox_nabbag` 节点授予的 HANDS 槽手持装备，不是容器袋：nabbag（锥形连抓）＋weapon（伤害随背包满度分档）＋tool(NET)（可当捕网）＋finiteuses(200) 的组合体。锥形拾取组件机制见 [[reference/components/nabbag]]，装备期三层表现实体见 [[reference/prefabs/wortox_soulecho_buff_fx]]。
- 优先使用：给角色技能做专属手持装备 → 抄本页"技能配方门禁＋equippable.restrictedtag 装备门禁"双层组装（[配方](../../../raw/current/scripts/recipes.lua#L357)、[门禁](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L223-L226)）；给物品做范围拾取或捕网复制 → 组件页接入；调数值 → `TUNING.SKILLS.WORTOX` 表（[tuning.lua L7432-L7448](../../../raw/current/scripts/tuning.lua#L7432-L7448)）。
- 重要副作用：右键 NABBAG 连抓锥内物品（主目标免费、锥内每处理一个候选目标扣 1 耐久，无论拾取是否成功；扣满单次上限后仍会继续尝试拾取，只有耐久耗尽、袋子销毁才提前中断）；左键 NET 捕捉 NET 类生物并复制到锥内同类目标（每次 10 耐久）；伤害 13.6→34 随主物品栏满度三档，`wortox_souljar_3` 技能下最高 ×2；耐久耗尽自毁（[OnUsesFinished](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L168-L174)）。
- 修改边界：本页覆盖物品注册、数值、双动作供给与技能门禁；nabbag 组件的锥形复制细节与上限偏移归 [[reference/components/nabbag]]，body 三注册的表现归 FX 页；解构流程、数据驱动生成点、多人逐帧预测未核验。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wortox_nabbag`（强抢袋） | `fn` → 公共体（动画/物理/`nabbag`＋`weapon` 标签/漂浮）→ SetPristine → 主模拟组装（nabbag/weapon/tool/finiteuses/inspectable/inventoryitem/equippable/fuel） | HANDS 槽手持装备；图标随满度三档换图（INV_IMAGE medium/full）；耐尽自毁。[公共构造](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L176-L201)、[主模拟组装](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L203-L239)、[注册](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L509) |
| `wortox_nabbag_body`／`_body_fx`／`_body_soulfx` | 同文件三注册的装备期表现族 | 联网载体＋本地机体动画＋本地灵魂粒子；注册归属与表现机制见 [[reference/prefabs/wortox_soulecho_buff_fx]]；本页只记[注册清单](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L509-L512)与 netvar 写入方（[UpdateStats](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L53-L62)） |

名称核验（简中 `.po`，[chinese_s.po](../../../raw/current/scripts/languages/chinese_s.po#L352494-L352497)）：`STRINGS.NAMES.WORTOX_NABBAG`="Knabsack"/"强抢袋"；动作 `STRINGS.ACTIONS.NABBAG`="Knab"/"强抢"（[L1359-L1362](../../../raw/current/scripts/languages/chinese_s.po#L1359-L1362)）、NET="Catch"/"捕捉"（[L1364-L1367](../../../raw/current/scripts/languages/chinese_s.po#L1364-L1367)）；检查语、配方与技能文案分别见 [L23569-L23572](../../../raw/current/scripts/languages/chinese_s.po#L23569-L23572)、[L359954-L359957](../../../raw/current/scripts/languages/chinese_s.po#L359954-L359957)、[L369149-L369157](../../../raw/current/scripts/languages/chinese_s.po#L369149-L369157)（技能标题"强抢贼"）。

配方：`wortox_soul`×4＋`silk`×2，TECH.NONE，`builder_skill="wortox_nabbag"`（[recipes.lua L357](../../../raw/current/scripts/recipes.lua#L357)）；builder 门禁在 KnowsRecipe/可合成判定读 `skilltreeupdater:IsActivated`（[builder.lua L876-L877](../../../raw/current/scripts/components/builder.lua#L876-L877)）。scripts 全文检索 `SpawnPrefab("wortox_nabbag")` 无命中，配方是唯一已核验获取途径。皮肤：`prefabskins.lua` 全文检索无 `wortox_nabbag` 登记，`.po` 亦无 Knabsack 皮肤名——无官方皮肤。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：nabbag | 锥形连抓主体；NABBAG 动作走 `DoNabFromAct`，NET 成功后走 `ReplicateNetFromAct` | 挂载于主模拟（L203）；锥形几何为 150° 前向扇区＋半径 3＋近距 1.25 圆豁免（[IsEntityInFrontConeSlice](../../../raw/current/scripts/entityscript.lua#L1471-L1488)） | [[reference/components/nabbag]]；[挂载](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L203) |
| 组件：weapon | `SetDamage(NABBAG_DAMAGE_MIN)` 起始、构造时 `attackwearmultipliers` 置 0；`UpdateStats` 再按满度桶位改写伤害与磨损 0／0.5／1 | 主模拟；磨损消费在命中时按修饰值扣耐久（[weapon.lua L105-L120](../../../raw/current/scripts/components/weapon.lua#L105-L120)） | [[reference/components/weapon]]；[初始化](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L205-L207)、[重写](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L49-L50) |
| 组件：tool | `SetAction(ACTIONS.NET)` 登记为 NET 工具（加 `NET_tool` 标签，[tool.lua L41-L45](../../../raw/current/scripts/components/tool.lua#L41-L45)） | NET 工作目标免 grabbable 检查（[actions.lua L1866](../../../raw/current/scripts/actions.lua#L1866)）；可工作对象如 bee/butterfly/fireflies/mosquito 等设置 NET 工作动作（[butterfly.lua L165-L168](../../../raw/current/scripts/prefabs/butterfly.lua#L165-L168)） | [[reference/components/tool]]；[登记](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L209-L210) |
| 组件：finiteuses | 总 200 起始满；NET 消耗 `200/20`=10/次；破损推 `toolbroke` 并 Remove | 主模拟；消耗经标准动作流的 `OnUsedAsItem`（[finiteuses.lua L101-L114](../../../raw/current/scripts/components/finiteuses.lua#L101-L114)） | [[reference/components/finiteuses]]；[设置](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L212-L217)、[破损](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L168-L174) |
| 组件：equippable | 默认 HANDS 槽（[equippable.lua L29](../../../raw/current/scripts/components/equippable.lua#L22-L51)）；`restrictedtag="nabbaguser"` 限制仅带该标签者装备 | 装备侧门禁：`Inventory:Equip` 拒绝受限装备（[inventory.lua L1266-L1274](../../../raw/current/scripts/components/inventory.lua#L1266-L1274)）、EQUIP 动作不供给（[componentactions.lua L1846](../../../raw/current/scripts/componentactions.lua#L1846)）、读档豁免走 `IsRestricted_FromLoad`（[equippable.lua L164-L174](../../../raw/current/scripts/components/equippable.lua#L164-L174)） | [[reference/components/equippable]]；[装备回调](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L223-L226) |
| 组件：fuel／inspectable／inventoryitem／作祟 | fuel 值 `TUNING.SMALL_FUEL`（=`seg_time`×0.5=15 秒，源码推导，[tuning.lua L1674](../../../raw/current/scripts/tuning.lua#L1674)）；图标名按档位 `ChangeImageName`（nil/medium/full，[L63-L67](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L63-L67)） | 主模拟挂载；作祟 `MakeHauntableLaunch`（[L231](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L231)） | [图标资产](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L1-L6) |
| Replica／同步：equiprestrictedtag | restrictedtag 经 `net_hash` 净变量同步到客户端，客户端 replica 供动作判定读同一值 | 双端；服务端改字段即同步（[equippable.lua L15-L18](../../../raw/current/scripts/components/equippable.lua#L15-L18) → [inventoryitem_classified.lua L210](../../../raw/current/scripts/prefabs/inventoryitem_classified.lua#L210)、[equippable_replica.lua L32-L46](../../../raw/current/scripts/components/equippable_replica.lua#L32-L46)） | [[reference/components/inventoryitem]] |
| Replica／同步：bodysize_netvar／bodyvfx_souls | UpdateStats 把满度桶位（0–2）与灵魂数档位（0–3）写给 body 载体净变量 | 双端；netvar 定义在 bodyfn（[L359-L360](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L352-L384)），表现归 FX 页 | [[reference/prefabs/wortox_soulecho_buff_fx]] |
| SG：SGwilson "nabbag" 状态 | NABBAG 动作执行：14/15 帧挥击音、16 帧 PerformBufferedAction、38 帧回 idle；状态含 `nodangle` 标签 | 服务端状态（[SGwilson.lua L25102-L25133](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25102-L25133)、[动作绑定 L1201](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1201)）；客户端预测同名状态（[SGwilson_client.lua L7427-L7455](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L7427-L7455)）；`nodangle`／骑乘切换持握外观（[ToggleOverrideSymbols](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L99-L111)） | Wiki 待建（玩家 SG 机制页未落地） |
| API／数据：ACTIONS.NABBAG／NET | NABBAG：rmb、distance 1.8（[actions.lua L681](../../../raw/current/scripts/actions.lua#L681)），fn 调组件（[L4557-L4567](../../../raw/current/scripts/actions.lua#L4557-L4567)）；动作供给条件：右键目标可拾取且非 _container/heavy/fire（[componentactions.lua L2406-L2410](../../../raw/current/scripts/componentactions.lua#L2406-L2410)）；NET：fn 内对 nabbag 物品触发复制（[actions.lua L1872-L1875](../../../raw/current/scripts/actions.lua#L1872-L1875)）；工具工作同样受 IsRestricted 拦截（[DoToolWork](../../../raw/current/scripts/actions.lua#L1693-L1698)） | 双端（供给在 replica、执行在服务端） | [[reference/components/topics/item-actions]] |
| API／数据：TUNING.SKILLS.WORTOX | 全部数值来源：伤害 13.6/34（wilson_attack=34，[tuning.lua L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）、耐久 200、NET 折算 20、单次抓取耐久上限 0.25、单次物件上限 400、锥角 150、半径 3、近距圆 1.25（[L7432-L7440](../../../raw/current/scripts/tuning.lua#L7432-L7440)）；灵魂加成 ×2、上限 100 灵魂（[L7446-L7448](../../../raw/current/scripts/tuning.lua#L7446-L7448)） | 伤害在每次 UpdateStats 现读 TUNING；耐久 200 与 NET 10 点/次在构造时写入组件，改表只影响新实例 | 见下文"伤害与耐久" |
| API／数据：技能树 | `wortox_nabbag` 节点（顽皮根节点）激活加 `nabbaguser`、遗忘移除（[skilltree_wortox.lua L639-L656](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L639-L656)）；`wortox_souljar_3` 激活/遗忘经 `UpdateNabBags` 重算已装备袋（[L87-L92](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L87-L92)、[L695-L703](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L695-L703)）；读档豁免映射 `nabbaguser→wortox`（[constants.lua L2159](../../../raw/current/scripts/constants.lua#L2153-L2160)） | 技能节点激活即挂标签；配方与装备两层门禁共用该节点 | Wiki 待建（技能树机制页未落地） |
| 关联 Prefab：wortox_soul／wortox_souljar／silk | wortox_soul×4＋silk×2 为配方原料；装备期扫描主物品栏内 wortox_soul 堆叠与 wortox_souljar.soulcount 计入伤害加成（另补算手持 active item）（[L76-L93](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L70-L97)） | 灵魂计数含手持 active item；`ms_souljar_count_changed` 事件触发重算（[L137](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L133-L137)、[wortox_souljar.lua L84](../../../raw/current/scripts/prefabs/wortox_souljar.lua#L84)） | 两 Prefab 页未建，源码即入口 |
| 资源：动画 | bank/build `wortox_nabbag`、待机 `idle_empty`；swap 数据 `swap_wortox_nabbag_empty`；挥击动画包 `wortox_actions_nabbag.zip` 随角色载入 | 图标 INV_IMAGE `wortox_nabbag_medium/full`；未验证压缩包内容 | [assets](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L1-L6)、[wortox.lua L14](../../../raw/current/scripts/prefabs/wortox.lua#L14) |

## 典型接入

### 伤害与耐久分档（数值语义）

- 前提与选择：伤害与磨损在 `UpdateStats`（[L22-L68](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L22-L68)）现读 TUNING；耐久总量与 NET 单次消耗在构造时写入组件。
- 生效与撤销：满度按“主物品栏占用格数/总格数”计（`ForEachItemSlot` 只数 itemslots，装备与额外容器不计；计件不计堆叠，[L78-L85](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L78-L85)），桶位 = `clamp(ceil(percent×3),1,3)` 映射 `_empty/_medium/_full`（[L24-L28](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L24-L28)），桶内不渐变——档内伤害取 `(MAX−MIN)×(桶−1)/2＋MIN`。
- 原版依据：三档伤害（wilson_attack=34×修饰 1）＝13.6／23.8／34（源码推导值）；攻击磨损修饰同桶＝0／0.5／1（空袋打人不磨损）；装备期监听 itemget/itemlose/stacksizechange/newactiveitem/ms_souljar_count_changed 触发重算（[L133-L137](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L133-L137)），读档后 `DoTaskInTime(0)` 补算（[L239](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L239)）。

| 档位 | 背包满度 | 伤害（无 souljar_3） | 攻击磨损/击 | 灵魂加成后上限（wortox_souljar_3，100 灵魂） |
|---|---|---|---|---|
| `_empty` | ≤1/3 格 | 13.6 | 0 | 27.2 |
| `_medium` | 1/3–2/3 格 | 23.8 | 0.5 | 47.6 |
| `_full` | >2/3 格 | 34 | 1 | 68 |

灵魂加成：`wortox_souljar_3` 激活时 `damage×(1＋(2−1)×灵魂数/100)`（[L36-L48](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L36-L48)），灵魂数=背包 wortox_soul 堆叠数＋wortox_souljar.soulcount，封顶 100；灵魂 ≥25/≥50/=100 时 body 粒子档位 1/2/3。

### 解除装备门禁并调整伤害（modmain）

- 前提与选择：目标模组 `modmain.lua`。门禁有两层——合成 `builder_skill` 在配方表、装备 `restrictedtag` 在组件字段，只改其一会出现"做得出戴不上"或反向缺口；伤害改 TUNING 即可，UpdateStats 每次重算都实时读表（[L33-L35](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L32-L35)）。
- 生效与撤销：postinit 对新实例生效，已有存档中的袋子下次重算（拾放物品/换手/读档）后取新值；restrictedtag 置 nil 经净变量同步到客户端（见导航表 Replica 行），卸载模组后新实例恢复原值。
- 原版依据：组件字段监视器（[equippable.lua L15-L18](../../../raw/current/scripts/components/equippable.lua#L15-L18)）、装备门禁消费（[inventory.lua L1266-L1274](../../../raw/current/scripts/components/inventory.lua#L1266-L1274)）。

```lua
-- modmain.lua：mod 环境由 CreateEnvironment 提供，无 __index=_G——
-- TheWorld 等全局须 GLOBAL. 前缀，TUNING 已在环境中可直用
local function OnNabbagPostInit(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return -- 客户端只构造到 pristine，无组件
    end
    if inst.components.equippable ~= nil then
        inst.components.equippable.restrictedtag = nil -- 解除"仅沃拓克斯可装备"
    end
end
AddPrefabPostInit("wortox_nabbag", OnNabbagPostInit)

TUNING.SKILLS.WORTOX.NABBAG_DAMAGE_MIN = 13.6
TUNING.SKILLS.WORTOX.NABBAG_DAMAGE_MAX = 51 -- 三档变 13.6/32.3/51
```

- 调整与验证：三档伤害恒为 `MIN＋(MAX−MIN)×{0,0.5,1}`；配方门禁如需一并放开，把配方记录的 `builder_skill` 置 nil 或改 `no_builder_skill`，本例只解除装备侧。合成门禁读点见 [builder.lua L876-L877](../../../raw/current/scripts/components/builder.lua#L876-L877)。

## 常见陷阱与限制

- **边界：名字带 bag 不是容器袋**：wortox_nabbag 无 container 组件，是手持装备；"袋＝容器"的反例收录在 [[reference/prefabs/pouch-bags#常见陷阱与限制]]（土豆袋同例），本页与其互链。
- **误区：NABBAG 是对生物的动作**：NABBAG 只对"可拾取物品"供给（[componentactions.lua L2406-L2410](../../../raw/current/scripts/componentactions.lua#L2406-L2410)）；对蜂/蝶/萤火虫等要按 NET 工具左键捕捉，两条路径耐久口径不同（NET 每次 10 点、连抓按尝试目标 1 点/次、主目标免费），照搬捕虫网的 1 点/次会算错耐久。
- **边界：双层门禁**：合成读配方 `builder_skill`（[builder.lua L876-L877](../../../raw/current/scripts/components/builder.lua#L876-L877)）、装备读 `equippable.restrictedtag`（[inventory.lua L1266-L1274](../../../raw/current/scripts/components/inventory.lua#L1266-L1274)），两者独立消费同一技能节点；只改技能标签不动字段（或反之）会留下不对称缺口。
- **边界：伤害是三档不是线性**：桶内不随满度渐变，且满度按占用格数计而非物品总数——一背包堆叠物与一背包散件满度相同（[L78-L85](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L78-L85)）。
- **边界：单次连抓按“尝试次数”扣耐久**：主目标不耗；锥内每处理一个候选目标就扣 1 点（`PICKUP.fn` 的返回值不参与判断），单次最多耗 `total×0.25`（满耐久时 50 点，与当前剩余耐久无关），物件上限 400−1；扣到上限后仍会继续尝试拾取，直到物件上限或耐久耗尽中断（[nabbag.lua L102-L107](../../../raw/current/scripts/components/nabbag.lua#L102-L107)）；上限偏移与两套黑名单差异见 [[reference/components/nabbag#常见陷阱与限制]]。
- **边界：NET 复制真实执行且用尽即毁**：复制路径对锥内目标逐个执行 `ACTIONS.NET.fn`＋`OnUsedAsItem`（10 点/次），中途耐久耗尽会触发 OnUsesFinished 自毁并中断复制（[nabbag.lua L28-L34](../../../raw/current/scripts/components/nabbag.lua#L28-L34)、[L168-L174](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L168-L174)）。
- **边界：遗忘技能不没收装备**：`wortox_nabbag` 遗忘只移除 `nabbaguser` 标签（[L653-L655](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L650-L655)），已装备的袋子保留在手上直到主动卸下，再装备才被 `IsRestricted` 拦截；读档时按 `SKILLTREE_EQUIPPABLE_RESTRICTED_TAGS` 映射豁免（[constants.lua L2159](../../../raw/current/scripts/constants.lua#L2153-L2160)）。
- **边界：图鉴展示值 ≠ 实际耐久**：scrapbookdata 记 `finiteuses=20`（[scrapbookdata.lua L1870](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1870)）是"20 次 NET 工作"（200 点÷10 点/次）的展示口径，实际耐久总量 200（[tuning.lua L7434](../../../raw/current/scripts/tuning.lua#L7432-L7440)）。

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394（[BUILD.txt](../../../raw/current/BUILD.txt)）。完整读取 prefabs/wortox_nabbag.lua（512 行）与 components/nabbag.lua（135 行）；核验 4 个注册名与工厂分界、UpdateStats 桶位/伤害/灵魂加成与事件链、restrictedtag 装备门禁全链（equippable→inventory→replica→classified）、builder_skill 配方门禁、skilltree_wortox 两节点与 UpdateNabBags、ACTIONS.NABBAG/NET 定义与动作供给、SGwilson 服务端/客户端 nabbag 状态与 nodangle 外观切换、TUNING 全数值、chinese_s.po 六处名称键、prefabskins 无皮肤、scrapbookdata 展示值、NET 可工作对象清单（bee/butterfly/fireflies/lightflier/moonbutterfly/moonstorm_spark/mosquito/mushtree spores）。
- 未覆盖：解构行为与掉落表等数据驱动生成未逐一排查（scripts 内 `SpawnPrefab("wortox_nabbag")` 无命中）；多人逐帧预测差异、动画压缩包内容、-efficientuser 消耗折算组合未验证。
- 游戏运行：未运行。
- 结构检查：2026-11-07 validate_page.py 通过（116 行 / 21874 字节，standard 档，上限 240 行 / 24576 字节）；validate_source_links.py --strict 0 error 0 warning（84 条源码引用）；validate_source_hashes.py check 本页未记录基线属预期（unrecorded_pages 含本页）。

相关：[[reference/components/nabbag]]；[[reference/prefabs/wortox_soulecho_buff_fx]]；[[reference/prefabs/pouch-bags]]；[[reference/components/equippable]]；[[reference/components/finiteuses]]；[[reference/components/weapon]]；[[reference/components/topics/item-actions]]。

返回 [[reference/prefabs/staffs-character-index|法杖与角色道具]]、[[reference/prefabs/index|Prefab 索引]]。
