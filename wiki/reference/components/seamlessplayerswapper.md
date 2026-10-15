---
title: "seamlessplayerswapper"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家无缝换体：不经选人界面把玩家替换为新 prefab（原版月风暴 wonkey 变身），转移背包与角色数据并隐藏旧实体。"
aliases: ["无缝换角色", "seamlessplayerswapper", "wonkey变身"]
tags: [component, lifecycle, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/seamlessplayerswapper.lua", "prefabs/player_common.lua", "networking.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# seamlessplayerswapper

## 快速判断

- 职责与适用：挂在玩家实体（所有玩家，[player_common](../../../raw/current/scripts/prefabs/player_common.lua#L2684)）；通过 `TheNet:SpawnSeamlessPlayerReplacement` 生成同 userid 的新玩家实体完成 prefab 切换，用于 wonkey 变身与还原（[入口](../../../raw/current/scripts/prefabs/player_common.lua#L2348-L2357)）。
- 优先使用：模组要做"不退场就换角色"时复用 `_StartSwap` 的两个公开包装 `DoMonkeyChange` / `SwapBackToMainCharacter`；自定义目标 prefab 无公开接口。
- 重要副作用：旧实体触发 `ms_playerreroll`，新实体触发 `ms_playerseamlessswaped`（[L60](../../../raw/current/scripts/components/seamlessplayerswapper.lua#L60)、[L84](../../../raw/current/scripts/components/seamlessplayerswapper.lua#L84)）；旧实体被隐藏并 `SetClassifiedTarget` 到新玩家（[L68-L72](../../../raw/current/scripts/components/seamlessplayerswapper.lua#L68-L72)）。
- 修改边界：swap 在网络层生成新实体，`OnSeamlessCharacterSwap` 运行在新玩家上（由 [networking.lua](../../../raw/current/scripts/networking.lua#L286-L288) 调用）；userid 为空直接放弃（防 c_spawn 的无主实体，[L14](../../../raw/current/scripts/components/seamlessplayerswapper.lua#L14)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `DoMonkeyChange()` | `-> void` | 变身 wonkey 的入口，内部 `_StartSwap("wonkey")`。[实现](../../../raw/current/scripts/components/seamlessplayerswapper.lua#L41-L43) |
| `SwapBackToMainCharacter()` | `-> void` | 变回原角色，内部 `_StartSwap(nil)`；原角色信息从 `main_data` 取，缺失兜底 wilson。[实现](../../../raw/current/scripts/components/seamlessplayerswapper.lua#L45-L47) |
| `_StartSwap(new_prefab)` | `可选 prefab -> void` | 记录/恢复 `swap_data`（各形态 skin_base）与 `main_data`（原角色 prefab+皮肤），携带衣物调 TheNet 生成替换者。[实现](../../../raw/current/scripts/components/seamlessplayerswapper.lua#L13-L39) |
| `OnSeamlessCharacterSwap(old_player)` | `entity -> void` | 新玩家侧收尾：解诅咒物品、`SaveForReroll/LoadForReroll` 迁移数据、`SwapAllCharacteristics`、隐藏旧实体、处理 mime 标记与状态机过渡。[实现](../../../raw/current/scripts/components/seamlessplayerswapper.lua#L50-L95) |
| `PostTransformSetup` | `-> void` | 按 `main_data.mime` 补 `mime` 标签并设 `talker.speechproxy` 为原角色。[实现](../../../raw/current/scripts/components/seamlessplayerswapper.lua#L97-L102) |
| `SaveForReroll/OnSave/OnLoad` | 存读钩子 | `swap_data`/`main_data` 持久化；player_common 的整体存读链也会单独接 `SaveForReroll`（[L1859](../../../raw/current/scripts/prefabs/player_common.lua#L1859)）。[实现](../../../raw/current/scripts/components/seamlessplayerswapper.lua#L104-L132) |
| `main_data.mime` | bool 或 nil | 只有旧角色天然是 mime 且新角色不是时置位，保证变身不"传染"哑剧标签（[L74-L81](../../../raw/current/scripts/components/seamlessplayerswapper.lua#L74-L81)）。 |

## 典型接入

### 复用原版变身通道

- 前提与选择：仅主机端调用；这是引擎级网络操作（生成新玩家实体），模组层面不建议自写替换逻辑而直接调用组件入口。
- 生效与撤销：调用后旧实体退场、新实体上场；还原用 `SwapBackToMainCharacter`，变身形态的皮肤由 `swap_data` 自动往返。

```lua
-- 主机端：把玩家无缝换成 wonkey（原版月风暴路径）
if inst.components.seamlessplayerswapper ~= nil then
    inst.components.seamlessplayerswapper:DoMonkeyChange()
end
```

- 调整与验证：目标仅限 wonkey/回原角色两个入口；新实体的状态机过渡查 `PLAYER_SWAP_TRANSITIONS`（变形/还原专用状态，[L86-L94](../../../raw/current/scripts/components/seamlessplayerswapper.lua#L86-L94)），缺失时落到 idle。

## 常见陷阱与限制

- **不是本地重建**：换体经 TheNet 全程生成新 GUID 实体，实体引用（包括旧实体的监听）不迁移；依赖旧实体的模组状态要在 `ms_playerreroll`/`ms_playerseamlessswaped` 里自行转移。
- **userid 必须有效**：`c_spawn("wonkey")` 之类无 userid 的实体不会换体（[L14-L16](../../../raw/current/scripts/components/seamlessplayerswapper.lua#L14-L16)）。
- **数据迁移走 reroll 管线**：`SaveForReroll/LoadForReroll` 只搬运 reroll 白名单数据；组件显式用完整 OnSave 数据覆盖 swap 部分（[L62-L64](../../../raw/current/scripts/components/seamlessplayerswapper.lua#L62-L64)），自定义物品要确认自身在 reroll 链路里。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、player_common 挂载与变身入口、networking.lua 替换生成流程。
- 游戏运行：未运行。
