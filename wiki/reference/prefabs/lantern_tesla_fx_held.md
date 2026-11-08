---
title: "特斯拉线圈提灯电光特效（lantern_tesla_fx_held）"
created: "2026-10-29"
updated: "2026-11-08"
type: prefab
summary: "特斯拉线圈提灯皮肤的电光动画特效：MakeFX 闭包按后缀注册 held 与 ground 两个 Prefab，带 bloom 渲染与 POPULATING 随机空白帧。"
aliases: ["lantern_tesla_fx_held", "lantern_tesla_fx_ground", "lantern_tesla_fx_", "特斯拉线圈提灯特效", "提灯电光特效"]
tags: [prefab, fx, lifecycle, server-client]
size_class: simple
dst_build: "steam-buildid:26474873"
sources: ["prefabs/lantern_tesla_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "prefabs/mininglantern.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 特斯拉线圈提灯电光特效（lantern_tesla_fx_held）

## 快速判断

- 职责与适用：特斯拉线圈提灯（官方皮肤译名，见下）的电光动画特效。文件名／census 里出现的 `lantern_tesla_fx_` 是 MakeFX 动态拼名的公共前缀，不是实际注册名：`Prefab("lantern_tesla_fx_"..suffix)` 按 suffix "held"／"ground" 注册 `lantern_tesla_fx_held` 与 `lantern_tesla_fx_ground`（工厂尾行注释也按全名标注便于搜索）。查"特斯拉线圈提灯特效"选本页。
- 优先使用：需要"同一 fn 拼名注册多变体"范式时参照本工厂；持续循环单实体比 [[reference/prefabs/lantern_flower_fx_held]] 族简单，与 [[reference/prefabs/lantern_crystal_fx_ground]] 同级，多出 bloom 与 POPULATING 随机帧两点。
- 重要副作用：PlayAnimation("idle_"..suffix, true) 显式传循环参数 true；SetBloomEffectHandle("shaders/anim.ksh") 是渲染配置，不是 Light——本工厂无 AddLight，照明仍由独立 lanternlight 实体承担。[动画与渲染](../../../raw/current/scripts/prefabs/lantern_tesla_fx.lua#L21-L23)、[照明实体](../../../raw/current/scripts/prefabs/mininglantern.lua#L206-L230)
- 修改边界：不安装 KillFX——宿主 lantern_off 走 else 分支直接 Remove，enterlimbo 还原协议也不装（仅 KillFX 非空才装）；两变体均有 AnimState，宿主符号覆盖对两变体生效。[KillFX／Remove 分派](../../../raw/current/scripts/prefabskin.lua#L3424-L3426)、[enterlimbo 条件](../../../raw/current/scripts/prefabskin.lua#L3466-L3468)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lantern_tesla_fx_held` | MakeFX("held") → fn：Transform／AnimState／Network＋FX 标签 → idle_held 显式循环 → bloom → SetPristine → 主模拟 persists=false＋POPULATING 随机帧 | bank lantern_tesla_fx、build lantern；POPULATING 时 SetFrame 取 BLANK_FRAMES={0,14,15,29,30,31,42} 中随机一帧。[fn](../../../raw/current/scripts/prefabs/lantern_tesla_fx.lua#L10-L38)、[拼名注册](../../../raw/current/scripts/prefabs/lantern_tesla_fx.lua#L40-L45) |
| `lantern_tesla_fx_ground` | MakeFX("ground")，除动画名 idle_ground 外与 held 完全同体 | 同一闭包工厂，无独立函数体。[MakeFX](../../../raw/current/scripts/prefabs/lantern_tesla_fx.lua#L9-L41) |

皮肤键 `STRINGS.SKIN_NAMES.lantern_tesla` 为"特斯拉线圈提灯"（官方译名）；未发现 FX 注册名的直接 NAMES 翻译条目，标题"电光特效"按皮肤符号 bolt_b／bolt_c 与 bloom 配置写的已核验效果描述（非官方译名，具体视觉未解包核验）。[译名](../../../raw/current/scripts/languages/chinese_s.po#L398334-L398337)。皮肤 `lantern_tesla` 另带 marketable=true；其 fx_prefab 两项即上述全名注册名。[皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L7424-L7435)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 宿主生成与清理 | 与其他提灯皮肤同一套协议：SKIN_FX_PREFAB → lantern_init_fn 的 _heldfx／_groundfx，turnon／turnoff 推事件驱动 lantern_on／lantern_off | 仅主模拟；无 KillFX → 卸下／关灯即时 Remove，无 pst 退场 | [init_fn](../../../raw/current/scripts/prefabskin.lua#L3475-L3499)、[off 分派](../../../raw/current/scripts/prefabskin.lua#L3411-L3429)、[事件推送](../../../raw/current/scripts/prefabs/mininglantern.lua#L87) |
| 宿主符号覆盖 | overridesymbols { "bolt_b", "bolt_c" }；两变体都有 AnimState，held／ground 都被覆盖 | 生成时一次设置；符号内容未解包 | [皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L7430)、[覆盖条件](../../../raw/current/scripts/prefabskin.lua#L3441-L3445) |
| POPULATING 随机帧 | 世界加载时 SetFrame 到 BLANK_FRAMES 随机一项 | 仅主模拟 POPULATING；两变体共享该逻辑 | [BLANK_FRAMES](../../../raw/current/scripts/prefabs/lantern_tesla_fx.lua#L7)、[随机帧](../../../raw/current/scripts/prefabs/lantern_tesla_fx.lua#L33-L35) |
| 照明分工 | 本 FX 无 Light；照明在 lanternlight 实体（强度／半径随燃料插值） | 独立联网实体 | [照明实体](../../../raw/current/scripts/prefabs/mininglantern.lua#L206-L230)、[燃料插值](../../../raw/current/scripts/prefabs/mininglantern.lua#L34-L41) |
| 资源 | anim/dynamic/lantern_tesla.zip＋PKGREF lantern_tesla.dyn；渲染着色器 shaders/anim.ksh（经 SetBloomEffectHandle） | 声明与使用已核验，包内容未解包 | [资源声明](../../../raw/current/scripts/prefabs/lantern_tesla_fx.lua#L1-L5) |

## 典型接入

### 借鉴拼名多变体工厂

需要"一套 fn 注册 held／ground 两个实体"时照抄 MakeFX 结构：外层闭包接 suffix，内层 fn 拼动画名，`return Prefab("lantern_tesla_fx_"..suffix, fn, assets)` 拼注册名。POPULATING 随机帧（BLANK_FRAMES）可按需保留——去掉它会让世界加载的所有实例同帧起播。若复用方需要 pst 退场，必须自行补 KillFX 并确认动画段存在，原版没有。[工厂](../../../raw/current/scripts/prefabs/lantern_tesla_fx.lua#L9-L45)

## 常见陷阱与限制

- **误区：census／搜索串 `lantern_tesla_fx_` 是注册名**：按该名 SpawnPrefab 会失败；实际注册名必须带后缀，工厂以尾注释复述全名防搜索遗漏。[拼名](../../../raw/current/scripts/prefabs/lantern_tesla_fx.lua#L40-L43)
- **误区：bloom 即照明**：SetBloomEffectHandle 只改渲染（泛光观感），实体没有 Light 组件，不能当作照明或防黑暗手段引用。[渲染](../../../raw/current/scripts/prefabs/lantern_tesla_fx.lua#L22)、[照明实体](../../../raw/current/scripts/prefabs/mininglantern.lua#L206-L230)
- **边界：无 KillFX 的取舍**：宿主卸载协议对其走即时 Remove，因此不需要 pst 动画段；若复用时改成延迟清理，需自行补 KillFX 并确认动画段存在。[分派](../../../raw/current/scripts/prefabskin.lua#L3411-L3429)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；完整 46 行工厂（含 MakeFX 拼名）、tesla 皮肤定义与 fx_prefab 映射、宿主 lantern 协议、照明实体与译名检索已核验。未覆盖：动画包内容未解包、BLANK_FRAMES 帧的可见内容、bloom 最终视觉、电光时序实测。
- 游戏运行：未运行。
- 结构检查：validate_page 与 validate_source_links --strict 通过（实际结果见本批交付报告）。

返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-03）。
