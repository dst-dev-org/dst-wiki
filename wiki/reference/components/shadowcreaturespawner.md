---
title: "shadowcreaturespawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级暗影生物（爬行梦魇/恐怖之喙/海洋梦魇）按玩家理智的人口管理器：目标人口渐进调整、离屏即删、影怪自我进化自动换跟踪。"
aliases: ["shadowcreaturespawner", "暗影生物生成器", "梦魇生成", "影怪人口"]
tags: [component, creature, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/shadowcreaturespawner.lua", "prefabs/shadowcreature.lua", "prefabs/cave.lua", "prefabs/forest.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shadowcreaturespawner

## 快速判断

- 职责与适用：洞穴与森林世界都安装的主模拟组件（[洞穴](../../../raw/current/scripts/prefabs/cave.lua#L520) [森林](../../../raw/current/scripts/prefabs/forest.lua#L660)），为每个玩家维护一个目标人口并渐进逼近：理智 >0.5 无怪、0.1～0.5 最多 1 只、≤0.1 最多 2 只（查理竞技场 3 只），精神疯狂（induced）最多 5 只。仅生成爬行梦魇/恐怖之喙/海洋梦魇，不管 [[reference/components/shadowhandspawner]] 的手与 waveyjones。[断言](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L7)
- 优先使用：调节奏改 `TUNING.SANITYMONSTERS_*`（人口上限、增减概率、间隔）；主动触发走 `SpawnShadowCreature(player, params?)`。[生成入口](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L100-L142) [调参](../../../raw/current/scripts/tuning.lua#L6127-L6147)
- 重要副作用：生成的怪带 `spawnedforplayer` 标记；离屏（entitysleep）下一 tick 即被移除；人口调低时随机挑怪标记 `wantstodespawn`+`persists=false` 由其自行消失；玩家移除时其所有怪同样处置。[离屏移除](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L63-L65) [人口收缩](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L153-L169)
- 修改边界：无 OnSave/OnLoad——人口与跟踪关系不持久化；任务挂玩家实体，玩家离开即取消。[玩家生命周期](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L326-L344)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `SpawnShadowCreature(player, params?)` | params 缺省取该玩家注册表。船+理智 <0.1 → 船半径+3～11 找海面生成 `oceanhorror`，连败 4 次改在可通行点生成陆地怪；陆地按 15 半径 8 次找可行走点生成 `crawlinghorror` 或（理智 <0.1 且运气 0.5）`terrorbeak`。[实现](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L87-L142) [运气回合](../../../raw/current/scripts/tuning.lua#L6147) |
| `ms_exchangeshadowcreature`（世界事件） | 影怪/ocean 影怪自我交换（变体/升级）时推送；组件把跟踪对象从旧怪换成新怪，人口计数不变。[处理](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L74-L85) [推送](../../../raw/current/scripts/prefabs/shadowcreature.lua#L217) |
| `GetDebugString()` | 输出当前管理的影怪总数。[实现](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L364-L372) |

### 人口节奏（UpdatePopulation）

每玩家 poptask 循环：疯狂状态走快速 5～6 秒档、目标上限 5；普通状态按理智分档（[分档](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L220-L261)、[上限与概率](../../../raw/current/scripts/tuning.lua#L6127-L6147)），每次至多 ±1 并经 `TryLuckRoll` 掷增减概率；查理竞技场单独 3 只上限与专属概率、7±7 秒节奏；vault 密室房间 0.2±0.1 秒快速档；非疯狂时 0.1±0.1 秒快速清理档（[节奏调度](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L283-L295)）。目标变化才启动 spawntask，按 `SANITYMONSTERS_SPAWN_INTERVAL(5)+方差(10)` 逐只补齐或收缩（[UpdateSpawn](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L144-L174)）。`inducedinsanity`/`sanitymodechanged` 事件重置计时立即重评（[重置](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L317-L324)）。

## 典型接入

### 调整影怪人口与出现阈值

- 前提与选择：服务端、世界装本组件。改 TUNING 需在世界组件读取前生效——人口逻辑每次调度都重新读 TUNING（与 birdspawner 的构造缓存不同），运行中改也生效，但已在飞的 task 不受影响。[读取时机](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L193-L195) [对比构造缓存](../../../raw/current/scripts/components/shadowhandspawner.lua#L13-L18)
- 生效与撤销：下一轮 poptask 即按新参数评估；想立即响应可对玩家推 `inducedinsanity` 类事件或等自然轮询。[轮询](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L282-L295)

```lua
-- 服务端：抬高影怪人口上限与生成概率（对照原版调参表）
TUNING.SANITYMONSTERS_MAXPOP = { 2, 4 }          -- 原 {1, 2}：中/低理智两档
TUNING.SANITYMONSTERS_CHANCES = {                -- 对应两档的增减概率
    { inc = 1, dec = 0.5 },
    { inc = 1, dec = 0.3 },
}
```

- 调整与验证：概率为每次 poptask 的单次掷骰，体感节奏还受间隔常量影响；验证用 `GetDebugString` 观察数量随理智变化（[调试串](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L364-L372)）。

## 常见陷阱与限制

- **读档后旧怪脱管：** 组件无 OnLoad，重绑不了 `spawnedforplayer` 与 entitysleep 移除监听；读档后旧怪成为“野生”个体（不会自动消失也不计入人口），组件按目标另开新怪。刷怪类模组要自处理野生清理。[无重绑](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L53-L72)
- **wantstodespawn 不是即时删除：** 收缩人口只设标记，实际移除由影怪自身逻辑完成；测试清场时别指望立刻消失。[标记语义](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L162-L166)
- **海上生成有 4 次失败回落：** 连续 4 次找不到海面才尝试陆地落点，且该计数是全组件共享（不分玩家）；近海战斗场景的刷怪位置可能落到岸上。[回落逻辑](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L104-L131)
- **terrorbeak 概率只在极低理智掷：** 生成分支先判理智 <0.1 再掷 0.5 运气；0.1～0.5 区间永远 crawlinghorror。[选怪分支](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L87-L94)
- **竞技场豁免高理智清怪：** 理智 0.1～0.5 时查理竞技场不适用 1 只上限分支（走 ≤0.1 的 3 只规则）；但 >0.5 的清怪分支在竞技场同样生效。[分档注释](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L228-L239)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 shadowcreaturespawner.lua，核验人口分档、节奏调度、海陆生成分支、跟踪与交换协议、存读缺口；核对 shadowcreature/oceanshadowcreature 推送点与 TUNING 表。未运行游戏；影怪个体 AI 与进化条件归实体页面。

相关：[[reference/components/shadowhandspawner]]、[[reference/components/brightmarespawner]]。
