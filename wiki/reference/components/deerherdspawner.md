---
title: "deerherdspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级羊鹿群调度器：按秋季召群、冬季迁徙排程，维护活跃鹿表并经事件驱动鹿长角与迁移动画，附带调试召群接口。"
aliases: ["deerherdspawner", "鹿群生成器", "羊鹿群迁徙", "DebugSummonHerd"]
tags: [component, creature, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/deerherdspawner.lua", "components/deerherding.lua", "prefabs/deerspawningground.lua", "prefabs/deer.lua", "prefabs/forest.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# deerherdspawner

## 快速判断

- 职责与适用：森林世界主模拟的鹿群生命周期组件：秋季到点召 5±1 只鹿，冬季排迁徙并触发长角；与 [[reference/components/deerherding]] 分工——本组件管“何时生、何时走”，deerherding 管“群怎么移动”。[断言](../../../raw/current/scripts/components/deerherdspawner.lua#L11) [安装](../../../raw/current/scripts/prefabs/forest.lua#L652)
- 优先使用：调试/脚本召群用源码注释声明的 `DebugSummonHerd(time)`；查询成员用 `GetDeer()`；想监听同一批“鹿生成地面”的世界事件用 `ms_registerdeerspawningground`。[调试接口](../../../raw/current/scripts/components/deerherdspawner.lua#L317-L322) [成员表](../../../raw/current/scripts/components/deerherdspawner.lua#L210-L212)
- 重要副作用：向每只活跃鹿推送 `queuegrowantler`（排迁徙时）与 `deerherdmigration`（迁徙时）事件，deer prefab 自行监听；召群成功会重设世界 deerherding 的群位置。[长角](../../../raw/current/scripts/components/deerherdspawner.lua#L189-L194) [迁徙](../../../raw/current/scripts/components/deerherdspawner.lua#L198-L204) [deer 监听](../../../raw/current/scripts/prefabs/deer.lua#L591-L593)
- 修改边界：更新组件按需启停（无定时任务常驻）；有 OnSave/OnLoad/LoadPostPass（计时器、上次召群日、活跃鹿 GUID 重绑）。[更新停止](../../../raw/current/scripts/components/deerherdspawner.lua#L224-L244) [存读](../../../raw/current/scripts/components/deerherdspawner.lua#L252-L297)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `GetDeer()` | 返回活跃鹿表 `_activedeer`（deer→true/false 的活跃标记，deerherding 会写入 false）。deerherding 与鹿大脑经此读群。[实现](../../../raw/current/scripts/components/deerherdspawner.lua#L210-L212) [deerherding 写活跃](../../../raw/current/scripts/components/deerherding.lua#L149-L151) |
| `SpawnDeer(pos,center)` | 生成 `deer`、随机朝向，并记忆 `herdoffset` knownlocation；登记 onremove/death 监听维护成员表。[实现](../../../raw/current/scripts/components/deerherdspawner.lua#L214-L222) |
| `DebugSummonHerd(time?)` | 设 `_timetospawn=time or 1` 并启动更新，约 time 秒后走正常召群流程；同时记录本次 cycle 防重复排程。源码注释给出世界组件调用式。[实现](../../../raw/current/scripts/components/deerherdspawner.lua#L318-L322) |
| `ms_registerdeerspawningground`（世界事件） | 鹿生成地面 prefab 在加载时推送自身注册；组件去重登记并在其移除时注销。klaussackspawner、wintersurprisespawner 也监听同一事件。[接收](../../../raw/current/scripts/components/deerherdspawner.lua#L67-L82) [推送](../../../raw/current/scripts/prefabs/deerspawningground.lua#L12) [其他消费者](../../../raw/current/scripts/components/klaussackspawner.lua#L194) |
| `OnUpdate(dt)`／`LongUpdate` | 倒计时 `_timetospawn`→SummonHerd 或 `_timetomigrate`→MigrateHerd；两者皆空即停更新。LongUpdate 直接复用 OnUpdate（快进也会消耗计时）。[实现](../../../raw/current/scripts/components/deerherdspawner.lua#L224-L246) |
| `OnSave()`／`OnLoad(data)`／`LoadPostPass` | 保存召群/迁徙计时与 `_prevherdsummonday`（兼容旧键 `_lastherdsummonday`），并返回活跃鹿 GUID 供实体引用重绑；LoadPostPass 恢复成员表并按需重启更新。[存读](../../../raw/current/scripts/components/deerherdspawner.lua#L252-L297) |

### 召群与迁徙排程

- 秋季进入时（`WatchWorldState("isautumn")`，[注册](../../../raw/current/scripts/components/deerherdspawner.lua#L328)）：距上次召群超过一个秋季长度才排队，延迟为秋季长度×天数×0.5（首 cycle）或 0.2，±0.33 随机（[排程](../../../raw/current/scripts/components/deerherdspawner.lua#L168-L179)）。
- SummonHerd：已有 7 只以上直接跳过（`HERD_OVERPOPULATION_SIZE`，[常量](../../../raw/current/scripts/components/deerherdspawner.lua#L17-L22)）；有现存群则优先在群位置附近补齐（玩家 35 半径内时改在 35 半径外找点，[找点](../../../raw/current/scripts/components/deerherdspawner.lua#L87-L118)）；无群则从注册的生成地面挑第一个离玩家 35 以上的（[地面选择](../../../raw/current/scripts/components/deerherdspawner.lua#L120-L129)）。目标 5±1 只，每只 4 半径找可行走点；全部失败且秋季剩余过半才按 1～2 天重试（[召群](../../../raw/current/scripts/components/deerherdspawner.lua#L131-L166)）。
- 冬季迁徙（[排程](../../../raw/current/scripts/components/deerherdspawner.lua#L181-L196)）：有鹿时延迟 0.75×秋季长度±0.1，先推 `queuegrowantler`；到期统一推 `deerherdmigration`，鹿实体据此自行离场（[deer 处理](../../../raw/current/scripts/prefabs/deer.lua#L174)）。
- 初始世界冬季开局（`OnPostInit`，[实现](../../../raw/current/scripts/components/deerherdspawner.lua#L331-L339)）：立即召群并直接排队迁徙。

## 典型接入

### 调试或活动脚本强制召群

- 前提与选择：服务端、森林世界已安装。`DebugSummonHerd` 是组件里唯一为外部调用写明用法的入口；走它可复用成员登记、deerherding 初始化与防重复排程，不必手工 SpawnPrefab "deer"。[注释入口](../../../raw/current/scripts/components/deerherdspawner.lua#L317)
- 生效与撤销：`time` 参数是秒（控制台环境常用 1）；召群结果取决于生成地面与玩家分布，失败会按秋季条件自动重试。撤销无入口——召出的鹿由 death/onremove 自然回收。[重试条件](../../../raw/current/scripts/components/deerherdspawner.lua#L163-L165)

```lua
-- 服务端（控制台或调试命令环境）：立即召唤一次鹿群
local spawner = TheWorld.components.deerherdspawner
if spawner ~= nil then
    spawner:DebugSummonHerd(1) -- 1 秒后执行正常召群流程
end
```

- 调整与验证：验证看世界调试串 `DebugSummonHerd` 后 `_timetospawn` 倒计时与随后成员数（[调试串](../../../raw/current/scripts/components/deerherdspawner.lua#L303-L315)）；若世界无 deerspawningground 且无现存群，召群会因找不到点而重试。

## 常见陷阱与限制

- **召群位置不由你指定：** 公开流程只有“生成地面/现有群附近”两种策略，SpawnDeer 虽公开但手工调用不会登记 deerherding 的群位置（那由 SummonHerd 内的 Init 完成），会造出无群鹿。[Init 时机](../../../raw/current/scripts/components/deerherdspawner.lua#L155-L159)
- **成员表与活跃标记混在一张表：** `_activedeer[deer]` 的值同时承担“存在”（deerherdspawner 写 true）与“是否活跃”（deerherding 写 false）两层语义，遍历判定活跃时必须检查值而非仅键存在。[双写方](../../../raw/current/scripts/components/deerherding.lua#L121-L123)
- **秋季防重复按 cycle 差判断：** `_prevherdsummonday` 初始为 -200 保证首秋必触发；自行调 DebugSummonHerd 也会推进该标记，连续调试可能吃掉正常排程窗口（[标记推进](../../../raw/current/scripts/components/deerherdspawner.lua#L320)）。
- **迁徙只对当时仍在表内的鹿生效：** 排迁徙后新召/新入表的鹿不在既有 `_timetomigrate` 推送对象之外没有补偿逻辑；死亡鹿由 RemoveDeer 即时移出（[移除](../../../raw/current/scripts/components/deerherdspawner.lua#L45-L49)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 deerherdspawner.lua，核验注册事件、召群/迁徙排程、成员表协议、存读重绑、调试入口与 deer/deerspawningground 接线；与 deerherding 页面共用同批核验。未运行游戏；deer 实体的迁徙/长角行为细节归 deer 相关页面。

相关：[[reference/components/deerherding]]、[[reference/components/klaussackspawner]]。
