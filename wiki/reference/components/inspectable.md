---
title: "inspectable"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "添加检查交互，区分角色检查语、固定描述、动态描述与物品显示名称。"
aliases: ["检查描述", "检查物品", "动态描述", "角色台词", "inspectable"]
tags: ["component", "item", "callback"]
dst_build: "steam-buildid:26230496"
sources: ["components/inspectable.lua", "modutil.lua", "prefabs/flint.lua", "stringutil.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# inspectable

## 快速判断

- 职责与适用：提供检查状态和描述，不负责拾取或显示名称。原版燧石分别添加 inspectable 与 inventoryitem。[prefabs/flint.lua:52](../../../raw/current/scripts/prefabs/flint.lua#L52
- 优先使用：固定描述用 `SetDescription(string)`，动态描述写 `descriptionfn(inst, viewer)`；角色差异走 `STRINGS.CHARACTERS.<角色>.DESCRIBE`。[components/inspectable.lua:15](../../../raw/current/scripts/components/inspectable.lua#L15 [stringutil.lua:396](../../../raw/current/scripts/stringutil.lua#L396
- 重要副作用：黑暗、幽灵／默剧角色和闷烧等分支可覆盖自定义文字。[components/inspectable.lua:53](../../../raw/current/scripts/components/inspectable.lua#L53
- 修改边界：服务端配置检查组件；改名字见 [[reference/components/named]]，不要用 `SetNameOverride` 改物品显示名。[components/inspectable.lua:7](../../../raw/current/scripts/components/inspectable.lua#L7

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetDescription(desc)` | 推荐传 string | 只赋值，不在读取时调用函数；动态描述使用下一行。[components/inspectable.lua:15](../../../raw/current/scripts/components/inspectable.lua#L15 |
| `descriptionfn` | `(inst, viewer) -> description` | 优先于 description；更前面还有 getspecialdescription 分支。[components/inspectable.lua:68](../../../raw/current/scripts/components/inspectable.lua#L68 |
| `getstatus` | `(inst, viewer) -> 状态字符串或 nil` | 自定义结果优先，否则回退 DEAD／HELD 等；状态用于检查语选择。[components/inspectable.lua:27](../../../raw/current/scripts/components/inspectable.lua#L27 |
| `SetNameOverride(key)` | 检查语键 string | 改 DESCRIBE 查询键；不改屏幕名称。[stringutil.lua:375](../../../raw/current/scripts/stringutil.lua#L375 |

## 典型接入

### 给燧石增加固定检查描述

目标模组的 `modmain.lua` 放以下完整接入代码；沿用原版 flint 和资源，仅服务端执行。适合统一描述，角色专属台词应走 STRINGS。Prefab postinit 注册见 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)，读描述见 [components/inspectable.lua:71](../../../raw/current/scripts/components/inspectable.lua#L71)。

```lua
AddPrefabPostInit("flint", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.inspectable ~= nil then
        inst.components.inspectable:SetDescription("可以用来制作基础工具。")
    end
end)
```

影响启用模组期间的实例；停用后新实例恢复原描述。若其他模组设置 descriptionfn，本固定值不会取代它；黑暗等覆盖仍保留。无新增资源。

## 常见陷阱与限制

- **把函数塞进 SetDescription**：源码注释提到函数，但当前 GetDescription 未执行 description 中的函数；用 descriptionfn，避免把函数对象当检查结果。[components/inspectable.lua:71](../../../raw/current/scripts/components/inspectable.lua#L71
- **检查状态等于持有状态**：HELD 只检查直接 owner 非 nil，箱子内容也可能命中；人物主栏判定另见 [[reference/components/inventory#携带范围的选择]]。[components/inspectable.lua:47](../../../raw/current/scripts/components/inspectable.lua#L47
- **固定描述覆盖所有情况**：幽灵／mime 强制走全局检查语，闷烧也有专门文字；按最终分支判断。[components/inspectable.lua:77](../../../raw/current/scripts/components/inspectable.lua#L77

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：固定／动态描述优先级、黑暗／幽灵／闷烧覆盖、检查语键与燧石描述示例，依据见上文对应锚点；未穷举全部角色台词和特殊查看者分支。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

相关：[[reference/components/inventoryitem]]；[[reference/components/named]]。
