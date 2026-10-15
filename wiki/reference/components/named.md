---
title: "named"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "为单个物品设置可同步、可保存的动态显示名，并区分固定名称与检查语。"
aliases: ["动态名称", "物品改名", "名称同步", "named"]
tags: ["component", "item", "network-sync"]
dst_build: "steam-buildid:26230496"
sources: ["components/inspectable.lua", "components/named.lua", "components/named_replica.lua", "components/stackable.lua", "modutil.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# named

## 快速判断

- 职责与适用：单个实例的动态显示名；固定 Prefab 名称一般由 `STRINGS.NAMES` 提供，检查描述另见 [[reference/components/inspectable]]。[components/named.lua:22](../../../raw/current/scripts/components/named.lua#L22
- 优先使用：服务端 `SetName(name, author?)`；重置用 `SetName(nil)`。[components/named.lua:22](../../../raw/current/scripts/components/named.lua#L22
- 重要副作用：更新 inst.name 和 named replica；有自定义名字时保存 name、nameformat 与作者网络 ID。[components/named.lua:9](../../../raw/current/scripts/components/named.lua#L9 [components/named.lua:33](../../../raw/current/scripts/components/named.lua#L33
- 修改边界：直接赋 inst.name 不经过此同步／存档链；nameformat 修改后需重新应用名字。[components/named.lua:9](../../../raw/current/scripts/components/named.lua#L9

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetName(name, author?)` | string 或 nil；author 为供网络 ID 查询的用户标识 | nil 回退默认名称，author 不是角色实体参数。[components/named.lua:22](../../../raw/current/scripts/components/named.lua#L22 |
| `nameformat` | string 或 nil；构造器未赋非 nil 值 | 用 string.format 格式化自定义名；自行保证格式和参数兼容。[components/named.lua:9](../../../raw/current/scripts/components/named.lua#L9 |
| `possiblenames`／`PickNewName()` | 非空字符串数组／随机选择 | 无数组或空数组时不改名；PickNewName 不接收新名称参数。[components/named.lua:15](../../../raw/current/scripts/components/named.lua#L15 |

## 典型接入

### 给新建燧石实例设置固定的自定义显示名

以下放目标模组 `modmain.lua`；沿用 flint 资源，服务端增加 named 并设置名称，postinit 入口见 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)，同步读取见 [components/named_replica.lua:1](../../../raw/current/scripts/components/named_replica.lua#L1)。

```lua
AddPrefabPostInit("flint", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.named == nil then inst:AddComponent("named") end
    inst.components.named:SetName("备用燧石")
end)
```

这是所有新实例使用同一自定义名的演示；普通统一改名优先 STRINGS.NAMES。存档加载会恢复保存的自定义名，不保证覆盖旧命名。停用模组后新实例回到默认定义，已保存名字的迁移需另处理。没有新增资源。

## 常见陷阱与限制

- **把检查语键当显示名**：inspectable.nameoverride 影响 DESCRIBE 查询；显示名改 named／STRINGS.NAMES。[components/inspectable.lua:7](../../../raw/current/scripts/components/inspectable.lua#L7 [components/named.lua:22](../../../raw/current/scripts/components/named.lua#L22
- **所有自定义状态随堆叠保留**：named 的名字不在 stackable 默认拆分复制／合并比较清单；有独立命名的物品需限制合并并处理拆分。[components/stackable.lua:72](../../../raw/current/scripts/components/stackable.lua#L72 [components/stackable.lua:109](../../../raw/current/scripts/components/stackable.lua#L109

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：SetName 的 replica／存档路径、清名回退、默认堆叠不保留任意名称与燧石改名示例，依据见上文对应锚点；未覆盖旧命名迁移或自定义拆分／合并方案。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/stackable#拆分与自定义状态]]。
