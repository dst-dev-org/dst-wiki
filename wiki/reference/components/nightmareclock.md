---
title: "nightmareclock：洞穴梦魇周期时钟"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴世界专属相位时钟：calm/warn/wild/dawn 四相位按段长自动轮转，经网络变量同步全端，驱动世界状态变量与梦魇环境音。"
aliases: [梦魇时钟, 噩梦周期, nightmareclock, 洞穴相位]
tags: [component, state-machine, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/nightmareclock.lua", "prefabs/cave_network.lua", "components/worldstate.lua", "prefabs/atrium_gate.lua", "components/charliecutscene.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# nightmareclock（洞穴梦魇周期时钟）

## 快速判断

- 职责与适用：仅洞穴世界挂载（[cave_network.lua:11](../../../raw/current/scripts/prefabs/cave_network.lua#L11）。四相位 `calm→warn→wild→dawn` 循环，段长默认 `NIGHTMARE_SEGS`（12/3/5/2 段，段=`SEG_TIME` 秒）加 `NIGHTMARE_SEG_VARIATION`(3 段) 随机（[常量](../../../raw/current/scripts/components/nightmareclock.lua#L11-L17)、[调谐](../../../raw/current/scripts/tuning.lua#L2772-L2779)）；每相位结束才同步一次网络值，其余时间各端本地推进（[更新](../../../raw/current/scripts/components/nightmareclock.lua#L225-L252)）。
- 优先使用：读当前相位与进度 → 监听世界事件 `nightmareclocktick`（每帧推送 `{phase, timeinphase, time}`）或直接读 [worldstate](../../../raw/current/scripts/components/worldstate.lua#L228) 的 `isnightmare*` 变量；控制相位 → `ms_setnightmaresegs`/`ms_setnightmarephase`/`ms_nextnightmarephase`/`ms_nextnightmarecycle`/`ms_locknightmarephase`（[主端事件](../../../raw/current/scripts/components/nightmareclock.lua#L211-L215)）。
- 重要副作用：相位变更推 `nightmarephasechanged` 并切换梦魇环境音；玩家处于 `Nightmare` 标签区域（areaaware）时才播本地环境音循环（[区域音](../../../raw/current/scripts/components/nightmareclock.lua#L69-L90)）。
- 修改边界：`_lockedphase` 锁相期间忽略一切推进与切换事件（[锁相](../../../raw/current/scripts/components/nightmareclock.lua#L152-L155)）；原版锁相来自远古大门（atrium_gate.lua:207/237/459）与查理过场（charliecutscene.lua:223/244）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| net `_phase` | `net_tinybyte`，dirty `nightmarephasedirty` | 相位索引 1~4；全端监听 dirty 置 `_phasedirty` 再推 `nightmarephasechanged`（[网络变量](../../../raw/current/scripts/components/nightmareclock.lua#L61)、[事件](../../../raw/current/scripts/components/nightmareclock.lua#L254-L258)）。 |
| net `_segs[i]` | 4 个 `net_smallbyte` | 各相位段长（主端 `ms_setnightmaresegs` 可改，[设置](../../../raw/current/scripts/components/nightmareclock.lua#L135-L150)）。 |
| net `_totaltimeinphase`/`_remainingtimeinphase` | `net_float` | 相位总长与剩余（秒）；主端每次换相按段长×SEG_TIME 重算并加随机偏移（[换相](../../../raw/current/scripts/components/nightmareclock.lua#L236-L247)）。 |
| `OnUpdate(dt)` | 组件更新 | 剩余>0 本地推进；主端到 0 换相并补溢出时间；从分片/客户端等主端同步（[实现](../../../raw/current/scripts/components/nightmareclock.lua#L225-L252)）；每帧推 `nightmareclocktick`（[推送](../../../raw/current/scripts/components/nightmareclock.lua#L270)）。 |
| `OnSave/OnLoad` | 存读钩子（仅主端定义） | 存段长、相位、剩余、锁相；读回无 phase 数据时按非零段长推断（[存读](../../../raw/current/scripts/components/nightmareclock.lua#L279-L315)）。 |
| `GetDebugString()` | debug 文本 | 相位名＋剩余（[debug](../../../raw/current/scripts/components/nightmareclock.lua#L321-L323)）。 |

世界状态桥：worldstate 组件监听 `nightmareclocktick` 写 `nightmarephase` 与 `isnightmarecalm/warn/wild/dawn` 变量（[桥接](../../../raw/current/scripts/components/worldstate.lua#L104-L113)）；无时钟的世界该相位恒为 `none`（[默认值](../../../raw/current/scripts/components/worldstate.lua#L220)）。

## 典型接入

### 锁定/推进梦魇相位（原版事件路径）

- 前提与选择：服务端且在洞穴世界（组件存在性先判空）；事件只对 `_ismastersim` 生效。
- 生效与撤销：`ms_locknightmarephase` 传相位名锁定，传 nil 解锁并立即恢复推进（[锁相实现](../../../raw/current/scripts/components/nightmareclock.lua#L183-L192)）；`ms_nextnightmarephase` 直接把剩余时间清零。

```lua
-- 服务端，洞穴世界
if TheWorld.components.nightmareclock then
    TheWorld:PushEvent("ms_nextnightmarephase")   -- 立即进入下一相位
    TheWorld:PushEvent("ms_locknightmarephase", "calm")  -- 锁定为 calm
    TheWorld:PushEvent("ms_locknightmarephase", nil)     -- 解锁
end
```

- 调整与验证：`ms_setnightmaresegs` 传 `{calm=..., warn=..., wild=..., dawn=...}`（段数）改节奏，按剩余比例保持相位进度（[改长](../../../raw/current/scripts/components/nightmareclock.lua#L135-L149)）。

## 常见陷阱与限制

- **相位名是 calm/warn/wild/dawn**：事件 payload 与锁相参数都用这套名字；`nightmarephase` 世界变量在无时钟世界是 `none`，判空须区分（[相位名](../../../raw/current/scripts/components/nightmareclock.lua#L11-L17)）。
- **nightmareclocktick 每帧推送**：高频事件，监听回调务必轻量；只要进度快照用 `nightmarephasechanged`（仅换相时一次）（[频率](../../../raw/current/scripts/components/nightmareclock.lua#L270)）。
- **环境音依赖 areaaware 的 Nightmare 区域**：不在标签区域内客户端不播循环音（[区域判定](../../../raw/current/scripts/components/nightmareclock.lua#L69-L74)），自建洞穴须配 Nightmare 标签区域才有音效。
- **只服务端可写**：五个 `ms_*` 事件处理器都以 `_ismastersim` 条件构造，客户端推了也没效果（[条件构造](../../../raw/current/scripts/components/nightmareclock.lua#L209-L216)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/nightmareclock.lua` 全部 330 行、cave_network 挂载行、worldstate 桥接与默认值、atrium_gate/charliecutscene 锁相调用、tuning NIGHTMARE 常量。未覆盖：SoundEmitter 音效资源细节、`rock_light` 对相位的引用上下文。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
