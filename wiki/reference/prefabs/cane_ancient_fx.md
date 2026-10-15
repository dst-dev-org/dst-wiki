---
title: "远古手杖影子拖尾（cane_ancient_fx）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "远古手杖的影子动画拖尾：通用及三种编号注册、随机缩放／翻转同步、联网代理与本地动画的分别回收。"
aliases: ["cane_ancient_fx", "cane_ancient_fx1", "cane_ancient_fx2", "cane_ancient_fx3", "远古手杖影子拖尾", "手杖影子残影", "cane_ancient"]
tags: [prefab, fx, lifecycle, server-client, network-sync]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["prefabs/cane_ancient_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 远古手杖影子拖尾（cane_ancient_fx）

## 快速判断

- 职责与适用：[[reference/prefabs/cane|步行手杖]] 远古皮肤的影子动画拖尾；不是跟随手部的常驻粒子。[皮肤模式](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2902-L2913)、[动画创建](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L10-L30)
- 优先使用：查“手杖影子残影”、编号变体和代理回收选本页；选点／生成节奏查 [[reference/prefabs/cane#跟随-fx-与拖尾任务如何选择和停止]]。
- 重要副作用：每次生成包含联网代理和非联网动画；代理设定 0.5 秒后变更网络目标、1.5 秒移除，本地动画监听 animover 自行移除。[代理调度](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L74-L82)、[动画退出](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L10-L30)
- 修改边界：本工厂未挂组件／SG／Brain；非专服响应同步事件生成本地动画，主模拟负责随机值和代理任务。[构造分界](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L48-L85)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cane_ancient_fx` | MakeShadowFX(name,nil,prefs)；构造时随机 variation，SetPrefabName 到对应编号名 | 不代表再 Spawn 一个编号 Prefab；共用同一代理工厂。[工厂](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L48-L85)、[注册](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L90-L101) |
| `cane_ancient_fx1`／`cane_ancient_fx2`／`cane_ancient_fx3` | MakeShadowFX(name,i)，variation 固定为 1／2／3 | 对应 shad1／shad2／shad3；仍随机缩放与翻转。[动画选择](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L32-L42)、[循环注册](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L92-L97) |

皮肤名称键 `STRINGS.SKIN_NAMES.cane_ancient` 为“远古手杖”；本次未发现对应 FX 注册名的直接 NAMES 翻译条目；标题中文部分是已核验的效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L394135-L394137)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源：cane_ancient | fx_prefab 第 2 项，由主体周期任务选合法落点后 SpawnPrefab | 卸装取消后续生成；已生成代理按自身任务退出 | [[reference/prefabs/cane#跟随-fx-与拖尾任务如何选择和停止]]；[皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2902-L2913)、[生成](../../../raw/current/scripts/prefabskin.lua#L3106-L3136)、[停止任务](../../../raw/current/scripts/prefabskin.lua#L3177-L3186) |
| net_smallbyte／randdirty | _rand=1～62 编码缩放与翻转，非专服监听并延迟创建本地动画 | _complete 避免重复；variation 随注册选择，不在 _rand 中编码 | 独立机制 Wiki 待建；[消费](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L32-L42)、[声明与设置](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L58-L82) |
| 本地动画实体 | CreateEntity，无 Network；SetFromProxy 取得代理变换，alpha=0.5，animover Remove | 非专服；不是另一个有注册名的 Prefab | [本地工厂](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L10-L30) |
| 资源／标签 | cane_shadow_fx 动画 bank／build；联网代理带 shadowtrail 标签 | 标签被主体附近去重查询消费，非伤害／照明配置 | [资源](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L1-L8)、[动画](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L23-L29)、[标签](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L55-L56)、[消费者](../../../raw/current/scripts/prefabskin.lua#L3106-L3131) |

## 典型接入

主体拖尾任务生成联网代理，代理按自身计时 Remove，本地动画另在 animover 时 Remove；两者分别管理存在期。本族使用 AnimState 动画，未创建 VFXEffect，无 SetMaxNumParticles／SetSortOffset／SetKillOnEntityDeath 调用，不套用粒子工厂参数。[主体生成](../../../raw/current/scripts/prefabskin.lua#L3134-L3136)、[本地动画](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L10-L30)、[代理回收](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L78-L82)

### 影子拖尾与双层回收

OnRandDirty 在 _complete 已真或 _rand<=0 时返回，不创建本地动画；通过门禁后，_rand>31 时翻转，1～31／32～62 各用 31 档将缩放从 1 映射至 1.8，再以零延迟任务播放 shad..variation 并置 _complete。新建本地动画还设置 SetCanSleep(false)，与代理任务分开；仅凭这些分支不能断言晚加入／离屏时的最终表现。[缩放常量](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L6-L8)、[同步消费](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L32-L42)、[禁休眠](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L10-L16)

代理主模拟 persists=false，0.5 秒后调用 Network:SetClassifiedTarget(inst)，1.5 秒后 Remove；前者只是网络目标变更，不能写成当场删除。可见动画实体有自己的 animover→Remove，不能把代理 1.5 秒直接当作资源动画长度。[网络目标](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L44-L46)、[代理任务](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L78-L82)、[本地退出](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L23-L29)

## 常见陷阱与限制

- **误区：一个文件只注册一个对象**：本文件实际返回通用入口加三个编号变体，均由本页登记，避免另造三篇重复页。[返回列表](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L90-L101)
- **误区：停止手杖拖尾任务会删除所有已落地影子**：卸装只取消后续任务；本族代理及本地动画各有退出路径。[主体停止](../../../raw/current/scripts/prefabskin.lua#L3177-L3186)、[代理退出](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L78-L82)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整工厂及四个注册、皮肤模式／译名、主体选点与停止、同步和本地动画回收已核验，主体已补链；未展开引擎网络实现。
- 游戏运行：未运行；未检查动画资源内容、断线／晚加入表现或网络目标切换后的实际可见性。无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
