---
title: "变身切换烟（werebeaver_transform_fx）"
created: "2026-10-31"
updated: "2026-11-01"
type: prefab
summary: "八个通用表注册承载三类形态切换烟团：伍迪三形态变身（含鹿形态中断补烟与鹅毛色覆写）、露西斧收编与还原（possessedaxe 两字段），以及被全玩家接住抛射物复用的 lucy_transform_fx；切斯特／哈奇形态切换烟同为死亡烟资源。"
aliases: ["werebeaver_transform_fx", "weremoose_transform_fx", "weremoose_transform2_fx", "weremoose_revert_fx", "weregoose_transform_fx", "lucy_transform_fx", "lucy_ground_transform_fx", "chester_transform_fx", "变身烟", "变形特效", "接住烟", "形态切换特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "stategraphs/SGwilson.lua", "prefabs/woodie.lua", "components/possessedaxe.lua", "prefabs/lucy.lua", "prefabs/chester.lua", "stategraphs/SGchester.lua", "prefabs/hutch.lua", "stategraphs/SGhermitcrab.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 变身切换烟（werebeaver_transform_fx）

## 快速判断

- 职责与适用：八个通用表注册承载三类"形态切换"烟团——伍迪三形态变身（5 名：海狸／鹿三段＋回退／鹅）、露西斧收编／还原（2 名）、切斯特／哈奇形态切换（1 名）；另被全玩家接住抛射物复用。工厂协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目组](../../../raw/current/scripts/fx.lua#L1062-L1107)
- 优先使用：查"变身烟／接住烟／形态切换特效"到本页；露西斧字段接线读 [[reference/components/possessedaxe|possessedaxe]]。
- 重要副作用：变身烟多为落点一次性表现；收编／接住烟经代理符号跟随或父挂一次性取位，不持续跟随宿主。[接住烟](../../../raw/current/scripts/stategraphs/SGwilson.lua#L18920-L18920)
- 修改边界：八名均无 fx 显示名称键，标题中文为效果描述；接住烟路径是全玩家共享行为，改动影响所有角色。[equip 分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2120-L2121)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `werebeaver_transform_fx` | 通用表条目，无 fn | bank／build=`werebeaver_fx`，anim=`transform_back`，sound=`deathpoof`。[条目](../../../raw/current/scripts/fx.lua#L1074-L1078) |
| `weremoose_transform_fx` | 同上协议 | bank／build=`weremoose_poof_fx`，anim=`transform`，**无 sound**（吼声由 SG 播）。[条目](../../../raw/current/scripts/fx.lua#L1081-L1084) |
| `weremoose_transform2_fx` | 同上协议 | 同 bank，anim=`transform2`，sound=`deathpoof`。[条目](../../../raw/current/scripts/fx.lua#L1087-L1091) |
| `weremoose_revert_fx` | 同上协议 | 同 bank，anim=`revert`，sound=`deathpoof`。[条目](../../../raw/current/scripts/fx.lua#L1094-L1098) |
| `weregoose_transform_fx` | 条目带 fn：覆写 `were_fur01` 符号为 `weregoose_fx` 版 | bank=`weregoose_fx`、**build 复用 `werebeaver_fx`**，anim=`transform_back`，sound=`deathpoof`。[条目](../../../raw/current/scripts/fx.lua#L1101-L1107) |
| `lucy_transform_fx` | 同上协议，无 fn | bank=`lucy_axe_fx`、build=`axe_transform_fx`，anim=`transform_chop`，无 sound。[条目](../../../raw/current/scripts/fx.lua#L1068-L1072) |
| `lucy_ground_transform_fx` | 同上协议，无 fn | 同 bank／build，anim=`transform_ground`，无 sound。[条目](../../../raw/current/scripts/fx.lua#L1062-L1066) |
| `chester_transform_fx` | 同上协议，无 fn | **bank=`die_fx`、build=`die`、anim=`small`**（死亡烟资源）。[条目](../../../raw/current/scripts/fx.lua#L558-L561) |

八名在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。伍迪／露西斧／切斯特／哈奇官方名见 [po:351765-351766](../../../raw/current/scripts/languages/chinese_s.po#L351765-L351766)、[po:344230-344231](../../../raw/current/scripts/languages/chinese_s.po#L344230-L344231)、[po:340945-340946](../../../raw/current/scripts/languages/chinese_s.po#L340945-L340946)、[po:343425-343426](../../../raw/current/scripts/languages/chinese_s.po#L343425-L343426)。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 八名均为代理＋一帧本地表现，fn(inst, proxy) 同步调用 | 落点式或符号取位两路 | [[reference/mechanisms/generic-fx-factory]] |
| 生成者：伍迪变身状态族 | SGwilson 六状态在动画关键帧／animover 以 `Transform:SetPosition(玩家位置)` 落点生成 | 服务端；`transform_werebeaver`／`transform_beaver_person`（beaver，animover）、`transform_weremoose`（10 帧 transform、27 帧 transform2、中断 onexit 补 transform2）、`transform_moose_person`（14 帧 revert）、`transform_weregoose`／`transform_goose_person`（animover） | [beaver](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3022-L3022)、[beaver 回](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3086-L3086)、[moose 帧组](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3139-L3157)、[中断补烟](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3188-L3188)、[moose 回](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3234-L3234)、[goose](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3314-L3314)、[goose 回](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3367-L3370) |
| 复用：闪避烟 | 鹅形态闪避技能（[[reference/components/attackdodger|attackdodger]] 消费方 `OnDodgeAttack`）落 `weregoose_transform_fx` 并放大 1.3 倍 | 服务端；`woodie_curse_goose_3` 技能线 | [OnDodgeAttack](../../../raw/current/scripts/prefabs/woodie.lua#L916-L921) |
| 复用：接住抛射物烟 | 玩家接住回旋镖类抛射物（`data.item.projectileowner ~= nil`）：busy 时 equip 事件分支与 `catch_equip` 状态（9 帧，onexit 未播则兜底）都落 `lucy_transform_fx`，代理挂为 `swap_object` 符号跟随器（偏移 50,-25） | 服务端生成、双端显示；**全玩家共享**，珍珠同款 equip 分支 | [equip 分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2120-L2121)、[catch_equip](../../../raw/current/scripts/stategraphs/SGwilson.lua#L18918-L18939)、[珍珠](../../../raw/current/scripts/stategraphs/SGhermitcrab.lua#L400-L400) |
| 生成者：露西斧收编／还原 | `woodie` 收编流程读 `possessedaxe.transform_fx` 落烟（符号跟随手部）；`PossessedAxe:Revert()` 落 `revert_fx`——地上分支在原地落点、装备中分支符号跟随持有者、**容器槽位分支无特效** | 服务端；字段由 lucy.lua 装配 | [收编](../../../raw/current/scripts/prefabs/woodie.lua#L685-L704)、[Revert 三分支](../../../raw/current/scripts/components/possessedaxe.lua#L167-L198)、[字段](../../../raw/current/scripts/prefabs/lucy.lua#L225-L226) |
| 生成者：切斯特／哈奇切换 | SGchester `transition`（56 帧，y+1；`CheckForMorph` 在满月或关容器时按容器内容物选暗影／雪切斯特）与 `morph`（36 帧，y+1；哈奇经 `PushEvent("morph")` 走 [[reference/components/amorphous|amorphous]] 表单切换）落 `chester_transform_fx` | 服务端；pop／raise 等声音由 SG 另播 | [CheckForMorph](../../../raw/current/scripts/prefabs/chester.lua#L528-L533)、[transition 帧](../../../raw/current/scripts/stategraphs/SGchester.lua#L313-L316)、[morph 帧](../../../raw/current/scripts/stategraphs/SGchester.lua#L363-L366)、[哈奇入口](../../../raw/current/scripts/prefabs/hutch.lua#L305-L305) |
| 资源 | 变身烟三套 bank（werebeaver_fx／weremoose_poof_fx／weregoose_fx）、露西一套（lucy_axe_fx）、切斯特复用死亡烟（die） | 动画时长与循环性未核验 | [条目组](../../../raw/current/scripts/fx.lua#L1062-L1107) |

## 典型接入

### 模组变身入口复用变身烟

- 前提与选择：模组自定义形态切换想沿用原版视觉时，直接落 `werebeaver_transform_fx`（自带通用烟声）；需要手部位置的表现再参考接住烟的符号跟随写法。
- 生效与撤销：落点一次性动画，animover 自删；本地表现一帧后生成于代理位置。
- 原版依据：[SGwilson 落点写法](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3022-L3022)、[ possessedaxe 字段](../../../raw/current/scripts/prefabs/lucy.lua#L225-L226)。

```lua
-- 服务端（如人物 master_postinit 的变身入口回调）：
local function OnMyTransform(inst)
    local fx = SpawnPrefab("werebeaver_transform_fx")
    fx.Transform:SetPosition(inst.Transform:GetWorldPosition())
end
-- 露西斧同款字段（仅 woodie 收编流程消费该字段，见陷阱）：
-- inst.components.possessedaxe.transform_fx = "lucy_transform_fx"
-- inst.components.possessedaxe.revert_fx = "lucy_ground_transform_fx"
```

- 调整与验证：换注册名即换表现（狸／鹿／鹅／露西）；鹅烟自动换毛色符号，无需额外覆写。

## 常见陷阱与限制

- **误区：`chester_transform_fx` 是变形专属动画**：实为死亡烟资源（`die` build `small`）；切斯特与哈奇的切换音效（raise／pop／clap）全部由 SG 播放，fx 本身无声。[条目](../../../raw/current/scripts/fx.lua#L558-L561)
- **边界：鹅烟与海狸烟同 build**：`weregoose_transform_fx` build 复用 `werebeaver_fx`，仅凭条目 fn 覆写 `were_fur01` 换鹅毛色；按 build 归类会漏掉差异。[条目](../../../raw/current/scripts/fx.lua#L1101-L1107)
- **边界：鹿形态首段无声**：`weremoose_transform_fx` 无 sound 字段，吼声由 SG 在 25 帧播放；transform2／revert 自带通用烟声。[帧组](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3139-L3157)
- **误区：`lucy_transform_fx` 是伍迪专属**：所有玩家接住抛射物（含珍珠）都用它当接住烟；修改该注册或其资源会改变全部角色的接住表现。[equip 分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2120-L2121)
- **边界：possessedaxe 还原的容器分支无特效**：`Revert()` 只在"无容器（原地）"与"装备中（符号跟随）"两分支生成 `revert_fx`，物品静躺在容器槽位时还原无表现。[分支](../../../raw/current/scripts/components/possessedaxe.lua#L167-L198)
- **边界：`transform_fx` 字段的消费方在伍迪收编流程**：其他实体给 possessedaxe 设该字段不会被自动消费（原版读取点在 woodie.lua）；复用表现请直接落点或自写跟随。[收编](../../../raw/current/scripts/prefabs/woodie.lua#L685-L704)
- **边界：符号取位不持续跟手**：接住烟／收编烟／还原烟的代理挂为符号跟随器后，本地表现一帧后一次性复制位置，不随挥动／移动刷新；协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[跟随写法](../../../raw/current/scripts/components/possessedaxe.lua#L189-L194)

## 验证记录

- 源码审阅：2026-10-31，Steam buildid 26230496。八注册配置（bank／build／anim／sound／fn）、SGwilson 六变身状态帧位与中断补烟、接住烟两条路径与珍珠复用、收编／还原三分支、切斯特／哈奇两状态与入口、po 四名核验。未覆盖：动画资源内容、wereness 组件触发链、wereplayer 技能树分支细节、晚加入时序。
- 游戏运行：未运行；烟团视觉未实测。

返回 [[reference/prefabs/fx-transition-index|变身、召唤、传送与退场]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
