---
title: "猴岛传送门假掉落（monkeyisland_portal_fxloot）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "猴岛传送门吐 loot 的表现层假掉落：四种小物件动画随机之一，带真实物理被掷出、落地才续播动画、播完自删；满额兜底路径不计入 _loot；未满额时也可被加权抽中并记入 _loot，不存档。"
aliases: ["monkeyisland_portal_fxloot", "传送门假掉落", "fxloot", "猴岛传送门特效掉落"]
tags: [prefab, fx, loot]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/monkeyisland_portal_fxloot.lua", "prefabs/monkeyisland_portal.lua", "components/lootdropper.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 猴岛传送门假掉落（monkeyisland_portal_fxloot）

## 快速判断

- 职责与适用：传送门"喷物品"的**纯表现假掉落**：bank/build `fx_portal_items`，动画从 grass/pine/rock/shell 四选一；MakeInventoryPhysics＋`FlingItem` 真实物理掷出（水平 4-6、竖直 8±2），落地才续播动画。联网实体、persists=false。[工厂](../../../raw/current/scripts/prefabs/monkeyisland_portal_fxloot.lua#L15-L47)
- 优先使用：查"传送门喷出来的东西能不能捡"选本页；生成入口是 monkeyisland_portal 的 try_portal_spawn → spawn_fx_loot。
- 重要副作用：生成即 `Pause()` 动画，逐帧查 `y < 0.2` 落地才 Resume；移除路径只有"续播完 animover 自删"一条。[落地续播](../../../raw/current/scripts/prefabs/monkeyisland_portal_fxloot.lua#L8-L13)
- 修改边界：groundshadowhandler 仅非专用机器挂载；通过 spawn_fx_loot 兜底生成时不进 `_loot`；spawn_real_loot 加权抽到本注册名时仍会入表；不存档（persists=false）。[影子](../../../raw/current/scripts/prefabs/monkeyisland_portal_fxloot.lua#L29-L32)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `monkeyisland_portal_fxloot` | fn：CreateEntity → Transform＋AnimState＋Network → MakeInventoryPhysics → bank/build/随机动画＋`Pause()` → 非专用挂 groundshadowhandler(0.8,0.5) → SetPristine → 主模拟侧 persists=false＋animover Remove＋每帧 test_for_ground | 单一注册；动画随机、暂停后由落地回调恢复；无 SG、无 inventoryitem。[工厂](../../../raw/current/scripts/prefabs/monkeyisland_portal_fxloot.lua#L15-L47)、[注册](../../../raw/current/scripts/prefabs/monkeyisland_portal_fxloot.lua#L49) |

注册名无 NAMES 条目；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：monkeyisland_portal | spawn_fx_loot：SpawnPrefab 后 `FlingItem(fx, 门位置+竖直4)`；try_portal_spawn 在 `_loot` 计数 < MONKEYISLAND_PORTAL_MAXLOOT(10) 时走真 loot，否则（或真 loot 生成失败）走本 fx | 主模拟循环：PORTALLOOT 计时器每 SPEWTIME(15 秒) 重触发（[计时](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L344-L357)） | [假 loot 生成](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L160-L166)、[分流](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L184-L194) |
| 权重：加权吐物表 | 本 fx 固定权重 10.0 是表内最高（其他条目权重来自常量或 TUNING）；未满额时 spawn_real_loot 也可按该权重选中本 FX 并记入 _loot；满额时另走不入表的 spawn_fx_loot 兜底 | 真 loot 权重可被 TUNING 覆写，fxloot 权重硬编码 | [权重表](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L35-L46)、[TUNING](../../../raw/current/scripts/tuning.lua#L6686-L6694) |
| API：LootDropper:FlingItem | 给 Physics 施加随机方向冲量实现抛掷：水平 speed 4-6、竖直 `GetRandomWithVariance(y_speed, y_speed_variance)` 即 8±2 | 本 fx 有 Physics 无 inventoryitem，照样被掷 | [FlingItem](../../../raw/current/scripts/components/lootdropper.lua#L321-L373)、[门配置](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L403-L406) |
| 关联 Prefab：monkeyisland_portal_lootfollowfx | 真 loot 与本 fx 生成后都会被 attach_light_fx 挂跟随光效子实体并临时加 outofreach 标签＋LightOverride(0.6)，`fx_len` 秒后复原；本 fx 不在 YOFFSET 表内，光效偏移 0 | followfx 的 fx_len 在主模拟侧赋值，故加光分支在主模拟恒真 | [attach_light_fx](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L173-L182)、[fx_len](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L96-L99) |
| 资源 | anim/fx_portal_items.zip（bank/build 同名） | 声明与使用已核验，包内容未解包 | [资产声明](../../../raw/current/scripts/prefabs/monkeyisland_portal_fxloot.lua#L1-L4) |

## 典型接入

### 复用"掷出后落地续播"的小物件表现

- 前提与选择：适合"从天而降的小道具"表现；核心是物理抛掷＋暂停动画＋落地恢复，不需要 SG 与 inventoryitem。
- 生效与撤销：抛出即开始下落，落地（y<0.2）自动续播并最终自删；生成方无需清理。
- 原版依据：portal 每 15 秒的吐物循环在真 loot 满 10 个后改喷本 fx，维持视觉不断（[分流](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L184-L194)）。

```lua
local fx = SpawnPrefab("monkeyisland_portal_fxloot")
SomeLootDropper:FlingItem(fx, Vector3(x, y + 4, z))
-- 落地（y<0.2）后自动 Resume 动画，播完 animover 自删
```

- 调整与验证：抛掷速度由 LootDropper 的 min/max_speed 与 y_speed_variance 决定；动画四种随机、无法指定单种（工厂内 `FX_TYPES[math.random(#FX_TYPES)]`）。

## 常见陷阱与限制

- **误区：它是可拾取的掉落物**：只有 MakeInventoryPhysics 没有 inventoryitem 组件，FlingItem 只给物理冲量；玩家无法捡起，是否记入 _loot 取决于生成路径（加权抽中会入表，满额兜底不入表）。[工厂](../../../raw/current/scripts/prefabs/monkeyisland_portal_fxloot.lua#L22-L27)
- **边界：没有兜底移除**：若实体始终未落到 y<0.2（如被卡在高处），动画保持暂停、animover 永不触发，实体滞留世界（persists=false 不存档但会一直存在）；周期任务也不自停。[落地判定](../../../raw/current/scripts/prefabs/monkeyisland_portal_fxloot.lua#L41-L46)
- **边界：开局事件不含假 loot**：trader 交月风暴火花触发的 fire_portal_event 只喷加权吐物表中的物品，本 fx 只出现在周期吐物循环。[事件](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L220-L245)
- **边界：真 loot 被清走会恢复真 loot 生成**：cleanup_outofscope_loot 把过远/失效的真 loot 移出 `_loot`（[清理](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L105-L115)），计数回落到 MAXLOOT 内后恢复加权抽取，其中仍包含权重 10 的本 FX；只有满额路径是必出 FX。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；49 行工厂全文、monkeyisland_portal 的吐物循环/权重/分流/事件/存档段、FlingItem 抛掷头段、tuning 相关常量、NAMES 检索（无命中）已核验；动画包内容、y<0.2 判定在船板等高架面的实际表现、晚加入时序未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-event-index|节庆、小游戏与事件模式]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
