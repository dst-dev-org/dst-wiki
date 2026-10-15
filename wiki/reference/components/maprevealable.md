---
title: "maprevealable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "实体全图图标挂载器：按揭示来源生成 globalmapicon 跟随实体上小地图，支持公共揭示、私人揭示与周期刷新。"
aliases: ["地图图标揭示", "小地图标记", "maprevealable"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/maprevealable.lua", "prefabs/atrium_key.lua", "prefabs/globalmapicon.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# maprevealable

## 快速判断

- 职责与适用：挂在需要在地图上常显图标的实体（atrium_key、chester、glommer、vault_key、woby 等十余种，[装配例](../../../raw/current/scripts/prefabs/atrium_key.lua#L141-L143)），为宿主生成跟随型 `globalmapicon`；对象是全图／私人地图图标。
- 优先使用：有固定来源用 `AddRevealSource(source, restriction)`；图标样式用 `SetIcon`／`SetIconPriority`／`SetIconTag`（[L18-L61](../../../raw/current/scripts/components/maprevealable.lua#L18-L61)）。
- 重要副作用：构造即启动 1.5s 周期刷新任务，扫描 30 半径内 `maprevealer` 标签实体决定公共／私人揭示（[L15](../../../raw/current/scripts/components/maprevealable.lua#L15)、[L189-L237](../../../raw/current/scripts/components/maprevealable.lua#L189-L237)）。
- 修改边界：揭示图标不随宿主存档（图标 prefab 自管）；组件移除／宿主移除都走 `Stop` 清理（[L253-L271](../../../raw/current/scripts/components/maprevealable.lua#L253-L271)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddRevealSource(source, restriction)` / `RemoveRevealSource(source)` | source 实体或标识；restriction 字符串 | 有任一来源即开始揭示；全来源带 restriction 时图标加 `MiniMapEntity:SetRestriction`（如 `"nightmaretracker"` 只给持钥者看，[L67-L88](../../../raw/current/scripts/components/maprevealable.lua#L67-L88)、[用例](../../../raw/current/scripts/prefabs/atrium_key.lua#L46-L59)）。 |
| `AddPrivateSource(source)` / `RemovePrivateSource(source)` | source 为玩家 | 为每个私人来源生成独立图标，restriction 用 `"player_"..userid` 区分分片可见性（[L90-L108](../../../raw/current/scripts/components/maprevealable.lua#L90-L108)、[L158-L178](../../../raw/current/scripts/components/maprevealable.lua#L158-L178)）。 |
| `SetIcon(iconname)` / `SetIconPriority(priority)` / `SetIconPrefab(prefab)` | 样式注入 | 未生成图标时先记配置，生成时套用；iconprefab 默认 `"globalmapicon"`（[L4-L7](../../../raw/current/scripts/components/maprevealable.lua#L4-L7)、[L18-L44](../../../raw/current/scripts/components/maprevealable.lua#L18-L44)）。 |
| `SetOnIconCreatedFn(fn)` | fn(inst, icon) | 图标生成后、`TrackEntity` 前回调，用于挂额外部件（[L63-L65](../../../raw/current/scripts/components/maprevealable.lua#L63-L65)、[L142-L144](../../../raw/current/scripts/components/maprevealable.lua#L142-L144)）。 |
| `Refresh()` / `SetOnRefreshFn(fn)` | 周期回调 | 扫描 `maprevealer` 实体（[[reference/components/maprevealer]] 周期加标签）：公共揭示者清私人来源；私人拥有者映射到对应玩家图标（[L189-L241](../../../raw/current/scripts/components/maprevealable.lua#L189-L241)）。 |
| `Start(delay)` / `Stop()` | 秒 | 启停周期任务；Stop 同时清公共与私人来源（[L247-L262](../../../raw/current/scripts/components/maprevealable.lua#L247-L262)）。 |

## 典型接入

### 给随行生物加全图标记（原版同构）

- 前提与选择：主机侧加组件并加一个揭示来源即常显；atrium_key 用自身作来源并把 restriction 设为 `"nightmaretracker"`，容器内／被持有收起时 `Stop`、落地 `Start`（[装配](../../../raw/current/scripts/prefabs/atrium_key.lua#L141-L143)、[收放](../../../raw/current/scripts/prefabs/atrium_key.lua#L45-L59)）。

```lua
AddPrefabPostInit("my_pet", function(inst)
    if not TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("maprevealable")
    inst.components.maprevealable:SetIcon("my_pet.png")
    inst.components.maprevealable:SetIconPriority(10)
    inst.components.maprevealable:AddRevealSource(inst)  -- 无 restriction: 所有人可见
end)
```

- 调整与验证：刷新间隔固定 1.5s（`refreshperiod`，[L4](../../../raw/current/scripts/components/maprevealable.lua#L4)）；私人揭示需要宿主被 `maprevealer` 扫到，见 [[reference/components/maprevealer]] 的范围常量 `PLAYER_REVEAL_RADIUS=30`（[常量](../../../raw/current/scripts/constants.lua#L24)）。

## 常见陷阱与限制

- **Refresh 只在任务活着时生效**：`Refresh` 开头判 `self.task ~= nil`，`Stop` 之后即使周围有 maprevealer 也不会再触发揭示；atrium_key 依赖手动 Start/Stop 配合持有状态（[L190-L193](../../../raw/current/scripts/components/maprevealable.lua#L190-L193)）。
- **来源为字符串时无 onremove 清理**：`AddRevealSource` 仅对 EntityScript 来源监听移除事件；传字符串来源（如内部 `"maprevealer"`）需自行保证增删配对（[L70-L72](../../../raw/current/scripts/components/maprevealable.lua#L70-L72)）。
- **restriction 语义在图标端**：`SetRestriction` 的可见性由客户端地图渲染与 `globalmapicon` 逻辑实现，组件只转发字符串；改可见性需核对图标端（[L145-L148](../../../raw/current/scripts/components/maprevealable.lua#L145-L148)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、atrium_key 装配与收放段、globalmapicon 的 TrackEntity 签名与 PLAYER_REVEAL_RADIUS 常量；未覆盖客户端 MiniMapRenderer 渲染细节与全部 14 个宿主的差异。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/maprevealer]]（揭示来源提供者）、[[reference/components/globaltrackingicon]]（双图标变体）。
