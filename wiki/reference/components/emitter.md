---
title: "emitter"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "按面积发射器持续生成粒子（雾/地面特效）：配置发射函数与密度后一次性 Emit，交 EmitterManager 每帧补粒。"
aliases: ["emitter", "粒子发射器", "面积发射器"]
tags: [component]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/emitter.lua", "prefabs/mistparticle.lua", "gamelogic.lua", "emitters.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# emitter

## 快速判断

- 职责与适用：让一个实体成为粒子发射源。组件本身不带渲染资源，只负责按 `area_emitter` 采样位置、按 `density_factor` 控制密度，把粒子喂给实体的 `VFXEffect`（或旧版 `ParticleEmitter`），并注册到 `EmitterManager` 每帧补充。[发射流程](../../../raw/current/scripts/components/emitter.lua#L15-L26)
- 优先使用：做世界级地面粒子（如雾）时，建带 `VFXEffect` 的 prefab 并挂本组件，参照 `mistparticle` 的配置方式。[原版参照](../../../raw/current/scripts/prefabs/mistparticle.lua#L92-L96)
- 重要副作用：`Emit()` 只应调用一次——它把 updateFunc 注册进 `EmitterManager`（`AddEmitter(inst, nil, updateFunc)`），之后每帧由管理器驱动补充粒子；重复调用会重复注册。[注册](../../../raw/current/scripts/components/emitter.lua#L74)
- 修改边界：仅客户端有意义（`mist` 只在 `not TheNet:IsDedicated()` 时生成）；发射速率固定为 `density_factor * 1` 粒/秒，不随 `max_lifetime` 缩放。[速率](../../../raw/current/scripts/components/emitter.lua#L30-L31)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `config` | table，须含 `max_num_particles` | 粒子上限乘以 `density_factor` 后写入 `SetMaxNumParticles`；缺失会在 Emit 时报错。[使用点](../../../raw/current/scripts/components/emitter.lua#L23-L26) |
| `area_emitter` | `function() -> px, pz` | 每粒采样出生位置；构造默认为打印"no emitter"的占位函数，使用前必须覆盖。[默认值](../../../raw/current/scripts/components/emitter.lua#L3) |
| `density_factor` | number（默认 1） | 同时缩放粒子上限与生成速率；原版按节点面积算 `math.ceil(node.area / 4) / 31`。[原版取值](../../../raw/current/scripts/gamelogic.lua#L571) |
| `max_lifetime` / `ground_height` / `particles_per_tick` | number（默认 1） | 粒子寿命（±10% 抖动）、出生高度 y、每帧期望增量。[构造默认](../../../raw/current/scripts/components/emitter.lua#L6-L10) |
| `Emit()` | `-> nil` | 一次性入口：设置粒子上限、构造 emit/update 函数并注册 `EmitterManager:AddEmitter`。[实现](../../../raw/current/scripts/components/emitter.lua#L74) |

## 典型接入

### 世界节点级雾粒子（原版参照）

- 前提与选择：prefab 内先建 `VFXEffect`（`InitEmitters`、色彩/缩放 envelope 等），再挂组件；世界生成时按地图节点摆放。[mist 配置](../../../raw/current/scripts/prefabs/mistparticle.lua#L68-L77)
- 生效与撤销：`Emit()` 后由 `EmitterManager` 持续驱动（含休眠/唤醒，见 `emitters.lua`）；实体移除即随管理器清理。

```lua
-- prefab master postinit 内，参照 mistparticle.lua
inst:AddComponent("emitter")
inst.components.emitter.config = my_config           -- 须含 max_num_particles
inst.components.emitter.max_lifetime = 31
inst.components.emitter.ground_height = 0.4
inst.components.emitter.particles_per_tick = 1

-- 世界生成侧（gamelogic.lua 的 mist 节点做法）
inst.components.emitter.area_emitter = my_area_fn    -- 返回 px, pz
inst.components.emitter.density_factor = 0.5
inst.components.emitter:Emit()
```

- 调整与验证：`density_factor` 同时放大上限与速率，调密度勿再单改 `particles_per_tick`。

## 常见陷阱与限制

- **未覆盖 `area_emitter` 就 `Emit()`：** 每粒采样会命中默认占位函数并打印 "no emitter"，位置为 nil 导致加粒子失败；必须先赋自己的采样函数。[默认占位](../../../raw/current/scripts/components/emitter.lua#L3)
- **`config.max_num_particles` 缺失：** Emit 内直接读该字段，构造后不给 `config` 会报错。[读取点](../../../raw/current/scripts/components/emitter.lua#L23)
- **重复调用 `Emit()`：** 会向 `EmitterManager` 重复注册 update 函数；组件设计为一次 Emit 长期生效。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（77 行），核对 mistparticle 配置、gamelogic 雾节点接入与 `emitters.lua` 的 EmitterManager 注册/驱动链。未审阅 `VFXEffect` 引擎侧实现。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
