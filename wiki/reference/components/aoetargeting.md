---
title: "aoetargeting"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "双端地点技能的启用状态、瞄准参数与本地 reticule 创建，区别最终施法验证。"
aliases: ["aoetargeting","地点瞄准","技能选点","禁止骑乘施法"]
tags: ["component","network","server-client","ui"]
dst_build: "steam-buildid:26230496"
sources: ["components/aoetargeting.lua","components/aoespell.lua","components/playercontroller.lua","components/map.lua","prefabs/slingshot.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# aoetargeting

## 快速判断

- 职责与适用：保存地点瞄准配置，以 enabled 网络值控制可用性，本地 StartTargeting 临时挂 reticule；实际效果由 [[reference/components/aoespell]] 执行。[components/aoetargeting.lua:17](../../../raw/current/scripts/components/aoetargeting.lua#L17)、[components/aoetargeting.lua:98](../../../raw/current/scripts/components/aoetargeting.lua#L98
- 优先使用：新工厂双端、SetPristine 前挂组件并填 reticule 表；动态禁用在服务端 SetEnabled，固定骑乘／地图参数双端一致设置。[prefabs/slingshot.lua:765](../../../raw/current/scripts/prefabs/slingshot.lua#L765)、[prefabs/slingshot.lua:519](../../../raw/current/scripts/prefabs/slingshot.lua#L519
- 重要副作用：禁用会停止本地瞄准并刷新玩家指示器；StartTargeting 将配置浅复制到新 reticule，不是持续绑定配置表。[components/aoetargeting.lua:1](../../../raw/current/scripts/components/aoetargeting.lua#L1)、[components/aoetargeting.lua:98](../../../raw/current/scripts/components/aoetargeting.lua#L98
- 修改边界：StartTargeting／StopTargeting 使用 ThePlayer，是本地玩家流程，不在独立服务器随意调用；只把组件挂服务端会缺客户端状态。[components/aoetargeting.lua:98](../../../raw/current/scripts/components/aoetargeting.lua#L98)、[components/aoetargeting.lua:114](../../../raw/current/scripts/components/aoetargeting.lua#L114

## 关键接口

| 接口／字段 | 默认／约定 | 条件、副作用与证据 |
|---|---|---|
| `SetEnabled(bool)`／`IsEnabled()` | 默认 true；服务端写网络值 | 关闭调用 OnEnabledDirty，刷新本地主人；不删除技能效果或自动清冷却。[components/aoetargeting.lua:40](../../../raw/current/scripts/components/aoetargeting.lua#L40)、[components/aoetargeting.lua:51](../../../raw/current/scripts/components/aoetargeting.lua#L51 |
| `reticule` 配置表 | prefab／颜色／targetfn 等 | StartTargeting 挂 reticule 时逐键赋值；字段语义见 [[reference/components/reticule#关键接口]]。[components/aoetargeting.lua:19](../../../raw/current/scripts/components/aoetargeting.lua#L19)、[components/aoetargeting.lua:98](../../../raw/current/scripts/components/aoetargeting.lua#L98 |
| `SetAllowRiding(val)` | 默认 true；仅 false 禁止 | playercontroller／aoespell 读取；普通字段不自动网络同步，配置两端保持一致。[components/aoetargeting.lua:70](../../../raw/current/scripts/components/aoetargeting.lua#L70)、[components/aoespell.lua:52](../../../raw/current/scripts/components/aoespell.lua#L52 |
| `SetAlwaysValid(val)`／`SetAllowWater(val)` | 默认 false／false | 只影响地图位置资格；不保证施法对象、距离或效果成功。[components/aoetargeting.lua:62](../../../raw/current/scripts/components/aoetargeting.lua#L62)、[components/map.lua:1101](../../../raw/current/scripts/components/map.lua#L1101 |
| `SetDeployRadius(radius)` | 默认 0 | 地图净空检查半径，不是技能范围伤害半径。[components/aoetargeting.lua:82](../../../raw/current/scripts/components/aoetargeting.lua#L82)、[components/map.lua:1101](../../../raw/current/scripts/components/map.lua#L1101 |
| `SetRange(range)`／`GetRange()` | 默认 8 | 这里仅存取字段；已核验的 aoespell／reticule 检查不据此强制服务端射程，实际消费者须另查。[components/aoetargeting.lua:74](../../../raw/current/scripts/components/aoetargeting.lua#L74)、[components/aoespell.lua:26](../../../raw/current/scripts/components/aoespell.lua#L26 |
| `SetShouldRepeatCastFn(fn)` | `(item,doer) → bool` | CanRepeatCast 只检查函数存在，ShouldRepeatCast 才调用，不等于自动重复施法。[components/aoetargeting.lua:86](../../../raw/current/scripts/components/aoetargeting.lua#L86 |
| `SetTargetFX(prefab)`／`SpawnTargetFXAt(pos)` | Vector3 或 DynamicPosition | 显式调用才生成，平台位置按父子关系处理；不是实际伤害效果。[components/aoetargeting.lua:123](../../../raw/current/scripts/components/aoetargeting.lua#L123 |

## 典型接入

### 双弹弓保留地点技能但禁止骑乘瞄准

目标模组 `modmain.lua`，复用原版 slingshot2ex，不增加网络变量／资源。此配置必须双端设置，所以没有 ismastersim 提前返回；服务端 CanCast 与本地 playercontroller 都消费 allowriding。[prefabs/slingshot.lua:770](../../../raw/current/scripts/prefabs/slingshot.lua#L770)、[components/playercontroller.lua:1609](../../../raw/current/scripts/components/playercontroller.lua#L1609)、[components/aoespell.lua:63](../../../raw/current/scripts/components/aoespell.lua#L63

```lua
AddPrefabPostInit("slingshot2ex", function(inst)
    local targeting = inst.components.aoetargeting
    if targeting ~= nil then
        targeting:SetAllowRiding(false)
    end
end)
```

仅禁止对应地点模式，不关闭普通射击；保留原版弹药／技能树驱动的 SetEnabled，不强行解锁能力。停用后新建／载入恢复默认骑乘许可。[prefabs/slingshot.lua:818](../../../raw/current/scripts/prefabs/slingshot.lua#L818

## 常见陷阱与限制

- **enabled 网络同步覆盖所有参数**：只有 enabled 是 net_bool，reticule／range／骑乘等普通字段不自动同步；静态配置双端初始化，动态变化另设计。[components/aoetargeting.lua:17](../../../raw/current/scripts/components/aoetargeting.lua#L17
- **改配置表即刷新已有 reticule**：它是创建时浅复制；运行中修改先明确当前 reticule 实例与重建流程，不能只改表就称已生效。[components/aoetargeting.lua:98](../../../raw/current/scripts/components/aoetargeting.lua#L98
- **SetAlwaysValid 取消全部验证**：地图净空半径仍检查，spellbook 权限／骑乘／技能门禁也独立存在。[components/map.lua:1101](../../../raw/current/scripts/components/map.lua#L1101)、[components/aoespell.lua:26](../../../raw/current/scripts/components/aoespell.lua#L26
- **独服调用 StartTargeting**：ThePlayer 可能不存在；让本地 controller 驱动，服务端只授权和执行效果。[components/aoetargeting.lua:98](../../../raw/current/scripts/components/aoetargeting.lua#L98

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 3 处（components/map.lua#L1068→L1101，区间文本比对一致，显示标签同步）；结论修订：无。C-23 新增 CanPointHaveRain 系 API 属雨湿门控，不触及本页 CanPlantAtPoint 施法资格结论；playercontroller C-27 两处改动（:447/:2034）不在引用区间。
- 依据：player-hubs 辅组（C-27）＋global-hubs（C-23）＋变更清单命中 components/map.lua、components/playercontroller.lua。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整 aoetargeting、必要 controller／aoespell／Map 消费、双弹弓共同工厂和启用规则。range 的其他潜在消费者未全库审校。
- 结构检查：发布时检查格式、链接、源码锚点和登记；完整示例作 Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；骑乘切换、手柄／鼠标、平台、重建和多人预测待实测。

相关：[[reference/components/reticule]]；[[reference/components/aoespell]]；[[reference/components/aoecharging]]。
