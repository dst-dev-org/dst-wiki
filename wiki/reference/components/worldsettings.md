---
title: "worldsettings"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界设置内存表组件：监听 ms_setworldsetting 事件记录 spawn_mode、resource_renewal 等设置值，供游戏模式查询。"
aliases: ["世界设置", "worldsettings", "ms_setworldsetting 存储"]
tags: [component, data]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/worldsettings.lua", "prefabs/world.lua", "worldsettings_overrides.lua", "gamemodes.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# worldsettings

## 快速判断

- 职责与适用：挂在 TheWorld 上的键值表，只做两件事：把 `ms_setworldsetting` 事件的 `{setting, value}` 写入内存、按名读取。当前原版读者是 gamemodes 的游戏模式回退查询。[读取方](../../../raw/current/scripts/gamemodes.lua#L3-L9)
- 优先使用：需要读世界级开关（`spawn_mode`、`resource_renewal`、`ghost_sanity_drain`、`ghost_enabled`、`portal_rez`、`reset_time`）时经 `TheWorld.components.worldsettings:GetSetting`；自定义设置键也可复用同一通道。[事件写入](../../../raw/current/scripts/worldsettings_overrides.lua#L3816-L3859)
- 重要副作用：纯内存，无 OnSave/OnLoad；数值在每次会话由世界生成选项经 worldsettings_overrides 重新推送建立。
- 修改边界：只对 `ms_setworldsetting` 响应；与 `worldsettingstimer`（怪物刷新计时开关）是不同组件，不要混用。[对照挂载](../../../raw/current/scripts/prefabs/world.lua#L529)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetSetting(setting)` | `-> value \| nil` | 返回 `self.settings[setting]`，未设置过返回 nil。[实现](../../../raw/current/scripts/components/worldsettings.lua#L16-L18) |
| `SetSetting(setting, value)` | `-> nil` | 直接写表；事件处理器 `OnSetWorldSetting` 即转调本方法。[实现](../../../raw/current/scripts/components/worldsettings.lua#L20-L22) |
| `ms_setworldsetting`（监听） | `data: {setting: string, value: any}` | 构造时注册在 TheWorld 上；推送方为 worldsettings_overrides 的难度应用函数。[监听](../../../raw/current/scripts/components/worldsettings.lua#L13) |

## 典型接入

### 读取原版世界设置或新增自定义键（原版参照）

- 前提与选择：服务端；`gamemodes.lua` 的 `GetWorldSetting` 先取组件再回退默认值，是推荐的容错读法。[GetWorldSetting](../../../raw/current/scripts/gamemodes.lua#L3-L9)
- 生效与撤销：写入口是 `TheWorld:PushEvent("ms_setworldsetting", {setting=..., value=...})`；原版六个设置键由难度函数推送（spawnmode L3816、basicresource_regrowth L3823、ghostsanitydrain L3830、ghostenabled L3836、portalresurection L3842、resettime L3859）。[推送块](../../../raw/current/scripts/worldsettings_overrides.lua#L3814-L3860)
- 原版依据：world.lua 在 worldstate 之前挂载，属于世界初始化的固定一环。[挂载](../../../raw/current/scripts/prefabs/world.lua#L529)

```lua
-- 服务端：读取（带默认值回退）与写入
local worldsettings = TheWorld.components.worldsettings
local spawn_mode = worldsettings and worldsettings:GetSetting("spawn_mode") or "fixed"
TheWorld:PushEvent("ms_setworldsetting", { setting = "my_mod_setting", value = true })
```

- 调整与验证：自定义键写入后同会话可读；组件无持久化，跨会话需在世界生成选项或每次加载时重新推送。

## 常见陷阱与限制

- **期待从组件改世界选项：** `SetSetting` 只改内存表，不影响世界生成选项 UI，也不触发其他组件的联动（联动由各 `ms_*` 事件自行处理，如 `ms_enableresourcerenewal` 与本组件并存）。[并存推送](../../../raw/current/scripts/worldsettings_overrides.lua#L3822-L3824)
- **与世界生成 customize 混淆：** `map/customize.lua` 的选项列表是生成期数据；本组件是运行期快照，读不到未推送过的选项键。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（24 行），核对 world.lua 挂载、worldsettings_overrides.lua 六处推送、gamemodes.lua 唯一原版读者；grep 确认 `GetSetting` 消费者仅 gamemodes.lua。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；自定义键的会话内读写未实测。
