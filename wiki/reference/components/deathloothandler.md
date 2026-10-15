---
title: "deathloothandler"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "记录生物死亡时的战利品等级与已生成未掉落的战利品表，让尸体化生物读档后还能吐出正确的战利品。"
aliases: ["死亡战利品记录", "尸体战利品", "deathloothandler"]
tags: [component, loot, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/deathloothandler.lua", "entityscript.lua", "componentutil.lua", "prefabs/deerclops.lua", "stategraphs/SGbeeguard.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# deathloothandler

## 快速判断

- 职责与适用：源码头注释"Just handle a value, doesn't actually drop the loot here"（[L1-L2](../../../raw/current/scripts/components/deathloothandler.lua#L1-L2)）：只存 `level` 与 `loot` 表，供"死亡／尸体化状态"在存读档之间保留战利品信息。正规入口是 EntityScript 的四个辅助方法，不需要直接 AddComponent（自动添加，[entityscript.lua#L2165-L2172](../../../raw/current/scripts/entityscript.lua#L2165-L2172)）。
- 优先使用：`SetDeathLootLevel(n)` 写等级；尸体化时 `DropDeathLoot()` 改为把 `lootdropper:GenerateLoot()` 存进组件；取尸体战利品用 `GetDeathLoot()`。
- 重要副作用：`OnSave` 只在 `level > 0` 时返回数据，并带 `add_component_if_missing = true` 标记，读档装配时引擎侧据此自动补挂组件（[存](../../../raw/current/scripts/components/deathloothandler.lua#L29-L31)、[装配](../../../raw/current/scripts/entityscript.lua#L1979-L1985)）。
- 修改边界：不掉落、不生成战利品；没有 level 时组件可完全缺省（EntityScript 读法均带 nil 防御）。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetLevel(num)`／`GetLevel()` | number，默认 0 | 死亡战利品等级；原版用 1 表示"处于死亡战利品状态"。[L21-L27](../../../raw/current/scripts/components/deathloothandler.lua#L21-L27)、[调用例](../../../raw/current/scripts/prefabs/deerclops.lua#L190) |
| `StoreLoot(prefabs)` | 无返回 | 把 prefab 名数组逐个插进 `loot` 表。[L11-L15](../../../raw/current/scripts/components/deathloothandler.lua#L11-L15) |
| `GetLoot()` | prefab 名数组 | 返回内部表引用（非拷贝）。[L17-L19](../../../raw/current/scripts/components/deathloothandler.lua#L17-L19) |
| `OnSave`／`OnLoad` | 表或 nil | level > 0 才存；`loot` 与 level 一起恢复。[L29-L38](../../../raw/current/scripts/components/deathloothandler.lua#L29-L38) |

## EntityScript 辅助方法

| 方法 | 行为与证据 |
|---|---|
| `SetDeathLootLevel(num)` | 组件缺失时自动 AddComponent 再设等级（[L2165-L2172](../../../raw/current/scripts/entityscript.lua#L2165-L2172)） |
| `GetDeathLootLevel()` | 无组件返回 0（[L2174-L2178](../../../raw/current/scripts/entityscript.lua#L2174-L2178)） |
| `DropDeathLoot()` | 先 `SetDeathLootLevel(1)`；若 `health.is_corpsing` 则 `StoreLoot(lootdropper:GenerateLoot())`，否则照常 `DropLoot`（[L2181-L2192](../../../raw/current/scripts/entityscript.lua#L2181-L2192)） |
| `GetDeathLoot()` | 无组件返回 nil（[L2194-L2198](../../../raw/current/scripts/entityscript.lua#L2194-L2198)） |

## 典型接入

### 原版尸体化消费链

- 生物死亡状态：BOSS 与部分生物的死亡 stategraph／死亡回调改调 `DropDeathLoot()`（如 SGbeeguard [L257](../../../raw/current/scripts/stategraphs/SGbeeguard.lua#L257)、SGeyeofterror、SGmosquito、SGrocky、SGmole、SGcharlie_boss、SGsmallbird、SGtallbird、SGbeequeen 等）；`warg`／`bearger`／`deerclops` 用 `SetDeathLootLevel` 标记复活战利品等级（[deerclops.lua#L190](../../../raw/current/scripts/prefabs/deerclops.lua#L190)）。
- 尸体侧：`componentutil` 从尸体记录取 `corpse.corpse_loot = inst:GetDeathLoot()`（[componentutil.lua#L1579](../../../raw/current/scripts/componentutil.lua#L1579)），尸体被摧毁／复活时再按表掉落。

## 常见陷阱与限制

- `GetLoot()` 返回引用：调用方直接 `table.insert`／清空会改到组件内部状态。
- `OnSave` 数据在 level 归 0 后整体消失：想清状态就 `SetLevel(0)`，不要只清 `loot` 表。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/deathloothandler.lua（40 行）与 entityscript.lua 的四个辅助方法及 add_component_if_missing 装配段，grep 核对全部 DropDeathLoot／SetDeathLootLevel／GetDeathLoot 调用点，抽查 SGbeeguard、deerclops、componentutil 上下文。未覆盖尸体实体本身的存读细节。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；60 行、4951 字节，simple 档内）；validate_source_links.py --strict 通过（16 处源码引用，0 error／0 warning）；本页无完整 Lua 代码块。
