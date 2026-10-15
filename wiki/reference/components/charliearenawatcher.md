---
title: "charliearenawatcher"
created: "2026-10-24"
updated: "2026-10-28"
type: "component"
summary: "查理Boss竞技场 FX 实体上的双端观察器：每帧检测玩家进出竞技场并广播 changearea，同时管理画面渐隐高度与粒子清理；判定本身委托 Map:IsPointInCharlieBossArena。"
aliases: ["查理竞技场进出检测", "竞技场 changearea 广播", "charliearenawatcher"]
tags: ["component", "server-client", "ui"]
dst_build: "steam-buildid:26230496"
sources: ["components/charliearenawatcher.lua", "prefabs/charlie_boss_trial.lua", "components/map.lua", "entityscript.lua", "components/hounded.lua", "components/shadowcreaturespawner.lua", "components/areaaware.lua", "components/playervision.lua", "tuning.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# charliearenawatcher

## 快速判断

- 职责与适用：挂在 charlie_boss_trial（竞技场地面 FX 实体，带 FX/NOCLICK 标签）上、双端运行的观察器：构造时按 ismastersim 选定 Master/Client 两套更新与移除实现，并 `StartUpdatingComponent` 每帧轮询玩家位置，进出竞技场时对玩家广播 `changearea` 并做视觉切换。[components/charliearenawatcher.lua:1](../../../raw/current/scripts/components/charliearenawatcher.lua#L1 [prefabs/charlie_boss_trial.lua:1045](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L1045
- 优先使用：判断某点是否在竞技场一律用 `Map:IsPointInCharlieBossArena(x, y, z)`——源码注释明确本组件不提供 IsInArena，只负责"广播与伴随变化"。[components/charliearenawatcher.lua:19](../../../raw/current/scripts/components/charliearenawatcher.lua#L19 [components/map.lua:1588](../../../raw/current/scripts/components/map.lua#L1588
- 重要副作用：进/出竞技场各推一次玩家事件 `changearea`（与 areaaware 同名同义，源码注释称大量系统监听该事件）；例如 playervision 借此在竞技场内强制梦魇视野。[components/charliearenawatcher.lua:47](../../../raw/current/scripts/components/charliearenawatcher.lua#L47 [components/areaaware.lua:59](../../../raw/current/scripts/components/areaaware.lua#L59 [components/playervision.lua:101](../../../raw/current/scripts/components/playervision.lua#L101
- 修改边界：宿主是 FX 实体；视觉调用（`SetUndergroundFadeHeight`、`VFXEffect:ClearAllParticles`）在主端仅非专用服务器执行，客户端恒执行；组件不提供任何查询接口。[components/charliearenawatcher.lua:48](../../../raw/current/scripts/components/charliearenawatcher.lua#L48 [components/charliearenawatcher.lua:65](../../../raw/current/scripts/components/charliearenawatcher.lua#L65

## 关键接口

### 双端构造分工

| 字段／方法 | Master sim | Client |
|---|---|---|
| `self.OnUpdate` | `OnUpdate_Master`：遍历 `AllPlayers`，用 `playersinarena`（按玩家实例索引的布尔表）记录状态 | `OnUpdate_Client`：只跟踪 `ThePlayer` 的 `inarena` 布尔值 |
| `self.OnRemoveEntity` | `OnRemoveEntity_Master`：延迟 1 tick 清理所有仍在场玩家 | `OnRemoveEntity_Client`：只处理 `ThePlayer` |
| 视觉调用 | 仅 `not TheNet:IsDedicated()` 分支执行 | 恒执行（客户端必然非专用） |

构造函数把两套实现直接赋给 `self.OnUpdate`/`self.OnRemoveEntity` 后 `StartUpdatingComponent`，因此运行期不切换端。[components/charliearenawatcher.lua:6](../../../raw/current/scripts/components/charliearenawatcher.lua#L6 [components/charliearenawatcher.lua:16](../../../raw/current/scripts/components/charliearenawatcher.lua#L16

### 进出与移除流程

- 进入（Master，[components/charliearenawatcher.lua:44](../../../raw/current/scripts/components/charliearenawatcher.lua#L44）：`IsPointInCharlieBossArena` 为真且未记录 → 置 `playersinarena[v]=true`、推 `v:PushEvent("changearea")`（不带数据）、非专用时 `SetUndergroundFadeHeight(0.001)` 关闭洞穴渐隐。Client 路径对 `ThePlayer` 相同且无条件做视觉切换。[components/charliearenawatcher.lua:65](../../../raw/current/scripts/components/charliearenawatcher.lua#L65
- 离开（Master，[components/charliearenawatcher.lua:52](../../../raw/current/scripts/components/charliearenawatcher.lua#L52）：置 false、推 `changearea`（带 `areaaware:GetCurrentArea()` 数据）、非专用时清空宿主实体 VFXEffect 的 0/1/2 号发射器并 `SetUndergroundFadeHeight(5)` 还原洞穴渐隐。[components/charliearenawatcher.lua:54](../../../raw/current/scripts/components/charliearenawatcher.lua#L54 [components/charliearenawatcher.lua:56](../../../raw/current/scripts/components/charliearenawatcher.lua#L56
- 移除（宿主实体被删）：`EntityScript:Remove` 会对带 `OnRemoveEntity` 的组件逐个调用（区别于 `OnRemoveFromEntity`）；Master 路径用 `DoTaskInTime(0)` 延迟 1 tick——源码注释说明不延迟的话 `IsPointInCharlieBossArena` 仍返回 true，清理会失效。[entityscript.lua:1749](../../../raw/current/scripts/entityscript.lua#L1749 [components/charliearenawatcher.lua:22](../../../raw/current/scripts/components/charliearenawatcher.lua#L22

### 与 map 竞技场判定的交互

- `Map:IsPointInCharlieBossArena(x, y, z)`：`VIRTUALROOMSETS.ATRIUM` 虚拟房间集存在且点在集内、且该集不处于大厅状态（`not IsVirtualRoomSetInLobby`）。本组件是调用方之一。[components/map.lua:1588](../../../raw/current/scripts/components/map.lua#L1588
- `Map:GetCharlieBossArenaCenterXZ()`：返回房间集原点图块中心坐标，无原点返回 nil；本组件不调用，调用方是 charlie_boss/shrouden 与 SGcharlie_boss（落点、特效定位）。[components/map.lua:1593](../../../raw/current/scripts/components/map.lua#L1593

### 竞技场内原版联动（均经 Map 判定，非本组件代码）

- 猎犬潮免疫：hounded 的 `CheckForLocationImmunity` 把竞技场内玩家标记为 `"charliearena"`，`ReleaseSpawn` 只对状态为 nil 或 `"land"` 的目标放怪——竞技场内玩家免疫猎犬潮。[components/hounded.lua:631](../../../raw/current/scripts/components/hounded.lua#L631 [components/hounded.lua:460](../../../raw/current/scripts/components/hounded.lua#L460
- 影怪参数：shadowcreaturespawner 在竞技场内启用专属键——`TUNING.SANITYMONSTERS_CHARLIEARENA_MAXPOP=3`、`CHANCES{inc=0.75, dec=0.2}`、`POP_CHANGE_INTERVAL=7 + VARIANCE=7*random`；且竞技场内无视 sanity>0.1 的低强度档（源码注释"在场内恒为最大强度"），仅保留 sanity>0.5 的清怪分支。[components/shadowcreaturespawner.lua:226](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L226 [components/shadowcreaturespawner.lua:231](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L231 [tuning.lua:9779](../../../raw/current/scripts/tuning.lua#L9779
- 天气与地震：`CanPointHaveRain`、`CanPointHaveQuaker`、`CanPointCauseQuaker` 对竞技场内的点恒返回 false。[components/map.lua:956](../../../raw/current/scripts/components/map.lua#L956 [components/map.lua:974](../../../raw/current/scripts/components/map.lua#L974 [components/map.lua:989](../../../raw/current/scripts/components/map.lua#L989
- 视野与幻影：playervision 监听 `changearea`（即本组件广播的事件之一）并在回调里用 `IsPointInCharlieBossArena` 强制梦魇视野；brightmarespawner 对竞技场内玩家跳过 gestalt 生成（build-26148036 变更报告 C-28/C-06 已核验行）。[components/playervision.lua:98](../../../raw/current/scripts/components/playervision.lua#L98 [components/playervision.lua:132](../../../raw/current/scripts/components/playervision.lua#L132

## 典型接入

### 模组响应玩家进出查理竞技场

- 前提与选择：目标模组 `modmain.lua`，双端可用；沿用源码建议的模式——监听与 areaaware 同名的 `changearea`，回调时用 `Map:IsPointInCharlieBossArena` 复查当前位置（事件参数本身不区分是否竞技场触发）。[components/charliearenawatcher.lua:19](../../../raw/current/scripts/components/charliearenawatcher.lua#L19
- 生效与撤销：竞技场进入/退出、宿主移除（Master 延迟 1 tick 补推）都会触发；不依赖竞技场存在时也能收到 areaaware 的常规区域事件。

```lua
AddPlayerPostInit(function(inst)
    inst:ListenForEvent("changearea", function(player)
        local x, y, z = player.Transform:GetWorldPosition()
        local in_arena = GLOBAL.TheWorld.Map:IsPointInCharlieBossArena(x, y, z)
        print("[mymod]", player.name, "in charlie arena:", in_arena)
    end)
end)
```

- 调整与验证：`changearea` 参数有两种形态（进入竞技场时不带数据、离开时带 areaaware 区域数据），消费端需兼容两种。[components/charliearenawatcher.lua:47](../../../raw/current/scripts/components/charliearenawatcher.lua#L47

## 常见陷阱与限制

- **用它做查询**：组件没有 IsInArena/名单查询接口，`playersinarena`/`inarena` 是私有状态；判断一律走 `Map:IsPointInCharlieBossArena`。[components/charliearenawatcher.lua:19](../../../raw/current/scripts/components/charliearenawatcher.lua#L19
- **移除清理依赖 1 tick 延迟**：宿主实体删除当帧地图判定仍为"在竞技场"，Master 移除路径因此 `DoTaskInTime(0)`；自定义竞技场类组件照抄该延迟。[components/charliearenawatcher.lua:22](../../../raw/current/scripts/components/charliearenawatcher.lua#L22
- **VFX 下标与宿主绑定**：清理用的 0/1/2 对应 charlie_boss_trial 的 3 个发射器（后两个源码标注 rock/ember）；替换宿主或改发射器数量需同步。[components/charliearenawatcher.lua:56](../../../raw/current/scripts/components/charliearenawatcher.lua#L56 [prefabs/charlie_boss_trial.lua:832](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L832
- **主端视觉分支看专用服务器**：专用服务器上 Master 路径跳过渐隐/粒子操作，仅推事件；测试视觉行为需非专用主机或客户端。[components/charliearenawatcher.lua:48](../../../raw/current/scripts/components/charliearenawatcher.lua#L48

## 验证记录

- 源码审阅：2026-10-24，Steam buildid 26148036；覆盖组件全部双端实现与移除路径、charlie_boss_trial 宿主与 3 发射器、map 两判定函数与 3 个天气/地震消费点、hounded 免疫标记与放怪门槛、shadowcreaturespawner 竞技场参数分支与 tuning 数值、playervision 消费；依据见上文锚点。未覆盖 charlie_boss/shrouden/SGcharlie_boss 对 GetCharlieBossArenaCenterXZ 的调用细节（超出本页接口范围）与实际运行表现。
- 游戏运行：未运行。
- 结构检查：2026-10-24 执行 validate_page.py（standard 档）与 validate_source_links.py（--strict）通过；页内 Lua 示例经 parser_probe.exe --syntax-only 语法检查通过。

相关：[[reference/components/charlie_tracker]]、[[reference/components/shard_charlieinfo]]（同属查理Boss战线的世界级组件）。
