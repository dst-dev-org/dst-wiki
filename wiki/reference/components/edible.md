---
title: "edible"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "配置食物类型、三维收益与吃下回调，区分基础值、食用者规则和腐败。"
aliases: ["新增食物", "食物三维", "吃下回调", "食物类型", "edible"]
tags: ["component", "item", "food", "callback"]
dst_build: "steam-buildid:26230496"
sources: ["components/eater.lua", "components/edible.lua", "modutil.lua", "prefabs/flint.lua", "prefabs/log.lua", "prefabs/veggies.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# edible

## 快速判断

- 职责与适用：物品侧食用类型和数值；食用者能否吃由 eater 等规则共同决定。腐败时间另见 [[reference/components/perishable]]。[components/edible.lua:19](../../../raw/current/scripts/components/edible.lua#L19 [components/eater.lua:275](../../../raw/current/scripts/components/eater.lua#L275
- 优先使用：设置 foodtype、healthvalue／hungervalue／sanityvalue；扩展效果用 SetOnEatenFn。[prefabs/veggies.lua:595](../../../raw/current/scripts/prefabs/veggies.lua#L595 [components/edible.lua:149](../../../raw/current/scripts/components/edible.lua#L149
- 重要副作用：实际收益可受腐败、食用者和调味等修正，oneaten 后正常流程会消耗食物。[components/edible.lua:72](../../../raw/current/scripts/components/edible.lua#L72 [components/eater.lua:275](../../../raw/current/scripts/components/eater.lua#L275
- 修改边界：服务端配置；组件提供 edible_类型 标签，不能只看“有 edible”就判断任意角色可吃。[components/edible.lua:9](../../../raw/current/scripts/components/edible.lua#L9

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `foodtype`／`secondaryfoodtype` | FOODTYPE 值／值或 nil | 默认 GENERIC；主副类型不可相同，属性更新类型标签。[components/edible.lua:9](../../../raw/current/scripts/components/edible.lua#L9 |
| `healthvalue`／`hungervalue`／`sanityvalue` | number；默认 10／10／0 | 是基础收益，GetHealth／GetHunger／GetSanity 再计算修饰。[components/edible.lua:19](../../../raw/current/scripts/components/edible.lua#L19 [components/edible.lua:72](../../../raw/current/scripts/components/edible.lua#L72 |
| `SetOnEatenFn(fn)` | `(inst, eater)` | 单回调；正常 eater 流程先处理收益，再 OnEaten，最后移除／消耗。[components/edible.lua:183](../../../raw/current/scripts/components/edible.lua#L183 [components/eater.lua:275](../../../raw/current/scripts/components/eater.lua#L275)、[:276](../../../raw/current/scripts/components/eater.lua#L276) |
| `degrades_with_spoilage` | boolean；默认 true | false 跳过本组件对应的腐败收益退化判断，不停止 perishable 腐败／替换。[components/edible.lua:74](../../../raw/current/scripts/components/edible.lua#L74 |
| `temperaturedelta`／`temperatureduration` | 数值／秒；默认 0 | 经 OnEaten 设置腹内温度效果，还受 chill／调味修饰；不等于食物本体恒温。[components/edible.lua:201](../../../raw/current/scripts/components/edible.lua#L201 |

## 典型接入

### 修改原版胡萝卜的基础三维收益

目标模组 `modmain.lua`；复用 carrot 的工厂和资源，保留腐败、堆叠、烹饪与原回调。原版蔬菜工厂在 veggies.lua 按 VEGGIES 名称注册；并非 carrot.lua。[prefabs/veggies.lua:595](../../../raw/current/scripts/prefabs/veggies.lua#L595 [prefabs/veggies.lua:1096](../../../raw/current/scripts/prefabs/veggies.lua#L1096 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("carrot", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local edible = inst.components.edible
    if edible ~= nil then
        edible.healthvalue = 6
        edible.hungervalue = 12
        edible.sanityvalue = 3
    end
end)
```

数值为每次标准食用的基础属性收益，最终值仍受食物状态与人物规则影响；不影响 carrot_cooked。停用后新实例恢复原基础值。没有新增资源或持续 buff。

### 普通食物与特殊食用者

本地片段参照（未随库分发，未测游戏／外部 Hook；[版本与文件指纹](../../../docs/reports/w05/local-mod-references-20261017.md)）：MCW 的 mcw_fruit_tea.lua:86～100 把 edible 三维、perishable 计时／替换、stackable 数量分别设置；Sora 的 sorafoods.lua:157～161 仅在数据含正 perishtime 时加腐败。新食物按需求组合这些能力，不自动腐败，也不把堆叠数当每次食用收益。

有 edible 的物品也未必是人物常规食物：原版 log 为 WOOD，flint 为 ELEMENTAL，说明容器过滤／“能吃”判定必须说明食用者。[prefabs/log.lua:33](../../../raw/current/scripts/prefabs/log.lua#L33 [prefabs/flint.lua:42](../../../raw/current/scripts/prefabs/flint.lua#L42

Eater:TestFood 同样匹配 edible_类型 标签；手动追加标签可能真的扩大食用资格，但不等于 foodtype／secondaryfoodtype 字段也同步改变。优先用组件字段维护主副类型；确需额外标签时，分别核验可食用性、收益计算、移组件清理与消费者分支，不把标签当无副作用的兼容装饰。[components/eater.lua:288](../../../raw/current/scripts/components/eater.lua#L288 [components/edible.lua:9](../../../raw/current/scripts/components/edible.lua#L9 [components/edible.lua:56](../../../raw/current/scripts/components/edible.lua#L56

## 常见陷阱与限制

- **OnEaten 内再次删整堆**：正常 eater 已在后面调用 HandleEatRemove；默认堆叠只消耗一件。额外自行 Remove 会删除整堆或改变流程。[components/edible.lua:216](../../../raw/current/scripts/components/edible.lua#L216 [components/eater.lua:276](../../../raw/current/scripts/components/eater.lua#L276
- **修改正精神值就一直有效**：陈旧／腐败时 GetSanity 可归零或改成惩罚；按最终 getter 和 eater 审阅。[components/edible.lua:78](../../../raw/current/scripts/components/edible.lua#L78
- **SetOnEatenFn 自动叠加**：它替换单字段；扩展既有食物应保存并串联旧回调，核验旧回调是否删实体。[components/edible.lua:149](../../../raw/current/scripts/components/edible.lua#L149 [components/edible.lua:183](../../../raw/current/scripts/components/edible.lua#L183

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对漂移文件 components/eater.lua 对照更新报告 C-12：SetCanEat* 四函数移至文件尾标 Deprecated（幂等去重，单次调用等价）、新增 ResetEdibleTagsCache（eater.lua:194）、SetDiet 仍不清 edible tags 缓存；Eat／TestFood／oneat 与 OnEaten→HandleEatRemove 顺序无语义变化，本页结论全部成立，仅重定位锚点 5 处；edible.lua、modutil.lua、veggies.lua、log.lua、flint.lua 哈希核对未变；未覆盖：运行验证。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：基础三维与腐败收益、食用资格标签、OnEaten 后消耗顺序、胡萝卜数值示例，依据见上文对应锚点；未覆盖全部食用者特例与自定义调味效果。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/eater#进食顺序与恢复|食用者收益与消耗顺序]]；[[reference/components/perishable]]；[[reference/components/stackable]]；[[reference/components/inspectable]]；[[reference/components/debuffable#吃胡萝卜施加或刷新加速效果|食用后限时效果]]。
