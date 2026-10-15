---
title: "minionspawner"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "管理食人花眼球草的固定编号落点、延时补生、死亡回收与 GUID 恢复，区分宿主休眠和生成任务。"
aliases: ["食人花眼球草生成", "眼球草补生", "眼球草编号落点", "minionspawner"]
tags: [component, creature, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/minionspawner.lua", "prefabs/lureplant.lua", "prefabs/eyeplant.lua", "brains/lureplantbrain.lua", "behaviours/controlminions.lua", "stategraphs/SGeyeplant.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# minionspawner

## 快速判断

- 职责与适用：食人花宿主管理眼球草实体、编号位置与补生任务；没有巢内储备或入巢接口。世界自然刷出食人花另见 [[reference/components/lureplantspawner]]。[minionspawner.lua:39](../../../raw/current/scripts/components/minionspawner.lua#L39 [lureplant.lua:391](../../../raw/current/scripts/prefabs/lureplant.lua#L391
- 优先使用：维护既有食人花时同时核对 `shouldspawn`、任务标志、位置池及宿主冬眠／火伤条件；源码开头明确提醒不要以本文件为新组件范本。[minionspawner.lua:1](../../../raw/current/scripts/components/minionspawner.lua#L1 [lureplant.lua:90](../../../raw/current/scripts/prefabs/lureplant.lua#L90
- 重要副作用：生成硬调用子体 SG 的 `spawn` 状态；丢失子体后延迟 3 秒回收编号；批量杀死并非立即移除，也不自动关闭生成门禁。[minionspawner.lua:207](../../../raw/current/scripts/components/minionspawner.lua#L207 [minionspawner.lua:291](../../../raw/current/scripts/components/minionspawner.lua#L291 [minionspawner.lua:298](../../../raw/current/scripts/components/minionspawner.lua#L298
- 修改边界：原版在主模拟安装，依赖眼球草 SG／health 等配套；不是给任意 prefab 设置名称便可安全工作的通用生成器。[lureplant.lua:360](../../../raw/current/scripts/prefabs/lureplant.lua#L360 [eyeplant.lua:127](../../../raw/current/scripts/prefabs/eyeplant.lua#L127

## 关键接口

| 接口／字段 | 参数或约束 | 条件、作用与证据 |
|---|---|---|
| `miniontype`／`maxminions`／`numminions`／`minions` | 默认 eyeplant／27／0／实体键值表 | 计数与编号池共同约束生成；造点不足会改写 maxminions。直接改上限不会自动同步已有位置与空位池。[minionspawner.lua:39](../../../raw/current/scripts/components/minionspawner.lua#L39 [minionspawner.lua:240](../../../raw/current/scripts/components/minionspawner.lua#L240 |
| `minionspawntime`／`GetNextSpawnTime()` | `{min=5,max=10}`；秒 | 每轮在 min/max 间取随机等待，构造另外安排 1 秒后的首次 StartNextSpawn，不是构造即生成。[minionspawner.lua:43](../../../raw/current/scripts/components/minionspawner.lua#L43 [minionspawner.lua:57](../../../raw/current/scripts/components/minionspawner.lua#L57 [minionspawner.lua:281](../../../raw/current/scripts/components/minionspawner.lua#L281 |
| `shouldspawn`／`StartNextSpawn()` | 默认 true／无参数 | 只有允许生成、非进行中且未满员才设置 spawninprogress 并排任务；不重置已有等待。[minionspawner.lua:347](../../../raw/current/scripts/components/minionspawner.lua#L347 |
| `MakeMinion()`／`SpawnNewMinion()` | 前者返回新实体或 nil；后者无显式返回 | MakeMinion 只查类型与数量上限并 SpawnPrefab，不登记／放置；完整路径另查门禁、位置池、SG 并接管子体。[minionspawner.lua:229](../../../raw/current/scripts/components/minionspawner.lua#L229 [minionspawner.lua:298](../../../raw/current/scripts/components/minionspawner.lua#L298 |
| `TakeOwnership(minion)` | 已存在实体 | 同一表内防重复，递增数量、写 minionlord／必要时选 minionnumber、安装监听并向宿主发 minionchange；本方法不占用编号池，也不放置或验证 SG。[minionspawner.lua:178](../../../raw/current/scripts/components/minionspawner.lua#L178 |
| `OnLostMinion(minion)` | 已登记实体 | 删除登记／监听、减计数、安排编号回收、发 minionchange，允许生成且未满时尝试续排；不杀死实体，也不清其 minionlord 字段。[minionspawner.lua:207](../../../raw/current/scripts/components/minionspawner.lua#L207 |
| `KillAllMinions()` | 无参数 | 清 spawninprogress；为每个已登记子体安排 0～1 秒随机延迟的 health:Kill，死亡事件再负责减计数。未取消 self.task，也未把 shouldspawn 置 false。[minionspawner.lua:285](../../../raw/current/scripts/components/minionspawner.lua#L285 |
| `onspawnminionfn`／`onminionattacked`／`onminionattack` | `(宿主,子体)`／`(宿主)`／`(宿主)` | 生成回调在登记与定位后调用；受击／攻击回调仅在 TakeOwnership 当时非 nil 才安装对应子体监听。onlostminionfn 虽初始化为 nil，OnLostMinion 未调用它。[minionspawner.lua:47](../../../raw/current/scripts/components/minionspawner.lua#L47 [minionspawner.lua:178](../../../raw/current/scripts/components/minionspawner.lua#L178 [minionspawner.lua:207](../../../raw/current/scripts/components/minionspawner.lua#L207 [minionspawner.lua:320](../../../raw/current/scripts/components/minionspawner.lua#L320 |

## 关键流程

### 固定落点、生成与回收

首次 SpawnNewMinion 缺位置表时调用 MakeSpawnLocations：以宿主当时坐标计算最多 100 个螺旋候选，distancemodifier 默认 11，目标位置数按 maxminions×1.2。候选须在地上、地皮白名单允许、Pathfinder 的 ignorewalls 路径检查通过、2.5 范围无 eyeplant、1 范围无实体，且不靠洞口。找不足目标数会将 maxminions 改为实际可用数量，重建空位编号；一个点也无则返回 nil。[minionspawner.lua:46](../../../raw/current/scripts/components/minionspawner.lua#L46 [minionspawner.lua:240](../../../raw/current/scripts/components/minionspawner.lua#L240

位置缓存是世界坐标，不随宿主移动。GetSpawnLocation 仅重查编号存在及当前地皮白名单，不重做全部空位／路径／洞口检查。选中编号失效时可重新造点；不能理解为每次生成都完整验证附近实体占用。[minionspawner.lua:269](../../../raw/current/scripts/components/minionspawner.lua#L269 [minionspawner.lua:298](../../../raw/current/scripts/components/minionspawner.lua#L298

生成门禁通过且有空编号时，先清 spawninprogress，再随机取编号；成功创建后进入 `spawn` 状态、写编号、TakeOwnership、定位、从空位池移除编号并调生成回调，未满时续排。造点整体失败会在清标志前直接返回；shouldspawn=false、已满或无空编号也不会走这段清标志／续排逻辑，因此“进行中”不总等于仍有一个未来回调。[minionspawner.lua:298](../../../raw/current/scripts/components/minionspawner.lua#L298

子体 death 监听先向该子体补发 `attacked`，再 OnLostMinion；onremove 也走 OnLostMinion，登记检查避免同一子体重复减数。回收任务捕获丢失当时的 freepositions 表，3 秒后调 AddPosition；若期间位置池重建，编号会归入旧表。[minionspawner.lua:59](../../../raw/current/scripts/components/minionspawner.lua#L59 [minionspawner.lua:202](../../../raw/current/scripts/components/minionspawner.lua#L202

AddPosition 只有遇到已有更小编号才插入，没有空表或遍历结束后的追加分支；不能把它当作保证成功归还任意编号的通用资源池。TakeOwnership 也不会自动从旧宿主转移账目，外来实体接管须另行处理旧登记与空位一致性。[minionspawner.lua:89](../../../raw/current/scripts/components/minionspawner.lua#L89 [minionspawner.lua:178](../../../raw/current/scripts/components/minionspawner.lua#L178

### 保存恢复与离线推进

OnSave 保存已登记子体的 GUID／NUMBER 并返回引用表，另存 maxminions、位置 x/z；生成中时保存剩余时间向上取整且至少 1 秒。shouldspawn、回调、地皮白名单、随机间隔和延迟回收任务不在保存数据中。[minionspawner.lua:118](../../../raw/current/scripts/components/minionspawner.lua#L118

OnLoad 恢复上限与位置表，若保存为生成中则 ResumeSpawn；该内部恢复口把时间限制为至少 1 秒后排任务，**不自行取消已有任务或检查 shouldspawn**，不宜作为任意重复调用的启动接口。LoadPostPass 按 GUID 找回子体、恢复编号并重新接管；有合法位置时重新传送／定位并占用编号，缺失引用跳过。此路径不调用 onspawnminionfn。[minionspawner.lua:144](../../../raw/current/scripts/components/minionspawner.lua#L144 [minionspawner.lua:156](../../../raw/current/scripts/components/minionspawner.lua#L156 [minionspawner.lua:354](../../../raw/current/scripts/components/minionspawner.lua#L354

LongUpdate 仅在生成中且 shouldspawn 时取消当前 task，再按 dt 反复抽取完整随机间隔估算生成尝试数，逐次 SpawnNewMinion 并取消中间新排的任务；它不是按 nextspawninfo 的精确剩余时间回放。宿主冬眠剩余时间另由 lureplant 的 OnLongUpdate 处理。[minionspawner.lua:359](../../../raw/current/scripts/components/minionspawner.lua#L359 [lureplant.lua:229](../../../raw/current/scripts/prefabs/lureplant.lua#L229

## 典型接入

### 原版食人花：冬眠、采肉与火伤共同控制眼球草

`Prefab("lureplant", fn, assets, prefabs)` 在服务端安装本组件，覆盖 validtiletypes，设置眼球草受击时 HideBait、新生时传递皮肤的回调。眼球草 Prefab 提供 health 与 SGeyeplant，其 spawn 状态负责停止物理、播放出生动画；这套配套解释了为何只替换 miniontype 不足以接入任意生物。[lureplant.lua:360](../../../raw/current/scripts/prefabs/lureplant.lua#L360 [lureplant.lua:391](../../../raw/current/scripts/prefabs/lureplant.lua#L391 [lureplant.lua:442](../../../raw/current/scripts/prefabs/lureplant.lua#L442 [lureplant.lua:303](../../../raw/current/scripts/prefabs/lureplant.lua#L303 [eyeplant.lua:127](../../../raw/current/scripts/prefabs/eyeplant.lua#L127 [SGeyeplant.lua:33](../../../raw/current/scripts/stategraphs/SGeyeplant.lua#L33

ResumeSleep、采走诱饵、freshspawn、宿主死亡及开始火伤都先 shouldspawn=false，再 KillAllMinions；WakeUp 非冬季时恢复门禁并 StartNextSpawn，停止火伤则还要求不在冬眠、宿主未死且非冬季。冬季周期性延长冬眠，宿主保存剩余唤醒时间并在 OnLoad 重建这层门禁；不能从组件默认 true 推导食人花全年持续补生。[lureplant.lua:75](../../../raw/current/scripts/prefabs/lureplant.lua#L75 [lureplant.lua:90](../../../raw/current/scripts/prefabs/lureplant.lua#L90 [lureplant.lua:108](../../../raw/current/scripts/prefabs/lureplant.lua#L108 [lureplant.lua:137](../../../raw/current/scripts/prefabs/lureplant.lua#L137 [lureplant.lua:199](../../../raw/current/scripts/prefabs/lureplant.lua#L199 [lureplant.lua:215](../../../raw/current/scripts/prefabs/lureplant.lua#L215 [lureplant.lua:242](../../../raw/current/scripts/prefabs/lureplant.lua#L242 [lureplant.lua:276](../../../raw/current/scripts/prefabs/lureplant.lua#L276

行为与物品转运另有消费者：lureplantbrain 使用 ControlMinions，根据位置表确定扫描半径，并给附近合适的眼球草分派采集／拾取动作；食人花每 2 秒 CollectItems 从已登记眼球草库存收物。读档后的皮肤通过宿主 OnLoadPostPass 的 SetSkin 重施，不能依赖只在新生路径调用的回调。[lureplantbrain.lua:8](../../../raw/current/scripts/brains/lureplantbrain.lua#L8 [controlminions.lua:56](../../../raw/current/scripts/behaviours/controlminions.lua#L56 [lureplant.lua:152](../../../raw/current/scripts/prefabs/lureplant.lua#L152 [lureplant.lua:307](../../../raw/current/scripts/prefabs/lureplant.lua#L307 [lureplant.lua:424](../../../raw/current/scripts/prefabs/lureplant.lua#L424

## 常见陷阱与限制

- **KillAllMinions 等于停机**：它不取消已经排出的生成回调；若 shouldspawn 仍真，子体死亡还可继续触发补生。原版停机组合及恢复条件见上节。[minionspawner.lua:207](../../../raw/current/scripts/components/minionspawner.lua#L207 [minionspawner.lua:291](../../../raw/current/scripts/components/minionspawner.lua#L291
- **只切 shouldspawn 就可靠暂停／恢复**：字段不取消任务，也不清 spawninprogress；旧回调若因门禁失败而退出，之后只调 StartNextSpawn 可能仍被该标志挡住。造点失败／编号池耗尽也需核对状态，不承诺自动重试。[minionspawner.lua:298](../../../raw/current/scripts/components/minionspawner.lua#L298 [minionspawner.lua:347](../../../raw/current/scripts/components/minionspawner.lua#L347
- **直接移除组件会自动清场**：本组件没有 OnRemoveFromEntity 或统一取消所有延时任务的入口；初始 1 秒启动任务也未保存在 self.task。主动拆卸前应按实际宿主流程安排门禁、任务及子体清理，本页不提供通用热移除方案。[minionspawner.lua:39](../../../raw/current/scripts/components/minionspawner.lua#L39 [minionspawner.lua:291](../../../raw/current/scripts/components/minionspawner.lua#L291

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、lureplant 生成门禁／冬眠／死亡／火伤／存读／皮肤、eyeplant 配套、ControlMinions 分派与 Brain 接线、SG 出生状态。未展开全部采集动作内部、眼球草完整战斗 SG 或引擎地图查询实现。
- 结构检查：本页 validate_page 及源码引用 strict 检查通过；无新增完整 Lua 示例。
- 游戏运行：未运行；地形改动、位置不足／回收边界、重复恢复、休眠与火伤交错、保存装载及离线推进待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/lureplantspawner]]；[[reference/components/childspawner]]；[[reference/components/objectspawner]]。
