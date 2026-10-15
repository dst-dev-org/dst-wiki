---
title: "stackable"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "配置物品堆叠、按件拆分和消耗，处理自定义状态、数量同步及合并边界。"
aliases: ["物品堆叠", "拆分一件", "堆叠上限", "无限堆叠", "stackable"]
tags: ["component", "item", "lifecycle", "network-sync"]
dst_build: "steam-buildid:26230496"
sources: ["components/stackable.lua", "components/stackable_replica.lua", "modutil.lua", "prefabs/flint.lua", "prefabs/walls.lua", "tuning.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# stackable

## 快速判断

- 职责与适用：同类物品的件数，不是工具使用次数；按次耐久见 [[reference/components/finiteuses]]。[components/stackable.lua:23](../../../raw/current/scripts/components/stackable.lua#L23
- 优先使用：上限用原版 `TUNING.STACK_SIZE_*`，数量用 `SetStackSize`；消耗一件用 `Get(1):Remove()`。[components/stackable.lua:103](../../../raw/current/scripts/components/stackable.lua#L103 [prefabs/walls.lua:159](../../../raw/current/scripts/prefabs/walls.lua#L159
- 重要副作用：拆分可能创建新 Prefab；合并会删除完全被吸收的实体。[components/stackable.lua:109](../../../raw/current/scripts/components/stackable.lua#L109 [components/stackable.lua:159](../../../raw/current/scripts/components/stackable.lua#L159
- 修改边界：服务端配置；默认兼容性看 Prefab、皮肤和自定义比较函数，不自动比较任意业务字段。[components/stackable.lua:72](../../../raw/current/scripts/components/stackable.lua#L72

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `maxsize` | 默认 TUNING.STACK_SIZE_MEDITEM | 属性同步要求原版编码支持的 STACK_SIZE 常量；不可随意设 999。[components/stackable_replica.lua:1](../../../raw/current/scripts/components/stackable_replica.lua#L1 [components/stackable_replica.lua:98](../../../raw/current/scripts/components/stackable_replica.lua#L98 |
| `SetStackSize(sz)`／`StackSize()` | 正整数推荐域／数量 | setter 只限制 MAXUINT，未限制 maxsize 或保证正数；发 stacksizechange。[components/stackable.lua:103](../../../raw/current/scripts/components/stackable.lua#L103 |
| `Get(num?)` | 默认 1；返回物品实体 | 足量时分出新实例，否则返回原实体；不是永远创建新对象。[components/stackable.lua:109](../../../raw/current/scripts/components/stackable.lua#L109 |
| `CanStackWith(item)`／`Put(item, source_pos?)` | boolean／剩余物品或 nil | 先判兼容；不兼容与完全合并都可能返回 nil，不能只据 nil 判断成功。[components/stackable.lua:72](../../../raw/current/scripts/components/stackable.lua#L72 [components/stackable.lua:159](../../../raw/current/scripts/components/stackable.lua#L159 |
| `SetOnDeStack(fn)` | `(newinst, oldinst)` | 拆分时复制自定义状态的入口，执行于原版其他状态继承之前。[components/stackable.lua:118](../../../raw/current/scripts/components/stackable.lua#L118 |

## 典型接入

### 将燧石上限提高到原版微小物品堆叠档位

目标模组 `modmain.lua`；使用原版 flint 的 stackable 和资源，服务端把上限从 SMALLITEM 的 40 提高到 TINYITEM 的 60。初始化见 [prefabs/flint.lua:47](../../../raw/current/scripts/prefabs/flint.lua#L47)，档位数值见 [tuning.lua:81](../../../raw/current/scripts/tuning.lua#L81)，postinit 见 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)。

```lua
AddPrefabPostInit("flint", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.stackable ~= nil then
        inst.components.stackable.maxsize = GLOBAL.TUNING.STACK_SIZE_TINYITEM
    end
end)
```

仅改可合并上限，不重新分配已保存数量；停用后新实例恢复原档位。无限容器堆叠用 [[reference/components/container#超堆叠与删除内容]] 的容器入口，不通过任意 maxsize 模拟。

### 拆分与自定义状态

Get 会继承部分腐败、冷却、湿度／温度状态，但不会复制任意自定义等级、命名或容器内容；拆分后的 OnPutInInventory 也不等于持有者已登记新槽位。合并会稀释新鲜度／湿度／温度。自定义状态先决定是否允许合并，再用 `inst.stackable_CanStackWithFn` 和 ondestack 保持规则；不建议直接给独立容器添加堆叠而不设计内容迁移。[components/stackable.lua:72](../../../raw/current/scripts/components/stackable.lua#L72 [components/stackable.lua:118](../../../raw/current/scripts/components/stackable.lua#L118 [components/stackable.lua:141](../../../raw/current/scripts/components/stackable.lua#L141 [components/stackable.lua:171](../../../raw/current/scripts/components/stackable.lua#L171

### 数据工厂与一个文件多个物品

本地片段参照（未随库分发，未测游戏／外部 Hook；[版本与文件指纹](../../../docs/reports/w05/local-mod-references-20261017.md)）：DSTKisaki 的 kisaki_items.lua:45～59 在 MakeItem 通用工厂统一挂 inspectable／inventoryitem／stackable／tradable，并由 master_postinit 扩展；这是“基础能力统一、特殊能力按定义添加”的模式。Sora 的 sora2base.lua:623～655 的 stonefn 则只给 sora2stone 添加物品／堆叠和专属使用组件，文件内建筑 fn 的 container 不能算到该石头上。批量扫描用于发现组合，统计必须按实际工厂／分支和 helper 核验，不能把文件共现当同一 Prefab 能力。

## 常见陷阱与限制

- **Get(1) 后把原实体也删掉**：只有一件时返回原物品，多件时返回分出的物品；只删除返回对象，操作后原对象不保证有效。[components/stackable.lua:109](../../../raw/current/scripts/components/stackable.lua#L109
- **改成 999 就能无限堆叠**：replica SetMaxSize 对固定查表值减一，未知值会出错；使用原版档位或明确改造同步方案。[components/stackable_replica.lua:98](../../../raw/current/scripts/components/stackable_replica.lua#L98
- **服务端大数量等于客户端精确显示**：当前 replica 数量编码在 4096 饱和；超堆叠 UI 不代表服务端真实数量。[components/stackable_replica.lua:47](../../../raw/current/scripts/components/stackable_replica.lua#L47

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 0 处（唯一命中的 tuning.lua#L81 STACK_SIZE 档位行未变）；结论修订：无。
- 依据：tuning 键级 hunk 地图＋变更清单命中 tuning.lua。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：Get 拆分／Put 合并及自定义状态边界、replica 档位／数量限制、燧石上限示例，依据见上文对应锚点；未交付独立容器堆叠或任意业务状态迁移方案。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

相关：[[reference/components/inventoryitem#生命周期]]；[[reference/components/perishable]]；[[reference/components/container]]。
