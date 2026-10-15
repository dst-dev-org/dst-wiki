---
title: "embarker"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "执行朝平台登乘点或固定离船点的水平运动，发出运动结束通知，并配合 locomotor／状态图完成或取消跳跃。"
aliases: ["embarker", "登船运动", "离船落点", "done_embark_movement", "跳船运动收尾"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/embarker.lua", "components/locomotor.lua", "components/walkableplatform.lua", "prefabs/player_common.lua", "stategraphs/commonstates.lua", "stategraphs/SGwilson.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# embarker

## 快速判断

- 职责与适用：装在正在登乘／离船的实体上，记录目标平台或固定x／z坐标，以 motor 速度向落点运动；平台扫描、是否可跳、碰撞切换与动画由外部协作，不是通用安全登船接口。[embarker.lua:1-32](../../../raw/current/scripts/components/embarker.lua#L1-L32) [locomotor.lua:1396-1414](../../../raw/current/scripts/components/locomotor.lua#L1396-L1414) [commonstates.lua:862-911](../../../raw/current/scripts/stategraphs/commonstates.lua#L862-L911)
- 优先使用：已有 locomotor／SG 的宿主沿 StartHopping → SG StartMoving → done事件 → Embark／Cancel 的接线使用；水陆状态的另一层协作见 [[reference/components/amphibiouscreature]]。[locomotor.lua:1396-1414](../../../raw/current/scripts/components/locomotor.lua#L1396-L1414) [commonstates.lua:877-888](../../../raw/current/scripts/stategraphs/commonstates.lua#L877-L888) [commonstates.lua:918-924](../../../raw/current/scripts/stategraphs/commonstates.lua#L918-L924)
- 重要副作用：到点只传送并发 `done_embark_movement`，不会自行停止组件更新或清目的地；超过位移上限也会 Cancel 后发同名事件，事件不等于已成功登船。[embarker.lua:10-32](../../../raw/current/scripts/components/embarker.lua#L10-L32)
- 修改边界：原版玩家服务端有本组件，启用客户端移动预测也会动态添加；不能统一视为仅服务端组件。预测关闭由玩家宿主清SG、移组件并停Physics。[player_common.lua:877-944](../../../raw/current/scripts/prefabs/player_common.lua#L877-L944) [player_common.lua:2611-2615](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2615) [player_common.lua:2678-2682](../../../raw/current/scripts/prefabs/player_common.lua#L2678-L2682)

## 关键接口

| 接口／字段 | 签名或约束 | 条件、作用与证据 |
|---|---|---|
| `SetEmbarkable(platform)` | 有效目标须带 walkableplatform | 保存平台，立即取登乘点并转向，清固定离船坐标。不会验证容量、距离、可通行或直接开始移动。[embarker.lua:34-41](../../../raw/current/scripts/components/embarker.lua#L34-L41) [embarker.lua:79-91](../../../raw/current/scripts/components/embarker.lua#L79-L91) |
| `SetDisembarkPos(x,z)` | 固定地图坐标 | 保存坐标并转向，清平台及缓存平台点；不查地形／水面，也不开始运动。[embarker.lua:43-52](../../../raw/current/scripts/components/embarker.lua#L43-L52) |
| `SetDisembarkActionPos(x,z)` | 使用全局辅助函数计算坐标 | 仅改 disembark_x/z，不像 SetDisembarkPos 那样清平台与缓存或主动转向；已有有效平台仍有更高取点优先级。[embarker.lua:54-56](../../../raw/current/scripts/components/embarker.lua#L54-L56) [embarker.lua:79-91](../../../raw/current/scripts/components/embarker.lua#L79-L91) |
| `HasDestination()` | 返回布尔值 | 有效平台或非nil disembark_x 即为真；缓存落点不计入，不代表正在移动或落点安全。[embarker.lua:75-77](../../../raw/current/scripts/components/embarker.lua#L75-L77) |
| `GetEmbarkPosition()` | 返回 x,z 两个数值 | 有效平台动态取平台登乘点；否则取固定离船坐标、上次缓存点，再回退当前位置。无需 HasDestination 为真也能返回坐标。[embarker.lua:79-91](../../../raw/current/scripts/components/embarker.lua#L79-L91) |
| `StartMoving()` | 需要 Physics、locomotor、位置接口 | 未开始时停Physics并开始更新，记录起点与1.5倍当前 GetHopDistance 的平方为位移上限，发 `start_embark_movement`；已开始时不重置上限。[embarker.lua:58-73](../../../raw/current/scripts/components/embarker.lua#L58-L73) |
| `embark_speed`／`embarker_min_dist` | 默认10的速度／可选平台内缩距离 | 前者用于 speed*dt 估算本帧行程并设置前向motor速度；后者透传平台取点，默认平台边缘内缩0.5再加该值。[embarker.lua:5](../../../raw/current/scripts/components/embarker.lua#L5 [embarker.lua:14-29](../../../raw/current/scripts/components/embarker.lua#L14-L29) [walkableplatform.lua:135-141](../../../raw/current/scripts/components/walkableplatform.lua#L135-L141) |
| `Embark()`／`Cancel()` | 需要 locomotor；无参数 | 都停Physics、清目的地／缓存／运动上限、停止更新并 FinishHopping；Embark 先按当前取点最终定位，Cancel 不改回起点。[embarker.lua:93-120](../../../raw/current/scripts/components/embarker.lua#L93-L120) |

## 关键流程

### 目的地与运动更新

平台目标的取点由 walkableplatform 根据“平台中心指向登乘者”的方向，取平台半径减0.5再减 embarker_min_dist 的圆周点；每帧重取，可跟随平台位置变化。若平台失效，组件优先回退到保存的固定坐标／最近缓存点，而不是自动判定取消；HasDestination 此时可能为false，但 GetEmbarkPosition 仍返回缓存位置。[walkableplatform.lua:135-141](../../../raw/current/scripts/components/walkableplatform.lua#L135-L141) [embarker.lua:10-16](../../../raw/current/scripts/components/embarker.lua#L10-L16) [embarker.lua:75-91](../../../raw/current/scripts/components/embarker.lua#L75-L91)

StartMoving 不校验 HasDestination，也不立即执行一帧运动。更新取剩余水平距离：若不大于 `embark_speed*dt`，用 TeleportRespectingInterpolation 定位到目标y=0并发done事件；否则检查“当前位置距起点的直线位移平方”是否超限，超限则 Cancel 后发done事件，未超限则转向目标并 SetMotorVel(embark_speed,0,0)。该上限不是累计路程、离目标最大距离或安全落地测试。[embarker.lua:10-32](../../../raw/current/scripts/components/embarker.lua#L10-L32) [embarker.lua:58-73](../../../raw/current/scripts/components/embarker.lua#L58-L73)

到点分支本身不 Stop／清状态，若消费者不 Embark／Cancel，后续更新还会继续取点并可能重复通知。最终 Embark 再取一次当前位置对应的目标点并 Transform:SetPosition；它只完成定位与 FinishHopping，不显式调用平台成员登记。Cancel 则原地停下，不回滚先前已经移动的距离。[embarker.lua:19-21](../../../raw/current/scripts/components/embarker.lua#L19-L21) [embarker.lua:93-120](../../../raw/current/scripts/components/embarker.lua#L93-L120)

### 离船坐标辅助函数的有限检查

全局 `GetDisembarkPosAndDistance(inst,x,z)` 以4为固定阈值：目标离宿主小于4时原样返回目标坐标和4；否则在线段上取离宿主4的点，仅当该点无平台且 IsAboveGroundAtPoint 为真时返回该点及原目标距离，失败则返回原目标坐标和0。返回第三值不是统一意义的最终行程长度，近点和失败回退也没有完整安全检查。[embarker.lua:122-144](../../../raw/current/scripts/components/embarker.lua#L122-L144)

SetDisembarkActionPos 只接收前两个返回值，并不处理第三值；它也不清已有平台目标。需要明确切到固定坐标的调用应理解两种setter的差异，不能把辅助函数或这个方法当寻路／安全岸点搜索。[embarker.lua:43-56](../../../raw/current/scripts/components/embarker.lua#L43-L56)

### 完成、取消与生命周期

locomotor.StartHopping 设置平台或固定落点，再向非jumping宿主发 onhop，标记 hopping 并设置0.2秒再跳等待。Embark／Cancel 最后调用 FinishHopping，后者清 hopping；有预约平台时，按当前平台关系尝试释放预约并清引用。停止本组件更新与释放预约是配套收尾，不能只停Physics代替。[locomotor.lua:1396-1414](../../../raw/current/scripts/components/locomotor.lua#L1396-L1414) [locomotor.lua:1186-1198](../../../raw/current/scripts/components/locomotor.lua#L1186-L1198) [embarker.lua:93-120](../../../raw/current/scripts/components/embarker.lua#L93-L120)

本组件无 OnSave／OnLoad、死亡／睡眠或 OnRemoveFromEntity 钩子，目的地与中途运动不会由它存档或自动按Cancel清理。平台碰撞mask的恢复、ignorewalkableplatforms标签撤销与动画打断由SG负责；原版关闭客户端预测时也由宿主整体清理，不能只删除 embarker 就假定运动和预约都已恢复。[embarker.lua:1-8](../../../raw/current/scripts/components/embarker.lua#L1-L8) [embarker.lua:93-120](../../../raw/current/scripts/components/embarker.lua#L93-L120) [commonstates.lua:895-911](../../../raw/current/scripts/stategraphs/commonstates.lua#L895-L911) [player_common.lua:921-944](../../../raw/current/scripts/prefabs/player_common.lua#L921-L944)

## 典型接入

### 原版玩家跳船：动画决定启动和收尾

玩家服务端安装本组件并初设速度为跑速；SGwilson 在hop_pre开始时重新按 `RunSpeed()*GetSpeedMultiplier()+1.5` 计算，钳制到4～10，不能认为全程永远使用构造默认10或普通跑速6。[player_common.lua:2678-2682](../../../raw/current/scripts/prefabs/player_common.lua#L2678-L2682) [SGwilson.lua:28939-28953](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28939-L28953) [tuning.lua:503-508](../../../raw/current/scripts/tuning.lua#L503-L508)

SGwilson调用公共 AddHopStates，并让pre第4帧开始运动。公共状态保存／切换碰撞mask，pre或loop调用StartMoving；loop收到done事件后切hop_pst，后者调用Embark。中途打断pre／loop则调用Cancel，并由状态图恢复碰撞与标签；落在水面时还可转宿主跌落状态。上述边界是原版整条接线的一部分，不由embarker单独提供。[SGwilson.lua:29003-29014](../../../raw/current/scripts/stategraphs/SGwilson.lua#L29003-L29014) [commonstates.lua:820-859](../../../raw/current/scripts/stategraphs/commonstates.lua#L820-L859) [commonstates.lua:862-942](../../../raw/current/scripts/stategraphs/commonstates.lua#L862-L942)

这是内部移动流程参照；自定义宿主需另具备兼容Physics、locomotor、平台选择、SG和资源，本页不提供直接调用StartMoving的通用登船demo。两栖生物的跳跃状态收尾有所不同，沿 [[reference/components/amphibiouscreature#岸边跳跃由移动与状态图协作]] 查询。

## 常见陷阱与限制

- **done事件就是登船成功：** 到点和超位移取消共用事件；应结合SG流程、实际位置和平台关系判断，不以该事件直接授予“已登船”业务结果。[embarker.lua:19-26](../../../raw/current/scripts/components/embarker.lua#L19-L26) [commonstates.lua:885-889](../../../raw/current/scripts/stategraphs/commonstates.lua#L885-L889)
- **取消会恢复出发点或碰撞：** Cancel只停运动、清状态并FinishHopping；位置不会回滚，碰撞mask由外部SG保存／恢复。[embarker.lua:109-120](../../../raw/current/scripts/components/embarker.lua#L109-L120) [commonstates.lua:895-906](../../../raw/current/scripts/stategraphs/commonstates.lua#L895-L906)
- **所有组件都只应放服务端：** embarker用于玩家客户端预测的实际运动执行；模组应保留原版预测模式的安装／拆卸边界，不能向任意客户端实体无条件重复添加。[player_common.lua:877-944](../../../raw/current/scripts/prefabs/player_common.lua#L877-L944)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件全文、locomotor StartHopping／FinishHopping、平台取点、玩家服务端与预测安装／关闭、公共完整AddHopStates、玩家hop速度与注册配置，覆盖目标优先级、运动、done事件两种来源、显式完成／取消及无组件存读清理边界。
- 游戏运行：未运行；未测移动平台追逐、超位移取消、网络预测校正、预约释放或存档重载中途跳跃。未穷举平台扫描与全部宿主SG。
- 结构检查：`validate_page.py`通过；两页合并执行`validate_source_links.py --strict`，92个行号引用、0错误／0警告。本页无新增Lua示例；机械检查不代替游戏测试。

相关：[[reference/components/locomotor]] · [[reference/components/walkableplatform]] · [[reference/components/amphibiouscreature]]
