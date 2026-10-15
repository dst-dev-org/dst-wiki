---
title: "蜘蛛治疗与突变（spider_heal_fx）"
created: "2026-10-31"
updated: "2026-11-01"
type: prefab
summary: "四个通用表注册承载蜘蛛阵营的治疗与突变表现：护士蜘蛛 heal 状态 30 帧群疗（自处地面圈＋本体＋每只被疗蜘蛛目标光）、韦伯治疗黏团使用链复用同三名，突变烟在蜘蛛两套状态图 animover 落点。治疗声由调用方播放，fx 无声。"
aliases: ["spider_mutate_fx", "spider_heal_fx", "spider_heal_target_fx", "spider_heal_ground_fx", "蜘蛛治疗", "护士蜘蛛", "治疗黏团", "蜘蛛突变", "治疗光效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/spider.lua", "prefabs/spider_healer_item.lua", "stategraphs/SGspider.lua", "stategraphs/SGspider_water.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 蜘蛛治疗与突变（spider_heal_fx）

## 快速判断

- 职责与适用：蜘蛛阵营两类表现——治疗（护士蜘蛛群疗与韦伯治疗黏团，三名）与蜘蛛突变（一名）；治疗声（heal_fartcloud）与突变声由调用方播放。工厂协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目组](../../../raw/current/scripts/fx.lua#L2599-L2626)
- 优先使用：查"蜘蛛治疗／护士蜘蛛／治疗黏团／蜘蛛突变"到本页；治疗量与组件结算读 [[reference/components/healer|healer]]（黏团对韦伯的手动治疗在物品侧）。
- 重要副作用：heal 三名均无 sound 字段；地面圈用 GroundOrientation 贴背景层。[条目](../../../raw/current/scripts/fx.lua#L2621-L2626)
- 修改边界：群疗范围与目标过滤在调用方（SPIDER_HEALING_RADIUS／HEALING_ITEM_RADIUS 两套）；护士蜘蛛的 DoHeal 是实体方法（spider.lua 装配）。[群疗](../../../raw/current/scripts/prefabs/spider.lua#L478-L508)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `spider_mutate_fx` | 通用表条目，无 fn | bank/build=`mutate_fx`，anim=`mutate`，`nofaced`，无声。[条目](../../../raw/current/scripts/fx.lua#L2599-L2604) |
| `spider_heal_fx` | 同上协议 | bank=`heal_fx`、build=`spider_heal_fx`，anim=`heal`，无声。[条目](../../../raw/current/scripts/fx.lua#L2607-L2611) |
| `spider_heal_target_fx` | 同上协议 | 同资源，anim=`heal_buff`（被疗个体光），无声。[条目](../../../raw/current/scripts/fx.lua#L2614-L2618) |
| `spider_heal_ground_fx` | 条目带 fn=`GroundOrientation`（OnGround＋LAYER_BACKGROUND） | 同资源，anim=`heal_aoe`（地面圈），无声。[条目](../../../raw/current/scripts/fx.lua#L2621-L2626) |

四名在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。相关实体官方名：护士蜘蛛 [po:349240-349242](../../../raw/current/scripts/languages/chinese_s.po#L349240-L349242)、治疗黏团 [po:349245-349247](../../../raw/current/scripts/languages/chinese_s.po#L349245-L349247)。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 代理＋一帧本地表现，落点式 | 位置由各 helper 的 SetPosition 决定 | [[reference/mechanisms/generic-fx-factory]] |
| 生成者：护士蜘蛛群疗 | `spider_healer`（韦伯突变体，`mutator_healer` 配方）的 SG `heal` 状态 30 帧调 `inst.DoHeal()`：1.35 倍缩放落 ground＋heal 两枚于自身，范围（SPIDER_HEALING_RADIUS，标签 spider／spiderwhisperer／spiderqueen）内按代码过滤后每只受疗蜘蛛落 target 光并回血——过滤读**治疗者自身** `combat.target`：`targetting_us` 判断该目标是否为治疗者自身／领袖／领袖随从，`targetted_by_us` 判断治疗者／领袖／领袖随从是否正以候选蜘蛛为目标（源码注释措辞与变量来源不一致，以代码为准） | 服务端；治疗声 `heal_fartcloud` 由 DoHeal 播 | [DoHeal](../../../raw/current/scripts/prefabs/spider.lua#L478-L508)、[heal 状态](../../../raw/current/scripts/stategraphs/SGspider.lua#L479-L490)、[装配](../../../raw/current/scripts/prefabs/spider.lua#L967-L992) |
| 生成者：治疗黏团 | 韦伯专属配方物品（蜂蜜2＋灰2＋蛛丝2）：工厂挂 `healer` 组件并 `SetHealthAmount(0)`、经 `SetOnHealFn(OnHealFn)` 接回调——具体回血全部由回调手动执行：目标处播声、韦伯本人（spiderwhisperer）手动回血并落 target 光、落 ground＋heal 两枚、半径（SPIDER_HEALING_ITEM_RADIUS）内每只蜘蛛回血＋target 光 | 服务端；healer 组件只提供使用协议，治疗量为 0 | [组件接线](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L71-L73)、[OnHealFn](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L25-L47)、[配方](../../../raw/current/scripts/recipes.lua#L328-L328) |
| 生成者：蜘蛛突变 | SGspider 与 SGspider_water 的 `mutate` 状态 animover：落点式生成 `spider_mutate_fx`，0.25 秒后执行 do_mutate | 服务端；水蜘蛛路径播 `webber2/common/mutate` | [SGspider](../../../raw/current/scripts/stategraphs/SGspider.lua#L614-L617)、[SGspider_water](../../../raw/current/scripts/stategraphs/SGspider_water.lua#L496-L505) |
| 生成 helper | spider.lua `SpawnHealFx(inst, name, scale)` 与物品侧 `SpawnFx`：SetNoFaced＋SetPosition＋可选 SetScale | 服务端；护士群疗带 1.35 缩放，黏团链默认 1 | [SpawnHealFx](../../../raw/current/scripts/prefabs/spider.lua#L468-L476) |
| 资源 | 突变一套（mutate_fx）、治疗一套（heal_fx bank／spider_heal_fx build 三 anim） | 动画时长未核验 | [条目组](../../../raw/current/scripts/fx.lua#L2599-L2626) |

## 典型接入

### 模组复用治疗表现

- 前提与选择：给自定义"范围治疗"接原版视觉时，直接落 `spider_heal_ground_fx`（地面圈）＋`spider_heal_fx`（本体）组合，被治疗个体另落 `spider_heal_target_fx`；治疗声需自播（fx 无声）。
- 生效与撤销：落点一次性动画，animover 自删。
- 原版依据：[DoHeal 组合](../../../raw/current/scripts/prefabs/spider.lua#L481-L482)。

```lua
-- 服务端：范围治疗的表现组合（结算由模组自理）
local function DoHealFx(inst, scale)
    SpawnPrefab("spider_heal_ground_fx").Transform:SetPosition(inst.Transform:GetWorldPosition())
    SpawnPrefab("spider_heal_fx").Transform:SetPosition(inst.Transform:GetWorldPosition())
end
```

- 调整与验证：原版带 1.35 缩放（Transform:SetScale）；仅给个体加光可单落 target 光注册。

## 常见陷阱与限制

- **误区：治疗量在 fx 里**：三名治疗注册只承担表现；黏团挂 healer 组件但治疗量置 0，回血数值与目标筛选（护士侧按治疗者自身 combat.target 双向检查）都在回调与 DoHeal 调用方，改表现不影响结算。[接线](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L71-L73)
- **边界：fx 全部无声**：治疗屁云声（`heal_fartcloud`，护士与黏团各走自己的 SoundPath）与突变声（`webber2/common/mutate`）由调用方播放；条目无 sound 字段。[DoHeal](../../../raw/current/scripts/prefabs/spider.lua#L479-L480)
- **边界：两套范围常量**：护士群疗用 `SPIDER_HEALING_RADIUS`，治疗黏团用 `SPIDER_HEALING_ITEM_RADIUS`；表现相同但范围与治疗对象不同（黏团链还单独手动治疗韦伯本人）。[OnHealFn](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L36-L45)
- **边界：突变烟与变身解耦**：`spider_mutate_fx` 在 animover 先落，实际突变（do_mutate）延迟 0.25 秒——表现先行不代表已完成突变。[时序](../../../raw/current/scripts/stategraphs/SGspider.lua#L614-L617)
- **边界：护士蜘蛛是突变体而非普通蜘蛛**：群疗入口挂在 `spider_healer`（spider_wolf_build、mutator_healer 配方）；普通蜘蛛与水蜘蛛没有 heal 状态入口（水蜘蛛仅复用突变烟）。[装配](../../../raw/current/scripts/prefabs/spider.lua#L967-L992)

## 验证记录

- 源码审阅：2026-10-31，Steam buildid 26230496。四注册配置（含 GroundOrientation）、护士蜘蛛 heal 状态与 DoHeal 群疗（范围／过滤／缩放）、治疗黏团 OnHealFn 与配方、两套 mutate 状态时序、po 两名核验。未覆盖：mutate_fx 动画资源、heal 状态的 Brain 触发条件、healtime 冷却消费、晚加入时序。
- 游戏运行：未运行；治疗与突变视觉未实测。

返回 [[reference/prefabs/fx-status-index|增益、受击、护盾与状态反馈]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
