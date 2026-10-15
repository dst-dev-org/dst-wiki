---
title: "stalkerinspectable"
created: "2026-10-24"
updated: "2026-10-28"
type: "component"
summary: "19 行小组件：为暗影 NPC 的检视目标提供统一名称键（GetName），供 NPC 脑记录「已检视」；标签添加/移除当前均为注释状态。"
aliases: ["暗影检视名称", "NPC 检视目标命名", "stalkerinspectable"]
tags: ["component", "data"]
dst_build: "steam-buildid:26230496"
sources: ["components/stalkerinspectable.lua", "util.lua", "brains/stalkerbrain.lua", "prefabs/statueruins.lua", "prefabs/altar_prototyper.lua", "prefabs/daywalker.lua", "prefabs/vault_key_activator.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
size_class: "simple"
---

# stalkerinspectable

## 快速判断

- 职责与适用：挂在被暗影（Fuelweaver）NPC 检视的对象上，提供稳定名称键；供 stalkerbrain 把 `inspected_<名称>` 写入世界数据表 [[reference/components/stalkermanager]]，避免重复检视。玩家侧检视描述另见 [[reference/components/inspectable]]。[brains/stalkerbrain.lua:371](../../../raw/current/scripts/brains/stalkerbrain.lua#L371
- 优先使用：主端挂组件后 `SetNameOverride(nameoverride)`（字符串或函数）；消费方调 `GetName(viewer)` 取键名，未设置时回退 `inst.prefab`。[components/stalkerinspectable.lua:10](../../../raw/current/scripts/components/stalkerinspectable.lua#L10
- 重要副作用：无网络、无存档、无事件；组件不维护标签——构造与 `OnRemoveFromEntity` 里的 AddTag/RemoveTag 均为注释，当前快照也没有任何 `AddTag("stalkerinspectable")` 调用。[components/stalkerinspectable.lua:3](../../../raw/current/scripts/components/stalkerinspectable.lua#L3 [components/stalkerinspectable.lua:7](../../../raw/current/scripts/components/stalkerinspectable.lua#L7
- 修改边界：纯主端数据组件（宿主均为主端添加，约 20 个原版 prefab 使用）；目标发现靠 NPC 脑遍历 `components.stalkerinspectable` 字段而非标签。[brains/stalkerbrain.lua:371](../../../raw/current/scripts/brains/stalkerbrain.lua#L371

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetNameOverride(nameoverride)` | 字符串或 function(inst, viewer) | 存入 `self.nameoverride`。原版多数传固定字符串：statueruins 按变体传 `"ruins_statue_head"`/`"ruins_statue_mage"`、altar_prototyper 传 `"ancient_altar"`、vault_key_activator 传 `"vault_key"`；vault_runes/vault_statues/ancient_husks 传函数动态计算。[components/stalkerinspectable.lua:10](../../../raw/current/scripts/components/stalkerinspectable.lua#L10 [prefabs/statueruins.lua:284](../../../raw/current/scripts/prefabs/statueruins.lua#L284 [prefabs/altar_prototyper.lua:450](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L450 |
| `GetName(viewer)` | 返回 string | `FunctionOrValue(self.nameoverride, self.inst, viewer)` 或回退 `self.inst.prefab`；viewer 是执行检视的 NPC（供函数型覆写取参数）。[components/stalkerinspectable.lua:14](../../../raw/current/scripts/components/stalkerinspectable.lua#L14 [util.lua:1718](../../../raw/current/scripts/util.lua#L1718 |
| `OnRemoveFromEntity()` | — | 当前仅含注释掉的 RemoveTag，无可执行逻辑。[components/stalkerinspectable.lua:6](../../../raw/current/scripts/components/stalkerinspectable.lua#L6 |
| 标签 `stalkerinspectable` | — | 快照内不存在：仅有的两处代码均为注释，也没有其他文件添加或查询该标签；不要依赖标签发现目标。[components/stalkerinspectable.lua:3](../../../raw/current/scripts/components/stalkerinspectable.lua#L3 |

## 典型接入

### 让自定义物件成为暗影 NPC 的检视目标

- 前提与选择：目标模组 `modmain.lua` 服务端；NPC 脑的目标过滤条件是"有 stalkerinspectable 组件且 `inspected_<GetName()>` 未记录"，随后走近检视并写世界数据。[brains/stalkerbrain.lua:371](../../../raw/current/scripts/brains/stalkerbrain.lua#L371 [brains/stalkerbrain.lua:404](../../../raw/current/scripts/brains/stalkerbrain.lua#L404
- 生效与撤销：卸载模组后组件消失；已写入的 `inspected_<键>` 数据残留在世界存档中，键名变更会导致 NPC 重新检视一次。

```lua
AddPrefabPostInit("my_statue", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:AddComponent("stalkerinspectable")
    inst.components.stalkerinspectable:SetNameOverride("my_statue")
end)
```

- 调整与验证：覆写值直接拼进数据键，保持稳定、避免与其他宿主键冲突；原版宿主示例见上表锚点。

## 常见陷阱与限制

- **与玩家 inspectable 混淆**：玩家检视描述走 `inspectable.nameoverride`（DESCRIBE 查询，见 [[reference/components/inspectable]]）；本组件只服务 NPC 脑的键命名，互不通用。[brains/stalkerbrain.lua:371](../../../raw/current/scripts/brains/stalkerbrain.lua#L371
- **名称是数据键不是显示名**：`GetName` 返回值直接拼成 `inspected_<名称>` 存入 stalkermanager 数据表；运行期更换覆写值会产生新键，NPC 对同目标再检视一次。[brains/stalkerbrain.lua:373](../../../raw/current/scripts/brains/stalkerbrain.lua#L373
- **宿主可能中途移除组件**：daywalker 脱锁时 RemoveComponent、vault_key_activator 用后移除；依赖该组件存在的逻辑需自行处理移除时机。[prefabs/daywalker.lua:914](../../../raw/current/scripts/prefabs/daywalker.lua#L914 [prefabs/vault_key_activator.lua:239](../../../raw/current/scripts/prefabs/vault_key_activator.lua#L239

## 验证记录

- 源码审阅：2026-10-24，Steam buildid 26148036；覆盖 19 行组件全部逻辑与标签现状（grep 全 scripts 无标签使用），及 stalkerbrain 消费链、statueruins/altar_prototyper/vault_key_activator/daywalker 宿主调用；依据见上文锚点。未覆盖全部约 20 个宿主 prefab 的完整检视流程。
- 游戏运行：未运行。
- 结构检查：2026-10-24 执行 validate_page.py（simple 档）与 validate_source_links.py（--strict）通过；页内 Lua 示例经 parser_probe.exe --syntax-only 语法检查通过。

相关：[[reference/components/stalkermanager]]、[[reference/components/inspectable]]。
