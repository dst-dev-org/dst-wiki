---
title: "法术精通 buff 与法球（spellmasterybuff 族）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "熔炉事件法术精通的三件表现：buff 锚点壳、环绕法球载体与单颗法球；buff／orbs 的主模拟初始化都在缺失的 lavaarena 事件包，快照内只能核验可见工厂与缺口。"
aliases: ["spellmasterybuff", "spellmasteryorb", "spellmasteryorbs", "法术精通", "法术精通法球", "熔炉法术精通特效"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/spellmasterybuff.lua", "entityscript.lua", "prefabs/wickerbottom.lua", "prefablist.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 法术精通 buff 与法球（spellmasterybuff 族）

## 快速判断

- 职责与适用：`prefabs/spellmasterybuff.lua` 一次返回 3 个注册：`spellmasterybuff`（buff 锚点壳，`anchor` 动画）、`spellmasteryorbs`（环绕法球的载体，带 `Follower`）、`spellmasteryorb`（单颗法球，`in` 接 `idle`）。三者都是 `FX` 表现，没有可见的伤害或玩法组件。[注册](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L101-L103)
- 优先使用：查"熔炉法术精通"的表现入口时用本页；但 `spellmasterybuff` 与 `spellmasteryorbs` 的**主模拟初始化在缺失的 lavaarena 事件包**，快照内只能确认可见工厂，不能确认 buff 数值、法球数量、跟随目标或清理。[缺口](#事件包缺口)
- 重要副作用：`spellmasterybuff`／`spellmasteryorbs` 在主模拟调用 `event_server_data("lavaarena", ...)` 的 `buff_postinit`／`orbs_postinit`；本快照没有 `lavaarena_event_server` 目录，这两个调用没有可用的模块返回值，主模拟构造无法完成。`spellmasteryorb` 不依赖事件包，自身只播动画并 `persists=false`。[buff 工厂](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L18-L43)、[orbs 工厂](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L79-L97)、[orb 工厂](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L47-L75)
- 修改边界：只覆盖 3 个注册的可见工厂、事件包依赖与快照缺口；熔炉事件玩法、Wickerbottom 技能树、buff 数值与法球行为不展开。
- 名称：3 个注册名在简中 `.po` 均无 `STRINGS.NAMES` 键；标题与别名中的"法术精通"是已核验效果描述，不是官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `spellmasterybuff` | `fn()`：Transform／AnimState／Network → `FX`＋`NOCLICK` → bank/build `lavaarena_arcane_orb`、播 `anchor`、`SetLightOverride(1)`、bloom → `SetPristine` → 非主模拟早退；主模拟调 `event_server_data("lavaarena", "prefabs/spellmasterybuff").buff_postinit(inst)`。 | 可见工厂只到 `anchor` 动画；`persists` 与后续组件都可能在缺失的 `buff_postinit` 里，不能从可见段推断。[工厂](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L18-L43) |
| `spellmasteryorbs` | `orbsfn()`：Transform／Follower／Network → `FX` → `SetPristine` → 非主模拟早退；主模拟调同路径的 `orbs_postinit(inst)`。 | 无 AnimState、无可见子实体；`Follower` 说明它设计上要跟随某物，但跟随目标与法球生成在缺失的 postinit。[工厂](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L79-L97) |
| `spellmasteryorb` | `orbfn()`：Transform／AnimState／SoundEmitter／Network → bank/build `lavaarena_arcane_orb`、播 `in` 再 `PushAnimation("idle")`、八向 → `FX`＋`NOCLICK` → `SetPristine`；主模拟 `persists=false`。 | 三件中唯一不依赖事件包的可见注册；但快照内没有它的生成语句，无法确认由谁、以何节奏生成。[工厂](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L47-L75) |

依赖关系：`spellmasterybuff` 的 `prefabs` 列 `spellmasteryorbs`，`spellmasteryorbs` 的 `prefabs` 列 `spellmasteryorb`；这只是加载依赖，不证明实际生成。[依赖](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L6-L14)

## 事件包缺口

`event_server_data(eventname, path)` 的实现是加载 `eventname.."_event_server/"..path`：

```lua
local fullpath = eventname.."_event_server/"..path
EventServerFiles[fullpath] = requireeventfile(fullpath)
```

（[实现](../../../raw/current/scripts/entityscript.lua#L77-L87)）

- 本快照 `raw/current/scripts` 下没有 `lavaarena_event_server` 目录（也没有任何 `*_event_server` 目录），因此 `buff_postinit`／`orbs_postinit` 在快照内不可读。[快照检查](#验证记录)
- 两个注册的主模拟段直接索引该返回值并调用；在缺失模块时没有可调用的函数，构造会在这一步失败。客户端的 `SetPristine` 早退发生在调用之前，所以客户端只看到可见动画壳。[buff 调用](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L40-L40)、[orbs 调用](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L94-L94)
- `spellmasteryorb` 的工厂不调用事件包，是唯一可以独立读完的注册；但它没有可见生成点，仍不能据此判断整套表现如何被使用。[orb 工厂](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L47-L75)
- `prefabs/wickerbottom.lua` 只把 `spellmasterybuff` 列进依赖表（[依赖](../../../raw/current/scripts/prefabs/wickerbottom.lua#L14-L14)），不是生成语句；全快照反查也没有 `SpawnPrefab("spellmastery...")`。

## 关联机制导航

| 类型／对象 | 本注册如何使用 | 变体、端与条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `事件包：lavaarena_event_server` | `buff_postinit`／`orbs_postinit` 的实际实现位置 | 本快照缺失；不可读 | [加载器](../../../raw/current/scripts/entityscript.lua#L77-L87) |
| `Prefab：wickerbottom` | 依赖表声明 `spellmasterybuff`；不是生成点 | 仅加载依赖 | [依赖](../../../raw/current/scripts/prefabs/wickerbottom.lua#L14-L14) |
| `Prefab：spellmasteryorb` | 单颗法球视觉；由 `spellmasteryorbs` 的依赖表引入 | 无可见生成点 | [注册](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L101-L103) |
| `资源：lavaarena_arcane_orb.zip` | 三件共用的 bank／build | 动画 `anchor`／`in`／`idle` | [资产](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L1-L4) |

## 常见陷阱与限制

- **边界：不能把"事件包缺失"写成"没有行为"。** `buff_postinit`／`orbs_postinit` 在事件包内，快照外可能定义数值、跟随、生成与清理；本页只确认快照内不可读。[缺口](#事件包缺口)
- **边界：`spellmasteryorb` 可构造不等于整套可运行。** 它自身没有事件包依赖，但 `spellmasteryorbs` 的 `orbs_postinit` 仍在缺失模块里；单独生成 orb 只能复现视觉，不能复现 buff 逻辑。[orb](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L47-L75)
- **边界：`FX`／`NOCLICK` 不能当"纯视觉"的充分证明。** 这两个标签只说明点击与筛选行为；`spellmasteryorbs` 还带 `Follower`，实际职责要等事件包可读后才能确认。[标签](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L25-L26)、[Follower](../../../raw/current/scripts/prefabs/spellmasterybuff.lua#L82-L84)
- **边界：没有官方名。** 3 个注册名都无 `STRINGS.NAMES` 键；不要从"spell mastery"直译出官方中文名。[名称](#快速判断)
- **误区：它是熔炉里可单独摆放的法球。** 快照内没有生成点，依赖表只说明加载顺序；实际生成与数量在事件包或调用方，不能靠依赖表推断。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/spellmasterybuff.lua`（103 行）；按需读取 `entityscript.lua` 的 `event_server_data`（L77-L87）、`prefabs/wickerbottom.lua` 依赖表（L10-L20）、`prefablist.lua`（文件清单）。快照检查：`raw/current/scripts` 下 `ls -d *_event_server` 无结果；全域反查 `SpawnPrefab("spellmastery` 无命中，`spellmastery` 仅本文件、`prefablist.lua:1192` 与 `wickerbottom.lua:14`。台账组 11 结论（3 注册、buff_postinit／orbs_postinit 在缺失事件包、旧 U4 误标无页）复核属实；补充实核：`spellmasteryorb` 为唯一不依赖事件包的可见注册、无可见生成点。
- 名称核验：`languages/chinese_s.po` 定向检索 `SPELLMASTERY` 无 `STRINGS.NAMES` 命中；3 个注册名均无官方名。
- 分档说明：`size_class: simple`——三注册同文件、可见工厂短，核心是事件包缺口；无多状态生命周期，未升 standard。
- 未覆盖：`lavaarena_event_server` 内的 buff／orbs 实现、熔炉事件玩法、Wickerbottom 技能树、法球数量与跟随行为；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- 返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]（角色技能／熔炉表现）
