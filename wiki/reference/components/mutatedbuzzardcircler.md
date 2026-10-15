---
title: "mutatedbuzzardcircler"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月雹变异秃鹫的影子盘旋组件：绕目标玩家循环飞行，找尸体降落投下真身，信号弹命中可击落，睡眠/失效回收入迁徒群。"
aliases: ["mutatedbuzzardcircler", "变异秃鹫盘旋", "月雹秃鹫影子", "circlingbuzzard_lunar"]
tags: [component, creature, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/mutatedbuzzardcircler.lua", "prefabs/circlingbuzzard.lua", "components/mutatedbuzzardmanager.lua", "prefabs/corpses.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# mutatedbuzzardcircler

## 快速判断

- 职责与适用：装在 FX 实体 `circlingbuzzard_lunar`（影子鸟，`persists=false`）上的盘旋组件；真秃鹫是 `inst.buzzard` 引用的另一实体，本组件只管影子移动与“投放/降落”时机。普通秃鹫影子用的是 [[reference/components/circler]]，两者不可互换。[prefab 分支](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L169-L181) [不持久化](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L135)
- 优先使用：指派目标用 `SetCircleTarget(target)`（由世界 mutatedbuzzardmanager 调用）；查询投放模式经 `mutatedbuzzardmanager:GetDropBuzzards()`。移动参数（min/max_speed、dist、scale）是构造公开字段，可在 Start 前改。[设目标](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L117-L140) [管理器指派](../../../raw/current/scripts/components/mutatedbuzzardmanager.lua#L118-L120)
- 重要副作用：组件直接迁移秃鹫实体（从 migrationmanager 人口组移除、传送、切状态）；降落成功或被击落时 `KillShadow()` 移除影子。降落/击落还会在目标尸体上绑定秃鹫行为。[降落](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L200-L227) [击落](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L28-L54)
- 修改边界：无 OnSave/OnLoad；组件停止的入口是 `Stop()`，影子消失靠 `inst:KillShadow()`（circlingbuzzard 实体方法，仅主模拟定义）。目标死亡/移除自动回收影子。[KillShadow](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L87-L99) [目标失效](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L131-L139)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `SetCircleTarget(tar)` | 维护目标 `_num_circling_buzzards` 计数；监听目标 onremove/death 回收影子。tar 为 nil 即清目标。[实现](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L117-L140) |
| `Start()` | 校验目标有效后随机化速度（5～7×0.01）、距离（8～12）、方向，把影子放到目标周围并启动更新；无目标直接清目标退出。[实现](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L92-L111) |
| `Stop()` | 仅停止更新组件；不清理目标引用。[实现](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L113-L115) |
| `UpdateMigrationNode()` | 目标所在迁移节点须与上次记录节点相邻（距离 ≤1）才更新记录；否则 `StoreInMigrationNode()` → 影子淡出回收。[实现](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L167-L180) [节点查询](../../../raw/current/scripts/components/migrationmanager.lua#L282-L286) |
| `FindCorpse(target)` | 目标 25 半径内随机取一个合格尸体：`creaturecorpse` 标签、非 NOCLICK、非秃鹫应忽略（[全局判定](../../../raw/current/scripts/brains/buzzardbrain.lua#L206)）、不会变异（[WillMutate](../../../raw/current/scripts/prefabs/corpses.lua#L388)）、无灵体将至（[HasGestaltArriving](../../../raw/current/scripts/prefabs/corpses.lua#L25)）。前置要求世界有 corpsepersistmanager 且存在任意尸体。[实现](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L182-L198) |
| `LandOnCorpse(corpse)` | 尸体旁找可行走非海洋落点，把真秃鹫传送到 y=30 进 `glide` 状态、延迟一拍 `SetOwnCorpse(corpse)`，随后 KillShadow+Stop。成功返回 true。[实现](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L200-L227) |
| `DropBuzzard()` | 在影子与目标中点生成 `buzzardcorpse`（继承秃鹫 build，10～15 秒淡出），秃鹫移出人口组后 Remove。[实现](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L229-L255) |
| `miniflare_detonated`／`megaflare_detonated`（世界事件） | 信号弹爆炸命中判定：影子距爆点 ≤25（`TUNING.BUZZARDSPAWNER_FLARE_HIT_DIST_SQ`）且未超配额（小型 1 只、大型 5 只）时延迟 0.2～1 秒投放秃鹫进 `fall` 状态并 KillShadow。[监听](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L28-L57) [调参](../../../raw/current/scripts/tuning.lua#L9075) |

### 盘旋更新与回收

`OnUpdate` 每帧执行：每 0.5～2.5 秒一次“决策冷却”——刷新迁移节点、检查投放模式（`GetDropBuzzards()` 为真时保持盘旋不降落）、尝试找尸体降落；其余时间做绕圈数学（正弦调速 5～7、随距离加速 0～10、缩放 0.8～1.2 反比于速度）（[更新](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L259-L323)）。`OnEntitySleep` 与迁移节点失效都会走 `StoreInMigrationNode` → `KillShadow` 回收（[睡眠回收](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L331-L342)）。

## 典型接入

### 由世界管理器批量派鸟

- 前提与选择：主模拟端使用。原版唯一生产者是 mutatedbuzzardmanager（月雹事件）：生成影子、SetCircleTarget(玩家)、由其驱动 Start。自定义玩法应复用同一链路并维持人口组登记，不要直接 SpawnPrefab 影子后自行摆放（会绕过 migrationmanager 的数量核算）。[管理器调用](../../../raw/current/scripts/components/mutatedbuzzardmanager.lua#L118-L120)
- 生效与撤销：目标无效（死亡/移除/跨迁移节点过远）自动回收；主动撤销用 `KillShadow()` 而不是 Remove 组件——组件不清理影子显示状态。[回收路径](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L131-L139)

```lua
-- 服务端示意：指派一个已存在的月雹影鸟跟随目标盘旋
local circler = shadow_ent.components.mutatedbuzzardcircler
if circler ~= nil then
    circler:SetCircleTarget(player)
    circler:Start()
end
```

- 调整与验证：速度/距离/缩放区间在 Start 时读取，Start 后修改不影响本轮；验证观察影子是否绕目标稳定绕圈、爆点 25 内信号弹是否击落（[Start 读参](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L98-L103)）。

## 常见陷阱与限制

- **影子不是秃鹫：** 影子是背景层 FX（`LAYER_BACKGROUND`、标签 FX），无碰撞伤害；攻击判定全在真秃鹫实体上。把它当可击中目标会失败。[影子实体](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L108-L116)
- **降落依赖尸体系统：** `FindCorpse` 前置检查 `corpsepersistmanager:AnyCorpseExists()`，且尸体必须通过三重合格判定；无合格尸体时只持续盘旋，不会凭空投放（除信号弹/管理器投放路径）。[前置](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L183-L185)
- **迁移节点漂移即回收：** 目标（玩家）跨迁移节点过远（距离 >1）时影子直接消失，这是防跟随作弊的设计；用影子做长距离标记不可靠。[失效分支](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L173-L179)
- **投放模式优先于降落：** `GetDropBuzzards()` 为真时决策冷却直接 return，跳过找尸体；该模式由 mutatedbuzzardmanager 控制，不是本组件字段。[优先级](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L272-L274)
- **信号弹配额按事件共享：** `data.hit_num_buzzards` 挂在事件数据上，全场共享配额（小 1/大 5）；多个影子竞争同一批配额。[配额](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L29-L53)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 mutatedbuzzardcircler.lua 与 circlingbuzzard.lua，核验盘旋数学、目标协议、尸体判定链、信号弹路径与 mutatedbuzzardmanager 生产点；核对 corpses.lua 的实体方法挂载与 buzzardbrain 全局判定。未运行游戏；真秃鹫（mutatedbuzzard）实体行为与月雹事件全流程不在本页范围。

相关：[[reference/components/circler]]。
