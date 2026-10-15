---
title: "裂隙晶体生成烟尘（lunarrift_crystal_spawn_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "月裂隙晶体（Ryftstal）破土生成时的环状烟尘：0.5 秒自移除的联网粒子实体，作为晶体子实体生成；专用服务器因工厂提前 return 不会自毁。"
aliases: ["lunarrift_crystal_spawn_fx", "裂隙晶体生成特效", "晶体烟尘", "ryftstal spawn fx"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/lunarrift_crystal_spawn_fx.lua", "prefabs/lunarrift_crystals.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 裂隙晶体生成烟尘（lunarrift_crystal_spawn_fx）

## 快速判断

- 职责与适用：月裂隙晶体（官方译名"裂隙晶体"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L344259-L344262)）完成 spawnin 时的**一次性环状烟尘**：联网单发射器粒子实体，父级挂到晶体上，非专用机器 0.5 秒后自移除；工厂无玩法组件。[工厂](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L64-L120)
- 优先使用：查"晶体生成特效／一次性粒子环绕"选本页；晶体的 spawnin 阻挡重试与挖矿玩法在晶体工厂源码（本批未建页）。
- 重要副作用：**专用服务器不自毁**：`TheNet:IsDedicated()` 提前 return 在前（[L76-L78](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L76-L78)），`DoTaskInTime(0.5, inst.Remove)` 在后（[L117](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L117)），专用机上主模拟生成的实体没有自移除任务，会滞留到晶体本体被移除为止（persists=false 不入存档）。
- 修改边界：persists=false；粒子参数（0.5 秒寿命、50 上限、圆域 1.0、高度 1.0、reveal 着色器）写死在工厂。[粒子配置](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L83-L97)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lunarrift_crystal_spawn_fx` | fn：Transform＋Network＋FX 标签 → SetPristine → persists=false → 专用服务器提前 return → 注册色/缩放包 → VFXEffect 单发射器（烟粒子）→ EmitterManager 每帧回调（父级 IsCloudEnabled 门控，无父级则常发）→ 0.5 秒自移除任务（仅非专用走到） | 单一注册；自移除任务在 fn 末尾，属于非专用路径的一部分。[工厂](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L64-L120)、[注册](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L122) |

晶体本体官方译名"裂隙晶体"（NAMES.LUNARRIFT_CRYSTAL_BIG，[译名](../../../raw/current/scripts/languages/chinese_s.po#L344259-L344262)）；本 FX 注册名无 NAMES 条目，标题"生成烟尘"为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 宿主：lunarrift_crystal_big／small | `finish_crystal_spawnin` 确认落点无阻挡后：晶体回场景、循环播本体动画、播 deathpoof 音效，再生成本实体 `SetParent` 到晶体 | 主模拟触发；spawnin 由 timer 驱动、被阻挡时每秒重试最多 30 次 | [finish_crystal_spawnin](../../../raw/current/scripts/prefabs/lunarrift_crystals.lua#L21-L47) |
| 开关门控：IsCloudEnabled | 发射回调读父级 `IsCloudEnabled`（父级无该方法视为常开）；晶体不实现该方法，实际总是发射 | 客户端本地；专用机无发射器 | [门控](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L109-L114) |
| 自移除：DoTaskInTime(0.5) | 非专用机器 0.5 秒后 Remove；与父级存活无关，先到先删 | 仅 fn 完整走完的机器（客户端＋非专用主模拟）；专用机例外见快速判断 | [自移除](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L117)、[提前 return](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L76-L81) |
| 资源 | fx/animsmoke.tex＋shaders/vfx_particle_reveal.ksh，色/缩放包 `lunarrift_crystal_*envelope_smoke` 首次生成时注册 | 客户端；声明与使用已核验 | [资源与包](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L1-L12)、[注册包](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L20-L42) |

## 典型接入

### 复用"短命一次性粒子提示"

- 前提与选择：主模拟 SpawnPrefab 后挂到宿主（SetParent）；非专服有 0.5 秒自删，专服分支没有，复用方需补专服清理。
- 生效与撤销：非专服实体寿命为 0.5 秒；发射回调带 `IsCloudEnabled` 门控协议，挂到实现该方法的宿主可提前停止发烟。
- 原版依据：唯一生成者是晶体的 spawnin 流程，每次 spawnin 生成一个；重试成功后才播（被阻挡期间不生成）。[finish_crystal_spawnin](../../../raw/current/scripts/prefabs/lunarrift_crystals.lua#L21-L47)

```lua
local fx = SpawnPrefab("lunarrift_crystal_spawn_fx")
fx.entity:SetParent(myhost.entity)
if TheNet:IsDedicated() then
    fx:DoTaskInTime(0.5, fx.Remove) -- 补齐专服未安排的删除任务
end
```

- 调整与验证：寿命想变长需 postinit 改工厂行为（自移除时间在 fn 内写死，[L117](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L117)）；专用服务器滞留问题见常见陷阱。

## 常见陷阱与限制

- **边界：专用服务器上实体滞留**：提前 return 跳过了自移除任务与发射器；主模拟（专用机即主模拟）生成的实体要等晶体父级移除才随之消失。给晶体做增删逻辑时注意这一差异。[提前 return](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L76-L81)、[自移除](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L117)
- **误区：门控协议是晶体专属开关**：`IsCloudEnabled` 检查只是工厂通用写法（同写法见 lunar_goop_cloud_fx），晶体本体没实现该方法，烟尘总是发射；别把门控当成晶体可控开关的证据。[门控](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L109-L114)
- **边界：粒子寿命与实体寿命独立计时**：每粒寿命为 0.45~0.5 秒，发射持续到非专服实体 0.5 秒删除前，后发粒子不会恰好同时到寿。移除后的残留渲染属引擎行为，未运行验证；不能据两个常量同值断言粒子自然消散或必被截断。[常量](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L46)、[自移除](../../../raw/current/scripts/prefabs/lunarrift_crystal_spawn_fx.lua#L117)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；123 行工厂全文、lunarrift_crystals 的 spawnin 全流程（阻挡重试／音效／父子关系）已核验；粒子视觉、专用服务器滞留实测、晶体整页机制未核验（属晶体页面范围）。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
