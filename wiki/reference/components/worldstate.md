---
title: "worldstate"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界状态组件：把时钟/季节/天气等世界事件汇聚为 TheWorld.state 数据表，提供按变量与 start/stop 切换的通知观察者。"
aliases: ["worldstate", "世界状态", "TheWorld.state", "WatchWorldState"]
tags: [component, data, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/worldstate.lua", "prefabs/world.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# worldstate

## 快速判断

- 职责与适用：装在每个世界（world.lua 公共段）上，监听约 15 种世界事件把值写入 `self.data`，且 `TheWorld.state` **就是这个表**（世界构造时直接赋引用）。默认值设计为“无对应组件时的合理回退”（无季节组件=autumn，洞穴世界 surface 时钟冻结在 night）。[安装与别名](../../../raw/current/scripts/prefabs/world.lua#L532-L533) [断言](../../../raw/current/scripts/components/worldstate.lua#L12) [默认值说明](../../../raw/current/scripts/components/worldstate.lua#L181-L186)
- 优先使用：读状态直接取 `TheWorld.state.<变量>`；监听变化用 `inst:WatchWorldState(var, fn)`（它转调 `AddWatcher`）；底层注册用 `AddWatcher(var,inst,fn,target)`。 [观察者](../../../raw/current/scripts/components/worldstate.lua#L281-L295) [实体包装](../../../raw/current/scripts/entityscript.lua#L1290-L1293)
- 重要副作用：变更通知**仅在值真正变化且非 nil 时**发出；toggle 变量（isday 等）额外触发 `("start"或"stop")..togglename` 键的观察者，可监听“进入/离开”而非值。[SetVariable](../../../raw/current/scripts/components/worldstate.lua#L24-L48)
- 修改边界：无 OnRemoveFromEntity；OnSave 复制整张 data 表，OnLoad 只覆盖已有键（每次 print 调试噪音）。[存读](../../../raw/current/scripts/components/worldstate.lua#L331-L347)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `AddWatcher(var,inst,fn,target)` | 变量下次变化时调用 `fn(target, newvalue)`；按 var→inst→{fn,target} 三层组织。WatchWorldState 的底层。[实现](../../../raw/current/scripts/components/worldstate.lua#L281-L295) |
| `RemoveWatcher(var,inst,fn?,target?)` | fn 为 nil 清空该实体的全部观察；否则按 fn+target 精确移除（while 连删同键）。[实现](../../../raw/current/scripts/components/worldstate.lua#L297-L325) |
| `GetWorldAge()` | 返回 `1 + cycles + time` 的世界年龄。[实现](../../../raw/current/scripts/components/worldstate.lua#L277-L279) |
| `OnSave()`／`OnLoad(data)` | 全表保存；加载只恢复已知键并逐项 print。[实现](../../../raw/current/scripts/components/worldstate.lua#L331-L347) |
| `Dump()` | 调试：按字母序输出全部变量名与值。[实现](../../../raw/current/scripts/components/worldstate.lua#L353-L360) |

### 变量清单与事件来源

- **时钟**（[clocktick/cycleschanged](../../../raw/current/scripts/components/worldstate.lua#L54-L61)）：`time`、`timeinphase`、`cycles`。
- **相位**（[phasechanged](../../../raw/current/scripts/components/worldstate.lua#L72-L80)）：`phase`、`isday/isdusk/isnight`、`isfullmoon/isnewmoon`（=夜+对应月相）。洞穴世界改听 [cave 版](../../../raw/current/scripts/components/worldstate.lua#L63-L70)：surface 的 phase/isnight 保持初值（isnight=true），真实昼夜走 `cavephase`、`iscaveday/iscavedusk/iscavenight`、`iscavefullmoon/iscavenewmoon`。
- **月相**（[moonphasechanged2](../../../raw/current/scripts/components/worldstate.lua#L89-L95)）：`moonphase`、`iswaxingmoon`。
- **暴风**（[ms_stormchanged](../../../raw/current/scripts/components/worldstate.lua#L97-L101)）：仅月亮风暴时更新 `isalterawake`。
- **梦魇**（[nightmareclocktick/phasechanged](../../../raw/current/scripts/components/worldstate.lua#L103-L114)）：`nightmarephase/time/timeinphase` + 四个 toggle；无梦魇钟时 phase="none"（[初值](../../../raw/current/scripts/components/worldstate.lua#L220)）。
- **季节**（[seasontick/lengthschanged](../../../raw/current/scripts/components/worldstate.lua#L116-L132)）：`season` + 四季 toggle、`elapseddaysinseason`、`remainingdaysinseason`、`seasonprogress`、四季节长度。
- **天气**（[temperaturetick/weathertick 等](../../../raw/current/scripts/components/worldstate.lua#L134-L176)）：`temperature`、`moisture`、`moistureceil`、`pop`、`precipitationrate`、`snowlevel`、`lunarhaillevel/lunarhailrate`、`wetness`、`precipitation` + `israining/issnowing/islunarhailing/isacidraining`、`issnowcovered`（顺带 `TheSim:HandleAllSnowSymbols`）、`iswet`。

## 典型接入

### 监听世界状态变化并读取

- 前提与选择：任意端（组件在世界公共段安装）；`WatchWorldState` 是实体方法，参数 var 为上表任一变量名或 toggle 键。**注册时不回调**——需要立即对齐时自行先读一次状态（原版组件均如此）。[注册实现](../../../raw/current/scripts/entityscript.lua#L1290-L1293)
- 生效与撤销：Watch/StopWatching 成对；实体移除走 `StopAllWatchingWorldStates` 兜底。[撤销](../../../raw/current/scripts/entityscript.lua#L1295-L1310)

```lua
-- 任意端：监听冬季切换并读取常用状态
inst:WatchWorldState("iswinter", function(inst, iswinter)
    if iswinter then
        -- 进入冬季时执行
    end
end)

local state = TheWorld.state
print(state.cycles, state.phase, state.temperature)
```

- 调整与验证：toggle 变量回调第二参为新布尔值；验证可用 `TheWorld.components.worldstate:Dump()` 对照（[Dump](../../../raw/current/scripts/components/worldstate.lua#L353-L360)）。

## 常见陷阱与限制

- **直接写 TheWorld.state 不通知：** `TheWorld.state` 是裸数据表，直接赋值绕过 SetVariable，观察者不触发且值会被 OnSave 持久化；改状态必须走对应世界组件（weather/seasons 等）的事件链。[别名引用](../../../raw/current/scripts/prefabs/world.lua#L533) [变更路径](../../../raw/current/scripts/components/worldstate.lua#L24-L48)
- **WatchWorldState 不立即回调：** 想在注册时执行初始逻辑必须自己读状态（如 [[reference/components/shadowhandspawner]] 的做法）；“注册后没触发”多半是值确实没变。[注册语义](../../../raw/current/scripts/entityscript.lua#L1290-L1293)
- **洞穴世界 surface 相位恒为夜：** cave 世界 `isday` 恒 false、`isnight` 恒 true，昼夜逻辑要用 `iscave*` 系列；两套变量不要混用。[分派](../../../raw/current/scripts/components/worldstate.lua#L214-L215) [洞穴分支](../../../raw/current/scripts/components/worldstate.lua#L63-L70)
- **OnLoad 打印每个变量：** 每次读档刷一屏 `setting ...`，是源码遗留调试输出；据此误判“加载异常”是浪费时间。[print](../../../raw/current/scripts/components/worldstate.lua#L344)
- **start/stop 观察者键不常见：** `AddWatcher("startday", ...)` 这类键由 togglename 拼接产生，可监听“进入白天”单沿；但要注意 SetVariable 里 togglename 只在值变化时触发一次。[拼接规则](../../../raw/current/scripts/components/worldstate.lua#L37-L46)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 worldstate.lua，核验全部变量/事件映射、观察者三层结构与移除语义、洞穴分派、存读与 WatchWorldState 实体包装；核对 world.lua 安装与 state 别名。未运行游戏；各世界组件（weather/seasons/clocks）如何推这些事件属各自页面范围。

相关：[[reference/components/worldsettings]]、[[reference/components/shadowhandspawner]]。
