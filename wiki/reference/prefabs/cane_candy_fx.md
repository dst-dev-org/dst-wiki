---
title: "拐杖糖雪花特效（cane_candy_fx）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "拐杖糖手杖的雪花跟随特效：雪花纹理、粒子寿命、移动增量发射及装备端创建／卸装清理。"
aliases: ["cane_candy_fx", "拐杖糖雪花特效", "手杖雪花特效", "雪花粒子", "cane_candycane"]
tags: [prefab, fx, lifecycle, server-client]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["prefabs/cane_candy_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "emitters.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 拐杖糖雪花特效（cane_candy_fx）

## 快速判断

- 职责与适用：[[reference/prefabs/cane|步行手杖]] 的拐杖糖皮肤使用的雪花粒子；依据是 SNOW 发射器与 wintersnow_cane 纹理，不是仅从皮肤名称推测。[雪花资源与配置](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L87-L102)
- 优先使用：找“手杖雪花特效”选本页；皮肤映射与装备跟随入口见 [[reference/prefabs/cane#皮肤差异与特效映射]]，本页查粒子参数。
- 重要副作用：位移提高发射基准，静止仍有低档发射；粒子寿命不代表实体会自动移除。[发射调度](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L106-L131)
- 修改边界：联网载体在各端构造，专服跳过本地 VFX；本工厂无 AddComponent，VFXEffect 是引擎实体接口。[构造分界](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L68-L88)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cane_candy_fx` | fn → Transform／Network／FX → SetPristine → persists=false → 非专服初始化包络与 VFX | 单一注册，无独立雪花子 Prefab。[工厂](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L68-L88)、[返回](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L133-L136) |

`STRINGS.SKIN_NAMES.cane_candycane` 为“拐杖糖”；本次未发现对应 FX 注册名的直接 NAMES 翻译条目；标题中文部分是已核验的效果描述，非官方译名。[皮肤译名](../../../raw/current/scripts/languages/chinese_s.po#L394140-L394142)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源：cane_candycane | fx_prefab 第 1 项由主模拟装备回调生成，添加 Follower 并跟随 owner 的 swap_object | 跟随载体；卸装由主体 Remove | [[reference/prefabs/cane#跟随-fx-与拖尾任务如何选择和停止]]；[皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2915-L2927)、[创建／撤销](../../../raw/current/scripts/prefabskin.lua#L3163-L3186) |
| VFXEffect／EnvelopeManager | 一个发射器，白色雪花渐入／淡出、缩小，8 帧 UV 随机选择 | 非专服；不挂 SG、Brain 或玩法组件 | 独立机制 Wiki 待建；[包络](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L20-L44)、[发射](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L49-L65)、[渲染](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L87-L102) |
| EmitterManager | nil lifetime 注册持续更新；实体 onremove 自动注销调度 | 非专服；管理器不替本实体设置退出时间 | [注册与注销](../../../raw/current/scripts/emitters.lua#L16-L40)、[本体注册](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L117-L131) |
| 资源 | wintersnow_cane.tex、vfx_particle.ksh；GroundPhysics、Premultiplied | 仅核验声明／使用，未解包纹理 | [资源](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L1-L12)、[渲染配置](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L90-L102) |

## 典型接入

原版由 [[reference/prefabs/cane#跟随-fx-与拖尾任务如何选择和停止|手杖装备回调]] 创建并跟随持有者，卸装 Remove。渲染器设置 SetMaxNumParticles(0,512)；本工厂未调用 SetKillOnEntityDeath，不据此推定引擎默认的残留策略。[创建／清理](../../../raw/current/scripts/prefabskin.lua#L3163-L3186)、[渲染配置](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L87-L102)

### 雪花寿命与移动发射

单粒子寿命为 4.5×(0.6+random×0.4)，即约 2.7～4.5 秒；缩放从 0.4 到 0.24，白色 alpha 在 0／0.05／0.85／1 生命周期比例处为 0／200／200／0。向下速度、随机旋转和八帧 UV 在 emit_fn 中设置。[寿命与速度](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L47-L65)、[包络](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L20-L40)

每次更新取 FX 自身位移，clamp(move×6,0,1) 插值低／高发射基准 3／50，再乘 tick_time 和 random×3，累计量大于 1 才发射。它不是固定每秒 50 片，也不是通过玩家 moving 标签开关。[实际算法](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L106-L131)

复用时保留父实体／Follower 和卸装清理，由主体管理存在期；单独生成本实体不会自动附着手杖或按粒子寿命销毁。CreateBoxEmitter 的参数也不能直接当空间最小／最大边界，其实现使用 [-1,1] 的 UnitRand。[主体生命周期](../../../raw/current/scripts/prefabskin.lua#L3163-L3186)、[发射位置](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L114-L115)、[helper 实现](../../../raw/current/scripts/emitters.lua#L157-L167)

## 常见陷阱与限制

- **误区：专服没 VFXEffect 就认为效果未生成**：专服返回联网载体，本地粒子走非专服分支。[分界](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L68-L88)
- **边界：SetMaxLifetime 只约束粒子，不 Remove 实体**：该值传给粒子；EmitterManager 使用无限 lifetime，实体清理由主体负责。[粒子限制](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L95-L98)、[持续调度](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L117-L131)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整读取本工厂，定向核验皮肤定义、装备／卸装、粒子 helper 与管理器注销及皮肤译名；主体与特效已互链。
- 游戏运行：未运行；未验证纹理图像、具体跟随观感、晚加入／换肤时序和引擎粒子销毁后的残留表现。无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
