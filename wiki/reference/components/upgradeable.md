---
title: "upgradeable"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "通用目标的升级累计、单件消费、阶段回调与保存；具体成长和外观恢复仍由宿主实现。"
aliases: ["蜘蛛巢每阶段三点升级值", "升级累计与阶段", "upgradeable"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/upgradeable.lua", "components/upgrader.lua", "prefabs/spiderden.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# upgradeable

## 快速判断

- 职责与适用：目标记录 stage／numupgrades，消费 [[reference/components/upgrader]] 材料、调升级／阶段回调；不直接管理建筑外观或生物成长。蜘蛛巢阶段回调另调 growable:DoGrowth。[组件](../../../raw/current/scripts/components/upgradeable.lua#L17-L32) [宿主](../../../raw/current/scripts/prefabs/spiderden.lua#L548-L565)
- 优先使用：调每级累计门槛用 upgradesperstage；业务门禁用 SetCanUpgradeFn；正常投入走原 UPGRADE 动作，保持双方检查和消费。[门禁](../../../raw/current/scripts/components/upgradeable.lua#L60-L72) [动作](../../../raw/current/scripts/actions.lua#L4056-L4067)
- 重要副作用：Upgrade 先累加并消费一件，再调 onupgradefn，随后至多 AdvanceStage 一次并清累计。回调拿到的 obj 可能已被删除，不能据对象参数承诺仍有效。[执行](../../../raw/current/scripts/components/upgradeable.lua#L75-L92)
- 修改边界：服务端；直接 Upgrade 不重检类型／角色／满级，AdvanceStage 也不限制上界。SetStage 只改数值／标签，不等于推进宿主全部成长表现。[阶段](../../../raw/current/scripts/components/upgradeable.lua#L43-L56) [原宿主同步](../../../raw/current/scripts/prefabs/spiderden.lua#L142-L160)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `stage`／`numstages`／`upgradesperstage`／`numupgrades` | 默认 1／3／5／0；阶段或总阶段属性变化刷新当前类型候选标签，累计量并非百分比或材料件数。[默认与钩子](../../../raw/current/scripts/components/upgradeable.lua#L17-L32) |
| `CanUpgrade()`／`SetCanUpgradeFn(fn)` | fn(inst)→allowed,reason，需业务允许且 stage<numstages；达到上限时业务回调仍可被调用。仅改业务状态不自动重建标签，执行时须最终检查。[方法](../../../raw/current/scripts/components/upgradeable.lua#L39-L72) |
| `Upgrade(obj,performer)` | 从 obj.upgrader.upgradevalue 累加，stackable:Get(1):Remove 或直接 Remove；不返回被消耗物品，不自动处理失败退款，末尾 true。[顺序](../../../raw/current/scripts/components/upgradeable.lua#L75-L92) |
| `SetOnUpgradeFn(fn)`／`onupgradefn` | fn(target,performer,obj)，执行在消费之后、阶段推进之前，返回值不作为取消升级。[设置](../../../raw/current/scripts/components/upgradeable.lua#L35-L36) [调用](../../../raw/current/scripts/components/upgradeable.lua#L75-L89) |
| `AdvanceStage()`／`onstageadvancefn` | stage+1、numupgrades=0，再 fn(inst)；返回回调结果，但 Upgrade 不拿此结果判成功。没有循环升级或溢出累计。[方法](../../../raw/current/scripts/components/upgradeable.lua#L51-L56) |
| `GetStage()`／`SetStage(num)` | 读／写当前阶段，SetStage 不清累计、不调阶段业务回调，不能替代宿主 SetStage 工厂。[方法](../../../raw/current/scripts/components/upgradeable.lua#L43-L56) |
| `upgradetype` | 与材料配对；赋值只有 CanUpgrade 真时才处理旧／新标签，满级时随意换类型可能留下旧标签，应审实际状态后变更。[标签实现](../../../raw/current/scripts/components/upgradeable.lua#L2-L14) |
| `OnSave()`／`OnLoad(data)` | 只保存累计和阶段；加载直接赋值，不调 onstageadvancefn／onupgradefn，不自动恢复外观或 numstages 等配置。[存读](../../../raw/current/scripts/components/upgradeable.lua#L96-L105) |

### 原蜘蛛巢的升级与成长协作

原蜘蛛巢业务门禁拒绝未剃除装饰的 bedazzled，并要求不在平台；onstageadvancefn 调 growable:DoGrowth。宿主自己的 SetStage 还改子体储备、生命、地图图标和 inst.data.stage，最后同步 upgradeable。仅写组件 stage 不会完成这些行为。[门禁与成长](../../../raw/current/scripts/prefabs/spiderden.lua#L548-L565) [宿主阶段](../../../raw/current/scripts/prefabs/spiderden.lua#L142-L160) [安装](../../../raw/current/scripts/prefabs/spiderden.lua#L785-L799)

## 典型接入

### 原蜘蛛巢每阶段只需三点升级累计

目标模组 `modmain.lua`，复用原资源与成长链；只改阈值，不覆盖 CanUpgrade、阶段回调或 growable 数据。与材料 upgradevalue 分开设计；原一件丝一点时通常需三件，若另改材料贡献则按实际值累计。[原安装](../../../raw/current/scripts/prefabs/spiderden.lua#L785-L799) [原累计](../../../raw/current/scripts/components/upgradeable.lua#L75-L92)

```lua
AddPrefabPostInit("spiderden", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.upgradeable ~= nil then
        inst.components.upgradeable.upgradesperstage = 3
    end
end)
```

已有累计不重置，也不因为阈值变小立刻升阶；下次正常 Upgrade 再判断。保留自然成长、装饰／平台限制及最大阶段，不能把三点阈值解释成固定生长秒数。[判断时机](../../../raw/current/scripts/components/upgradeable.lua#L75-L92)

阈值由 PostInit 重施，停用并重载恢复原默认；已经保存的阶段／累计不回滚。自然成长与升级业务的同步可参考 [[reference/components/growable]]、[[reference/components/childspawner]]；自定义目标还须设计自己的外观和保存恢复。

## 常见陷阱与限制

- **回调返回 false 能取消扣材料／推进阶段**：消费已经发生，Upgrade 忽略两种回调返回；资格放 CanUpgrade 或实际动作入口。[执行](../../../raw/current/scripts/components/upgradeable.lua#L75-L92)
- **SetStage 就是完整升阶／OnLoad 会重放业务**：组件只写值／标签；完整宿主恢复另有责任，不在阶段回调无条件发奖励。[接口](../../../raw/current/scripts/components/upgradeable.lua#L43-L56) [加载](../../../raw/current/scripts/components/upgradeable.lua#L103-L105)
- **调材料值与调阈值能随意叠加**：一次投入最多升一阶、溢出清零；跨模组同时改两端要一起计算实际消耗。[算法](../../../raw/current/scripts/components/upgradeable.lua#L51-L56) [累计](../../../raw/current/scripts/components/upgradeable.lua#L75-L92)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 3 处（actions.lua UPGRADE 与 spiderden.lua 安装段行号迁移，区间文本逐段比对一致；蜘蛛巢本版仅删一处无关行）；结论修订：无，仅锚点迁移。
- 依据：枢纽 diff 审阅（actions.lua／componentactions.lua hunk 地图）＋变更清单命中 actions.lua、prefabs/spiderden.lua。
- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖完整 upgradeable、材料匹配／动作、蜘蛛巢安装／业务门禁／阶段与成长协作。未完整审校所有目标、蜘蛛女王生成、巢装饰全部边界、自然成长保存或物品回调销毁后的容错。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整门槛例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；已有累计、溢出、最大阶段、装饰／平台、自然成长／手动升级交错和重载待实测。
