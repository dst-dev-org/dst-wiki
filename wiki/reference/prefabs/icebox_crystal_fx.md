---
title: "水晶冰箱持续霜气特效（icebox_crystal_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "水晶冰箱皮肤的持续霜气动画实体：开箱时在冰箱处生成，关箱或移除宿主时播退场动画再移除。"
aliases: ["icebox_crystal_fx", "水晶冰箱霜气", "水晶冰箱冷气", "水晶冰箱特效", "icebox_crystal"]
tags: [prefab, fx, lifecycle, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/icebox_crystal_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "components/container.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 水晶冰箱持续霜气特效（icebox_crystal_fx）

## 快速判断

- 职责与适用：[[reference/prefabs/icebox|冰箱]] 水晶皮肤的持续霜气动画实体：纯 AnimState 联网实体，无组件、声音或光照；宿主开箱时在冰箱位置生成，实体驻留至关箱或宿主移除。工厂播 pre 后排队 loop 段，未显式传循环参数，实际是否重复未核验。[工厂](../../../raw/current/scripts/prefabs/icebox_crystal_fx.lua#L19-L46)
- 优先使用：查“水晶冰箱霜气／冷气特效”选本页；皮肤本体与开箱节流查 [[reference/prefabs/icebox#皮肤差异与开箱特效]]。同族 [[reference/prefabs/icebox_kitchen_fx]]／[[reference/prefabs/icebox_porcelain_fx]]／[[reference/prefabs/icebox_victorian_frost_fx]] 工厂同构，仅 bank 与皮肤资源不同。
- 重要副作用：生命周期由宿主驱动：首次 onopen 生成，onclose／onremove 时宿主调用本工厂安装的 Kill——等当前动画段播完、播 pst 退场动画后才移除，不是立即消失。[宿主调用](../../../raw/current/scripts/prefabskin.lua#L3697-L3702)
- 修改边界：主模拟安装 Kill 与 loop 段排队；客户端实体靠网络同步动画，不装 Kill；persists=false 不入存档，SetFinalOffset(1) 压盖显示。不创建 Light、SoundEmitter 或粒子发射器。[主模拟分界](../../../raw/current/scripts/prefabs/icebox_crystal_fx.lua#L35-L43)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `icebox_crystal_fx` | fn → Transform／AnimState／Network＋FX 标签 → SetPristine → 主模拟 PushAnimation("loop")、persists=false、安装 Kill | 单一注册；bank icebox_crystal（皮肤 dynamic 动画包）、build ice_box、动画 pre 后排队 loop 段（未显式传循环参数）、SetFinalOffset(1)。[构造](../../../raw/current/scripts/prefabs/icebox_crystal_fx.lua#L19-L46)、[注册](../../../raw/current/scripts/prefabs/icebox_crystal_fx.lua#L48) |

皮肤键 `STRINGS.SKIN_NAMES.icebox_crystal` 为“水晶冰箱”（官方译名）；未发现 FX 注册名的直接 NAMES 翻译条目，标题中“持续霜气特效”是已核验效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L398129-L398132)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源：icebox_crystal 皮肤 | fx_prefab 表存入 SKIN_FX_PREFAB，icebox_init_fn 读出第 1 项为 frost_fx 并监听 onopen／onclose／onremove | 主模拟；placer 分支只设皮肤外观、不接 FX | [[reference/prefabs/icebox#皮肤差异与开箱特效]]；[皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L6968-L6978)、[映射保存](../../../raw/current/scripts/prefabskin.lua#L4094-L4096)、[初始化](../../../raw/current/scripts/prefabskin.lua#L3704-L3731) |
| 宿主事件：onopen／onclose | 容器组件开／关时推送事件，宿主监听后生成或 Kill 本实体 | 监听装在冰箱宿主上，本实体不监听事件 | [[reference/components/container#关键接口]]；[开](../../../raw/current/scripts/components/container.lua#L656)、[关](../../../raw/current/scripts/components/container.lua#L719)、[宿主监听](../../../raw/current/scripts/prefabskin.lua#L3719-L3725) |
| 宿主清理协议：Kill | icebox_closed 调用本工厂安装的 inst.Kill：等当前动画播完→播 pst→延时 pst 长度＋2 帧后 Remove；_killtask 防重复调度 | 仅主模拟安装；同一实体重复 Kill 被挡 | [Kill](../../../raw/current/scripts/prefabs/icebox_crystal_fx.lua#L13-L17)、[安装](../../../raw/current/scripts/prefabs/icebox_crystal_fx.lua#L43)、[宿主](../../../raw/current/scripts/prefabskin.lua#L3697-L3702) |
| 皮肤符号覆盖 | 生成时对 cold_air／blink_dot 两符号 OverrideItemSkinSymbol 指向皮肤 build，基 build 仍为 ice_box | 生成时一次设置；符号内含未解包 | [覆盖](../../../raw/current/scripts/prefabskin.lua#L3687-L3694) |
| 资源 | anim/ice_box.zip＋DYNAMIC_ANIM／PKGREF 的 icebox_crystal 动态包 | 声明与使用已核验，包内容未解包 | [资源声明](../../../raw/current/scripts/prefabs/icebox_crystal_fx.lua#L1-L6) |

## 典型接入

### 复用霜气或调整退场

原版仅经皮肤系统生成：皮肤 fx_prefab 第 1 项＋icebox_init_fn 协议——onopen 时宿主发现 _frostfx 为空则 SpawnPrefab 并 SetPosition（无父级跟随），onclose／onremove 调 Kill。模组复用要么沿用该协议，要么自写等价生成与清理；需要 pst 退场就调用 Kill（本工厂安装的方法），直接 Remove 则立即消失、无退场动画。[生成与覆盖](../../../raw/current/scripts/prefabskin.lua#L3687-L3694)、[清理](../../../raw/current/scripts/prefabskin.lua#L3697-L3702)

## 常见陷阱与限制

- **误区：Kill 立即删除实体**：Kill 只安排任务——先等当前动画段播完（长度减已播时间），切 pst，再等 pst 长度＋2 帧才 Remove；期间实体继续显示。[Kill](../../../raw/current/scripts/prefabs/icebox_crystal_fx.lua#L13-L17)
- **误区：Kill 是实体通用方法**：宿主调用的 Kill 是本工厂 `inst.Kill = Kill` 安装的自定义方法；对没有安装 Kill 的实体照抄这段宿主清理协议会调用 nil 报错。[安装](../../../raw/current/scripts/prefabs/icebox_crystal_fx.lua#L43)、[宿主调用](../../../raw/current/scripts/prefabskin.lua#L3698-L3700)
- **边界：每个开箱周期都是新实体**：onclose 即 Kill 并清宿主 _frostfx 引用，重开箱生成新实例而非复用旧实体；换肤 clear_fn 同样经 icebox_closed 清理。[宿主](../../../raw/current/scripts/prefabskin.lua#L3697-L3702)、[清皮肤](../../../raw/current/scripts/prefabskin.lua#L3732-L3740)
- **边界：无跟随定位**：位置只在生成时 SetPosition 一次，工厂与宿主都没有 SetParent／Follower；宿主被代码移位后霜气不跟随。

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整 48 行工厂、五种皮肤定义与 fx_prefab 映射、SKIN_FX_PREFAB 保存与 icebox_init_fn／opened／closed／clear_fn、container 开关事件推送点、prefablist 枚举与 gamelogic 加载、皮肤译名检索已核验；同族三页（kitchen／porcelain／victorian）工厂对照同构。未覆盖：动画包内容未解包、最终视觉、多次开关／换肤时序实测。
- 游戏运行：未运行。
- 结构检查：validate_page 与 validate_source_links --strict 通过（实际结果见本批交付报告）。

返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
