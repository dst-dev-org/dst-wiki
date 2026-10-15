---
title: "腐败手杖紫烟特效（cane_sharp_fx）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "腐败手杖的紫色烟雾跟随效果：烟雾包络、固定基准发射、FollowEmitter 与实体清理边界。"
aliases: ["cane_sharp_fx", "腐败手杖紫色烟雾", "手杖紫烟特效", "手杖烟雾特效", "cane_sharp", "腐败手杖紫烟特效"]
tags: [prefab, fx, lifecycle, server-client]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["prefabs/cane_sharp_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "emitters.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 腐败手杖紫烟特效（cane_sharp_fx）

## 快速判断

- 职责与适用：[[reference/prefabs/cane|步行手杖]] 腐败皮肤的紫烟效果；animsmoke 纹理和 (75,0,130) 渐隐颜色共同支持该描述。[资源](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L1-L12)、[颜色](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L20-L41)
- 优先使用：查“手杖紫烟／烟雾特效”选本页；皮肤选择、穿脱及加速能力查 [[reference/prefabs/cane#皮肤差异与特效映射]]。
- 重要副作用：本效果不按移动距离插值，持续以基准 3 配合随机量累计发射；粒子设置 FollowEmitter 和 KillOnEntityDeath。[发射调度](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L103-L119)、[粒子配置](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L84-L98)
- 修改边界：联网载体与本地粒子分离，专服提前返回；工厂没有玩法组件／SG／Brain。[构造](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L65-L85)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cane_sharp_fx` | fn → Transform／Network／FX → SetPristine → persists=false → 非专服 VFX | 仅一个注册、一个发射器。[构造](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L65-L85)、[返回](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L121-L124) |

皮肤 `STRINGS.SKIN_NAMES.cane_sharp` 的官方中文名是“腐败手杖”；本次未发现对应 FX 注册名的直接 NAMES 翻译条目；标题中文部分是已核验的效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L394160-L394162)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源：cane_sharp | fx_prefab 第 1 项，由主模拟装备回调生成并添加 Follower；卸装 Remove | 皮肤本体负责持有与释放，本工厂不自建跟随关系 | [[reference/prefabs/cane#跟随-fx-与拖尾任务如何选择和停止]]；[皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2970-L2982)、[主体生命周期](../../../raw/current/scripts/prefabskin.lua#L3163-L3186) |
| VFXEffect／EnvelopeManager | AlphaBlended／Bloom；紫色逐渐透明并扩张，粒子跟随发射器 | 非专服；KillOnEntityDeath 只记录调用，不代替运行验证 | 独立机制 Wiki 待建；[包络](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L20-L41)、[渲染](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L84-L98) |
| EmitterManager | nil lifetime 注册更新，onremove 注销；使用半径 0.05 的球面 helper | 无移动门禁或位移密度插值 | [调度](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L103-L119)、[管理器](../../../raw/current/scripts/emitters.lua#L16-L40)、[球面 helper](../../../raw/current/scripts/emitters.lua#L139-L155) |
| 资源 | fx/animsmoke.tex、shaders/vfx_particle_reveal.ksh | 只核验声明／使用，不提供原版资源文件 | [资源声明](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L1-L12) |

## 典型接入

原版由 [[reference/prefabs/cane#跟随-fx-与拖尾任务如何选择和停止|手杖装备回调]] 创建并跟随持有者，卸装 Remove。本通道 SetMaxNumParticles(0,128)、SetSortOffset(0,-1)、SetKillOnEntityDeath(0,true)；本批五种粒子工厂中只有本工厂显式设置后者，调用不等于已实测销毁后的视觉残留。[创建／清理](../../../raw/current/scripts/prefabskin.lua#L3163-L3186)、[渲染配置](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L84-L98)

### 紫烟大小、寿命与发射频率

GLOW_MAX_LIFETIME=2.1，单粒子寿命为 2.1×(0.9+random×0.1)，约 1.89～2.1 秒；缩放从 0.21×0.7 增至 0.21×1.3，alpha 从 160 渐隐到 0。[包络](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L20-L41)、[粒子生成](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L48-L63)

每次调度先消费已有累计量，再加 3×tick_time×random×3；不要把 desired_pps=3 当作确定每秒三粒，也不要套用雪花／闪光的位移插值。本地粒子跟随开关 SetFollowEmitter 与主体实体的 Follower 是不同层次，后者连接持有者。[VFX 开关](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L97-L98)、[更新顺序](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L103-L119)、[主体跟随](../../../raw/current/scripts/prefabskin.lua#L3163-L3170)

## 常见陷阱与限制

- **误区：sharp 名称说明会造成伤害**：工厂只配置 VFX，没有 weapon／combat；伤害与加速仍属于手杖主体。[完整构造入口](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L65-L121)
- **误区：非持久化等于自动删除**：persists=false 只说明不按普通实体持久化；本地管理器 lifetime=nil，原版依赖主体卸装／移除清理。[工厂](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L73-L85)、[主体清理](../../../raw/current/scripts/prefabskin.lua#L3177-L3186)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整工厂、皮肤定义／译名、主体跟随与清理、球面 helper 与调度注销已核验；以源码颜色／纹理描述紫烟，主体已回补链接。
- 游戏运行：未运行；未验证引擎 FollowEmitter／KillOnEntityDeath 的最终观感、实际资源内容或其他换肤路径。无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
