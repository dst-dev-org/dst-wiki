---
title: "physicsmodifiedexternally"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "按来源实体汇总水平外部 motor 速度，协调船锚阻力与玩家速度同步，并在最后来源撤销时清零和移除组件。"
aliases: ["physicsmodifiedexternally", "多来源外部速度", "漩涡牵引速度", "外部物理速度撤销"]
tags: [component, lifecycle, event, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/physicsmodifiedexternally.lua", "components/oceanwhirlportalphysics.lua", "prefabs/oceanwhirlbigportal.lua", "components/boatphysics.lua", "components/locomotor.lua", "prefabs/player_classified.lua", "entityscript.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# physicsmodifiedexternally

## 快速判断

- 职责与适用：装在受外部速度影响的目标实体上，以来源实体为键保存 x／z 速度分量，求和后写 `Physics:SetMotorVelExternal`；不是通用物理属性快照或质量／碰撞恢复器。原版海洋漩涡服务端按受影响对象动态添加。[physicsmodifiedexternally.lua:1-20](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L1-L20) [physicsmodifiedexternally.lua:38-73](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L38-L73) [oceanwhirlportalphysics.lua:25-30](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L25-L30) [oceanwhirlportalphysics.lua:121-130](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L121-L130)
- 优先使用：多个持续来源共同影响水平外部 motor 速度时，先 AddSource，再 SetVelocityForSource；来源退出时 RemoveSource。普通行走倍率或船桨冲击沿 [[reference/components/locomotor]]、[[reference/components/boatphysics]] 的对应接口，不混作同一种量。[physicsmodifiedexternally.lua:60-82](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L60-L82)
- 重要副作用：重算时覆盖目标的外部 motor 速度，存在 locomotor 时同步写其 externalvelocityvectorx/z；最后来源消失会自动移除本组件并清零外部速度，不恢复添加前的旧向量。[physicsmodifiedexternally.lua:9-14](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L9-L14) [physicsmodifiedexternally.lua:22-58](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L22-L58)
- 修改边界：实现注明服务端通过本组件统一管理 SetMotorVelExternal；玩家客户端由 classified 脏事件应用向量。来源表无定时更新、衰减或存读，效果生产者必须负责持续写入与退出。[physicsmodifiedexternally.lua:1-2](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L1-L2) [player_classified.lua:667-671](../../../raw/current/scripts/prefabs/player_classified.lua#L667-L671)

## 关键接口

| 接口／字段 | 签名或约束 | 条件、作用与证据 |
|---|---|---|
| `AddSource(src)` | 来源实体；可为目标自身 | 幂等建立 `{x=0,z=0}`；非自身来源监听 onremove。仅注册不触发重算，来源必须支持事件协议。[physicsmodifiedexternally.lua:66-73](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L66-L73) |
| `SetVelocityForSource(src,velx,velz)` | 已登记来源、水平速度分量 | 直接访问 sources[src] 写值并重算；未先登记会访问nil，不自动 AddSource。分量按地图x／z轴汇总，不是局部前向速度、力或每次调用位移。[physicsmodifiedexternally.lua:38-64](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L38-L64) |
| `RecalculateExternalVelocity()` | 无参数；无显式返回 | 所有来源分量相加，经可选船锚修正后写外部 motor 与 locomotor 字段；不会重新读取生产者的位置或计算牵引方向。[physicsmodifiedexternally.lua:38-58](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L38-L58) |
| `RemoveSource(src)` | 来源键 | 无记录时无动作；有记录则撤非自身来源监听并删记录。无剩余来源时移除组件；仍有来源时不立即重算，须区分撤记录与撤旧合速度。[physicsmodifiedexternally.lua:9-14](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L9-L14) [physicsmodifiedexternally.lua:75-82](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L75-L82) |
| `sources`、`totalvelocityx/z` | 来源→向量表；最近重算的合速度 | 初始空表与0；不代表外部Physics当前值会一直与之相同，绕过组件直接写Physics会破坏统一归属。[physicsmodifiedexternally.lua:1-14](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L1-L14) [physicsmodifiedexternally.lua:52-57](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L52-L57) |
| `OnRemoveFromEntity()` | 删除组件钩子 | 外部 motor 置(0,0,0)，locomotor相应字段置0，撤剩余来源监听，再发 `losephysicsmodifiedexternally`；不调用普通 Physics:Stop 或还原全部物理属性。[physicsmodifiedexternally.lua:22-36](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L22-L36) |

## 关键流程

### 速度合成与船锚修正

每次 setter／显式重算从0开始累加各来源x、z，不乘dt、不保存惯性，也不自动限幅。传入的是外部速度分量，时间与距离公式由生产者负责。原版漩涡在地图坐标里计算径向、切向向量后传 vx/vz，本组件不理解漩涡中心或半径。[physicsmodifiedexternally.lua:38-64](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L38-L64) [oceanwhirlportalphysics.lua:222-245](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L222-L245)

若目标有 boatphysics，令合速度长度加0.001作除数得到方向，再以 `-0.5*clamp(总锚阻力/MAX_ALLOWED_VELOCITY,0,1)` 沿该方向修正x／z。默认分母常量为10；它不是把合速度限制到10，也不直接使用所有船阻力／帆力参数。GetTotalAnchorDrag 只加已登记 boatdrag 的 drag；船增删 boatdrag 时会显式触发本组件重算。[physicsmodifiedexternally.lua:43-57](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L43-L57) [boatphysics.lua:386-392](../../../raw/current/scripts/components/boatphysics.lua#L386-L392) [boatphysics.lua:222-242](../../../raw/current/scripts/components/boatphysics.lua#L222-L242) [tuning.lua:4150](../../../raw/current/scripts/tuning.lua#L4150

### 玩家同步与组件事件的时机

重算同步写 locomotor.externalvelocityvectorx/z；服务端字段 setter 会更新 player_classified 的同名 net_float。客户端收到 `externalvelocityvectordirty` 后，使用两个网络分量直接 SetMotorVelExternal；组件本身不复制 sources 表给客户端，也不要求每个来源都在客户端重算。[physicsmodifiedexternally.lua:52-57](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L52-L57) [locomotor.lua:79-89](../../../raw/current/scripts/components/locomotor.lua#L79-L89) [locomotor.lua:347-348](../../../raw/current/scripts/components/locomotor.lua#L347-L348) [player_classified.lua:667-671](../../../raw/current/scripts/prefabs/player_classified.lua#L667-L671) [player_classified.lua:1224](../../../raw/current/scripts/prefabs/player_classified.lua#L1224 [player_classified.lua:1619-1620](../../../raw/current/scripts/prefabs/player_classified.lua#L1619-L1620)

构造会立即向目标发 `gainphysicsmodifiedexternally`。EntityScript 在构造函数返回后才把实例写入 components[name]，因此该事件回调内检查 `inst.components.physicsmodifiedexternally` 可能仍为空；不要据此判定事件无效。删除组件则先从 components 表移除，再调用清理钩子，清零后发 lose 事件。[physicsmodifiedexternally.lua:16-36](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L16-L36) [entityscript.lua:653-667](../../../raw/current/scripts/entityscript.lua#L653-L667) [entityscript.lua:670-685](../../../raw/current/scripts/entityscript.lua#L670-L685)

### 来源撤销与临时状态

非自身来源 onremove 自动走 `_onremovesource`；没有 death、休眠或超时自动撤销。自身来源跳过 onremove 监听，需由生产者显式撤销或结束目标生命周期。组件没有 OnUpdate、OnSave／OnLoad，不会自动衰减数值、保存来源或重建效果。[physicsmodifiedexternally.lua:3-20](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L3-L20) [physicsmodifiedexternally.lua:66-82](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L66-L82)

删掉来源记录后若还有其他来源，当前实现不调用 RecalculateExternalVelocity，旧合速度和 totalvelocity 字段会保留到下一次 setter／显式重算；最后一个来源删掉时才通过删除组件立即清零。需要即时撤销单个来源的调用方应在 RemoveSource 后重新取得仍存在的组件并重算，不能在最后来源自动移组件后盲调旧引用。[physicsmodifiedexternally.lua:9-14](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L9-L14) [physicsmodifiedexternally.lua:22-36](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L22-L36) [physicsmodifiedexternally.lua:75-82](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L75-L82)

## 典型接入

### 海洋漩涡：目标持有组件，漩涡作为来源

原版 oceanwhirlbigportal 在服务端安装 oceanwhirlportalphysics。该生产者记住受影响实体时，给目标补本组件并 `AddSource(漩涡实体)`；忘记实体时 `RemoveSource(同一漩涡实体)`。目标有 boatphysics 时按船分支修正，有 locomotor 时另接玩家同步；来源与目标不能颠倒。[oceanwhirlbigportal.lua:312-343](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L312-L343) [oceanwhirlportalphysics.lua:121-141](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L121-L141)

生产者更新时检查周边对象、忘记已离开对象，再按径向／切向公式逐目标调用 SetVelocityForSource；默认有对象时采用0.1秒检查节奏，传入牵引公式的tickperiod固定0.1，而不是本组件读取dt。生产者未休眠时关闭、或启用期间进入休眠，会忘记正在追踪的目标；醒来按启用状态恢复更新。其筛选、伤害、传送与完整漩涡机制不属于本组件。[oceanwhirlportalphysics.lua:10-12](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L10-L12) [oceanwhirlportalphysics.lua:47-62](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L47-L62) [oceanwhirlportalphysics.lua:102-115](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L102-L115) [oceanwhirlportalphysics.lua:215-267](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L215-L267)

这是服务端内部来源管理参照。本页不提供可直接套用到任意实体的demo；目标需有兼容Physics，生产者要负责选取目标、计算速度、更新及配对撤销，并接受多来源移除后重算的上述边界。

## 常见陷阱与限制

- **同来源可用不同key叠多个向量：** 接口只有来源实体键，每来源一个x／z槽；重复 AddSource 不清旧值，SetVelocityForSource 直接覆盖该来源值。[physicsmodifiedexternally.lua:60-73](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L60-L73)
- **外部组件删除会恢复原来的物理：** 清理只清外部motor和locomotor向量，不保存旧外部速度，也不撤销普通motor、碰撞、质量或生产者其他副作用。[physicsmodifiedexternally.lua:22-36](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L22-L36)
- **直接 SetMotorVelExternal 与组件可各管一部分：** 每次重算都是覆盖调用，混用会相互覆盖；组件头注释明确要求服务端由此处统一维护来源向量。[physicsmodifiedexternally.lua:1-2](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L1-L2) [physicsmodifiedexternally.lua:38-58](../../../raw/current/scripts/components/physicsmodifiedexternally.lua#L38-L58)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件全文、海洋漩涡服务端安装及生产者记忆／遗忘／速度更新／休眠启停、boatphysics锚阻力求和与注册重算、locomotor字段同步、classified消费、EntityScript添加／删除顺序及分母配置，覆盖来源归属、合成、撤销时序与玩家同步。
- 游戏运行：未运行；未测多人预测、重叠漩涡来源退出、弱速度叠加船锚修正、目标重载或各类Physics兼容性。未展开完整漩涡行为或船体物理积分。
- 结构检查：`validate_page.py` 通过；两页合并执行 `validate_source_links.py --strict`，99个行号引用、0错误／0警告。本页无新增 Lua 示例；机械检查不代替游戏测试。

相关：[[reference/components/locomotor]] · [[reference/components/boatphysics]] · [[reference/components/boatdrag]] · [[reference/components/pushable]]
