---
title: "玫瑰红手杖花瓣特效（cane_rose_fx）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "玫瑰红手杖的红白黄花瓣粒子：位移门槛、三个实际配置通道、逐帧发射取整与跟随清理。"
aliases: ["cane_rose_fx", "玫瑰红手杖花瓣特效", "手杖花瓣特效", "花瓣粒子", "cane_rose"]
tags: [prefab, fx, lifecycle, server-client]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["prefabs/cane_rose_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "emitters.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 玫瑰红手杖花瓣特效（cane_rose_fx）

## 快速判断

- 职责与适用：[[reference/prefabs/cane|步行手杖]] 玫瑰红皮肤的花瓣效果；使用 petal 纹理、红／白／黄三套颜色包络。[资源](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L7-L17)、[颜色](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L23-L45)
- 优先使用：查“手杖花瓣特效”选本页；皮肤与装备回调查 [[reference/prefabs/cane#皮肤差异与特效映射]]，彩纸效果另见 [[reference/prefabs/cane_harlequin_fx]]。
- 重要副作用：以 FX 每次更新的位移触发；实际逐通道发射量存在向上取整，不能把 high=15 写成固定每秒 15 粒。[调度](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L141-L168)
- 修改边界：专服只返回联网载体，非专服创建粒子；工厂无 AddComponent，未注册独立花瓣实体。[构造](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L94-L114)、[注册](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L170-L173)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cane_rose_fx` | fn → Transform／Network／FX → SetPristine → persists=false → 非专服包络与 VFX | InitEmitters(6)，但配置／发射循环仅遍历 0～2 三个通道。[工厂](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L94-L133)、[发射循环](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L161-L168) |

皮肤译名来自 `STRINGS.SKIN_NAMES.cane_rose`：“玫瑰红手杖”。本次未发现对应 FX 注册名的直接 NAMES 翻译条目；标题中文部分是已核验的效果描述，非官方译名。[皮肤译名](../../../raw/current/scripts/languages/chinese_s.po#L394155-L394157)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源：cane_rose | fx_prefab 第 1 项由主模拟装备回调生成，添加 Follower；卸装 Remove | 跟随 owner 的 swap_object；本工厂不设置父实体 | [[reference/prefabs/cane#跟随-fx-与拖尾任务如何选择和停止]]；[定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2957-L2968)、[创建／撤销](../../../raw/current/scripts/prefabskin.lua#L3163-L3186) |
| VFXEffect／EnvelopeManager | 三种颜色、八帧 UV；GroundPhysics、向下加速度 -0.2、drag=0.1 | 非专服；缩放包络交替压扁两个轴，形成翻转式变化 | 独立机制 Wiki 待建；[形状包络](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L50-L65)、[渲染与物理参数](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L113-L133) |
| EmitterManager／球面 helper | nil lifetime 持续调度，实体 onremove 注销；采样半径 0.25 的球面 | 位移影响发射量，不依赖玩家 moving 标签 | [注册](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L149-L168)、[注销](../../../raw/current/scripts/emitters.lua#L16-L40)、[球面采样](../../../raw/current/scripts/emitters.lua#L139-L155) |
| 资源 | fx/petal.tex、shaders/vfx_particle.ksh，Premultiplied／Bloom | 资源内容未解包验证 | [声明](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L7-L17)、[使用](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L118-L129) |

## 典型接入

原版由 [[reference/prefabs/cane#跟随-fx-与拖尾任务如何选择和停止|手杖装备回调]] 创建并跟随持有者，卸装 Remove。每个已配置通道 SetMaxNumParticles(i,200)；num_emitters=2 是循环上界，0～2 共三路；本工厂未调用 SetKillOnEntityDeath。[创建／清理](../../../raw/current/scripts/prefabskin.lua#L3163-L3186)、[通道配置](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L113-L133)

### 花瓣触发门槛与实际发射量

move=clamp((本次位移−0.2)×10,0,1)：位移不超过 0.2 时为零，到 0.3 时达到高档。这里量的是 FX 相邻更新位置差，不是人物每秒移速；跟随骨骼运动也不能未经运行验证就等同人物平移。[位置与门槛](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L149-L159)

基准 0／15 乘 tick_time 后，循环为每个通道重新令 num_to_emit=per_tick，再在其大于 0 时发射并减 1，没有跨更新累计小数。正数不足 1 仍每通道发射一粒；例如 tick_time=1/30、高档 per_tick=0.5 时，一次更新共发射三粒。此为源码分支推算，不是游戏测得的稳定帧率／粒子速率。[基准与循环](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L141-L168)

单粒子寿命为 2×(0.5+UnitRand×0.5)，UnitRand 在 [-1,1]，表达式范围约 0～2 秒；不能误按 [0,1] 写成 1～2 秒。粒子寿命不控制载体存在期，载体仍由主体清理。[粒子生成](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L73-L92)、[随机函数](../../../raw/current/scripts/emitters.lua#L101-L103)、[主体清理](../../../raw/current/scripts/prefabskin.lua#L3177-L3186)

## 常见陷阱与限制

- **误区：InitEmitters(6) 代表六路花瓣都在发射**：实际配置与发射只处理 0～2，不能按预分配数量描述效果。[配置循环](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L113-L133)、[发射循环](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L161-L168)
- **误区：所有花瓣都是红色**：三个通道分别使用红、白、黄包络，皮肤名字不代替效果实现。[颜色定义](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L23-L45)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整工厂、颜色／发射循环、皮肤定义／译名、主体创建与清理、随机／球面 helper 和调度注销已核验，主体已补链。
- 游戏运行：未运行；未验证纹理内容、零寿命粒子的引擎处理、实际更新频率、视觉密度或换肤时序。无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
