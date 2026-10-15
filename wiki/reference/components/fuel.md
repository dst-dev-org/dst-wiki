---
title: "fuel"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "把可拾取物品作为燃料来源，配置类型、燃料量与被取用回调。"
aliases: ["燃料物品", "木头燃料", "燃料类型", "fuel"]
tags: ["component", "item", "energy", "callback"]
dst_build: "steam-buildid:26230496"
sources: ["components/fuel.lua", "components/fueled.lua", "modutil.lua", "prefabs/log.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# fuel

## 快速判断

- 职责与适用：燃料来源物品；接收／消耗量由另一实体 [[reference/components/fueled]] 管，fuel 自己不持续燃烧。[components/fuel.lua:10](../../../raw/current/scripts/components/fuel.lua#L10
- 优先使用：配置 fueltype 和 fuelvalue；被取用副作用用 SetOnTakenFn。[components/fuel.lua:27](../../../raw/current/scripts/components/fuel.lua#L27
- 重要副作用：类型更新 `_fuel` 标签；Taken 发物品 fueltaken 后回调，真正删除由 fueled 完成。[components/fuel.lua:1](../../../raw/current/scripts/components/fuel.lua#L1 [components/fueled.lua:214](../../../raw/current/scripts/components/fueled.lua#L214
- 修改边界：服务端配置；燃料量不等于燃烧秒数，接收端 rate、湿燃料与其他倍率可改变实际时长。[components/fueled.lua:205](../../../raw/current/scripts/components/fueled.lua#L205

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `fueltype`／`fuelvalue` | 默认 FUELTYPE.BURNABLE／1 | 类型决定资格，数值提供基础补充值；属性回调维护类型标签。[components/fuel.lua:1](../../../raw/current/scripts/components/fuel.lua#L1 |
| `SetOnTakenFn(fn)` | `(fuel_inst, target)` | target 是接收燃料的实体，不是使用者 doer；单回调。[components/fuel.lua:27](../../../raw/current/scripts/components/fuel.lua#L27 |
| `Taken(target)` | 通知接口 | 本身不减堆叠／删实体；常由 fueled:TakeFuelItem 调用。[components/fuel.lua:31](../../../raw/current/scripts/components/fuel.lua#L31 [components/fueled.lua:214](../../../raw/current/scripts/components/fueled.lua#L214 |

## 典型接入

### 将原版木头的基础燃料量翻倍

目标模组 `modmain.lua`，沿用 log 的资源和 BURNABLE 类型；只改基础量，保留取用流程。原版 log 配 fuelvalue 见 [prefabs/log.lua:39](../../../raw/current/scripts/prefabs/log.lua#L39)，postinit 见 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)。

```lua
AddPrefabPostInit("log", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.fuel ~= nil then
        inst.components.fuel.fuelvalue = 2 * GLOBAL.TUNING.MED_FUEL
    end
end)
```

影响此模组期间 log 的补充基础值；停用后新实例恢复。实际补充量受接收端上限、倍率、湿度等影响；不更改燃烧动画／蔓延／可燃性。

## 常见陷阱与限制

- **fuel 就是 burnable**：fuel 是补充来源，burnable 是自身可燃行为，互不替代；原版木头分别配置。[prefabs/log.lua:39](../../../raw/current/scripts/prefabs/log.lua#L39 [prefabs/log.lua:41](../../../raw/current/scripts/prefabs/log.lua#L41
- **fuelvalue=60 就烧一分钟**：接收端消耗率与倍率参与，先核验接收 fueled。[components/fueled.lua:210](../../../raw/current/scripts/components/fueled.lua#L210 [components/fueled.lua:325](../../../raw/current/scripts/components/fueled.lua#L325

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；本页 sources 全部未变（fuel.lua、fueled.lua、log.lua、modutil.lua 均不在 25643519→26148036 变更清单）；已核关键消费者：本轮 eater.lua 变更（SetCanEat* 弃用、SetDiet 迁移）与 fuel 组件无交集，fueled:TakeFuelItem→fuel:Taken 链与 tuning MED_FUEL 未动；修正既有锚点偏移 log.lua:38→:39（log.lua 未变，偏移为旧有）；未覆盖：运行验证。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：fueltype 标签、Taken 事件／回调与接收端删除顺序、木头基础燃料翻倍示例，依据见上文对应锚点；未覆盖全部燃料接收方与燃烧时长组合。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/fueled#常见陷阱与限制]]；[[reference/components/stackable]]；[[reference/components/burnable#典型接入]]。
