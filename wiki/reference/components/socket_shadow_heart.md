---
title: "socket_shadow_heart"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "WX-78 影子心脏插座的视觉配套组件：在备用机身上挂载心静脉特效实体，供影插座品质刷新时加卸。"
aliases: ["影子心脏插座", "socket_shadow_heart", "WX78 心静脉特效"]
tags: [component, lifecycle]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/socket_shadow_heart.lua", "prefabs/wx78_common.lua", "prefabs/wx78_backupbody.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# socket_shadow_heart

## 快速判断

- 职责与适用：WX-78 影子插座（Socket）系统的视觉配套件，本身无玩法数值；唯一行为是在非玩家且 prefab 为 `wx78_backupbody` 的宿主上生成 `wx78_heartveinspawner` 特效实体并父子挂载。[构造](../../../raw/current/scripts/components/socket_shadow_heart.lua#L1-L13)
- 优先使用：不要手动挂载。原版由 [RefreshShadowSocketBuffs](../../../raw/current/scripts/prefabs/wx78_common.lua#L715) 在影子插座品质为 HIGH（Shadow Heart）或 PERFECT 时加入，品质回退时移除。[加入/移除块](../../../raw/current/scripts/prefabs/wx78_common.lua#L729-L737)
- 重要副作用：仅玩家判定 `isplayer` 为假且 prefab 匹配时才生成特效实体；玩家形态的 WX-78 心静脉表现由 [SetHeartVeins](../../../raw/current/scripts/prefabs/wx78_common.lua#L879) 直接驱动，不经本组件。
- 修改边界：无 OnSave/OnLoad、无网络变量、无标签；组件随 `RemoveComponent` 卸载并连带移除特效实体。[清理](../../../raw/current/scripts/components/socket_shadow_heart.lua#L15-L19)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `Class(function(self, inst))` | 非玩家且 `inst.prefab == "wx78_backupbody"` 时 `SpawnPrefab("wx78_heartveinspawner")` 并 `SetParent`；监听特效实体 `onremove` 置空引用。[实现](../../../raw/current/scripts/components/socket_shadow_heart.lua#L4-L11) |
| `OnRemoveFromEntity()` | `-> nil` | `RemoveComponent` 时若特效实体仍在则 `Remove()`，避免残留子实体。[实现](../../../raw/current/scripts/components/socket_shadow_heart.lua#L15-L19) |

## 典型接入

### 复刻 WX-78 影子插座品质驱动的加卸（原版参照）

- 前提与选择：服务端、仅 `wx78_backupbody` 与 WX-78 玩家 prefab 语境；原版入口是 [RefreshShadowSocketBuffs](../../../raw/current/scripts/prefabs/wx78_common.lua#L715-L737) 读取 `SOCKETQUALITY.HIGH/PERFECT` 的 `heart = true` 配置。[品质表](../../../raw/current/scripts/prefabs/wx78_common.lua#L701-L707)
- 生效与撤销：品质刷新函数按当前最高品质插座决定加/卸；`wx78_backupbody` 的 prefab 列表预声明了特效实体 `wx78_heartveinspawner`。[预声明](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L21)
- 原版依据：无存读钩子，状态完全由插座品质在运行期重算，重载后由 socketholder 数据重新触发刷新。

```lua
-- 服务端：仅示意原版等价调用；mod 应复用 RefreshShadowSocketBuffs 而非直加
if inst.prefab == "wx78_backupbody" then
    inst:AddComponent("socket_shadow_heart")
end
```

- 调整与验证：组件对玩家宿主是空操作（构造内 `isplayer` 拦截），玩家侧外观改动应走 `WX78Common.SetHeartVeins`。

## 常见陷阱与限制

- **以为组件自带影子心脏玩法效果：** 全部逻辑只有特效实体的生成与清理；伤害/减益/采集限制由 `socket_shadow_mimicry`、debuffer、harvester 等其他插座件承担，本组件不参与。[职责](../../../raw/current/scripts/components/socket_shadow_heart.lua#L1-L19)
- **在自定义 prefab 上挂载：** 特效实体只在 `wx78_backupbody` 上生成；其他 prefab 挂上后无任何可见效果，仅空耗一个组件位。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（21 行），核对 wx78_common.lua 的品质表与加卸块、wx78_backupbody.lua 的 prefab 预声明；全库 grep 确认消费者仅 wx78_common.lua 与 wx78_backupbody.lua。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；插座品质刷新链路未实测。

相关：[[reference/components/socketholder]]、[[reference/components/upgrademoduleowner]]
