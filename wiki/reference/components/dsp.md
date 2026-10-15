---
title: "dsp"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "客户端世界的混音器滤镜组件：按季节/温度维护低通高通基线，并经 pushdsp/popdsp 事件栈叠加玩家侧闷音效果。"
aliases: ["dsp", "低通高通滤镜", "闷音效果", "TheMixer DSP"]
tags: [component, mechanism, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/dsp.lua", "components/playerhearing.lua", "prefabs/cave.lua", "prefabs/forest.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# dsp

## 快速判断

- 职责与适用：**仅客户端**（森林、洞穴都在 `not TheNet:IsDedicated()` 分支安装）的世界组件，管理 `TheMixer` 的低通/高通滤镜栈：季节默认层 + 玩家事件叠加层。不产生声音、不管音乐播放，也不进存档。[洞穴安装](../../../raw/current/scripts/prefabs/cave.lua#L339-L346) [森林安装](../../../raw/current/scripts/prefabs/forest.lua#L628-L630)
- 优先使用：给本地玩家加闷音走 `pushdsp`/`popdsp` 事件协议（与原版 playerhearing 同路）；改季节默认值只能改本文件顶部常量表。[协议](../../../raw/current/scripts/components/dsp.lua#L204-L237) [季节常量](../../../raw/current/scripts/components/dsp.lua#L17-L56)
- 重要副作用：滤镜按总线名（`set_music`、`set_sfx/voice` 等）全局生效，影响整个混音而非单个声源；pop 用表身份匹配，需复用同一张表。[栈合并](../../../raw/current/scripts/components/dsp.lua#L81-L127)
- 修改边界：跟随 `playeractivated`/`playerdeactivated` 切换监听对象；玩家注销时清空叠加层并 2 秒过渡回净混音。无 OnSave/OnLoad、无 OnRemoveFromEntity。[激活切换](../../../raw/current/scripts/components/dsp.lua#L274-L289) [注销清理](../../../raw/current/scripts/components/dsp.lua#L251-L272)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `pushdsp`（玩家事件） | data：`{lowdsp=?, highdsp=?, duration=?}`；非 nil 的 lowdsp/highdsp 各压入栈尾并按 duration 过渡刷新混音。叠加层优先于基线（后合并覆盖）。[处理](../../../raw/current/scripts/components/dsp.lua#L204-L212) |
| `popdsp`（玩家事件） | 从栈顶向索引 2 搜索第一个与 data.lowdsp/highdsp **同一张表**的项移除；索引 1 的季节基线永不出栈；无匹配不刷新。[处理](../../../raw/current/scripts/components/dsp.lua#L214-L237) |
| `seasontick`（世界事件） | 进入季节时以 5 秒过渡换默认滤镜：冬季对 movement/player/voice 压 5000 Hz 低通；夏季对同三总线按温度档压高通。[季节处理](../../../raw/current/scripts/components/dsp.lua#L200-L202) [冬季表](../../../raw/current/scripts/components/dsp.lua#L19-L32) |
| `temperaturetick`（玩家事件，仅夏季） | 温度 ≤65/70/75/80/其他 对应高通频率 100/250/500/750/1000 Hz，5 档 0.5 秒过渡；离开夏季注销监听。[分档](../../../raw/current/scripts/components/dsp.lua#L34-L41) [处理](../../../raw/current/scripts/components/dsp.lua#L172-L198) |
| `GetDebugString()` | 输出当前生效的低通/高通总线与频率。[实现](../../../raw/current/scripts/components/dsp.lua#L306-L332) |

### 叠加协议的完整链路

玩家激活时组件监听该玩家的 pushdsp/popdsp，并立即把 `playerhearing:GetDSPTables()` 里已有的层压入栈（[激活同步](../../../raw/current/scripts/components/dsp.lua#L239-L249)）。playerhearing 在装备变化时按预设更新：`mufflehat`（装备带同名标签时全总线 750 Hz 低通）与 `wormdigestionmuffle`（600 Hz、creature 例外 2500），推送时长 0.5 秒（[预设](../../../raw/current/scripts/components/playerhearing.lua#L3-L56)）。构造时把空的季节默认表压在栈底作基线（[基线](../../../raw/current/scripts/components/dsp.lua#L299-L300)）。

## 典型接入

### 为本地玩家叠加一段闷音

- 前提与选择：客户端、非专用服务器、世界已装 dsp（原版森林/洞穴都装）。选事件协议而非直接调 TheMixer，可与季节基线及其他叠加层正确合并、且随玩家注销自动清理。[合并逻辑](../../../raw/current/scripts/components/dsp.lua#L86-L111)
- 生效与撤销：push 即时生效；pop 必须传**同一张表**（身份比较），并自行保证 push/pop 成对。duration 是过渡秒数。[匹配规则](../../../raw/current/scripts/components/dsp.lua#L216-L224)

```lua
-- 客户端（非专用服务器）：给本地玩家叠加一层闷音滤镜
local muffle = { lowdsp = { ["set_music"] = 800, ["set_sfx/voice"] = 800 }, duration = 0.5 }
ThePlayer:PushEvent("pushdsp", muffle)
-- 效果结束后撤销（复用同一张表）：
ThePlayer:PushEvent("popdsp", muffle)
```

- 调整与验证：总线名沿用混音器命名（对照 playerhearing 预设的八条总线）；验证可用调试串查看 LOW/HIGH 行（[调试串](../../../raw/current/scripts/components/dsp.lua#L306-L332)）。低通数值为截止频率（Hz），越小越闷。

## 常见陷阱与限制

- **专用服务器上调用无效：** 组件只在非 dedicated 分支安装，且 TheMixer 是客户端设施；服务端模组 push dsp 事件不会有任何效果。需要服务器通知客户端时先自建网络同步。[安装条件](../../../raw/current/scripts/prefabs/forest.lua#L628-L630)
- **popdsp 语义比想象窄：** 按表身份从顶向下只删一个匹配项，且永远不会动基线（索引 1 起始循环）；同一张表 push 两次需要 pop 两次。传新表去 pop 是无效调用。[循环边界](../../../raw/current/scripts/components/dsp.lua#L217-L233)
- **夏季高通对“低频闷”是反向效果：** 夏季是高通滤镜（去低频），与冬季低通方向相反；想统一“闷”的听感需自行叠加，不能只调季节表。[夏季表](../../../raw/current/scripts/components/dsp.lua#L43-L56)
- **叠加层与装备状态无心跳：** playerhearing 只在 equip/unequip（客户端另有 inventoryclosed）时重算；模组若直接改玩家标签绕过装备事件，不会触发更新（[重算时机](../../../raw/current/scripts/components/playerhearing.lua#L59-L73)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 dsp.lua 与 playerhearing.lua，核验栈合并/身份弹出、季节与温度分档、玩家激活协议与两处安装条件。未运行游戏；TheMixer 的 C++ 侧行为（实际听感、过渡曲线）超出脚本可核验范围。

相关：[[reference/components/ambientlighting]]、[[reference/components/playervision]]。
