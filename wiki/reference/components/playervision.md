---
title: "playervision"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家视觉状态组件：按装备标签与强制栈维护夜视/幽灵/梦魇等颜色立方表，经事件驱动 colourcube 切换画面滤镜。"
aliases: ["playervision", "玩家视觉", "夜视组件", "颜色立方覆盖"]
tags: [component, mechanism, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/playervision.lua", "prefabs/player_common.lua", "components/colourcube.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# playervision

## 快速判断

- 职责与适用：玩家实体组件，维护各视觉开关并计算当前颜色立方（CC）表；真正的画面切换由世界 colourcube 组件监听玩家 `ccoverrides`/`ccphasefn` 事件完成。原版在 player_common 公共段安装，主客机都有。[安装](../../../raw/current/scripts/prefabs/player_common.lua#L2493) [事件消费](../../../raw/current/scripts/components/colourcube.lua#L343-L344)
- 优先使用：装备驱动的视觉只需给装备 prefab 加对应标签（`nightvision`、`goggles`、`nutrientsvision`、`scrapmonolevision`、`inspectaclesvision`、`roseglassesvision`）；脚本强制用 `ForceNightVision(force)` 或强制栈 `PushForcedNightVision`。[标签检测](../../../raw/current/scripts/components/playervision.lua#L41-L80) [强制栈](../../../raw/current/scripts/components/playervision.lua#L229-L248)
- 重要副作用：CC 表变化向玩家推 `ccoverrides` 与 `ccphasefn`；夜视环境覆盖推 `nightvisionambientoverrides`；营养视觉事件推到**世界**而非玩家。[CC 推送](../../../raw/current/scripts/components/playervision.lua#L188-L193) [世界事件](../../../raw/current/scripts/components/playervision.lua#L56-L61)
- 修改边界：无 OnSave/OnLoad——全部状态由装备、区域与调用方在运行时重算；强制栈是纯内存。[状态重算](../../../raw/current/scripts/components/playervision.lua#L41-L103)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `ForceNightVision(force)` | 设 `forcenightvision` 并在无标签夜视时刷新 CC 表、推 `nightvision` 事件。`HasNightVision()` 为标签或强制任一为真。[实现](../../../raw/current/scripts/components/playervision.lua#L212-L220) [查询](../../../raw/current/scripts/components/playervision.lua#L143-L145) |
| `PushForcedNightVision(source,priority,customcctable,blend,ambienttable?,nonightvisioncc?)` | 每来源单条目（同 source 自动先 pop），按 priority 降序排序；栈顶变化时应用其自定义 CC 表/混合/环境覆盖，并顺带 ForceNightVision(true)。[实现](../../../raw/current/scripts/components/playervision.lua#L229-L248) [原版消费者](../../../raw/current/scripts/prefabs/player_common.lua#L1942-L1946) |
| `PopForcedNightVision(source)` | 移除该来源；栈空则全部复位（含环境覆盖与自定义 CC），栈顶变化则换用新顶。[实现](../../../raw/current/scripts/components/playervision.lua#L250-L275) |
| `SetCustomCCTable(cctable,blend)` | 设置 override 层（优先级仅次于幽灵视觉）；blend=true 时夜视相位函数参与混合。[实现](../../../raw/current/scripts/components/playervision.lua#L333-L339) [优先级](../../../raw/current/scripts/components/playervision.lua#L175-L186) |
| `SetGhostVision(enabled)` / `SetNightmareVision(enabled)` | 幽灵 CC 固定 `ghost_cc.tex`；梦魇 CC 按 `nightmarephase` 相位、8 秒混合。梦魇由 `changearea`（区域带 `Nightmare` 标签或查理 Boss 竞技场）自动维护。[设置](../../../raw/current/scripts/components/playervision.lua#L196-L210) [区域判定](../../../raw/current/scripts/components/playervision.lua#L98-L103) [相位表](../../../raw/current/scripts/components/playervision.lua#L24-L39) |
| `ForceGoggleVision` / `ForceNutrientVision` / `ForceScrapMonoleVision` / `ForceInspectaclesVision` / `ForceRoseGlassesVision` | 对应强制开关，语义同 ForceNightVision（各自事件）；营养视觉只在本地玩家上推世界事件。[实现](../../../raw/current/scripts/components/playervision.lua#L288-L331) |
| `ForceNoNightVisionCC(force)` | 抑制夜视 CC 表但保留 `HasNightVision` 真值（供只想要事件不要滤镜的场景）。[实现](../../../raw/current/scripts/components/playervision.lua#L222-L227) |
| `GetCCTable()` / `GetCCPhaseFn()` / `GetNightVisionAmbientOverrides()` | 当前生效值查询。[实现](../../../raw/current/scripts/components/playervision.lua#L167-L173) [L284-L286](../../../raw/current/scripts/components/playervision.lua#L284-L286) |

### CC 表选择优先级

`UpdateCCTable` 顺序：幽灵视觉 > `overridecctable` > 夜视（标签或强制，且未被 `nonightvisioncc` 抑制）> 梦魇视觉 > 无（[实现](../../../raw/current/scripts/components/playervision.lua#L175-L194)）。夜视表按昼夜相位在 mole_vision on/off 间切换、0.25 秒混合；梦魇表监听客户端世界事件 `nightmarephasechanged`（[相位函数](../../../raw/current/scripts/components/playervision.lua#L32-L39)）。

## 典型接入

### 为角色或装备开启夜视

- 前提与选择：装备常驻视觉走标签（ equip/unequip 自动维护，客户端补 inventoryclosed 事件）；脚本短时视觉走强制栈并配对 pop。两者叠加时标签状态优先级在事件层面已去重（强制只在无标签时推事件）。[标签路径](../../../raw/current/scripts/components/playervision.lua#L41-L49)
- 生效与撤销：Force 调用即时生效；强制栈撤销必须用**同一 source 字符串**。组件无存档，重进世界后需调用方自行重设。[栈复位](../../../raw/current/scripts/components/playervision.lua#L257-L262)

```lua
-- 任意端：为玩家临时开启夜视滤镜（鼹鼠视角 CC）
inst.components.playervision:ForceNightVision(true)
-- 撤销：
inst.components.playervision:ForceNightVision(false)

-- 需要自定义滤镜与优先级时用强制栈（source 需唯一）：
inst.components.playervision:PushForcedNightVision("mymod_blackout", 1, nil, true)
```

- 调整与验证：`customcctable` 需是 `day/dusk/night/full_moon` 四键的贴图表（参照组件顶部三张内置表）；验证看 `HasNightVision()` 与画面是否切换（[内置表](../../../raw/current/scripts/components/playervision.lua#L1-L15)）。

## 常见陷阱与限制

- **以为加标签就自动有滤镜：** 标签只在本组件检测到 equip/unequip 变化时生效，装备需走正常装备槽事件；动态改标签不触发事件就不会刷新（客户端关背包时靠 inventoryclosed 兜底重算）。[检测时机](../../../raw/current/scripts/components/playervision.lua#L82-L96)
- **强制栈 source 不唯一会互相顶替：** 同 source 推送会先移除旧条目再入栈；原版占用 `"vision"`（blackout 网络变量）。[单条目约束](../../../raw/current/scripts/components/playervision.lua#L232-L237) [原版 source](../../../raw/current/scripts/prefabs/player_common.lua#L1944)
- **梦魇视觉会被区域事件覆盖：** `changearea` 每次都用“是否梦魇区”重算 SetNightmareVision，脚本手工设置的值在跨区后丢失；常驻梦魇滤镜应改用 SetCustomCCTable。[覆盖点](../../../raw/current/scripts/components/playervision.lua#L98-L103)
- **营养视觉事件目标是世界：** 消费方需监听 `TheWorld` 的 `nutrientsvision`，监听玩家实体收不到；这是与其他视觉事件不同的特例。[推送目标](../../../raw/current/scripts/components/playervision.lua#L58-L60)
- **ForceInspectacles/RoseGlasses 的守卫少一个 not：** 判式为 `not self.x ~= force`（其余为 `not self.x ~= not force`），传 nil 也会进入更新分支；行为上仍收敛为假，但会产生一次多余事件。[判式差异](../../../raw/current/scripts/components/playervision.lua#L315-L322)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 playervision.lua，核验标签检测、CC 优先级、强制栈、事件目标与 colourcube/player_common 消费点。未运行游戏；CC 资源的实际渲染效果不在脚本核验范围。

相关：[[reference/components/ambientlighting]]、[[reference/components/dsp]]。
