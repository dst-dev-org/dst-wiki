---
title: "shadowhandspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "夜晚按玩家调度暗影之手（shadowhand）与船上 waveyjones 的世界级生成器，含年龄、理智、虚拟房间门禁与按火焰数量上限。"
aliases: ["shadowhandspawner", "暗影之手生成器", "查理之手", "影子手刷怪", "waveyjones 生成"]
tags: [component, creature, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/shadowhandspawner.lua", "prefabs/cave.lua", "prefabs/forest.lua", "brains/waveyjoneshandbrain.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shadowhandspawner

## 快速判断

- 职责与适用：主模拟世界的夜间暗影之手调度器，构造即断言 `TheWorld.ismastersim`；原版在洞穴与森林的 master_postinit 安装。按玩家挂任务，在火边生成 `shadowhand`，或在合格船只上放置 `waveyjones_marker`。[断言](../../../raw/current/scripts/components/shadowhandspawner.lua#L7) [洞穴安装](../../../raw/current/scripts/prefabs/cave.lua#L521) [森林安装](../../../raw/current/scripts/prefabs/forest.lua#L661)
- 优先使用：想改生成节奏只能改文件顶部常量（构造后无可改公开参数）：间隔 `INTERVAL/VARIANCE`、重试 `RETRY_INTERVAL/RETRY_VARIANCE`、首次门槛 `INITIAL_SPAWN_THRESHOLD`、单火上限 `MAX_HANDS_PER_FIRE`。[常量](../../../raw/current/scripts/components/shadowhandspawner.lua#L13-L18)
- 重要副作用：组件把 `checkwaveyjonestarget` 等方法直接挂到世界实体上，被 waveyjones 手的大脑消费；船只被预留后本生成器不再向该船派 marker。[预留接口](../../../raw/current/scripts/components/shadowhandspawner.lua#L288-L302) [大脑消费](../../../raw/current/scripts/brains/waveyjoneshandbrain.lua#L80)
- 修改边界：任务挂在玩家实体上，玩家离开即取消；组件自身无 OnSave/OnLoad、无 OnRemoveFromEntity，移除组件不会撤销已挂任务或已生成实体。[任务生命周期](../../../raw/current/scripts/components/shadowhandspawner.lua#L209-L226)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `inst.checkwaveyjonestarget(target)` | 挂在世界实体上的方法；目标 GUID 已被预留时返回 true。waveyjones 手大脑据此筛选目标。[实现](../../../raw/current/scripts/components/shadowhandspawner.lua#L288-L292) [消费者](../../../raw/current/scripts/brains/waveyjoneshandbrain.lua#L80) |
| `inst.reservewaveyjonestarget(target)`／`inst.removewaveyjonestarget(target)` | 预留／释放 `_boattargets[GUID]`；大脑选定目标后调用 reserve，防止重复指派。[实现](../../../raw/current/scripts/components/shadowhandspawner.lua#L294-L302) [消费](../../../raw/current/scripts/brains/waveyjoneshandbrain.lua#L103) |
| `inst.spawnwaveyjones(boat)` | 生成 `waveyjones_marker` 放到船的位置，并用其 entitytracker 以 `"boat"` 追踪该船，返回 marker。仅 boats 分支内部调用。[实现](../../../raw/current/scripts/components/shadowhandspawner.lua#L304-L310) |
| `GetDebugString()` | 返回当前追踪的 shadowhand 总数文本，仅调试。[实现](../../../raw/current/scripts/components/shadowhandspawner.lua#L317-L325) |

### 排程与玩家生命周期

- 每玩家参数表 `{ents={}}`：首个玩家加入时开始监听 `isnight` 世界状态，全部离开时停止；`ms_playerjoined`/`ms_playerleft` 驱动注册。夜晚 `Start`、天亮 `Stop`。[玩家注册](../../../raw/current/scripts/components/shadowhandspawner.lua#L245-L267) [昼夜开关](../../../raw/current/scripts/components/shadowhandspawner.lua#L232-L243)
- `Reschedule(player,params,delay?)`：用 `player:DoTaskInTime` 重挂任务，默认延迟 `INTERVAL + VARIANCE*random`；`Start` 仅在无任务时生效；`Stop` 取消任务并把剩余延迟存回 `params.delay`，下次 Start 沿用。[排程实现](../../../raw/current/scripts/components/shadowhandspawner.lua#L209-L226)
- 失败重试走 `Retry`，固定用 5～10 秒短延迟，不是常规间隔。[重试](../../../raw/current/scripts/components/shadowhandspawner.lua#L76-L78)

### SpawnHand 决策链（每次任务触发）

顺序执行，多数分支只是重排或短重试：已有手或玩家年龄不足 4 天 → 重排（[年龄](../../../raw/current/scripts/components/shadowhandspawner.lua#L130)，[常量](../../../raw/current/scripts/components/shadowhandspawner.lua#L18)）；玩家在 vault 的 `puzzle2` 房或非大厅的 Atrium 房 → 重排，这两个虚拟房间各有自己的暗影之手控制器（[虚拟房间](../../../raw/current/scripts/components/shadowhandspawner.lua#L106-L128)）；理智模式百分比 >0.75 → 短重试（[理智](../../../raw/current/scripts/components/shadowhandspawner.lua#L141-L146)）；之后进入船只或地面分支：

- **船只分支**：`GetCurrentPlatform` 且带 `boat` 标签才算船；船体半径内存在修补贴片、mast、anchor 或可特殊熄灭的 fueled 组件则视为合格（[资格测试](../../../raw/current/scripts/components/shadowhandspawner.lua#L80-L101)）；该船已有 marker 就重排，否则 `spawnwaveyjones` 放置 marker 并登记 `_boats` 与玩家 onremove 清理（[船只分支](../../../raw/current/scripts/components/shadowhandspawner.lua#L148-L161)）。
- **地面分支**：60 半径内找 `fire` 标签、非 `_equippable`/`shadow_fire`、且燃料类型标签（`FUELTYPE` 各项 + `_fueled`）命中的火（[找火](../../../raw/current/scripts/components/shadowhandspawner.lua#L164-L169)，[燃料标签构建](../../../raw/current/scripts/components/shadowhandspawner.lua#L274-L278)）；每火最多 `MAX_HANDS_PER_FIRE`(2) 只（[上限](../../../raw/current/scripts/components/shadowhandspawner.lua#L170-L175)）；在火的最大光照半径（缺省 8）内扇形尝试落点，要求实际光照 ≤ `TUNING.DARK_SPAWNCUTOFF`(0.1)、可通行且不近地洞（[落点与生成](../../../raw/current/scripts/components/shadowhandspawner.lua#L177-L199)）。生成后 `SetTargetFire(fire)` 并登记追踪。

任一分支有产出就按常规间隔重排下一轮，否则短重试（[收尾](../../../raw/current/scripts/components/shadowhandspawner.lua#L201-L206)）。

## 典型接入

### 与 waveyjones 目标预留协议协作

- 前提与选择：服务端、世界已装本组件（洞穴或森林 master_postinit 之后）。自定义“派手拆船”类行为若要与原版互斥，应走同一套 check/reserve/remove 协议，而不是自建去重表；这是调用协议指引，不改生成规则本身。
- 生效与撤销：reserve 立即写入 `_boattargets`（仅内存），行为结束或目标移除时必须 remove 释放；本组件只读该表做去重，不负责清理调用方的状态。[协议实现](../../../raw/current/scripts/components/shadowhandspawner.lua#L288-L302)

```lua
-- 服务端 master sim，世界已安装 shadowhandspawner（原版洞穴/森林 master_postinit 之后）
if TheWorld.components.shadowhandspawner ~= nil then
    if not TheWorld:checkwaveyjonestarget(target) then
        TheWorld:reservewaveyjonestarget(target)
        -- ...占用 target 执行自己的行为...
        TheWorld:removewaveyjonestarget(target) -- 结束或目标移除时释放
    end
end
```

- 调整与验证：`target` 为带 GUID 的实体；协议只在内存，重载后需重新预留。对照原版消费见 [大脑筛选](../../../raw/current/scripts/brains/waveyjoneshandbrain.lua#L80) 与 [大脑预留](../../../raw/current/scripts/brains/waveyjoneshandbrain.lua#L103)。

## 常见陷阱与限制

- **把它当通用夜间刷怪器：** 生成的对象固定为 `shadowhand` 与 `waveyjones_marker`，且不处理伤害、目标选择等后续行为（由各实体/大脑自理）；暗影触手等由 [[reference/components/shadowcreaturespawner]] 负责，不可混用。[生成对象](../../../raw/current/scripts/components/shadowhandspawner.lua#L189) [marker](../../../raw/current/scripts/components/shadowhandspawner.lua#L305)
- **以为改 TUNING 能调频：** 所有节奏常量在构造时从 `TUNING.SEG_TIME`、`TUNING.TOTAL_DAY_TIME` 等取值捕获为局部量，运行中改 TUNING 不回灌；构造后也没有 setter。[常量捕获](../../../raw/current/scripts/components/shadowhandspawner.lua#L13-L18)
- **地面分支找不到火就完全静默：** 不查看全图火源，只按玩家 60 半径 + 燃料类型标签筛；影火（`shadow_fire`）被排除，营火添燃料类型决定标签命中。无法复现时先核对这些条件再怀疑调度。[找火条件](../../../raw/current/scripts/components/shadowhandspawner.lua#L103-L104) [燃料标签](../../../raw/current/scripts/components/shadowhandspawner.lua#L274-L278)
- **任务与追踪均不持久化：** 组件无 OnSave/OnLoad；`params.ents`、`_fires`、`_boats`、`_boattargets` 都是内存态，世界重载后重新排队，不会跨存档延续手数或预留。[排程](../../../raw/current/scripts/components/shadowhandspawner.lua#L209-L226) [初始化](../../../raw/current/scripts/components/shadowhandspawner.lua#L273-L286)
- **世界实体方法依赖组件存在：** `checkwaveyjonestarget` 等仅在组件构造时挂上；未装组件（如自定义世界）直接调用会报 nil 调用，自定义世界需自行加装或判空。[挂载](../../../raw/current/scripts/components/shadowhandspawner.lua#L288-L310)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 shadowhandspawner.lua，核验常量、玩家注册与昼夜排程、SpawnHand 决策链（年龄/虚拟房间/理智/船/火）、waveyjones 协议挂载与大脑消费、洞穴与森林安装点。未运行游戏；shadowhand、waveyjones 实体本体行为不在本页范围。
- 结构检查：validate_page.py 通过；Lua 示例经 parser_probe 语法检查。

相关：[[reference/components/shadowcreaturespawner]]、[[reference/components/brightmarespawner]]、[[reference/components/groundshadowhandler]]。
