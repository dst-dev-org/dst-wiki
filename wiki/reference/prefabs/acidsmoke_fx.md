---
title: "酸雨烟雾特效（acidsmoke_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "acidinfusible 宿主在酸雨中被侵蚀时挂在身上的烟雾子实体：等级动画 idle_1~idle_4、主模拟 animover 链重播、按剩余动画长度延迟移除。"
aliases: ["acidsmoke_fx", "酸雨烟雾", "酸液烟雾", "acid smoke"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/acidsmoke_fx.lua", "components/acidinfusible.lua", "prefabs/hutch.lua", "prefabs/cave_vent_mite.lua", "prefabs/mushtree.lua", "prefabs/rocky.lua", "prefabs/cave.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 酸雨烟雾特效（acidsmoke_fx）

## 快速判断

- 职责与适用：挂在受酸雨侵蚀宿主身上的烟雾视觉子实体。工厂只有 Transform／AnimState／Network 与 FX 标签，无任何玩法组件；烟雾的生成、等级与移除全由 [[reference/components/acidinfusible|acidinfusible]] 组件驱动。[工厂](../../../raw/current/scripts/prefabs/acidsmoke_fx.lua#L21-L54)
- 优先使用：要"实体在酸雨中被侵蚀冒烟"就给宿主加 acidinfusible 并 `SetFXLevel(1~4)`（[哈奇示例](../../../raw/current/scripts/prefabs/hutch.lua#L443-L445)）；本页查该特效自身的动画与生命周期。
- 重要副作用：无玩法副作用。KillFX 延迟路径要读 `GetCurrentAnimationLength`，因此工厂出生即 SetFrame 到最后一帧，保证动画长度可查（[注释](../../../raw/current/scripts/prefabs/acidsmoke_fx.lua#L33)、[KillFX](../../../raw/current/scripts/components/acidinfusible.lua#L204-L215)）。
- 修改边界：客户端工厂初始化 `idle_1` 并跳到最后一帧；重播调度与等级字段只由主模拟修改，不能据此断言客户端只播放初始动画，实际网络表现未运行核验。宿主传 nil 等级则跳过生成（[mushtree](../../../raw/current/scripts/prefabs/mushtree.lua#L745-L746)）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `acidsmoke_fx` | fn：Transform＋AnimState＋Network＋FX 标签 → bank/build `acidsmoke`、播 `idle_1`、SetFrame 最后一帧（为 acidinfusible kill 案例提供动画长度）、SetFinalOffset(3) → SetPristine → 主模拟：暴露 SetLevel／OnAnimOver／PlayAnim，persists=false，注册 animover→5~8 秒后重播链并先调度一次 | 单一注册。`PlayAnimation` 未显式传循环参数；主模拟的持续冒烟是事件链（animover → DoTaskInTime(5~8s) → PlayAnim），客户端无此链。[注册](../../../raw/current/scripts/prefabs/acidsmoke_fx.lua#L56)、[工厂](../../../raw/current/scripts/prefabs/acidsmoke_fx.lua#L21-L54)、[重播链](../../../raw/current/scripts/prefabs/acidsmoke_fx.lua#L13-L15) |

无 NAMES.ACIDSMOKE 条目；官方仅 NAMES.ACIDRAIN "酸雨"（[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L337894-L337897)）。标题中文为已核验效果描述（acidsmoke_fx），非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：acidinfusible | 唯一生成者：OnInfuse 时 `SpawnChild("acidsmoke_fx")` 并 `SetLevel(fxlevel)`；OnUninfuse／OnEntitySleep 走 KillFX | fxlevel 非 nil 且宿主未休眠才生成；OnEntityWake 若已侵蚀会复挂；生成的是网络子实体 | [[reference/components/acidinfusible#关键接口]]；[SpawnFX](../../../raw/current/scripts/components/acidinfusible.lua#L194-L202)、[OnInfuse 条件](../../../raw/current/scripts/components/acidinfusible.lua#L155-L157)、[Wake/Sleep](../../../raw/current/scripts/components/acidinfusible.lua#L220-L228) |
| API／数据：SetLevel(level) | 主模拟把重播动画改成 `idle_..level`；宿主经 `AcidInfusible:SetFXLevel` 转发 | 只影响主模拟之后的重播；idle_2~4 是否都在 anim 资源内未解包核验 | [SetLevel](../../../raw/current/scripts/prefabs/acidsmoke_fx.lua#L17-L19)、[转发](../../../raw/current/scripts/components/acidinfusible.lua#L124-L130) |
| 宿主与等级 | 哈奇/猴/蜘蛛/触手/蠕虫 1、兔人/骑士/战车/兔王 2、蝙蝠/蝙蝠王/主教/树精/岩石虾 3、洞穴螨动态 2/4 | mushtree 系调 `SetFXLevel()`（nil）→ 永不生成；rocky 有 `canacidrain and 3 or 0` 动态档，注释"0 means disabled" | 等级示例：[1](../../../raw/current/scripts/prefabs/hutch.lua#L445)、[4](../../../raw/current/scripts/prefabs/cave_vent_mite.lua#L206)、[nil](../../../raw/current/scripts/prefabs/mushtree.lua#L746)、[0/3](../../../raw/current/scripts/prefabs/rocky.lua#L79)；完整宿主见 [[reference/components/acidinfusible#典型接入]] |
| 声明：cave.lua 依赖 | `acidsmoke_fx` 列于洞穴 Prefab 依赖清单，供洞块注册，不是生成证据 | 洞块宿主（如洞穴螨）生成时经 acidinfusible | [清单](../../../raw/current/scripts/prefabs/cave.lua#L151) |
| 资源 | anim/acidsmoke.zip；bank/build `acidsmoke` | 仅声明与调用已核验，帧数与 idle_0 是否存在未解包 | [assets](../../../raw/current/scripts/prefabs/acidsmoke_fx.lua#L1-L4)、[bank](../../../raw/current/scripts/prefabs/acidsmoke_fx.lua#L30-L31) |

## 典型接入

### 让实体在酸雨中冒烟

- 前提与选择：不要直接 SpawnPrefab 本特效——生成与 KillFX 由 acidinfusible 统一管理，直接生成的实例没人负责清理。
- 生效与撤销：酸雨开始且满足组件雨免疫配置 → OnInfuse 生成子实体；雨停走 KillFX 的延迟路径（剩余动画长度＋1 帧），宿主休眠走 KillFX(true) 立即移除。
- 原版依据：[OnInfuse](../../../raw/current/scripts/components/acidinfusible.lua#L140-L162)、[KillFX](../../../raw/current/scripts/components/acidinfusible.lua#L204-L215)。

```lua
inst:AddComponent("acidinfusible")
inst.components.acidinfusible:SetMultipliers(TUNING.ACID_INFUSION_MULT.STRONGER)
inst.components.acidinfusible:SetFXLevel(1)  -- 决定烟雾档位，运行中可改
```

- 调整与验证：宿主既有用法覆盖 1~4 档；SetFXLevel 运行中切换后，主模拟下一次重播即换 `idle_..level`。

## 常见陷阱与限制

- **边界：本地回调与网络动画状态需区分**：SetLevel 只改主模拟实例的 `_anim`，主模拟随后用 AnimState 播放对应动画；客户端未注册 Lua 重播链不等于收不到动画状态，实际网络表现未运行核验。[主模拟块](../../../raw/current/scripts/prefabs/acidsmoke_fx.lua#L42-L51)
- **边界：循环未显式传参**：`PlayAnimation` 未传循环参数；主模拟的重复冒烟由 animover→5~8 秒→重播事件链显式实现，客户端无该链，实际重复表现未核验。[重播链](../../../raw/current/scripts/prefabs/acidsmoke_fx.lua#L13-L15)、[调度](../../../raw/current/scripts/prefabs/acidsmoke_fx.lua#L50-L51)
- **边界：nil／0 等级**：fxlevel 为 nil 时 OnInfuse 跳过 SpawnFX；rocky 的 0 档会拼出 `idle_0` 动画名，该动画是否为空未核验。[条件](../../../raw/current/scripts/components/acidinfusible.lua#L155-L157)、[rocky](../../../raw/current/scripts/prefabs/rocky.lua#L79)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；56 行工厂全文、acidinfusible 的 SpawnFX／KillFX／OnEntityWake／OnEntitySleep 链、宿主等级抽样（hutch、cave_vent_mite、mushtree、rocky、bat）、cave 依赖声明、ACIDRAIN 译名已核验；anim 资源内实际动画帧数、客户端重播与等级表现、acidinfusible 其余宿主逐个复核未覆盖。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
