---
title: "inventoryitemmoisture"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "维护物品自身湿度、湿润标志与环境目标，区分角色湿度、容器暴露和外部控制。"
aliases: ["物品湿度", "湿物品", "物品自动干燥", "暂停物品吸湿", "物品湿润回调", "inventoryitemmoisture"]
tags: [component, item, lifecycle, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/inventoryitemmoisture.lua", "components/inventoryitem.lua", "components/inventoryitem_replica.lua", "components/waterproofer.lua", "components/stackable.lua", "prefabs/inventoryitem_classified.lua", "prefabs/flint.lua", "prefabs/desiccant.lua", "tuning.lua", "entityscript.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# inventoryitemmoisture

## 快速判断

- 职责与适用：`inventoryitem` 的服务端扩展，保存**物品自身**湿度及 `iswet`。宿主无 waterproofer 时默认启用；角色湿度用 [[reference/components/moisture]]。[inventoryitem.lua:87](../../../raw/current/scripts/components/inventoryitem.lua#L87
- 优先使用：宿主 `AddMoisture(delta)`／`DryMoisture()` 改值，`GetMoisture()`／`GetMoisturePercent()`／`IsWet()` 读值；扩展配置见[关键接口](#关键接口)。[inventoryitem.lua:127](../../../raw/current/scripts/components/inventoryitem.lua#L127)、[:172](../../../raw/current/scripts/components/inventoryitem.lua#L172)
- 重要副作用：数值与湿润标志[分开同步](#同步存读与清理)；外部控制冻结自然目标，仍允许直接改值和堆叠混合。
- 修改边界：[有限速率追随目标](#自然目标与更新)，不与人物湿度实时相等；waterproofer 增删会移除／重建此扩展，丢失原配置。[waterproofer.lua:9](../../../raw/current/scripts/components/waterproofer.lua#L9

## 关键接口

| 接口／字段 | 参数与作用 | 条件与证据 |
|---|---|---|
| `inventoryitem:EnableMoisture(enable)` | false 移除；其余值在缺失时添加并 `AttachReplica` | 绑定后初始化 moisture=0、iswet=false；ComponentPostInit 发生在绑定前，不能提前写属性。[inventoryitem.lua:116](../../../raw/current/scripts/components/inventoryitem.lua#L116)、[inventoryitemmoisture.lua:74](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L74)、[entityscript.lua:655](../../../raw/current/scripts/entityscript.lua#L655 |
| `SetMoisture(value)`／`DoDelta(delta)` | 湿度点；钳制 `0..TUNING.MAX_WETNESS`，本 build 上限100 | 普通模式输入≥35变湿、≤15变干，中间保留旧标志；DoDelta 转交 setter。宿主 DryMoisture 等价设0。[inventoryitemmoisture.lua:146](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L146)、[tuning.lua:88](../../../raw/current/scripts/tuning.lua#L88)、[:2930](../../../raw/current/scripts/tuning.lua#L2930) |
| `InheritMoisture(value, iswet)` | 复制数值与湿润历史 | 非普通 setter：中间区间按传入 iswet 决定；宿主有同名代理，拆堆会使用。[inventoryitemmoisture.lua:113](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L113)、[stackable.lua:141](../../../raw/current/scripts/components/stackable.lua#L141 |
| `SetExternallyControlled(bool)` | true 令目标等于当前值；false 恢复环境追随 | 不归零／取消任务，不拦截外部改值。[inventoryitemmoisture.lua:166](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L166)、[:179](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L179) |
| `SetOnlyWetWhenSaturated(bool)` | true 时后续 setter 仅在值等于100时置湿 | 设置此开关本身不重算标志；OnLoad 直接恢复存档 wet，也不按开关重算。[inventoryitemmoisture.lua:150](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L150)、[:170](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L170)、[:251](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L251) |
| `SetOnMoistureDeltaCallback(fn?)` | `(inst, oldmoisture, newmoisture)`；nil 清除 | 单槽回调，重复设置覆盖；值未变也可调用，OnLoad 的 old 固定0；不发角色 moisturedelta。[inventoryitemmoisture.lua:174](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L174)、[:251](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L251) |

### 自然目标与更新

`GetTargetMoisture()` 先检查外部控制；未接管时按以下规则求值，再由 `UpdateMoisture(dt)` 以**增湿0.5点／秒、干燥1点／秒**接近目标，不一步复制目标。[inventoryitemmoisture.lua:179](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L179)、[:229](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L229)

| 位置／持有者 | 自然目标 |
|---|---|
| 自身 floater 正在漂浮 | 100，优先于以下环境分支；与刚入海的初始赋湿并非同一入口——该初始赋湿（inventoryitem 的 `InheritWorldWetnessAtXZ`／`AtTarget`）26148036 起在雨罩检查外另需该点 `CanPointHaveRain`，无雨区入水不再立即继承世界湿度。[初始赋湿门控](../../../raw/current/scripts/components/inventoryitem.lua#L150-L164) |
| 无 owner 的地面物品 | 下雨且自身无 rainimmunity 时为世界 wetness，否则0。 |
| 直接 owner 为有 moisture 的角色 | 使用该 owner 当前湿度作为目标；不直接写角色湿度。 |
| 普通不暴露的容器 | 通常0，原有湿度逐渐干燥；不能据此认为入箱瞬间变干。 |
| `container.isexposed` 的容器 | 沿暴露容器的 inventoryitem.owner 链向外；遇不暴露容器终止为干燥，遇非容器 owner 改按其 moisture，遇 rideable 且有骑手则按骑手的雨水／rainimmunity 暴露条件处理。到最外地面容器则按它的雨水条件。 |

变化时周期1秒，到目标后2秒；睡眠取消任务，醒来用**当前目标**补算睡眠时长，再随机延迟启动，不回放天气／归属历史。宿主统一分发睡醒入口；运行中启用扩展不会自动补 wake。[inventoryitemmoisture.lua:46](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L46)、[:90](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L90)、[mainfunctions.lua:717](../../../raw/current/scripts/mainfunctions.lua#L717)、[:751](../../../raw/current/scripts/mainfunctions.lua#L751)、[entityscript.lua:632](../../../raw/current/scripts/entityscript.lua#L632

### 同步、存读与清理

- 属性回调将 moisture 写入 inventoryitem classified 的 net_float，将 iswet 写入物品 replica 的 net_bool。客户端通过 `inst.replica.inventoryitem` 读；classified 的可见目标随 owner／容器打开者改变，不能假定任意客户端都持有完整数值。[inventoryitemmoisture.lua:14](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L14)、[inventoryitem_replica.lua:171](../../../raw/current/scripts/components/inventoryitem_replica.lua#L171)、[:446](../../../raw/current/scripts/components/inventoryitem_replica.lua#L446)、[inventoryitem_classified.lua:208](../../../raw/current/scripts/prefabs/inventoryitem_classified.lua#L208
- `wetnesschange` 的 data 是 boolean，由标志变化及客户端 dirty 桥接产生；它不是每次数值变化的通知。数值网络变量没有在该定义绑定专用 dirty 事件。[inventoryitem_replica.lua:472](../../../raw/current/scripts/components/inventoryitem_replica.lua#L472)、[inventoryitem_classified.lua:117](../../../raw/current/scripts/prefabs/inventoryitem_classified.lua#L117
- OnSave 只存正 moisture 与真 wet；OnLoad 恢复这两个值并可调用上述回调。开关、回调、任务均不存档，常驻配置须在实例初始化重建。移组件先将数值／湿润标志清零并取消任务；移除 inventoryitem 会连同扩展一起移除。[inventoryitemmoisture.lua:242](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L242)、[:80](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L80)、[inventoryitem.lua:106](../../../raw/current/scripts/components/inventoryitem.lua#L106

## 典型接入

### 暂停燧石的自然吸湿和干燥，保留外部赋湿

完整代码放已加载模组 `modmain.lua`，服务端复用 flint 工厂和资源，无新增资源／任务；跳过已被禁用的扩展。原宿主及 PostInit 加载依据见 [flint.lua:52](../../../raw/current/scripts/prefabs/flint.lua#L52)、[[reference/components/inventoryitem#典型接入]]。

```lua
AddPrefabPostInit("flint", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local moisture = inst.components.inventoryitemmoisture
    if moisture ~= nil then
        moisture:SetExternallyControlled(true)
    end
end)
```

保留初始化／读档湿度，每次实例重建重登开关，OnLoad 不覆盖它。实时撤销调用 `SetExternallyControlled(false)`，下一次更新恢复自然目标；停用并重载后不再登记，已存湿度保留。该布尔开关没有来源计数，多模组须协调所有权；外部写入边界见下节。

原版 desiccant 则由 `temperaturedelta`／`onputininventory` 动态切换外部控制，吸湿回调显式累积物品湿度，并设置仅饱和变湿与湿度回调；复制开关不足以实现玩家干燥。[desiccant.lua:24](../../../raw/current/scripts/prefabs/desiccant.lua#L24)、[:53](../../../raw/current/scripts/prefabs/desiccant.lua#L53)、[:147](../../../raw/current/scripts/prefabs/desiccant.lua#L147)

## 常见陷阱与限制

- **直接添加扩展再写属性**：会调用尚无 replica 的 setter；按[启用接口](#关键接口)装配。[inventoryitemmoisture.lua:14](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L14
- **把所有加湿入口当成 SetMoisture**：`MakeMoistureAtLeast(min)` 不做上限钳制，普通模式 `min>15` 就可置湿；只给合法范围的 min。其规则不同于35点湿润阈值，继承和读档也各有规则。[inventoryitemmoisture.lua:133](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L133
- **外部控制等于数值锁定**：显式 setter、读档、拆堆继承与合堆按件数加权仍能改值；固定值需求须另审阅写入入口。[inventoryitemmoisture.lua:126](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L126)、[stackable.lua:175](../../../raw/current/scripts/components/stackable.lua#L175

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 4 处（inventoryitem AddMoisture、tuning 湿润阈值、mainfunctions 睡醒分发两处）；结论修订 1 处：入海／对目标初始赋湿入口 26148036 起追加 `TheWorld.Map:CanPointHaveRain` 位置门控（R-08 族，inventoryitem.lua:153/:161），无雨区入水不再立即继承世界湿度；物品自身的 GetTargetMoisture 自然目标与更新速率在 inventoryitemmoisture.lua 内、本版未变，逐访问条件（地面／owner／暴露容器）旧结论均保留。依据：全局枢纽 diff 审阅＋变更清单命中 inventoryitem.lua、tuning.lua、entityscript.lua、mainfunctions.lua。
- 源码审阅：2026-10-21，steam-buildid:25643519；扩展全文及本页宿主、owner 链、更新／存读／清理、同步、堆叠和两种原版消费者。角色湿度／PostInit 复用同版本 Wiki。
- 检查：本页结构、来源存在性、链接／源码行号及完整 modmain 示例 Lua 5.1 语法通过，未检查导航登记。
- 游戏运行：未运行；多人可见性、睡眠天气变化、暴露容器／骑乘、读档与模组冲突待实测；未枚举全部加湿消费者或审校全组件类型。

相关：[[reference/components/inventoryitem]]；[[reference/components/moisture]]；[[reference/components/waterproofer]]；[[reference/components/inventoryitemtemperature]]。
