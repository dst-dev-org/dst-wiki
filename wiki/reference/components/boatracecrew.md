---
title: "boatracecrew"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "赛影船的船员名册：登记船长与成员、跟随赛点目标，成员死亡/传送/移除自动出队，供赛船 AI 读取航向。"
aliases: ["赛船船员", "影船队伍", "boatracecrew"]
tags: [component, ai]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/boatracecrew.lua", "prefabs/yotd_boats.lua", "prefabs/boatrace_primemate.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatracecrew

## 快速判断

- 职责与适用：挂在生肖年赛影船 `dragonboat_shadowboat` 上的成员名册（prefabs/yotd_boats.lua:607，prefab 定义 :922）；成员为带 `crewmember` 组件的猴子船员，生成时加入船长（yotd_boats.lua:565）。
- 优先使用：给船补人→`AddMember(ent, is_captain)`；改航向目标→`SetTarget(checkpoint)`（消费例 prefabs/boatrace_primemate.lua:98-105）。
- 重要副作用：每 2 秒 `OnUpdate` 只把 `status` 置 `"assault"`（components/boatracecrew.lua:19、:109-111）；成员 `death`/`teleported`/`onremove` 即自动移出（:43-47）。
- 修改边界：成员的 `crewmember:SetBoat`/`OnLeftCrew` 由本组件驱动（:71-72、:88-101）；全员退出时回调 `on_crew_empty`（:104-106）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddMember` | `(new_member, is_captain?) -> nil` | 重复添加忽略；挂监听并调 `crewmember:SetBoat(self.inst)`；`is_captain` 同时 `SetCaptain`。:65-83 |
| `RemoveMember` | `(member) -> nil` | 回调 `on_member_removed`、`crewmember:OnLeftCrew` 与 `SetBoat(nil)` 后清名册。:85-107 |
| `SetTarget` | `(target) -> nil` | 换目标时清旧 onremove 监听；`GetHeadingNormal()` 依赖它。:32-40 |
| `GetHeadingNormal` | `() -> hx, hz \| nil` | 指向 target 的单位向量 x/z；无 target 返回 nil。:114-123 |
| `SetCaptain` | `(captain) -> nil` | 船长仅作记录，onremove 时置空。:55-63 |
| 回调字段 | `on_member_added/on_member_removed/on_crew_empty` | 可选，组件内不默认赋值。:21-23、:75-77、:93-95、:104-106 |

## 典型接入

### 读取赛船队伍状态（服务器）

- 前提与选择：影子船 `persists = false`（yotd_boats.lua:603），存读路径（:144-166）实际不落档；mod 介入应在船生成后读取或操作名册。
- 生效与撤销：`crewmember` 读船时同时认 `boatcrew` 或 `boatracecrew`（components/crewmember.lua:41、:99），两类船共用船员协议。

```lua
-- 服务器：统计一条赛船的船长、人数与目标
local function DumpRaceCrew(boat)
    local crew = boat.components.boatracecrew
    if crew ~= nil then
        print(crew.captain, GetTableSize(crew.members), crew.target)
    end
end
```

- 调整与验证：直接 `AddMember` 外部生物可行，但该生物需带 `crewmember` 组件才会挂船；否则仅入名册不联动。

## 常见陷阱与限制

- `OnUpdate` 无条件置 `status = "assault"`，无其他行为（:109-111）；不要期待它驱动战斗或航向。
- 成员按 GUID 保存（:147-152），但宿主船不入档，跨档恢复路径未被原版行使。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 167 行；核验挂载与船长添加（yotd_boats.lua:607、:565）、目标更新消费（boatrace_primemate.lua:75-105）与 crewmember 兼容读取（crewmember.lua:41、:99）。
- 游戏运行：未运行。

## 相关链接

- 成员侧协议：[[reference/components/crewmember]]。
