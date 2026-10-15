---
title: "frograin"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "春季蛙雨调度器：按世界状态窗口为每个玩家定时落蛙，控制总蛙量与本地蛙量，月裂开启时有概率换成月蛙。"
aliases: ["蛙雨", "青蛙雨", "frograin"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/frograin.lua", "prefabs/forest.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# frograin

## 快速判断

- 职责与适用：挂在地表世界实体（[安装](../../../raw/current/scripts/prefabs/forest.lua#L650)，[断言](../../../raw/current/scripts/components/frograin.lua#L7)），在"春＋雨＋降水率／湿度达标"窗口内给每个在线玩家排蛙雨计时；对象是 `frog`／`lunarfrog` 的落蛙生成与追踪。
- 优先使用：调参数改 `TUNING.FROG_RAIN_*`（[常量](../../../raw/current/scripts/tuning.lua#L1086-L1096)）；运行期改节奏用 `SetSpawnTimes`，改全局蛙上限用 `SetMaxFrogs`（[L194-L201](../../../raw/current/scripts/components/frograin.lua#L194-L201)）。
- 重要副作用：生成的蛙 `persists = false`（不进存档），睡着即删除；月裂门户开启时按 `FROG_RAIN_LUNARFROG_CHANCE` 概率落月蛙（[落蛙](../../../raw/current/scripts/components/frograin.lua#L51-L64)）。
- 修改边界：仅主机模拟（断言）；只在春季生效，`precipitationrate > 0.55` 且 `moistureceil > 2500` 才开窗（[ToggleUpdate](../../../raw/current/scripts/components/frograin.lua#L97-L124)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetSpawnTimes(times)` | times 为 `{min, max}` 秒表 | 覆盖 `TUNING.FROG_RAIN_DELAY`（默认 0.1–2 秒）并强制重排（[L194-L197](../../../raw/current/scripts/components/frograin.lua#L194-L197)）。 |
| `SetMaxFrogs(max)` | 数字 | 直接覆盖本轮雨的 `_frogcap`；下次 `OnIsRaining` 会按概率重算（[L199-L201](../../../raw/current/scripts/components/frograin.lua#L199-L201)、[L136-L143](../../../raw/current/scripts/components/frograin.lua#L136-L143)）。 |
| `StartTracking` / `StopTracking` | `StartTrackingFn` 的冒号包装 | 追踪蛙的 persists 并在 entitysleep 时移除；`StopTracking` 自带 FIXME"无人调用"（[L203-L229](../../../raw/current/scripts/components/frograin.lua#L203-L229)）。 |
| `OnSave` / `OnLoad` | 仅存 `frogcap` | 生成节奏与世界状态窗口不持久化，读档后重算（[L235-L246](../../../raw/current/scripts/components/frograin.lua#L235-L246)）。 |

## 生成规则（关键流程）

1. 开窗：`isspring && israining && precipitationrate > FROG_RAIN_PRECIPITATION(0.55) && moistureceil > FROG_RAIN_MOISTURE(2500)`，进入更新并给全部玩家排计时（[L97-L124](../../../raw/current/scripts/components/frograin.lua#L97-L124)）。
2. 单场雨的蛙帽：开始下雨时掷 `FROG_RAIN_CHANCE(0.16)`，命中则 `_frogcap = random(FROG_RAIN_LOCAL_MIN 12, FROG_RAIN_LOCAL_MAX 35)`，否则为 0（[L136-L143](../../../raw/current/scripts/components/frograin.lua#L136-L143)）。
3. 每次落蛙：随机 0.1–2 秒，检查全局 `_frogs < FROG_RAIN_MAX(300)` 且玩家 50 半径内蛙数 `< _frogcap`，在 60 半径内取地面有效点，蛙 `persists=false`、随机朝向、`sg:GoToState("fall")`、Teleport 高度 35（[L67-L80](../../../raw/current/scripts/components/frograin.lua#L67-L80)、[落蛙](../../../raw/current/scripts/components/frograin.lua#L55-L64)）。
4. 月蛙替换：`riftspawner:IsLunarPortalActive()` 时按玩家掷 `TryLuckRoll(player, 0.6, LuckFormulas.RiftPossession)` 决定 `lunarfrog`（[L51-L53](../../../raw/current/scripts/components/frograin.lua#L51-L53)；[常量](../../../raw/current/scripts/tuning.lua#L7955)）。
5. 清理：蛙 entitysleep 后下一帧删除；玩家离开取消其计时（[L126-L130](../../../raw/current/scripts/components/frograin.lua#L126-L130)、[L157-L165](../../../raw/current/scripts/components/frograin.lua#L157-L165)）。

## 常见陷阱与限制

- **参数是双层限制**：`SetMaxFrogs` 改的是全局 `_frogcap`，而 `_frogcap` 在每次开始下雨时被本地随机值覆盖；要稳定改单场蛙量需改 `FROG_RAIN_LOCAL_MIN/MAX` 调 tuning 而非只调接口。
- **frogcap 的存档局限**：OnSave 只存 `_frogcap`，OnLoad 后立即 `ToggleUpdate(true)` 重排；若存档时在窗口外，读档后窗口判定重新开始，不恢复"这场雨还剩多少蛙"。
- **追踪表泄漏边界**：蛙的移除依赖 `entitysleep`／`onremove`／`enterlimbo` 监听；`StopTracking` 死代码不影响运行（蛙被删时监听随实体销毁）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、forest.lua 挂载行、FROG_RAIN_* 调 tuning 常量段；未覆盖 frog／lunarfrog 的 fall 状态内部与 riftspawner 细节。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/birdspawner]]（同为世界实体季节生成器）、[[reference/components/wildfires]]。
