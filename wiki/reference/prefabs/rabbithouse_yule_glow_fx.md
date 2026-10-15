---
title: "倒挂树屋光晕特效（rabbithouse_yule_glow_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "兔屋“倒挂树屋”皮肤的光晕动画实体：打上皮肤时随宿主生成并父子跟随，AnimState 自发光渲染属性，无 Light 组件。"
aliases: ["rabbithouse_yule_glow_fx", "倒挂树屋特效", "兔屋光晕", "yule glow fx"]
tags: [prefab, fx, lifecycle, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/rabbithouse_yule_glow_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 倒挂树屋光晕特效（rabbithouse_yule_glow_fx）

## 快速判断

- 职责与适用：兔屋（官方名"兔屋"）的"倒挂树屋"皮肤（官方译名）附带的持续光晕动画实体：单个联网动画实体随皮肤宿主生成、SetParent 跟随，工厂无 assets、无组件、无本地粒子。与提灯 FX 族的差别：不区分 held／ground、无 KillFX、生成不经事件协议而是皮肤初始化直接拉起。
- 优先使用：查"倒挂树屋／兔屋光晕"选本页；想要"皮肤自带常驻氛围 FX"范式时本页是最短原版参照。照明注意：SetLightOverride(0.7) 是 AnimState 渲染属性，全文件无 AddLight——不是 Light 组件，不能当作照明光源引用。[工厂](../../../raw/current/scripts/prefabs/rabbithouse_yule_glow_fx.lua#L5-L25)
- 重要副作用：PlayAnimation("idle", true) 显式传循环参数 true；SetFinalOffset(1) 压盖显示；persists=false 不入存档。
- 修改边界：工厂体内没有 ismastersim 分界，SetPristine 后两端执行同一函数体；皮肤 init_fn 的引擎调用点在脚本外，双端实际调用情况未核验（参见下方陷阱）。[工厂体](../../../raw/current/scripts/prefabs/rabbithouse_yule_glow_fx.lua#L20-L24)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `rabbithouse_yule_glow_fx` | glowfn → Transform／AnimState／Network → bank rabbithouse_glowfx、build rabbit_house → idle 显式循环 → SetLightOverride(0.7)／SetFinalOffset(1) → FX 标签 → SetPristine → persists=false（无主模拟分界） | 单一注册；无 assets 声明；POPULATING 无处理。[工厂](../../../raw/current/scripts/prefabs/rabbithouse_yule_glow_fx.lua#L5-L25)、[注册](../../../raw/current/scripts/prefabs/rabbithouse_yule_glow_fx.lua#L29) |

皮肤键 `STRINGS.SKIN_NAMES.rabbithouse_yule` 为"倒挂树屋"（官方译名，msgid "Topsy-Turvy Treehouse"）；宿主 `STRINGS.NAMES.RABBITHOUSE` 为"兔屋"；未发现 FX 注册名的直接 NAMES 翻译条目，标题"光晕特效"是按 SetLightOverride 与 idle 动画写的已核验效果描述，非官方译名。[皮肤译名](../../../raw/current/scripts/languages/chinese_s.po#L405779-L405782)、[宿主译名](../../../raw/current/scripts/languages/chinese_s.po#L347804-L347807)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源皮肤：rabbithouse_yule | fx_prefab 单项经 SKIN_FX_PREFAB 保存；皮肤仅此一个附加 Prefab | 非季节门控：皮肤在即生成，名字里的 yule 只是皮肤主题 | [皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L9862-L9872)、[映射保存](../../../raw/current/scripts/prefabskin.lua#L4094-L4096) |
| 宿主初始化：rabbithouse_init_fn | basic_init_fn 设皮肤外观后，非 placer 且非 burnt 时 SpawnPrefab 本 FX、SetParent(宿主)、对 "glow" 符号 OverrideItemSkinSymbol | placer 分支（放置预览）与 burnt 状态不生成；无 ismastersim 分界（与 lantern／icebox init_fn 不同） | [init_fn](../../../raw/current/scripts/prefabskin.lua#L1760-L1773) |
| 宿主清理：rabbithouse_clear_fn | 换肤时 Remove 本实体并清宿主 glow_fx 引用 | 与 basic_clear_fn 一起调用 | [clear_fn](../../../raw/current/scripts/prefabskin.lua#L1774-L1781) |
| 符号覆盖 | 宿主把皮肤 build 的 "glow" 符号覆盖到本实体（覆盖与工厂自身的 SetBuild 并存） | 生成时一次设置；符号内容未解包 | [覆盖](../../../raw/current/scripts/prefabskin.lua#L1767-L1769) |
| 资源 | 无 assets 声明；bank rabbithouse_glowfx、build rabbit_house 由 AnimState 调用 | 包是否随皮肤动态加载未解包核验 | [AnimState 设置](../../../raw/current/scripts/prefabs/rabbithouse_yule_glow_fx.lua#L12-L14) |

## 典型接入

### 借鉴皮肤常驻光晕范式

原版通道：皮肤 fx_prefab 表第 1 项 → rabbithouse_init_fn 在非 placer、非 burnt 时生成并父子挂到宿主。模组仿制要点：FX 工厂保持零组件＋显式循环；符号覆盖由宿主做（本工厂体内没有 OverrideItemSkinSymbol）；清理入口交给皮肤 clear_fn，不要在 FX 体内自带 KillFX。[init_fn](../../../raw/current/scripts/prefabskin.lua#L1763-L1772)

## 常见陷阱与限制

- **误区：SetLightOverride 就是光源**：它只让该动画贴图按比例摆脱光照压暗（自发光观感的渲染属性）；实体无 Light 组件、无 AddLight 调用，不会照亮周围，数值 0.7 的实际观感未运行核验。[工厂](../../../raw/current/scripts/prefabs/rabbithouse_yule_glow_fx.lua#L15)
- **边界：工厂体无主模拟分界**：与 lantern／icebox 的 init_fn 不同，本工厂和 rabbithouse_init_fn 都不检查 ismastersim。仓库内 icebox_init_fn 注释表明 placer 情形皮肤 init_fn 会在客户端也运行（[注释证据](../../../raw/current/scripts/prefabskin.lua#L3704-L3707)）；rabbithouse 用 `components.placer == nil` 挡掉预览分支，真实实体上双端是否各生成一份未核验——若复用此范式，先在双端实测是否有重影。
- **边界：无 KillFX／无退场**：本实体没有安装 KillFX，也不进提灯那样的 lantern_off 协议；移除只发生在皮肤 clear_fn（Remove）或宿主移除时由引擎父子关系处理（脚本内未检得）。[clear_fn](../../../raw/current/scripts/prefabskin.lua#L1774-L1781)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；完整 30 行工厂、rabbithouse_yule 皮肤定义与 fx_prefab 映射、SKIN_FX_PREFAB 保存、rabbithouse_init_fn／clear_fn、皮肤与宿主译名检索已核验。未覆盖：动画包内容未解包、SetLightOverride 实际观感、init_fn 双端调用情况、burnt 重建流程实测。
- 游戏运行：未运行。
- 结构检查：validate_page 与 validate_source_links --strict 通过（实际结果见本批交付报告）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
