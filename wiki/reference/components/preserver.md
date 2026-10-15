---
title: "preserver"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "按直接持有者控制物品腐败和温度变化倍率，区分减慢、暂停与反向恢复。"
aliases: ["保鲜背包", "背包放冰箱", "嵌套容器保鲜", "防腐", "反鲜", "暂停腐败", "容器保鲜", "preserver"]
tags: ["component", "item", "mechanism"]
dst_build: "steam-buildid:26230496"
sources: ["components/perishable.lua", "components/preserver.lua", "components/temperature.lua", "modutil.lua", "prefabs/backpack.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# preserver

## 快速判断

- 职责与适用：加在直接持有者／容器上，为内部物品提供倍率；不提供格子，也不自动传到任意嵌套层。[components/perishable.lua:68](../../../raw/archive/build-26148036/scripts/components/perishable.lua#L68
- 优先使用：SetPerishRateMultiplier 控腐败；SetTemperatureRateMultiplier 控 temperature 更新倍率，两个效果分开。[components/preserver.lua:8](../../../raw/archive/build-26148036/scripts/components/preserver.lua#L8 [components/temperature.lua:333](../../../raw/archive/build-26148036/scripts/components/temperature.lua#L333
- 重要副作用：preserver 比 fridge 标签分支优先；负腐败倍率可以恢复时间，但环境分支仍会参与。[components/perishable.lua:79](../../../raw/archive/build-26148036/scripts/components/perishable.lua#L79 [components/perishable.lua:133](../../../raw/archive/build-26148036/scripts/components/perishable.lua#L133
- 修改边界：倍率 0 不等于所有状态下永久保鲜，frozen 低温分支可能改写倍率；设置温度倍率也不等于把物品恒定到某温度。[components/perishable.lua:112](../../../raw/archive/build-26148036/scripts/components/perishable.lua#L112 [components/temperature.lua:514](../../../raw/archive/build-26148036/scripts/components/temperature.lua#L514

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetPerishRateMultiplier(rate)` | number 或 `(holder, item)->number` | 默认 1；0 暂停基础倒计时、负数反向恢复、正数按倍率衰减，边界见下文。[components/preserver.lua:8](../../../raw/archive/build-26148036/scripts/components/preserver.lua#L8 |
| `GetPerishRateMultiplier(item)` | 返回数值，nil 回退 1 | 函数按持有者实体和物品调用；Lua 的 0 不会回退 1。[components/preserver.lua:12](../../../raw/archive/build-26148036/scripts/components/preserver.lua#L12 |
| `SetTemperatureRateMultiplier(rate)` | number 或 `(holder, item)->number` | 默认 1；由物品 temperature 的直接 owner 分支消费，不是 edible 温度字段的统一倍率。[components/preserver.lua:16](../../../raw/archive/build-26148036/scripts/components/preserver.lua#L16 [components/temperature.lua:328](../../../raw/archive/build-26148036/scripts/components/temperature.lua#L328 |

## 典型接入

### 让原版背包内物品以四分之一基础速率腐败

以下放目标模组 `modmain.lua`，不新增资源，不改背包格子。服务端在 backpack 加 preserver，影响直接存入该包且由 perishable 更新的物品。[prefabs/backpack.lua:277](../../../raw/archive/build-26148036/scripts/prefabs/backpack.lua#L277 [components/perishable.lua:79](../../../raw/archive/build-26148036/scripts/components/perishable.lua#L79 [modutil.lua:593](../../../raw/archive/build-26148036/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("backpack", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.preserver == nil then inst:AddComponent("preserver") end
    inst.components.preserver:SetPerishRateMultiplier(0.25)
end)
```

0.25 是基础倍率，湿度／天气等仍参与；这不是固定四倍总保质期承诺。移出该包后使用新直接 owner 的规则；停用后新实例恢复原定义，已改变的新鲜度不自动恢复。

### 模组中的减慢、暂停与反鲜模式

本地片段参照（未随库分发，未测游戏／外部 Hook；[版本与文件指纹](../../../docs/reports/w05/local-mod-references-20261017.md)）：MCW 的 mcw_gembox.lua:229～230 用 0.05，mcw_snack_sack.lua:85～86 用 -5；Sora 的 sora3chest.lua:91～92 用 0。这三类分别对应正倍率减慢、负倍率恢复、零倍率暂停基础衰减，不能把模组备注直接当作全条件效果保证。它们的容器布局／携带权限仍独立配置；通用条件以 [components/perishable.lua:79](../../../raw/archive/build-26148036/scripts/components/perishable.lua#L79)、[components/perishable.lua:112](../../../raw/archive/build-26148036/scripts/components/perishable.lua#L112)、[components/perishable.lua:133](../../../raw/archive/build-26148036/scripts/components/perishable.lua#L133 为准。无需复制专属升级器或世界管理器。

## 常见陷阱与限制

- **背包放进冰箱就能保鲜内部食物**：原版普通 backpack 明确 `cangoincontainer = false`，不能直接放入冰箱等容器。若模组另行允许嵌套，perishable 仍读取食物的直接 owner，不沿 GetGrandOwner 继承外层冰箱的保鲜效果；应给实际持有食物的背包配置 preserver。[背包容器资格](../../../raw/archive/build-26148036/scripts/prefabs/backpack.lua#L268-L269)、[直接持有者与倍率](../../../raw/archive/build-26148036/scripts/components/perishable.lua#L68-L85)
- **0 永远不腐败**：低温 frozen 分支可替换倍率；其他消耗／替换逻辑也不由 preserver 禁用。按物品标签与环境核验。[components/perishable.lua:112](../../../raw/archive/build-26148036/scripts/components/perishable.lua#L112
- **负倍率必定复活／生成新物品**：只将剩余时间向 perishtime 上限恢复；已经 Perish 替换／删除的实体无法靠此恢复。[components/perishable.lua:133](../../../raw/archive/build-26148036/scripts/components/perishable.lua#L133 [components/perishable.lua:277](../../../raw/archive/build-26148036/scripts/components/perishable.lua#L277

## 验证记录

- 局部增补：2026-10-27，Steam buildid 26148036；核验普通 backpack 的 cangoincontainer=false 与 perishable 的直接 owner 倍率分支，补明原版限制及模组允许嵌套时的条件。未审校全部背包／冰箱过滤或模组 Hook，原倍率／温度及示例覆盖沿用既有记录；未运行游戏。
- 版本增量复核 26148036（2026-10-25）：锚点重定位 1 处（perishable Perish 定义 -2，显示标签同步）；perishable.lua 本版两处改动（Update 酸雨分支追加 `CanPointHaveAcidRain` 位置门控于 :98、AddTime 移除 updatetask 门控即未启动计时也可加时，C-15）均不在本页引用面，倍率结论逐访问条件保留；preserver／temperature／backpack／modutil 未变。依据：组内共享 perishable diff＋变更清单命中 components/perishable.lua。
- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：直接 owner 消费、正／零／负腐败倍率及 frozen 改写、温度倍率入口与背包示例，依据见上文对应锚点；未覆盖全部环境／嵌套容器组合，不保证固定总保质期。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/container]]；[[reference/components/perishable]]；[[reference/components/temperature#自然温度绝热与容器如何选择]]。
