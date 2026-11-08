---
title: "魔术师高礼帽影子特效（tophat_shadow_fx）"
created: "2026-10-29"
updated: "2026-11-08"
type: prefab
summary: "魔术师高礼帽的影子烟环三件套：礼帽常驻影子（网络壳加前后两层本地动画）、取物时的旋涡与悬浮微粒，由礼帽转换逻辑与玩家使用状态分别生成。"
aliases: ["tophat_shadow_fx", "魔术师高礼帽特效", "魔术师礼帽影子", "tophat_swirl_fx", "tophat_using_shadow_fx", "魔术师高礼帽"]
tags: [prefab, fx, server-client, network-sync]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/tophat_shadow_fx.lua", "prefabs/hats.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "recipes.lua", "prefablist.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 魔术师高礼帽影子特效（tophat_shadow_fx）

## 快速判断

- 职责与适用：魔术师高礼帽（由礼帽 tophat 转换而来）的影子表现三件套：常驻影子 `tophat_shadow_fx`、取物动画中的旋涡 `tophat_swirl_fx` 与悬浮微粒 `tophat_using_shadow_fx`。常驻影子是“无 AnimState 的网络壳＋每端本地两层动画”结构，非纯客户端粒子，也不是皮肤特效。[常驻影子工厂](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L139-L165)
- 优先使用：查“魔术师高礼帽／礼帽影子特效”选本页；礼帽实体与转换逻辑见 [[reference/components/magiciantool]]，玩家侧取物流程见 [[reference/components/magician]]。
- 重要副作用：常驻影子生命周期完全由礼帽侧驱动——装备挂人到 swap_hat 符号、脱下挂回礼帽本体、进 Limbo 移除；旋涡与微粒由服务器使用状态生成、状态退出移除，客户端预测状态不生成。[礼帽驱动](../../../raw/current/scripts/prefabs/hats.lua#L936-L985)、[状态生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22945-L22954)
- 修改边界：常驻影子的两层动画与粒子实体都是本地非网络实体（专服不建）；网络壳在主模拟仅 persists=false。前后动画层用 CLASSIFIED 标签，散出粒子仍用 FX 标签——源码注明 DST 中被 parent 的 FX 仍会触发鼠标悬停。[本地子实体](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L104-L118)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `tophat_shadow_fx` | shadow_fn：Transform＋Follower＋Network＋FX／NOCLICK → 非专服建前后两个本地动画子实体（同相位随机帧 0-31）→ SetPristine → 客户端返回 → 主模拟 persists=false | 网络壳无 AnimState；可见部分是子实体。[工厂](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L139-L165) |
| `tophat_swirl_fx` | swirl_fn：四朝向 Transform＋AnimState 循环播 swirl_front → SetPristine → 客户端返回 → 主模拟安装 AttachToTopHatUser | 背层实例由 attachfn 播 swirl_back 循环；骑乘时改六朝向。[旋涡](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L182-L211)、[挂载](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L169-L180) |
| `tophat_using_shadow_fx` | using_shadow_fn：同旋涡骨架，循环播 using_particles＋半透明 | attachfn 跟随 swap_fx_particles_using_tophat 符号。[微粒](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L223-L253) |

三者同文件一次返回注册，共用 anim/tophat_fx.zip；prefablist 枚举仅列 tophat_shadow_fx。[注册](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L257-L259)、[枚举](../../../raw/current/scripts/prefablist.lua#L1285)

`STRINGS.NAMES.TOPHAT_MAGICIAN` 官方译名“魔术师高礼帽”；未发现三个 FX 注册名的 NAMES 翻译条目，标题中“影子特效”为已核验效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L349850-L349852)

## 关联机制导航

### 常驻影子（tophat_shadow_fx）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源：礼帽转换 | tophat_magician 配方合成或读档 data.magician 时 ConvertToMagician：挂 magiciantool 并把 equippable 换绑到带 FX 的 onequip／onunequip；普通礼帽无 FX | 转换只发生在主模拟（OnLoad／OnPreBuilt 挂接在 fns.top 主模拟段） | [配方](../../../raw/current/scripts/recipes.lua#L294)、[读档](../../../raw/current/scripts/prefabs/hats.lua#L1019-L1035)、[转换](../../../raw/current/scripts/prefabs/hats.lua#L987-L1017)、[主模拟挂接](../../../raw/current/scripts/prefabs/hats.lua#L1051-L1068) |
| 常驻影子生命周期 | 装备 → 挂 owner 跟随 swap_hat(0,-100,0)；脱下 → 未进 Limbo 则挂回礼帽本体原点；enterlimbo/exitlimbo 补移除或补挂；转换完成时按当前状态立即补挂 | 只管理 inst.fx 一个实例 | [挂人](../../../raw/current/scripts/prefabs/hats.lua#L953-L959)、[挂地](../../../raw/current/scripts/prefabs/hats.lua#L943-L951)、[事件](../../../raw/current/scripts/prefabs/hats.lua#L975-L985) |
| 本地动画子实体 | shadow_createfx 建前后两层：Hide 对侧、FinalOffset ±1、animover 重播 "hatfx" 形成循环，按帧表（前层 13/15/17/27/29/31，背层 9/11/13/23/25/27）调度粒子 | 非网络本地实体；可见性沿父链检查，任一层不可见即不产粒子 | [建层](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L104-L137)、[前层帧表](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L73-L83)、[背层帧表](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L85-L95) |
| 粒子池 | 粒子实体 animover 后 Hide 回池复用；出生时 Follower 跟随层实体 swap_front／swap_back 符号一帧后 StopFollowing 停在原地；层实体移除时清池并置 invalid，其后粒子 animover 直接 Remove | 池随层实体创建；客户端粒子 AddClientSleepable | [粒子生成](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L21-L65)、[回池](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L8-L15)、[清池](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L97-L102) |

### 取物旋涡与微粒（tophat_swirl_fx 与 tophat_using_shadow_fx）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 取物状态消费 | 服务器 using_tophat 状态生成 1 个 using 微粒＋前背 2 个旋涡并 AttachToTopHatUser；onexit 移除三个；客户端预测同名状态不生成 | 需要玩家 components.magician；FX 实体联网复制 | [生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22945-L22954)、[移除](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23010-L23029)、[客户端状态](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L6781-L6816) |

### 共享资源

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 资源 | anim/tophat_fx.zip：hatfx／particle_top／mid／btm／swirl_front／swirl_back／using_particles | 声明与使用已核验，包内容未解包 | [资源声明](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L1-L4) |

## 典型接入

### 复用常驻影子或旋涡

- 前提与选择：常驻影子依赖礼帽侧 inst.fx 单实例协议（equip／unequip／limbo 三事件），模组复用要么复刻该协议，要么 SpawnPrefab 后自行管理挂载；旋涡与微粒只要调用 `AttachToTopHatUser(owner[, front])` 即可完成挂载与跟随，不依赖礼帽。
- 生效与撤销：常驻影子的可见性沿父链检查——挂在不可见实体（如 Limbo）下时自动不产粒子；旋涡挂在骑乘玩家身上会切六朝向。[可见性](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L22-L28)、[骑乘](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L170-L172)
- 原版依据：转换入口与事件绑定见上文导航；取物流程的状态与清理见 SGwilson 行号。

## 常见陷阱与限制

- **误区：这是礼帽皮肤的特效**：FX 不走皮肤系统——普通礼帽没有任何影子；只有 tophat_magician 配方产物或读档标记为 magician 的礼帽经 ConvertToMagician 后才绑 FX。[转换](../../../raw/current/scripts/prefabs/hats.lua#L987-L1017)
- **误区：tophat_shadow_fx 自身可见**：网络壳没有 AnimState，直接 SpawnPrefab 放地上只会看到每端本地创建的两层 hatfx 动画（专服上完全不可见）；粒子跟随的是层实体动画内的符号。[工厂](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L139-L154)
- **边界：hatfx 循环靠 animover 重播**：PlayAnimation 未传循环参数，播完由 animover 监听重播；初始帧随机 0-31 且前后两层同帧，改动画长度时帧表偏移逻辑要同步核验。[重播](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L73-L83)、[随机帧](../../../raw/current/scripts/prefabs/tophat_shadow_fx.lua#L150-L153)
- **边界：礼帽实体被直接移除时无 FX 清理监听**：礼帽侧只在 equip／unequip／limbo 路径管理 inst.fx；equippable／inventoryitem 的 OnRemoveFromEntity 不触发 unequip，礼帽耐久耗尽走 DepletedFn=inst.Remove 直接移除时 FX 是否经其他路径清理未核验。[清理路径](../../../raw/current/scripts/prefabs/hats.lua#L936-L941)、[耗尽移除](../../../raw/current/scripts/prefabs/hats.lua#L1057-L1060)
- **边界：旋涡／微粒只由服务器状态图生成**：客户端预测状态不建实体，依赖网络复制显示；把生成逻辑照抄进客户端状态图会造成双份。[客户端状态](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L6781-L6816)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；完整 259 行工厂与三个注册名（文件末尾 return 核实）、礼帽转换与 inst.fx 生命周期、SGwilson 生成与 onexit 清理、SGwilson_client 预测状态对照、配方入口、译名已核验；本地子实体与粒子池逐函数审阅。未覆盖：动画包内容未解包、取物全程实测、礼帽直接移除时 FX 残留与否、prefablist 枚举缺 swirl／using 的用途。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-04）。
