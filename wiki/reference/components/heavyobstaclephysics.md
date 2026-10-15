---
title: "heavyobstaclephysics"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "切换可搬重物的物品／障碍／下落碰撞状态，处理落地后的避人调径，推行动力及落地检测由其他组件负责。"
aliases: ["重物碰撞切换", "重物落地卡人", "可搬障碍物理", "heavyobstaclephysics"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/heavyobstaclephysics.lua", "standardcomponents.lua", "prefabs/potatosack.lua", "prefabs/cavein_boulder.lua", "prefabs/snowman.lua", "components/pushable.lua", "components/lootdropper.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# heavyobstaclephysics

## 快速判断

- 职责与适用：给已配重物 Physics 的实体切换 ITEM／OBSTACLE／FALLING，响应收起、丢下及可选推行／下落事件；不负责创建物理体、施加推力或判断何时落地。[heavyobstaclephysics.lua:1](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L1 [heavyobstaclephysics.lua:225](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L225
- 优先使用：沿原版先 MakeHeavyObstaclePhysics，再在主模拟加组件并 SetRadius；需要下落／推行事件才分别 AddFallingStates／AddPushingStates。[potatosack.lua:44](../../../raw/current/scripts/prefabs/potatosack.lua#L44 [potatosack.lua:61](../../../raw/current/scripts/prefabs/potatosack.lua#L61 [heavyobstaclephysics.lua:237](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L237
- 重要副作用：障碍化会延后半秒，再按附近角色间隙调整半径／角色碰撞，满半径后停止扫描；物理质量、碰撞组／掩码会被改写。SetRadius 只是逻辑配置，不实际调整胶囊。[heavyobstaclephysics.lua:62](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L62 [heavyobstaclephysics.lua:91](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L91 [heavyobstaclephysics.lua:225](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L225
- 修改边界：原版将 Physics 基础初始化放在客户端返回之前，状态组件放主模拟；组件不是完整网络物理复制层，也不保证任意尺寸／任意 Physics 都适用。[potatosack.lua:36](../../../raw/current/scripts/prefabs/potatosack.lua#L36 [standardcomponents.lua:698](../../../raw/current/scripts/standardcomponents.lua#L698

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetRadius(radius)` | 世界距离 | 设置 maxradius/currentradius，逻辑状态直接记作 OBSTACLE，监听 onputininventory／ondropped；**不调用 SetCapsule、设置质量或碰撞组**，不是通用实时缩放接口。[heavyobstaclephysics.lua:225](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L225 |
| `MakeSmallObstacle()` | 无参数 | 只设 issmall；后续障碍化使用 SMALLOBSTACLES 组，mask 为 ITEMS。初始物理应另配 MakeSmallHeavyObstaclePhysics，而非仅改该字段。[heavyobstaclephysics.lua:91](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L91 [heavyobstaclephysics.lua:233](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L233 [standardcomponents.lua:719](../../../raw/current/scripts/standardcomponents.lua#L719 |
| `AddFallingStates()`／`AddPushingStates()` | 无参数 | 分别监听 startfalling/stopfalling、startpushing/stoppushing；本方法不产生事件，不新增 SG 状态或发动运动。[heavyobstaclephysics.lua:237](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L237 |
| `GetPhysicsState()`／`IsItem/IsObstacle/IsFalling()` | 状态名或 nil／布尔 | 只读组件逻辑状态；构造时 state=nil，SetRadius 后才初记 OBSTACLE；不能代替检查实际质量／碰撞配置。[heavyobstaclephysics.lua:187](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L187 [heavyobstaclephysics.lua:247](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L247 |
| `SetOnPhysicsStateChangedFn(fn)` | `fn(inst, "ITEM"/"OBSTACLE"/"FALLING")` | SetPhysicsState 写状态后通知；SetRadius 的直接赋值不会触发它。[heavyobstaclephysics.lua:20](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L20 [heavyobstaclephysics.lua:263](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L263 |
| 其他 SetOn 回调 | `fn(inst)` | SetOnChangeToItemFn／SetOnChangeToObstacleFn／SetOnStartFallingFn／SetOnStopFallingFn 在对应转换前调用；推行设置器实际拼写为 `Setonstartpushingfn`、`Setonstoppushingfn`，同样是单槽回调。回调移除实体时对应处理会提前返回。[heavyobstaclephysics.lua:35](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L35 [heavyobstaclephysics.lua:113](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L113 [heavyobstaclephysics.lua:127](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L127 [heavyobstaclephysics.lua:161](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L161 [heavyobstaclephysics.lua:267](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L267 |
| `ForceDropPhysics()` | 无参数 | 同步 ChangeToItem，再 ChangeToObstacle 安排延迟障碍化；不直接进入 FALLING，也不设置位置／抛射速度。[heavyobstaclephysics.lua:291](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L291 |

## 关键流程

### 收起、丢下与避人恢复碰撞

基础 MakeHeavyObstaclePhysics 加 blocker 标签、Physics，设质量 0、OBSTACLES 组，碰撞 ITEMS／CHARACTERS／GIANTS，胶囊半径取参数、高度默认 2；小障碍版本使用 SMALLOBSTACLES 且初始不碰 GIANTS。后续组件需要改半径时 SetCurrentRadius 固定用高度 2，不能假设自定义初始高度永远保留。[standardcomponents.lua:698](../../../raw/current/scripts/standardcomponents.lua#L698 [standardcomponents.lua:719](../../../raw/current/scripts/standardcomponents.lua#L719 [heavyobstaclephysics.lua:13](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L13

收起事件取消未完成障碍任务，转 ITEM；通常恢复 maxradius、质量 1、阻尼 0、ITEMS 组，mask 为 WORLD／OBSTACLES／SMALLOBSTACLES。从 FALLING 转 ITEM 时复用已有质量／半径／组，只恢复 mask。重复收起仍先执行 onchangetoitemfn，不是只在状态真正变化时回调。[heavyobstaclephysics.lua:35](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L35

丢下先执行 onchangetoobstaclefn、取消旧任务；只有逻辑状态非 OBSTACLE 才安排 0.5 秒后的转换。转换时设质量 0、相应障碍组，先不碰 CHARACTERS；随后立即并每 0.5 秒扫描距离 2 内可见且带 character、locomotor 的实体，排除 INLIMBO／NOCLICK／flying／ghost。[heavyobstaclephysics.lua:60](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L60 [heavyobstaclephysics.lua:91](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L91 [heavyobstaclephysics.lua:113](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L113

候选中心距离减去角色 Physics 半径后取最小间隙，夹在 0～maxradius。间隙大于 0 才调整自身胶囊并启用角色碰撞；为 0 时保持暂时穿人，不把胶囊写成 0；达到满半径后停止扫描。这里扫描固定半径 2，并不随 maxradius 扩张，不能据此承诺任意巨大自定义障碍都能安全避人。[heavyobstaclephysics.lua:62](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L62

### 下落、推行与重新唤醒

startfalling 先执行宿主回调、取消障碍任务；必要时恢复质量 1、ITEMS 组与满半径，再清空碰撞 mask，进入 FALLING。stopfalling 先交给宿主回调，实体仍有效才直接执行障碍化，不再等待丢下路径的半秒。组件没有高度／速度阈值判断，事件发出时机由外部负责。[heavyobstaclephysics.lua:127](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L127 [heavyobstaclephysics.lua:149](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L149

startpushing 调推行回调后转 ITEM；stoppushing 调结束回调，仅在 inventoryitem 不存在或未被持有时转回障碍路径。它不根据推行者计算推力；原版 pushable 才维护 doer、校验其 pushing_walk SG 标签、设置 SetMotorVel，停止时 Physics:Stop 并发 stoppushing。[heavyobstaclephysics.lua:161](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L161 [pushable.lua:48](../../../raw/current/scripts/components/pushable.lua#L48 [pushable.lua:64](../../../raw/current/scripts/components/pushable.lua#L64 [pushable.lua:77](../../../raw/current/scripts/components/pushable.lua#L77

实体醒来时，若未被持有、无旧浮空兼容标记、非 FALLING 且 y>0.01，会 ForceDropPhysics 并将速度清零，交重力使其下降；这不是主动传送到地面。OnSave／OnLoad 仅保存 deprecated_floating_exploit 为真的兼容标记，不保存当前状态、半径、任务、回调或推行者。[heavyobstaclephysics.lua:214](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L214 [heavyobstaclephysics.lua:297](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L297

## 典型接入

### 原版重物的三种配套

- **土豆袋基础组装**：potatosack 用相同的 0.45 半径初始化重物 Physics、physicsradiusoverride 和服务端组件；另有 heavy 标签、inventoryitem 与 BODY equippable。重物可搬运的对象契约来自这些配套，不是仅加状态组件。[potatosack.lua:8](../../../raw/current/scripts/prefabs/potatosack.lua#L8 [potatosack.lua:36](../../../raw/current/scripts/prefabs/potatosack.lua#L36 [potatosack.lua:61](../../../raw/current/scripts/prefabs/potatosack.lua#L61
- **落石下落回调**：cavein_boulder 在两端初始化 Physics，服务端 SetRadius、AddFallingStates 并装状态／下落回调。OnStopFalling 在玩家重叠等情况下可直接移除实体，此时组件检测无效后不再障碍化；完整落地判定／石阵机制不在本页展开。[cavein_boulder.lua:571](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L571 [cavein_boulder.lua:494](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L494
- **雪球尺寸与推行**：snowman 服务端 AddPushingStates，另装 pushable；RefreshPhysicsSize 更新 override，并在主模拟分支同时 SetCapsule 与 SetRadius。客户端监听 basesizedirty 更新半径 override；推动动画／成长仍由宿主回调与 pushable 负责。[snowman.lua:235](../../../raw/current/scripts/prefabs/snowman.lua#L235 [snowman.lua:209](../../../raw/current/scripts/prefabs/snowman.lua#L209 [snowman.lua:636](../../../raw/current/scripts/prefabs/snowman.lua#L636 [snowman.lua:674](../../../raw/current/scripts/prefabs/snowman.lua#L674

新生重物需要立即抛出时，原版 LootDropper:SpawnLootPrefab 在 FlingItem 之前先 ForceDropPhysics；这解决质量 0 的障碍初态，不意味着该方法本身会施加抛力。[lootdropper.lua:382](../../../raw/current/scripts/components/lootdropper.lua#L382

## 常见陷阱与限制

- **SetRadius 后实体已经实际变大／变成障碍**：它只改字段和逻辑状态；初始物理与后续尺寸变更参照上文宿主组装，不能把状态查询当 Physics 校验。[heavyobstaclephysics.lua:225](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L225 [snowman.lua:253](../../../raw/current/scripts/prefabs/snowman.lua#L253
- **移除组件会恢复原始 Physics 并取消延迟**：OnRemoveFromEntity 只撤六种事件监听，没有 CancelObstacleTask、质量／mask 回滚或任务清理；不是可靠热拆卸入口。应按具体宿主的结束状态处理，不在任务运行中随意拆组件。[heavyobstaclephysics.lua:205](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L205

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、普通／小重物基础 Physics、土豆袋组装、落石下落回调、雪球尺寸／推行接线、pushable 动力及 lootdropper 抛出入口。未审阅全部落石／雪球机制、完整推行 SG、引擎碰撞求解或网络复制内部。
- 结构检查：validate_page 及源码引用 strict 检查通过；无新增完整 Lua 示例。机械检查不代替游戏运行。
- 游戏运行：未运行；落地避人、不同尺寸、收放／下落／推行交错、读档唤醒和任务运行时拆卸待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：使用搬运中重物作用于另一目标见 [[reference/components/heavyobstacleusetarget]]；掉落实体生成见 [[reference/components/lootdropper]]。
