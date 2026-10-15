---
title: "globaltrackingicon"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "为单个玩家全图追踪移动实体的小地图图标：sender 专属 globalicon 加他人可见的 revealableicon 双图标体系。"
aliases: ["全局追踪图标", "小地图追踪", "globaltrackingicon"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/globaltrackingicon.lua", "prefabs/wx78_backupbody.lua", "prefabs/wx78_drone_scout.lua", "prefabs/wx78_drone_delivery.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# globaltrackingicon

## 快速判断

- 职责与适用：让某玩家（owner）在小地图上全图看到会移动的宿主（WX-78 备用躯壳、无人机等），同时宿主附近的他人能看到 revealable 图标；图标预制体由 globalmapicon.lua 的 MakeGlobalTrackingIcons 体系注册（[头注](../../../raw/current/scripts/components/globaltrackingicon.lua#L1-L4)）。
- 优先使用：宿主构造即 `StartTracking(nil, "prefab名")` 先立图标，指定 owner 后再调一次（[wx78_backupbody](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L950-L951)）。
- 重要副作用：StartTracking 会禁用宿主自身 MiniMapEntity，改由两个图标实体代播（[禁用](../../../raw/current/scripts/components/globaltrackingicon.lua#L23-L25)）；还给宿主装/配 maprevealable 支持远处揭示。
- 修改边界：图标实体的可见性靠 classified owner 与 proxy 机制，非 owner 玩家在全图外看不到 globalicon；组件随宿主移除时走 StopTracking。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartTracking(owner, name)` | `owner: player 或 nil；name: string 或 nil（默认 inst.prefab）` | 生成 `<name>_globalicon`（TrackEntity 绑定 + SetClassifiedOwner）与 `<name>_revealableicon`（parent 宿主 + SetAsNonProxyExcludingOwner）；宿主无 maprevealable 时加装并注册远处图标的 proxy 回调。[实现](../../../raw/current/scripts/components/globaltrackingicon.lua#L17-L50) |
| `StopTracking()` | 无参数 | 恢复 MiniMapEntity、移除两图标、移除 maprevealable、清 owner；也是 OnRemoveFromEntity 钩子。[实现](../../../raw/current/scripts/components/globaltrackingicon.lua#L52-L71) |
| `owner` | `player 或 nil` | 当前追踪归属；传 nil 表示暂不归属但仍立图标（原版备用躯壳用法）。[字段](../../../raw/current/scripts/components/globaltrackingicon.lua#L10) [用法](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L667) |

## 典型接入

### 给可移动储物实体接全图追踪

- 前提与选择：需要同时存在 `<name>_globalicon` 与 `<name>_revealableicon` 两个图标预制体（原版由 globalmapicon.lua::MakeGlobalTrackingIcons 批量注册）；只 StartTracking 不建预制体会 Spawn nil。
- 生效与撤销：StartTracking 即刻生效；换 owner 直接再调 StartTracking 覆盖；彻底撤除用 StopTracking。
- 原版依据：备用躯壳（[安装](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L950-L951)、[owner 换绑](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L649)）；无人机 scout／delivery 同模式（[scout](../../../raw/current/scripts/prefabs/wx78_drone_scout.lua#L316-L317)、[delivery](../../../raw/current/scripts/prefabs/wx78_drone_delivery.lua#L621)）。

```lua
-- 宿主 master_postinit 内（仿 wx78_backupbody）
inst:AddComponent("globaltrackingicon")
inst.components.globaltrackingicon:StartTracking(nil, "my_roaming_thing")

-- 指定归属玩家时（如被拾取者）
inst.components.globaltrackingicon:StartTracking(owner, "my_roaming_thing")
```

- 调整与验证：`name` 决定图标预制体名，必须与 globalmapicon 注册名一致；换名等价于换一套图标体系。

## 常见陷阱与限制

- 重复 StartTracking 不会重建已存在的图标（判 nil 后复用），但会把 globalicon 换绑到新 owner；换 owner 时旧 owner 立即失去全图视角，无过渡（[实现](../../../raw/current/scripts/components/globaltrackingicon.lua#L28-L32)）。
- 图标可见性是网络行为：globalicon 仅 classified owner 可见，revealable 对非 owner 显示；在无 owner（nil）时 SetClassifiedOwner(nil)，全图图标对所有客户端隐藏程度以源码为准，模组应自测（[头注](../../../raw/current/scripts/components/globaltrackingicon.lua#L1-L2)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 wx78_backupbody／scout／delivery 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
