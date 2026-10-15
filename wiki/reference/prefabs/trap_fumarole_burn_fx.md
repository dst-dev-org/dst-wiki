---
title: "喷气孔陷阱贴身灼烧与藤蔓陷阱底座（trap_fumarole_burn_fx、trap_vines_base_fx）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "两个宿主驱动的联网地面/贴身 FX：喷气孔煤炭陷阱给受灼实体挂的贴身火焰层（端上由 trapfumaroleburning 组件管生灭、无自删），投石机暗影弹落点藤蔓陷阱的地面底座（复用 shadow_pillar_fx 的 bank/build，随宿主生成与收尾）。"
aliases: ["trap_fumarole_burn_fx", "trap_vines_base_fx", "喷气孔陷阱灼烧特效", "煤炭陷阱火焰层", "藤蔓陷阱底座", "投石机藤蔓陷阱", "shadow_pillar_fx 复用底座", "trapfumaroleburning 表现层"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/trap_fumarole.lua", "prefabs/trap_fumarole_util.lua", "components/trapfumaroleburning.lua", "prefabs/trap_vines.lua", "prefabs/winona_catapult_projectile.lua", "stategraphs/SGwinona_catapult.lua", "componentutil.lua", "fx.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 喷气孔陷阱贴身灼烧与藤蔓陷阱底座（trap_fumarole_burn_fx、trap_vines_base_fx）

## 快速判断

- 职责与适用：两个 FX 都"生在宿主、随宿主收尾"——①`trap_fumarole_burn_fx` 是喷气孔陷阱灼烧判定给受灼实体（生物／物品）父挂的**贴身火焰动画层**（`burn_hit_small/med/large` 循环＋泛光），由 `trapfumaroleburning` 组件管理生灭；②`trap_vines_base_fx` 是投石机暗影弹落点藤蔓陷阱 `trap_vines` 的**地面底座贴片**（pre→idle 循环、半透明点采样），工厂暴露 `KillFX` 做退场。[burn 注册](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L718-L720)、[底座注册](../../../raw/current/scripts/prefabs/trap_vines.lua#L308-L309)
- 本体辨析：`trap_fumarole`（官方名"煤炭"，[po](../../../raw/current/scripts/languages/chinese_s.po#L350539-L350542)）是可合成可放置的**玩法陷阱物品**——按温度分 3 档、烤制／点燃／火伤全在其 `OnUpdate` 与 `trapfumaroleburning` 组件里，本页只辨析链条不展开；`trap_vines` 本体自带 combat／planardamage／减速 debuff（14＋20 平面伤害、×2/3 减速），**是会打人的玩法载体**而非纯 FX，其玩法细节见下节必要部分。[陷阱工厂](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L550-L654)、[藤蔓工厂](../../../raw/current/scripts/prefabs/trap_vines.lua#L215-L266)
- 重要副作用：两个 FX 自身都无伤害；`trap_fumarole_burn_fx` **不带惯用的 `FX` 标签**（只有 `DECOR`／`NOCLICK`），且**无自删**——清理完全依赖组件移除；`trap_vines_base_fx` 复用影柱法术的 `shadow_pillar_fx` bank／build（台账批 8 结论复核属实；资产声明见[藤蔓资产表](../../../raw/current/scripts/prefabs/trap_vines.lua#L6-L9)），但 alpha 0.6、无涟漪、父挂藤蔓，[[reference/prefabs/shadow_pillar|暗影囚牢·影柱法术体系]]页另记此复用关系。[burn 标签](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L695-L696)、[底座 bank](../../../raw/current/scripts/prefabs/trap_vines.lua#L285-L292)
- 修改边界：只覆盖 2 注册的工厂、生成点、父挂与清理；喷气孔陷阱的温度／烤制数值体系、投石机弹药与 Winona 技能树、`fumarole_ember`／`fumarole_cook_fx`（注册于 fx.lua 的链条 FX，本页不覆盖）不展开。
- 名称：`TRAP_FUMAROLE`＝"煤炭"（msgid "Coals"，[po](../../../raw/current/scripts/languages/chinese_s.po#L350539-L350542)）；`TRAP_VINES`、`TRAP_FUMAROLE_BURN_FX`、`TRAP_VINES_BASE_FX` 在简中 `chinese_s.po` 均无 `STRINGS.NAMES` 键（"TRAP_VINES" 全文 0 命中），标题中文为已核验效果描述，非官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `trap_fumarole_burn_fx` | `fxfn`：Transform／AnimState／Network → 标签 `DECOR`＋`NOCLICK`（**无 `FX`**）→ bank=build=`trap_fumarole_burn_fx`、循环播 `burn_hit_small`、泛光 `shaders/anim.ksh`、FinalOffset 3 → `SetPristine` → 客户端早退 → 服务端暴露 `SetFxSize` 并置 `persists=false`。 | 贴身火焰层：`SetFxSize` 按 `burn_hit_`..size 换动画（不重复播当前档）；无 Follower——父挂由生成方做；无自删监听。[工厂](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L688-L714)、[SetFxSize](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L681-L686)、[注册](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L720) |
| `trap_vines_base_fx` | `base_fn`：Transform／AnimState／Network → 标签 `FX`＋`NOCLICK` → bank=build=`shadow_pillar_fx`、播 `pre`、alpha 0.6、点采样、贴地、背景层、SortOrder 3 → `SetPristine` → 客户端早退 → 服务端续播 `idle` 循环、暴露 `KillFX`、`persists=false`。 | 地面底座：`KillFX` 播 `pst` 后 `animover` 自删；与影柱贴片共用同一动画包（`shadow_pillar_fx.zip`，ALT_RENDERPATH），但工厂内无缩放、无海面涟漪。[工厂](../../../raw/current/scripts/prefabs/trap_vines.lua#L275-L306)、[KillFX](../../../raw/current/scripts/prefabs/trap_vines.lua#L270-L273)、[注册](../../../raw/current/scripts/prefabs/trap_vines.lua#L309) |

同文件未被本页收为覆盖注册的还有：`trap_fumarole` 本体、`trap_fumarole_placer`（[工厂](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L550-L654)、[placer](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L658-L677)）与 `trap_vines` 本体（[工厂](../../../raw/current/scripts/prefabs/trap_vines.lua#L215-L266)）；喷气孔链的 `fumarole_ember`／`fumarole_cook_fx` 注册在 [fx.lua L3966／L3997](../../../raw/current/scripts/fx.lua#L3966-L3997)，属链条 FX 不入本页。

## 关联机制导航

### 灼烧链：谁给谁挂 burn_fx、什么时候挂

| 环节 | 触发与条件 | 端 | 证据 |
|---|---|---|---|
| 组件安装 | 陷阱激活（`settrap` 且 `_temperaturerange>=2`）时 0.5 秒周期扫描半格瓦片方形区域，对带 health／inventoryitemtemperature／cookable／可引燃 propagator 的实体 `AddComponent("trapfumaroleburning")` | 主模拟 | [OnUpdate](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L45-L73) |
| FX 生成 | 组件 `SetFxEnabled(true)`：首次生成 `trap_fumarole_burn_fx` 并 `SetParent(受灼实体)`，给宿主 `colouradder:PushColour(0.1,0,0,0)` 微红染；尺寸按宿主（骑乘时按坐骑）`GetCombatFxSize`，`tiny` 档归并为 `small` | 主模拟 | [SetFxEnabled](../../../raw/current/scripts/components/trapfumaroleburning.lua#L24-L51)、[体型档](../../../raw/current/scripts/componentutil.lua#L959-L976) |
| 显隐周期 | 离开有效区（`cleartime` 分支）只 `Hide`＋`_hidden=true`＋PopColour，不删除；回到有效区再 `Show` 复用同一实体 | 主模拟 | [隐藏分支](../../../raw/current/scripts/components/trapfumaroleburning.lua#L46-L50)、[OnUpdate](../../../raw/current/scripts/components/trapfumaroleburning.lua#L135-L146) |
| FX 移除 | 宿主连续 1 秒不在有效区 → `RemoveComponent("trapfumaroleburning")` → 组件 `OnRemoveFromEntity` 里 `self.fx:Remove()`；宿主被移除时同样走组件移除路径 | 主模拟 | [组件移除](../../../raw/current/scripts/components/trapfumaroleburning.lua#L143-L144)、[FX 移除](../../../raw/current/scripts/components/trapfumaroleburning.lua#L15-L22) |

灼烧玩法链（辨析）：组件按陷阱温度档乘 `TRAP_FUMAROLE_IGNITE_MULTS`（0／0.5／1）累计点燃值，每 14 帧对宿主（或坐骑）结算 `TRAP_FUMAROLE_DAMAGE`(10) 火伤；点燃完成后 cookable 物品变熟并父挂 `fumarole_cook_fx`，可引燃 propagator 加热（[点燃与烤制](../../../raw/current/scripts/components/trapfumaroleburning.lua#L77-L130)、[调值](../../../raw/current/scripts/tuning.lua#L9501-L9512)）。陷阱本体在客户端也注册进瓦片登记表（用于脚步声等表现，[util.SetTrap](../../../raw/current/scripts/prefabs/trap_fumarole_util.lua#L51-L62)），激活陷阱期间空抽 5%／10% 概率落 `fumarole_ember` 火星（[落星](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L75-L84)）。

### 藤蔓链：投石机弹落到藤蔓陷阱到底座

投石机挥击在 `SGwinona_catapult` 的攻击状态生成联网弹体并写入元素／巨型标记（[弹体生成](../../../raw/current/scripts/stategraphs/SGwinona_catapult.lua#L450-L465)）；弹体落地回调在**巨型（元素齐射）暗影／混合弹**时调 `SpawnAOETrap`：按 AOE 等级排 1～3 圈点位，可通行且部署点干净才逐点生成（[落点撒环](../../../raw/current/scripts/prefabs/winona_catapult_projectile.lua#L291-L303)、[落地触发](../../../raw/current/scripts/prefabs/winona_catapult_projectile.lua#L406-L408)）。`SpawnTrapRing` 先清掉 1 格内旧藤蔓（逐个 `DespawnTrap`），再 `SpawnPrefab("trap_vines")` 写入 `attacker`（投石机）与 `caster`（玩家），本轮第一个藤蔓起循环音效（[环内生成](../../../raw/current/scripts/prefabs/winona_catapult_projectile.lua#L267-L288)）。

藤蔓本体是**联网持久玩法实体**：随机延迟后 `OnInit` 才 `Show` 并生成底座 `trap_vines_base_fx` 父挂自身、播 `spawn..variation`（3 变体），0.1 秒周期扫描 `AOE_RADIUS(1)+3` 内可目标——移动中的敌人吃 14＋20 平面伤害（每目标 1.5 秒冷却）与 ×2/3 减速（[OnInit](../../../raw/current/scripts/prefabs/trap_vines.lua#L178-L187)、[伤害与减速](../../../raw/current/scripts/prefabs/trap_vines.lua#L72-L137)、[调值](../../../raw/current/scripts/tuning.lua#L3692-L3696)）。300 秒 `decay` 计时到期 `DespawnTrap`（[计时](../../../raw/current/scripts/tuning.lua#L3695)）。

与 [[reference/prefabs/shadow_pillar|暗影囚牢·影柱法术体系]] 的关系：该页已记"`trap_vines.lua` 只复用同名 `shadow_pillar_fx.zip`／bank"（其验证记录反查条目）；`trap_vines_base_fx` 的工厂与影柱 `shadow_pillar_base_fx` 的差异是——无 1 秒延迟的海面涟漪任务、生成后由藤蔓控制父挂与收尾、藤蔓读档时直接播 `idle` 而非重走 `pre`。bank 同源、行为各自维护。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `trap_fumarole_burn_fx` | 无自删；唯一显式移除在组件 `OnRemoveFromEntity` 的 `fx:Remove()`（组件移除或宿主移除时触发） | 显隐周期里实体一直存活；若宿主不经组件移除路径直接消失，依赖引擎父挂级联（未运行核验）。[移除点](../../../raw/current/scripts/components/trapfumaroleburning.lua#L19-L21) |
| `trap_vines_base_fx` | 宿主 `decay` 到期 `DespawnTrap`：底座先 `SetParent(nil)`＋挪回藤蔓坐标再 `KillFX`（播 `pst`→`animover` 自删）；藤蔓本体另播 `despawn..variation` 后自删；宿主被直接移除时靠引擎父挂级联 | `DespawnTrap` 前底座随藤蔓移动； OnInit 前（随机 0～0.3 秒内）被 despawn 走"无底座直接 Remove"分支。[DespawnTrap](../../../raw/current/scripts/prefabs/trap_vines.lua#L143-L163) |
| 链上本体 | `trap_vines` 持久存档（OnSave 记 variation／sound，[存读](../../../raw/current/scripts/prefabs/trap_vines.lua#L189-L213)），读档补建底座播 `idle`；`trap_fumarole` OnSave 只记 `settrap`（[存读](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L518-L528)），石头群与地面火星都是非联网本地实体按动画帧重建 | 两本体的落点撒环／放置逻辑跨会话重建路径不同：藤蔓读档续跑任务，煤炭陷阱读档只还原"已放置"态。 |

## 典型接入

### 给"灼烧区域"里的实体挂随生灭的表现层

- 前提与选择：区域判定实体（陷阱／地表）在服务端周期扫描并给命中者装标记组件，组件负责 FX 的生成、显隐与移除；FX 工厂只管外观（尺寸换动画交给生成方调 `SetFxSize`）。适合贴身持续表现（灼烧、腐蚀）。
- 生效与撤销：`SpawnPrefab` 后 `SetParent(宿主)`；撤销必须落在组件的 `OnRemoveFromEntity`，而不是指望 FX 自删——原版 FX 工厂没有 `animover` 监听。[工厂](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L688-L714)、[组件移除](../../../raw/current/scripts/components/trapfumaroleburning.lua#L15-L22)

```lua
-- 服务端：区域表现组件的挂接与回收（对应 trapfumaroleburning 的模式）
self.fx = SpawnPrefab("trap_fumarole_burn_fx")
self.fx.entity:SetParent(self.inst.entity)
self.fx:SetFxSize("med")
-- 组件移除时：
if self.fx then self.fx:Remove() end
```

- 调整与验证：替换点是注册名与动画名；`_hidden` 是本链的自定义字段（Hide／Show 复用同实体），复刻时要自己维护。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：`trap_fumarole_burn_fx` 不带 `FX` 标签。** 工厂只加 `DECOR`／`NOCLICK`（[L695-L696](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L695-L696)）；按 `"FX"` 标签检索该实体必然落空，喷气孔陷阱自己的扫描排除表也因此不需要单列它。
- **误区：burn_fx 是"燃烧"状态。** 它不是着火（`burnable` 无关），显示的是"站在灼热瓦片上"的贴身受热动画；宿主真的烧起来时反而 `SetFxEnabled(false)` 隐藏（[隐藏条件](../../../raw/current/scripts/components/trapfumaroleburning.lua#L134)）。
- **边界：火焰尺寸只有三档且看宿主不看陷阱。** `GetCombatFxSize` 的 `tiny` 在本链归并为 `small`，最终动画名只有 `burn_hit_small/med/large`（[归并](../../../raw/current/scripts/components/trapfumaroleburning.lua#L41-L45)、[档位](../../../raw/current/scripts/componentutil.lua#L964-L976)）；骑乘坐骑时按坐骑体型。
- **边界：`trap_vines_base_fx` 的 `KillFX` 只在主模拟副本存在。** 该方法在工厂 SetPristine 之后的服务端段赋值；客户端靠网络移除。直接 `SpawnPrefab` 后忘了调 `KillFX` 会得到一个永远 `idle` 的底座（[KillFX 赋值](../../../raw/current/scripts/prefabs/trap_vines.lua#L300-L303)）。
- **边界：藤蔓是玩法载体，别当纯 FX 复用。** 本体有 combat（14）＋planardamage（20）＋减速 debuff，且**持久存档**；300 秒 decay 是唯一自然消失路径，投石机撒新环时会主动清 1 格内旧藤蔓（[清旧](../../../raw/current/scripts/prefabs/winona_catapult_projectile.lua#L273-L275)）。
- **边界：底座收尾时会先脱离宿主。** `DespawnTrap` 把底座 `SetParent(nil)` 挪到藤蔓坐标再播 `pst`——退场期间底座不再跟随藤蔓（也不再有藤蔓可跟随），这是有意设计（[脱挂](../../../raw/current/scripts/prefabs/trap_vines.lua#L148-L152)）。
- **边界：`shadow_pillar_fx` bank 复用不等于实体复用。** 本页底座与影柱 `shadow_pillar_base_fx` 是两个注册、两套清理；需要涟漪／缩放行为请看影柱页，需要藤蔓联动行为看本页。[[reference/prefabs/shadow_pillar|暗影囚牢·影柱法术体系]]

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/trap_fumarole.lua`、`prefabs/trap_vines.lua`、`prefabs/trap_fumarole_util.lua`、`components/trapfumaroleburning.lua`，按需读取 `prefabs/winona_catapult_projectile.lua` 的撒环／落地段、`stategraphs/SGwinona_catapult.lua` 攻击状态弹体段、`componentutil.lua` GetCombatFxSize、`fx.lua` 两个链条 FX 注册、`tuning.lua` 两组调值。对 2 个注册名以完整名在 `raw/current/scripts` 全域反查：`trap_fumarole_burn_fx` 命中注册（trap_fumarole.lua:720）、依赖（:16）、资产（:22）、bank/build（:698/:699）、生成（trapfumaroleburning.lua:28）——**生成点唯一**；`trap_vines_base_fx` 命中注册（trap_vines.lua:309）、依赖（:13）、生成（:179 OnInit、:200 OnLoad）——两生成点都在宿主生命周期内。`trap_vines` 的生成点为 winona_catapult_projectile.lua:276（唯一字面 SpawnPrefab）。`GetCombatFxSize` 的 `sz` 取值为 tiny／small／med／large，本链映射到 burn_hit_ 三档已核对。未运行游戏。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.TRAP_FUMAROLE`＝"煤炭"（锚点见"快速判断"）；"TRAP_VINES" 在该 po 全文 0 命中，`TRAP_FUMAROLE_BURN_FX`／`TRAP_VINES_BASE_FX` 无 NAMES 键，标题使用效果描述。
- 分档说明：`standard`——2 注册双链，各链独立小节；未选 complex，因两条链均为单层生成／清理，无多变体状态机。
- 未覆盖：动画包帧数与观感、喷气孔陷阱温度／烤制数值体系全貌、投石机弹药与技能树门禁全链、`fumarole_ember`／`fumarole_cook_fx` 工厂细节（仅记注册位置）、`DoSyncAnim` 客户端石头群重建逐帧行为、宿主不经组件路径消失时的引擎级联实测；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/shadow_pillar|暗影囚牢·影柱法术体系]]（底座复用其 `shadow_pillar_fx` bank／build；对照 `shadow_pillar_base_fx` 的涟漪与缩放差异）
- 返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]
