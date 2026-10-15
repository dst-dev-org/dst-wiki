---
title: "reticule"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "本地瞄准指示器的目标位置、颜色、输入与生命周期，不承担服务端授权或伤害判定。"
aliases: ["reticule","瞄准指示器","技能范围圈","手柄选点","瞄准颜色"]
tags: ["component","ui","callback","server-client"]
dst_build: "steam-buildid:26230496"
sources: ["components/reticule.lua","prefabs/slingshot.lua","components/playercontroller.lua","components/aoetargeting.lua","components/map.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# reticule

## 快速判断

- 职责与适用：本地鼠标／手柄目标点与视觉指示器，由玩家 controller 创建／销毁；地点技能通过 [[reference/components/aoetargeting]] 临时挂本组件。[components/reticule.lua:35](../../../raw/current/scripts/components/reticule.lua#L35)、[components/playercontroller.lua:1723](../../../raw/current/scripts/components/playercontroller.lua#L1723
- 优先使用：在 aoetargeting.reticule 表配 targetfn／颜色／位置函数，让已有 controller 驱动；不是服务端生成一个圈就有完整选点。[components/aoetargeting.lua:98](../../../raw/current/scripts/components/aoetargeting.lua#L98
- 重要副作用：CreateReticule 注册摄像机监听，鼠标模式还注册输入；DestroyReticule 清理两者及实体，移组件／删实体钩子均复用它。[components/reticule.lua:35](../../../raw/current/scripts/components/reticule.lua#L35)、[components/reticule.lua:72](../../../raw/current/scripts/components/reticule.lua#L72)、[components/reticule.lua:362](../../../raw/current/scripts/components/reticule.lua#L362
- 修改边界：依赖 TheInput／TheCamera 的本地表现；颜色合法只反映地图／validfn，不能代替 [[reference/components/aoespell]] 的服务端技能条件。[components/reticule.lua:159](../../../raw/current/scripts/components/reticule.lua#L159

## 关键接口

| 字段／接口 | 约定 | 条件、副作用与证据 |
|---|---|---|
| `reticuleprefab`／`pingprefab` | 默认 reticule／nil | 前者持续显示，后者显式 PingReticuleAt；不是攻击实体。[components/reticule.lua:7](../../../raw/current/scripts/components/reticule.lua#L7)、[components/reticule.lua:87](../../../raw/current/scripts/components/reticule.lua#L87 |
| `targetfn`／`mousetargetfn` | `(item)→Vector3`／`(item,mousepos)→Vector3?` | 鼠标映射 nil 会回落到原 mousepos，不能靠返回 nil 强制判无效。[components/reticule.lua:35](../../../raw/current/scripts/components/reticule.lua#L35)、[components/reticule.lua:209](../../../raw/current/scripts/components/reticule.lua#L209 |
| `validcolour`／`invalidcolour` | RGBA 四元素表 | currentcolour 按 Map:CanCastAtPoint 与 validfn 切换，再 UpdateColour；改表不改变真正技能授权。[components/reticule.lua:145](../../../raw/current/scripts/components/reticule.lua#L145)、[components/reticule.lua:159](../../../raw/current/scripts/components/reticule.lua#L159 |
| `validfn` | `(item,fx,targetpos,alwayspassable,allowwater,deployradius)→bool` | 地图通过后才调用，额外本地判断；不自动发给服务端。[components/reticule.lua:172](../../../raw/current/scripts/components/reticule.lua#L172 |
| `updatepositionfn` | `(item,pos,fx,ease,smoothing,dt)` | 持续更新传六参，Ping 路径只传前三参，函数需兼容可选参数。[components/reticule.lua:108](../../../raw/current/scripts/components/reticule.lua#L108)、[components/reticule.lua:198](../../../raw/current/scripts/components/reticule.lua#L198 |
| `mouseenabled`／`ease`／`smoothing` | false／false／6.66 | 鼠标开关、插值与系数，依赖 controller 实际创建条件。[components/reticule.lua:7](../../../raw/current/scripts/components/reticule.lua#L7)、[components/playercontroller.lua:1723](../../../raw/current/scripts/components/playercontroller.lua#L1723 |
| `twinstickmode`／`twinstickrange` | nil／nil | mode 1／2 以右摇杆偏移或方向插值，默认范围 8；是本地输入范围，不自动成为服务端射程。[components/reticule.lua:243](../../../raw/current/scripts/components/reticule.lua#L243)、[components/reticule.lua:305](../../../raw/current/scripts/components/reticule.lua#L305 |
| `CreateReticule()`／`DestroyReticule()` | 创建／清理 | controller 在装备／刷新选择实例，不需每帧重建；Blip 只做 alpha 闪烁。[components/reticule.lua:35](../../../raw/current/scripts/components/reticule.lua#L35)、[components/reticule.lua:72](../../../raw/current/scripts/components/reticule.lua#L72)、[components/reticule.lua:126](../../../raw/current/scripts/components/reticule.lua#L126 |

## 典型接入

### 双弹弓合法瞄准颜色改为浅蓝

目标模组 `modmain.lua`；在原版 slingshot2ex 的 aoetargeting 配置表初始化时改颜色，双端执行且不新增资源。该工厂已有 reticulelong、鼠标映射与方向更新，保留这些函数。[prefabs/slingshot.lua:765](../../../raw/current/scripts/prefabs/slingshot.lua#L765

```lua
AddPrefabPostInit("slingshot2ex", function(inst)
    local targeting = inst.components.aoetargeting
    if targeting ~= nil then
        targeting.reticule.validcolour = { 0.2, 0.8, 1, 1 }
    end
end)
```

颜色分量为 0～1；新建本地 reticule 时复制配置。此例不修改无效红色、地图资格、伤害范围或服务端门禁，也不承诺热加载修改已存在实例；停用后重新生成／载入恢复工厂值。[components/aoetargeting.lua:98](../../../raw/current/scripts/components/aoetargeting.lua#L98)、[components/reticule.lua:159](../../../raw/current/scripts/components/reticule.lua#L159

## 常见陷阱与限制

- **挂 reticule 会始终显示**：controller 还看 mouseenabled／手柄、启用与 ShouldHide；以实际本地刷新入口为准。[components/playercontroller.lua:1723](../../../raw/current/scripts/components/playercontroller.lua#L1723)、[components/reticule.lua:358](../../../raw/current/scripts/components/reticule.lua#L358
- **直接改 targetpos 就不再被输入覆盖**：摄像机更新可能重新调用鼠标／targetfn；应改目标计算源。[components/reticule.lua:209](../../../raw/current/scripts/components/reticule.lua#L209
- **给 updatepositionfn 假定 dt 非 nil**：首次更新和 Ping 不传完整时间参数；否则首次选点便可能报错。[components/reticule.lua:108](../../../raw/current/scripts/components/reticule.lua#L108)、[components/reticule.lua:198](../../../raw/current/scripts/components/reticule.lua#L198
- **只删 FX 无需清监听**：保留输入／摄像机监听会继续更新，使用 DestroyReticule 或现有移除钩子。[components/reticule.lua:72](../../../raw/current/scripts/components/reticule.lua#L72)、[components/reticule.lua:362](../../../raw/current/scripts/components/reticule.lua#L362

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 0 处（playercontroller#L1723 引用区间未变）；结论修订：无。
- 依据：player-hubs 辅组（C-27）＋变更清单命中 components/playercontroller.lua。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整 reticule、targeting 复制／移除、controller 刷新及双弹弓目标／颜色工厂；未审校所有 UI 遮挡、底层输入或 reticule 资源。
- 结构检查：发布时核对格式、链接／锚点、大小及唯一登记；完整示例作 Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；鼠标／手柄切换、浅蓝效果、平台、相机、销毁及多人表现待测。

相关：[[reference/components/aoetargeting]]；[[reference/components/aoespell]]；[[reference/components/aoecharging]]。
