---
title: "电击命中火花（sparks 族）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "独立电击命中火花工厂的三个注册：随机三选一动画＋橙色闪光＋灯光衰减，AlignToTarget 贴目标朝攻击者一侧；内置目标闪白机制但被代理无 AnimState 的门禁短路，电击命中集中入口按免疫与否选名。"
aliases: ["sparks", "electrichitsparks", "electrichitsparks_electricimmune", "电击火花", "命中火花", "电击命中特效", "电击火花特效", "目标闪白"]
tags: [prefab, fx, network-sync]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/sparks.lua", "componentutil.lua", "prefabs/foodbuffs.lua", "prefabs/nightstick.lua", "prefabs/blowdart.lua", "prefabs/slingshotammo.lua", "wx78_moduledefs.lua", "prefabs/spear_wathgrithr.lua", "prefabs/wx78_drone_zap.lua", "prefabs/storage_robot.lua", "prefabs/wx78.lua", "prefabs/winona_battery_sparks.lua", "prefabs/halloweenpotion_embers.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 电击命中火花（sparks 族）

## 快速判断

- 职责与适用：实体被电／命中时的火花爆点＋橙色闪光，共 3 注册；**独立工厂**，不走 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]] 的表协议——代理带 net_tinybyte 随机帧、`AlignToTarget` 定位与目标闪白是本工厂自有逻辑。[工厂](../../../raw/current/scripts/prefabs/sparks.lua#L128-L175)
- 优先使用：查"电击命中火花／目标闪白"选本页；持续带电表现选 [[reference/prefabs/electricchargedfx|electricchargedfx]]、玩家电击态与连锁选 [[reference/prefabs/electrocute_fx|electrocute_fx]]、附着动画选 [[reference/prefabs/shock_fx|shock_fx 族]]。
- 重要副作用：带目标闪白机制，但**现有实现下不生效**（`OnUpdateFlash` 检查的是代理的 AnimState，代理并未创建该组件——见陷阱）；本地实体无条件带橙色 Light（专服也有，无 SoundEmitter）。[灯光](../../../raw/current/scripts/prefabs/sparks.lua#L55-L59)
- 修改边界：代理 1 秒自删；本地表现由"灯光衰减＋动画结束"双条件回收；动画是否重复未核验。[回收](../../../raw/current/scripts/prefabs/sparks.lua#L1-L27)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `sparks` | `MakeSparks(name, build, sound)` → 代理（Transform＋Network＋2 倍缩放＋`_rand` net_tinybyte）→ randdirty 一帧后 `StartFX` 本地实体 | build=`sparks`，音效用默认 `dontstarve/characters/wx78/spark`。[注册](../../../raw/current/scripts/prefabs/sparks.lua#L177-L179)、[默认音效](../../../raw/current/scripts/prefabs/sparks.lua#L2-L6) |
| `electrichitsparks` | 同上协议 | build=`elec_hit_fx`，默认音效。[注册](../../../raw/current/scripts/prefabs/sparks.lua#L178-L178) |
| `electrichitsparks_electricimmune` | 同上协议 | build=`elec_immune_fx`，音效 `electrocute_immune`（电免目标的专用表现）。[注册](../../../raw/current/scripts/prefabs/sparks.lua#L179-L179) |

本地实体：bank／build 同 build 参数、`PlayAnimation("sparks_"..随机 1～3)`、泛光、Light（半径 2／强度 0.9 起、橙色）。三注册在简中 .po 均无 `STRINGS.NAMES` 键，标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 代理 `_rand:set(math.random(3))` 推 randdirty；各模拟 `OnRandDirty` 一次性门禁（`_complete`）后延迟一帧建本地表现（专服也建，但无 SoundEmitter）。本地表现挂到**代理的父级**（随宿主移动），再 `SetFromProxy` 复制代理变换——代理自身后续位移不被跟随 | 随机帧由代理 netvar 带到客户端；重复 dirty 不再触发；父挂语义同通用工厂（区分宿主移动与代理位移） | [随机与门禁](../../../raw/current/scripts/prefabs/sparks.lua#L134-L156)、[父挂与取位](../../../raw/current/scripts/prefabs/sparks.lua#L44-L48) |
| 生成者：电击命中入口 | `SpawnElectricHitSparks(inst, target, flash)`：双端有效性检查 → `IsEntityElectricImmune(target)` 选 immune 变体 → 生成并 `AlignToTarget(target, inst, flash)` | 服务端；调用者：电击食物 buff 监听 `onattackother` 攻击附加火花 [foodbuffs](../../../raw/current/scripts/prefabs/foodbuffs.lua#L85-L85)、晨星锤（官方名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L345550-L345552)）攻击与充能 [nightstick](../../../raw/current/scripts/prefabs/nightstick.lua#L104-L119)、电吹箭 [blowdart](../../../raw/current/scripts/prefabs/blowdart.lua#L263-L263)、弹弓电击路径 [slingshotammo](../../../raw/current/scripts/prefabs/slingshotammo.lua#L764-L764)、WX78 电击模块受击／格挡反击 [wx78_moduledefs](../../../raw/current/scripts/wx78_moduledefs.lua#L650-L650)、奔雷矛／充能奔雷矛（官方名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L349140-L349147)）攻击 [spear_wathgrithr](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L149-L153)、无人机电击 [wx78_drone_zap](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L320-L320)——`Lightning_OnAttack` 由奔雷矛两工厂经 `SetOnAttack` 挂载 [挂载](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L474-L474) | [入口](../../../raw/current/scripts/componentutil.lua#L1022-L1030) |
| 生成者：直接落点 | storage_robot 在自身位置直接 `SpawnPrefab("sparks")`；WX78 的湿度跟踪更新在非强制干燥分支生成于自身上方（y 加 1～2.5），均不经 AlignToTarget | 服务端 | [机器人](../../../raw/current/scripts/prefabs/storage_robot.lua#L347-L347)、[WX78](../../../raw/current/scripts/prefabs/wx78.lua#L268-L283) |
| AlignToTarget | 把**代理**定位到目标朝攻击者一侧边缘（半径＋0.2，y＋1；attacker 可以是投射物）；`flash=true` 时给代理挂 updatelooper 并注册 `OnRemoveFlash`——但 `OnUpdateFlash` 首次执行即因**代理没有 AnimState** 走早退分支：拆掉 updater 并清空清理回调，目标不会被染白（见陷阱） | 主模拟调用；闪白染白路径当前不可达 | [定位](../../../raw/current/scripts/prefabs/sparks.lua#L106-L126)、[门禁](../../../raw/current/scripts/prefabs/sparks.lua#L80-L84) |
| 闪白机制（不可达） | `OnUpdateFlash` 原设计对**目标**闪白：colouradder Push／Pop，无则直接 SetAddColour；强度 largecreature 0.1、其余 0.2；结束或中途失效时按 freezable／无组件还原 | 检查的是**代理**的 `inst.AnimState`（L81），普通代理只建 Transform＋Network（L147-L148），染白与 `OnRemoveFlash` 还原在当前实现下均不可达 | [闪白体](../../../raw/current/scripts/prefabs/sparks.lua#L80-L104)、[代理构造](../../../raw/current/scripts/prefabs/sparks.lua#L144-L148) |
| 回收 | 双条件：20Hz 任务把灯光从 0.9 按 dt×2 衰减（约 0.45 秒），首次 tick 顺带播音效；animover 时若灯光未衰减完则 `killfx` 标记等衰减完 Remove，灯光先衰减完则取消任务等 animover；代理另有 1 秒自删 | 各模拟本地表现独立回收；任务在本地实体上 | [onupdate](../../../raw/current/scripts/prefabs/sparks.lua#L1-L18)、[animover](../../../raw/current/scripts/prefabs/sparks.lua#L20-L27) |
| 清理：闪白还原（不可达） | `OnRemoveFlash` 原设计在代理移除时还原目标：有 colouradder 时交给其色栈，freezable 走 `UpdateTint()`，否则 SetAddColour 清零 | 早退路径把 `OnRemoveEntity` 清空，还原回调当前同样不可达 | [OnRemoveFlash](../../../raw/current/scripts/prefabs/sparks.lua#L70-L78) |
| 资源 | 三注册各自独立 ANIM build（sparks／elec_hit_fx／elec_immune_fx）；同库 bank 名 `sparks` 另被独立实体 winona_battery_sparks 使用 [winona_battery_sparks](../../../raw/current/scripts/prefabs/winona_battery_sparks.lua#L17-L18)，万圣节药水口味 "sparks" 亦为同名不同物 [药水注册](../../../raw/current/scripts/prefabs/halloweenpotion_embers.lua#L212-L212) | 动画是否重复未核验 | [注册](../../../raw/current/scripts/prefabs/sparks.lua#L177-L179) |

## 典型接入

### 借用电击命中火花

- 前提与选择：电击武器／弹道想复用原版表现，直接调 `SpawnElectricHitSparks(attacker, target, true)`（集中入口带免疫分流与有效性检查）；只要火花不要闪白传 `flash=nil`。
- 生效与撤销：火花自动回收；闪白机制当前不可达（见陷阱），宿主侧无需清理。
- 原版依据：晨星锤攻击路径 [nightstick](../../../raw/current/scripts/prefabs/nightstick.lua#L104-L104)。

```lua
-- 服务端命中处理内
SpawnElectricHitSparks(attacker, target, true)   -- 火花表现；flash=true 当前不产生闪白（工厂门禁）
```

- 调整与验证：当前实现**没有暴露固定动画选择的接口**——随机值在代理构造时写入 `_rand`，randdirty 门禁只消费一次，事后改 `_rand` 不会重建表现；动画固定需求只能另建工厂。[随机与门禁](../../../raw/current/scripts/prefabs/sparks.lua#L134-L156)

## 常见陷阱与限制

- **误区：本族走通用表协议**：随机帧、一次性门禁、`AlignToTarget` 与双条件回收都是 sparks.lua 自有实现；通用工厂的 animover 自删、父挂规则不能套用。[工厂](../../../raw/current/scripts/prefabs/sparks.lua#L128-L175)
- **边界：immune 变体由目标免疫状态决定**：`IsEntityElectricImmune(target)` 为真才用电免变体（不同 build 与专用音效）；"打在电免目标上"与"普通命中"是两个注册。[分流](../../../raw/current/scripts/componentutil.lua#L1028-L1029)
- **边界：音效约一帧后才播**：音效在本地实体首次 20Hz tick 播出，且专服根本没有 SoundEmitter；"生成即有声"不成立。[音效](../../../raw/current/scripts/prefabs/sparks.lua#L2-L6)
- **边界：目标闪白机制在当前实现下不生效**：`OnUpdateFlash` 首行检查的是**代理**的 `inst.AnimState`（[门禁](../../../raw/current/scripts/prefabs/sparks.lua#L80-L84)），而普通代理只创建 Transform＋Network（[代理构造](../../../raw/current/scripts/prefabs/sparks.lua#L144-L148)）——`AlignToTarget` 挂上 updatelooper 并直接调用一次后立即早退：`RemoveComponent("updatelooper")` 且 `OnRemoveEntity` 被清空，染白与 `OnRemoveFlash` 还原都不可达。`flash=true` 只影响这条死代码路径的进入，不产生可见效果。
- **边界：动画是否重复未核验**：本地实体只调一次 `PlayAnimation` 并监听 animover，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/sparks.lua#L52-L52)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。工厂全文（代理／随机门禁／本地实体／灯光与音效／AlignToTarget／双条件回收）、集中入口七个调用文件、storage_robot 直落点、同名资源辨析逐条核验；2026-11-02 补核 26335394 的 WX78 湿度跟踪直接生成点，全 scripts 检索集中入口共 8 处调用。闪白与还原路径被代理无 AnimState 门禁短路。.po 名称键（三注册）定向核验无结果，消费者官方名经 NAMES 键核验。未覆盖：`sparks_*`／`elec_*` 动画资源内容、晚加入时序。
- 游戏运行：未运行；火花视觉、灯光衰减与闪白还原未实测。

返回 [[reference/prefabs/fx-status-index|增益、受击、护盾与状态反馈]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
