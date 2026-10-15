---
title: giftreceiver：私人礼物提示与开礼物流程
created: 2026-10-27
updated: "2026-10-28"
type: component
summary: 追踪玩家私人礼物计数和可用机器，衔接 classified 提示、状态图与礼物弹窗。
aliases: [giftreceiver, 私人礼物计数, 开礼物, 礼物机器]
tags: [component, event, network-sync, ui, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/giftreceiver.lua, components/builder.lua, prefabs/player_common.lua, prefabs/player_classified.lua, prefabs/scienceprototyper.lua, stategraphs/SGwilson.lua, widgets/giftitemtoast.lua, networkclientrpc.lua, popupmanager.lua, screens/giftitempopup.lua]
reliability: verified
validation: source-reviewed
status: active
---

# giftreceiver

## 快速判断

- 职责与适用：服务端玩家组件，保存该玩家的 `giftcount` 与 `giftmachine`；原版仅在游戏模式未隐藏收礼时添加。[player_common.lua:2613](../../../raw/current/scripts/prefabs/player_common.lua#L2613)、[player_common.lua:2836](../../../raw/current/scripts/prefabs/player_common.lua#L2836
- 优先使用：理解原版私人礼物提示、科技机器礼物动画与开礼物状态的协作；计数来自 `TheInventory:GetClientGiftCount(userid)`。[giftreceiver.lua:57](../../../raw/current/scripts/components/giftreceiver.lua#L57
- 重要副作用：计数及机器引用变化写入 classified 的两个布尔量，并向机器发出增删接收者事件；网络侧不传真实数量和机器实体。[giftreceiver.lua:15](../../../raw/current/scripts/components/giftreceiver.lua#L15)、[giftreceiver.lua:74](../../../raw/current/scripts/components/giftreceiver.lua#L74
- 修改边界：本组件只读私人礼物数量并分发流程事件；没有赠送、授权库存物品、扣减礼物或选择奖励的实现。调用 `OpenNextGift()` 不代表礼物已授予。[giftreceiver.lua:57](../../../raw/current/scripts/components/giftreceiver.lua#L57)、[giftreceiver.lua:92](../../../raw/current/scripts/components/giftreceiver.lua#L92

## 关键接口

| 接口／字段 | 参数或返回 | 条件、副作用与证据 |
|---|---|---|
| `giftcount`／`HasGift()` | 初始 `0`／返回 `giftcount > 0` | 赋值回调向 `player_classified.hasgift` 写入是否有礼物；不要将客户端提示当实际数量。[giftreceiver.lua:15](../../../raw/current/scripts/components/giftreceiver.lua#L15)、[giftreceiver.lua:27](../../../raw/current/scripts/components/giftreceiver.lua#L27)、[giftreceiver.lua:53](../../../raw/current/scripts/components/giftreceiver.lua#L53 |
| `RefreshGiftCount()` | 无参数，无显式返回 | 按 `self.inst.userid` 读取引擎计数；变化时更新字段，有机器时仅在无礼物与有礼物之间切换才发增删事件。[giftreceiver.lua:57](../../../raw/current/scripts/components/giftreceiver.lua#L57 |
| `SetGiftMachine(inst)` | 机器实体或 `nil` | 仅引用变化时处理；有礼物时向旧／新机器分别发 `ms_removegiftreceiver`／`ms_addgiftreceiver`，事件数据为玩家。设为 `nil` 还会停止开礼物。[giftreceiver.lua:74](../../../raw/current/scripts/components/giftreceiver.lua#L74 |
| `OpenNextGift()` | 无显式返回 | 只检查计数大于零且机器非空，向玩家发 `ms_opengift`；本方法不自行检查危险、骑乘或 SG 忙碌。[giftreceiver.lua:92](../../../raw/current/scripts/components/giftreceiver.lua#L92 |
| `OnStartOpenGift()` | 无参数 | 向当前机器发 `ms_giftopened`，原版消费者用于机器动画与音效。[giftreceiver.lua:98](../../../raw/current/scripts/components/giftreceiver.lua#L98)、[scienceprototyper.lua:148](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L148 |
| `OnStopOpenGift(usewardrobe)` | 真值表示尝试换装 | 向玩家发 `ms_doneopengift`；真值时数据含当前机器 `wardrobe`，否则传 `nil`。实际是否换装由 SG 决定。[giftreceiver.lua:104](../../../raw/current/scripts/components/giftreceiver.lua#L104)、[SGwilson.lua:6730](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6730 |

构造时监听玩家的 `ms_closepopup`，只处理 `POPUPS.GIFTITEM`；同时用两次零延迟任务等待 userid，再监听玩家的 `ms_updategiftitems` 并刷新计数。该更新事件的引擎来源只有源码注释指向 `GiftingManager.cpp`，本页未审阅 C++ 实现。[giftreceiver.lua:1](../../../raw/current/scripts/components/giftreceiver.lua#L1)、[giftreceiver.lua:33](../../../raw/current/scripts/components/giftreceiver.lua#L33

## 关键流程

1. **选择机器**：`builder:EvaluateTechTrees()` 复用当前科技站选择结果；仅当站点带 `giftmachine`、玩家能看见目标且 inventory 的 `isopen` 为真时传入站点，否则 `SetGiftMachine(nil)`。单独添加标签不构成完整接入。[builder.lua:235](../../../raw/current/scripts/components/builder.lua#L235)、[builder.lua:311](../../../raw/current/scripts/components/builder.lua#L311
2. **同步提示**：classified 定义 `hasgift`／`hasgiftmachine` 两个 `net_bool`；`giftsdirty` 在拥有 HUD 的父玩家上发 `giftreceiverupdate`，数据为 `numitems=1/0`、`active=机器是否存在`。toast 依此显示并启停点击。[player_classified.lua:1575](../../../raw/current/scripts/prefabs/player_classified.lua#L1575)、[player_classified.lua:954](../../../raw/current/scripts/prefabs/player_classified.lua#L954)、[player_classified.lua:1252](../../../raw/current/scripts/prefabs/player_classified.lua#L1252)、[giftitemtoast.lua:24](../../../raw/current/scripts/widgets/giftitemtoast.lua#L24
3. **请求开礼物**：toast 在非忙碌且满足点击间隔时，客户端发送 `RPC.OpenGift`，主机直接调组件；服务端 RPC 也调用 `OpenNextGift()`。[giftitemtoast.lua:145](../../../raw/current/scripts/widgets/giftitemtoast.lua#L145)、[networkclientrpc.lua:1066](../../../raw/current/scripts/networkclientrpc.lua#L1066
4. **进入状态与弹窗**：SG 仅在非 `busy` 时接受 `ms_opengift`；`opengift` 进入时拒绝附近危险或骑乘。成功路径暂停控制、隐藏背包和动作，显示 `POPUPS.GIFTITEM`，再通知组件开始开礼物。[SGwilson.lua:2370](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2370)、[SGwilson.lua:6667](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6667
5. **结束或换装**：弹窗选择立即使用时 `Close(owner, true)`，普通关闭动画结束时 `Close(owner)`；客户端经 RPC 回到服务端，转为玩家的 `ms_closepopup`。组件转换为 `ms_doneopengift`，SG 检查 wardrobe、可见性及换装条件，失败则回 idle；状态退出关闭弹窗并按分支恢复界面。[giftitempopup.lua:93](../../../raw/current/scripts/screens/giftitempopup.lua#L93)、[giftitempopup.lua:138](../../../raw/current/scripts/screens/giftitempopup.lua#L138)、[popupmanager.lua:11](../../../raw/current/scripts/popupmanager.lua#L11)、[networkclientrpc.lua:1073](../../../raw/current/scripts/networkclientrpc.lua#L1073)、[SGwilson.lua:6730](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6730)、[SGwilson.lua:6742](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6742

`POPUPS.GIFTITEM` 只接受可选布尔关闭参数；其显示处理调用 HUD 的物品管理屏幕入口，打开失败也会关闭弹窗。本页没有追查 HUD 内部领取物品或库存服务链。[popupmanager.lua:91](../../../raw/current/scripts/popupmanager.lua#L91

## 典型接入

### 原版科学机器与炼金引擎接收礼物提示

`scienceprototyper.lua` 的 `createmachine` 工厂实际注册 `researchlab` 和 `researchlab2`。共同端添加 `giftmachine`／`prototyper` 标签，主模拟端添加 prototyper 与 wardrobe；后者关闭普通换装动作并允许共享，服务于开礼物后的换装流程。[scienceprototyper.lua:192](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L192)、[scienceprototyper.lua:205](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L205)、[scienceprototyper.lua:213](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L213)、[scienceprototyper.lua:219](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L219)、[scienceprototyper.lua:299](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L299

机器监听组件的三种事件：接收者增删刷新运行状态，`ms_giftopened` 播放礼物动画。判断是否有待开礼物时遍历 `prototyper.doers`，还要求玩家 `HasGift()` 且机器引用正是自己。该模式依赖 builder 的科技站选择与使用者维护，适合解释原版科技站协作；本页不提供自定义礼物授予教程。[scienceprototyper.lua:11](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L11)、[scienceprototyper.lua:114](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L114)、[scienceprototyper.lua:243](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L243

## 常见陷阱与限制

- **直接伪造计数**：修改 `giftcount` 只会影响本地字段、classified 提示和后续门槛；下一次刷新重新读取引擎计数。它不提供账户物品授权能力。[giftreceiver.lua:15](../../../raw/current/scripts/components/giftreceiver.lua#L15)、[giftreceiver.lua:57](../../../raw/current/scripts/components/giftreceiver.lua#L57
- **把机器字段当完整资格校验**：`SetGiftMachine` 不自行查距离、标签或可见性；这些由原版 builder 路径提供，危险和骑乘由 SG 补查。绕过调用链会丢失这些条件。[giftreceiver.lua:74](../../../raw/current/scripts/components/giftreceiver.lua#L74)、[builder.lua:311](../../../raw/current/scripts/components/builder.lua#L311)、[SGwilson.lua:6676](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6676
- **移除清理的裸 `inst` 疑点**：`OnRemoveFromEntity()` 两行使用 `inst:RemoveEventCallback`，但该方法无 `inst` 参数，文件内也无供此处捕获的外层局部 `inst`；构造函数的参数不在此方法作用域内。因此它会查找环境中的 `inst`，不能按 `self.inst` 理解；若值为 `nil`，执行首行即会出错，两个监听的清理不能视为可靠。未做运行复现，不据此推断正常玩家移除时的具体表现，也未修改原版脚本。[giftreceiver.lua:27](../../../raw/current/scripts/components/giftreceiver.lua#L27)、[giftreceiver.lua:48](../../../raw/current/scripts/components/giftreceiver.lua#L48

## 验证记录

- 2026-10-27，`steam-buildid:26148036`：审阅 giftreceiver 全文；定向核验 player_common 添加边界、builder 机器选择、classified 同步、toast／RPC 请求、SG 开关状态、popup 关闭分发及 scienceprototyper 注册与消费者。
- 未运行游戏；未核验 C++ 私人礼物服务、HUD 内部领取流程、真实账户库存变化、危险／骑乘／换装场景和组件移除异常的运行结果。
- `validate_page.py` 与 `validate_source_links.py --strict` 检查通过；它们不替代上述源码审阅或游戏运行。未提供完整 Lua 示例，无示例语法探针范围。

相关机制：[[reference/components/builder]]、[[reference/components/prototyper]]、[[reference/components/wardrobe]]。
