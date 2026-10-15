---
title: "山羊头手杖闪光特效（cane_victorian_fx）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "山羊头手杖的淡黄色闪光粒子：闪烁包络、加法混合、移动增量发射与跟随载体清理。"
aliases: ["cane_victorian_fx", "山羊头手杖闪光特效", "手杖闪光粒子", "手杖星光特效", "cane_victorian"]
tags: [prefab, fx, lifecycle, server-client]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["prefabs/cane_victorian_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "emitters.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 山羊头手杖闪光特效（cane_victorian_fx）

## 快速判断

- 职责与适用：[[reference/prefabs/cane|步行手杖]] 的山羊头皮肤闪光效果，使用 sparkle 纹理和淡黄闪烁包络。[颜色](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L20-L32)、[资源](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L91-L102)
- 优先使用：查闪光、星光粒子选本页；皮肤本体与加速能力仍查 [[reference/prefabs/cane#皮肤差异与特效映射]]。
- 重要副作用：低／高发射基准按 FX 位移插值，静止并非完全停止；Bloom 和加法混合只是渲染配置，不等同游戏 Light。[渲染](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L88-L102)、[调度](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L106-L131)
- 修改边界：专服仅保留联网 FX 载体；工厂不挂玩法组件，未创建 Light。[工厂分界](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L69-L89)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cane_victorian_fx` | fn → Transform／Network／FX → SetPristine → persists=false → 非专服包络／VFX | 单一注册，一个粒子发射器。[初始化](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L69-L89)、[注册](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L133-L136) |

皮肤键 `STRINGS.SKIN_NAMES.cane_victorian` 为“山羊头手杖”；本次未发现对应 FX 注册名的直接 NAMES 翻译条目；标题中文部分是已核验的效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L394165-L394167)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源：cane_victorian | fx_prefab 第 1 项，主模拟装备时创建并跟随持有者，卸装移除 | Follower 由主体添加，不在本工厂创建 | [[reference/prefabs/cane#跟随-fx-与拖尾任务如何选择和停止]]；[皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2984-L2996)、[创建／清理](../../../raw/current/scripts/prefabskin.lua#L3163-L3186) |
| VFXEffect／EnvelopeManager | 淡黄 alpha 周期闪烁；缩放 0.4→0.2，四帧 UV，Additive／Bloom | 非专服；粒子寿命与实体生命周期分开 | 独立机制 Wiki 待建；[包络](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L20-L45)、[粒子](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L48-L67)、[渲染](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L88-L102) |
| EmitterManager | nil lifetime 注册；onremove 自动移出调度表 | 主体管理实体 Remove；本地调度不另设实体定时退出 | [调用](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L117-L131)、[注销机制](../../../raw/current/scripts/emitters.lua#L16-L40) |
| 资源 | fx/sparkle.tex、shaders/vfx_particle_add.ksh | 仅源码声明／使用已核验 | [资源声明](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L1-L12) |

## 典型接入

原版由 [[reference/prefabs/cane#跟随-fx-与拖尾任务如何选择和停止|手杖装备回调]] 创建并跟随持有者，卸装 Remove。本通道 SetMaxNumParticles(0,256)、SetSortOffset(0,2)；本工厂未调用 SetKillOnEntityDeath。[创建／清理](../../../raw/current/scripts/prefabskin.lua#L3163-L3186)、[渲染配置](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L88-L102)

### 闪烁与移动密度

颜色包络交替使用 (255,255,150) 的不透明／透明节点；粒子寿命表达式是 1.75×(0.7+UnitRand×0.3)。UnitRand 为 [-1,1]，因此约 0.7～1.75 秒，不能将它当 [0,1] 得出更窄范围。[包络](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L20-L45)、[寿命](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L48-L67)、[随机函数](../../../raw/current/scripts/emitters.lua#L101-L103)

球面发射半径 0.25；位移×6 限幅后插值基准 5／50，乘 tick_time 与 random×3 后累计、超过 1 发射。调密度要保留累计机制，不把 high=50 解释成实际固定每秒数量。[调度](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L106-L131)、[球面 helper](../../../raw/current/scripts/emitters.lua#L139-L155)

需要复用跟随效果时沿用主体的 Follower／卸装入口；本页只核验参数与关系，不提供未经验证的独立生成工程。[主体入口](../../../raw/current/scripts/prefabskin.lua#L3163-L3186)

## 常见陷阱与限制

- **误区：星光就是照明光源**：本工厂只创建 VFXEffect，没有 AddLight；Bloom 不能据此推定照明或防黑暗能力。[工厂](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L69-L102)
- **误区：粒子到寿命即删除载体**：实体由主体清理，管理器 nil lifetime 不设置自动退出。[调度注册](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L117-L131)、[管理器](../../../raw/current/scripts/emitters.lua#L16-L40)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整工厂、皮肤定义／译名、装备生成与卸装、UnitRand／球面 helper 和调度注销已核验，主体已回补链接。
- 游戏运行：未运行；未解包纹理、未验证最终视觉亮度、换肤／晚加入与资源兼容，无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
