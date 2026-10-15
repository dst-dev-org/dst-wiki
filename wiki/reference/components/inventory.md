---
title: "inventory"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "从持有者侧管理入栏、装备、鼠标手持与背包，明确查找和消耗的覆盖范围。"
aliases: ["人物物品栏", "主栏检测", "携带范围", "发放物品", "消耗材料", "inventory"]
tags: ["component", "item", "lifecycle", "callback"]
dst_build: "steam-buildid:26230496"
sources: ["components/inventory.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# inventory

## 快速判断

- 职责与适用：持有者侧的收纳／装备流程；物品侧须 [[reference/components/inventoryitem]]，箱子内部用 [[reference/components/container]]。[components/inventory.lua:1029](../../../raw/current/scripts/components/inventory.lua#L1029
- 优先使用：发放用 GiveItem，穿脱用 Equip／Unequip；先明确主栏、装备、activeitem 和背包范围再挑查询接口。[components/inventory.lua:1266](../../../raw/current/scripts/components/inventory.lua#L1266
- 重要副作用：GiveItem 会处理旧 owner、拾取回调、合并、溢出与失败丢落；不保证返回物品仍有效。[components/inventory.lua:1051](../../../raw/current/scripts/components/inventory.lua#L1051 [components/inventory.lua:1129](../../../raw/current/scripts/components/inventory.lua#L1129
- 修改边界：服务端操作实际组件，客户端走 replica／正常动作；不直接写 itemslots 或 owner 模拟转移。[components/inventory.lua:1149](../../../raw/current/scripts/components/inventory.lua#L1149

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `GiveItem(item, slot?, src_pos?)` | 多分支返回槽号、true、false 或 nil | 返回不是新 owner；满栏可走背包、activeitem 或丢落。[components/inventory.lua:1029](../../../raw/current/scripts/components/inventory.lua#L1029 [components/inventory.lua:1166](../../../raw/current/scripts/components/inventory.lua#L1166 |
| `RemoveItem(item, wholestack?, checkallcontainers?, keepoverstacked?)` | 返回移出对象；默认可只取一件 | 包含主栏／手持／装备／overflow；找不到也可能原样返回 item，不能只看非 nil。[components/inventory.lua:1390](../../../raw/current/scripts/components/inventory.lua#L1390 |
| `GetItemInSlot(slot)`／`GetEquippedItem(eslot)`／`GetActiveItem()` | 实体或 nil | 对应主栏／装备／鼠标手持；不是互换查询。[components/inventory.lua:773](../../../raw/current/scripts/components/inventory.lua#L773 [components/inventory.lua:502](../../../raw/current/scripts/components/inventory.lua#L502 |
| `FindItem(fn)`／`FindItems(fn)` | 一个对象或 nil／数组 | 两者搜索范围不同，详见下节。[components/inventory.lua:623](../../../raw/current/scripts/components/inventory.lua#L623 |
| `Has(prefab, amount, checkallcontainers?)` | enough:boolean, count:number | 默认含主栏、activeitem、overflow；第三参可查打开且允许制作的容器。[components/inventory.lua:1507](../../../raw/current/scripts/components/inventory.lua#L1507 |
| `ConsumeByName(prefab, amount?)` | 默认 1；无成功数量返回 | 主栏 → activeitem → overflow；不含任意打开箱子，预检使用相同范围。[components/inventory.lua:1772](../../../raw/current/scripts/components/inventory.lua#L1772 |

## 典型接入

### 检测人物主栏是否持有燧石

目标模组 `modmain.lua` 加以下查询函数，供后续服务端业务代码调用；不自动建周期任务，不产生效果或资源。函数只查看主栏，装备和背包不计入；GetItemInSlot 依据 [components/inventory.lua:777](../../../raw/current/scripts/components/inventory.lua#L777)。

```lua
local function HasFlintInMainSlots(player)
    if not GLOBAL.TheWorld.ismastersim then return false end
    local inventory = player ~= nil and player.components.inventory or nil
    if inventory == nil then return false end
    for slot = 1, inventory.maxslots do
        local item = inventory:GetItemInSlot(slot)
        if item ~= nil and item.prefab == "flint" then return true end
    end
    return false
end
```

调用位置应由具体效果的更新／事件方案决定；这里不承诺完整入栏光照或精神 buff。发放新物品可用 `inventory:GiveItem(GLOBAL.SpawnPrefab(...))`，但应先检查创建成功，并按 GiveItem 的失败分支设计反馈。

### 携带范围的选择

| 需求 | 读取范围／推荐方式 |
|---|---|
| 只要在人物主栏 | 遍历 1..maxslots 的 GetItemInSlot；不读装备／activeitem／背包 |
| 普通查找一件材料 | FindItem：主栏 → activeitem → overflow，不查装备 |
| 查所有普通随身匹配实体 | FindItems：主栏、装备、activeitem、overflow；返回实体数组而非逐件计数 |
| 已装备背包 | GetOverflowContainer 当前查 BODY 槽且 canbeopened 的 container；不等于任意嵌套包或所有特殊储存 |

依据 [components/inventory.lua:623](../../../raw/current/scripts/components/inventory.lua#L623 [components/inventory.lua:638](../../../raw/current/scripts/components/inventory.lua#L638 [components/inventory.lua:1459](../../../raw/current/scripts/components/inventory.lua#L1459)；归属事件顺序见 [[reference/components/inventoryitem#生命周期]]，不能只监听 ondropped 维护携带效果。

## 常见陷阱与限制

- **FindItem 与 FindItems 只差返回数量**：前者不查装备，后者查装备；携带触发条件不能随意互换。[components/inventory.lua:623](../../../raw/current/scripts/components/inventory.lua#L623 [components/inventory.lua:647](../../../raw/current/scripts/components/inventory.lua#L647
- **材料预检查箱子，扣除不查箱子**：Has 第三参 true 可计打开容器，而 ConsumeByName 没有此扩展；用相同范围预检或明确逐容器扣除，避免免费生效。[components/inventory.lua:1536](../../../raw/current/scripts/components/inventory.lua#L1536 [components/inventory.lua:1772](../../../raw/current/scripts/components/inventory.lua#L1772
- **RemoveItem 非 nil 即转移成功**：找不到物品时可返回原 item；核验原持有槽位和新归属，不能自己写 owner。[components/inventory.lua:1453](../../../raw/current/scripts/components/inventory.lua#L1453

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：FindItem／FindItems 范围、Has 与 ConsumeByName 的容器差异、主栏检测函数，依据见上文对应锚点；未交付完整携带光照／精神效果或自定义储存适配。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；查询函数经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查，未交付携带效果工程。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/equippable]]；[[reference/components/stackable]]。
