---
title: "boatcrew：海盗船员编队与战术状态"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "挂在海盗船上的编队管理：成员登记与阵亡退队、掠夺目标与战利品胜负判定、hunt/assault/retreat/delivery 状态与划船朝向。"
aliases: [海盗船员编队, 船员编队, boatcrew, 海盗船编队]
tags: [component, state-machine, ai]
dst_build: "steam-buildid:26230496"
sources: ["components/boatcrew.lua", "components/piratespawner.lua", "components/crewmember.lua", "brains/powdermonkeybrain.lua", "stategraphs/SGpowdermonkey.lua", "prefabs/primemate.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatcrew（海盗船员编队与战术状态）

## 快速判断

- 职责与适用：服务端组件，挂在海盗猴子船上。无 prefab 构造期挂载——运行时由 [piratespawner](../../../raw/current/scripts/components/piratespawner.lua#L116) 在生成海盗船时 `boat:AddComponent("boatcrew")` 并注入成员；成员全空时组件自移除（[自移除](../../../raw/current/scripts/components/boatcrew.lua#L247-L251)）。
- 优先使用：要做海盗式船战 → 与 [[reference/components/crewmember]] 成对使用（成员组件读本组件的状态与 `GetHeadingNormal` 划船）；目标与状态也直接被 powdermonkey 脑消费（[脑读取](../../../raw/current/scripts/brains/powdermonkeybrain.lua#L433)）。
- 重要副作用：成员死亡/移除/传送即退队（[监听](../../../raw/current/scripts/components/boatcrew.lua#L5-L15)）；战利品数超过 `成员数 × BOATCREW_LOOT_PER_MEMBER`(3) 或任一成员 `victory` 即可判定胜利撤退（[胜负](../../../raw/current/scripts/components/boatcrew.lua#L58-L68)）。
- 修改边界：2 秒周期任务只刷新 `status` 字段，实际行为（登船、投掷、划船）在各成员的脑与 stategraph；`captain` 移除联动经注明"DOESN'T WORK"的注释保留原状（[源注](../../../raw/current/scripts/components/boatcrew.lua#L184-L188)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddMember(inst, setcaptain)` | 成员实体、布尔 | 记入 `members`、挂退队监听、`crewmember:SetBoat`；setcaptain 则 `SetCaptain`（[实现](../../../raw/current/scripts/components/boatcrew.lua#L208-L225)）。 |
| `RemoveMember(inst)` | 成员实体 | 调 `crewmember:OnLeftCrew` 与 `removemember` 回调，解绑监听；成员清空时 `RemoveShipData` 并自移除 `vanish_on_sleep`＋本组件（[实现](../../../raw/current/scripts/components/boatcrew.lua#L227-L253)）。 |
| `SetTarget(target)`／`SetHeading(deg)` | 实体或 Vector3／角度 | 目标可为实体或坐标；`GetHeadingNormal` 返回归一化行进向量——撤退且全员在船时按目标角度重算（仍朝目标侧），真正的"反向"由 [[reference/components/crewmember]] 在 away 分支取反（([crewmember.lua:123-157](../../../raw/current/scripts/components/crewmember.lua#L123-L157)）；攻击时按目标 `boatphysics` 速度做提前量（[朝向](../../../raw/current/scripts/components/boatcrew.lua#L129-L162)）。 |
| `SetMemberTag(tag)` | 标签串 | 生成 `membersearchtags = { "crewmember", tag }` 供脑搜索（[实现](../../../raw/current/scripts/components/boatcrew.lua#L115-L118)）。 |
| `SetAddMemberFn/SetRemoveMemberFn(fn)` | `fn(boat, member)` | 成员增删回调（[注册](../../../raw/current/scripts/components/boatcrew.lua#L176-L182)）。 |
| `OnUpdate()` | 周期 | `delivery` 不变；有目标且无可偷战利品/胜负已定/flee → `retreat`；有目标 → `assault`；否则 `hunting`（[状态机](../../../raw/current/scripts/components/boatcrew.lua#L280-L290)）。 |
| `CountPirateLoot()`／`CountCrew()` | 数值 | 数成员背包中非 `personal_possession` 物品的堆叠总数／成员数（[战利品](../../../raw/current/scripts/components/boatcrew.lua#L86-L96)）。 |
| `CrewCheer()` | 无参 | 胜利后延迟随机让成员欢呼（[欢呼](../../../raw/current/scripts/components/boatcrew.lua#L78-L84)）。 |
| `OnSave/LoadPostPass` | 存读钩子 | 存成员 GUID 列表，读回重挂成员（[存读](../../../raw/current/scripts/components/boatcrew.lua#L292-L315)）。 |

## 典型接入

### 原版路径：由 piratespawner 组队

- 前提与选择：服务端；不要自己给船挂组件——piratespawner 生成船后挂组件、`AddMember(captain, true)` 设船长、再补普通成员（[挂载](../../../raw/current/scripts/components/piratespawner.lua#L116)、[船长](../../../raw/current/scripts/components/piratespawner.lua#L161)）。
- 生效与撤销：成员脑按 `status` 与 `target` 行动；炮击命中可置 `flee = true` 强制撤退（[flee](../../../raw/current/scripts/components/piratespawner.lua#L82-L88)），投递任务态由 spawner 直接改 `status = "delivery"` 并设目标点（[投递](../../../raw/current/scripts/components/piratespawner.lua#L409-L416)）。

```lua
-- 服务端：读取海盗船编队状态（模组一般只读不改）
local boat = player.components.hostiletracker  -- 示例：任意拿到船实体的途径
if boat and boat.components.boatcrew then
    local bc = boat.components.boatcrew
    print(bc.status, bc:CountCrew(), bc:CountPirateLoot(), bc.target)
end
```

- 调整与验证：每成员战利品配额 [tuning.lua:6714](../../../raw/current/scripts/tuning.lua#L6714)；`primemate` 也读取本组件（[primemate.lua:181](../../../raw/current/scripts/prefabs/primemate.lua#L181）。

## 常见陷阱与限制

- **成员必须带 crewmember 组件**：`AddMember`/`RemoveMember` 调 `crewmember:SetBoat`/`OnLeftCrew`；没有该组件的"成员"不参与划船且退队逻辑不完整（[成员协作](../../../raw/current/scripts/components/boatcrew.lua#L215-L217)）。
- **组件会自毁**：最后一个成员退队时组件直接 `RemoveComponent("boatcrew")`，外部持有引用会失效（[自毁](../../../raw/current/scripts/components/boatcrew.lua#L247-L251)）。
- **captain 移除联动未生效**：`removecaptain` 注释标明 DOESN'T WORK——船长死后引用即断，实际依赖"成员清空自毁"路径清理（[源注](../../../raw/current/scripts/components/boatcrew.lua#L189-L196)）。
- **stategraph 读组件判空**：SGpowdermonkey 欢呼前检查船与组件存在（[SG 检查](../../../raw/current/scripts/stategraphs/SGpowdermonkey.lua#L248-L249)），模组自建成员也建议同样判空。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/boatcrew.lua` 全部 318 行、piratespawner 挂载与成员注入、crewmember 的划船与退队消费、powdermonkey 脑/SG/primemate 读取点、tuning 配额。未覆盖：powdermonkey 脑全树、primemate 行为。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
