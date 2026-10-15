---
title: "upgrader"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "通用升级材料的类型、单位升级值和执行者标签检查；消费与阶段累计由 upgradeable 处理。"
aliases: ["蜘蛛丝每件提供两点升级值", "升级材料类型", "upgrader"]
tags: [component, item, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/upgrader.lua", "components/upgradeable.lua", "prefabs/silk.lua", "prefabs/webber.lua", "componentactions.lua", "actions.lua", "constants.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# upgrader

## 快速判断

- 职责与适用：材料侧保存 upgradetype／upgradevalue，检查目标类型及 doer 的 `<type>_upgradeuser` 标签。材料不是 WX-78 安装模块，后者见 [[reference/components/upgrademodule]]。[定义](../../../raw/current/scripts/components/upgrader.lua#L10-L28)
- 优先使用：改一次材料投入的升级量用 upgradevalue；配 [[reference/components/upgradeable]]，保留其最终 CanUpgrade 门禁及消费逻辑，不再手工 Remove 材料。[目标执行](../../../raw/current/scripts/components/upgradeable.lua#L75-L92)
- 重要副作用：类型赋值维护 `<type>_upgrader` 标签；CanUpgrade 本身不消费，也不检查目标是否满级／业务条件，动作在它通过后再检查目标。[标签](../../../raw/current/scripts/components/upgrader.lua#L1-L7) [动作](../../../raw/current/scripts/actions.lua#L4056-L4067)
- 修改边界：服务端，目标和 doer 均须符合原接口前提；方法直接访问 target.components.upgradeable 和 doer，不是对任意实体的安全探测函数。[方法](../../../raw/current/scripts/components/upgrader.lua#L21-L28)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `upgradetype` | 默认 UPGRADETYPES.DEFAULT，赋值替换旧／新类型标签；候选枚举 UPGRADETYPES，任意新字符串不自动获得原动作支持。[默认](../../../raw/current/scripts/components/upgrader.lua#L10-L18) [枚举](../../../raw/current/scripts/constants.lua#L1969-L1977) [候选](../../../raw/current/scripts/componentactions.lua#L1888-L1896) |
| `upgradevalue` | 默认 1，每次 Upgrade 消费一件时给目标加该值，**不按原材料整堆数量乘算**。组件不验证正数／整数，模组应自行约束。[默认](../../../raw/current/scripts/components/upgrader.lua#L13-L14) [消费](../../../raw/current/scripts/components/upgradeable.lua#L75-L85) |
| `CanUpgrade(target,doer)` | 仅检查类型相等、doer 有对应 upgradeuser 标签；不读取材料自定义 canupgradefn。原 silk 虽赋该字段，不能据赋值宣称额外回调生效。[方法](../../../raw/current/scripts/components/upgrader.lua#L21-L28) [原赋值](../../../raw/current/scripts/prefabs/silk.lua#L43-L45) |

原蜘蛛丝用 SPIDER，Webber 同时有 spiderwhisperer 与 SPIDER 对应 upgradeuser 标签；当前材料组件实际读后者。要增加新的执行资格应包真实 CanUpgrade 或目标入口，不能只照抄丝的未消费字段。[材料](../../../raw/current/scripts/prefabs/silk.lua#L43-L45) [人物](../../../raw/current/scripts/prefabs/webber.lua#L38-L42)

## 典型接入

### 蜘蛛丝每件提供两点升级值，保留原类型与角色门禁

目标模组 `modmain.lua`；沿用原 silk 资源／库存／堆叠，不变更所有目标的阶段阈值。设置影响该材料用于原通用升级动作的贡献，不改丝的交易或其他用途。[原组件](../../../raw/current/scripts/prefabs/silk.lua#L35-L45) [实际读取](../../../raw/current/scripts/components/upgradeable.lua#L75-L85)

```lua
AddPrefabPostInit("silk", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.upgrader ~= nil then
        inst.components.upgrader.upgradevalue = 2
    end
end)
```

一件丝给两点，但每次 Upgrade 仍只扣一件；目标默认阈值五点时第三件达六点，会只升一级并清累计，溢出一点不结转。不要用“翻倍升级值”承诺每次动作升两级或整堆自动升级。[累计／消费](../../../raw/current/scripts/components/upgradeable.lua#L75-L92) [清零](../../../raw/current/scripts/components/upgradeable.lua#L51-L56)

该字段无组件存读，工厂／PostInit 重建时重设；停用并重载恢复默认，目标已有累计和阶段不会回滚。新增自定义材料的网络标签、注册与资源仍须另核验；不能把此原物品修改当完整新材料工程。

## 常见陷阱与限制

- **CanUpgrade=true 表示目标可升且已完成升级**：材料只做两项匹配，目标 CanUpgrade 和最终 Upgrade 是后续步骤。[完整动作](../../../raw/current/scripts/actions.lua#L4056-L4067)
- **canupgradefn 是本组件扩展回调**：当前完整实现没读取它；应对实际被调用的方法扩展，并保持返回和目标业务检查。[完整组件](../../../raw/current/scripts/components/upgrader.lua#L1-L31)
- **升级值增大可跨多级／保留余点**：目标只做一次 AdvanceStage，随后计数清零；想改升级模型须审目标而非盲增材料值。[目标](../../../raw/current/scripts/components/upgradeable.lua#L51-L56) [阈值](../../../raw/current/scripts/components/upgradeable.lua#L88-L92)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 4 处（actions UPGRADE／componentactions 候选／constants 枚举行号迁移，区间文本逐段比对一致；UPGRADETYPES 与收集语义未变）；结论修订：无，仅锚点迁移。
- 依据：枢纽 diff 审阅（actions.lua／componentactions.lua hunk 地图）＋变更清单命中 actions.lua、componentactions.lua、constants.lua。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖完整 upgrader、目标累计／消费、UPGRADE 执行／候选、类型常量、丝与 Webber 的标签／配置。未全面审校全部材料、目标宿主、定制类型复制或多人并发。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整贡献例经已有 Full Moon Lua 5.1 syntax-only，结果见本轮报告。
- 游戏运行：未运行；单件／整堆、角色门禁、跨阈值溢出、满级／业务拒绝和保存恢复待实测。
