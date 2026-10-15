---
title: "skilltreeupdater"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家技能树端组件：包装 skilltreedata 提供查询／激活／XP 三类接口，处理主机-客户端 RPC 分派与技能 blob 存读迁移。"
aliases: ["技能树更新器", "技能点接口", "skilltreeupdater"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/skilltreeupdater.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "networking.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# skilltreeupdater

## 快速判断

- 职责与适用：挂在每个玩家（[安装](../../../raw/current/scripts/prefabs/player_common.lua#L2518-L2520)），是技能树数据（skilltreedata）与游戏逻辑之间的抽象层；角色技能效果、技能 XP、UI 显示都经它读写（[消费例](../../../raw/current/scripts/prefabs/wortox.lua#L70-L71)）。
- 优先使用：技能效果判断用 `IsActivated(skill)`／`CountSkillTag(tag)`／`HasSkillTag(tag)`；加点经验用 `AddSkillXP(amount)`（[L27-L65](../../../raw/current/scripts/components/skilltreeupdater.lua#L27-L65)）。
- 重要副作用：激活／取消／XP 三类操作按端分派——主机调 `SKILLTREE_DEFS` 的 onactivate／ondeactivate 回调并推 `onactivateskill_server` 等事件，客户端发 `RPC.SetSkillActivatedState`／`RPC.AddSkillXP`（[L84-L112](../../../raw/current/scripts/components/skilltreeupdater.lua#L84-L112)、[RPC 处理](../../../raw/current/scripts/networkclientrpc.lua#L1159-L1192)）。
- 修改边界：存档是编码 blob（`EncodeSkillTreeData`），OnSave/OnLoad 只搬 blob；角色换肤／换角色经 `TransferComponent` 迁移（[L216-L235](../../../raw/current/scripts/components/skilltreeupdater.lua#L216-L235)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsActivated(skill)` / `IsValidSkill(skill)` | → `bool` | 按 `self.inst.prefab` 查 skilltreedata；skill 用技能名字符串（[L27-L33](../../../raw/current/scripts/components/skilltreeupdater.lua#L27-L33)）。 |
| `GetAvailableSkillPoints()` / `GetSkillXP()` / `GetPointsForSkillXP(xp)` | 查询 | 可用点数／经验与换算（[L35-L45](../../../raw/current/scripts/components/skilltreeupdater.lua#L35-L45)）。 |
| `CountSkillTag(tag)` / `HasSkillTag(tag)` | tag 字符串 | 按已激活技能统计 `skilltreedefs.FN.CountTags`；装备／跟随者等跨实体效果常按标签计数（[L59-L65](../../../raw/current/scripts/components/skilltreeupdater.lua#L59-L65)）。 |
| `ActivateSkill(skill)` / `DeactivateSkill(skill)` | skill 名 | 内部先改 skilltreedata 再按端分派：主机侧执行定义回调＋`_skilltreeactivatedany:push()`，需时 `SendRPCToClient(CLIENT_RPC.SetSkillActivatedState)`；本地玩家侧推客户端事件＋`SendRPCToServer`（[L84-L158](../../../raw/current/scripts/components/skilltreeupdater.lua#L84-L158)）。 |
| `AddSkillXP(amount)` | amount 数字 | 数据层加 XP 后同上分派；本地玩家有可用点时推 `newskillpointupdated` 弹提示（[L170-L202](../../../raw/current/scripts/components/skilltreeupdater.lua#L170-L202)）。 |
| `GetPlayerSkillSelection()` / `GetNamesFromSkillSelection(sel)` | 位域数组 | 与 Network 组件的技能位域互转；playerinspectable 用前者同步给客户端（[L47-L57](../../../raw/current/scripts/components/skilltreeupdater.lua#L47-L57)、[消费](../../../raw/current/scripts/components/playerinspectable.lua#L17)）。 |
| `SetSilent(silent)` / `SetSkipValidation(skip)` | bool | 数据层直改开关：silent 跳过网络与回调，skip 跳过合法性校验；批量重建选择时成对使用（[L206-L214](../../../raw/current/scripts/components/skilltreeupdater.lua#L206-L214)）。 |
| `SendFromSkillTreeBlob(inst)` | 激活握手回调 | 解码 blob→校验→先全取消再全激活；在 POSTACTIVATEHANDSHAKE 的 STC_SENDINGSTATE 阶段调用（[L250-L279](../../../raw/current/scripts/components/skilltreeupdater.lua#L250-L279)、[调用](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L1096-L1098)）。 |

## 典型接入

### 按技能激活状态改装备效果（原版同构）

- 前提与选择：效果挂在装备／技能对象侧读持有者的 `skilltreeupdater`；无技能树的角色（`SKILLTREE_DEFS` 无该 prefab）查询返回未激活，不必额外判角色（[查询实现](../../../raw/current/scripts/components/skilltreeupdater.lua#L27-L33)）。

```lua
-- 示例：物品在装备时按技能强化（伪场景，读取方式为原版同构）
local function onequip(inst, owner)
    local updater = owner.components.skilltreeupdater
    if updater ~= nil and updater:IsActivated("wortox_soulhop_2") then
        -- 强化分支
    end
    if updater ~= nil and updater:HasSkillTag("soulbearer") then
        -- 按标签组效果的分支
    end
end
```

- 调整与验证：技能名与标签必须与 `prefabs/skilltree_defs.lua` 中对应角色树一致；给自定义角色加技能树需在 `SKILLTREE_DEFS` 注册后再经本组件查询。

## 常见陷阱与限制

- **客户端本地副本只为 UI**：非主机侧组件持本地 skilltreedata，经 RPC 与主机对齐（[握手前禁存](../../../raw/current/scripts/components/skilltreeupdater.lua#L6-L11)）；客户端直接 `ActivateSkill` 会走 `SendRPCToServer`，绕过组件直改数据不会生效于主机。
- **blob 与 prefab 绑定**：`skilltreeblobprefab` 记录来源角色，换角色后 blob 校验失败会被静默丢弃（注释说明旧档无前缀时容忍尝试）（[L250-L259](../../../raw/current/scripts/components/skilltreeupdater.lua#L250-L259)）。
- **RPC 双轨**：同一激活操作在"主机上的其他玩家"与"本地玩家"两条路径都要覆盖，自定义快捷加点逻辑需复用 `ActivateSkill` 而不是自己发 RPC，否则绕过 onactivate 回调（[L93-L109](../../../raw/current/scripts/components/skilltreeupdater.lua#L93-L109)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、player_common 安装行、握手调用（player_common_extensions）、networking 无缝换角色调用与 networkclientrpc 两端 RPC 处理；未覆盖 skilltreedata 内部位域实现与 skilltree_defs 全部角色树。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/attunable]]、[[reference/components/experiencecollector]]（同属玩家成长侧）。
