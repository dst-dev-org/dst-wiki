---
title: "squidspawner"
created: "2026-10-28"
updated: "2026-11-08"
type: component
summary: "森林世界的主模拟乌贼群体生成器：黄昏与夜晚按月相概率在玩家附近海面生成 squidherd 与 squid，鱼群与捕捞网提高概率。"
aliases: ["乌贼生成器", "squidspawner", "刷乌贼", "月相乌贼"]
tags: [component, creature, lifecycle]
dst_build: "steam-buildid:26474873"
sources: ["components/squidspawner.lua", "prefabs/forest.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# squidspawner

## 快速判断

- 职责与适用：森林世界主模拟组件（构造即断言 `TheWorld.ismastersim`），在 master_postinit 安装；不对外提供服务接口，只按世界相位自发调度乌贼群。[断言](../../../raw/current/scripts/components/squidspawner.lua#L6) [安装](../../../raw/current/scripts/prefabs/forest.lua#L645)
- 调度节奏：构造即跑一次 `spawntask` 并随 `phase` 世界状态重跑；仅黄昏／夜晚进入检测并在 `SEG_TIME + rand*SEG_TIME` 后自续，白天取消任务。判相实现读 `state.phase` 字符串而非 `isdusk`/`isnight` 布尔——重跑时机恰在相位变更事件内，此时派生布尔尚未更新（26474873 起改用此写法）。玩家集合由 `ms_playerjoined`／`ms_playerleft` 维护。[任务循环](../../../raw/current/scripts/components/squidspawner.lua#L148-L164) [玩家表](../../../raw/current/scripts/components/squidspawner.lua#L176-L182)
- 重要副作用：判定命中后生成 `squidherd` 群体实体，再按 `math.random(TUNING.SQUID_MIN_COUNT, moonphase_max)` 抽出**投放尝试次数**并逐次找落点：优先取附近鱼群实体，否则在玩家位置附近（`SQUID_SPAWN_RADIUS` 内）找可游泳点，找到才把一只 `squid` 排进 5～7 秒后的生成任务（该次找不到则作废，实际生成量 ≤ 尝试次数）。`SQUID_MIN_COUNT` 只是尝试次数下限（世界设置档位 0/1/2/3/3，默认档 2），不是保证生成量。[成群](../../../raw/current/scripts/components/squidspawner.lua#L93-L110)
- 修改边界：概率、抽样次数下限与月相上限全部来自 `TUNING.SQUID_*`（含 26474873 新键 `SQUID_MIN_COUNT`；构造未缓存，运行期改 TUNING 即生效）；组件本身无存读钩子，乌贼持久化由实体自身承担。[数值](../../../raw/current/scripts/tuning.lua#L6097-L6113)

## 关键流程

每位乱序玩家依次判定：相位为白天（`phase == "day"`）且未强制时跳过；统计 80 半径内 squid 数与可钓鱼数（`SQUID_TEST_RADIUS`/`SQUID_MAX_FISH`）；概率 = `SQUID_CHANCE[moonphase]` × 附近 `oceantrawler` 修正 × 按鱼数 Remap（0→满额），夜晚 ×2，盈月 ÷3 且把「附近已有 squid 数」的门槛压到 2（单群抽样仍用 `moonphase_max`）；过 `TryLuckRoll(LuckFormulas.SquidHerdSpawn)` 后成群，再对 40 单位内的其他玩家去重。[检测主体](../../../raw/current/scripts/components/squidspawner.lua#L54-L125) [网修正](../../../raw/current/scripts/components/squidspawner.lua#L34-L42)

月相参数（tuning.lua）：上限 new 6 / quarter 3 / half 3 / threequarter 2 / full 0；概率 0.25 / 0.125 / 0.065 / 0.03 / 0——默认档满月日完全不刷（always 世界设置档把 full 改成立概率 0.065、上限 3，见 [R-2](../../updates/build-26474873/detail-root.md)）。[数值](../../../raw/current/scripts/tuning.lua#L6097-L6112)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Debug_ForceTestForSquid()` | `-> nil；主模拟` | 以 forcesquid=true 跳过白天门禁与概率骰，直接走成群分支；唯一公开方法，用于调试与验证。注意 `moonphase_max < SQUID_MIN_COUNT` 时（默认档满月 0<2、rare 满月 0<1、often 满月 0<3）成群分支会执行空区间 `math.random(下限, 上限)` 而报错；always 档满月为 3/3，可用。[实现](../../../raw/current/scripts/components/squidspawner.lua#L187-L189) [分支](../../../raw/current/scripts/components/squidspawner.lua#L93) |
| `inst.spawntask` | 函数字段 | 构造把内部 `spawntask` 挂到实体上供任务自续引用；不是稳定公开接口。[挂载](../../../raw/current/scripts/components/squidspawner.lua#L176-L177) |

## 典型接入

### 调试与验证乌贼生成

- 前提与选择：主模拟端控制台或模组代码；强制路径仍受地点可游泳性约束（找不到水面偏移则该次不投放）；满月等 `SQUID_MAX_NUMBERS[moonphase] < SQUID_MIN_COUNT` 的档位组合下不要调用（抽样为空区间）。[落点](../../../raw/current/scripts/components/squidspawner.lua#L101-L109)

```lua
local function DebugSpawnSquid()
    local spawner = TheWorld.components.squidspawner
    if TheWorld.ismastersim and spawner ~= nil then
        spawner:Debug_ForceTestForSquid()
    end
end
```

- 调整与验证：想改频率先看 `SQUID_CHANCE`／`SQUID_MAX_NUMBERS`／`SQUID_TEST_RADIUS`/`SQUID_MAX_FISH`；捕捞网的额外修正由 `oceantrawler:GetOceanTrawlerSpawnChanceModifier` 提供。[网侧](../../../raw/current/scripts/components/squidspawner.lua#L34-L42)

## 常见陷阱与限制

- **白天不是安全期：** 白天只跳过自然检测，已投放的乌贼照常存在；盈月概率被压到 1/3、且「附近 squid < 2」才放行，但不是零。[盈月分支](../../../raw/current/scripts/components/squidspawner.lua#L86-L91)
- **`SQUID_MIN_COUNT` 是尝试次数不是生成量：** 每次尝试都要找到可游泳落点才排生成任务，实际数量 ≤ 抽样值；且上限小于下限时该抽样为空区间（默认档满月 0<2），自然路径靠概率 0 与「squid 数 < 上限」门槛规避，`Debug_ForceTestForSquid` 会直接踩到。[抽样](../../../raw/current/scripts/components/squidspawner.lua#L93-L110)
- **判定以玩家为锚：** 玩家离海远或玩家列表为空时整轮无检测；两玩家相距 40 单位内只判第一个。[去重](../../../raw/current/scripts/components/squidspawner.lua#L117-L123)
- **无存读钩子：** 组件不存档；载入后靠下一次 `spawntask` 自行恢复调度，不要指望它记录半成品 herd。[循环](../../../raw/current/scripts/components/squidspawner.lua#L148-L164)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（190 行）、forest.lua 安装点与 tuning.lua 月相数值段。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。

相关：[[reference/components/herd]]
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
- 2026-11-08 build 26474873：语义修订——判相实现改读 `state.phase` 字符串（相位事件内派生布尔未更新）、新增 `SQUID_MIN_COUNT`（投放尝试次数下限，抽样值不是保证生成量）并登记世界设置档位；锚点迁移 8 处。依据：[C-2](../../updates/build-26474873/detail-components.md)、[R-1](../../updates/build-26474873/detail-root.md)；本页升版与批次记录见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-01）。
