---
title: "playeravatardata"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "把玩家头像信息（名字/皮肤/年龄/装备）存为实体上的 net 变量，让尸体、骨架、稻草人等非玩家实体也能像客户端表一样提供 GetData。"
aliases: ["玩家头像数据", "playeravatardata", "尸体外观数据", "稻草人装扮数据"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/playeravatardata.lua", "prefabs/player_common.lua", "prefabs/corpses_defs.lua", "prefabs/scarecrow.lua", "prefabs/skeleton.lua", "prefabs/shallow_grave.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# playeravatardata

## 快速判断

- 职责与适用：需要"某个实体对外呈现一名玩家的客户端数据"（头像名、prefab、皮肤、年龄、装备）时挂载；核心是替代 `TheNet:GetClientTableForUser` 供 UI 复用同一数据形状（[GetData](../../../raw/current/scripts/components/playeravatardata.lua#L100-L105)）。
- 优先使用：先按需调 `AddXxxData(save)` 声明字段组，再 `SetData(client_obj)` 写入、`GetData()` 读取；玩家原版直接 `AddPlayerData(true)`（[挂载](../../../raw/current/scripts/prefabs/player_common.lua#L2511)）。
- 重要副作用：每组字段创建 net 变量（`net_string` 等），写入即跨端同步；`GetData()` 总是返回新表，调用方可随意改写（[注释](../../../raw/current/scripts/components/playeravatardata.lua#L100-L102)）。
- 修改边界：未调用任何 `AddXxxData` 时 `hasdata` 为 nil，`GetData` 退化为按 `inst.userid` 查真实客户端表，`SetData`/`OnSave`/`OnLoad` 直接返回（[守卫](../../../raw/current/scripts/components/playeravatardata.lua#L143-L145)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddNameData(save)` / `AddBaseSkinData(save)` / `AddClothingData(save)` / `AddAgeData(save)` / `AddEquipData(save)` | save: bool | 惰性创建对应 net 变量组（名字/prefab、base_skin、身体四件、playerage、按 EquipSlot 数量的装备串）（[实现](../../../raw/current/scripts/components/playeravatardata.lua#L30-L98)）。 |
| `AddPlayerData(save)` | save: bool | 依序调用上述五组（[实现](../../../raw/current/scripts/components/playeravatardata.lua#L92-L98)）。 |
| `SetData(client_obj)` | client_obj: table or nil | 把客户端表写入 net 变量；nil 时清空为 ""/0（[实现](../../../raw/current/scripts/components/playeravatardata.lua#L143-L169)）。 |
| `GetData()` | → `table`（新表） | 汇总 net 变量为客户端表形状；皮肤空串转 nil；无数据则查 `TheNet:GetClientTableForUser(userid)`（[实现](../../../raw/current/scripts/components/playeravatardata.lua#L103-L141)）。 |
| `SetAllowEmptyName(allow)` / `SetAllowBurnt(allow)` | bool | 空名或 burnt 实体默认 `GetData` 返回 nil，须显式允许（[实现](../../../raw/current/scripts/components/playeravatardata.lua#L22-L28)、[判定](../../../raw/current/scripts/components/playeravatardata.lua#L106-L108)）。 |
| `OnSave(data)` / `OnLoad(data)` | 组件存档钩子 | save=true 的组参与存读；装备槽 id 与名字互译，未知槽位存入 `unsupported_equips`（[实现](../../../raw/current/scripts/components/playeravatardata.lua#L171-L234)）。 |

## 典型接入

### 让雕像/展示物承载玩家外观

- 前提与选择：服务端写 `SetData`，任何端都可 `GetData` 读（net 变量）；仅名字需求用 `AddNameData`，整套装扮用 `AddClothingData`，减少网络量。
- 生效与撤销：写入即刻同步；`save=false` 的组不进存档，重载后需重新写入。
- 原版依据：稻草人挂名字+服装并 `SetAllowEmptyName(true)`，装扮时写入名字与皮肤（[挂载](../../../raw/current/scripts/prefabs/scarecrow.lua#L178-L181)、[写入](../../../raw/current/scripts/prefabs/scarecrow.lua#L111-L121)）；玩家尸体挂全量数据并随骨架风化迁移给骨架（[尸体](../../../raw/current/scripts/prefabs/corpses_defs.lua#L1392-L1393)、[迁移](../../../raw/current/scripts/prefabs/corpses_defs.lua#L157-L158)）；玩家死亡骨架从 client_obj 取数据（[骨架](../../../raw/current/scripts/prefabs/skeleton.lua#L45)）。

```lua
inst:AddComponent("playeravatardata")
inst.components.playeravatardata:AddNameData(false)
inst.components.playeravatardata:AddClothingData(false)
inst.components.playeravatardata:SetAllowEmptyName(true)
inst.components.playeravatardata:SetData({ name = doer:GetDisplayName(), prefab = doer.prefab,
    body_skin = "body_skin_swashbuckler" })
-- UI 侧读取（形状同 TheNet:GetClientTableForUser）：
local data = inst.components.playeravatardata:GetData()
```

- 调整与验证：客户端动作预览也走该组件——`playercontroller` 对目标 `GetData()` 拿 client_obj（[读取](../../../raw/current/scripts/components/playercontroller.lua#L2385-L2386)）；装扮弹窗同样按此读（[弹窗](../../../raw/current/scripts/widgets/dressupavatarpopup.lua#L124)）。

## 常见陷阱与限制

- `SetData` 传入的表键必须与字段组对齐：名字组是 `name`/`prefab`，服装组是 `body_skin`/`hand_skin`/`legs_skin`/`feet_skin`，错键静默丢弃（[键名](../../../raw/current/scripts/components/playeravatardata.lua#L146-L156)）。
- 玩家尸体上的 `AddPlayerData(true)` 说明尸体数据要进存档；若模组自建实体想跨会话保留外观，同样要传 save=true 并自行在 OnLoad 时机重写（组件自身 OnLoad 只回填 net 变量，[实现](../../../raw/current/scripts/components/playeravatardata.lua#L215-L234)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 player_common、corpses_defs、skeleton、shallow_grave、scarecrow、playercontroller、dressupavatarpopup 消费点；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
