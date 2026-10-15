---
title: "targettracker"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "在装备等宿主上记录单个存活目标的持续追踪秒数，支持暂停、条件失效及配套装备间的目标同步。"
aliases: ["targettracker", "持续锁定计时", "追踪目标时间", "战斗装备叠层", "W.A.R.B.I.S.同步", "亮茄装备目标"]
tags: [component, callback, event, lifecycle, equipment, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/targettracker.lua, prefabs/armor_wagpunk.lua, prefabs/hats.lua, components/combat.lua, entityscript.lua]
reliability: verified
validation: source-reviewed
status: active
---

# targettracker

## 快速判断

- 职责与适用：保存一个目标和累积追踪时间，由宿主按时间阈值增加装备效果；原版消费者是亮茄装备的头盔与护甲。组件不寻敌、不移动、不向 Combat 设置攻击目标。[状态](../../../raw/current/scripts/components/targettracker.lua#L1-L15) [实际叠层](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L161-L208) [Combat 自有目标](../../../raw/current/scripts/components/combat.lua#L1127-L1129)
- 优先使用：TrackTarget 前核验目标并先清旧目标；SetOnTimeUpdateFn 接阈值效果，SetShouldKeepTrackingFn 接范围等宿主条件，卸装备时主动 StopTracking。[启动](../../../raw/current/scripts/components/targettracker.lua#L86-L100) [原版换目标](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L128-L156) [卸下](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L269-L274)
- 重要副作用：启动／停止在**组件宿主**推 `targettracker_starttrack`／`targettracker_stoptrack`；StopTracking 清掉目标、时间、暂停并停更新，`reset=true` 另调用宿主重置回调。[事件及清理](../../../raw/current/scripts/components/targettracker.lua#L91-L121)
- 修改边界：两种原版装备均仅服务端添加；目标显示／效果由各宿主额外同步，不是组件自带 replica。本组件没有存档与 OnRemoveFromEntity，新增宿主须自行接效果撤销与动态移除清理。[护甲添加](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L358-L382) [头盔额外网络字段](../../../raw/current/scripts/prefabs/hats.lua#L4615-L4643) [实体移除组件行为](../../../raw/current/scripts/entityscript.lua#L670-L683)

## 关键接口

| 接口／状态 | 参数、返回与条件 | 作用与证据 |
|---|---|---|
| `TrackTarget(target)` | 有效实体；调用方先验 Health 等条件 | 已有任意 target 则直接返回，不切换。设置 target、timetracking=0 后发启动事件，注册宿主 onremove 并开始更新；不返回成功值。[实现](../../../raw/current/scripts/components/targettracker.lua#L86-L100) |
| `HasTarget()`／`IsTracking(testtarget)` | 布尔 | 仅检查引用是否非 nil／相等，不保证目标此时仍有效；失效由更新检查。[实现](../../../raw/current/scripts/components/targettracker.lua#L41-L46) |
| `GetTimeTracking()`／`SetTimeTracking(time)` | 未追踪时通常 nil；有效计时为秒 | 设置值即调用 ontimeupdatefn(inst,newtime,0)，旧时间参数固定为 0；不会启动追踪或更新。[实现](../../../raw/current/scripts/components/targettracker.lua#L57-L67) |
| `SetOnTimeUpdateFn(fn)` | fn(inst,time,lasttime) | 活跃非暂停帧传新旧累积秒数；可比较阈值跨越，不能假定 SetTimeTracking 也传真实旧值。[设置](../../../raw/current/scripts/components/targettracker.lua#L31-L32) [逐帧](../../../raw/current/scripts/components/targettracker.lua#L163-L168) |
| `SetShouldKeepTrackingFn(fn)` | fn(inst,target) 返回真值继续 | 基础有效性检查通过后调用，暂停期间也要满足；false 即 StopTracking(true)。[设置](../../../raw/current/scripts/components/targettracker.lua#L35-L36) [检查顺序](../../../raw/current/scripts/components/targettracker.lua#L137-L155) |
| `Pause(time)`／`IsPaused()` | time 为秒，按用途传正数 | 设 pausetime 并调用 onpausefn(inst)；仅 pausetime 非 nil 视为暂停。无独立 Resume 接口，更新中倒计时归零后调用 onresumefn(inst)。[查询](../../../raw/current/scripts/components/targettracker.lua#L49-L50) [暂停](../../../raw/current/scripts/components/targettracker.lua#L127-L131) [恢复](../../../raw/current/scripts/components/targettracker.lua#L152-L159) |
| `SetOnPauseFn(fn)`／`SetOnResumeFn(fn)` | fn(inst) | 分别由 Pause 和暂停到期触发；恢复那一帧不再累计剩余 dt，下个非暂停更新才计时。[设置](../../../raw/current/scripts/components/targettracker.lua#L23-L28) [互斥分支](../../../raw/current/scripts/components/targettracker.lua#L152-L168) |
| `StopTracking(reset)`／`SetOnResetTarget(fn)` | reset 为真才调 fn(inst,nil) | 先清引用再发停止事件、移除宿主 onremove 监听、可选重置回调、停更新；不返回旧目标。普通 StopTracking 不自动撤销宿主自定义增益。[实现](../../../raw/current/scripts/components/targettracker.lua#L103-L121) |
| `CloneTargetFrom(item,pausetime)` | item 必须有 targettracker 且有目标／计时 | 对本宿主调用 TrackTarget(来源.target)，不复制累计时间；若传 pausetime，两件物品均 Pause。若接收方已有目标，启动被拒但仍可能同时暂停。[实现](../../../raw/current/scripts/components/targettracker.lua#L71-L80) |
| `IsCloningTarget()` | 布尔 | 仅判断 `(timetracking or 0)<=0` 且暂停，并非独立同步来源或克隆完成标记。[实现](../../../raw/current/scripts/components/targettracker.lua#L53-L54) |
| `OnUpdate(dt)`／`LongUpdate(dt)` | dt 为秒；后者直接复用前者 | 先检查目标，再处理暂停或累积；无效、INLIMBO、休眠、无 Health、死亡或自定义拒绝均重置停止。LongUpdate 不绕过失效判断。[更新](../../../raw/current/scripts/components/targettracker.lua#L137-L173) |

## 典型接入

### 连续对同一敌人作战时提升装备效果

- **宿主与入口：** `armor_wagpunk` 服务端工厂设置 TimeCheck、ResetBuff、ShouldKeepTarget、UnpauseFn；OnEquip 监听**穿戴者**的 `onattackother`，OnAttack 从当前身体装备取回组件。追踪者实体是装备，目标是攻击事件里的敌人。[工厂](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L364-L382) [监听对象](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L232-L239) [取目标](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L128-L156)
- **触发与效果：** 换敌人先 StopTracking(true)，用可取消的 2 秒任务等待，再 SetNewTarget 开始追踪；TimeCheck 按累计秒数跨阈值调整 walkspeedmult、FX 与声音，ShouldKeepTarget 控制距离。这个 2 秒等待属于宿主，不是 targettracker 固有延迟；追踪开始后也不要求每帧持续攻击。[等待](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L142-L156) [开始](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L84-L114) [效果与距离](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L161-L214)
- **撤销：** ResetBuff 取消待选目标任务、复原速度与 FX；卸下时另外移除穿戴者监听、取消任务、恢复速度并 StopTracking()。复制该模式须连同这些宿主清理，单独挂组件不产生任何增益。[重置](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L62-L80) [卸装清理](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L253-L287)

### 头盔与护甲共享锁定目标，延迟同步积累进度

| 分工 | 原版接入与限制 |
|---|---|
| 装备时取得另一件的目标 | 两侧 OnEquip 调 CloneTargetFrom 并传 WAGPUNK_SYNC_TIME；此时接收方的计时从 0 开始，两件同时暂停。[护甲](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L232-L235) [头盔](../../../raw/current/scripts/prefabs/hats.lua#L4450-L4457) |
| 到期后追平时间 | 各自恢复回调查另一件 GetTimeTracking，只有对方更大才 SetTimeTracking；这是宿主 UnpauseFn，非 CloneTargetFrom 自动复制时间。[护甲恢复](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L291-L301) [头盔恢复](../../../raw/current/scripts/prefabs/hats.lua#L4528-L4538) |
| 卸装时清同步与增益 | 原版对仍在初始克隆暂停态的另一件也 StopTracking；头盔移除穿戴者 Combat 的 externaldamagemultipliers，再停止自身追踪。[护甲](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L253-L274) [头盔](../../../raw/current/scripts/prefabs/hats.lua#L4476-L4498) |
| 客户端显示目标 | 头盔监听组件宿主的启动／停止事件，将目标写进额外 net_entity；不要在客户端直接读取服务端 target 字段作为同步方案。[事件转网络](../../../raw/current/scripts/prefabs/hats.lua#L4545-L4550) [监听与网络字段](../../../raw/current/scripts/prefabs/hats.lua#L4615-L4620) |

## 常见陷阱与限制

- **把 TrackTarget 当 SetTarget**：已有目标直接拒绝；HasTarget 也不做实时有效性检查。换目标沿用原版先停止、再核验和启动的顺序，效果清理由 reset 回调或宿主显式执行。[入口门禁](../../../raw/current/scripts/components/targettracker.lua#L86-L100) [原版切换](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L142-L156)
- **误读 targetremovedfn 的监听对象**：TrackTarget 的 ListenForEvent 未传第三参数，实际监听**宿主** onremove，不是被追踪目标；目标被移除靠后续 OnUpdate 的 IsValid 检查停止，不能承诺目标 onremove 同帧同步回调。[注册](../../../raw/current/scripts/components/targettracker.lua#L96-L97) [事件默认 source](../../../raw/current/scripts/entityscript.lua#L1214-L1216) [目标检查](../../../raw/current/scripts/components/targettracker.lua#L140-L148)
- **重置回调读取旧目标**：StopTracking 先 `self.target=nil`，随后才 `onresettarget(inst,self.target)`，所以第二参为 nil；如撤销效果依赖旧目标，在宿主另行保留引用，不按回调名猜测参数。[顺序](../../../raw/current/scripts/components/targettracker.lua#L103-L116)
- **暂停时目标不会失效**：有效性／ShouldKeepTracking 检查在暂停分支前；目标休眠、死亡、进 limbo 或离开宿主范围仍会停止。追踪秒数不是无条件墙钟计时；Pause(nil) 还会调用 onpausefn，却不会保持暂停。[暂停接口](../../../raw/current/scripts/components/targettracker.lua#L127-L131) [先验条件](../../../raw/current/scripts/components/targettracker.lua#L137-L168)
- **动态移除组件能自动撤销装备效果**：RemoveComponent 停更新，却没有替本组件调用 StopTracking 或宿主的速度／伤害清理；先按宿主协议撤销增益并停止追踪，再移除。本组件无 OnSave／OnLoad，不能承诺重载后保留目标和叠层。[组件生命周期](../../../raw/current/scripts/entityscript.lua#L670-L683) [需显式清理的原版](../../../raw/current/scripts/prefabs/hats.lua#L4485-L4498)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 8 处），未运行游戏。

- 源码审阅：2026-10-23，Steam buildid 25643519；完整审阅 targettracker、核对护甲与头盔的启动／条件／阈值效果／同步／卸装消费者，特别核实宿主 onremove 与 reset 第二参为 nil。Combat 仅查独立攻击目标与事件入口，不将本组件解读为 Brain 寻敌器。
- 结构检查：本页格式／本地链接及严格源码引用检查通过；本页复用原版装备组装参照，无独立完整 Lua 示例。
- 游戏运行：未运行；装备组合、暂停期间卸装／换敌与客户端目标显示未实测。

相关：[[reference/components/equippable]]、[[reference/components/combat]]、[[reference/components/inventoryitem]]。
