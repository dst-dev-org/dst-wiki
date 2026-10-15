---
title: "愚人手杖彩纸特效（cane_harlequin_fx）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "愚人手杖的五色彩纸粒子：向上初速度、位移发射门槛、通道取整与主体跟随清理。"
aliases: ["cane_harlequin_fx", "愚人手杖彩纸特效", "手杖彩纸特效", "彩纸粒子", "cane_harlequin"]
tags: [prefab, fx, lifecycle, server-client]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["prefabs/cane_harlequin_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "emitters.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 愚人手杖彩纸特效（cane_harlequin_fx）

## 快速判断

- 职责与适用：[[reference/prefabs/cane|步行手杖]] 愚人皮肤的彩纸效果；confetti 纹理与红／绿／蓝／白／黄五套颜色包络。[资源](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L7-L17)、[颜色](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L23-L61)
- 优先使用：查“手杖彩纸特效”选本页；皮肤连接见 [[reference/prefabs/cane#皮肤差异与特效映射]]，花瓣选 [[reference/prefabs/cane_rose_fx]]。
- 重要副作用：位移门槛控制发射，彩纸初始向上再受负加速度；五通道分别发射，不是单粒子依次换五种颜色。[生成](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L89-L109)、[通道](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L133-L150)
- 修改边界：专服无本地 VFXEffect，但仍有联网载体；本工厂未挂玩法组件。[端分界](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L111-L131)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cane_harlequin_fx` | fn → Transform／Network／FX → SetPristine → persists=false → 非专服包络与 VFX | InitEmitters(6)，实际配置／发射为 0～4 五路，不是六路。[构造与配置](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L111-L150)、[注册](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L187-L190) |

`STRINGS.SKIN_NAMES.cane_harlequin` 为“愚人手杖”；本次未发现对应 FX 注册名的直接 NAMES 翻译条目；标题中文部分是已核验的效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L394150-L394152)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源：cane_harlequin | fx_prefab 第 1 项，装备时由主模拟生成并添加 Follower，卸装 Remove | 跟随 swap_object；存在期由主体管理 | [[reference/prefabs/cane#跟随-fx-与拖尾任务如何选择和停止]]；[定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2943-L2955)、[创建／清理](../../../raw/current/scripts/prefabskin.lua#L3163-L3186) |
| VFXEffect／EnvelopeManager | 五色、四帧 UV、缩放交替压扁，GroundPhysics、加速度 -0.8、drag=0.1 | 非专服；初速度 y=py×0.75+0.3，随后受下落配置影响 | 独立机制 Wiki 待建；[形状](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L66-L81)、[初速度](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L91-L109)、[渲染](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L133-L150) |
| EmitterManager／球面 helper | 半径 0.25 采样，nil lifetime 调度；实体 onremove 注销 | 按 FX 位移触发，未查玩家 SG | [调度](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L158-L185)、[管理器](../../../raw/current/scripts/emitters.lua#L16-L40)、[球面 helper](../../../raw/current/scripts/emitters.lua#L139-L155) |
| 资源 | fx/confetti.tex、shaders/vfx_particle.ksh，Premultiplied／Bloom | 仅源码声明／使用核验 | [声明](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L7-L17)、[使用](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L135-L146) |

## 典型接入

原版由 [[reference/prefabs/cane#跟随-fx-与拖尾任务如何选择和停止|手杖装备回调]] 创建并跟随持有者，卸装 Remove。每个已配置通道 SetMaxNumParticles(i,200)；num_emitters=4 是循环上界，0～4 共五路；本工厂未调用 SetKillOnEntityDeath。[创建／清理](../../../raw/current/scripts/prefabskin.lua#L3163-L3186)、[通道配置](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L130-L150)

### 彩纸门槛、粒子寿命与花瓣区别

位移门槛和发射取整模式与 [[reference/prefabs/cane_rose_fx#花瓣触发门槛与实际发射量]] 同型：move=clamp((位移−0.2)×10,0,1)，基准 0／15 乘 tick_time，每路独立在局部 num_to_emit>0 时循环，没有跨帧小数累计。正数不足 1 时五路各发一粒；不能把 15 直接视为最终每秒总量。[实际循环](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L158-L185)

彩纸单粒子寿命为 2.3×(0.5+UnitRand×0.5)，表达式约 0～2.3 秒；花瓣的最大寿命为 2 秒，初速度和下落加速度也不同。复用时保留目标效果自身的 UV 帧宽、颜色通道及动力学配置，不只替换纹理名字。[彩纸生成](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L89-L109)、[通道参数](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L133-L150)、[随机函数](../../../raw/current/scripts/emitters.lua#L101-L103)

## 常见陷阱与限制

- **误区：彩纸和花瓣只是不同贴图**：本版彩纸为五路、四帧 UV、初始向上与 -0.8 加速度；花瓣的路数、帧宽、速度均不同，参照两页分别核验。
- **误区：只要处于移动状态就一定发射**：门槛检查的是 FX 位移，既不读取 moving 状态，也不保证每次持有者移动都越过 0.2。[位移判定](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L169-L185)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整工厂、五通道颜色与循环、皮肤定义／译名、主体装备清理和管理器／随机 helper 已核验；与花瓣页按实际差异互链，主体已补链。
- 游戏运行：未运行；未验证最终彩纸图像、实际帧率与密度、零寿命粒子处理或骨骼位移触发效果。无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
