---
title: "小惊吓的冷热寻物指示（hotcold_fx）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "小惊吓寻物任务中挂在它头发上的冷热提示：按玩家到最近失落玩具的距离缩放，越近越大；随任务开始生成、完成时移除（放弃路径不清理）。"
aliases: ["hotcold_fx", "冷热指示", "寻物冷热特效", "小惊吓头顶指示", "热冷提示"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/smallghost.lua", "stategraphs/SGsmallghost.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 小惊吓的冷热寻物指示（hotcold_fx）

## 快速判断

- 职责与适用：`prefabs/smallghost.lua` 用 `hotcold_fx_fn` 注册的 1 个联网 FX，只在**小惊吓**（`smallghost`，官方名"小惊吓"）的寻物任务期间出现：挂在它的头发符号上，按"玩家到最近失落玩具"的距离缩放，越近越大、超过 40 距离则缩到 0，充当冷热提示。[注册](../../../raw/current/scripts/prefabs/smallghost.lua#L573-L573)、[工厂](../../../raw/current/scripts/prefabs/smallghost.lua#L548-L571)
- 优先使用：做"追踪目标越近、指示越明显"的冷热提示时，可复用这套"宿主挂点＋距离→缩放"结构；它只负责表现，不判定拾取。[更新](../../../raw/current/scripts/prefabs/smallghost.lua#L48-L68)
- 重要副作用：工厂只播 `idle1` 循环，`animover` 时随机切 `idle1/2/3`；`persists=false`；没有 `FX`／`NOCLICK` 标签，也没有伤害或碰撞。父挂到小惊吓并跟随 `smallghost_hair` 符号。[动画](../../../raw/current/scripts/prefabs/smallghost.lua#L544-L571)、[挂接](../../../raw/current/scripts/prefabs/smallghost.lua#L50-L53)
- 修改边界：只覆盖该注册的工厂、挂接、距离缩放与任务结束清理；小惊吓本体 AI、任务奖励、玩具生成与"找到玩具"判定不展开。
- 名称：`STRINGS.NAMES.SMALLGHOST`＝"小惊吓"（[po](../../../raw/current/scripts/languages/chinese_s.po#L349669-L349672)）；`hotcold_fx` 无 `STRINGS.NAMES` 键，标题中的"冷热寻物指示"是已核验效果描述。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `hotcold_fx` | `hotcold_fx_fn()`：Transform／AnimState／Network → bank `trinket_echoes`／build `trinket_echoes_fx`、播 `idle1` 循环、`OnGround`、`FinalOffset(1)` → `SetPristine` → 非主模拟早退；主模拟 `persists=false`、监听 `animover` 随机切 `idle1..3`。 | 单一注册；无 `FX`／`NOCLICK`、无 physics／combat／health；工厂只播循环动画，没有自删代码。[工厂](../../../raw/current/scripts/prefabs/smallghost.lua#L548-L571)、[资产](../../../raw/current/scripts/prefabs/smallghost.lua#L8-L11) |

## 生成与距离缩放

小惊吓在任务开始／读档恢复后启动 0.25 秒周期任务，首次更新时生成 FX（[启动](../../../raw/current/scripts/prefabs/smallghost.lua#L224-L224)、[读档](../../../raw/current/scripts/prefabs/smallghost.lua#L379-L395)）：

```lua
inst._hotcold_fx = SpawnPrefab("hotcold_fx")
inst._hotcold_fx.entity:SetParent(inst.entity)
inst._hotcold_fx.entity:AddFollower():FollowSymbol(inst.GUID, "smallghost_hair", 0, 0.2, 0)
```

随后每次更新取"跟随者（玩家）或小惊吓自身"的位置，遍历 `_toys` 找最近距离（[更新](../../../raw/current/scripts/prefabs/smallghost.lua#L48-L68)）：

| 条件 | 缩放 |
|---|---|
| `closest_toy_dsq >= MAX_HUNT_HOT_DSQ` | 0（不可见） |
| 更近 | `clamp(1 − sqrt(dsq / MAX_HUNT_HOT_DSQ), 0.20, 0.90)` |

`MAX_HUNT_HOT_DSQ`＝`TUNING.GHOST_HUNT.MINIMUM_HINT_DIST²`＝40²＝1600（[调值](../../../raw/current/scripts/tuning.lua#L4756-L4778)），因此有效提示半径是 40；缩放上下限 0.20／0.90 写在同文件（[常量](../../../raw/current/scripts/prefabs/smallghost.lua#L46-L47)）。距离取的是玩家（有 leader 时）到玩具的直线距离，不是小惊吓到玩具。

## 清理路径

- **任务完成**：`check_for_quest_finished` 在所有玩具移出 `_toys` 后取消周期任务并 `inst._hotcold_fx:Remove()`（[清理](../../../raw/current/scripts/prefabs/smallghost.lua#L86-L100)）。
- **任务放弃**：`on_abandon_quest` 只解除玩家跟随并切 `quest_abandoned` 状态，**没有取消 `_hotcold_task`、没有移除 FX、也没有清 `_toys`**；该状态只播 `sad` 动画（[放弃](../../../raw/current/scripts/prefabs/smallghost.lua#L237-L243)、[状态](../../../raw/current/scripts/stategraphs/SGsmallghost.lua#L212-L224)）。因此放弃任务后指示可能继续挂在幽灵头上，直到任务完成检查或调用方另行清理；实体移除的父挂级联未核验。
- **小惊吓被移除**：`on_smallghost_removed` 只把剩余玩具 `ErodeAway` 并置 `_cancelled`，**没有显式移除 FX**（[移除处理](../../../raw/current/scripts/prefabs/smallghost.lua#L261-L268)）；引擎父挂是否级联删除未在本页核验，按缺口处理。
- **读档**：FX 不存档（`persists=false`）；若任务仍在，`OnLoad` 重建 `_toys` 并于 0.25 秒任务里重新生成。父挂与 Follower 都在重建时重新建立。

## 关联机制导航

| 类型／对象 | 本注册如何使用 | 变体、端与条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `Prefab：smallghost` | 宿主与生成者；任务期间每 0.25 秒驱动缩放 | 只有寻物任务会生成 | [注册](../../../raw/current/scripts/prefabs/smallghost.lua#L574-L574) |
| `TUNING.GHOST_HUNT.MINIMUM_HINT_DIST` | 提示半径 40，决定缩放公式的分母 | 同段还有 `MAXIMUM_HINT_DIST`＝180（本 FX 未用） | [调值](../../../raw/current/scripts/tuning.lua#L4756-L4778) |
| `Follower：FollowSymbol` | 把 FX 固定在 `smallghost_hair` 符号上 | 父挂＋跟随符号，不随动画单独偏移 | [挂接](../../../raw/current/scripts/prefabs/smallghost.lua#L52-L53) |
| 关联页面：`fx-creature-index` | 生物／Boss 类特效索引 | 本页登记在该叶 | [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]] |

## 常见陷阱与限制

- **边界：它不是"纯灯光"而是缩放提示。** 距离超出 40 时 `SetScale(0,0)`，实体仍在；不要用可见性判断任务是否存在。[公式](../../../raw/current/scripts/prefabs/smallghost.lua#L64-L67)
- **边界：工厂没有 `FX`／`NOCLICK` 标签。** 按 FX 标签筛选不会命中它；它也没有 physics，正常情况下不挡交互，但标签层面不是 FX。[工厂](../../../raw/current/scripts/prefabs/smallghost.lua#L548-L571)
- **边界：`animover` 随机切动画在循环动画下是否可达未核验。** 工厂初始 `PlayAnimation("idle1", true)` 是循环；监听器只负责后续随机化，未运行游戏确认循环动画会不会触发 `animover`。[监听](../../../raw/current/scripts/prefabs/smallghost.lua#L544-L546)
- **边界：小惊吓被移除时没有显式删 FX。** 确认的显式 `Remove()` 只在任务完成检查；放弃回调不直接清理；父挂级联删除未核验，长会话里若怀疑残留，按实际引擎行为补 `Remove()`。[完成清理](../../../raw/current/scripts/prefabs/smallghost.lua#L86-L100)、[移除处理](../../../raw/current/scripts/prefabs/smallghost.lua#L261-L268)
- **边界：放弃任务不会自动移除指示。** `on_abandon_quest` 不取消 `_hotcold_task`、不删 FX；确认的清理只有任务完成检查，实体移除路径没有显式删除、父挂级联未核验。复刻时应在放弃回调里自行 `Cancel()`＋`Remove()`。[放弃](../../../raw/current/scripts/prefabs/smallghost.lua#L237-L243)
- **误区：它是小惊吓的常驻挂件。** 只有寻物任务（`_toys` 非空）才存在；任务完成会删，但放弃路径不会。[生成条件](../../../raw/current/scripts/prefabs/smallghost.lua#L48-L57)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/smallghost.lua` 的冷热段（资产 L8-L11、常量与更新 L44-L68、任务启动 L224、完成清理 L86-L100、移除处理 L261-L268、读档 L392-L396、工厂 L544-L571、注册 L573-L574）；按需读取 `tuning.lua` 的 `GHOST_HUNT`（L4756-L4778）。全域反查（`grep -rn "hotcold_fx" --include=*.lua`）：仅本文件依赖 L16、生成 L51、缩放 L67、移除 L99-L100、注册 L573，无其它生成点。台账分诊结论（1 注册、生成 smallghost.lua:51）复核属实；补充实核：`FX`／`NOCLICK` 标签缺失、缩放半径 40、完成路径清理而**放弃路径不清理**、实体移除路径不显式删 FX。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.SMALLGHOST`＝"小惊吓"（msgid "Pipspook"，[L349669-L349672](../../../raw/current/scripts/languages/chinese_s.po#L349669-L349672)）；`hotcold_fx` 无 `STRINGS.NAMES` 键。
- 分档说明：`size_class: simple`——单注册、单公式、无多状态；清理缺口与标签异例用条目说明即可。
- 未覆盖：小惊吓任务全流程、玩偶拾取与奖励、`animover` 在循环动画下的实际触发、父挂删除的引擎级联；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- 返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]（小惊吓）
