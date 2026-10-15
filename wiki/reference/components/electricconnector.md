---
title: "electricconnector"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "电围栏配对、共享电场实体、连接标签和 GUID 重建；分清寻找条件、直接连接与状态机。"
aliases: ["电围栏找连接范围限制为四", "电围栏连接", "断开电场", "electricconnector"]
tags: [component, callback, lifecycle, state-machine, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/electricconnector.lua", "prefabs/fence_electric.lua", "prefabs/fence_electric_field.lua", "stategraphs/SGfence_electric.lua", "componentactions.lua", "actions.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# electricconnector

## 快速判断

- 职责与适用：原 fence_electric 两端记录同一个 field 实体，创建线段、登记双方并支持存读重建；薇诺娜供电节点是 [[reference/components/circuitnode]]，不用本协议。[构造与连接](../../../raw/current/scripts/components/electricconnector.lua#L1-L12) [登记](../../../raw/current/scripts/components/electricconnector.lua#L63-L97)
- 优先使用：原宿主 StartLinking／EndLinking 由 SG 完成找对端，断全部连接用 Disconnect；不绕过 SG 只设置标签。[方法](../../../raw/current/scripts/components/electricconnector.lua#L15-L49) [SG 消费](../../../raw/current/scripts/stategraphs/SGfence_electric.lua#L57-L93)
- 重要副作用：FindAndLinkConnector 按条件搜索；ConnectTo 本身只检查本端数量，再生成 field／登记对端／发 linked_to，不重检距离、平台、重复、自连或对端上限。[寻找](../../../raw/current/scripts/components/electricconnector.lua#L33-L60) [直接连接](../../../raw/current/scripts/components/electricconnector.lua#L63-L82)
- 修改边界：服务端，IsLinking 假定宿主有 sg；当前资格拒绝任一端位于平台。原电场还提供物理与触电行为，不能把生成的 field 视为纯装饰 FX。[限制](../../../raw/current/scripts/components/electricconnector.lua#L25-L36) [电场行为](../../../raw/current/scripts/prefabs/fence_electric_field.lua#L124-L135)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `StartLinking()`／`EndLinking()` | 只发 start_linking／end_linking 并返回 true；不是已找到对端／已连上，也不等于断开现有边。SG 从预备动画进入 linking_loop 再查找。[方法](../../../raw/current/scripts/components/electricconnector.lua#L15-L26) [SG](../../../raw/current/scripts/stategraphs/SGfence_electric.lua#L3-L12) |
| `IsLinking()`／`HasConnection()` | 前者查 sg 的 linking 状态标签，后者检查 fields 非空。普通 idle 也可能已有电场。[查询](../../../raw/current/scripts/components/electricconnector.lua#L25-L30) |
| `CanLinkTo(guy,on_load?)` | 对端有组件，且 on_load 或对端正在 linking；双方不在平台且尚未登记。自身 linking、双方距离／数量、自连并不全在此检查。[方法](../../../raw/current/scripts/components/electricconnector.lua#L33-L36) |
| `FindAndLinkConnector()`／`link_range` | 按 electric_connector 找附近对象，排 fully_electrically_linked，再 CanLinkTo；找到后 ConnectTo，成功返回对端，否则 nil。原范围 10。[实现](../../../raw/current/scripts/components/electricconnector.lua#L52-L60) [配置](../../../raw/current/scripts/tuning.lua#L8814-L8815) |
| `ConnectTo(connector)`／`field_prefab` | 本端满则无返回；否则生成场、置于中点、SetBeam，再双方登记同一 field。默认 field_prefab=fence_electric_field，要求该 Prefab 支持 SetBeam。[实现](../../../raw/current/scripts/components/electricconnector.lua#L63-L82) |
| `RegisterField(other,field)` | 先 onlinkedfn(inst,other,field)，后写 fields／field.fences 和连接／满额标签；回调此时未见新登记。原 max_links=2。[实现](../../../raw/current/scripts/components/electricconnector.lua#L85-L97) [配置](../../../raw/current/scripts/tuning.lua#L8814-L8815) |
| `UnregisterField(other)` | 清本端登记与满额标签；仅当**最后一边**消失才调 onunlinkedfn(inst,other) 并清连接标签，不是每条边删除通知。[实现](../../../raw/current/scripts/components/electricconnector.lua#L101-L110) |
| `Disconnect()` | 发本端 disconnect_links，逐场 Remove 并清双方登记，再通知对端动画；返回 true。不要仅清 fields 或 Remove field 而漏另一端。[方法](../../../raw/current/scripts/components/electricconnector.lua#L39-L49) |
| `OnSave()`／`LoadPostPass(newents,data)` | 保存对端 GUID 列表／依赖；加载 CanLinkTo(other,true) 后重建场，跳过 linking 要求但仍拒绝平台与重复连接，不重检原搜索距离。[存读](../../../raw/current/scripts/components/electricconnector.lua#L118-L137) |

### 宿主与场实体的清理

原围栏 pristine 前加 electric_connector，服务端设组件回调与 SG；onlinked／onunlinked 控制灯和电流显示。组件两个移除钩子均指 Disconnect，锤拆宿主也显式 Disconnect；不要叠加另一套场实体管理。field 自身 persists=false，由连接关系恢复，不独立保存线段。[宿主](../../../raw/current/scripts/prefabs/fence_electric.lua#L135-L139) [安装](../../../raw/current/scripts/prefabs/fence_electric.lua#L179-L186) [移除钩子](../../../raw/current/scripts/components/electricconnector.lua#L147-L148) [场保存边界](../../../raw/current/scripts/prefabs/fence_electric_field.lua#L361-L369)

## 典型接入

### 原电围栏以后寻找对端的距离限制为四个世界单位

目标模组 `modmain.lua`；保留原资源、SG、配对数和电场协议，只缩小原 finder 使用的半径，不改所有 TUNING 消费者。[原安装](../../../raw/current/scripts/prefabs/fence_electric.lua#L179-L186) [搜索半径消费](../../../raw/current/scripts/components/electricconnector.lua#L58-L60)

```lua
AddPrefabPostInit("fence_electric", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local connector = inst.components.electricconnector
    if connector ~= nil then
        connector.link_range = math.min(connector.link_range, 4)
    end
end)
```

正常原值 10 被缩到 4；若先前模组已设置更小则保留。只影响此端随后 FindAndLinkConnector，**另一端仍可能主动找到它**，所以不同范围的混合对象不构成全局四单位硬限制；直接 ConnectTo 和保存重连也不由本字段限距。[搜索与连接](../../../raw/current/scripts/components/electricconnector.lua#L58-L82) [重建](../../../raw/current/scripts/components/electricconnector.lua#L132-L137)

既有长连接不自动断开；显示用 placer 圆仍按原 TUNING，不因此同步成四单位，本例未修改预览资源。参数由工厂／PostInit 重设，停用并重载恢复默认搜索配置，不保证删除已有保存关系；重新设计距离规则需同时核验双方发现、加载与预览链。[预览比例](../../../raw/current/scripts/prefabs/fence_electric.lua#L282-L283)

## 常见陷阱与限制

- **StartLinking 返回 true 就能立即 IsLinking／已有连接**：它只发事件，linking 标签在 SG 的 loop 状态，不在预备状态；结束等待也不等于移除旧场。[方法](../../../raw/current/scripts/components/electricconnector.lua#L15-L26) [loop](../../../raw/current/scripts/stategraphs/SGfence_electric.lua#L57-L66)
- **max_links 对任何调用都双端硬限额**：ConnectTo 仅查本端，通常 finder 排对端满额；直调／重建不可据此保证任意修改后的双方容量。[实现](../../../raw/current/scripts/components/electricconnector.lua#L52-L82)
- **只 Remove field 能完整断边**：双方 fields／标签和显示回调另有责任；正常入口用 Disconnect。本页未提供损坏引用或生成场失败时的完整容错。[断开](../../../raw/current/scripts/components/electricconnector.lua#L39-L49)

## 验证记录

- 版本沿用：2026-10-25，`steam-buildid:26148036`；原结论复用、锚点迁移 4 处（tuning.lua 电围栏常量、fence_electric.lua 标签／安装／预览比例）；依据：变更清单命中 fence_electric.lua、tuning.lua，fence_electric 本版仅改 OnHit 触电判定（排除远程武器），不触及连接协议；ELECTRIC_FENCE_MAX_LINKS＝2、ELECTRIC_FENCE_MAX_DIST＝10 未变。未运行游戏。
- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整 electricconnector、原围栏安装／显示／锤拆、SG 连接循环、动作／候选，以及电场 SetBeam／非持久化／触电入口和范围配置。未全面审校电场分段物理、所有碰撞免疫／AI、坏引用、跨平台支持或加载容量冲突。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整半径例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；两端点击顺序、混合半径、已有长边、保存重建、末边断开、锤拆与预览差异待实测。
