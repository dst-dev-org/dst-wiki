---
title: "强心针（lifeinjector）"
created: "2026-11-10"
updated: "2026-11-11"
type: prefab
size_class: simple
summary: "唯一挂 maxhealer 组件的原版物品：用 HEAL 动作把目标生命上限惩罚减 0.25（不回当前生命）；候选门禁无 healerbuffs 例外，禁疗目标（Wanda、墙类）根本出不来动作。"
aliases: ["lifeinjector", "强心针", "生命注入器", "Booster Shot", "生命上限惩罚恢复"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/lifeinjector.lua", "prefablist.lua", "components/maxhealer.lua", "components/health.lua", "components/health_replica.lua", "actions.lua", "componentactions.lua", "recipes.lua", "recipes_filter.lua", "prefabs/wanda.lua", "standardcomponents.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 强心针（lifeinjector）

## 快速判断

- 职责与适用：可拾取、可堆叠的**库存物品**（`lifepen` 外观），全库唯一挂 `maxhealer` 的原版注册；用它执行治疗不减当前生命，而是把目标的**生命上限惩罚**（复活扣上限的那个 penalty）减少 0.25。组件接口与符号语义归 [[reference/components/maxhealer#关键接口]]，本页只写物品实际接法与限制。
- 优先使用：查「一次减多少、还剩几次」看[注册与构造](#注册与构造)；查「为什么对某目标出不来治疗动作」看[与 healer 链的差异](#与-healer-链的差异)。
- 重要副作用：只要目标有 `health` 组件，使用即 `DeltaPenalty(-0.25)` 并消耗一份——惩罚已为零或目标根本没有惩罚（多数生物）**不拒绝、照样耗药**。[Heal](../../../raw/current/scripts/components/maxhealer.lua#L11-L22)
- 修改边界：不展开 health penalty 的存档与 HUD 刷新（归 [[reference/components/maxhealer]] 与 [[reference/components/health]]）；未运行游戏验证手感与多人表现。

## 注册与构造

[注册行](../../../raw/current/scripts/prefabs/lifeinjector.lua#L41)；[prefablist 登记](../../../raw/current/scripts/prefablist.lua#L669)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lifeinjector`（强心针） | `fn`：bank/build `lifepen`、`PlayAnimation("idle")`、浮水 → 主模拟挂 `stackable`(40)／`inspectable`／`inventoryitem`／`maxhealer` | `maxhealer` **不带任何设置调用**——治疗量即组件默认 `TUNING.MAX_HEALING_NORMAL`=-0.25；随后 `MakeHauntableLaunch`。[maxhealer 挂载](../../../raw/current/scripts/prefabs/lifeinjector.lua#L34)、[默认值](../../../raw/current/scripts/components/maxhealer.lua#L3)、[常量](../../../raw/current/scripts/tuning.lua#L2019)、[作祟 helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |

- 数值含义：`-0.25` 是 penalty 的**有符号增量**，经 `SetPenalty` 夹取到 0～`MAXIMUM_HEALTH_PENALTY`=0.75；「组件注释按复活次数解释」的辨析归 [[reference/components/maxhealer#恢复的是哪一种上限]]，本页不重复。[夹取](../../../raw/current/scripts/components/health.lua#L460-L465)、[上限常量](../../../raw/current/scripts/tuning.lua#L2170)
- 名称核验：简中实显「强心针」（`STRINGS.NAMES.LIFEINJECTOR`，英文 Booster Shot）；「生命注入器」是派发工作题名，仅作检索别名保留，无官方条目。[po](../../../raw/current/scripts/languages/chinese_s.po#L344705-L344707)

## 与 healer 链的差异

使用入口与治疗药膏族相同（[[reference/prefabs/healing-salves#healer-链共用规则族级主章节]]第 1 条），差异有四：

| 维度 | 强心针（maxhealer） | 治疗药膏族（healer） |
|---|---|---|
| 恢复对象 | penalty（可用上限） | 当前生命点数 |
| 执行函数分派 | `HEAL.fn` 的 `elseif` 分支：同件同挂两组件时 healer 优先 | 首选分支 |
| 自定义回调 | 无 `SetOnHealFn`／`SetCanHealFn`，不乘 `efficientuser` 倍率 | 支持，倍率有效 |
| 禁疗目标 | 候选仍要求目标 `CanHeal()`，且**没有** `healerbuffs` 例外——Wanda（`canheal=false`）与墙类目标不出治疗动作 | acid/fumarole 凭标签可对禁疗目标挂 buff |

[分派](../../../raw/current/scripts/actions.lua#L3217-L3226)、[场景候选](../../../raw/current/scripts/componentactions.lua#L1492-L1508)、[背包候选](../../../raw/current/scripts/componentactions.lua#L2803-L2806)、[CanHeal 语义](../../../raw/current/scripts/components/health_replica.lua#L192-L202)、[Wanda](../../../raw/current/scripts/prefabs/wanda.lua#L234)、[healerbuffs 对照](../../../raw/current/scripts/componentactions.lua#L1346-L1362)

## 获得与消耗

- 配方：烂食物 8＋硝石 2＋蜂刺 1，`TECH.SCIENCE_TWO`；登记在合成栏「恢复」（RESTORATION）分类。[配方](../../../raw/current/scripts/recipes.lua#L421)、[分类](../../../raw/current/scripts/recipes_filter.lua#L1144-L1160)
- 全库按注册名检索无 `SpawnPrefab` 生成点与掉落表登记（仅 debug 命令与配方／图鉴引用；检索范围同 [[reference/prefabs/healing-salves#验证记录]]）。
- 消耗侧：无其他配方或消费端引用本注册名；每次使用拆一件消耗。[消耗](../../../raw/current/scripts/components/maxhealer.lua#L15-L20)

## 常见陷阱与限制

- **给无惩罚的目标用＝白耗**：`Heal` 只判目标有无 `health`，不看当前 penalty；非玩家生物与惩罚为零的玩家照常扣针。组件级结论与门禁建议见 [[reference/components/maxhealer#常见陷阱与限制]]。
- **误区：Wanda 能用强心针自救**：她 `canheal=false`（游戏设计禁疗），maxhealer 又无 `healerbuffs` 类例外，两条路径的候选都不会给动作；只有 mod 绕过候选直接调组件才可能生效。[Wanda 禁疗](../../../raw/current/scripts/prefabs/wanda.lua#L234)
- **作祟只抛飞**：`MakeHauntableLaunch` 无打碎／点燃分支。[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。核对 `lifeinjector.lua` 全文件、`maxhealer.lua` 全文件、`health.lua` penalty 夹取段、`health_replica.lua` CanHeal、`actions.lua` HEAL 分派、`componentactions.lua` maxhealer 两处候选与 healer 候选对照、`recipes.lua` 配方、`recipes_filter.lua` 分类、`wanda.lua` 禁疗位、`standardcomponents.lua` 作祟 helper、`tuning.lua` 两个常量、`prefablist.lua` 登记。
  名称核验：简中名按 `chinese_s.po` 的 `STRINGS.NAMES.LIFEINJECTOR` msgctxt 实显核对（强心针）。
  生成点否定断言范围：全 `scripts` 目录字面检索 `"lifeinjector"`（排除本页文件、语言、配方、图鉴、登记表、分类表后无残留消费点）；动态拼名未检索。
  未覆盖：未实测读档、多人使用、`efficientuser` 无关性（代码路径已核，无调用点）。
- 游戏运行：未运行。治疗量 −0.25 与夹取范围由常量推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 待主代理验收时统一运行（见交付回报）。

## 相关链接

- 组件语义与 penalty 机制：[[reference/components/maxhealer]]、[[reference/components/health]]
- 同入口对照：[[reference/prefabs/healing-salves]]

返回 [[reference/prefabs/index|Prefab 索引]]。
