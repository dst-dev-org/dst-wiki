---
title: "lavaarenamusic"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "熔炉竞技场 BGM 组件：挂世界 net 实体，用 net_tinybyte 关卡号在双端切换背景音乐。"
aliases: ["熔炉音乐", "竞技场BGM", "lavaarenamusic"]
tags: [component, network]
dst_build: "steam-buildid:26230496"
sources: ["components/lavaarenamusic.lua", "prefabs/lavaarena_network.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# lavaarenamusic

## 快速判断

- 职责与适用：熔炉（LavaArena）专用世界级组件，挂在世界 net 实体 `lavaarena_network` 上（[挂接](../../../raw/current/scripts/prefabs/lavaarena_network.lua#L55)），按关卡 netvar 播放对应战斗 BGM；只含播放逻辑，不含关卡切换决策。
- 优先使用：客户端听 `leveldirty` 即可生效——音乐在所有客户端各自播放，不依赖 RPC（[netvar](../../../raw/current/scripts/components/lavaarenamusic.lua#L30-L33)）。
- 重要副作用：播放走 `TheFocalPoint.SoundEmitter` 并用 `"bgm"` 声道，切换前先 KillSound 防叠加（[切换](../../../raw/current/scripts/components/lavaarenamusic.lua#L39-L47)）。
- 修改边界：监听 TheWorld 的 `playeractivated`／`playerdeactivated` 维护当前焦点玩家，玩家全部失活时停止播放（[生命周期](../../../raw/current/scripts/components/lavaarenamusic.lua#L49-L69)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `MUSIC` | `table`，索引 1 = `"dontstarve/music/lava_arena/fight_1"` | 关卡号到音效事件的映射表；本快照仅一首（[表](../../../raw/current/scripts/components/lavaarenamusic.lua#L11-L14)）。 |
| `_netvars.level` | `net_tinybyte`，脏事件 `"leveldirty"` | 关卡号网络变量，组件内只读消费（[定义](../../../raw/current/scripts/components/lavaarenamusic.lua#L30-L33)）。 |
| `master_postinit`（快照外） | `event_server_data("lavaarena", "components/lavaarenamusic")` | master 端初始化与 level 写入者在 `lavaarena_event_server/`，不在本快照内（[加载机制](../../../raw/current/scripts/entityscript.lua#L77-L87)）。 |

## 典型接入

### 宿主与播放链路（原版参照）

- 前提与选择：`lavaarena_network` 创建时断言自身为 `TheWorld.net`、`persists=false`、加 `CLASSIFIED`（[宿主前置](../../../raw/current/scripts/prefabs/lavaarena_network.lua#L41-L48)）；组件构造时向 TheWorld 注册玩家激活事件。
- 生效与撤销：master 经 event_server_data 拿到 `master_postinit` 初始化网络写入（[调用](../../../raw/current/scripts/components/lavaarenamusic.lua#L79-L81)）；客户端 `OnLevelDirty` 换曲（[处理](../../../raw/current/scripts/components/lavaarenamusic.lua#L39-L47)）。

```lua
-- 模仿组件内部的客户端换曲逻辑（供自建同类世界 BGM 组件参考）
local MUSIC = { "dontstarve/music/lava_arena/fight_1" }

local function OnLevelDirty(inst)
    TheFocalPoint.SoundEmitter:KillSound("bgm")
    local sound = MUSIC[1] -- 组件内为 MUSIC[_netvars.level:value()]
    if sound ~= nil then
        TheFocalPoint.SoundEmitter:PlaySound(sound, "bgm")
    end
end
```

- 调整与验证：想扩 BGM 需同时改 `MUSIC` 表与写 level 的服务端代码（快照外）。

## 常见陷阱与限制

- **服务端初始化不在快照内**：`event_server_data` 从 `lavaarena_event_server/` 目录 require，本快照未包含该目录，无法核验谁在何时写 `_netvars.level`（[加载机制](../../../raw/current/scripts/entityscript.lua#L77-L87)）。
- **只支持单播放声道**：统一 `"bgm"` key；多音乐并行需自行换 key，否则互相 Kill（[切换](../../../raw/current/scripts/components/lavaarenamusic.lua#L41-L44)）。
- **组件本身无公开方法**：除 netvar 与事件监听外不暴露接口，定制应改编译进 `MUSIC` 表的内容而非调用组件。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、lavaarena_network 宿主、event_server_data 加载机制；master 端写入逻辑未核验（快照外）。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
