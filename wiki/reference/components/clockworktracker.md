---
title: "clockworktracker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "发条随从上限组件：按车／马／象三类统计跟随的发条棋子，支持加成槽扩容与超限解除，供棋子好友玩家与 WX-78 电路模块。"
aliases: ["发条随从计数", "棋子好友上限", "clockworktracker"]
tags: [component, ai, callback]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/clockworktracker.lua", "prefabs/clockwork_common.lua", "wx78_moduledefs.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# clockworktracker

## 快速判断

- 职责与适用：挂在玩家（或可收随从的 doer）身上，按 `rook`／`knight`／`bishop` 三类键登记跟随的发条棋子（components/clockworktracker.lua:3-8），控制数量上限并处理超限。
- 优先使用：被动路径——棋子经 `followermemory` 重新认主时自动挂组件并登记（prefabs/clockwork_common.lua:249-252）；WX-78 chess 电路模块用 `SetBonus` 加跨类型加成槽（wx78_moduledefs.lua:1613-1618）。
- 重要副作用：登记期间监听棋子 `onremove`／`leaderchanged`，移除或换主自动销账（lines 14-19、117-118、123-138）；超限释放会直接 `follower:SetLeader(nil)`（line 151）。
- 修改边界：加成清零且无登记时组件把自己 `RemoveComponent`（lines 160-173）；`OnRemoveFromEntity` 解除所有棋子监听（lines 22-29）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `ents` | `{rook, knight, bishop} 每类 {num, ents}` | 类型识别靠实体 `rook`／`knight`／`bishop` 标签（lines 3-8、31-37） |
| `bonus` | `SourceModifierList`（additive） | 跨类型共享的加成槽总量；例：2 槽可多带 2 只但不能同类型（lines 10-12 注释、87-108） |
| `GetMaxForType(chesstype)` | `-> number` | 每类默认 `TUNING.CLOCKWORK_MAX_FOLLOWING`（2）；宿主带 `chessfriend` 标签为 3（line 75；tuning.lua:9453-9454）；YOTH 活动期间马无上限（lines 71-73） |
| `GetBonusForType(chesstype)` | `-> number` | `ceil(bonus/3)`；YOTH 期间马 0、其他类 `ceil(bonus/2)`（lines 78-85） |
| `CanAddClockwork(ent)` | `-> bool` | 基础上限内直接放行，超基础部分按全类型已用加成槽校验（lines 87-108） |
| `AddClockwork(ent)`／`RemoveClockwork(ent)` | 登记／销账 | 自动挂监听；同类清零时尝试自移除（lines 110-138） |
| `SetBonus(source, bonus, key)` | 加成槽 | 总量下降即重校验随从（lines 54-63）；WX-78 以模块数为值（wx78_moduledefs.lua:1613-1618） |
| `OverrideMaxFollowersForType(chesstype, max)` | 单类上限覆盖 | 传 nil 恢复默认；下调时立即重校验（lines 39-52） |
| `RevalidateFollowers()` | 释放超额 | 按 bishop→rook→knight 顺序解除（lines 140-158） |

## 典型接入

### 给玩家加发条加成槽（服务端）

- 前提与选择：组件通常由棋子认主流程自动创建；模组主动给玩家扩容时才手动挂载。`chessfriend` 标签提升单类上限（原版 WX-78 常驻带标签，prefabs/wx78.lua:692）。
- 生效与撤销：`SetBonus` 的 source／key 支持幂等更新；bonus 归 0 且无随从时组件自动移除（lines 60-62、160-173），下次登记需重新 AddComponent。
- 原版依据：clockwork_common.lua:247-255（认主登记、超限推 `ms_maxclockworks` 事件）；components/wisecracker.lua:452 监听该事件出吐槽台词。

```lua
-- master 端，给玩家 3 个跨类型加成槽：
inst:AddComponent("clockworktracker")
inst.components.clockworktracker:SetBonus(inst, 3, "mymod_bonus")

-- 查询与登记：
local max = inst.components.clockworktracker:GetMaxForType("knight")
inst.components.clockworktracker:AddClockwork(some_clockwork_ent)
```

- 调整与验证：加成槽是全局池而非每类一份（lines 10-12）；需要单类上限变化用 `OverrideMaxFollowersForType`。

## 常见陷阱与限制

- **两个相似标签职责不同：** 上限检查读 `chessfriend`（line 75）；WX-78 chess 技能激活加的是 `chessbff`（wx78_moduledefs.lua:1603），只影响棋子敌对判定（clockwork_common.lua:91），不提升上限。
- **类型识别只认三个标签：** 实体没有 `rook`／`knight`／`bishop` 标签时 `GetChessType` 返回 nil，登记与销账都会被跳过（lines 31-37、111-113）。
- **释放顺序固定：** 超限释放按 bishop→rook→knight（line 142），随机取 `next(v.ents)` 解除（line 149），不能指定保留哪只。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。通读 components/clockworktracker.lua 全文（175 行）；核验 clockwork_common.lua:240-255、wx78_moduledefs.lua:1602-1620、wx78.lua:692、tuning.lua:9453-9454、wisecracker.lua:452。未运行游戏。
- 结构检查：validate_page.py / validate_source_links.py 通过（见批级自验）。

相关：[[reference/components/follower]]、[[reference/components/followermemory]]
