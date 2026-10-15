---
title: "circuitnode"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "薇诺娜工程节点的启用、按标签发现和双向集合；供电、功率、重新连接由宿主协作。"
aliases: ["服务端断开聚光灯电路", "薇诺娜电路节点", "双向节点连接", "circuitnode"]
tags: [component, callback, lifecycle, energy, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/circuitnode.lua", "prefabs/winona_battery_low.lua", "prefabs/winona_spotlight.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# circuitnode

## 快速判断

- 职责与适用：管理 nodes 集合／数量、启用和连接回调。原薇诺娜电源通过 ForEachNode 调目标 AddBatteryPower，电量与实际照明不是组件内置功能。[组件](../../../raw/current/scripts/components/circuitnode.lua#L1-L29) [供电消费者](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L88-L100)
- 优先使用：ConnectTo(tag) 启用并发现现有目标；ConnectTo(nil) 只启用，Disconnect 移除连接并禁用。改变 range 不会自动重建现有拓扑。[发现](../../../raw/current/scripts/components/circuitnode.lua#L32-L56) [断开／范围](../../../raw/current/scripts/components/circuitnode.lua#L59-L71)
- 重要副作用：AddNode／RemoveNode 先更新本端并回调，再递归处理另一端；回调期间不能假定双方集合已一致，也不要在遍历回调中随意改连接。[加入／移出](../../../raw/current/scripts/components/circuitnode.lua#L82-L107)
- 修改边界：服务端；无自动轮询／存档重连。原电源与聚光灯通过初始化、建造／加载协作重连；电围栏的实体线段协议另看 [[reference/components/electricconnector]]。[电源初始化](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L643-L652) [聚光灯加载](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1015-L1027)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `IsEnabled()`／`IsConnected()`／`NumConnectedNodes()` | nodes 非 nil 表示启用，numnodes>0 表示有连接；刚 ConnectTo(nil) 可以已启用但零连接。[查询](../../../raw/current/scripts/components/circuitnode.lua#L20-L34) |
| `ConnectTo(tag?)` | 创建本端集合；有 tag 时按本端 range 找可见、带 tag 且已启用的其他 circuitnode。不会顺带启用未启用对端，也不会周期追踪新实体。[发现](../../../raw/current/scripts/components/circuitnode.lua#L32-L56) |
| `range`／`SetRange(number)` | 默认 5，直接改搜索半径，不删旧边；原薇诺娜宿主用 TUNING.WINONA_BATTERY_RANGE=6.6 并另设 footprint 规则。[默认](../../../raw/current/scripts/components/circuitnode.lua#L1-L10) [安装](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L935-L941) [配置](../../../raw/current/scripts/tuning.lua#L3739) |
| `SetFootprint(number)`／`rangeincludesfootprint` | footprint 默认 0、开关默认 false；启用时以“本端 range 减目标 footprint”作额外距离测试，不是加双方半径，也不检查对端 range。[计算](../../../raw/current/scripts/components/circuitnode.lua#L44-L52) |
| `connectsacrossplatforms` | 默认 true；false 时要求与本端同 GetCurrentPlatform，岸上 nil／nil 相等。原电源和聚光灯显式设 false。[过滤](../../../raw/current/scripts/components/circuitnode.lua#L39-L52) [宿主](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1136-L1142) |
| `SetOnConnectFn(fn)`／`SetOnDisconnectFn(fn)` | fn(inst,node)；先更新本端数量、后通知对端，宿主用它刷新电路／显示／供电任务。[设置](../../../raw/current/scripts/components/circuitnode.lua#L74-L100) [原消费者](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L161-L172) |
| `AddNode(node)`／`RemoveNode(node)` | 低层集合操作，不重查范围／标签／平台／可见性；AddNode 只要求本端 nodes 已建，对端未启用会产生单边登记，调用者需确保双方可用。[方法](../../../raw/current/scripts/components/circuitnode.lua#L82-L100) |
| `Disconnect()`／`ForEachNode(fn)` | 前者逐条双向移除后 nodes=nil；后者 fn(inst,node) 遍历当前直接邻居，不递归整网。[断开](../../../raw/current/scripts/components/circuitnode.lua#L59-L63) [遍历](../../../raw/current/scripts/components/circuitnode.lua#L104-L107) |

### 重新连接与移除

组件没有保存方法。原聚光灯 OnLoad 取消初始化任务后只 ConnectTo(nil)，等待电源 post-load 扫描；新实例再按 engineeringbattery 扫描。清理钩子先清本端 ondisconnectfn 再 Disconnect，所以本端回调不会重放，对端仍可收到移除通知。[加载／初始化](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1015-L1027) [电源 postpass](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L643-L652) [清理](../../../raw/current/scripts/components/circuitnode.lua#L13-L18)

## 典型接入

### 服务端断开原薇诺娜聚光灯的当前节点关系

目标模组 `modmain.lua`，复用原聚光灯资源和断开回调；不删 fields、不模拟电池消息。原回调会按剩余节点刷新电路，无连接时撤线并 SetPowered(false)。[安装](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1136-L1142) [原回调](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L935-L943)

```lua
AddPrefabPostInit("winona_spotlight", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_disconnect_circuit", function(light)
        if light.components.circuitnode ~= nil then
            light.components.circuitnode:Disconnect()
        end
    end)
end)
```

服务端已有逻辑向实际灯实体发 `light:PushEvent("my_mod_disconnect_circuit")`。本例关闭本端连接资格并调用双向移除，不提供永久禁用标志、RPC 或新 UI。需要重新启用时应沿宿主可工作／未烧毁等条件再调用 ConnectTo；不能在任何状态强制重连。[原加载门禁](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1015-L1027)

断开不保存，后续宿主初始化或重建仍可连接；若在原零秒初始化任务之前发事件，也可能被其重新启用。停用并重载后不再安装监听，并由原流程恢复拓扑。本例不保证原供电余量、灯光动画在同一帧全部归零，也不覆盖 powerload 机制。

## 常见陷阱与限制

- **节点已连接就自动有电**：集合只提供邻居，实际电源 tick 调 AddBatteryPower，宿主功率／燃料另管。[供电](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L88-L100)
- **设置较小 range 自动断远边／定期找新边**：SetRange 只赋值，ConnectTo 只在被调用时扫描；要重建需保留宿主初始化／回调与门禁。[实现](../../../raw/current/scripts/components/circuitnode.lua#L32-L71)
- **销毁时本端断开回调必执行**：组件清理先清此回调，再移除；不要只依赖它撤销宿主自身周期任务。[清理](../../../raw/current/scripts/components/circuitnode.lua#L13-L18)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 1 处（tuning WINONA_BATTERY_RANGE +14）；无结论修订，仅锚点迁移——WINONA_BATTERY_RANGE=6.6 值未变（tuning-hunk-map 5 改值键不含），circuitnode.lua 及两个薇诺娜宿主 prefab 全部未变。依据：tuning-hunk-map 键级审阅＋变更清单命中 tuning.lua。
- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整 circuitnode、低级电源供电／连接／建造／加载、聚光灯回调／加载／初始化与参数。未完整审校所有工程设备、功率分摊、灯光余量、移动平台／跨分片或模组篡改集合。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整断开事件例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；零／多连接、满载供电、断开后重连、初始化顺序、烧毁／锤拆／读档和平台场景待实测。
