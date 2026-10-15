---
title: "snowmandecor"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "雪人装饰材料标记：让物品对 heavy 雪人（snowmandecoratable 宿主）生成 DECORATESNOWMAN，原版约 30 种材料挂接。"
aliases: ["雪人装饰材料", "装饰雪人的物品", "snowmandecor"]
tags: [component, item, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/snowmandecor.lua", "componentactions.lua", "prefabs/ash.lua", "prefabs/snowman.lua", "debugcommands.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# snowmandecor

## 快速判断

- 职责与适用：空组件，仅 `self.inst`，源码注明动作注册在 componentactions；作用对象是装饰雪人的小材料。USEITEM 候选：目标有 snowmandecoratable 组件、带 `heavy` 标签、无 `waxedplant` 标签，且玩家未搬运重物。[组件](../../../raw/current/scripts/components/snowmandecor.lua#L1-L7)、[候选](../../../raw/current/scripts/componentactions.lua#L1745-L1753)
- 优先使用：装饰物 prefab 主机侧加组件；原版材料面很宽——ash、cutgrass、twigs、flint、charcoal、goldnugget、gears、petals、meats、seeds、mushrooms、feathers 等（模式同一，[ash 宿主](../../../raw/current/scripts/prefabs/ash.lua#L102)）。
- 重要副作用：组件本体无逻辑；调试生成面板把"有 snowmandecor 组件"列为材料信息项（[调试](../../../raw/current/scripts/debugcommands.lua#L3373-L3374)）。
- 修改边界：组件加在 mastersim 判定之后，仅主机持有；候选经 actionreplica 组件 id 列表在客户端生成，与 [[reference/components/vasedecoration]] 同一机制（[注册](../../../raw/current/scripts/componentactions.lua#L3182-L3190)）。

## 典型接入

### 让自定义物品成为雪人装饰材料

- 前提与选择：物品主机侧加组件即可，无标签要求；目标侧 snowmandecoratable 是另一个组件（雪人在 pristine 前添加，客户端存在，[目标](../../../raw/current/scripts/prefabs/snowman.lua#L648-L649)）。
- 生效与撤销：手持材料右键雪人出现 DECORATESNOWMAN；装饰位置与层数由 snowmandecoratable 侧管理，本组件不参与。

```lua
AddPrefabPostInit("my_trinket", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("snowmandecor")
end)
```

- 调整与验证：driedplants 曾挂接后注释停用（[注释](../../../raw/current/scripts/prefabs/driedplants.lua#L49)），说明挂接只加一行组件、可随时收回。

## 常见陷阱与限制

- **可抛雪球被 heavy 条件排除**：snowball_item 也挂 snowmandecoratable，但不带 `heavy` 标签，走的是另一条 snowmandecoratable 候选（堆叠小雪球），与材料装饰路径分离（[候选注释](../../../raw/current/scripts/componentactions.lua#L1746-L1748)）。
- **装饰不认材料种类**：候选与执行都不检查材料 prefab，任何带本组件的物品都能当装饰；差异表现由 snowmandecoratable 与美术层决定。
- **waxedplant 目标不可装饰**：目标带 `waxedplant` 标签时两条候选都跳过（[候选](../../../raw/current/scripts/componentactions.lua#L1748)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、USEITEM 候选与 snowmandecoratable 候选边界、ash 宿主形态、snowman 目标挂接、调试面板引用及 driedplants 注释停用例。未覆盖：全部约 30 个宿主逐文件审阅、snowmandecoratable 内部、运行验证。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
