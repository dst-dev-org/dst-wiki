---
title: "海草花粉喷云与海面诱饵云（waterplant_pollen_fx 等 2 注册）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "海草（Sea Weed）喷粉玩法的 2 个表现注册：空中花粉云（35 帧后在其位置放出海面诱饵云并触发宿主放鱼）与海面诱饵云（挂 chum 标签吸引鱼群、每 1 秒维护最多 5 个 chumpiece，25 秒后 2 秒淡出自删）；含与鱼食袋 chum_aoe 的同源参数辨析。"
aliases: ["waterplant_pollen_fx", "waterplant_pollen_fx_ocean", "海草花粉", "花粉喷云", "落海烟", "诱饵云", "鱼饵云", "chum 云", "海草喷粉"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/waterplant_pollen.lua", "prefabs/waterplant.lua", "prefabs/chum_aoe.lua", "prefabs/chum.lua", "brains/waterplantbrain.lua", "brains/oceanfishbrain.lua", "stategraphs/SGwaterplant.lua", "entityscript.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 海草花粉喷云与海面诱饵云（waterplant_pollen_fx 等 2 注册）

## 快速判断

- 职责与适用：2 个注册是海草（官方名"海草"，见下）喷粉链的两段表现——①`waterplant_pollen_fx`：喷粉时升空的空气花粉云，35 帧后在自身位置生成②号并触发宿主放鱼，动画播完自删；②`waterplant_pollen_fx_ocean`：贴在海面的诱饵云，持 `chum` 标签吸引鱼群、每 1 秒在周围海面维护最多 5 个可食用 `chumpiece`，25 秒后摘标签、2 秒淡出自删。②号是有真实玩法作用的混合载体：既挂 `FX` 标签又承担引鱼与投喂。[两注册](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L195-L196)
- 优先使用：查"海草喷的那团粉是什么/什么时候放鱼"用①号链；查"海面上那滩引鱼的云雾怎么来的、鱼为什么围过去"用②号链；查"鱼食袋扔出来的那滩"是另一注册 `chum_aoe`，归 [[reference/prefabs/eye_charge|眼睛炮塔弹体与诱鱼区页]]。
- 重要副作用：②号不是纯视觉——`chum` 标签被海鱼脑用于索饵（FindClosest `chum`），`chumpiece` 挂 `edible`（次食类型 MEAT）可被进食；①号自身无玩法。②号默认持久（`persists` 保持 true 直到 disperse 计时），会存档。[鱼脑索饵](../../../raw/current/scripts/brains/oceanfishbrain.lua#L219-L220)、[disperse 置 persists=false](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L120-L132)
- 修改边界：只覆盖 2 注册的工厂、生成点、端分界、淡出与清理；海草本体（生长/战斗/ barnacle/口水鱼 childspawner）与 `chumpiece` 的进食消费端不展开。
- 名称：宿主 `WATERPLANT`＝"海草"（[po](../../../raw/current/scripts/languages/chinese_s.po#L351834-L351837)，`Sea Weed`；与"海带叶"（KELP Fronds）不是同一注册）；被引鱼 `OCEANFISH_SMALL_9`＝"口水鱼"（[po](../../../raw/current/scripts/languages/chinese_s.po#L346549-L346552)）。`WATERPLANT_POLLEN_FX`／`WATERPLANT_POLLEN_FX_OCEAN`／`CHUMPIECE` 在简中 `chinese_s.po` 均无 `STRINGS.NAMES` 显示名键，标题与表内中文为已核验效果描述，非官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `waterplant_pollen_fx` | `front_fn`：Transform／AnimState／Network（无 SoundEmitter）→ bank=build=`pollen_cloud`、播一次 `pollen` → 标签 `FX`／`NOCLICK`／`notarget`／`pollen` → `SetPristine` → 客户端早退 → 服务端：35 帧任务生成②号＋检查放鱼、`animover` 自删、`persists=false`。 | 空中一跳的烟云；`_source_flower` 由生成方赋值（不存档，`persists=false`）。[工厂](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L41-L71)、[注册](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L195) |
| `waterplant_pollen_fx_ocean` | `ocean_fn`：Transform／AnimState／SoundEmitter／Network → bank=build=`pollen_chum`、`fish_chum_base_pre`→循环 `fish_chum_base_idle`、OnGround 朝向＋`LAYER_BELOW_GROUND`＋SortOrder 3＋FinalOffset 3 → 标签 `chum`／`FX`／`NOCLICK`／`pollen`、循环音 `spore_cloud_LP` → 净变量 `_fading`（`fadingdirty`）→ `SetPristine` → 非专服监听淡出 → 服务端：`_chumpieces` 计数与 1 秒周期补块任务、timer `disperse`（25 秒）、`onremove` 清块。 | 贴地诱饵云；渲染分层与海树根影同用层值 2（常量名不同，见"对照与边界"）。[工厂](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L142-L193)、[注册](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L196)、[层常量](../../../raw/current/scripts/constants.lua#L56-L60) |

## 关联机制导航

### 喷粉链：海草怎么放出①号云

| 环节 | 触发与条件 | 证据 |
|---|---|---|
| 决策 | 海草脑：`_can_cloud` 且非睡眠/死亡、附近（喷撒范围＋3）没有现存 `pollen` 标签实体时喷云；烧灼/燃烧时无视冷却立即喷（应急粉）。 | [should_spray_cloud](../../../raw/current/scripts/brains/waterplantbrain.lua#L29-L38)、[应急](../../../raw/current/scripts/brains/waterplantbrain.lua#L20-L24)、[距离](../../../raw/current/scripts/brains/waterplantbrain.lua#L26-L27) |
| 进入状态 | 脑 PushEvent `spray_cloud` → SGwaterplant 查睡眠/死亡/busy/spraying 后进 `cloud` 状态。 | [事件](../../../raw/current/scripts/stategraphs/SGwaterplant.lua#L24-L31)、[cloud 状态](../../../raw/current/scripts/stategraphs/SGwaterplant.lua#L413-L429) |
| 生成①号 | 状态内调 `inst.SpawnCloud`（即 `spawn_pollen_cloud`）：在植物位置生成 `waterplant_pollen_fx`、写入 `_source_flower`、置 `_can_cloud=false` 并起 `resetcloud` 计时（默认 6 段、粉色株改 3 段，± 半段方差）。 | [spawn_pollen_cloud](../../../raw/current/scripts/prefabs/waterplant.lua#L303-L313)、[计时调值](../../../raw/current/scripts/tuning.lua#L5541-L5545)、[粉色分支](../../../raw/current/scripts/prefabs/waterplant.lua#L64) |
| ①号→②号 | ①号 35 帧后在自身位置生成 `waterplant_pollen_fx_ocean`；若 `_source_flower` 仍有效且 21 距离（`SPRINKLER_DETECT_RANGE`(7)×3）内口水鱼少于 3 条，对花 PushEvent `pollenlanded`。 | [spawn_ocean_pollen](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L24-L39)、[检测距离](../../../raw/current/scripts/tuning.lua#L5092) |
| 放鱼 | 海草监听 `pollenlanded` → `childspawner:ReleaseAllChildren()`（childname=`oceanfish_small_9`，仅水中）。 | [监听](../../../raw/current/scripts/prefabs/waterplant.lua#L497)、[释放](../../../raw/current/scripts/prefabs/waterplant.lua#L315-L317)、[childspawner](../../../raw/current/scripts/prefabs/waterplant.lua#L469-L475) |

### ②号诱饵云的 chumpiece 生成链

服务端 `DoPeriodicTask(1.0, spawn_chum_piece)`：每秒尝试一次，块数少于 5（`MAX_PIECES`）时按随机角在 2–4 距离（`sqrt(random())*2+2`，环形带）取点，仅当该点为海面（`IsOceanAtPoint`）才生成 `chumpiece`，写入 `piece._source` 并监听其 `onremove`（[spawn_chum_piece](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L100-L118)、[周期](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L183)）。块被移除时回调 `on_piece_removed`：计数回退，且**只要宿主仍 `persists`（未开始消散）就立即补一块**，维持"始终约 5 块"的面（[on_piece_removed](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L86-L98)）。这个 `persists` 判断只作用于即时补位；1 秒周期任务在 `disperse` 开始后仍会继续，直到实体删除，所以淡出的 2 秒里仍可能补块。`chumpiece` 注册在 `chum_aoe.lua`（edible、次食类型 MEAT、无 AnimState、`persists=false`，[工厂](../../../raw/current/scripts/prefabs/chum_aoe.lua#L184-L211)），两个云共用同一被生成物。鱼群围拢靠鱼脑 `FindClosest(chum)` 索饵与进食行为，不是云主动寻鱼（[鱼脑](../../../raw/current/scripts/brains/oceanfishbrain.lua#L219-L220)）。

与鱼食袋链的 `chum_aoe` 同源辨析（`chum_aoe` 的完整机制归 [[reference/prefabs/eye_charge|眼弹与诱鱼区页]]）：

| 参数 | `waterplant_pollen_fx_ocean`（本页） | `chum_aoe`（鱼食袋落海） |
|---|---|---|
| 生成者 | 海草喷粉链（唯一生成点，见上） | `chum` 投掷物命中海面（[OnHit](../../../raw/current/scripts/prefabs/chum.lua#L19-L33)） |
| 寿命 | `POLLEN_DURATION`＝25 秒（[调值](../../../raw/current/scripts/tuning.lua#L5541-L5545)） | 常量 `DURATION`＝20 秒（[常量](../../../raw/current/scripts/prefabs/chum_aoe.lua#L11-L23)） |
| 补块周期／上限 | 1.0 秒／最多 5 块 | 0.5 秒／最多 7 块 |
| 落块范围 | 2–4 环形带，逐点验海面 | 半径 3 内随机，逐点验海面 |
| 放鱼方式 | 不生成新鱼群；`pollenlanded` 让海草释放自带的口水鱼 | `schoolspawner:SpawnSchool` 尝试生成鱼群（最多 5 次） |
| 存档 | 默认持久，disperse 才置 `persists=false`；读档经 `fn` 重建任务与计时 | `OnSave`/`OnLoad` 只存剩余放鱼尝试 |

### 端分界与清理

淡出是跨端协作：服务端 `on_timer_done`（`disperse` 到点）把 `persists` 置 false、杀 `spore_loop` 循环音、**摘 `chum` 标签**（先于视觉消失，鱼群即刻失去索饵目标）、置 `_fading` 净变量、2 秒后自删（[on_timer_done](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L120-L132)）；客户端（非专服）监听 `fadingdirty` 起逐帧任务，把 alpha 在 2 秒内从 1 降到 0（[client_fading_dirty](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L80-L84)、[update_fade](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L73-L78)）。宿主被直接移除（未走 disperse）时 `onremove` 监听 `on_ocean_removed` 把现存 chumpiece 一并删除（[on_ocean_removed](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L134-L140)）。①号云的清理只有一条路：主模拟监听 `animover` 自删（[L64](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L41-L71)）；该监听在客户端早退之后才注册，客户端不各自删除。

## 典型接入

### 复用"子实体补位循环"维持一组海面小块

- 前提与选择：需要在某点位周围长期维持至多 N 个子实体（诱饵/食物点/标记）时，原版模式是"宿主持计数表＋周期任务补块＋子实体 onremove 回调即时补位"，但 `persists` 只挡即时补位，不会停周期任务。
- 生效与撤销：宿主消散时置 `persists=false` 只能停掉 `onremove` 的即时补位，周期补块仍会跑到实体删除；若要"消散即停止所有新生成"，周期入口也要检查消散标志或持有并取消周期任务。宿主被直接移除时用 onremove 监听清场。[补位回调](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L86-L98)、[周期任务](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L180-L188)、[清场](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L134-L140)

```lua
-- 服务端：维持最多 5 个子块的补位循环（waterplant_pollen_fx_ocean 模式）
inst._pieces, inst._numpieces = {}, 0
inst._spawn_piece = function(inst)
    if inst._numpieces < 5 then
        local piece = SpawnPrefab("chumpiece")
        piece.Transform:SetPosition(inst.Transform:GetWorldPosition())
        piece._source = inst
        inst._pieces[piece] = true
        inst._numpieces = inst._numpieces + 1
        piece:ListenForEvent("onremove", function(p)
            inst._pieces[p] = nil
            inst._numpieces = inst._numpieces - 1
            if inst.persists then inst:_spawn_piece() end
        end)
    end
end
inst:DoPeriodicTask(1.0, inst._spawn_piece)
```

- 调整与验证：替换点是子块 prefab、落点判定与上限；注意补位任务不会因 `persists=false` 自动停——消散前若要停止所有新生成，必须让周期入口也检查消散标志或取消任务；只置 `persists=false` 仅挡即时补位。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：诱饵云是纯 FX。** 它同时挂 `chum`（鱼群索饵）与 `FX`／`NOCLICK` 标签，周期生成可食用块并持久存档；把它当装饰删掉会同时断掉引鱼与投喂（[工厂标签](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L160-L163)、[鱼脑](../../../raw/current/scripts/brains/oceanfishbrain.lua#L219-L220)）。
- **边界：`pollenlanded` 的放鱼是"释放海草自带的鱼"，不是新鱼群。** 口水鱼来自海草的 childspawner 存量（`ReleaseAllChildren`），放不放取决于 21 距离内口水鱼是否少于 3 条；诱饵云本身没有任何 schoolspawner。[spawn_ocean_pollen](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L33-L38)、[释放](../../../raw/current/scripts/prefabs/waterplant.lua#L315-L317)
- **边界：`chum` 标签在淡出开始时就摘掉。** disperse 到点先摘标签再淡出 2 秒，鱼群在视觉消失前约 2 秒就已失去目标；淡出只是观感。[on_timer_done](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L120-L132)
- **边界：诱饵云与 chum_aoe 的参数不可互换记忆。** 两云同 bank 系（`pollen_chum` 与 `fish_chum` 系动画同名族）、同被生成物 `chumpiece`，但寿命／周期／上限／放鱼方式都不同，见上表；改其一不影响另一个。
- **边界：①号云的自删与 35 帧生成都只在主模拟。** `animover` 删除监听在客户端早退之后才注册，客户端没有这条监听；35 帧生成任务也只在主模拟，且 `Remove` 会取消待执行任务，所以它不能当作动画被改短时的独立兜底。②号与放鱼的前提是"实体仍存活且该任务未被取消"。[L57-L68](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L57-L68)、[Remove 取消任务](../../../raw/current/scripts/entityscript.lua#L1739-L1746)
- **边界：层值同为 2 但常量不同。** 诱饵云用 `LAYER_BELOW_GROUND`(2)，海树根影系用 `LAYER_WIP_BELOW_OCEAN`(2)——数值相同、语义常量不同，跨页对比时按常量名核对，不要按数值归并（[层常量](../../../raw/current/scripts/constants.lua#L56-L68)）。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/waterplant_pollen.lua` 全文；按需读取 `prefabs/waterplant.lua` 喷粉/放鱼/childspawner 段、`brains/waterplantbrain.lua` 全文、`stategraphs/SGwaterplant.lua` 的 spray_cloud 事件与 cloud 状态、`prefabs/chum_aoe.lua` 的常量/补块/chumpiece 工厂段、`prefabs/chum.lua` 的 OnHit 段、`brains/oceanfishbrain.lua` 索饵段、`tuning.lua`／`constants.lua` 相关值。对 2 个注册名以完整名在 `raw/current/scripts` 全域反查：`waterplant_pollen_fx` 命中注册（waterplant_pollen.lua:195）、依赖（waterplant.lua:22）、生成（waterplant.lua:304）——**生成点唯一**；`waterplant_pollen_fx_ocean` 命中注册（waterplant_pollen.lua:196）、依赖（:13）、生成（:27）——**生成点唯一**（在①号工厂的 35 帧任务内）。台账批 9"每 1 秒生成 chumpiece"经核一致（`DoPeriodicTask(1.0, spawn_chum_piece)`，waterplant_pollen.lua:183）。`.po` 对两注册名与 `CHUMPIECE` 定向检索为 0 命中；`WATERPLANT`／`OCEANFISH_SMALL_9` 官方名已核。未运行游戏。
- 分档说明：`standard`——2 注册一条宿主链加一张同源对比表；未选 simple，因诱饵云有跨端淡出与持久存档语义需完整说明。
- 未覆盖：`chumpiece` 的进食消费端细节（鱼是否实际吃块、 hunger 结算）、海草本体三色/三阶段玩法、childspawner 生成时序、动画帧数与视觉、②号读档后剩余寿命与任务重建的运行表现（timer 组件存剩余时间，fn 重建周期任务，未运行核验）；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/eye_charge|眼睛炮塔弹体、诱鱼区与速爆弹]]（`chum_aoe`／`chumpiece` 同源链的主要页）
- [[reference/prefabs/oceantree_ripples_short|海树波纹、根须与落叶表现]]（贴地／水下分层渲染的另一组海面 FX）
- 返回 [[reference/prefabs/fx-water-index|水面、航海与船件]]
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
