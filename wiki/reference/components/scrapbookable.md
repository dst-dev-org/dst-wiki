---
title: "scrapbookable"
created: "2026-10-27"
updated: "2026-10-28"
type: component
summary: "把库存 TEACH 动作转交宿主的图鉴教学回调；具体学习、客户端往返和物品消耗由宿主完成。"
aliases: ["图鉴纸页教学", "笔记阅读回调", "scrapbookable"]
tags: [component, callback, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/scrapbookable.lua", "componentactions.lua", "actions.lua", "prefabs/scrapbook_page.lua", "prefabs/scrapbook_notes.lua", "scrapbookpartitions.lua", "networkclientrpc.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# scrapbookable

## 快速判断

- 职责与适用：挂在可通过库存动作教学的物品上，保存回调并转交 `Teach(doer)`；组件本身不选择图鉴条目、不消费物品。[组件全文](../../../raw/current/scripts/components/scrapbookable.lua#L1-L17)
- 优先使用：宿主以 `SetOnTeachFn(fn)` 定义实际阅读／学习协议；原版纸页和笔记都在 master 分支添加组件。[纸页](../../../raw/current/scripts/prefabs/scrapbook_page.lua#L125-L158) [笔记](../../../raw/current/scripts/prefabs/scrapbook_notes.lua#L120-L137)
- 重要副作用：`Teach` 忽略回调返回值并恒返回 true，即使没设回调也如此；动作成功不证明学到新条目。[执行](../../../raw/current/scripts/components/scrapbookable.lua#L9-L15)
- 修改边界：教学内容、单人预约、RPC、成功后消耗和失败提示均属宿主／Scrapbook 分区协议；仅加组件不能复现纸页整套行为。[纸页协议](../../../raw/current/scripts/prefabs/scrapbook_page.lua#L6-L56)

## 关键接口

| 接口／入口 | 条件、作用与证据 |
|---|---|
| `SetOnTeachFn(fn)` | 将 fn 存为 `onteach`；回调签名 `fn(inst, doer)`。无参数校验、无自动连锁旧回调。[设置与调用](../../../raw/current/scripts/components/scrapbookable.lua#L5-L15) |
| `Teach(doer)` | 有回调就调用，随后返回 true；没有 CanTeach、失败原因或自动扣除接口。回调返回 false 也不会传给动作。[执行](../../../raw/current/scripts/components/scrapbookable.lua#L9-L15) |
| 库存候选 | `COMPONENT_ACTIONS.INVENTORY.scrapbookable` 直接插入 ACTIONS.TEACH，不判断已学会／预约中；执行阶段由宿主处理。[库存入口](../../../raw/current/scripts/componentactions.lua#L2570) [候选](../../../raw/current/scripts/componentactions.lua#L2879-L2881) |
| `ACTIONS.TEACH` | 有 invobject 时优先分发 scrapbookable，传入 `act.doer`；这里不把 `act.target` 作为学习者。动作显示变体由 `scrapbook_note`→NOTES、`scrapbook_data`→SCRAPBOOK 决定，标签不负责选择实际学习算法。[显示与分发](../../../raw/current/scripts/actions.lua#L3249-L3266) |

## 典型接入

### 普通纸页与特殊纸页：客户端决定学习结果，宿主消耗

`prefabs/scrapbook_page.lua` 以 commonfn 构造 `scrapbook_page`／`scrapbook_page_special`，服务端配置 OnTeach 和 OnScrapbookDataTaught。特殊页同步 `_id` 并保存对应 name，普通页可堆叠；两者共用组件回调。[构造与 ID](../../../raw/current/scripts/prefabs/scrapbook_page.lua#L72-L158) [工厂注册](../../../raw/current/scripts/prefabs/scrapbook_page.lua#L169-L178)

OnTeach 有预约时提示 INUSE 并返回；有 doer.userid 才继续。远端玩家通过 CLIENT_RPC 请求当地分区学习，并设置 10 秒取消预约任务；本地主机玩家直接调用 `TryToTeachScrapbookData(true, inst)`，再走同一结果回调。[预约与派发](../../../raw/current/scripts/prefabs/scrapbook_page.lua#L34-L56)

分区先检查 `_id > 0`，特殊页遍历该页 entries，把等级 `< 1` 的条目标为已见；否则普通页随机选择 3～4 个未知、属于有效分类的条目标为已见。它们不是直接全部提升到“已检查”的等级 2。[选择分支](../../../raw/current/scripts/scrapbookpartitions.lua#L388-L404) [随机与特殊教学](../../../raw/current/scripts/scrapbookpartitions.lua#L325-L372) [等级含义](../../../raw/current/scripts/scrapbookpartitions.lua#L304-L323)

客户端调用分区后将 learned_something 通过服务器 RPC 回传；服务端入口调用物品 `OnScrapbookDataTaught(player,response)`。纸页回调仅接受与 reserved_userid 匹配的玩家，清预约及超时任务；diduse 为真时从堆叠取 1 个删除，或直接删除非堆叠物品；为假时提示图鉴已满。超时只取消预约，不消费物品。[双向 RPC](../../../raw/current/scripts/networkclientrpc.lua#L1205-L1214) [客户端入口](../../../raw/current/scripts/networkclientrpc.lua#L1516-L1518) [回传](../../../raw/current/scripts/scrapbookpartitions.lua#L400-L404) [结果与取消](../../../raw/current/scripts/prefabs/scrapbook_page.lua#L6-L32)

### 六种 Wagstaff 笔记：检查本条目并打开图鉴，不消费

`scrapbook_notes.lua` 的 NOTES 生成 `wagstaff_mutations_note`、`wagstaff_materials_note`、`wagstaff_energy_note`、`wagstaff_containment_note`、`wagstaff_thermal_note`、`wagstaff_electricity_note`；用各自 prefab 选择学习内容，统一名称／库存图像不改变该选择。[数据](../../../raw/current/scripts/prefabs/scrapbook_notes.lua#L10-L41) [构造与注册](../../../raw/current/scripts/prefabs/scrapbook_notes.lua#L92-L156)

笔记沿用单人预约、10 秒超时和远端 RPC 分支；带 scrapbook_note 时，分区调用 `SetInspectedByCharacter(entry, ThePlayer.prefab)`，有 HUD 就打开图鉴并选择该条目。笔记的结果回调只清预约／取消任务，不根据 diduse 删除笔记，因此可再次阅读。[宿主教学](../../../raw/current/scripts/prefabs/scrapbook_notes.lua#L52-L90) [分区笔记分支](../../../raw/current/scripts/scrapbookpartitions.lua#L374-L404)

## 常见陷阱与限制

- **用 onteach 返回 false 表示动作失败**：组件丢弃该返回，仍返回 true；原版通过提示和 diduse 处理“没学到”，不能依赖 Teach 的布尔值判断消费。[组件](../../../raw/current/scripts/components/scrapbookable.lua#L9-L15) [纸页反馈](../../../raw/current/scripts/prefabs/scrapbook_page.lua#L11-L32)
- **只加 scrapbook_data 标签就能教随机条目**：该标签仅选择动作文字；真正教学路径需要组件回调、userid 预约、分区调用与结果处理。直接调用 Teach 也不会自动搭建这些协议。[动作](../../../raw/current/scripts/actions.lua#L3249-L3266) [宿主](../../../raw/current/scripts/prefabs/scrapbook_page.lua#L34-L56)
- **统一删除可教物品**：纸页按实际学到内容消耗 1 个，笔记不删除；是否消耗由具体宿主的结果回调决定。[纸页](../../../raw/current/scripts/prefabs/scrapbook_page.lua#L11-L32) [笔记](../../../raw/current/scripts/prefabs/scrapbook_notes.lua#L57-L65)

## 验证记录

- 2026-10-27，`steam-buildid:26148036`：审阅组件全文、库存 TEACH 候选／分发、两种纸页及六种笔记的完整宿主、分区教学／等级相关函数和双方 RPC 消费。覆盖回调返回、内容选择、预约超时、消费差异及主机／客户端边界。
- 游戏运行：未运行；未实测图鉴 UI、RPC 延迟／超时、多人争用与物品删除。无 Lua 示例，未执行语法探针。
- 结构检查：页面校验通过；严格源码链接校验通过，0 错误、0 警告。
