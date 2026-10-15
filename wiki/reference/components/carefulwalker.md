---
title: "carefulwalker"
created: 2026-10-26
updated: "2026-11-01"
type: component
summary: "响应不平地面通知，追踪目标范围与有效期，在满足移动能力条件时施加谨慎行走减速并通知玩家表现层。"
aliases: ["carefulwalker", "不平地面减速", "谨慎行走", "carefulwalking", "地形范围限时减速"]
tags: [component, lifecycle, event, network-sync, server-client]
dst_build: "steam-buildid:26335394"
sources: ["components/carefulwalker.lua", "components/unevenground.lua", "components/locomotor.lua", "prefabs/player_common.lua", "prefabs/player_classified.lua", "stategraphs/SGwilson.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# carefulwalker

## 快速判断

- 职责与适用：接收宿主 `unevengrounddetected` 通知，短暂追踪不平地面目标；距离内有有效目标且满足移动配置时，添加 `carefulwalking` 速度来源。它不采样地图崎岖度、不绕障或保证寻路安全。[carefulwalker.lua:1-13](../../../raw/current/scripts/components/carefulwalker.lua#L1-L13) [carefulwalker.lua:54-118](../../../raw/current/scripts/components/carefulwalker.lua#L54-L118)
- 优先使用：沿 [[reference/components/unevenground]] 的通知与玩家现有组件接线来控制谨慎行走；一般独立速度效果用 [[reference/components/locomotor]] 的来源倍率，不必伪造地面目标。[carefulwalker.lua:72-88](../../../raw/current/scripts/components/carefulwalker.lua#L72-L88) [unevenground.lua:13-26](../../../raw/current/scripts/components/unevenground.lua#L13-L26)
- 重要副作用：状态翻转时增删宿主自身来源、key=`carefulwalking` 的移动倍率，并发 `{careful=true/false}` 事件；同状态不重复发送。默认倍率0.3，与其他 locomotor 来源相乘。[carefulwalker.lua:5-13](../../../raw/current/scripts/components/carefulwalker.lua#L5-L13) [carefulwalker.lua:72-88](../../../raw/current/scripts/components/carefulwalker.lua#L72-L88) [locomotor.lua:480-487](../../../raw/current/scripts/components/locomotor.lua#L480-L487) [tuning.lua:3363](../../../raw/current/scripts/tuning.lua#L3363
- 修改边界：原版玩家在服务端添加组件，客户端表现另由 player_classified 转发；组件本身没有 replica、存读或睡眠钩子。移除钩子已改用 `self.inst`（旧版裸 `inst` 的缺陷在本 build 修复），但钩子仍不主动撤减速来源——停用应走 `Enable(false)`（清目标、停更新并撤 careful），直接 RemoveComponent 仍不是已验证撤销方案。[player_common.lua:2611-2615](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2615) [player_common.lua:2737-2744](../../../raw/current/scripts/prefabs/player_common.lua#L2737-L2744) [carefulwalker.lua:15-17](../../../raw/current/scripts/components/carefulwalker.lua#L15-L17) [carefulwalker.lua:19-30](../../../raw/current/scripts/components/carefulwalker.lua#L19-L30)

## 关键接口

| 接口／字段 | 签名或约束 | 条件、作用与证据 |
|---|---|---|
| `TrackTarget(target,radius,duration)` | 实体、地图半径、秒 | 以目标实体为键保存半径平方及 `duration+0.05`；新目标开始组件更新，同目标通知覆盖半径和剩余有效期，不累加；组件被 `Enable(false)` 关闭时直接忽略通知。[carefulwalker.lua:54-66](../../../raw/current/scripts/components/carefulwalker.lua#L54-L66) |
| `SetCarefulWalkingSpeedMultiplier(mult)` | 数值钳制到0～1 | 没有追踪目标时只改字段；有目标且正在 careful 时更新当前倍率；有目标且设为1则关闭 careful；从原倍率≥1改低可立即 OnUpdate(0)。没有输入类型校验。[carefulwalker.lua:32-52](../../../raw/current/scripts/components/carefulwalker.lua#L32-L52) |
| `IsCarefulWalking()` | 返回 `careful` 布尔值 | 只查当前组件标记，不重新判断距离、剩余时长或运动状态；初始为false。[carefulwalker.lua:5-13](../../../raw/current/scripts/components/carefulwalker.lua#L5-L13) [carefulwalker.lua:68-70](../../../raw/current/scripts/components/carefulwalker.lua#L68-L70) |
| `ToggleCareful(careful)` | 真值开启／假值关闭 | 直接变更标记、速度来源和事件，不做 TrackTarget／FasterOnRoad 门槛检查；后续 OnUpdate 会重新按追踪结果决定状态，不能当永久开关。[carefulwalker.lua:72-118](../../../raw/current/scripts/components/carefulwalker.lua#L72-L118) |
| `unevengrounddetected` | 宿主收到 `{inst=target,radius=...,period=...}` | 组件把 `period` 作为 TrackTarget 的 duration；事件没有来源过滤或参数防御，发送者需提供可用实体与数值。[carefulwalker.lua:1-13](../../../raw/current/scripts/components/carefulwalker.lua#L1-L13) |

## 关键流程

### 目标时效、距离与真正的减速资格

更新时先要求倍率<1、宿主有 locomotor 且 `locomotor:FasterOnRoad()` 为真，才检查距离。服务端 FasterOnRoad 返回自身 `fasteronroad`，骑乘时取当前坐骑 locomotor 的该配置；它不是“此刻脚下是否为道路”的判定。因此不能将此减速限定为站在路上，也不能说所有范围内实体都会受影响。[carefulwalker.lua:90-100](../../../raw/current/scripts/components/carefulwalker.lua#L90-L100) [locomotor.lua:116-124](../../../raw/current/scripts/components/locomotor.lua#L116-L124)

对每个目标，只有 `remaining>dt` 且目标仍有效，才减去 dt 并检查宿主到目标的距离平方严格小于 radius²；命中一个就足够开启 careful，多目标不重复叠加同一减速。其余目标到期／失效时移出表；全部清空后停止组件更新，最后 ToggleCareful(false) 撤速度并通知。即使倍率1或没有 locomotor，目标仍继续老化。[carefulwalker.lua:90-118](../../../raw/current/scripts/components/carefulwalker.lua#L90-L118)

TrackTarget 不立即计算距离，通常等下一次组件更新才进入状态；0.05秒是记录时额外加的容差，不代表固定更新周期。组件没有独立取消单个目标的方法，也没有 `OnSave`／`OnLoad` 保存目标或剩余时长，重建时回到空目标／false，依赖新的环境通知。[carefulwalker.lua:5-13](../../../raw/current/scripts/components/carefulwalker.lua#L5-L13) [carefulwalker.lua:54-66](../../../raw/current/scripts/components/carefulwalker.lua#L54-L66) [carefulwalker.lua:90-118](../../../raw/current/scripts/components/carefulwalker.lua#L90-L118)

### 原版不平地面通知的生产端

unevenground 默认影响半径3、检测玩家半径15、通知周期0.6秒；它对检测范围内非 `playerghost` 的 AllPlayers 发事件，事件携带真实影响半径与周期。故“收到通知”只表示进入较大的检测圈，接收端仍要检查更小的 radius。生产端会立即通知一次，再周期刷新；休眠／禁用时停止通知，消费者旧记录靠到期撤销，不是生产端直接发关闭事件。[unevenground.lua:1-26](../../../raw/current/scripts/components/unevenground.lua#L1-L26) [unevenground.lua:34-64](../../../raw/current/scripts/components/unevenground.lua#L34-L64)

### 玩家事件、网络标记与状态图表现

player_classified 的服务端监听玩家 `carefulwalking`，写 `iscarefulwalking` net_bool；客户端该变量变脏时再向父玩家发相同数据事件。玩家公开 `IsCarefulWalking()` 读取 classified 值，而非要求客户端也装 carefulwalker。[player_classified.lua:152-154](../../../raw/current/scripts/prefabs/player_classified.lua#L152-L154) [player_classified.lua:661-665](../../../raw/current/scripts/prefabs/player_classified.lua#L661-L665) [player_classified.lua:1189](../../../raw/current/scripts/prefabs/player_classified.lua#L1189 [player_classified.lua:1223](../../../raw/current/scripts/prefabs/player_classified.lua#L1223 [player_classified.lua:1633-1634](../../../raw/current/scripts/prefabs/player_classified.lua#L1633-L1634) [player_common.lua:282-284](../../../raw/current/scripts/prefabs/player_common.lua#L282-L284) [player_common.lua:2285](../../../raw/current/scripts/prefabs/player_common.lua#L2285

SGwilson 的 ConfigureRunState 在骑乘、重物、吟唱、变身、风暴、teetering、groggy 等分支之后才选 careful 表现（本 build 起装备 `flyingbroom` 时跳过 careful 分支），并加 `noslip`；run 状态内 carefulwalking 事件按其当前状态条件重新进入 run。组件标记与“当前一定播放谨慎动画／一定防滑”不能划等号，这些表现取决于 SG 的选择。[SGwilson.lua:445-514](../../../raw/current/scripts/stategraphs/SGwilson.lua#L445-L514) [SGwilson.lua:11750-11763](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11750-L11763)

## 典型接入

### 原版玩家：沿现有环境通知与倍率恢复协议

玩家服务端工厂已有 carefulwalker，环境实体用 unevenground 通知玩家；这是实际生产／消费参照，本页不另造一个兼容全部人物模式的 demo。若修改谨慎行走倍率，应在目标服务端已有组件上调用 setter，数值是移动倍率而非百分比点数；恢复用当前宿主应有的默认／模式配置，不要直接删除组件来撤销减速。[player_common.lua:2611-2615](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2615) [player_common.lua:2737-2744](../../../raw/current/scripts/prefabs/player_common.lua#L2737-L2744) [carefulwalker.lua:32-52](../../../raw/current/scripts/components/carefulwalker.lua#L32-L52)

原版玩家的物品吟唱开始回调会用 MinMult 调整本组件倍率，使其与吟唱减速组合后达到较低者；停止回调恢复 `TUNING.CAREFUL_SPEED_MOD`。这说明临时效果有宿主自己的成对恢复边界，不能将构造时0.3无条件覆盖到所有模式中。[player_common.lua:294-326](../../../raw/current/scripts/prefabs/player_common.lua#L294-L326)

## 常见陷阱与限制

- **减速半径等于通知半径：** 通知范围与实际影响半径分开；目标自身位置动态参与距离判断，同一目标反复通知只是刷新一个记录。[unevenground.lua:1-18](../../../raw/current/scripts/components/unevenground.lua#L1-L18) [carefulwalker.lua:54-66](../../../raw/current/scripts/components/carefulwalker.lua#L54-L66) [carefulwalker.lua:90-100](../../../raw/current/scripts/components/carefulwalker.lua#L90-L100)
- **手动 ToggleCareful(false) 会永久免疫：** 仍有有效目标时，下一次更新可再次开启；倍率设为1可令自动资格为假，但已有目标仍老化，降低倍率后可再次进入判定。[carefulwalker.lua:32-52](../../../raw/current/scripts/components/carefulwalker.lua#L32-L52) [carefulwalker.lua:90-118](../../../raw/current/scripts/components/carefulwalker.lua#L90-L118)
- **停用要走 `Enable(false)`，不要 RemoveComponent：** `Enable(false)` 清空目标、停组件更新并撤 careful 减速；`Enable(true)` 只回位标志，不恢复被清目标。`OnRemoveFromEntity` 本 build 起用 `self.inst` 正确撤监听（旧版裸 `inst` 缺陷已修复），但仍不撤减速来源，直接 RemoveComponent 仍不是已验证撤销方案。[carefulwalker.lua:19-30](../../../raw/current/scripts/components/carefulwalker.lua#L19-L30) [速度撤销实现 carefulwalker.lua:81-87](../../../raw/current/scripts/components/carefulwalker.lua#L81-L87)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件全文、unevenground 全文、locomotor FasterOnRoad／来源倍率实现、玩家服务端安装与吟唱倍率配对、classified 事件／net_bool 接线、SGwilson ConfigureRunState 和一个 run 事件消费者，覆盖目标时效、真实减速资格、网络分工、表现选择及移除／存读边界。
- 游戏运行：未运行；未复现移除钩子缺口，未测网络延迟、通知中断、骑乘／变身或所有 SG 分支表现。客户端 SG 和其他事件生产者未逐一展开。
- 结构检查：`validate_page.py` 通过；两页合并执行 `validate_source_links.py --strict`，87个行号引用、0错误／0警告。本页无新增 Lua 示例；机械检查不代替游戏测试。

- 2026-10-28 build 26230496：SGwilson ConfigureRunState 谨慎分支区间随 inventory 局部变量调整；组件逻辑未变（该轮注记把 flyingbroom 判定提前记到本版，实为下条 build 的变化，特此纠正）。
- 2026-11-01 build 26335394：组件新增 `enabled` 字段与 `Enable(enable)` 开关（false 清目标、停更新、撤 careful）、`TrackTarget` 增加 enabled 门、`OnRemoveFromEntity` 裸 `inst` 缺陷修复（改 `self.inst`）；SGwilson ConfigureRunState 谨慎分支加 flyingbroom 豁免、ConfigureWitchBroomFlying 开关时同步 Enable 本组件并挂 nonslipgrituser 源。本页全文按新版重对锚点；见 updates/build-26335394 C-3、SB-8。
相关：[[reference/components/locomotor]] · [[reference/components/slipperyfeet]]
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
