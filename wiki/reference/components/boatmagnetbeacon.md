---
title: "boatmagnetbeacon"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "磁铁信标：登记配对磁铁与开关状态、随拾取或放置切换跟随目标，本身不产生牵引力。"
aliases: ["磁铁信标", "船用信标", "自动航行信标", "boatmagnetbeacon"]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/boatmagnetbeacon.lua", "components/boatmagnet.lua", "components/boatphysics.lua", "prefabs/boat_magnet_beacon.lua", "prefabs/boat_magnet.lua", "stategraphs/SGboatmagnetbeacon.lua", "actions.lua", "componentactions.lua", "tuning.lua", "entityscript.lua", "recipes.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatmagnetbeacon

## 快速判断

- 职责与适用：`boat_magnet_beacon` 物品／结构的服务端组件，登记“被哪块磁铁配对”（`magnet`＋`magnet_guid`）、开关与拾取状态。宿主是 prefab `boat_magnet_beacon`（工厂在 `ismastersim` 后 `AddComponent`，注册名相同，可制作，配方见 `recipes.lua`），实体带 `boatmagnetbeacon` tag。[宿主](../../../raw/current/scripts/prefabs/boat_magnet_beacon.lua#L67-L85)、[注册](../../../raw/current/scripts/prefabs/boat_magnet_beacon.lua#L110)、[配方](../../../raw/current/scripts/recipes.lua#L633)
- 优先使用：配对入口是磁铁的 `PairWithBeacon`，它回调本组件 `PairWithMagnet`；玩家开关走动作 `BOAT_MAGNET_BEACON_TURN_ON`／`TURN_OFF`（已放置或拿在手里都可）；查询用 `PairedMagnet()`、`IsTurnedOff()`、`GetBoat()`。
- 重要副作用：`PairWithMagnet` 会 `TurnOnBeacon()`→推 `onturnon`、切自身 SG 到 `activate`、换物品图；`TurnOffBeacon` 推 `onturnoff`、切 `deactivate`、加 `turnedoff` tag；配对双方都加 `paired` tag。开关不解除配对。
- 修改边界：只在服务端存在；本组件不产生力、不改速度，力由 boatmagnet 计算、boatphysics 施加（见 [[reference/components/boatmagnet]] 与 [[reference/components/boatphysics]]）。未配对时开关没有牵引效果，不能单独当“吸引器”。

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `PairedMagnet()`／`magnet`／`magnet_guid` | 磁铁实体／GUID | 配对判据与存档键；`magnet_guid` 存的是磁铁实体 GUID，信标自己的 GUID 不参与配对。[L145-L147](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L145-L147)、[L67-L74](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L67-L74) |
| `PairWithMagnet(magnet)` | 无返回 | 已有 `self.magnet` 或参数为空就早退（一块信标只记一块磁铁）；否则记 `magnet`／GUID、挂磁铁 `onremove`／`death` 监听、`TurnOnBeacon()`、加 `paired`。[L149-L162](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L149-L162) |
| `UnpairWithMagnet()` | 无返回 | 撤磁铁监听、清 `magnet`／`magnet_guid`、`TurnOffBeacon()`、去 `paired`。[L164-L177](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L164-L177) |
| `IsTurnedOff()`／`TurnOnBeacon()`／`TurnOffBeacon()` | 布尔／无返回 | `turnedoff` 状态；开启换 `boat_magnet_beacon_on` 贴图、`sg:GoToState("activate")`、推 `onturnon`、去 tag；关闭相反。`TurnOn` 有 `if inst.sg` 保护，`TurnOff` 没有。[L179-L209](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L179-L209) |
| `GetBoat()` | 船实体或 nil | 先取父实体（被拿在手里时是持有者）再 `GetCurrentPlatform()`，只认带 `boat` tag 的平台；每次调用刷新 `self.boat`。[L116-L121](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L116-L121) |
| `SetBoat(boat)` | 平台实体或 nil | 只挂／撤平台 `onremove`／`death` 监听，不建物理关系；同值早退。[L123-L137](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L123-L137) |
| `IsPickedUp()`／`SetIsPickedUp(pickedup)` | 布尔／无返回 | 监听 `onpickup`／`ondropped`；被拿时把 `self.boat` 置 nil，放下时重新 `GetBoat()`（直接赋值，不经 `SetBoat`）。[L1-L9](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L1-L9)、[L211-L218](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L211-L218) |
| `magnet_must_tags`／`magnet_cant_tags`／`magnet_distance` | 表／表／世界单位 | 读档重连的搜索条件：默认 `{"boatmagnet"}`、`{"paired"}`、`MAX_DISTANCE`=48；只在 0 秒任务里用一次。[L43-L56](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L43-L56) |
| `OnSave()`／`OnLoad(data)` | `{turnedoff, ispickedup, magnet_guid}` | 存开关、拾取与配对 GUID；读档恢复字段与 `turnedoff` tag，但不恢复 `magnet`，磁铁要靠 0 秒任务重连。[L67-L99](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L67-L99) |
| `SetupBoatTask`／`OnRemoveFromEntity`／`OnRemoveEntity` | 初始化／清理 | 0 秒任务先 `SetBoat(GetCurrentPlatform())` 再按 GUID 重连；移除组件取消任务并撤 `onpickup`／`ondropped`，移除实体只 `SetBoat(nil)`，配对由磁铁侧监听清理。[L11-L41](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L11-L41)、[L101-L114](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L101-L114) |

### 登记、范围与重连

- 正常玩法中信标不主动找磁铁：`PairWithMagnet` 只等磁铁调用；磁铁侧用 24 的配对半径找带 `boatmagnetbeacon` tag 且未配对的信标。信标自己的 48 半径只用于读档重连，两者不是同一个范围。[配对半径](../../../raw/current/scripts/components/boatmagnet.lua#L116-L126)
- 读档重连：0 秒任务以信标世界坐标为心，`TheSim:FindEntities` 找 48 内带 `boatmagnet` tag、不带 `paired` tag 的实体，逐个比较 `magnet.components.boatmagnet.magnet_guid == self.magnet_guid`；命中就 `PairWithBeacon` 并更新 GUID，否则清空本组件的 `magnet`／`magnet_guid`，配对丢失。[L19-L38](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L19-L38)
- 磁铁的 `magnet_guid` 字段由磁铁自己的 `OnLoad` 写入（存的是自身 GUID）；新建未读档的磁铁该字段为 nil，所以重连实际上要求磁铁也从存档恢复。[磁铁存档](../../../raw/current/scripts/components/boatmagnet.lua#L56-L70)
- 信标只跑一次这个搜索；当时不在 48 内（被带走、在容器里或磁铁尚未就绪）就会清 GUID，之后只能重新用磁铁搜索配对。[L30-L37](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L30-L37)

### 开关状态的消费者

- 自身 SG：`onturnon` → `activate`；`TurnOffBeacon` 直接 `GoToState("deactivate")`，不推事件给自身 SG 的监听表。[SG](../../../raw/current/scripts/stategraphs/SGboatmagnetbeacon.lua#L1-L6)
- 配对的磁铁：监听 `onturnon`／`onturnoff`，分别转调 prefab 注入的 `beaconturnedonfn`／`beaconturnedofffn`，把磁铁 SG 切到 `pull`／`idle`／`pull_pst`。[磁铁监听](../../../raw/current/scripts/components/boatmagnet.lua#L27-L38)、[监听注册](../../../raw/current/scripts/components/boatmagnet.lua#L136-L139)
- 力：boatphysics 的磁力汇总会跳过 `IsTurnedOff()` 为真的配对，因此关闭信标后力为 0、`CalcMaxVelocity` 也返回 0，但配对关系与 `paired` tag 都还在。[施力条件](../../../raw/current/scripts/components/boatphysics.lua#L551-L557)
- 动作可见性：已放置信标只在 `right`、`paired`、未烧毁时出现开／关动作；背包内动作也要求 `paired`。未配对的信标没有玩家可用的开关。[放置](../../../raw/current/scripts/componentactions.lua#L195-L203)、[背包](../../../raw/current/scripts/componentactions.lua#L2572-L2580)

## 典型接入

### 原版链路：磁铁配对→信标自动开启→船跟随

- 前提与选择：玩家制作 `boat_magnet_beacon`（`cutstone` 2、`transistor` 1、`trinket_6` 1），把它丢在海上、放到某条船上或拿在手里；船上的 `boat_magnet` 执行激活动作后搜索 24 内最近未配对信标并 `PairWithBeacon`，本组件随即 `PairWithMagnet`→`TurnOnBeacon`。[配方](../../../raw/current/scripts/recipes.lua#L633)、[配对](../../../raw/current/scripts/components/boatmagnet.lua#L128-L148)
- 端与单位：全部在服务端；距离是世界单位，配对 24、断开 48；信标本身没有速度与力。
- 生效与撤销：配对并开启后磁铁所在船开始被牵引；拿起信标会改变跟随目标，关闭信标只是暂停牵引，解除配对要由磁铁侧动作或超出 48。锤掉信标（3 次）或磁铁会终止连接。
- 原版依据：`GetFollowTarget` 的三级回退决定跟随对象：信标所在船→被拿时的父实体→信标本体（丢在水里时追漂浮的信标）。[目标](../../../raw/current/scripts/components/boatmagnet.lua#L175-L188)

### 三种放置方式对应的目标

- 丢在海上：`GetBoat()` 为 nil、`IsPickedUp()` 为 false，磁铁直接朝信标位置牵引，可作为可移动航点；把信标扔到更远处，船会继续追。
- 放在另一条船上：`GetBoat()` 解析到该船，磁铁按两船心距离与船速跟随；这也是“船跟船”的原版用法。
- 拿在玩家手里：若持有者站在船上，`GetBoat()` 仍解析到那条船；人不在船上时才返回持有者实体，磁铁按玩家 `locomotor:GetRunSpeed()`（≤`TUNING.BOAT.MAX_VELOCITY`=1.2）跟随，船会追着人走。[L185-L188](../../../raw/current/scripts/components/boatmagnet.lua#L185-L188)

## 常见陷阱与限制

- 开关不等于配对：`TurnOffBeacon` 只置 `turnedoff`、换图、推事件，`magnet`／`magnet_guid`／`paired` 都保留；重新开启会恢复牵引，不需要重新搜索。[L198-L209](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L198-L209)
- 未配对时开关无效果：`componentactions` 用 `paired` tag 过滤动作，绕过界面直接调 `TurnOnBeacon()` 只会改状态与贴图，磁铁不会因此被吸引。
- `PairWithMagnet` 拒绝第二块磁铁，但磁铁侧 `PairWithBeacon` 不检查已配对：绕过搜索直接改配会出现“磁铁自认已配对、信标仍记旧磁铁”的单边关系，之后解除还会拆掉旧配对（详见 [[reference/components/boatmagnet]] 的陷阱）。
- `SetIsPickedUp` 直接改 `self.boat`，不走 `SetBoat`：船的 `onremove`／`death` 监听不会随搬运转移，旧船的回调仍会清空 `self.boat`；功能上靠 `GetBoat()` 每次重算兜底，但不要依赖 `self.boat` 判断当前船。[L211-L218](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L211-L218)、[SetBoat](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L123-L137)
- `TurnOffBeacon` 没有 SG 保护：自定义宿主若没有 `SGboatmagnetbeacon`，关闭时会 `inst.sg:GoToState` 报错；`TurnOnBeacon` 则有 `if self.inst.sg`。[L190-L205](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L190-L205)
- 重连窗口只有一次：0 秒任务里搜索 48 内同 GUID 的磁铁，失败就清空 `magnet_guid`，不会重试、也不会跨区域保存“曾配对”；读档后信标与磁铁必须都在附近且都已恢复。[L19-L38](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L19-L38)
- 信标移除不自己解除配对：`OnRemoveFromEntity` 只取消任务与拾取监听，`OnRemoveEntity` 只 `SetBoat(nil)`；配对清理由磁铁侧的 `onremove`／`death` 监听完成，磁铁已不存在时没有额外清理。[L101-L114](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L101-L114)
- 读档贴图按 `self.boat` 判断，而该字段要到 0 秒任务才赋值：读档瞬间总用未开启贴图；重连成功时 `TurnOnBeacon` 会补成开启图，未重连则保持未开启图（此时 `turnedoff` 可能仍是 false）。[L91-L98](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L91-L98)
- 开关靠事件而不是直接回调：`onturnon`／`onturnoff` 都推在信标实体上，任何监听者都会收到；别再额外监听后重复切 SG，否则一次开关会多次 `GoToState`。[L193-L206](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L193-L206)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 1 处（componentactions.lua INVENTORY 背包开关动作）；结论修订：无，仅锚点迁移。依据：枢纽 diff 审阅＋变更清单命中 componentactions.lua、recipes.lua（命中区域逐行核对均未漂移）。
- 源码审阅：2026-10-21，steam-buildid:25643519。完整阅读 `components/boatmagnetbeacon.lua`（225 行），覆盖构造与 0 秒任务、拾取／放置、GUID 重连、开关与事件、动作可见性、存读与清理；核对宿主 `prefabs/boat_magnet_beacon.lua` 的 AddComponent（L85）与注册名 `boat_magnet_beacon`、配方 `recipes.lua` L634，以及消费者 `boatmagnet`、`SGboatmagnetbeacon`、`boatphysics`、`actions`、`componentactions`。未覆盖贴图／音效资源与 scrapbook 表现。
- 游戏运行：未运行；本页结论均为源码静态核验，未测实际牵引手感与多人网络表现。
- 结构检查：`validate_page.py` 通过（0 issue；87 行、13324 字节，低于 180 行／14 KiB 上限）；`validate_source_links.py --strict` 通过（38 处引用、33 个目标、0 error／0 warning）；本页无完整 Lua 代码块。

相关：[[reference/components/boatmagnet]]、[[reference/components/boatphysics]]、[[reference/components/hull]]。
