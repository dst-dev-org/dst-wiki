---
title: "sheltered"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "服务端逐帧判定实体是否处于庇护，驱动庇护副本、sheltered 事件与雨天防水贡献。"
aliases: ["庇护", "树荫庇护", "避雨判定", "庇护等级", "sheltered"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/sheltered.lua", "components/sheltered_replica.lua", "components/moisture.lua", "components/rider.lua", "components/lunarhailmanager.lua", "prefabs/player_common.lua", "prefabs/wx78_possessedbody.lua", "prefabs/wx78_backupbody.lua", "prefabs/player_common_extensions.lua", "prefabs/oceantree_pillar.lua", "prefabs/torch.lua", "entityreplica.lua", "entityscript.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sheltered

## 快速判断

- 职责与适用：挂在**需要避雨／遮荫状态的实体**上，逐帧用世界实体重算庇护，并把结果写进副本、事件与雨天防雨贡献；原版宿主只有玩家与 WX78 备份身体。[构造](../../../raw/current/scripts/components/sheltered.lua#L1-L13) [玩家装配](../../../raw/current/scripts/prefabs/player_common.lua#L2737-L2739) [WX78 装配](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L655)
- 优先使用：判断“是否被遮住”在服务端读 `inst.components.sheltered.sheltered`，变更用 `sheltered` 事件（payload 含 `sheltered`、`level`）；要**提供**庇护则给附近实体加 `shelter` 标签，或维护目标的 `canopytrees` 计数。[等级判定](../../../raw/current/scripts/components/sheltered.lua#L65-L82)
- 重要副作用：庇护不等于免湿，只向 moisture 的雨水速率贡献 `waterproofness = 0.35`，等级 2 不额外加成；原版还据它去掉雨天火把的额外耗油，并让庇护中的玩家不被月雹碎块命中。[防水贡献](../../../raw/current/scripts/components/moisture.lua#L212-L235) [火把](../../../raw/current/scripts/prefabs/torch.lua#L212-L218) [月雹](../../../raw/current/scripts/components/lunarhailmanager.lua#L146-L148)
- 修改边界：仅服务端运行，客户端只有 replica；构造即开始逐帧更新，死亡 `Stop`、复活 `Start`；无存档字段，读档由工厂重建后自行重算。[停止／重启](../../../raw/current/scripts/components/sheltered.lua#L19-L33)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `sheltered`／`presheltered` | boolean | 前者表示已发出 `sheltered` 事件，后者表示已向副本下发庇护；两者时序不同，见下节。[状态机](../../../raw/current/scripts/components/sheltered.lua#L35-L61) |
| `sheltered_level` | number | 最近一次 `SetSheltered` 的等级；只进事件 payload，原版消费者不区分 1／2。[赋值](../../../raw/current/scripts/components/sheltered.lua#L39) |
| `waterproofness` | number，默认 `TUNING.WATERPROOFNESS_SMALLMED`＝0.35 | 仅当 `sheltered` 为真时被 moisture 计入雨水防水汇总。[默认值](../../../raw/current/scripts/components/sheltered.lua#L10) [常量](../../../raw/current/scripts/tuning.lua#L2950) [汇总](../../../raw/current/scripts/components/moisture.lua#L218-L220) |
| `mounted` | boolean，默认 false | rider 骑乘／下骑时置位；为真时 `level < 2` 一律不算庇护（骑乘只享受树冠级）。[骑乘](../../../raw/current/scripts/components/rider.lua#L116-L118) [门槛](../../../raw/current/scripts/components/sheltered.lua#L36-L38) |
| `announcecooldown` | number，秒 | 庇护台词冷却；触发后置 `TUNING.TOTAL_DAY_TIME`（480 秒）。[触发](../../../raw/current/scripts/components/sheltered.lua#L55-L59) [常量](../../../raw/current/scripts/tuning.lua#L19-L20) |
| `Start()`／`Stop()` | 无参 | 重新开始／停止逐帧更新；`Stop` 以停机时刻记录 `stoptime` 并立即解除庇护。[L19-L33](../../../raw/current/scripts/components/sheltered.lua#L19-L33) |
| `SetSheltered(issheltered, level)` | boolean, number? | 唯一写入口：改副本、发事件、播台词；`level` 省略时仍会写 `sheltered_level = nil`。[L35-L61](../../../raw/current/scripts/components/sheltered.lua#L35-L61) |
| `OnUpdate(dt)` | number | 每帧：`canopytrees > 0` → 等级 2；否则 `CountEntities(...,2,"shelter",...)` 有命中 → 等级 1。[L65-L82](../../../raw/current/scripts/components/sheltered.lua#L65-L82) |

副本侧：服务端构造组件时也会建一份（`AddComponent` 先登记），客户端由 `_sheltered` 标签建；`StartSheltered()`／`StopSheltered()` 写 net bool，`IsSheltered()` 要求遮罩已降到 0.6，`OnUpdate` 每帧向 0.6／1 过渡。[副本接口](../../../raw/current/scripts/components/sheltered_replica.lua#L39-L51) [遮罩过渡](../../../raw/current/scripts/components/sheltered_replica.lua#L66-L78)

### 副本与事件时序

1. 每帧 `OnUpdate` 先算布尔值，再调 `SetSheltered`。[L65-L82](../../../raw/current/scripts/components/sheltered.lua#L65-L82)
2. 首次为真：`presheltered = true` 并 `replica.sheltered:StartSheltered()`，net bool 立即下发。[L49-L51](../../../raw/current/scripts/components/sheltered.lua#L49-L51)
3. 副本遮罩按 `TIME_TO_SHELTER = 0.2` 秒向 0.6 过渡；`IsSheltered()` 在过渡完成前为假。[常量](../../../raw/current/scripts/components/sheltered_replica.lua#L1-L4) [判定](../../../raw/current/scripts/components/sheltered_replica.lua#L49-L51)
4. 过渡完成后的某一帧才置 `sheltered = true` 并 `PushEvent("sheltered", { sheltered = true, level = ... })`。[L52-L54](../../../raw/current/scripts/components/sheltered.lua#L52-L54)
5. 离开时 `presheltered` 立即清、`StopSheltered()` 与 `sheltered=false` 事件在同一帧完成。[L40-L48](../../../raw/current/scripts/components/sheltered.lua#L40-L48)

因此从踏进庇护范围到 `sheltered = true` 事件约晚 0.2 秒，离开则同一帧发出；依赖“当前是否已庇护”的逻辑应读字段，需要一次通知才用事件。

## 典型接入

### 原版玩家宿主

- 前提与位置：`player_common` 在 `SetPristine` 与服务端守卫之后装配 moisture／sheltered，任何角色 prefab 经 `MakePlayerCharacter` 都会带上。[守卫](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2614) [装配](../../../raw/current/scripts/prefabs/player_common.lua#L2737-L2739)
- 端与资源：仅服务端；无贴图／音频资源；单位为世界单位，`shelter` 查询半径 2。
- 生效与撤销：常驻更新；死亡流程 `Stop()`，复活流程 `Start()` 并抵扣停机期间的台词冷却。[死亡](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L632-L646) [复活](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L267-L302)
- 原版依据：庇护来源由 `oceantree_pillar`／`watertree_pillar` 的 playerprox 增减玩家 `canopytrees`，其余来源靠 `shelter` 标签。[增减](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L68-L82)

### 非玩家宿主参照

- `wx78_possessedbody` 在服务端守卫之后同样 `AddComponent("sheltered")`，由 `wx78_backupbody` 生成备份身体时走同一工厂。[守卫](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L569-L571) [装配](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L655) [生成点](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L175)
- 可加在任意服务端实体上：`AddComponent` 会先登记可复刻组件再构造，`inst.replica.sheltered` 在构造函数内已可用，客户端副本是否存在由 `_sheltered` 标签决定。[可复刻清单](../../../raw/current/scripts/entityreplica.lua#L5-L26) [登记标签](../../../raw/current/scripts/entityreplica.lua#L39-L45) [标签判定](../../../raw/current/scripts/entityreplica.lua#L30-L32) [构造顺序](../../../raw/current/scripts/entityscript.lua#L648-L656)
- 给实体提供庇护的两种等级：① 在目标 2 单位内放置带 `shelter` 标签的实体（`FX`／`NOCLICK`／`DECOR`／`INLIMBO`／`stump`／`burnt` 会被排除）；② 维护目标的 `canopytrees` 计数，原版由 playerprox 的 near／far 成对增减。[排除标签](../../../raw/current/scripts/components/sheltered.lua#L63-L64) [增减](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L68-L82) [距离](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L581-L585)
- 撤销：移除或隐藏提供者；用 `canopytrees` 方案时必须在提供者移除时归零，原版在 `OnRemoveEntity` 遍历玩家补发 `OnFar`。[清理](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L353-L368)

## 常见陷阱与限制

- **进入事件晚于状态约 0.2 秒**：由副本遮罩过渡决定；把 `sheltered` 事件当“刚踏进树荫的瞬间”会漏掉最初几帧。[判定](../../../raw/current/scripts/components/sheltered_replica.lua#L49-L51)
- **等级 2 不等于更防雨**：moisture 只读 `sheltered` 布尔与固定 0.35；等级差异只体现在事件 payload 与骑乘门槛。[汇总](../../../raw/current/scripts/components/moisture.lua#L218-L220)
- **骑乘会取消低等级庇护**：`mounted` 为真且 `level < 2` 时强制不庇护；`Stop()`／`OnRemoveFromEntity()` 调用 `SetSheltered(false)` 不带 `level`，此时若 `mounted` 为真会对 nil 做 `<` 比较，自定义调用应显式传等级；原版死亡流程是否已先解除骑乘本页未核验。[门槛](../../../raw/current/scripts/components/sheltered.lua#L36-L38) [清理](../../../raw/current/scripts/components/sheltered.lua#L15-L17)
- **`canopytrees` 泄漏会永久保持 2 级**：它是玩家身上的裸计数，增减必须成对，且提供者移除时要补偿；原版为此专门写了清理循环。[清理](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L353-L368)
- **不要用 `RemoveComponent` 代替 `Stop`**：移除会先停更新再摘 `_sheltered` 副本标签，客户端副本随之失效，遮罩状态也不再维护。[移除顺序](../../../raw/current/scripts/entityscript.lua#L670-L682)
- **每帧一次世界查询**：`CountEntities` 半径 2 每帧执行；给多个实体挂 sheltered 会增加服务器开销，不要用它做高频范围检测。[L76-L79](../../../raw/current/scripts/components/sheltered.lua#L76-L79)
- **手动写字段无效**：下一帧 `OnUpdate` 会覆盖；`Stop()` 只会强制解除并停止更新，无法制造庇护。要临时遮住只能改世界侧来源（`shelter` 标签实体或 `canopytrees` 计数）。

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 3 处（player_common／tuning 行号迁移）；结论修订：无。R-08 的 CanPointHaveRain 只改 iswet 回退判定，不触及 moisture 对 sheltered 的 0.35 防水汇总与月雹／火把消费。
- 依据：全局枢纽 hunk 地图（R-08）＋变更清单命中 prefabs/player_common.lua、prefabs/player_common_extensions.lua、prefabs/wx78_possessedbody.lua、tuning.lua、entityscript.lua。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖构造、Start／Stop、SetSheltered 全部分支、OnUpdate 两路判定、副本 net bool／遮罩过渡与事件时序、玩家与 WX78 装配位置、rider 的 mounted 置位、moisture 0.35 贡献、火把与月雹消费、canopy 计数与移除清理、组件移除顺序。
- 未运行游戏；未覆盖：全部 `shelter` 标签提供者、骑乘中死亡的实际时序、talker 台词触发、客户端表现、多人联机与模组叠加。
- 本页无完整 Lua 示例，未执行 Lua 语法检查。
- 结构检查：`validate_page.py` 无 issue；`validate_source_links.py --strict` 为 0 error／0 warning（本页 51 处源码引用）；无完整示例，未执行 Lua 语法检查。

相关：[[reference/components/moisture]]；[[reference/components/rider]]；[[reference/components/rainimmunity]]；[[reference/components/talker]]。
