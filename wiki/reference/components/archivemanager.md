---
title: "archivemanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "远古档案馆供电的世界开关：切换全局 ARCHIVES_ENERGIZED 世界状态标签并广播供电/断电事件，供锁箱、吊灯、传送门读取。"
aliases: ["archivemanager", "档案馆供电", "远古档案馆电源"]
tags: [component, event]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/archivemanager.lua", "prefabs/cave.lua", "prefabs/archive_props.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# archivemanager

## 快速判断

- 职责与适用：洞穴世界（`cave.lua:565` 挂载）上的单例开关，记录远古档案馆是否通电。开/关时写 `WORLDSTATETAGS` 的 `ARCHIVES_ENERGIZED` 标签并推世界事件。[切换](../../../raw/current/scripts/components/archivemanager.lua#L34-L44)
- 优先使用：需要"档案馆是否供电"的判断一律读 `TheWorld.components.archivemanager:GetPowerSetting()`（注意组件可能不存在，判空后按断电处理是原版惯例）。[读取示例](../../../raw/current/scripts/prefabs/archive_lockbox.lua#L196-L197)
- 重要副作用：事件名是带历史拼写错误的 `arhivepoweron`/`arhivepoweroff`（少一个 c），监听时必须沿用；标签声明在 `mainfunctions.lua:2395`。[事件拼写](../../../raw/current/scripts/components/archivemanager.lua#L38)
- 修改边界：仅主机（构造即 `assert(TheWorld.ismastersim)`）；无存读钩子——V2C 注释明确存档被删除，供电状态改由 `archive_switch` 的宝石状态在加载后重建。[存档注释](../../../raw/current/scripts/components/archivemanager.lua#L65-L66)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SwitchPowerOn(setting)` | setting: boolean | 仅在状态实际变化时动作：置 `_power_enabled`、`WORLDSTATETAGS.SetTagEnabled("ARCHIVES_ENERGIZED", setting)`、推 `arhivepoweron`/`arhivepoweroff`。[实现](../../../raw/current/scripts/components/archivemanager.lua#L34-L44) |
| `GetPowerSetting()` | `-> boolean` | 返回当前供电布尔。[实现](../../../raw/current/scripts/components/archivemanager.lua#L46-L48) |
| `ARCHIVES_ENERGIZED` | 世界状态标签 | 由本组件写入；`archive_orchestrina_main`、`archive_lockbox`、`archive_chandelier`、`vault_teleporter` 等按其状态改变行为。[声明](../../../raw/current/scripts/mainfunctions.lua#L2395) |
| `arhivepoweron` / `arhivepoweroff` | 世界事件（TheWorld 上） | 供电变化通知；吊灯、档案装置、宝库传送门监听以刷新灯光/状态。[监听示例](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L407-L408) |

## 典型接入

### 宝石插拔驱动供电（原版参照）

- 前提与选择：供电入口在 `archive_props.lua` 的 switch 装置：集齐 3 颗宝石 `SwitchPowerOn(true)`（`archive_props.lua:803-805`），取走宝石 `SwitchPowerOn(false)`（`archive_props.lua:850-852`）；模组自定义供电装置应沿用该调用而非直接改标签。
- 生效与撤销：供电持久化不靠本组件——重进洞穴后由 `archive_switch` 上宝石是否仍在重建状态（OnGemGiven/OnGemTaken 逻辑在 `archive_props.lua`）。

```lua
-- 服务端：监听供电变化刷新自身状态
inst:ListenForEvent("arhivepoweron", function() inst:RefreshPowered() end, TheWorld)
inst:ListenForEvent("arhivepoweroff", function() inst:RefreshPowered() end, TheWorld)

-- 查询（容忍洞穴外世界没有该组件）
local powered = TheWorld.components.archivemanager ~= nil
    and TheWorld.components.archivemanager:GetPowerSetting()
```

- 调整与验证：世界状态标签可在客户端经世界状态系统查询；事件只在 TheWorld 上推，勿挂在其他实体。

## 常见陷阱与限制

- **事件名照抄拼写错误：** 写成 `archivepoweron` 会静默收不到事件；组件内是 `arhivepoweron`/`arhivepoweroff`。[推送点](../../../raw/current/scripts/components/archivemanager.lua#L38-L42)
- **在客户端访问组件：** 构造断言仅主机；客户端请走 `WORLDSTATETAGS` 的标签或监听世界事件。[断言](../../../raw/current/scripts/components/archivemanager.lua#L7)
- **手动改 `_power_enabled`：** 该字段是局部私有，只能经 `SwitchPowerOn` 变化，否则标签与事件不会同步。[私有字段](../../../raw/current/scripts/components/archivemanager.lua#L23)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（79 行），核对 cave.lua:565 挂载、archive_props.lua:803-805/850-852 调用、archive_lockbox/archive_chandelier/vault_teleporter/orchestrina 的读取与事件监听、`mainfunctions.lua:2395` 标签声明。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
