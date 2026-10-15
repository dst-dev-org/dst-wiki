---
title: "balloonmaker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "在物品上提供 MakeBalloon 在指定坐标生成气球；原版已标注弃用，仅为模组保留，MAKEBALLOON 动作无登记入口。"
aliases: ["balloonmaker", "气球制造器", "生成气球", "MAKEBALLOON"]
tags: [component, item]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/balloonmaker.lua", "prefabs/balloons_empty.lua", "actions.lua", "stategraphs/SGwilson.lua", "recipes.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# balloonmaker

## 快速判断

- 职责与适用：唯一方法 `MakeBalloon(x,y,z)` 在指定坐标 SpawnPrefab(`balloon`)；挂载对象是 `balloons_empty`（空气球），源码注释明确"deprecated, but left here for mods"。[实现](../../../raw/current/scripts/components/balloonmaker.lua#L5-L15) [弃用注释](../../../raw/current/scripts/prefabs/balloons_empty.lua#L105)
- 优先使用：模组要复用"充气出气球"的旧行为时可直接调 `MakeBalloon`；原版玩家侧气球的实际生产路径是 recipes 里以 `balloons_empty` 为材料的配方，而非本组件。[配方路径](../../../raw/current/scripts/recipes.lua#L372-L377)
- 重要副作用：仅生成实体并设坐标；世界气球数量上限的说明保留在注释里，计时移除逻辑整段被注释掉，生成的气球不会自动飞走。[注释](../../../raw/current/scripts/components/balloonmaker.lua#L9-L13)
- 修改边界：无字段、无存读；ACTIONS.MAKEBALLOON 虽有定义、fn 与 SGwilson 动作处理，但 componentactions.lua 中不存在 balloonmaker 条目，原版 UI 不会提供该动作。[动作处理](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1040)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `MakeBalloon(x, y, z)` | `x, y, z: number` | SpawnPrefab(`balloon`) 成功后 `Transform:SetPosition(x,y,z)`；不扣资源、无冷却、无返回值。定时移除代码被注释，注释称先依赖世界气球上限。[实现](../../../raw/current/scripts/components/balloonmaker.lua#L5-L15) |

## 典型接入

### 原版 MAKEBALLOON 动作的预期调用方式（当前不可达）

- 前提与选择：动作 fn 要求 doer 有 `balloonomancer` 标签（维斯）且 invobject 有本组件；扣 SANITY_TINY 后按朝向左右偏移约 0.5 距离生成气球。[动作 fn](../../../raw/current/scripts/actions.lua#L831-L856)
- 生效与撤销：SGwilson 保留 `makeballoon` 状态与客户端处理；但动作从未被 componentactions 插入玩家动作表，需要模组自行登记或直接调用组件方法。[SG 状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1040)
- 原版依据：维斯实际获得气球靠 recipes.lua 的 balloonomancy 配方组（balloon、balloonspeed、balloonparty 等），材料含 `balloons_empty` 与理智。[配方](../../../raw/current/scripts/recipes.lua#L372-L377)

## 常见陷阱与限制

- **以为给物品挂组件就会出现"充气"动作：** componentactions.lua 无任何 balloonmaker／balloon 相关注册；MAKEBALLOON 在本 build 是无入口的遗留动作，组件注释与代码状态一致（弃用）。[动作定义](../../../raw/current/scripts/actions.lua#L454) [fn](../../../raw/current/scripts/actions.lua#L831-L856)
- **以为生成的气球带飞行计时：** `timer:StartTimer("flyoff", ...)` 整段在注释中，MakeBalloon 生成的气球是普通实体，由世界的气球上限逻辑（其他系统）约束；复用时需自行处理清理。[注释](../../../raw/current/scripts/components/balloonmaker.lua#L9-L13)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（17 行），核对 balloons_empty 弃用注释、MAKEBALLOON 定义与 fn、SGwilson 处理、recipes 替代路径；grep 确认 componentactions.lua 无 balloonmaker 条目。未审阅气球世界数量上限的具体实现位置。
- 结构检查：2026-10-28 validate_page.py 通过（balloonmaker.md）；validate_source_links.py --strict 本页全部源码锚点通过；本页无完整 Lua 示例。
- 游戏运行：未运行；MakeBalloon 生成与维斯配方路径未实测。
