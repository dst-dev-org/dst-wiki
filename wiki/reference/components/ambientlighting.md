---
title: "ambientlighting"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界环境光组件：按昼夜相位/季节/满月/洞穴插值 SetAmbientColour，叠加天气亮度、夜视视觉通道与闪电屏幕闪光。"
aliases: ["ambientlighting", "环境光", "环境颜色", "屏幕闪光"]
tags: [component, mechanism, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/ambientlighting.lua", "prefabs/cave.lua", "prefabs/forest.lua", "prefabs/moon_fissure.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# ambientlighting

## 快速判断

- 职责与适用：世界组件（森林、洞穴、熔炉、暴食均装；洞穴在世界初始化早期安装，注释标明“light watchers 需要”），把相位/季节/月亮/天气换算为两条环境色通道并调 `TheSim:SetAmbientColour`（真实光）与 `SetVisualAmbientColour`（视觉覆盖）。颜色表全部是文件局部常量，不能运行时改表。[安装](../../../raw/current/scripts/prefabs/cave.lua#L333-L339) [森林安装](../../../raw/current/scripts/prefabs/forest.lua#L622) [颜色表](../../../raw/current/scripts/components/ambientlighting.lua#L11-L47)
- 优先使用：固定世界色调用 `overrideambientlighting` 事件（原版熔炉/暴食写法，但源码注释标记 NOT safe）；触发屏幕闪光用 `screenflash` 事件；查询当前视觉亮度供消费用 `GetVisualAmbientValue()`。[覆盖事件](../../../raw/current/scripts/components/ambientlighting.lua#L252-L259) [亮度查询](../../../raw/current/scripts/components/ambientlighting.lua#L289-L292) [消费示例](../../../raw/current/scripts/prefabs/moon_fissure.lua#L49)
- 重要副作用：双通道独立插值，夜视玩家会把**视觉通道**换成灰白 `NIGHTVISION_COLOURS`（由 [[reference/components/playervision]] 的 `nightvision`/`nightvisionambientoverrides` 事件驱动）；真实通道不受夜视影响。[夜视切换](../../../raw/current/scripts/components/ambientlighting.lua#L179-L194)
- 修改边界：更新自限——仅插值/闪光期间 StartUpdating，完成即停；LongUpdate 直接吸附到目标色。无 OnSave/OnLoad（视觉态每次重算）。[自限更新](../../../raw/current/scripts/components/ambientlighting.lua#L338-L347) [快进](../../../raw/current/scripts/components/ambientlighting.lua#L349-L359)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `phasechanged`（世界事件） | 换相位颜色：洞穴用 `CAVE_COLOUR`，满月夜用 `FULL_MOON_COLOUR`，否则 `PHASE_COLOURS[季节][相位]`（缺季节回落 default），过渡时长取表内 time（4/6/8 秒）。[处理](../../../raw/current/scripts/components/ambientlighting.lua#L155-L162) [选色](../../../raw/current/scripts/components/ambientlighting.lua#L125-L149) |
| `moonphasechanged2`（世界事件） | 夜间满月状态变化时重算（先按相位判断，避免多余插值）。[处理](../../../raw/current/scripts/components/ambientlighting.lua#L164-L169) |
| `weathertick`（世界事件） | 用天气 `light` 更新两通道 `lightpercent` 乘数并立即推送颜色（闪光期间跳过）。[处理](../../../raw/current/scripts/components/ambientlighting.lua#L171-L177) |
| `screenflash`（世界事件） | data 为强度数值：瞬时把两通道拉到闪光灰度（按玩家 Profile 的 `SCREEN_FLASH_SCALING` 缩放），保持 8 帧后 0.5 秒过渡恢复；插值中的过渡被截断。[实现](../../../raw/current/scripts/components/ambientlighting.lua#L201-L224) [闪光更新](../../../raw/current/scripts/components/ambientlighting.lua#L319-L336) |
| `nightvision`／`nightvisionambientoverrides`（激活玩家事件） | 切换视觉通道颜色表（夜视灰白表或自定义 ambienttable），0.25 秒过渡；只在激活玩家变更时重挂监听。[处理](../../../raw/current/scripts/components/ambientlighting.lua#L179-L194) [挂载](../../../raw/current/scripts/components/ambientlighting.lua#L235-L246) |
| `overrideambientlighting`（世界事件） | data 为 Point 颜色或 nil：设固定覆盖色，**优先级高于洞穴/满月/相位全部规则**，0 秒过渡。源码注释标记“NOT safe to use!”。[处理](../../../raw/current/scripts/components/ambientlighting.lua#L252-L259) [注册与警告](../../../raw/current/scripts/components/ambientlighting.lua#L282) |
| `GetVisualAmbientValue()` | 返回视觉通道当前平均亮度（0～1 量级），原版月缝用它按世界亮度调特效。[实现](../../../raw/current/scripts/components/ambientlighting.lua#L289-L292) [消费者](../../../raw/current/scripts/prefabs/moon_fissure.lua#L49) |
| `LongUpdate(dt)` | 世界快进时直接吸附目标色并停止更新。[实现](../../../raw/current/scripts/components/ambientlighting.lua#L349-L359) |

### 双通道与选色优先级

真实通道（SetAmbientColour）按“固定覆盖 > 洞穴 > 满月 > 相位表”选色再乘天气 light；视觉通道（SetVisualAmbientColour）同样规则但颜色集可被夜视换成灰白表或自定义表（[推送实现](../../../raw/current/scripts/components/ambientlighting.lua#L115-L123)、[选色实现](../../../raw/current/scripts/components/ambientlighting.lua#L125-L149)）。表面世界才监听 `seasontick`（洞穴固定无季节表）（[注册](../../../raw/current/scripts/components/ambientlighting.lua#L276-L278)）。

## 典型接入

### 固定或调整世界环境色

- 前提与选择：客户端才有渲染效果（组件专用服务器上也运行，颜色调用无视觉）。想整体定调走 override 事件（带源码警告）；只想瞬时效果用 screenflash。这是事件调用指引，不改颜色常量本身。[警告注释](../../../raw/current/scripts/components/ambientlighting.lua#L282)
- 生效与撤销：override 传 nil 恢复常规规则；同色重复推送被去重（[去重](../../../raw/current/scripts/components/ambientlighting.lua#L253)）。

```lua
-- 客户端语义：固定世界环境色（原版熔炉/暴食世界同款写法）
if TheWorld.components.ambientlighting ~= nil then
    TheWorld:PushEvent("overrideambientlighting", Point(200 / 255, 200 / 255, 200 / 255))
end
-- 恢复常规昼夜/洞穴颜色：
-- TheWorld:PushEvent("overrideambientlighting", nil)
```

- 调整与验证：Point 分量为 0～1 RGB；验证画面环境色立即变化、且不受天气 tick 影响（覆盖分支跳过 lightpercent 以外逻辑，见 [选色](../../../raw/current/scripts/components/ambientlighting.lua#L126-L139)）。注意源码对该事件的“NOT safe”标注：与其他改色手段叠加时行为未定义。

## 常见陷阱与限制

- **以为改 NORMAL_COLOURS 就能调色：** 颜色表是构造前局部常量，构造后无任何 setter；覆盖途径只有 override 事件（全局单色）与夜视 ambienttable（视觉通道），不能按相位/季节定制。[常量局部](../../../raw/current/scripts/components/ambientlighting.lua#L11-L47)
- **override 会压过洞穴与满月：** 固定覆盖在选色第一优先级，洞穴世界用它定调后 CAVE_COLOUR 失效；撤销必须显式传 nil。[优先级](../../../raw/current/scripts/components/ambientlighting.lua#L126-L128)
- **screenflash 的 data 是数值不是表：** 推送 `{intensity=0.5}` 之类的表会把 `_flashintensity` 设为表对象，后续运算报错；必须直接推数字。[data 用法](../../../raw/current/scripts/components/ambientlighting.lua#L201-L204)
- **闪光明暗与玩家设置耦合：** 强度按本地 Profile 的屏幕闪光缩放设置折算，不同客户端观感不同；该通道纯客户端。[缩放](../../../raw/current/scripts/components/ambientlighting.lua#L88) [重载时机](../../../raw/current/scripts/components/ambientlighting.lua#L261-L263)
- **夜视自定义表需要 fixedcolour 才免疫天气：** ambienttable 带 `fixedcolour` 时视觉通道 lightpercent 固定 1，否则仍随天气变暗；字段缺失不会报错但效果不同。[表约定](../../../raw/current/scripts/components/ambientlighting.lua#L187-L193)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 ambientlighting.lua，核验双通道模型、选色优先级、事件接线（含 playervision 联动与 override 警告）、闪光状态机与 moon_fissure 消费点；核对四处世界安装。未运行游戏；渲染端（TheSim C++）实际效果不在脚本核验范围。

相关：[[reference/components/playervision]]、[[reference/components/dsp]]。
