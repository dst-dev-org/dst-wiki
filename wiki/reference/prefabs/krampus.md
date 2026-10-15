---
title: "坎普斯（krampus）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "俗称小偷的坎普斯：淘气值生成入口、Brain 拾取与砸箱、SG 执行动作和退出、战斗与掉落组装。"
aliases: ["krampus", "坎普斯", "小偷", "Krampus", "坎普斯偷东西", "小偷砸箱子", "坎普斯掉落"]
tags: [prefab, creature, ai, state-machine, server-client]
dst_build: "steam-buildid:26230496"
sources: ["prefabs/krampus.lua", "brains/krampusbrain.lua", "stategraphs/SGkrampus.lua", "components/kramped.lua", "standardcomponents.lua", "tuning.lua", "components/lootdropper.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 坎普斯（krampus）

## 快速判断

- 职责与适用：原版偷取地面物品、砸开特定箱子的生物；“小偷”为用户提供的俗称，用于检索，不保证其他语境唯一指向本对象。[偷取选物与砸箱](../../../raw/current/scripts/brains/krampusbrain.lua#L12-L50)
- 优先使用：淘气值和生成数量查 [[reference/components/kramped#关键接口]]；改变坎普斯偷取对象或离场条件查本页 Brain／SG 导航。它没有在工厂挂通用 thief，不能直接套青蛙的命中偷窃机制。[工厂组装](../../../raw/current/scripts/prefabs/krampus.lua#L85-L120)
- 重要副作用：Brain 的偷取使用 PICKUP，砸箱使用 HAMMER；达到贪欲条件转 exit，进入不可打断／无敌离场流程，动画结束 Remove。[决策](../../../raw/current/scripts/brains/krampusbrain.lua#L91-L110)、[退出态](../../../raw/current/scripts/stategraphs/SGkrampus.lua#L156-L199)
- 修改边界：公共构造只布置物理、外观和标签；主模拟安装 SG／Brain 及生命战斗组件。本文只覆盖相关动作与退出主链，不替代完整 SG／Brain 专页或生成管理器知识。[端分界](../../../raw/current/scripts/prefabs/krampus.lua#L51-L120)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `krampus` | `fn` → 公共 CharacterPhysics／四向动画／标签 → SetPristine → inventory、locomotor → SGkrampus → KrampusBrain → 其余主模拟组件 | locomotor 先于 SG；require 仅加载 Brain 模块，实际安装是 SetBrain。[构造与安装](../../../raw/current/scripts/prefabs/krampus.lua#L51-L120)、[唯一注册](../../../raw/current/scripts/prefabs/krampus.lua#L125) |

名称键 `STRINGS.NAMES.KRAMPUS` 对应“坎普斯”／Krampus；工厂无名称覆盖。[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L343820-L343822)、[默认名称映射](../../../raw/current/scripts/entityscript.lua#L840-L844)。本文件还声明 krampus_sack、krampuscorpse 等依赖，但并未注册这些 Prefab。[依赖声明](../../../raw/current/scripts/prefabs/krampus.lua#L8-L14)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| inventory／locomotor | inventory.ignorescangoincontainer=true；移动速度读 KRAMPUS_SPEED，先装 locomotor 再装 SG | 主模拟；偷物数量和移动由 Brain 使用，不是 container | [[reference/components/inventory#关键接口]]；[[reference/components/locomotor#移动与动作边界]]；[配置](../../../raw/current/scripts/prefabs/krampus.lua#L85-L92) |
| health／combat／sleeper | 最大生命、伤害、攻击间隔读 TUNING；sleeper.watchlight=true | 主模拟；本版基础生命 300、伤害 50、间隔 1.2、跑速 7 | [[reference/components/health#关键接口]]；[[reference/components/combat#关键接口]]；[[reference/components/sleeper#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/krampus.lua#L101-L109)、[数值](../../../raw/current/scripts/tuning.lua#L1547-L1550) |
| burnable／propagator／freezable／hauntable | LargeBurnableCharacter 同时加大型 propagator 并将 acceptsheat 置 false；LargeFreezableCharacter 配抗性 3；作祟唤醒并恐慌 | 主模拟 helper 间接添加；不能忽略 helper 内的组件关系 | [[reference/components/burnable#关键接口]]；[[reference/components/propagator#关键接口]]；[[reference/components/freezable#状态机受击与清理]]；[[reference/components/hauntable#按效果选工厂]]；[燃烧 helper](../../../raw/current/scripts/standardcomponents.lua#L271-L281)、[冻结 helper](../../../raw/current/scripts/standardcomponents.lua#L353-L359)、[作祟 helper](../../../raw/current/scripts/standardcomponents.lua#L1339-L1357) |
| lootdropper／inspectable | 设置共享概率表 krampus；检视记录 RecordViews | 主模拟；声明掉落与最终生成分开 | [[reference/components/lootdropper#关键接口]]；[[reference/components/inspectable#关键接口]]；[配置](../../../raw/current/scripts/prefabs/krampus.lua#L111-L115) |
| Brain：KrampusBrain | 选取可偷物、非空普通木箱；追击、偷取、离场、躲玩家与跟随 | SetBrain 实际安装；SetTarget 跟踪目标并监听其移除，OnStop 清理 | Wiki 待建：[定义与目标](../../../raw/current/scripts/brains/krampusbrain.lua#L59-L89)、[决策树](../../../raw/current/scripts/brains/krampusbrain.lua#L91-L112)；[安装](../../../raw/current/scripts/prefabs/krampus.lua#L92) |
| SG：SGkrampus | PICKUP→steal、HAMMER→hammer；动画时间线执行缓冲动作；exit 控无敌与移除 | 主模拟安装；公共睡眠／奔跑／冻结／尸体状态由 helper 补入，本文未逐一展开 | Wiki 待建：[动作映射](../../../raw/current/scripts/stategraphs/SGkrampus.lua#L3-L7)、[偷取时间线](../../../raw/current/scripts/stategraphs/SGkrampus.lua#L202-L225)、[退出态](../../../raw/current/scripts/stategraphs/SGkrampus.lua#L156-L199) |
| attacked／newcombattarget | 受击把攻击者设为 combat target；新战斗目标转通知 Brain.SetTarget | 主模拟；不可把 combat.target 和 Brain.mytarget 当一个字段 | [回调](../../../raw/current/scripts/prefabs/krampus.lua#L26-L49)、[监听](../../../raw/current/scripts/prefabs/krampus.lua#L117-L118) |
| kramped：世界侧生成者 | 达到淘气阈值后生成 krampus，记录 spawnedforplayer 供 Brain 首次读取 | 此组件在世界上，不由坎普斯工厂安装 | [[reference/components/kramped#关键接口]]；[生成](../../../raw/current/scripts/components/kramped.lua#L45-L55)、[阈值分支](../../../raw/current/scripts/components/kramped.lua#L58-L81)、[Brain 消费](../../../raw/current/scripts/brains/krampusbrain.lua#L88-L89) |
| 掉落：monstermeat／charcoal／krampus_sack | 概率表列一份肉、两份炭及袋子，lootdropper 消费 | krampus_sack 基础表值 0.01；这些对象 Wiki 待建，本文只维护掉落关系 | [掉落表](../../../raw/current/scripts/prefabs/krampus.lua#L18-L24)、[绑定](../../../raw/current/scripts/prefabs/krampus.lua#L111-L112) |
| 尸体依赖／资源 | krampuscorpse 列入依赖；声明 krampus_basic、krampus_build 动画及音效包 | 尸体工厂和完整死亡生成链未展开，不从依赖数组推导生成时机；不因强关联复制尸体／背包行为 | [声明](../../../raw/current/scripts/prefabs/krampus.lua#L1-L14)、[死亡入口](../../../raw/current/scripts/stategraphs/SGkrampus.lua#L135-L152) |

## 典型接入

### 小偷如何选择地面物品和箱子

StealAction 在 inventory 未满时，于 30 距离搜索可拾取、处于有效地面、离任何玩家至少判定半径 6 之外的物品，并排除 INLIMBO、catchable、fire、irreplaceable、heavy、prey、bird、outofreach、_container 标签，返回 PICKUP 缓冲动作。不要用“任意物品都能偷”概括。[选物条件](../../../raw/current/scripts/brains/krampusbrain.lua#L9-L32)

EmptyChest 只选择 prefab 为 treasurechest、带非空 container、远离玩家且在有效地面的对象，返回 HAMMER；不是泛化的“打开所有容器偷内容”。本页不覆盖箱子受锤击后的实现。[砸箱条件](../../../raw/current/scripts/brains/krampusbrain.lua#L34-L50)

### Brain 决策、SG 执行和离场条件

Brain 贪欲值初始化为 `2 + math.random(4)`；当 inventory:NumItems() 达到该值且 SG 不 busy 时请求 exit。高优先级的恐慌／追击等分支仍可能先执行，不能仅看数量就保证立即消失。[初值](../../../raw/current/scripts/brains/krampusbrain.lua#L59-L63)、[优先级](../../../raw/current/scripts/brains/krampusbrain.lua#L97-L110)

偷取动作交由 SG 的 steal 状态，在 18 帧时 PerformBufferedAction；exit 状态停 Brain、停移动、设无敌并去碰撞，退出动画完成时移除；onexit 有恢复无敌／物理／Brain 的兜底。若修改离场，须同时处理状态和清理，而非只删 Brain 的一个判断。[偷取执行](../../../raw/current/scripts/stategraphs/SGkrampus.lua#L202-L225)、[离场生命周期](../../../raw/current/scripts/stategraphs/SGkrampus.lua#L156-L199)

### 调整掉落应改哪一层

本工厂给 lootdropper 绑定共享表 krampus，表内袋子基础概率 0.01；GenerateLoot 消费概率表时还会经过 GetChance 的幸运修正路径。因此表值不应表述为所有场景最终恒定的百分之一。通用修改方式查 [[reference/components/lootdropper#关键接口]]，共享表修改会影响使用该表的其他实例。[绑定与表值](../../../raw/current/scripts/prefabs/krampus.lua#L18-L24)、[绑定](../../../raw/current/scripts/prefabs/krampus.lua#L111-L112)、[幸运修正](../../../raw/current/scripts/components/lootdropper.lua#L173-L184)、[表消费](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)

## 常见陷阱与限制

- **误区：小偷就应该挂 thief**：本版是 Brain 选择 PICKUP／HAMMER，由 SG 调用缓冲动作；[[reference/components/thief#沿用普通青蛙的命中偷窃链]] 是另一种机制，不可据俗称混用。
- **误区：SpawnPrefab("krampus") 等同淘气值惩罚生成**：世界生成者还设置 spawnedforplayer 和移除监听，Brain 启动时读取该目标；直接生成没有这些额外接线。[生成者](../../../raw/current/scripts/components/kramped.lua#L45-L55)、[启动读取](../../../raw/current/scripts/brains/krampusbrain.lua#L88-L89)
- **误区：源文件出现 eater 就表示会吃肉**：对应 AddComponent 和配置整段被长注释包围，不属于实际工厂组件。[注释片段](../../../raw/current/scripts/prefabs/krampus.lua#L97-L99)
- **误区：死亡与离场都直接掉落全部背包物品**：死亡态调用 DropDeathLoot，后者按 health.is_corpsing 决定存储或生成 lootdropper 名单；exit 走移除。本页未审校背包内容、尸体回收和所有死亡 helper 的完整清理，不能作同一掉落承诺。[死亡入口](../../../raw/current/scripts/stategraphs/SGkrampus.lua#L135-L152)、[掉落分流](../../../raw/current/scripts/entityscript.lua#L2181-L2192)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整读取 krampus.lua 和 krampusbrain.lua，定向核验 SG 的动作／偷取／死亡／离场、kramped 的生成与初始目标、工厂 helper、TUNING、概率表消费及名称条目。“小偷”按用户明确提供的俗称收录。
- 游戏运行：未运行；未审校全部公共 SG 状态、行为节点内部调度、尸体工厂、所有来源生成方式、存档和多人目标竞争。资源仅核验声明／使用，未解包验证；无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
