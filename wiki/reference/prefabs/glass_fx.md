---
title: 玻璃化、灰烬消散与睡眠炸弹爆裂（glass_fx 等三族）
created: 2026-11-01
updated: 2026-11-08
type: prefab
summary: "三条互不相关的单注册表现 FX：glass_fx 沙晶点燃后的玻璃化转场起爆、erode_ash 化石尖刺到期消散的灰烬团、sleepbomb_burst 睡眠炸弹命中爆裂；同页仅为本批零散 FX 归档，无共享资源或机制。"
aliases: ["glass_fx", "erode_ash", "sleepbomb_burst", "玻璃化特效", "灰烬消散", "睡眠炸弹爆裂"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["fx.lua", "prefabs/sand_spike.lua", "stategraphs/SGantlion_angry.lua", "prefabs/fossil_spike.lua", "prefabs/fossil_spike2.lua", "prefabs/moonspider_spike.lua", "prefabs/stalker.lua", "prefabs/sleepbomb.lua", "prefabs/sleepcloud.lua", "recipes.lua", "simutil.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 玻璃化、灰烬消散与睡眠炸弹爆裂（glass_fx 等三族）

## 快速判断

- 职责与适用：本批三个各只有一条消费链的单注册表现 FX——`glass_fx` 沙晶尖刺/沙块被点燃后按 22 帧任务落点的起爆尘（借用 `mining_fx` bank＋`mining_ice_fx` build 资源；与换物各自调度，相对时序未核验）、`erode_ash` 化石尖刺到期消散时落点的灰烬团、`sleepbomb_burst` 睡眠炸弹命中爆裂动画。三者无共享资源、互不调用，同页只是同批零散 FX 归档（各自单开一页内容过薄），按小节分立。均为纯表现，玩法作用在生成者；公共协议见 [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂·关键流程]]。标题为已核验效果描述，非官方译名。
- 优先使用：查"沙晶变玻璃的起爆、化石尖刺消散的灰烬、睡眠炸弹落地爆开"读本页对应小节；尖刺族自身的发射/破坏链见 [[reference/mechanisms/spike-launcher]]；新增条目走 [[reference/mechanisms/generic-fx-factory#典型接入|通用 FX 工厂·典型接入]]。
- 重要副作用：三者的声音都来自条目 `sound` 字段，由工厂在各端本地实体播放：`sand_to_glass`／`dust_blowaway`／`infection_burst`。
- 修改边界：全部主模拟生成→网络代理→各端本地表现，专服无表现；fx 实体自身无组件、无玩法作用；触发条件归各自宿主，不在本页展开。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `glass_fx` | 通用工厂：bank `mining_fx`、build `mining_ice_fx`、anim `anim`，sound `sand_to_glass` | 借用采矿碎屑资源；无 fn；[条目](../../../raw/current/scripts/fx.lua#L1694-L1698)、资源族 [[reference/prefabs/mining_fx]] |
| `erode_ash` | bank/build `erode_ash`，anim `idle`，sound `dust_blowaway` | 独立资源；无 fn；[条目](../../../raw/current/scripts/fx.lua#L1701-L1705) |
| `sleepbomb_burst` | bank/build `sleepbomb`，anim `used`，sound `infection_burst` | 与弹体同资源；无 fn；[条目](../../../raw/current/scripts/fx.lua#L1708-L1712) |

## 消费者与触发窗口

### glass_fx：沙晶玻璃化转场

- 生成窗口：`sand_spike.lua` `OnIgnite`（尖刺/沙块被点燃时）安排 `GLASS_TIME − 2 帧`（GLASS_TIME=24 帧）后 `PlayGlassFX`，在尖刺自身位置落一枚 glass_fx；同时尖刺加 NOCLICK、无敌并播 `_transform` 动画，动画结束（animover）`ChangeToGlass` 移除尖刺、同点生成 `glassspike_<size>`／`glassblock` 并 Sparkle。[PlayGlassFX](../../../raw/current/scripts/prefabs/sand_spike.lua#L170-L173)、[OnIgnite](../../../raw/current/scripts/prefabs/sand_spike.lua#L175-L188)、[换物](../../../raw/current/scripts/prefabs/sand_spike.lua#L162-L168)、[GLASS_TIME](../../../raw/current/scripts/prefabs/sand_spike.lua#L29-L29)
- 宿主与来源：`sandspike_short/med/tall`、`sandblock`（蚁狮震怒沙攻击在 SGantlion_angry 生成）；glass_fx 与玻璃产物在各宿主 prefabs 依赖声明。[SGantlion_angry 生成](../../../raw/current/scripts/stategraphs/SGantlion_angry.lua#L68-L90)、[依赖声明](../../../raw/current/scripts/prefabs/sand_spike.lua#L326-L331)
- 端与边界：服务端点燃链；落点式不父挂；fx 由点燃后 22 帧的任务落点，换物由 `_transform` 动画的 animover 触发。两者是独立调度，动画包未核验，不能据此断言固定先后或同帧／不同帧；22 帧任务也可能在动画被提前移除或打断时不执行。

### erode_ash：化石尖刺消散

- 生成者 1：`fossilspike`（stalker 圈套尖刺，由 stalker `RestartSpike` 按 STALKER_SNARE_TIME 6±0.5s 排期，自生成默认 5+random() 秒）到期 `KillSpike`：底座播 `base_pst` 后 1s（或底座失效时立即）`OnKill` 落一枚 erode_ash 于尖刺位置，再 0.5s 后尖刺自身 `ErodeAway(inst, 1)` 淡出移除。[KillSpike](../../../raw/current/scripts/prefabs/fossil_spike.lua#L126-L148)、[OnKill](../../../raw/current/scripts/prefabs/fossil_spike.lua#L121-L124)、[stalker 生成与排期](../../../raw/current/scripts/prefabs/stalker.lua#L568-L588)、[时长](../../../raw/current/scripts/tuning.lua#L3379-L3380)
- 生成者 2：`fossilspike2`（stalker 进攻尖刺）`OnImpact`（appear 动画 animover 播 impact 段）后 0.35s `KillSpike`：落一枚 erode_ash，0.5s 后自身 ErodeAway。[KillSpike](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L109-L129)、[计时起点](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L131-L163)、[stalker 生成](../../../raw/current/scripts/prefabs/stalker.lua#L845-L845)
- 声明而未生成：`moonspider_spike` 在依赖里列出 erode_ash，但其 `KillSpike` 播 `spike_pst` 后直接 Remove，不生成该 fx。[依赖](../../../raw/current/scripts/prefabs/moonspider_spike.lua#L8-L8)、[KillSpike](../../../raw/current/scripts/prefabs/moonspider_spike.lua#L61-L79)
- 端与边界：服务端；落点式；`ErodeAway` 是尖刺实体的淡出 helper，与 erode_ash fx 是两个东西（见陷阱）。

### sleepbomb_burst：睡眠炸弹命中爆裂

- 生成窗口：`sleepbomb`（可合成投掷武器，[配方](../../../raw/current/scripts/recipes.lua#L484-L484)）complexprojectile 命中回调 `OnHit`：先移除弹体，再在命中点生成 `sleepbomb_burst` 与 `sleepcloud` 各一枚。[OnHit](../../../raw/current/scripts/prefabs/sleepbomb.lua#L16-L20)
- 分工：爆裂动画与 `infection_burst` 声在本 fx；同点的 `sleepcloud` 承担睡眠玩法（[常量](../../../raw/current/scripts/prefabs/sleepcloud.lua#L14-L23)、[DoAreaDrowsy](../../../raw/current/scripts/prefabs/sleepcloud.lua#L242-L242)），不属于本 fx。
- 端与边界：服务端命中结算；落点式。

## 常见陷阱与限制

- **误区：`ErodeAway` 与 `erode_ash` 名字相近**：前者是尖刺实体自身的淡出移除全局 helper（[simutil.lua](../../../raw/current/scripts/simutil.lua#L575-L575)），后者才是本页 fx 条目；化石尖刺消散时两者先后发生（ash 先落点、0.5s 后尖刺淡出），不能把 ErodeAway 说成"生成 erode_ash"。
- **误区：`moonspider_spike` 声明 erode_ash 即月蛛尖刺消散会出灰烬**：依赖声明不证明生成；其消散走 spike_pst＋Remove。全 scripts 检索 erode_ash 生成点仅 fossil_spike／fossil_spike2 两处（范围见验证记录）。
- **边界：glass_fx 与玻璃化产物各自调度**：fx 由点燃后 22 帧的任务落点，尖刺由 `_transform` 动画 animover 换成 glassspike/glassblock；动画包未核验，不能从调度推出固定先后，也不能假设 fx 出现瞬间产物已存在。
- **边界：sleepbomb_burst 不带睡眠**：睡眠作用在同点生成的 sleepcloud；不要给 fx 实体加组件实现玩法。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。三条目逐项核对 bank/build/anim/sound；生成点全量反查 `rg "glass_fx|erode_ash|sleepbomb_burst"` 于 raw/current/scripts 全域（根级、prefabs/、components/、stategraphs/、map/），命中生成点与仅依赖声明处均已列入；sand_spike／fossil_spike／fossil_spike2／moonspider_spike／sleepbomb 消费链读至触发窗口与端。第 5 轮复核将 22 帧任务与 `_transform` animover 换物拆为独立调度，动画包未核验，不主张固定先后或同帧关系。未运行游戏。
- 语言文件：三条目均无 `nameoverride`／`description` 字段，不可检视，`languages/chinese_s.po` 无对应显示名键；标题为非官方效果描述。
- 结构检查：validate_page.py（--format --check-indexes）与 validate_source_links.py --strict 已执行，结果见交付报告。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
