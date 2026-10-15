---
title: "staffsanity"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "在施法者上缩放经 DoCastingDelta 接入的精神变化，保留法杖各自的扣费时机。"
aliases: ["staffsanity", "法杖精神消耗倍率", "施法精神减耗", "旺达施法减耗"]
tags: [component, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/staffsanity.lua", "prefabs/staff.lua", "prefabs/staff_common.lua", "prefabs/wanda.lua", "prefabs/wilson.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# staffsanity

## 快速判断

- 职责与适用：挂在施法者上，把传入变化量乘倍率，再交给同实体的 sanity；不是武器组件，也不会自动监听施法。[完整实现](../../../raw/current/scripts/components/staffsanity.lua#L1-L15)
- 优先使用：已走 `DoCastingDelta` 的法杖减耗；原版火杖攻击和橙杖传送先找攻击者／施法者的此组件，否则直接调用 sanity。[火杖](../../../raw/current/scripts/prefabs/staff_common.lua#L63-L70)、[橙杖](../../../raw/current/scripts/prefabs/staff_common.lua#L479-L492)
- 重要副作用：正数恢复和负数消耗均会缩放；组件不管施法成功、次数或耐久，橙杖另外扣一次 finiteuses。[结算](../../../raw/current/scripts/components/staffsanity.lua#L9-L13)、[宿主](../../../raw/current/scripts/prefabs/staff_common.lua#L479-L492)
- 修改边界：在服务端施法者上配置；旺达原版在 `master_postinit` 添加。一般精神变化仍用 [[reference/components/sanity#关键接口]]，不经过本组件。[旺达挂载](../../../raw/current/scripts/prefabs/wanda.lua#L390-L403)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `SetMultiplier(mult)` | 数值倍率；调用者保证类型 | 直接替换 `multiplier`，不累乘；构造没有初始化该字段。结算时 nil／false 回退 1，数值 0 生效。[实现](../../../raw/current/scripts/components/staffsanity.lua#L1-L13) |
| `DoCastingDelta(amount)` | 数值精神点变化；无返回值 | 仅存在 sanity 才计算 `amount * (multiplier or 1)` 并调用 `sanity:DoDelta`；无 sanity 就无效果，不返回实际扣费结果。[实现](../../../raw/current/scripts/components/staffsanity.lua#L9-L13) |

组件没有任务、事件监听、`OnSave`、`OnLoad` 或移除钩子；倍率需由构造／宿主重新配置，组件本身不保存。旺达年龄切换会再次写入倍率，不能依赖一次赋值永久覆盖。[组件全文](../../../raw/current/scripts/components/staffsanity.lua#L1-L15)、[老年](../../../raw/current/scripts/prefabs/wanda.lua#L90-L96)、[中年](../../../raw/current/scripts/prefabs/wanda.lua#L125-L133)、[青年](../../../raw/current/scripts/prefabs/wanda.lua#L155-L163)

## 典型接入

### 威尔逊经本接口结算的施法变化减半

完整目标文件为模组 `modmain.lua`，普通生存模式、服务端；无需资源。给 `wilson` 施法者补组件，再包装原方法，不改法杖回调或既有倍率。`AddPrefabPostInit` 按注册名接入。[人物注册](../../../raw/current/scripts/prefabs/wilson.lua#L170)、[注册接口](../../../raw/current/scripts/modutil.lua#L592-L599)、[原版调用链](../../../raw/current/scripts/prefabs/staff_common.lua#L479-L492)

```lua
AddPrefabPostInit("wilson", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.sanity == nil then return end
    if inst.components.staffsanity == nil then
        inst:AddComponent("staffsanity")
    end
    local component = inst.components.staffsanity
    local old = component.DoCastingDelta
    component.DoCastingDelta = function(self, amount)
        return old(self, amount * 0.5)
    end
end)
```

`0.5` 是额外缩放因子，保留原方法与当前 multiplier；正向恢复也减半。仅影响此人物实例经该接口提交的变化，直接 `sanity:DoDelta` 的技能不受影响；不会减少法杖耐久。对新建／读档重建的实例重新应用；无额外任务需清理。撤销示例后重新载入世界，恢复原构造，已发生的精神变化不回滚。若做临时效果，应另行成对恢复本次包装，并处理其他模组后续包装，不能简单把倍率设成 1。

### 自制法杖采用同一人物倍率

在服务端已确认施法成功的回调中，照橙杖分支对 `caster.components.staffsanity` 调用负变化量，无此组件时才回退 sanity；消耗次数仍由物品独立处理。不要在预览和成功回调各扣一次，也不要只往物品添加组件。[橙杖完整扣费分支](../../../raw/current/scripts/prefabs/staff_common.lua#L479-L492)

## 常见陷阱与限制

- **以为添加组件就覆盖全部法术**：它没有事件钩子，只响应显式调用；新法术需要自己接入上述分支。[组件全文](../../../raw/current/scripts/components/staffsanity.lua#L1-L15)
- **以为 `DoCastingDelta` 成功返回代表可施法**：无返回／余额判断；sanity 的限制与事件仍由原组件负责，见 [[reference/components/sanity#关键接口]]。
- **在旺达 post-init 固定倍率**：后续年龄切换会重写；额外折扣可包装结算入口，保留原版年龄倍率，勿替换年龄函数。[三年龄写入](../../../raw/current/scripts/prefabs/wanda.lua#L90-L96)、[中年](../../../raw/current/scripts/prefabs/wanda.lua#L125-L133)、[青年](../../../raw/current/scripts/prefabs/wanda.lua#L155-L163)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 3 处（火杖 staff.lua#L64-L71→staff_common.lua#L63-L70)、橙杖 #L414-L424→staff_common.lua#L479-L492)、wilson#L171→L170）；结论修订：无。P-112 迁移后 staffsanity 优先／sanity 回退分支逐字一致，橙杖 finiteuses 扣次保留（新 rechargeable 分支为王杖共用）；prefabs/staff_common.lua 已加入 sources；wanda 四处引用区间未变。
- 依据：报告 P-112（detail-prefabs.md:174）＋变更清单命中 prefabs/staff.lua、prefabs/staff_common.lua、prefabs/wanda.lua、prefabs/wilson.lua。

- 源码审阅：2026-10-22，Steam buildid 25643519；完整组件、旺达服务端挂载／年龄重设、火杖与橙杖调用及独立次数消耗、post-init 注册已核验；不声称覆盖所有法术消费者。
- 结构检查：本页元数据、大小、本地链接和源码行锚点通过；完整 `modmain.lua` 经 Lua 5.1 语法解析通过。
- 游戏运行：未运行；实际施法扣费、读档、多模组包装顺序待实测。
