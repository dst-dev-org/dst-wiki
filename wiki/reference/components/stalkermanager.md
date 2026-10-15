---
title: "stalkermanager"
created: "2026-10-24"
updated: "2026-10-28"
type: "component"
summary: "世界实体上的服务端组件：为暗影（Fuelweaver）任务线保存跨实例的键值数据表，并登记当前唯一存活的 stalker。"
aliases: ["暗影任务线世界数据", "Fuelweaver 数据跨实例保存", "stalkermanager"]
tags: ["component", "data", "lifecycle"]
dst_build: "steam-buildid:26230496"
sources: ["components/stalkermanager.lua", "prefabs/stalker.lua", "prefabs/world.lua", "brains/stalkerbrain.lua", "prefabs/fossil_mound.lua", "debugcommands.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
size_class: "simple"
---

# stalkermanager

## 快速判断

- 职责与适用：挂在世界实体上的数据表＋登记器（源码注释：森林与洞穴世界都有）；任务线数据在 stalker 实例反复生成/移除间保留，登记器记录当前唯一的 stalker。[components/stalkermanager.lua:5](../../../raw/current/scripts/components/stalkermanager.lua#L5 [components/stalkermanager.lua:11](../../../raw/current/scripts/components/stalkermanager.lua#L11
- 优先使用：读写任务线状态用 `GetData`/`SetData`；查询当前 stalker 用 `GetStalker`/`StalkerExists`；登记由 stalker 生成时推送世界事件 `ms_registerstalker` 完成，调用方不手动登记。[components/stalkermanager.lua:51](../../../raw/current/scripts/components/stalkermanager.lua#L51 [components/stalkermanager.lua:76](../../../raw/current/scripts/components/stalkermanager.lua#L76
- 重要副作用：`OnSave` 仅在 `_data` 非空时导出键 `stalkerdata`；`_stalker` 实体引用不入档，读档后靠新 stalker 再次推送事件重建登记。[components/stalkermanager.lua:82](../../../raw/current/scripts/components/stalkermanager.lua#L82
- 修改边界：仅服务端（构造即 assert ismastersim），客户端世界无此组件；`Debug_ResetData` 只被控制台命令 `d_resetfuelweavernpc` 调用。[components/stalkermanager.lua:11](../../../raw/current/scripts/components/stalkermanager.lua#L11 [debugcommands.lua:5222](../../../raw/current/scripts/debugcommands.lua#L5222

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `GetData(var)`／`SetData(var, val)` | var 为字符串键，val 任意 | 读写 `_data` 表；stalker prefab 暴露 `inst.SetNPCData`/`GetNPCData` 包装，NPC 脑用 `"seen_biome_<id>"`、`"inspected_<名称>"` 键记录进度。[prefabs/stalker.lua:1719](../../../raw/current/scripts/prefabs/stalker.lua#L1719 [brains/stalkerbrain.lua:337](../../../raw/current/scripts/brains/stalkerbrain.lua#L337 |
| `GetStalker()` | 实体或 nil | 返回本次会话登记的 stalker；该实体 onremove 后自动置 nil。[components/stalkermanager.lua:59](../../../raw/current/scripts/components/stalkermanager.lua#L59 |
| `StalkerExists()` | boolean | 化石堆交易检查用它阻止已存在 stalker 时用暗影心脏复活。[prefabs/fossil_mound.lua:33](../../../raw/current/scripts/prefabs/fossil_mound.lua#L33 |
| `Debug_ResetData()` | — | 清空 `_data`，不触发存档流程。[components/stalkermanager.lua:67](../../../raw/current/scripts/components/stalkermanager.lua#L67 |
| `ms_registerstalker`（世界事件） | data 为 stalker 实体 | stalker 主端构造末尾推送；组件记录实体并对其监听 `onremove` 自动移除登记。[prefabs/stalker.lua:1508](../../../raw/current/scripts/prefabs/stalker.lua#L1508 [components/stalkermanager.lua:37](../../../raw/current/scripts/components/stalkermanager.lua#L37 |
| `OnSave`／`OnLoad` | 存档键 `stalkerdata` | 仅导出非空 `_data`；`OnLoad` 用存档表整体替换 `_data`。[components/stalkermanager.lua:82](../../../raw/current/scripts/components/stalkermanager.lua#L82 |

## 典型接入

### 让模组数据随暗影任务线一起跨实例、跨存档保留

- 前提与选择：目标模组 `modmain.lua` 服务端；用 `AddPrefabPostInit("world", ...)` 挂钩——world.lua 以 `SetPrefabName("world")` 统一 postinit 入口，森林与洞穴世界都命中。[prefabs/world.lua:555](../../../raw/current/scripts/prefabs/world.lua#L555
- 生效与撤销：键值写入 `_data` 后随世界存档保存并在读档恢复；卸载模组后旧档中的 `stalkerdata` 仍在，读取方需容错未知键。

```lua
AddPrefabPostInit("world", function(inst)
    if GLOBAL.TheWorld.ismastersim and inst.components.stalkermanager ~= nil then
        if inst.components.stalkermanager:GetData("mymod_flag") == nil then
            inst.components.stalkermanager:SetData("mymod_flag", true)
        end
    end
end)
```

- 调整与验证：键名建议自带命名空间避免与原版键冲突；原版现存键见 NPC 脑调用。[brains/stalkerbrain.lua:337](../../../raw/current/scripts/brains/stalkerbrain.lua#L337

## 常见陷阱与限制

- **登记不入档**：`OnSave` 只保存 `stalkerdata`；读档后 `GetStalker()` 为 nil，直到任务逻辑再次生成 stalker 并推送 `ms_registerstalker`。长期持有返回值的代码需用 `StalkerExists()` 复查。[components/stalkermanager.lua:82](../../../raw/current/scripts/components/stalkermanager.lua#L82
- **重复登记只覆盖不清监听**：dev 分支的防重复 assert 已注释；第二次登记直接覆盖 `_stalker`，前一个实体的 `onremove` 监听仍挂着（触发时把 `_stalker` 置 nil）。[components/stalkermanager.lua:37](../../../raw/current/scripts/components/stalkermanager.lua#L37
- **客户端不可用**：构造 assert ismastersim；组件在 world.lua 主端分支添加，客户端读到 nil。[components/stalkermanager.lua:11](../../../raw/current/scripts/components/stalkermanager.lua#L11 [prefabs/world.lua:596](../../../raw/current/scripts/prefabs/world.lua#L596

## 验证记录

- 源码审阅：2026-10-24，Steam buildid 26148036；覆盖组件全部公开方法、事件登记与存读档，及 stalker.lua 包装（`SetNPCData`/`GetNPCData`）、stalkerbrain 键用法、fossil_mound 交易检查与 debug 命令入口；依据见上文锚点。未覆盖 stalker 各 master_postinit 变体的完整行为（超出本页接口范围）。
- 游戏运行：未运行。
- 结构检查：2026-10-24 执行 validate_page.py（simple 档）与 validate_source_links.py（--strict）通过；页内 Lua 示例经 parser_probe.exe --syntax-only 语法检查通过。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/stalkerinspectable]]（NPC 检视名，其 `GetName` 结果拼入本组件数据键）。
