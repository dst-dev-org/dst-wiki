---
title: "boatmagnet"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "船载磁铁结构：配对信标并算出牵引方向、力与限速，由 boatphysics 施加到船体。"
aliases: ["船用磁铁", "自动航行磁铁", "磁铁与信标配对", "boatmagnet"]
tags: [component, lifecycle, callback, state-machine]
dst_build: "steam-buildid:26230496"
sources: ["components/boatmagnet.lua", "components/boatmagnetbeacon.lua", "components/boatphysics.lua", "prefabs/boat_magnet.lua", "prefabs/boat_magnet_beacon.lua", "stategraphs/SGboatmagnet.lua", "actions.lua", "componentactions.lua", "tuning.lua", "simutil.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatmagnet

## 快速判断

- 职责与适用：`boat_magnet` 结构上的服务端组件，负责“找信标→配对→维持或断开→算牵引方向、力与速度上限”，本身不改船速。宿主是 prefab `boat_magnet`（工厂在 `ismastersim` 后 `AddComponent`，注册名同为 `boat_magnet`，另有可部署套件 `boat_magnet_kit`）。[宿主](../../../raw/current/scripts/prefabs/boat_magnet.lua#L94-L119)、[注册](../../../raw/current/scripts/prefabs/boat_magnet.lua#L145-L146)
- 优先使用：玩家流程已由动作与 SG 封装：`BOAT_MAGNET_ACTIVATE` 进 `search_pre`，搜索动画结束后在 24 内找最近未配对信标并 `PairWithBeacon`；`BOAT_MAGNET_DEACTIVATE` 调 `UnpairWithBeacon`。模组在服务端读 `PairedBeacon()` 或调这两个方法即可。[动作](../../../raw/current/scripts/actions.lua#L5001-L5013)、[搜索落点](../../../raw/current/scripts/stategraphs/SGboatmagnet.lua#L120-L142)
- 重要副作用：配对即 `StartUpdatingComponent(self)`，每帧推 `boatmagnet_pull_start`（不是边沿事件）并旋转磁铁模型；磁铁与信标都加 `paired` tag。目标消失或距离超过 48 会自动解除。[配对](../../../raw/current/scripts/components/boatmagnet.lua#L128-L148)、[每帧](../../../raw/current/scripts/components/boatmagnet.lua#L294-L334)
- 修改边界：只在服务端存在，客户端实体没有该组件；本组件只产出力参数，真正改速度的是 boatphysics，推进链见 [[reference/components/boatphysics]]；信标侧只登记配对与开关，见 [[reference/components/boatmagnetbeacon]]。

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `SetBoat(boat)` | 平台实体或 nil | 挂接 boatphysics 的唯一入口。同值早退；旧船 `RemoveMagnet` 并撤 `onremove`／`death` 监听，新船 `AddMagnet` 并挂监听；要求平台带 `boat` tag 且已有 `boatphysics`。[L84-L100](../../../raw/current/scripts/components/boatmagnet.lua#L84-L100) |
| `PairedBeacon()`／`IsActivated()` | 信标实体／布尔 | 配对状态唯一判据；`IsActivated` 等价于 `beacon ~= nil`。[L102-L108](../../../raw/current/scripts/components/boatmagnet.lua#L102-L108) |
| `FindNearestBeacon()` | 实体或 nil | 以磁铁为心、半径 `PAIR_RADIUS`=24、`ignoreheight=true`，找最近且 `PairedMagnet()==nil` 的 `boatmagnetbeacon`；`pair_tags` 可替换 tag 集合。[L116-L126](../../../raw/current/scripts/components/boatmagnet.lua#L116-L126) |
| `PairWithBeacon(beacon)` | 无返回 | 不检查本磁铁是否已配对。写 `self.beacon`、调信标 `PairWithMagnet`、挂 4 个监听、启动更新、回调并加 `paired`。[L128-L148](../../../raw/current/scripts/components/boatmagnet.lua#L128-L148) |
| `UnpairWithBeacon()` | 无返回 | 撤 4 个监听；信标仍有效时调其 `UnpairWithMagnet`（清 GUID、关信标）；停更新、去 `paired`。[L150-L173](../../../raw/current/scripts/components/boatmagnet.lua#L150-L173) |
| `CalcMagnetDirection()` | `Vector3` 单位向量＋距离 | 船心到跟随目标的世界 xz 方向；目标可为信标所在船、携带者或信标本体。[L240-L256](../../../raw/current/scripts/components/boatmagnet.lua#L240-L256) |
| `CalcMagnetForce()` | 速度增量／秒 | 未配对、信标关闭或无 boatphysics 时 0；距离大于 `1+hull 半径` 时为 `MAGNET_FORCE`=0.6，否则 0。不直接施加。[L258-L292](../../../raw/current/scripts/components/boatmagnet.lua#L258-L292) |
| `CalcMaxVelocity()` | 速度上限 | 目标在船上读该船 `boatphysics:GetVelocity()`（≤`MAX_FORCE_VELOCITY`=3.5），步行携带读 `locomotor:GetRunSpeed()`（≤`MAX_VELOCITY`=1.2）；超过最小距离按 24 的距离尺度指数追赶，封顶 `2.5+0.5`。[L190-L238](../../../raw/current/scripts/components/boatmagnet.lua#L190-L238) |
| `OnUpdate(dt)` | 每更新帧 | 同船时推 `boatmagnet_pull_stop` 并返回；否则信标未关就推 `pull_start`；目标消失或距离 >48 解除；不同船时旋转磁铁模型。[L294-L334](../../../raw/current/scripts/components/boatmagnet.lua#L294-L334) |
| `onpairedwithbeaconfn`／`onunpairedwithbeaconfn`／`beaconturnedonfn`／`beaconturnedofffn` | 函数 | prefab 注入；按 `IsTurnedOff()` 与 `IsBeaconOnSameBoat` 决定 SGboatmagnet 的 `idle`／`pull`／`pull_pre`／`pull_pst`。[回调](../../../raw/current/scripts/prefabs/boat_magnet.lua#L39-L61) |
| `OnSave()`／`OnLoad(data)` | `{magnet_guid}` | 只存自身 GUID（键名沿用 `magnet_guid`），供信标重连；不存配对与船，船由 0 秒任务按平台重建。[L56-L70](../../../raw/current/scripts/components/boatmagnet.lua#L56-L70) |
| `ClearEverything()`／`OnRemoveFromEntity`／`OnRemoveEntity` | 清理 | 前者取消 0 秒任务、`SetBoat(nil)`、解除配对；移除组件只解除配对，实体移除才完整清理。[L14-L21](../../../raw/current/scripts/components/boatmagnet.lua#L14-L21)、[L72-L82](../../../raw/current/scripts/components/boatmagnet.lua#L72-L82) |

### 配对、重连与初始化

- 构造用 `DoTaskInTime(0)` 读 `GetCurrentPlatform()`，只接受带 `boat` tag 的平台，随后 `SetBoat`；0 秒任务之后不再重算平台，平台变化需显式再调 `SetBoat`。[L46-L53](../../../raw/current/scripts/components/boatmagnet.lua#L46-L53)
- 正常配对由磁铁发起；读档重连反过来由信标发起：信标 0 秒任务在自身 48 内找 `magnet_guid` 相同的磁铁并 `PairWithBeacon`。磁铁 `OnLoad` 不找信标，所以信标不在场或超距时读档后保持未配对。[重连](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L19-L38)
- 搜索走 `FindClosestEntity`：排除自身、死亡／鬼魂与不可见实体，再用 `canpairwithfn` 过滤掉已配对信标；比较点 y=0。[util](../../../raw/current/scripts/simutil.lua#L45-L66)

### 力的分工：磁铁算参数，boatphysics 施力

- 磁铁只产出三个参数：单位方向、`0.6` 的加速度标量、本帧速度上限；不写 `velocity_x/z`。[L190-L292](../../../raw/current/scripts/components/boatmagnet.lua#L190-L292)
- boatphysics 每帧遍历 `self.magnets`，跳过未配对或信标已关的项，把 `direction × force` 累加后交给 `ApplyMagnetForce`，按 `force*dt` 增量速度并夹到上限；`GetMaxVelocity` 另把磁铁上限与帆上限按 0.7 递减权重相加。[汇总](../../../raw/current/scripts/components/boatphysics.lua#L551-L563)、[施力](../../../raw/current/scripts/components/boatphysics.lua#L488-L504)、[上限](../../../raw/current/scripts/components/boatphysics.lua#L344-L384)
- 信标不提供力也不改速度：只登记 `magnet`／`magnet_guid`，维护开关与被拾取状态，供磁铁查询。[登记](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L145-L177)

## 典型接入

### 原版链路：部署磁铁→搜索配对→拖船

- 前提与选择：玩家用 `boat_magnet_kit` 在船上部署 `boat_magnet`；工厂在 `ismastersim` 后加组件，并把 4 个回调接到 SGboatmagnet，客户端没有该组件。[kit](../../../raw/current/scripts/prefabs/boat_magnet.lua#L145-L146)
- 端与单位：全链在服务端；半径与距离是世界单位（配对 24、断开 48），`MAGNET_FORCE` 0.6 是速度增量／秒，船速总上限 10。
- 生效与撤销：对磁铁执行激活动作→`search_pre`→`search_loop` 动画结束时找 24 内最近未配对信标；找不到进 `fail`，不建立连接。已配对时动作变 DEACTIVATE，立即解除；信标关闭只让力变 0，配对仍在。
- 原版依据：SG 的 `pull_pre`／`pull`／`pull_pst` 由 `boatmagnet_pull_start/stop` 与 4 个回调驱动；烧毁时 prefab 先 `SetBoat(nil)` 再移除组件。[SG](../../../raw/current/scripts/stategraphs/SGboatmagnet.lua#L1-L13)、[烧毁](../../../raw/current/scripts/prefabs/boat_magnet.lua#L21-L31)

### 两种跟随目标：信标在船上／被人拿着

- 信标放在另一条船：`GetFollowTarget` 返回信标所在船，方向按两船心算，限速读该船 `boatphysics:GetVelocity()`；最小距离会把两条船的 hull 半径相加。[目标](../../../raw/current/scripts/components/boatmagnet.lua#L175-L188)
- 信标被人拿着且脚下没有船：返回携带者实体，按 `locomotor:GetRunSpeed()` 且 ≤1.2 跟随；船会追着人走。拿起／放下只改变目标，不会自动解除配对。[L205-L223](../../../raw/current/scripts/components/boatmagnet.lua#L205-L223)

### 模组改配：先解除再配对

- 前提与选择：目标文件 `modmain.lua`，只在服务端调用；`PairWithBeacon` 自带“已配对也不拒绝”的缺口，换目标必须显式先解除。下面是完整辅助函数，需由同一文件的服务端逻辑调用，不自动运行。[L128-L173](../../../raw/current/scripts/components/boatmagnet.lua#L128-L173)
- 生效与撤销：执行后立即换绑并开始施力；删掉调用即撤销。前提是磁铁已由 0 秒任务挂到船上（`SetBoat` 完成），否则配对成功也不会产生力。

```lua
-- modmain.lua：服务端安全改配，示例调用 RepointMagnet(magnet_inst, beacon_inst)
local function RepointMagnet(magnet_inst, beacon_inst)
    local magnet = magnet_inst ~= nil and magnet_inst.components.boatmagnet or nil
    if magnet == nil or magnet.boat == nil or beacon_inst == nil
        or beacon_inst.components.boatmagnetbeacon == nil then
        return false
    end
    magnet:UnpairWithBeacon()
    magnet:PairWithBeacon(beacon_inst)
    return magnet:PairedBeacon() == beacon_inst
end
```

- 验证：示例只做 Lua 5.1 语法检查，未在游戏内执行。

## 常见陷阱与限制

- 二次配对会留下单边关系：`PairWithBeacon` 不检查本磁铁，直接覆盖 `self.beacon`；旧信标仍带 `paired` 与指向本磁铁的监听。换目标必须先 `UnpairWithBeacon()`。[L128-L148](../../../raw/current/scripts/components/boatmagnet.lua#L128-L148)
- 信标侧相反：`PairWithMagnet` 已有磁铁就早退。绕过 `FindNearestBeacon` 对已配对信标调 `PairWithBeacon`，磁铁会单方面施力，信标仍记着旧磁铁；之后解除还会拆掉旧配对。[信标侧](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L149-L162)
- `boatmagnet_pull_start/stop` 不是边沿事件：非同船且信标开启时每更新帧都推 start，消费方要幂等；原版 SG 只在 `idle` tag 时切状态。[L307-L313](../../../raw/current/scripts/components/boatmagnet.lua#L307-L313)
- 平台必须有 boatphysics：`SetBoat` 直接 `boat.components.boatphysics:AddMagnet`，带 `boat` tag 却没有 boatphysics 的自定义平台会在 0 秒任务里报错。[L84-L100](../../../raw/current/scripts/components/boatmagnet.lua#L84-L100)
- hull 半径有两套口径：`CalcMaxVelocity` 在两船都有 hull 时把半径相加，`CalcMagnetForce` 优先自身 hull、缺自身 hull 才用信标船 hull；信标船 `GetRadius()` 为 nil 时该加法报错，船体要先 `SetRadius`。[L211-L213](../../../raw/current/scripts/components/boatmagnet.lua#L211-L213)、[hull](../../../raw/current/scripts/components/hull.lua#L66)
- 多磁铁同船的合力口径：boatphysics 循环里 `magnet_direction` 先累加再乘当前磁铁的力，第二块磁铁的 0.6 乘在“已累加方向”上，不是各自方向；单磁铁不受影响。[汇总](../../../raw/current/scripts/components/boatphysics.lua#L551-L563)
- 读档只恢复“信标→磁铁”这一半：信标在 48 内找不到对应 GUID 就清空 `magnet_guid`，不保留曾配对状态；磁铁自身不重连。[重连](../../../raw/current/scripts/components/boatmagnetbeacon.lua#L19-L38)
- `OnInventoryBeaconLoaded` 在本快照没有调用者（只在构造里定义），不能当重连钩子；`prev_guid` 只是老存档字段兼容。[L40-L44](../../../raw/current/scripts/components/boatmagnet.lua#L40-L44)
- 运行期移除组件不清 boatphysics 表：`OnRemoveFromEntity` 只解除配对，不 `SetBoat(nil)`；boatphysics.magnets 里的项要到船移除或 `OnRemoveEntity` 才清，手动移除组件应先 `SetBoat(nil)`。[L72-L82](../../../raw/current/scripts/components/boatmagnet.lua#L72-L82)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 1 处（actions.lua BOAT_MAGNET 激活／解除）；结论修订：无，仅锚点迁移。依据：枢纽 diff 审阅＋变更清单命中 actions.lua、simutil.lua（命中区域逐行核对均未漂移）。
- 源码审阅：2026-10-21，steam-buildid:25643519。完整阅读 `components/boatmagnet.lua`（341 行），覆盖构造与 0 秒任务、配对与解除、SetBoat 与 boatphysics 增删、方向／力／限速公式、OnUpdate 起停与断开、存读与清理；核对宿主 `prefabs/boat_magnet.lua` 的 AddComponent（L115）与注册名 `boat_magnet`／`boat_magnet_kit`，以及 `boatphysics`、`SGboatmagnet`、`actions`、`componentactions`、`boatmagnetbeacon` 的消费点。未覆盖部署动画、音效与 scrapbook 表现。
- 游戏运行：未运行；本页结论均为源码静态核验，未测手感与网络复制表现。
- 结构检查：`validate_page.py` 通过（0 issue；107 行、14104 字节，低于 180 行／14 KiB 上限）；`validate_source_links.py --strict` 通过（43 处引用、35 个目标、0 error／0 warning）；正文 Lua 示例另存 510 字节，`parser_probe --syntax-only` 通过（只查 Lua 5.1 语法，未执行）。

相关：[[reference/components/boatphysics]]、[[reference/components/boatmagnetbeacon]]、[[reference/components/hull]]。
