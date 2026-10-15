---
title: "温蒂墓碑重生双幽灵动画（abigail_gravestone_rebirth_fx、wendy_gravestone_rebirth_fx）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "温蒂玩家在多年生植物祭坛复活时同一状态相邻两行生成的联网幽灵动画对：阿比盖尔层（ghost_abigail_build·wendy_resurrect）与温蒂幽灵层（ghost_wendy_build·wendy_ghost_resurrect）；只有温蒂本人触发，非温蒂走无 FX 的 gravestone_rebirth 状态。含墓碑本体一次性退场辨析与未置 persists 的存档窗口语义。"
aliases: ["abigail_gravestone_rebirth_fx", "wendy_gravestone_rebirth_fx", "温蒂墓碑重生特效", "多年生植物祭坛重生动画", "墓碑重生幽灵", "温蒂复活动画", "阿比盖尔重生动画"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/wendy_resurrectiongrave.lua", "stategraphs/SGwilson.lua", "prefabs/player_common_extensions.lua", "prefabs/wendy.lua", "prefabs/abigail.lua", "simutil.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 温蒂墓碑重生双幽灵动画（abigail_gravestone_rebirth_fx、wendy_gravestone_rebirth_fx）

## 快速判断

- 职责与适用：2 注册是**同一次重生演出的两层联网幽灵动画**——温蒂玩家在多年生植物祭坛（`wendy_resurrectiongrave`）复活时，SGwilson 的 `wendy_gravestone_rebirth` 状态在 onenter 于玩家位置**相邻两行同时生成**：`abigail_gravestone_rebirth_fx`（bank ghost、build `ghost_abigail_build`、播 `wendy_resurrect`、FinalOffset -2）与 `wendy_gravestone_rebirth_fx`（bank ghost、build `ghost_wendy_build`、播 `wendy_ghost_resurrect`、FinalOffset -1）。台账批 9"两注册同一重生动画"结论复核属实。[生成点](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3643-L3644)
- 优先使用：查"温蒂复活时的幽灵是谁、播什么动画"直接看两注册；查"别的角色用这个墓碑会怎样"——**没有任何 FX**（非温蒂走 `gravestone_rebirth` 状态，只播 `grave_spawn`）。
- 重要副作用：两 FX 无父挂（`SetPosition` 定格在玩家脚下）、生成即放手（状态不持引用）；各自监听 animover 自删；工厂**无 SetPristine／无主客分支**——全端对称执行；**未置 persists=false**。
- 修改边界：只覆盖 2 注册的工厂、生成点、端与清理；墓碑本体玩法（联接/扣血/可锤）与 SG 状态的玩家表现（无敌/相机/HUD）只作链条辨析不展开。
- 名称：墓碑官方名 `WENDY_RESURRECTIONGRAVE`＝"多年生植物祭坛"（msgid "Perennial Altar"，[po](../../../raw/current/scripts/languages/chinese_s.po#L351964-L351967)），联动命名键 `WENDY_RESURRECTIONGRAVE_NAMED`＝"{name}的多年生植物祭坛"（[po](../../../raw/current/scripts/languages/chinese_s.po#L351969-L351972)，named 组件实际使用）；2 个 FX 注册名在简中 po 全文 0 命中（含 DESCRIBE），标题中文为已核验效果描述，非官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `abigail_gravestone_rebirth_fx` | `abigailfx_fn`：Transform／AnimState／Network；标签 NOCLICK＋`FX`；bank `ghost`、build `ghost_abigail_build`、播 `wendy_resurrect`；FinalOffset -2；`ListenForEvent("animover", Remove)` | **阿比盖尔层**（更靠后的绘制层）：无专属资产声明——`ghost_abigail_build.zip` 由 abigail 实体资产提供（[abigail.lua:6](../../../raw/current/scripts/prefabs/abigail.lua#L6)），两 FX 仅温蒂玩家触发故资产随角色已加载。[工厂](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L211-L229)、[注册](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L252) |
| `wendy_gravestone_rebirth_fx` | `wendyfx_fn`：同上结构；bank `ghost`、build `ghost_wendy_build`、播 `wendy_ghost_resurrect`；FinalOffset -1；animover 自删 | **温蒂幽灵层**：`ghost_wendy_build` 是 DYNAMIC_ANIM 资产，由本注册的 Prefab 构造参数自带（ghostwendy_assets，[L21-L25](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L21-L25)）。[工厂](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L231-L249)、[注册](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L253) |

同文件未被本页收为覆盖注册的还有：`wendy_resurrectiongrave` 本体（[工厂](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L132-L204)）与 `wendy_resurrectiongraveplacer`（[L254](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L254)）；两 FX 同时列于墓碑的 prefabs 依赖（[L17-L18](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L12-L19)）与温蒂角色的 prefabs 表（[wendy.lua:36-37](../../../raw/current/scripts/prefabs/wendy.lua#L36-L37)）。

## 关联机制导航

### 重生链：谁在什么条件下生成这两个 FX

| 环节 | 触发与条件 | 端 | 证据 |
|---|---|---|---|
| 状态选择 | 玩家死亡后经联接的墓碑复活：`source.prefab == "wendy_resurrectiongrave"` 时**按角色分流**——`inst.prefab == "wendy"` 进 `wendy_gravestone_rebirth`，其他角色进 `gravestone_rebirth`（播 `grave_spawn`，**不生成任何 FX**） | 主模拟 | [分流](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L376-L381) |
| onenter | 禁玩家控制器、给玩家 override `wormmovefx` 符号（mole_build，钻出表现）、播 `wendy_resurrect`（玩家侧动画，资产 [wendy.lua:19](../../../raw/current/scripts/prefabs/wendy.lua#L19)）、**两 SpawnPrefab 定格在玩家位置**、`AddOverrideBuild("wendy_resurrect")`、无敌、隐藏 HUD、相机拉远 12、播 howl | 主模拟（实体联网） | [onenter](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3635-L3653) |
| 时间线 | 50 帧音效 `abigail_wendy_revive_f50`；80 帧 fader 白闪（27 帧升＋5 帧降，AddColour 全白） | 主模拟 | [timeline](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3655-L3671) |
| 收尾 | 玩家 animover → idle；onexit 清 override build、恢复控制器/无敌/HUD/相机并 `SerializeUserSession` | 主模拟 | [animover 与 onexit](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3673-L3697) |
| FX 退场 | 两 FX 各自 `animover → Remove`（每个端独立监听独立移除，无宿主引用需要清） | 全端各自 | [abigailfx_fn L226](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L226)、[wendyfx_fn L246](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L246) |

### 墓碑本体与重生玩法辨析

- **本体是建造设施而非 FX**：配方幽灵花 10＋石砖 1＋扣血 40（`EFFIGY_HEALTH_PENALTY`，[tuning.lua:2173](../../../raw/current/scripts/tuning.lua#L2173)）、门禁 `builder_skill="wendy_ghostflower_grave"`（[recipes.lua:166](../../../raw/current/scripts/recipes.lua#L166)）；attunable 组件挂 `gravestoneresurrector` 标签供远程联接（联接时按 `_NAMED` 键改名并亮蓝光，[onlink](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L65-L74)），共鸣扣血逻辑同肉像雕像（[onattunecost](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L49-L63)）。
- **一次性退场**：监听 `activateresurrection`（[L200-L201](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L200-L201)）→ 移除碰撞、加 DECOR、本体 `persists=false`、67 帧后播 `slide` 下沉，animover 后 `ErodeAway(inst, 4)` 侵蚀 4 秒消失（[L124-L130](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L124-L130)、[ErodeAway](../../../raw/current/scripts/simutil.lua#L575-L595)）——同一事件里 FX 未置 persists 而本体置了，两者存档语义相反。
- **可锤**：HAMMER 4 下线，掉落并由 `collapse_big`（石头材质）收尾（[onhammered](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L34-L42)）。
- 与阿比盖尔既有表现页的关系：花召唤/收回/升级见 [[reference/prefabs/abigailsummonfx|阿比盖尔之花召唤、收回与升级特效]]，反击/怨咒见 [[reference/prefabs/abigail_retaliation|阿比盖尔反击、怨咒与暗影增益特效]]——本页两 FX 是"温蒂+阿比盖尔同框"的独有表现，其余阿比盖尔动画都只渲染阿比盖尔一方。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `abigail_gravestone_rebirth_fx` | animover 自删（全端各自）；无引用持有方 | 若动画未到末帧时服务器存档，实体以 persists 默认 true 入档（读档重播后再自删；源码推导，未运行核验）。 |
| `wendy_gravestone_rebirth_fx` | 同上 | `ghost_wendy_build` 为 DYNAMIC_ANIM 按需加载资产，注册自带动 Asset 声明（[ghostwendy_assets](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L21-L25)）。 |
| 墓碑本体 | 复活后 persists=false＋slide＋ErodeAway(4)；被锤则掉落＋collapse_big＋Remove | 两者收尾互不依赖：FX 自删不通知本体，本体退场不引用 FX。 |

## 典型接入

### 做"状态机一次性演出"的生成即放手动画层

- 前提与选择：状态 onenter 需要叠加多层独立动画（不同 build/层序）时，参照本链——每层一个联网 FX 注册、`SetPosition` 定格、animover 自删，状态机不持引用也不用管清理。
- 生效与撤销：状态被中断（如传送）时动画层会留在原地播完；需要中断即清的话要在状态 onexit 里显式 Remove——本链选择"播完为止"，interrupt 场景由 SG 的 silentmorph／respawn 流程覆盖。

```lua
-- SG 状态 onenter 内（对应 wendy_gravestone_rebirth 的写法）
SpawnPrefab("abigail_gravestone_rebirth_fx").Transform:SetPosition(inst.Transform:GetWorldPosition())
SpawnPrefab("wendy_gravestone_rebirth_fx").Transform:SetPosition(inst.Transform:GetWorldPosition())
```

- 调整与验证：替换点是注册名与 FinalOffset 层序（负值在角色身后，正值会盖住角色）；工厂无主客分支，加逻辑前先确认要不要 SetPristine。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：两个 FX 是"二选一"或一个是另一个的皮肤。** 两行相邻生成、同位置同时播放，是同一演出的两层（阿比盖尔 -2 在更后层、温蒂幽灵 -1）；缺任一层演出都不完整（[L3643-L3644](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3643-L3644)）。
- **边界：只有温蒂本人才有 FX。** 非温蒂角色（含双人联机里帮温蒂守墓的其他玩家死亡后）在此墓碑复活走 `gravestone_rebirth`，无任何 FX 生成（[分流](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L376-L381)）——把这两注册当"墓碑通用特效"检索会落空。
- **边界：工厂没有 SetPristine／主客分支。** 两工厂在服务端与客户端执行同一份代码（含 animover 监听），各端各自播各自删；照搬"SetPristine＋ismastersim 早退"模板往里加主客逻辑会破坏对称性（[abigailfx_fn](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L211-L229)）。
- **边界：未置 persists=false 的存档窗口。** 重生动画期间若触发存档，两 FX 会随 persists 默认 true 写入世界存档，读档后按工厂重播并再次 animover 自删（源码推导，未运行核验）；同一事件的墓碑本体反而是 persists=false——别把两边的存档语义记反。
- **边界：FinalOffset 全为负值。** 两层都绘制在角色身后（-2/-1）；复用时改成正値会让幽灵盖住角色本体。
- **边界：玩家动画与 FX 动画是两套。** 玩家本体播 `wendy_resurrect`（wilson bank 派生 build），FX 层播 ghost bank 的 `wendy_resurrect`／`wendy_ghost_resurrect`——同名 `wendy_resurrect` 分属不同动画包，检索时注意 bank。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/wendy_resurrectiongrave.lua`，按需读取 `stategraphs/SGwilson.lua`（gravestone_rebirth／wendy_gravestone_rebirth 两状态全文）、`prefabs/player_common_extensions.lua`（复活分流段）、`prefabs/wendy.lua`（prefabs 表与资产行）、`simutil.lua`（ErodeAway）、`recipes.lua`／`tuning.lua`（配方与扣血常量）、`prefabs/abigail.lua`（资产行）。对 2 个注册名以完整名在 `raw/current/scripts` 全域反查：`abigail_gravestone_rebirth_fx` 命中注册（wendy_resurrectiongrave.lua:252）、依赖（:17、wendy.lua:36）、生成（SGwilson.lua:3643）——**生成点唯一**；`wendy_gravestone_rebirth_fx` 命中注册（:253）、依赖（:18、wendy.lua:37）、生成（SGwilson.lua:3644）——**生成点唯一**。两状态的选择条件（player_common_extensions.lua:376-381）与状态全文（SGwilson.lua:3581-3697）逐行核验。未运行游戏。
- 名称核验：`languages/chinese_s.po` 定向检索——`WENDY_RESURRECTIONGRAVE`＝"多年生植物祭坛"、`WENDY_RESURRECTIONGRAVE_NAMED`＝"{name}的多年生植物祭坛"（锚点见"快速判断"，后者为 named 组件实际键）；`ABIGAIL_GRAVESTONE_REBIRTH_FX`／`WENDY_GRAVESTONE_REBIRTH_FX` 全文 0 命中，标题使用效果描述。
- 分档说明：`size_class: standard`（省略字段按默认档）——2 注册单链，生成/清理对称，主体辨析集中在状态选择与存档语义两处。
- 未覆盖：温蒂技能树（`wendy_ghostflower_grave` 门禁）与 attuner 远程复活系统全链、`gravestone_rebirth`（非温蒂状态）的完整表现、玩家 SG 的 silentmorph 标签语义、动画包帧数与观感、存档窗口的运行时实测；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/abigailsummonfx|阿比盖尔之花召唤、收回与升级特效]]（阿比盖尔主题的表现链主页）
- [[reference/prefabs/ghost_transform_overlay_fx|幽灵复活转生光效]]（通用幽灵复活叠层，与本页角色限定 FX 对照）
- [[reference/prefabs/ghostflower_spirit1_fx|幽灵花灵体飘光]]（本墓碑配方的材料幽灵花）
- [[reference/prefabs/ghostlyelixir_slowregen_fx|幽灵灵药环绕与滴落特效]]（阿比盖尔灵药表现）
- 返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]（温蒂/阿比盖尔重生）
