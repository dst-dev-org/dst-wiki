---
title: "盐灯持续光点特效（lantern_crystal_fx_ground）"
created: "2026-10-29"
updated: "2026-11-08"
type: prefab
summary: "盐灯皮肤提灯的持续光点动画特效：同一工厂用共同函数注册 ground 与 held 两个实体，显式循环播放 idle 动画，无粒子；KillFX 立即 Remove，无 pst 动画。"
aliases: ["lantern_crystal_fx_ground", "lantern_crystal_fx_held", "盐灯特效", "盐灯光点", "提灯光尘特效"]
tags: [prefab, fx, lifecycle, server-client]
size_class: simple
dst_build: "steam-buildid:26474873"
sources: ["prefabs/lantern_crystal_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "prefabs/mininglantern.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 盐灯持续光点特效（lantern_crystal_fx_ground）

## 快速判断

- 职责与适用：盐灯（官方皮肤译名，见下）提灯的持续光点动画特效。工厂用 commonfn 同时注册 `lantern_crystal_fx_ground`（idle_ground）与 `lantern_crystal_fx_held`（idle_held）：比 [[reference/prefabs/lantern_flower_fx_held]] 族简单得多——没有本地粒子、没有 pre／loop／pst 状态机，只是单个联网动画实体循环播 idle。查"盐灯特效"选本页。
- 优先使用：需要"极简持续 FX 范式"（Transform／AnimState／Network＋显式循环）时本页是更短的原版参照；宿主生成与清理协议同提灯皮肤系统，见下表。
- 重要副作用：PlayAnimation(anim, true) 显式传了循环参数 true，与 flower 族的 animover 重播不同；KillFX 即 Remove，无 pst 退场。本工厂无 AddLight，照明仍由独立 lanternlight 实体承担。[循环](../../../raw/current/scripts/prefabs/lantern_crystal_fx.lua#L22)、[照明实体](../../../raw/current/scripts/prefabs/mininglantern.lua#L206-L230)
- 修改边界：ground 与 held 都安装 KillFX（即 Remove）；两变体均有 AnimState，宿主的符号覆盖对两变体都生效。bank lantern_crystal_fx、build lantern。[构造](../../../raw/current/scripts/prefabs/lantern_crystal_fx.lua#L11-L36)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lantern_crystal_fx_ground` | commonfn("idle_ground") → Transform／AnimState／Network＋FX 标签 → SetPristine → 主模拟 persists=false＋安装 KillFX | 联网动画实体；显式循环；POPULATING 无处理（不随机帧）。[groundfn](../../../raw/current/scripts/prefabs/lantern_crystal_fx.lua#L38-L40)、[commonfn](../../../raw/current/scripts/prefabs/lantern_crystal_fx.lua#L11-L36) |
| `lantern_crystal_fx_held` | commonfn("idle_held")，其余完全同 ground | 仅动画名不同；Follower 由宿主装载协议添加。[heldfn](../../../raw/current/scripts/prefabs/lantern_crystal_fx.lua#L42-L44)、[注册](../../../raw/current/scripts/prefabs/lantern_crystal_fx.lua#L46-L47) |

皮肤键 `STRINGS.SKIN_NAMES.lantern_crystal` 为"盐灯"（官方译名，msgid "Salt Lamp"）；未发现 FX 注册名的直接 NAMES 翻译条目，标题"持续光点特效"是按 ddot_small 符号与 idle 动画写的已核验效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L398289-L398292)。皮肤 `lantern_crystal` 的 init_fn 传 overridesymbols { "ddot_small" } 与跟手偏移 Vector3(93, -55, 0)（其余提灯皮肤多为 (67, -7, 0)）。[皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L7319-L7329)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 宿主生成与清理 | 与 flower 族同一套协议：SKIN_FX_PREFAB 保存 fx_prefab，lantern_init_fn 读为 _heldfx／_groundfx，turnon／turnoff 推事件驱动 lantern_on／lantern_off | 仅主模拟；ground 有 KillFX，enterlimbo 还原协议对它生效 | [init_fn](../../../raw/current/scripts/prefabskin.lua#L3475-L3499)、[on](../../../raw/current/scripts/prefabskin.lua#L3431-L3473)、[off](../../../raw/current/scripts/prefabskin.lua#L3411-L3429)、[事件推送](../../../raw/current/scripts/prefabs/mininglantern.lua#L87) |
| 宿主符号覆盖 | 因两变体都有 AnimState，lantern_on 对 held／ground 都按 ddot_small 做 OverrideItemSkinSymbol | 生成时一次设置；符号内容未解包 | [覆盖条件](../../../raw/current/scripts/prefabskin.lua#L3441-L3445) |
| 照明分工 | 本 FX 无 Light；照明在 lanternlight 实体（强度／半径随燃料插值） | 独立联网实体 | [照明实体](../../../raw/current/scripts/prefabs/mininglantern.lua#L206-L230)、[燃料插值](../../../raw/current/scripts/prefabs/mininglantern.lua#L34-L41) |
| 资源 | anim/dynamic/lantern_crystal.zip＋PKGREF lantern_crystal.dyn；bank lantern_crystal_fx、build lantern | 声明与使用已核验，包内容未解包 | [资源声明](../../../raw/current/scripts/prefabs/lantern_crystal_fx.lua#L1-L5) |

## 典型接入

### 借鉴极简持续 FX 范式

原版通道：皮肤 fx_prefab 两项 → 宿主协议生成／Kill（与 flower 族完全一致，见上表）。本工厂适合当"最小可用皮肤 FX"模板：一个 commonfn 按后缀选 idle 动画，显式 `PlayAnimation(anim, true)` 循环，`inst.KillFX = KillFX` 满足宿主 lantern_off 的分派检查（有 KillFX 走解绑＋重定位＋KillFX 分支）。直接 Remove 也能工作，但会跳过宿主的 Follower 停止与 _lastpos 还原。[分派](../../../raw/current/scripts/prefabskin.lua#L3414-L3424)

## 常见陷阱与限制

- **边界：KillFX 是立即 Remove**：与 flower 族的地 pst 退场不同，本工厂 KillFX 无延迟、无退场动画；关灯／卸下时光点立刻消失。[KillFX](../../../raw/current/scripts/prefabs/lantern_crystal_fx.lua#L7-L9)
- **边界：无 POPULATING 处理**：世界加载时不像 flower 族那样随机帧去同步，未做随机起始帧；不能保证不同生成时刻的实例同帧起播，晚加入时序未实测。[工厂](../../../raw/current/scripts/prefabs/lantern_crystal_fx.lua#L11-L36)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；完整 48 行工厂、crystal 皮肤定义与 fx_prefab 映射、宿主 lantern 协议（与 flower 页共用的核验）、照明实体与译名检索已核验。未覆盖：动画包内容未解包、最终视觉、ddot_small 符号内容、多次开关／换肤时序实测。
- 游戏运行：未运行。
- 结构检查：validate_page 与 validate_source_links --strict 通过（实际结果见本批交付报告）。

返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-03）。
