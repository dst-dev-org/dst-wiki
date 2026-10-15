---
title: "mapdeliverable"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "管理地图选点会话与按时间推进的投递，接入无人机的移动、权限、落地和保存恢复回调。"
aliases: ["地图选点投递", "运输无人机配送", "侦察无人机移动", "mapdeliverable"]
tags: [component, callback, actions, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/mapdeliverable.lua", "prefabs/bufferedmapaction.lua", "prefabs/wx78_drone_delivery.lua", "prefabs/wx78_drone_scout.lua", "actions.lua", "componentactions.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# mapdeliverable

## 快速判断

- 职责与适用：调度地图选点与投递计时，记录起点／终点并调用进度回调；本组件不内置实体移动、寻路或货物转移。原版运输与侦察无人机自行实现移动效果。[计时主体](../../../raw/current/scripts/components/mapdeliverable.lua#L101-L147) [运输移动](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L301-L306)
- 优先使用：需要玩家地图选点先 `StartMapAction(doer)`；已有目标点用 `SendToPoint(pt, doer)`。接入时同时核验 start／progress／stop 回调，不只加一个组件。[地图会话](../../../raw/current/scripts/components/mapdeliverable.lua#L68-L89) [宿主接线](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L613-L619)
- 重要副作用：投递启动先调用 start 回调，成功后取消地图会话，再记录位置和时长；自然完成与 `Stop()` 共用 stop 回调，调用时进度字段已清空。[启动](../../../raw/current/scripts/components/mapdeliverable.lua#L113-L129) [完成与停止](../../../raw/current/scripts/components/mapdeliverable.lua#L132-L159)
- 修改边界：原版服务端挂载；地图选点依赖 `bufferedmapaction` 的定向网络实体，权限／飞行边界由动作和宿主补充，直接调用组件不会自动执行全部门禁。[端边界](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L593-L619) [缓冲实体](../../../raw/current/scripts/prefabs/bufferedmapaction.lua#L62-L75) [动作验证](../../../raw/current/scripts/actions.lua#L6882-L6891)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `StartMapAction(doer)` | true；或 false, reason? | 已有缓冲动作直接失败；可由 `onstartmapactionfn(inst, doer)` 否决。成功生成缓冲实体并将选点动作定向给 doer，需有效玩家。[实现](../../../raw/current/scripts/components/mapdeliverable.lua#L68-L82) |
| `CancelMapAction()` | 无返回 | 删除缓冲实体并清组件引用，不停止已启动投递；缓冲实体 onremove 通知 `oncancelmapactionfn(inst, doer)`。[实现](../../../raw/current/scripts/components/mapdeliverable.lua#L85-L89) [监听](../../../raw/current/scripts/components/mapdeliverable.lua#L12-L16) |
| `SendToPoint(pt, doer)` | true；或 false, reason? | 正在投递时拒绝；有缓冲动作时须匹配 MAPDELIVER_MAP 及同一 doer。start 回调可修改 pt，之后复制目标；无缓冲动作不要求曾调用 StartMapAction。[实现](../../../raw/current/scripts/components/mapdeliverable.lua#L101-L129) |
| `SetDeliveryTime(t)`／`SetDeliveryTimeFn(fn)` | 秒／`fn(inst, dest, doer)` | 默认 5 秒；有函数优先取函数结果，nil／false 回退默认。setter 不校验负数／类型，不改变已启动的 `len`。[默认与 setter](../../../raw/current/scripts/components/mapdeliverable.lua#L1-L9) [设置](../../../raw/current/scripts/components/mapdeliverable.lua#L40-L45) [取值](../../../raw/current/scripts/components/mapdeliverable.lua#L122-L127) |
| 三种投递回调 setter | Start／Progress／Stop | start 为 `(inst, pt, doer) → success, reason?`；progress 为 `(inst, elapsed, total, origin, dest)`；stop 只传 inst，无 doer／到达标志。分别见上方启动与下方停止顺序。 |
| `IsDelivering()`／`GetProgress()` | boolean／比例或 nil | `t ~= nil` 即投递中，t=0 也成立；未投递返回 nil 进度，len=0 返回 1。[查询](../../../raw/current/scripts/components/mapdeliverable.lua#L93-L98) |
| `Stop()` | 无返回 | 仅在投递中清状态、停更新并回调；不会将物体传送到目标，不另取消空闲选点会话。[实现](../../../raw/current/scripts/components/mapdeliverable.lua#L151-L159) |

进度更新先将 `t` 增加到不超过 `len`，再执行 progress 回调；该回调可 `Stop()`，后续完成判断会检查 t 是否仍存在。正常完成先清 `origin/dest/t/len`、停更新，才调用 stop；要识别实际到达，可在 progress 内按 t/len 处理并维护宿主标志，不能等 stop 再取 dest。[顺序](../../../raw/current/scripts/components/mapdeliverable.lua#L19-L23) [更新](../../../raw/current/scripts/components/mapdeliverable.lua#L132-L147)

## 典型接入

### 调整运输无人机投递时间，保留原有移动与落地

目标模组 `modmain.lua` 完整入口，复用两种原版运输无人机与资源。原版使用随距离计算的 `deliverytimefn`，因此只改 `SetDeliveryTime(…)` 不会覆盖它；包装原函数使新投递时长乘 1.5。[原函数](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L202-L210) [挂载与工厂注册](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L613-L619) [两种名称](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L671-L672)

```lua
local function ExtendDeliveryTime(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local delivery = inst.components.mapdeliverable
    local oldtimefn = delivery ~= nil and delivery.deliverytimefn or nil
    if oldtimefn ~= nil then
        delivery:SetDeliveryTimeFn(function(item, dest, doer)
            return oldtimefn(item, dest, doer) * 1.5
        end)
    end
end

AddPrefabPostInit("wx78_drone_delivery", ExtendDeliveryTime)
AddPrefabPostInit("wx78_drone_delivery_small", ExtendDeliveryTime)
```

只作用于此后启动的投递；当前 `len` 不重算，加载中的投递使用已保存的时长。本组件计时改变不等于统一按比例改变速度：运输宿主有固定起飞阶段，再按进度与距离插值，仍保留原版飞行许可检查和中止／落地逻辑。[起飞阶段](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L250-L267) [移动与阻挡中止](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L301-L309) [落地](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L344-L355)

去掉包装后，新建／重载实例恢复原版时间函数，已经保存的 len 不因此恢复。示例不改 start／progress／stop、地图技能／非空容器门禁、外观和资源，也不命中 `wx78_drone_scout`。

### 原版地图选点与侦察的两种入口

- **运输无人机：** 右键发现 `STARTMAPDELIVER`，客户端 `canmapdeliver` 检查物品非空和 batteryuser；服务端 start 回调检查技能、占用、容器与非空，再锁定使用者。地图选点由 `MAPDELIVER_MAP` 验证飞行路线并调用 `SendToPoint`。[动作发现](../../../raw/current/scripts/componentactions.lua#L561-L575) [客户端条件](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L491-L502) [服务端条件](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L515-L526) [最终动作](../../../raw/current/scripts/actions.lua#L6882-L6891)
- **侦察无人机：** 同样挂本组件及投递回调，但有 `mapscout` 标签，排除上述右键发现；`MAPSCOUT_MAP` 先 Stop 再 SendToPoint，可重新选定目的地。其 progress 自行更新位置、马达与扫描，不能只复制组件期待自动侦察。[宿主挂载](../../../raw/current/scripts/prefabs/wx78_drone_scout.lua#L290-L320) [重定向](../../../raw/current/scripts/actions.lua#L6822-L6837) [位置与马达](../../../raw/current/scripts/prefabs/wx78_drone_scout.lua#L150-L168)

## 常见陷阱与限制

- **取消选点等于取消投递：** CancelMapAction 与 Stop 操作不同状态；`SendToPoint` 自己也会取消选点，因此取消回调不能一概视为配送失败。原版取消仅通知玩家中断连续动作，由 stopcontinuousaction 再释放锁定。[启动取消](../../../raw/current/scripts/components/mapdeliverable.lua#L113-L125) [宿主取消协作](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L498-L506) [释放锁时清会话](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L181-L189)
- **外部删除缓冲实体即清完组件状态：** bufferedmapaction 自身移除只清宿主 `inst.bufferedmapaction`，组件监听只通知取消回调；自定义宿主应完成相应 `CancelMapAction` 清理，不把宿主字段与组件 `self.bufferedmapaction` 当成同一个字段。[缓冲移除](../../../raw/current/scripts/prefabs/bufferedmapaction.lua#L43-L47) [组件监听与清理](../../../raw/current/scripts/components/mapdeliverable.lua#L12-L16) [清组件引用](../../../raw/current/scripts/components/mapdeliverable.lua#L85-L89)
- **start 回调假设 doer 永远存在：** OnLoad 恢复未运行投递时会再次调用 `onstartdeliveryfn(inst, pt, nil)`，可被否决；不调用时间函数，沿用保存进度。回调需接受 nil doer，宿主另存发送者身份。[加载](../../../raw/current/scripts/components/mapdeliverable.lua#L183-L203) [运输发送者恢复](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L359-L376)
- **以为精确计时会完整保存：** 仅投递中保存起终点和进度，坐标约 0.001、t 到整数秒、len 到 0.1 秒；未保存地图选点或 doer，加载将 t 上限夹到 len。完整组件没有 LongUpdate，不能据此承诺离线时间自动推进。[保存与恢复](../../../raw/current/scripts/components/mapdeliverable.lua#L163-L203)
- **移除组件就会执行落地：** `OnRemoveFromEntity` 只 CancelMapAction，不调用 Stop／投递停止回调；EntityScript 虽停止组件更新，也不替宿主恢复外观和交互。整实体移除钩子另清缓冲并通知取消，二者都不是正常落地流程。[组件移除](../../../raw/current/scripts/components/mapdeliverable.lua#L26-L37) [框架移组件](../../../raw/current/scripts/entityscript.lua#L670-L683) [整实体清监听后调组件钩子](../../../raw/current/scripts/entityscript.lua#L1741-L1750)
- **直接发点绕过全部限制：** 本组件不校验世界可达性、玩家技能、容器货物等；这些由动作与宿主检查。自定义调用必须另定权限和目标边界，不能把返回 true 当成已到达或已交货。[组件准入](../../../raw/current/scripts/components/mapdeliverable.lua#L101-L129) [原版动作路线检查](../../../raw/current/scripts/actions.lua#L6882-L6891)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 变更文件（actions.lua、componentactions.lua、entityscript.lua、wx78_drone_delivery.lua、wx78_drone_scout.lua，更新报告 R-01/R-04/R-08）逐条核验，MAPDELIVER_MAP／MAPSCOUT_MAP 动作与 mapdeliverable 采集无语义变化；两无人机变更仅为 MiniMapEntity 虚拟房间可见性标记，EntityScript 变更（GetWet／距离函数）不触及本页引用段，锚点重定位 9 处、无结论修订；未覆盖：运行验证。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；完整组件、缓冲地图实体、地图动作发现／执行、运输两种注册工厂及回调、侦察工厂／移动回调、EntityScript 移除顺序。覆盖启动／否决、选点取消、计时停止、保存恢复及示例包装范围。
- 游戏运行：未运行；跨客户端地图操作、飞行障碍、加载恢复、延长投递时长的动画与其他模组叠加未实测。
- 结构检查：页面格式／链接与源码锚点严格检查通过；完整 `modmain.lua` 示例通过 Lua 5.1 语法检查，未执行。
