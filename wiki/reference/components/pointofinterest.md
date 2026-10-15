---
title: "pointofinterest"
created: "2026-10-27"
updated: "2026-10-28"
type: component
summary: "依据本地 Scrapbook 认识等级显示兴趣点世界标记与 HUD 方向提示，并在已检查后播放移除脉冲。"
aliases: ["兴趣点问号提示", "图鉴未检查地点标记", "pointofinterest"]
tags: [component, ui, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/pointofinterest.lua", "scrapbookpartitions.lua", "screens/redux/scrapbookdata.lua", "screens/playerhud.lua", "widgets/targetindicator.lua", "playerprofile.lua", "tuning.lua", "prefabs/pigking.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# pointofinterest

## 快速判断

- 职责与适用：为 Scrapbook 数据集已有、当地玩家认识等级小于 2 的宿主提供世界问号和 HUD 方向提示。它不写入图鉴进度，也不负责创建小地图图标。[启动门槛](../../../raw/current/scripts/components/pointofinterest.lua#L82-L89) [逐帧消费](../../../raw/current/scripts/components/pointofinterest.lua#L238-L278)
- 优先使用：沿用原版兴趣点的本地提示时配置 `SetHeight`、必要的 `SetShouldShowFn`；先确认 `scrapbook_proxy`／prefab 对应数据项。[配置](../../../raw/current/scripts/components/pointofinterest.lua#L93-L99) [名称重定向](../../../raw/current/scripts/scrapbookpartitions.lua#L139-L145)
- 重要副作用：辅助实体登记进 HUD 的 POI 列表；方向头像另经 TargetIndicator 列表管理。认识等级达到 2 后，方向提示撤除并播放世界标记缩放、圆环和音效。[本地实体](../../../raw/current/scripts/components/pointofinterest.lua#L13-L45) [移除入口](../../../raw/current/scripts/components/pointofinterest.lua#L134-L153)
- 修改边界：原版 `pigking` 在非专服环境、客户端返回之前添加组件；组件创建的辅助实体没有 Network。它依赖当地 `ThePlayer`、HUD、Profile 和 Scrapbook 分区，不是全服同步的通用地图标记系统。[宿主](../../../raw/current/scripts/prefabs/pigking.lua#L671-L684) [实体构造](../../../raw/current/scripts/components/pointofinterest.lua#L13-L34)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `TryStartUpdating()` | 构造时排零延迟任务调用；仅重定向后的名称存在于数据集且等级 `< 2` 才 `StartUpdatingComponent`。不符合时不启动，也不主动清理已存在表现。[构造与启动](../../../raw/current/scripts/components/pointofinterest.lua#L65-L89) |
| `SetShouldShowFn(fn)` | 保存 `fn(inst)`，缺省 nil 视为允许；只参与 OnUpdate 的主分支门槛，不代替启动门槛，也不是一变 false 就清除全部指示器。[设置](../../../raw/current/scripts/components/pointofinterest.lua#L93-L95) [消费者](../../../raw/current/scripts/components/pointofinterest.lua#L248-L272) |
| `SetHeight(height)` | 缺省 0；用作 marker 与圆环跟随 stand 的 `marker` 符号时的 Y 偏移，不是地图坐标或距离阈值。创建后修改字段不会重新执行已有标记的 FollowSymbol。[跟随](../../../raw/current/scripts/components/pointofinterest.lua#L120-L132) [圆环跟随](../../../raw/current/scripts/components/pointofinterest.lua#L221-L229) |
| `CreateWorldIndicator()` | `stand` 已存在或正在移除时直接返回；否则创建 stand 子实体及跟随其符号的 marker，辅助实体默认缩放 2。只创建表现，不检查图鉴等级。[构造](../../../raw/current/scripts/components/pointofinterest.lua#L13-L61) [入口](../../../raw/current/scripts/components/pointofinterest.lua#L120-L132) |
| `ShouldShowHudIndicator(distsq)` | 虽参数名为 distsq，真实 OnUpdate 传入的是 `sqrt(GetDistanceSqToInst)`；比较普通世界距离，含端点。此快照阈值为 20～50。[调用](../../../raw/current/scripts/components/pointofinterest.lua#L254-L265) [比较](../../../raw/current/scripts/components/pointofinterest.lua#L179-L181) [常量](../../../raw/current/scripts/tuning.lua#L3092-L3093) |
| `RemoveHudIndicator()` | 仅当 `_showinghud` 为真且当地 HUD 存在时清标志并 `RemoveTargetIndicator(inst)`；不删除 stand／marker。[实现](../../../raw/current/scripts/components/pointofinterest.lua#L113-L118) |
| `TriggerRemove()`／`TriggerPulse()` | 前者先撤方向提示，再启动后者；没有 marker 则脉冲无效果。脉冲设 `_removing=true`，播放音效及 dark 动画，后续更新完成两轮圆环／缩放后清理。[入口](../../../raw/current/scripts/components/pointofinterest.lua#L134-L153) [动画更新](../../../raw/current/scripts/components/pointofinterest.lua#L200-L236) |
| `RemoveEverything()` | 停组件更新、复位 `_removing`，删除 ring1、ring2、marker、stand 并清字段；本方法不调用 `RemoveHudIndicator`。[清理](../../../raw/current/scripts/components/pointofinterest.lua#L155-L177) |
| `DebugForceShowIndicator()` | 绕过数据集和等级启动门槛，直接建世界标记并启动更新；后续仍由 OnUpdate 消费设置和等级，不是正式注册数据入口。[调试入口](../../../raw/current/scripts/components/pointofinterest.lua#L287-L292) |

## 关键流程

### 更新、HUD 注册与重新显示

1. 构造排任务尝试启动。`RedirectThing` 对实体优先取 `scrapbook_proxy`，否则取 prefab；`GetLevelFor` 返回 0 未认识、1 已见未检查、2 已检查。数据集不存在的自定义对象即使等级为 0，也不会正常启动。[启动](../../../raw/current/scripts/components/pointofinterest.lua#L65-L89) [重定向](../../../raw/current/scripts/scrapbookpartitions.lua#L139-L145) [等级](../../../raw/current/scripts/scrapbookpartitions.lua#L304-L323)
2. 每帧先按 POI 显示设置、当地玩家／HUD 和 `HUD.shown` Show／Hide 已有 marker。接着仅在玩家／HUD 存在、非 `_removing` 且 shouldshowfn 允许时进入主分支；设置启用且等级 `< 2` 时建世界标记，再按距离增删方向提示。HUD 的 shown 只约束前一处 marker 显隐，不是整个主分支门槛。[OnUpdate](../../../raw/current/scripts/components/pointofinterest.lua#L238-L272)
3. 世界 stand／marker／ring 创建时才调用 `AddPOIIndicator` 并安装其删除回调；删除时从该列表撤销。HUD 的 Toggle 会批量隐藏／显示该列表。方向提示则由 `AddTargetIndicator` 创建独立 widget，使用 `poi_question.tex`、`images/avatars.xml`；widget 持续按目标位置、距离更新方向和大小。[创建与删除](../../../raw/current/scripts/components/pointofinterest.lua#L5-L45) [HUD 注册](../../../raw/current/scripts/screens/playerhud.lua#L1711-L1736) [HUD 切换](../../../raw/current/scripts/screens/playerhud.lua#L322-L345) [widget 更新](../../../raw/current/scripts/widgets/targetindicator.lua#L95-L152)
4. 设置关闭时，主分支撤方向提示，首段隐藏 marker；不会停更新或销毁 stand。再次开启设置且条件允许时，可恢复 marker 并按距离重建方向提示。`GetPOIDisplay` 读取本地设置／Profile 的 poidisplay，未明确置 false 时启用。[显示设置](../../../raw/current/scripts/playerprofile.lua#L625-L639) [更新分支](../../../raw/current/scripts/components/pointofinterest.lua#L238-L272)
5. 等级达到 2 时，只有执行到启用显示的主分支才 `TriggerRemove`；脉冲仍在 `_removing` 分支推进，最后 `RemoveEverything` 停更。它不订阅图鉴变化事件；停更后不会因本地数据变化自动重启。[等级分支与脉冲](../../../raw/current/scripts/components/pointofinterest.lua#L256-L278) [最终清理](../../../raw/current/scripts/components/pointofinterest.lua#L200-L236)

### 休眠、移除与重新创建

- `OnEntitySleep` 标注用于 master sim，只调用 `RemoveEverything`；`OnEntityWake` 再尝试启动。非 master 的辅助实体设为不可休眠。醒来时若启动条件仍满足，后续更新会重新创建世界标记。[休眠钩子](../../../raw/current/scripts/components/pointofinterest.lua#L103-L109) [辅助实体睡眠](../../../raw/current/scripts/components/pointofinterest.lua#L13-L19)
- 组件 `OnRemoveEntity` 先撤方向提示，再清表现；移除组件复用同一方法。HUD 内 RemoveTargetIndicator 依目标找到 widget 并 Kill；世界辅助实体删除回调撤的是 POI 列表。[完整删除](../../../raw/current/scripts/components/pointofinterest.lua#L280-L285) [方向 widget 删除](../../../raw/current/scripts/screens/playerhud.lua#L1749-L1763) [POI 删除回调](../../../raw/current/scripts/components/pointofinterest.lua#L7-L11)

## 典型接入

### 猪王保留小地图图标并增加本地图鉴提示

`prefabs/pigking.lua` 的 fn 分别设置 `MiniMapEntity` 图标，再在 `not TheNet:IsDedicated()` 分支添加 pointofinterest、设置高度 70；该分支位于 `SetPristine` 与非 master 返回之前，因此覆盖普通客户端及本地主机。数据集已有 pigking 的 POI 条目，正常启动依旧要求当地等级 `< 2`。[小地图与宿主组装](../../../raw/current/scripts/prefabs/pigking.lua#L629-L684) [图鉴条目](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1116)

这说明两个表现入口独立：修改组件高度影响其符号跟随偏移；不改变猪王小地图图标、图鉴内容或服务端交易。撤掉组件走自身清理；只调用 `RemoveEverything` 后若还需继续表现，应通过既有启动条件恢复更新，不能把它当仅隐藏接口。[世界标记](../../../raw/current/scripts/components/pointofinterest.lua#L120-L132) [组件移除](../../../raw/current/scripts/components/pointofinterest.lua#L280-L285)

## 常见陷阱与限制

- **shouldshowfn 变 false／HUD 暂缺不等于统一隐藏**：这会跳过主分支，不会立即删除既有方向提示；首段 marker 显隐也不检查 shouldshowfn。需要严格隐藏的宿主须明确处理已有表现与恢复时机。[条件顺序](../../../raw/current/scripts/components/pointofinterest.lua#L238-L278)
- **仅睡眠清理可能保留方向提示状态**：RemoveEverything 不清 `_showinghud`、不撤方向 widget；醒来后 `_showinghud` 仍真时不会重复 AddTargetIndicator。不能把睡眠钩子的效果描述为与组件删除相同。[睡眠](../../../raw/current/scripts/components/pointofinterest.lua#L103-L109) [两类清理](../../../raw/current/scripts/components/pointofinterest.lua#L113-L118) [世界清理](../../../raw/current/scripts/components/pointofinterest.lua#L163-L177)
- **HUD 后出现不会自动补登记已有世界实体**：AddPOIIndicator 与 OnRemoveEntity 回调仅在辅助实体创建时 HUD 已存在才安装；CreateWorldIndicator 见 stand 已有就返回。运行中替换 HUD 的恢复行为不由本组件保证。[注册时机](../../../raw/current/scripts/components/pointofinterest.lua#L40-L45) [重复创建门禁](../../../raw/current/scripts/components/pointofinterest.lua#L120-L123)

## 验证记录

- 2026-10-27，`steam-buildid:26148036`：审阅组件全文，以及猪王构造／注册、分区重定向和等级、数据集对应条目、Profile 开关、HUD 两类列表与 TargetIndicator 的相关消费路径。覆盖正常启动、条件跳过、距离、解锁脉冲、休眠、删除及重建边界；不扩展为全部兴趣点宿主清单。
- 游戏运行：未运行；未验证实际画面、HUD 重建、联机流入／流出或休眠残留的可见表现。源码中的非统一清理按实记录，不据此声称已复现游戏故障。
- 结构检查：页面校验通过；严格源码链接校验通过，0 错误、0 警告。无 Lua 示例，未执行语法探针。
