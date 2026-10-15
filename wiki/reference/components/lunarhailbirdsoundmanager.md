---
title: "lunarhailbirdsoundmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月雹鸟事件声效档位的世界网络组件：服务端 SetLevel 经 net_tinybyte 同步，客户端驱动环境音切换。"
aliases: ["月雹鸟音效管理", "雹鸟声档位", "lunarhailbirdsoundmanager"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/lunarhailbirdsoundmanager.lua", "prefabs/forest_network.lua", "components/birdspawner.lua", "components/ambientsound.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# lunarhailbirdsoundmanager

## 快速判断

- 职责与适用：挂在 `TheWorld.net`（forest_network）上的声效状态机，0-3 四档描述月雹鸟事件烈度；非专用服务器才本地发声（[组件](../../../raw/current/scripts/components/lunarhailbirdsoundmanager.lua#L6-L18)）。
- 优先使用：世界组件 birdspawner 按雹鸟阶段调 `SetLevel(level)`（[调用](../../../raw/current/scripts/components/birdspawner.lua#L97-L100)）；读档只需 SetLevel，无存档。
- 重要副作用：档位变化推 `updateambientsoundparams` 世界事件让环境音系统响应；客户端经 `hailbirddirty` 事件自播/停 `lunarhail_event/amb/gestalt_attack_storm`（[PlaySoundLevel](../../../raw/current/scripts/components/lunarhailbirdsoundmanager.lua#L37-L49)）。
- 修改边界：不持久化，读档后由 birdspawner 重新 SetLevel；dedicated 服务器 SetLevel 只写 netvar 不播声。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetLevel(level)` | `level: number`（0-3） | 服务端写 `sound_level` 与 `birds_dropping_param`；非 dedicated 时本地同步播放。[实现](../../../raw/current/scripts/components/lunarhailbirdsoundmanager.lua#L66-L73) |
| `GetIsBirdlessAmbience()` | → `boolean` | `sound_level > 0`，ambientsound 用它判"鸟声已灭"混音（[消费](../../../raw/current/scripts/components/ambientsound.lua#L166-L170)）。[实现](../../../raw/current/scripts/components/lunarhailbirdsoundmanager.lua#L60-L62) |
| `HAIL_SOUND_LEVELS` | 常量表 `NONE=0／SCUFFLES=1／CORPSES=2／NO_AMBIENCE=3` | 档位语义；`self.HAIL_SOUND_LEVELS` 对外暴露。[定义](../../../raw/current/scripts/components/lunarhailbirdsoundmanager.lua#L29-L35) |
| `birds_dropping_param` | `net_tinybyte`，dirty 事件 `hailbirddirty` | 双端同步通道；客户端监听后按 value 复现声效。[构造](../../../raw/current/scripts/components/lunarhailbirdsoundmanager.lua#L15-L16) [监听](../../../raw/current/scripts/components/lunarhailbirdsoundmanager.lua#L56-L58) |
| `sound_level` | `number`，默认 0 | 本地缓存的当前档位。 |

## 典型接入

### 从世界事件驱动雹鸟声档位

- 前提与选择：目标是 forest 类世界（world network 装了组件，[安装](../../../raw/current/scripts/prefabs/forest_network.lua#L21)）；客户端只读，档位迁移一律在服务端发。
- 生效与撤销：SetLevel 立即双端生效；1/2 档循环播风暴底噪并用参数区分，0/3 档 KillSound 收尾。
- 原版依据：birdspawner 的 SetLunarHailSoundLevel 包装（[调用](../../../raw/current/scripts/components/birdspawner.lua#L97-L100)）。

```lua
-- 服务端世界脚本内（如 AddPrefabPostInit 后的自定义事件）
local function SetHailBirdLevel(level)
    if TheWorld.net ~= nil and TheWorld.net.components.lunarhailbirdsoundmanager ~= nil then
        TheWorld.net.components.lunarhailbirdsoundmanager:SetLevel(level)
    end
end
SetHailBirdLevel(TheWorld.net.components.lunarhailbirdsoundmanager.HAIL_SOUND_LEVELS.CORPSES)
```

- 调整与验证：档位含义见 HAIL_SOUND_LEVELS 注释（0 无声、1 空中骚动、2 尸体坠落、3 鸟声消失）；自定义档位序列需自己保证单调，组件不做状态机约束。

## 常见陷阱与限制

- 声效挂在 `TheFocalPoint.SoundEmitter` 而非实体上（[播放](../../../raw/current/scripts/components/lunarhailbirdsoundmanager.lua#L44-L47)），跟玩家视线无关；单独测试时注意 focal point 在专用客户端上的表现差异。
- 组件头注写 "Handled by birdmanager.lua"，实际库内无该文件名；现实调用方是 birdspawner.lua，头注属陈旧注释（[头注](../../../raw/current/scripts/components/lunarhailbirdsoundmanager.lua#L5)）。

## 相关链接

- [[reference/components/birdspawner]]：档位的唯一原版服务端驱动者。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 forest_network／birdspawner／ambientsound 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
