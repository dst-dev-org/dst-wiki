---
title: "盒中泰拉召唤光柱（terrarium_fx）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "盒中泰拉夜间召唤眼魔时从本体升起的联网光柱：激活动画接循环、关闭时播退场并定时移除；纯 DECOR 表现，随宿主父挂移动。"
aliases: ["terrarium_fx", "盒中泰拉光柱", "召唤光柱", "泰拉召唤特效", "眼魔召唤光柱"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/terrarium.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 盒中泰拉召唤光柱（terrarium_fx）

## 快速判断

- 职责与适用：`prefabs/terrarium.lua` 末尾与宿主一起返回的 `terrarium_fx`，是**盒中泰拉**（`terrarium`，官方名"盒中泰拉"）夜间召唤眼魔时从本体升起的联网光柱。它只做激活／循环／退场三段动画，没有伤害、没有交互，`DECOR` 标签、`persists=false`。[注册](../../../raw/current/scripts/prefabs/terrarium.lua#L646-L647)、[工厂](../../../raw/current/scripts/prefabs/terrarium.lua#L621-L644)
- 优先使用：做"设施启动后持续光柱、关闭时播完退场再删"的表现时，可复用这套"生成即父挂＋显式动画切换＋定时 Remove"的结构；驱动入口是宿主侧的 `StartSummoning`／`TurnOff`，不是 FX 自己。[生成](../../../raw/current/scripts/prefabs/terrarium.lua#L203-L245)、[关闭](../../../raw/current/scripts/prefabs/terrarium.lua#L246-L295)
- 重要副作用：光柱贴着宿主父挂，宿主移动／转身会带着走；关闭时先播 `deactivate_fx`，10 帧后才删，期间仍可见；`TurnOff` 立即把宿主引用置空，重复关闭不会二次播动画。[退场](../../../raw/current/scripts/prefabs/terrarium.lua#L271-L274)
- 修改边界：只覆盖该注册的工厂、生成／关闭时机与清理；盒中泰拉本体的激活、交易、 Boss 召唤、夜间计时与掉落不展开。
- 名称：`STRINGS.NAMES.TERRARIUM`＝"盒中泰拉"（[po](../../../raw/current/scripts/languages/chinese_s.po#L350359-L350362)）；`terrarium_fx` 无 `STRINGS.NAMES` 键，标题中的"召唤光柱"是已核验效果描述。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `terrarium_fx` | `terrarium_fx()`：Transform／AnimState／Network → bank `terrarium`／build `terrarium`、播 `activated_idle_fx` 循环、`SetBloomEffectHandle("shaders/anim.ksh")`、`FinalOffset(-1)` → `DECOR` 标签 → `SetPristine` → 非主模拟早退；主模拟 `persists=false`。 | 单一注册；无 `FX`／`NOCLICK`、无 physics／combat／health；无自删，寿命由宿主 `TurnOff` 的定时 `Remove` 决定。[工厂](../../../raw/current/scripts/prefabs/terrarium.lua#L621-L644) |

共享资产只有 `anim/terrarium.zip`（宿主与光柱同包，[assets](../../../raw/current/scripts/prefabs/terrarium.lua#L1-L7)）；光柱依赖列在宿主 `prefabs` 表里（[依赖](../../../raw/current/scripts/prefabs/terrarium.lua#L10-L15)）。

## 生成、激活与退场

宿主在 `StartSummoning` 里按需生成并父挂（[实现](../../../raw/current/scripts/prefabs/terrarium.lua#L203-L245)）：

```lua
inst._summoning_fx = SpawnPrefab("terrarium_fx")
inst._summoning_fx.entity:SetParent(inst.entity)
inst._summoning_fx.AnimState:PlayAnimation("activate_fx")
inst._summoning_fx.AnimState:PushAnimation("activated_idle_fx", true)
```

- 生成时先把宿主切成不可拾取、不可激活、不可交易；光柱是这套"施法中"状态的一部分。[L207-L215](../../../raw/current/scripts/prefabs/terrarium.lua#L207-L215)
- 宿主只在 `is_loading` 或"已开启＋夜晚＋没有光柱＋没有 `summon_delay` 计时"时进入 `StartSummoning`；白天激活只置 `is_on`、不立刻起计时；入夜后 `on_night` 才起 5 秒 `summon_delay`，到点再进入 `StartSummoning`。[条件](../../../raw/current/scripts/prefabs/terrarium.lua#L203-L206)
- 调用方还会把宿主灯光延迟 7 帧打开、播放 `beam_loop` 声、启动 `warning` 10 秒计时；这些是宿主表现，不是光柱工厂的内容。[L222-L241](../../../raw/current/scripts/prefabs/terrarium.lua#L222-L241)、[调值](../../../raw/current/scripts/tuning.lua#L6371-L6372)

关闭路径在 `TurnOff`（[实现](../../../raw/current/scripts/prefabs/terrarium.lua#L246-L295)）：

```lua
inst._summoning_fx.AnimState:PlayAnimation("deactivate_fx")
inst._summoning_fx:DoTaskInTime(DEACTIVATE_TIME, inst._summoning_fx.Remove)
inst._summoning_fx = nil
```

- `DEACTIVATE_TIME`＝10 帧；`deactivate_fx` 播完前实体仍在，10 帧后删除（[常量](../../../raw/current/scripts/prefabs/terrarium.lua#L245-L245)、[退场](../../../raw/current/scripts/prefabs/terrarium.lua#L271-L274)）。
- 宿主把 `_summoning_fx` 立即置 nil，因此连续 `TurnOff` 不会重复播退场或重复排删除；但排删除的任务挂在光柱实体上，光柱被提前删则任务随实体取消。
- `TurnOff` 还会关闭灯光、停 `beam`／`shimmer` 声、恢复可拾取与交易；这些是宿主状态，不是光柱自身。[L247-L270](../../../raw/current/scripts/prefabs/terrarium.lua#L246-L269)

### 读档与宿主状态

- `persists=false`，光柱不入存档；宿主 `OnSave` 只存 `is_on`／`is_crimson`／Boss 引用，不存光柱。
- `OnLoadPostPass`（[L486-L505](../../../raw/current/scripts/prefabs/terrarium.lua#L486-L505)）在 `is_on` 且 Boss 仍在场或 `warning` 计时存在时，调用 `TurnOn(inst, true)`＋`StartSummoning(inst, true)` 重建光柱；`is_loading` 让重建绕过"夜晚"条件。[读档](../../../raw/current/scripts/prefabs/terrarium.lua#L486-L505)
- 若 `is_on` 但没有 Boss／warning、且 `SPAWN_EYEOFTERROR` 为真，只 `TurnOn(inst, true)`，不在此处重建光柱；之后由夜间计时／激活链再触发。[分支](../../../raw/current/scripts/prefabs/terrarium.lua#L500-L505)

## 关联机制导航

| 类型／对象 | 本注册如何使用 | 变体、端与条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `Prefab：terrarium` | 宿主与唯一生成者；父挂、切动画、定时删 | 只有 `StartSummoning` 生成；`TurnOff` 退场 | [注册](../../../raw/current/scripts/prefabs/terrarium.lua#L646-L646)、[生成](../../../raw/current/scripts/prefabs/terrarium.lua#L216-L221) |
| `组件：activatable` | 激活／关闭宿主，间接触发本 FX | `OnActivate` 调 `TurnOn`／`TurnOff` | [激活](../../../raw/current/scripts/prefabs/terrarium.lua#L415-L421) |
| `计时器：summon_delay`／`warning` | 5 秒准备后生成光柱，10 秒警告后召 Boss | 只在夜间开启时起 `summon_delay` | [调值](../../../raw/current/scripts/tuning.lua#L6371-L6372)、[TimerDone](../../../raw/current/scripts/prefabs/terrarium.lua#L443-L451) |
| `Prefab：eyeofterror` | 召唤产物；光柱本身不生成它 | 由宿主 `SpawnEyeOfTerror` 生成 | [依赖](../../../raw/current/scripts/prefabs/terrarium.lua#L10-L15) |
| 关联页面：`terrariumchest_fx` | 同为"泰拉"前缀但不同注册、不同宿主；避免混淆 | 宝箱沙棺，与本页无生成关系 | [[reference/prefabs/terrariumchest_fx|显眼箱子沙棺特效]] |

## 常见陷阱与限制

- **边界：它不是 `FX` 标签实体。** 工厂只加 `DECOR`，没有 `FX`／`NOCLICK`；按 FX 标签筛选不会命中它。[标签](../../../raw/current/scripts/prefabs/terrarium.lua#L634-L634)
- **边界：光柱不自己计时消失。** 只有宿主 `TurnOff` 会播退场并排删除；宿主若被直接移除而没走 `TurnOff`，光柱是否随引擎父挂级联删除未核验，按缺口处理。[退场](../../../raw/current/scripts/prefabs/terrarium.lua#L271-L274)
- **边界：`deactivate_fx` 期间仍算"已删引用、未删实体"。** 宿主已把 `_summoning_fx` 置 nil，10 帧内再调 `TurnOff` 不会再次操作这只光柱；期间它仍显示退场动画。[置空](../../../raw/current/scripts/prefabs/terrarium.lua#L274-L274)
- **边界：白天激活不会立即出光柱。** `StartSummoning` 的非读档分支要求 `TheWorld.state.isnight`；白天激活只置 `is_on`、不启动 `summon_delay`，入夜后 `on_night` 才起 5 秒计时。[条件](../../../raw/current/scripts/prefabs/terrarium.lua#L203-L206)、[入夜](../../../raw/current/scripts/prefabs/terrarium.lua#L378-L385)
- **误区：它是"眼魔的传送门"。** 它只是盒中泰拉施法状态的光柱表现，Boss 由宿主 `SpawnEyeOfTerror` 生成（红眼状态换 `twinmanager`），光柱本身无生成或伤害逻辑。[宿主生成](../../../raw/current/scripts/prefabs/terrarium.lua#L336-L372)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/terrarium.lua` 的宿主状态与光柱段（资产 L1-L15、`TurnOn` L161-L195、`StartSummoning` L203-L245、`TurnOff` L247-L295、`OnActivate` L415-L421、`TimerDone` L443-L451、`OnLoadPostPass` L478-L505、工厂 L621-L644、注册 L646-L647）；按需读取 `tuning.lua` 的 `TERRARIUM_WARNING_TIME`／`TERRARIUM_SUMMON_DELAY`（L6371-L6372）。全域反查（`grep -rn "terrarium_fx" --include=*.lua`）：仅本文件依赖 L14、生成 L217、注册 L647，无其它生成点。台账分诊结论（1 注册、生成 terrarium.lua:217、DECOR、persists=false）复核属实；补充实核：无 `FX` 标签、`FinalOffset(-1)`、退场 10 帧、`StartSummoning` 的夜晚门控与读档重建。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.TERRARIUM`＝"盒中泰拉"（msgid "Terrarium"，[L350359-L350362](../../../raw/current/scripts/languages/chinese_s.po#L350359-L350362)）；`terrarium_fx` 无 `STRINGS.NAMES` 键。
- 分档说明：`size_class: standard`——单注册，但需承载夜间计时→生成→激活循环→关闭退场→读档重建的完整宿主链；无多状态机，未升 complex。
- 未覆盖：盒中泰拉本体交易／Boss 战斗／掉落、`eyeofterror`、动画包实际帧长、宿主直接移除时的引擎级联；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/terrariumchest_fx|显眼箱子沙棺特效]]（同为"泰拉"前缀，不同注册／宿主）
- 返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]（盒中泰拉）
