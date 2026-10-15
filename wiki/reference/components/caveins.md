---
title: "caveins"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴世界的落石执行器：接收主分片经 shard_sinkholes 网络同步的蚁狮目标，在洞穴本地重演警告（碎石+镜头摇晃）与攻击（ms_miniquake 落巨石）。"
aliases: ["洞穴落石", "caveins", "蚁狮洞穴报复", "落石警告"]
tags: [component, server-client, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/caveins.lua", "components/shard_sinkholes.lua", "components/sinkholespawner.lua", "components/quaker.lua", "prefabs/cave.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# caveins

## 快速判断

- 职责与适用：仅洞穴（非主分片）安装的洞穴世界主模拟组件（构造即断言 mastersim）；蚁狮目标决策在主分片，本组件只负责在洞穴里"演"出来。[安装](../../../raw/current/scripts/prefabs/cave.lua#L530) [断言](../../../raw/current/scripts/components/caveins.lua#L7)
- 优先使用：不要直接调用本组件触发落石——原版入口是 `ms_miniquake` 世界事件（quaker 组件消费）；本组件的攻击路径也只是构造该事件。[攻击](../../../raw/current/scripts/components/caveins.lua#L72-L82) [quaker 消费](../../../raw/current/scripts/components/quaker.lua#L743)
- 重要副作用：警告目标会被 `ShakeAllCameras`、淋 `cavein_debris` 碎石（追踪玩家 4 单位内）并说 `ANNOUNCE_CAVEIN`；攻击目标触发半径 4、30 发、3.5 秒的小地震，碎片固定 `cavein_boulder`。[警告](../../../raw/current/scripts/components/caveins.lua#L55-L66)
- 修改边界：目标数据跨分片经 `shard_sinkholes` 的 netvar（userhash+state）传输，本组件不感知蚁狮愤怒度等决策细节；组件无 OnRemoveEntity 清理，随世界销毁。[同步](../../../raw/current/scripts/components/shard_sinkholes.lua#L64-L78)

## 关键流程

主分片侧：`sinkholespawner` 决策后推 `master_sinkholesupdate` → shard_sinkholes（主分片）写入 2 组 netvar。洞穴侧：netvar dirty → shard_sinkholes（非主分片）解码推 `secondary_sinkholesupdate` → 本组件 `OnSinkholesUpdate` 把 userhash 映射回本地玩家，attack 立即 `DoTargetAttack`，warn 则登记目标并 `DoTargetWarning`（同一目标的冷却 `warncd` 内不重复警告）。玩家离线/失效时按旧坐标继续执行。[决策源](../../../raw/current/scripts/components/sinkholespawner.lua#L194) [网络中继](../../../raw/current/scripts/components/shard_sinkholes.lua#L40-L78) [洞穴处理](../../../raw/current/scripts/components/caveins.lua#L88-L137)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnUpdate(dt)` | 组件更新 | 有目标时运行：跟踪玩家实时坐标（失效则冻结坐标）、递减 `warncd`。[实现](../../../raw/current/scripts/components/caveins.lua#L152-L163) |
| `OnSave`／`OnLoad` | 标准存读钩子 | 存目标坐标（仅 x/z）；载入对每个存档点 0 帧后直接 `DoTargetAttack`（不再追人）。[实现](../../../raw/current/scripts/components/caveins.lua#L169-L190) |
| `GetDebugString()` | 调试串 | 列出目标与坐标。[实现](../../../raw/current/scripts/components/caveins.lua#L196-L202) |
| 攻击事件 `ms_miniquake` | TheWorld | 参数 `rad=4, minrad=1.5, num=30, duration=3.5, pos, target, debrisfn`；由 quaker 组件实际执行落石。[构造](../../../raw/current/scripts/components/caveins.lua#L72-L82) [消费](../../../raw/current/scripts/components/quaker.lua#L743) |

## 典型接入

### 复现/调试蚁狮洞穴落石

- 前提与选择：洞穴世界主模拟端；直接事件方式绕过分片同步，适合验证表现层。[事件消费](../../../raw/current/scripts/components/quaker.lua#L743)

```lua
local function DebugCaveInAt(pos)
    if TheWorld.ismastersim and TheWorld:HasTag("cave") then
        TheWorld:PushEvent("ms_miniquake", {
            rad = 4, minrad = 1.5, num = 30, duration = 3.5,
            pos = pos, debrisfn = function() return "cavein_boulder", 0 end,
        })
    end
end
```

- 调整与验证：warn 路径（碎石雨＋摇晃＋玩家喊话）走 `secondary_sinkholesupdate`，无法用上述事件单独触发；`DoTargetWarning` 有 `CanPointHaveQuaker` 地点门禁。[警告门禁](../../../raw/current/scripts/components/caveins.lua#L55-L57)

## 常见陷阱与限制

- **只在非主分片生效：** `_ismastershard` 为真时 `OnSinkholesUpdate` 与监听都不存在——地面世界装了也不会做事（原版只装在洞穴）。[门禁](../../../raw/current/scripts/components/caveins.lua#L88-L89) [监听注册](../../../raw/current/scripts/components/caveins.lua#L144-L146)
- **BOULDERS 常量是死的：** 文件顶部 `BOULDERS` 表定义后无任何引用，实际碎片名写死在 `GetDebrisFn`；改碎片类型要改函数而不是该表。[死表](../../../raw/current/scripts/components/caveins.lua#L13-L16) [实际来源](../../../raw/current/scripts/components/caveins.lua#L68-L70)
- **警告有冷却字段但无全局限流：** `warncd` 只对同一目标条目生效（`WARNING_DELAY * 0.8`），新目标立即警告；载入重演的攻击不带警告。[冷却](../../../raw/current/scripts/components/caveins.lua#L65)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（208 行），核验 shard_sinkholes 双侧协议、sinkholespawner 决策源与 quaker 的事件消费。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。

相关：[[reference/components/shard_sinkholes]]
