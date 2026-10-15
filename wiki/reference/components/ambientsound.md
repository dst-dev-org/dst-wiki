---
title: "ambientsound"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界环境音混音器：按玩家周围 11x11 地块统计混合至多 3 路环境声，随季节/降雨/昼夜调参，并叠加海浪与理智/启发循环声。"
aliases: ["环境音", "环境声混合", "ambientsound 组件"]
tags: [component, ui]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/ambientsound.lua", "prefabs/forest.lua", "prefabs/cave.lua", "worldsettings_overrides.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# ambientsound

## 快速判断

- 职责与适用：挂在 TheWorld 上的客户端环境音驱动（forest.lua:628、cave.lua:341、lavaarena.lua:139、quagmire.lua:164 均在 `not TheNet:IsDedicated()` 分支内）；以玩家位置为中心的地块统计决定播放哪些环境声。
- 优先使用：改某地块的环境音 → `overrideambientsound` 事件（`{tile, override}`，:195-197、:265）；调昼夜参数 → `setambientsounddaytime` 事件（:199-218）；音区重算 → `updateambientsoundparams` 事件（:225-231）。
- 重要副作用：常驻两路循环声——SANITY（:288-294、:308-309）与 ENLIGHT（:280-286、:310-311），参数随 `ThePlayer.replica.sanity` 每帧更新（:457-475）。
- 修改边界：组件不决定混音总线（reverb 用 `TheSim:SetReverbPreset`，:252-254）；洞穴世界初始化即 `SetWavesEnabled(false)` 并预设 `"cave"`（cave.lua:342-343）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetReverbPreset` | `(preset) -> nil` | 透传 `TheSim:SetReverbPreset`；构造时设 `"default"`（:274）。 |
| `SetWavesEnabled` | `(enabled) -> nil` | 开关海浪声轨；洞穴世界传 false。:256-258、cave.lua:343 |
| `AMBIENT_SOUNDS`（局部表） | `tile -> {sound, wintersound, springsound, summersound, rainsound}` | 地块→环境声映射（:29-97）；quagmire 模式下 ROAD 被替换（:99-101）。 |
| `OnUpdate` | `(dt) -> nil` | 主循环：重采样混音、渐变音量、更新理智/启发参数（:329-476）；`OnPostInit` 先以 dt=20 快进一次（:319-323）。 |
| `GetDebugString` | `() -> string` | 调试输出当前混音、雨态、季节与参数。:482-493 |

## 关键流程：混音与衰减

1. 重采样触发：玩家移动超过 2 单位（距离平方 ≥16，:341）后，以玩家为中心扫 11x11 地块（`HALF_TILES`=5，:17）：不可通行地块计 wavecount，其余按 `_tileoverrides[tile] or tile` 查表取声（:349-378）。
2. 选声优先级：大雨且在下雨取 `rainsound`，否则按季节取对应键（`SEASON_SOUND_KEY`，:103-109），再退回 `sound`（:358-361）；每声计数排序后截到 `MAX_MIX_SOUNDS`=3（:18、:380-388），ABYSS 计数减半（:364-366）。
3. 音量：每路 = 计数占比 × 环境总音量（:436-447）；夜间/黄昏按玩家 LightWatcher 衰减——亮于 0.9 降到 0.5、暗于 0.2 全量，中间 outCubic 过渡（:395-406）。
4. 海浪：音量 = wavecount × `WAVE_VOLUME_SCALE`（3/(5²×8)，:19、:390），季节切换换曲（:237-246、:408-428）。
5. 理智两轨：insanity 模式取 `1-GetPercent()`，lunacy 模式取 `GetPercent()`，`dappereffects` 标签使参数平方（:457-475）。

## 典型接入

### 替换或屏蔽某地块的环境音（原版洞穴做法）

- 前提与选择：世界初始化后推 `overrideambientsound`；worldsettings_overrides.lua:3879-3896 的 `areaambientdefault` 用它把地面地块映射到 SINKHOLE/ROCKY、把不可通行映射到 `"ABYSS"`，实现洞穴"清空地面声"。
- 生效与撤销：事件处理直接写 `_tileoverrides[data.tile]`（:195-197），撤销需再推一次传回原 tile；override 值可以是 WORLD_TILES 常量或映射表中的字符串键（如 `"ABYSS"`，:94）。
- 原版依据：worldsettings_overrides.lua:3883-3893、components/ambientsound.lua:265。

```lua
-- 世界初始化后（客户端也需听到效果，事件在世界实体上广播）
AddPrefabPostInit("forest", function(world)
    world:DoTaskInTime(0, function()
        world:PushEvent("overrideambientsound",
            { tile = WORLD_TILES.GRASS, override = WORLD_TILES.FOREST })
    end)
end)
```

- 调整与验证：override 只影响查表不落存档，每次世界加载需重推；验证可对照 `GetDebugString` 输出（:485-490）。

## 常见陷阱与限制

- 专用服务器不挂载：代码读取 `ThePlayer` 与世界实体 SoundEmitter（:330、:440-452），客户端专属；主机玩家（非 dedicated 的 host）同样由本组件出声。
- 无声地块无兜底：查不到 `AMBIENT_SOUNDS[tile]` 的地块不计入混音（:356-357），自定义地块必须自行登记映射。
- 映射表不落存档、不随事件自动重算位置：`_lastplayerpos` 在部分事件里手动置 nil 触发重采样（:180-192），新事件若改混音条件需同样处理。
- 鸟鸣参数依赖 lunarhailbirdsoundmanager：`GetDayTimeParam` 查 `inst.net.components.lunarhailbirdsoundmanager`（:165-174），该组件缺失时按常规昼夜参数。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 499 行；挂载与洞穴预设 forest.lua:628、cave.lua:341-343，override 用例 worldsettings_overrides.lua:3879-3896 逐行核对。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/ambientlighting]]
- [[reference/components/dsp]]
