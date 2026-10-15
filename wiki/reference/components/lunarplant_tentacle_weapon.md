---
title: "lunarplant_tentacle_weapon"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月晶武器的触手附伤：持有者普攻命中后按幸运概率在目标旁刷出月球触手协战，默认仅虫木 Weld 技能树解锁。"
aliases: ["lunarplant_tentacle_weapon", "月晶触手武器", "触手附伤"]
tags: [component, item]
size_class: standard
dst_build: "steam-buildid:26230496"
sources: ["components/lunarplant_tentacle_weapon.lua", "prefabs/pickaxe_lunarplant.lua", "componentutil.lua", "prefabs/lunarplanttentacle.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lunarplant_tentacle_weapon

## 快速判断

- 职责与适用：挂在月晶工具/武器（镐/斧/铲三件套）上，监听持有者 `onattackother`，命中后按 20% 基础概率 + 幸运值加成在目标附近刷 `lunarplanttentacle` 并令其攻击目标。[攻击钩子](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L59-L64)
- 优先使用：宿主 prefab 只需 AddComponent（三件原版武器均未覆盖任何字段）；默认触发条件 `should_do_tentacles_fn` 要求持有者是玩家且已激活技能 `wormwood_allegiance_lunar_plant_gear_2`——非虫木玩家拿武器不会触发。[默认条件](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L1-L4)
- 重要副作用：组件随装备/卸下动态绑定 owner 事件（`equipped`/`unequipped` → `onattackother` + `onremove`），卸下即停；刷出的触手 `tentacle.owner = owner` 供归属逻辑用。[owner 绑定](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L41-L53)
- 修改边界：只对 `attack_data.weapon == self.inst` 的直接攻击生效，投掷物/前置发射的弹体命中会被明确过滤（源码注释点名此场景）；触手生成还要求 `FindWalkableOffset` 找到可行走落点。[过滤条件](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L60-L62)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `spawn_chance` | number（默认 0.2） | TryLuckRoll 的基础概率；`LuckFormulas.ShadowTentacleSpawn = CommonChanceLuckAdditive(0.2)`（`componentutil.lua:2169`），幸运值在此基础上加成。[概率入口](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L69) |
| `tentacle_prefab` | string（默认 `lunarplanttentacle`） | 生成实体；可覆盖换别的协战单位。[使用点](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L74) |
| `should_do_tentacles_fn` | `fn(inst, owner, attack_data) -> boolean` | 触发门槛；默认实现查 `skilltreeupdater:IsActivated("wormwood_allegiance_lunar_plant_gear_2")`，置 nil 则无条件触发。[默认实现](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L12) |
| `SetOwner(owner)` | owner: entity 或 nil | 内部方法：换绑 `onattackother`/`onremove` 监听；由 equipped/unequipped 事件驱动，勿手工调用。[实现](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L41-L53) |
| `TryLuckRoll(inst, chance, formula)` | `-> boolean`（`componentutil.lua:2237`） | 幸运掷点通用入口；inst 为持有者玩家，幸运系统见 componentutil 的 LuckFormulas。[定义](../../../raw/current/scripts/componentutil.lua#L2237) |

## 典型接入

### 给自定义武器加触手协战（原版参照）

- 前提与选择：宿主需为可装备武器（触发链依赖 `equipped`/`unequipped` 与 `onattackother`）；原版三件月晶工具都在 master postinit 一行挂载，无额外配置（`pickaxe_lunarplant.lua:213`、`sword_lunarplant.lua:269`、`shovel_lunarplant.lua:212`）。[挂载点](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L213)
- 生效与撤销：装备时绑定持有者攻击事件，卸下/实体移除自动解绑；触手是独立实体，落地即按自身 AI 作战，与组件不再交互（仅 `owner` 字段标记来源，`lunarplanttentacle.lua:45`）。[触手归属注释](../../../raw/current/scripts/prefabs/lunarplanttentacle.lua#L45)

```lua
-- 服务端：自定义武器接入并放开技能门槛
inst:AddComponent("lunarplant_tentacle_weapon")
inst.components.lunarplant_tentacle_weapon.should_do_tentacles_fn = nil  -- 无条件
inst.components.lunarplant_tentacle_weapon.spawn_chance = 0.1
inst.components.lunarplant_tentacle_weapon.tentacle_prefab = "lunarplanttentacle"
```

- 调整与验证：触发流程是"普攻命中 → should_fn → TryLuckRoll → 找 2 距离内可行走偏移（3 次尝试）→ 生成触手并 SetTarget"；落点找不到就静默放弃，不保证每次成功。[落点逻辑](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L70-L80)

## 常见陷阱与限制

- **非虫木持有者无效果：** 默认门槛绑定 Wormwood 专属技能节点，其他角色装备原版三件武器不会出触手；想让全角色生效需覆盖 `should_do_tentacles_fn`。[门槛](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L1-L4)
- **投掷/弹道命中不触发：** `attack_data.weapon ~= self.inst` 直接 return，飞镖、弹弓等远程路径均不满足；组件只服务近战直接命中。[过滤](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L60-L62)
- **技能树依赖是字符串硬编码：** 默认条件里技能 id 写死在组件内，技能节点更名需同步改组件或覆盖条件函数；组件不校验持有者是否有技能树系统（非玩家 owner 会在 `owner.components.skilltreeupdater` 上判 nil 安全通过后返回 false）。[判空路径](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L2-L3)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（83 行），核对三件原版武器挂载、componentutil 的 TryLuckRoll 与 ShadowTentacleSpawn 公式（`componentutil.lua:2169,2237`）、lunarplanttentacle 的 owner 注释；grep 确认三宿主均未覆盖组件字段。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
