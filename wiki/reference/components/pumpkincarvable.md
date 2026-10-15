---
title: "pumpkincarvable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "巨型蔬菜南瓜雕刻组件：承接雕刻弹窗会话，校验并压缩存储雕刻数据，按 net 变量在客户端重建切割 FX 并随昼夜变化发光。"
aliases: ["南瓜雕刻", "雕刻数据", "pumpkincarvable 组件"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/pumpkincarvable.lua", "prefabs/veggies.lua", "actions.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# pumpkincarvable

## 快速判断

- 职责与适用：挂在 `plant_def.iscarvable` 的巨型蔬菜上（prefabs/veggies.lua:854、:944），打蜡后经 `TransferComponent` 转移到 waxed 实体（veggies.lua:302-303）；客户端也挂载（仅 net 同步与 FX，:318-321，componentactions.lua:1586 注释确认）。
- 优先使用：接入雕刻动作 → `CanBeginCarving`/`BeginCarving`（actions.lua:6062-6088 的 `CARVEPUMPKIN` 已调用）；读取/迁移数据 → `GetCutData`／`TransferComponent`（:383-385、:463-477）。
- 重要副作用：雕刻数据经 `net_string`（`pumpkincarvable.cutdata`，dirty 事件 `cutdatadirty`，:316）全网同步；客户端收到 dirty 即重建 FX（:279-281）。
- 修改边界：雕刻会话由状态图 `pumpkincarving` 与 `POPUPS.PUMPKINCARVING` 弹窗承载（:407、:330-334）；换弹窗/状态图须同步改 `onclosepopup` 过滤条件。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CanBeginCarving` | `(doer) -> ok, reason\|nil` | doer busy/已在雕返回 false；燃烧→`"BURNING"`，被占用→`"INUSE"`。:387-396 |
| `BeginCarving` | `(doer) -> boolean` | 仅主机；记 `carver`、监听 doer 的 onremove 与 `ms_closepopup`、进 `pumpkincarving` 状态、`StartUpdatingComponent`。:398-417 |
| `EndCarving` | `(doer) -> nil` | 清监听与 carver，`PushEventImmediate("ms_endpumpkincarving")`，停更新。:419-436 |
| `DoRefreshCutData` | `() -> boolean` | 解码 `cutdata` 并重建 FX（symbol `follow_cut`，:438-440）；FX 生成见 `ApplyCuts`（:225-243）。 |
| `GetCutData` | `() -> string` | 当前压缩编码串（ZipAndEncodeString 格式）。:383-385 |
| `LoadCutData` | `(cutdata) -> nil` | 写 net 值并立即刷新；trophyscale_oversizedveggies.lua:89-90 战利品秤迁移用。:442-450 |
| `OnSave`/`OnLoad` | 常规存取 | 存取 `{ cuts = <编码串> }`。:452-461 |
| 模块级导出 | `AddCutData`/`ApplyCuts`/`SHAPE_NAMES`/`SHAPE_IDS`/`TOOL_SHAPES`/`NIGHT_LIGHT_OVERRIDE` | 供同类组件复用（pumpkinhatcarvable、ugc_swap_fx.lua:1 直接 require 本模块）。:357-362 |

## 关键流程：数据与校验

- 数据格式：每刀 4 个数 `{shape_id, rot, x, y}`（`AddCutData`，:172-178）；9 种形状 arc/circle/crescent/diamond/heart/hexagon/square/star/triangle（:3-15）。越界丢弃：`_IsOnPumpkin` 按 x∈(-50,40)、圆心距 ≤65+1 判定（:180-197，注释要求与 pumpkincarvingscreen.lua 保持同步）。
- 服务端校验 `_ValidateCutData`（:245-275）：解码后按 doer 背包 `inventory:Has("pumpkincarver1/2/3",1,true)` 收集可用形状，只保留持有工具对应的形状，总量截到 `TUNING.HALLOWEEN_PUMPKINCARVER_MAX_CUTS * 4`（=60 个数，tuning.lua:8536 定义 15 刀），再 `ZipAndEncodeString` 回写。
- 会话关闭：`onclosepumpkin`（:335-349）做上述校验、set net 值、非 dedicated 立即刷新、有刀时生成 `pumpkincarving_shatter_fx`，最后 `EndCarving`。
- 自动断开：`OnUpdate`（:483-489）在雕刻者离开 `self.range`（默认 3，:326）或看不见目标时 `EndCarving`。

## 典型接入

### 让巨型蔬菜支持雕刻（原版路径复现）

- 前提与选择：prefab 生成时 `AddComponent("pumpkincarvable")` 并初始化 `inst.highlightchildren = {}`（veggies.lua:853-856）；客户端实体同样需要，否则动作不可选。
- 生效与撤销：动作入口 `ACTIONS.CARVEPUMPKIN`（actions.lua:6060-6091）先 `CanBeginCarving`，黑暗中静默失败，被持有时先 `DropItem` 再 `BeginCarving`；会话由弹窗关闭自然收尾，或超距/离视野自动断开。
- 原版依据：veggies.lua:854/:944、actions.lua:6062-6088。

```lua
local function MakeCarvable(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst:AddComponent("pumpkincarvable")
    inst.highlightchildren = {}
end
```

- 调整与验证：`range`（:326）控制断开距离；雕刻结果亮度随昼夜切换 `NIGHT_LIGHT_OVERRIDE`=0.12（:39、:307-309）；装备态在持有者身上显示 `pumpkincarving_swap_fx`（:283-297）。

## 常见陷阱与限制

- FX 是非网络实体：`CreateCut` 生成的切割 FX `persists = false`（:133-135），仅靠 net 串在端上重建；直接改实体存档不会带上 FX，只有 `cutdata` 会存（:452-455）。
- 客户端也有组件：客户端分支只监听 `cutdatadirty`（:318-321），`BeginCarving` 等在客户端直接 return（:399-400）；勿在客户端调会话接口。
- 校验按背包工具裁剪：没有雕刻刀时提交任何数据都会归零（`_ValidateCutData` 返回 `""`，:274），换刀后旧刀形状会被过滤。
- 坐标系 y 轴与雕刻屏相反：`_IsOnPumpkin` 注释明确 y 取反（:189），自定 UI 传坐标时需换算。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 491 行；挂载与迁移 veggies.lua:302-303/:854/:944、动作 actions.lua:6060-6091、上限 tuning.lua:8536、秤迁移 trophyscale_oversizedveggies.lua:89-90 逐行核对。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/pumpkinhatcarvable]]
- [[reference/components/pumpkincarver]]
