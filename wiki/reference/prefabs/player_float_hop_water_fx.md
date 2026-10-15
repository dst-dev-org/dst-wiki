---
title: "玩家出水水环（player_float_hop_water_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "玩家从水里跳上船与从温泉池跳出时的贴身水环动画：SG 状态生成联网 FX 挂在玩家身上、按玩家动画对帧，数帧后解绑留在原地，播完自删。"
aliases: ["player_float_hop_water_fx", "player_hotspring_water_fx", "player_float_fx", "出水水环", "跳船水花", "温泉出水"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/player_float_fx.lua", "stategraphs/SGwilson.lua", "stategraphs/SGhermitcrab.lua", "stategraphs/SGwx78_possessedbody.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 玩家出水水环（player_float_hop_water_fx）

## 快速判断

- 职责与适用：玩家出水动作的**贴身水环动画实体**：联网 FX，bank 统一用 `wilson`（玩家动作库）＋专属 build，FinalOffset -1 压在角色身后；两个注册名分别服务跳船上船与温泉跳出。[工厂](../../../raw/current/scripts/prefabs/player_float_fx.lua#L16-L47)
- 优先使用：查"玩家落水/出水脚下的一圈水"选本页；入口为各 SG 的 `float_hop_pre`／`soakin_jumpout` 状态。
- 重要副作用：动画单参播放（未显式传循环参数），animover 自删；SG 打断时若仍父子绑定则手动 Remove 兜底。[自删](../../../raw/current/scripts/prefabs/player_float_fx.lua#L43-L44)
- 修改边界：实体联网但无净变量，位置/朝向全由生成方驱动；生成方需自行调 MakeFacingDirty 与 SetTime 对帧。[生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26765-L26783)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `player_float_hop_water_fx` | MakeFx 共同工厂：CreateEntity → Transform＋AnimState＋Network → SetBank("wilson")＋SetBuild＋播 anim（单参）→ SetFinalOffset(-1) → facings=6 → FX/NOCLICK 标签 → SetPristine → 主模拟侧 persists=false＋animover Remove | build `player_boat_sink`＋bankfile `player_float`（两包都声明）、anim `float_water_pst`、六向；跳船上船用。[注册](../../../raw/current/scripts/prefabs/player_float_fx.lua#L52) |
| `player_hotspring_water_fx` | 同工厂，facings=4 | build `player_hotspring`、无 bankfile（单包）、anim `hotspring_water_pst`、四向；温泉跳出用。[注册](../../../raw/current/scripts/prefabs/player_float_fx.lua#L53) |

两注册名均无 NAMES 条目；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：SGwilson `float_hop_pre` | onenter 生成 hop_water 并 `SetParent(玩家)`＋MakeFacingDirty；第 1 帧 `SetTime(玩家动画时间)` 对帧；第 6 帧解绑、定位到玩家脚下并按玩家转向 | 跳船上船（embarker 起跳流程）；打断时 onexit Remove 仍绑定的 fx | [生成与对帧](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26765-L26783)、[解绑](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26785-L26799)、[onexit](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26827-L26831) |
| 生成者：SGwilson `soakin_jumpout` | 同"生成→父子→第 1 帧对时→第 5 帧解绑定位转向"模板，换 hotspring 变体 | 玩家跳出温泉池（bathing pool） | [生成与解绑](../../../raw/current/scripts/stategraphs/SGwilson.lua#L27418-L27453) |
| 生成者：SGhermitcrab / SGwx78_possessedbody | hermitcrab 与 WX-78 备用躯体的 `soakin_jumpout` 状态复用同一"生成→对时→解绑"模板 | hermitcrab（[生成](../../../raw/current/scripts/stategraphs/SGhermitcrab.lua#L5079-L5098)）；WX-78 备用躯体（[生成](../../../raw/current/scripts/stategraphs/SGwx78_possessedbody.lua#L3865-L3884)） | 两状态结构与 SGwilson 一致 |
| 关联组件：embarker | float_hop_pre 的起跳位置与移动来自 embarker 组件 | 跳船流程本体见组件页 | [[reference/components/embarker]] |
| 资源 | anim/player_boat_sink.zip＋anim/player_float.zip（hop）；anim/player_hotspring.zip（hotspring） | bank 名为 wilson；本工厂通过上述 build／bankfile 动画包声明资源，不能从 bank 名推断缺失依赖 | [资产与注册](../../../raw/current/scripts/prefabs/player_float_fx.lua#L7-L14)、[L52-L53](../../../raw/current/scripts/prefabs/player_float_fx.lua#L52-L53) |

## 典型接入

### 复用"跟随数帧后解绑留在原地"的水花模式

- 前提与选择：适合角色动作带动、随后要留在动作落点的表现；核心是 SpawnPrefab → SetParent → 延迟解绑三步，帧数由生成方状态时间线控制。
- 生效与撤销：绑定期间随玩家移动，解绑后固定；播完 animover 自删，打断路径需要生成方自己兜底 Remove。
- 原版依据：两个注册名的四处生成点全部采用同一模板（生成→第 1 帧 SetTime→第 5/6 帧解绑）。

```lua
local fx = SpawnPrefab("player_float_hop_water_fx")
fx.entity:SetParent(inst.entity)
fx.AnimState:MakeFacingDirty()
-- 第 1 帧：fx.AnimState:SetTime(inst.AnimState:GetCurrentAnimationTime())
-- 需要留在原地时：
fx.entity:SetParent(nil)
fx.Transform:SetPosition(x, y, z)
fx.Transform:SetRotation(inst.Transform:GetRotation())
```

- 调整与验证：对帧与解绑时机决定视觉是否同步；跳过 SetTime 会从动画第 0 帧播起。验证入口为 SGwilson 两状态时间线。

## 常见陷阱与限制

- **误区：它会一直跟着玩家**：生成方在 5/6 帧内解绑并留在原地，之后只是原地播完自删的独立实体。[解绑](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26794-L26799)
- **边界：六向/四向是按使用场景分配的**：hop 系配六向（配合状态里玩家 SetSixFaced [L26757](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26769)），hotspring 系四向；换用前核对自己实体的 facings，否则侧面朝向错位。
- **边界：bank 是 "wilson" 不是 build 名**：动画在玩家动作库里，build 只提供外观符号；自备动画要放进 wilson 兼容 bank。[工厂](../../../raw/current/scripts/prefabs/player_float_fx.lua#L23-L25)
- **边界：动画是否重复未核验**：`PlayAnimation(anim)` 单参播放、未显式传循环参数；pst 类动画以播完自删为设计预期。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；54 行工厂全文、SGwilson float_hop_pre 与 soakin_jumpout 全程、SGhermitcrab/SGwx78_possessedbody 生成段、NAMES 检索（无命中）已核验；动画包内容、客户端预测下的对帧表现、晚加入时序未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
