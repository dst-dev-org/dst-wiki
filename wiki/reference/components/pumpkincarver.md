---
title: "pumpkincarver"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "南瓜雕刻刀标记：让工具对 pumpkincarvable／pumpkinhatcarvable 目标生成 CARVEPUMPKIN；刀的 prefab 名决定可刻图形。"
aliases: ["南瓜雕刻刀", "雕刻南瓜的工具", "pumpkincarver"]
tags: [component, item, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/pumpkincarver.lua", "prefabs/pumpkincarver.lua", "componentactions.lua", "components/pumpkincarvable.lua", "scenarios/magic_pumpkin.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# pumpkincarver

## 快速判断

- 职责与适用：空组件，仅 `self.inst`，源码注明动作注册在 componentactions；作用对象是雕刻刀类工具。USEITEM 候选：目标有 pumpkincarvable 组件即生成 CARVEPUMPKIN；目标是 pumpkinhatcarvable 时还要求未装备且未漂浮。[候选](../../../raw/current/scripts/componentactions.lua#L1584-L1593)
- 优先使用：工具 prefab 主机侧加组件；可刻图形由工具 prefab 名映射——`pumpkincarver1/2/3` 各对应 3 种图形，雕刻校验按库存 `Has(tool)` 汇总支持图形（[映射](../../../raw/current/scripts/components/pumpkincarvable.lua#L17-L35)、[校验](../../../raw/current/scripts/components/pumpkincarvable.lua#L245-L258)）。
- 重要副作用：组件本体无逻辑；目标侧 pumpkincarvable／pumpkinhatcarvable "exist on clients"，客户端候选直接查 `target.components`（[注释](../../../raw/current/scripts/componentactions.lua#L1585-L1586)）。
- 修改边界：组件加在 mastersim 判定之后（[宿主](../../../raw/current/scripts/prefabs/pumpkincarver.lua#L34-L35)），仅主机持有；原版 3 件工具由 `NUM_HALLOWEEN_PUMPKINCARVERS` 循环生成（[循环](../../../raw/current/scripts/prefabs/pumpkincarver.lua#L53-L55)）。

## 典型接入

### 让自定义工具能雕刻南瓜

- 前提与选择：工具主机侧加组件；目标侧南瓜（pumpkincarvable）或南瓜帽（pumpkinhatcarvable）已有原版组件，无需另加。
- 生效与撤销：手持工具对南瓜出现 CARVEPUMPKIN，雕刻图形选择与提交走雕刻屏流程（屏幕按 `pumpkincarver1..3` 取图，[取图](../../../raw/current/scripts/screens/redux/pumpkincarvingscreen.lua#L179)）。

```lua
AddPrefabPostInit("my_carver", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("pumpkincarver")
end)
```

- 调整与验证：magic_pumpkin 场景按 `components.pumpkincarver` 识别雕刻刀触发陷阱（[场景判定](../../../raw/current/scripts/scenarios/magic_pumpkin.lua#L14-L15)），加组件即一并纳入。

## 常见陷阱与限制

- **图形映射硬编码 prefab 名**：TOOL_SHAPES 只登记 `pumpkincarver1..3`，校验与雕刻屏都按这三个名字拼串（[校验拼名](../../../raw/current/scripts/components/pumpkincarvable.lua#L250-L254)）；自定义命名的雕刻工具能生成动作，但没有对应的可刻图形路径，需模组自行扩展映射。
- **南瓜帽候选有附加条件**：目标 pumpkinhatcarvable 时要求未装备（`equippable:IsEquipped()`）且未漂浮，否则不生成候选（[条件](../../../raw/current/scripts/componentactions.lua#L1588-L1593)）。
- **组件无雕刻逻辑**：刻什么、怎么刻、成品生成都在 pumpkincarvable 侧与雕刻屏；本组件只是"这是雕刻刀"的身份标记。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、USEITEM 候选、pumpkincarver prefab 工厂、TOOL_SHAPES 映射与 _ValidateCutData 拼名逻辑、magic_pumpkin 场景判定。未覆盖：雕刻屏完整提交流程、pumpkincarvable 全文、运行验证。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
