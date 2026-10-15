---
title: "mightygym"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "沃尔夫冈健身器（mighty_gym）的器械逻辑：双槽配重装载与换片、入/出健身房的状态交换与动画同步，把配重换算成体格收益。"
aliases: ["mightygym", "健身器", "mighty gym", "举重台", "健身房"]
tags: [component, building, state-machine]
dst_build: "steam-buildid:26230496"
sources: ["components/mightygym.lua", "prefabs/mighty_gym.lua", "components/mightiness.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# mightygym

## 快速判断

- 职责与适用：mighty_gym 实体的组件（master 端）：自带 2 槽 inventory 收纳 heavy 标签配重，`CanWorkout` 校验后让 strongman 玩家进入健身会话（藏真身、用器械动画替代）；体格收益由健身房状态机（SGwilson_gymstates）按节拍调 `CalculateMightiness` 并经 `DoDelta(fromgym)` 发放。[安装](../../../raw/current/scripts/prefabs/mighty_gym.lua#L116 [收益发放](../../../raw/current/scripts/stategraphs/SGwilson_gymstates.lua#L127-L130)
- 优先使用：放配重走 heavyobstacleusetarget 的 OnUseHeavy → `LoadWeight(weight, slot?)`；开始健身走动作 → `CharacterEnterGym(player)`；退出用 `CharacterExitGym(player)`。[器械安装](../../../raw/current/scripts/prefabs/mighty_gym.lua#L118-L119)
- 重要副作用：装载满 2 槽时给世界置 `loaded` 标签、同步配重符号与力量条美术（player_classified.inmightygym = 配重-1）；入健身房会隐藏真健身房实体、关闭其碰撞、玩家加 `fireimmune` 并改障碍物理。[装载](../../../raw/current/scripts/components/mightygym.lua#L159-L201) [入器械](../../../raw/current/scripts/components/mightygym.lua#L337-L395)
- 修改边界：只有 `strongman` 标签且有 mightiness 的玩家能健身；健身中器械免疫火（fireimmune）并记录皮肤以同步玩家外观；被打/被删即触发玩家退出。[资格](../../../raw/current/scripts/components/mightygym.lua#L231-L243) [退出监听](../../../raw/current/scripts/components/mightygym.lua#L363-L364)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CalcWeight()` | → number | 统计器械 inventory 中 heavy 物品的 `gymweight`（缺省 2）；仅当多于 1 件时累计，单件配重按 0 算（平衡才有效）。[计算](../../../raw/current/scripts/components/mightygym.lua#L51-L69) |
| `LoadWeight(weight, slot?)` | — | 双槽满时丢出 `full_drop_slot`（1↔2 轮换）再入新片并播音效；按 materialid/oversized_veggie/MATERIAL_SOUNDS 选落地音；同步槽位符号、南瓜雕刻 fx 与 strongman 的力量条美术。[装载](../../../raw/current/scripts/components/mightygym.lua#L159-L201) |
| `UnloadWeight()` | — | DropEverything、清符号/雕刻 fx、复位 full_drop_slot、移除 loaded 标签；hammer 拆除与换片前的清场都走它。[卸载](../../../raw/current/scripts/components/mightygym.lua#L210-L229) |
| `CanWorkout(doer)` | → bool, reason | 校验链：strongman+mightiness → 未燃烧/阴燃 → 无人使用（strongman==nil）→ 饥饿 ≥ `CALORIES_SMALL` → 两槽都有配重（单槽为 UNBALANCED）。reason 是 ONFIRE/SMOULDER/FULL/HUNGRY/NOWEIGHT/UNBALANCED 字符串。[资格](../../../raw/current/scripts/components/mightygym.lua#L231-L260) |
| `CalculateMightiness(perfect)` | → TUNING 值 | 配重 <7：LOW（perfect 升 MED）；≥7：MED（perfect 升 HIGH）。数值取 `TUNING.GYM_RATE`，由健身房状态机（SGwilson_gymstates）按节拍结果调用并 DoDelta(fromgym=true)。[收益](../../../raw/current/scripts/components/mightygym.lua#L262-L276) [调用点](../../../raw/current/scripts/stategraphs/SGwilson_gymstates.lua#L127-L130) |
| `StartWorkout(doer)` | — | 绑定 strongman↔器械（player.gym、组件.strongman）、按配重档给 hunger.burnratemodifiers 加消耗、抓取玩家皮肤并 `SetSkinModeOnGym` 同步器械外观、置 hasstrongman 标签。[开始](../../../raw/current/scripts/components/mightygym.lua#L283-L307) |
| `StopWorkout()` | — | 还原器械皮肤/build、strongman 侧停会话、移除饥饿消耗来源、清 hasstrongman。[结束](../../../raw/current/scripts/components/mightygym.lua#L309-L320) |
| `CharacterEnterGym(player)` | — | 藏器械、关碰撞、玩家 ApplyAnimScale(1)、按 gym_skin 覆盖器械符号或 AddOverrideBuild("mighty_gym"/"fx_wolfgang")、传送至器械位、进 `mighty_gym_active_pre` 状态、改障碍物理、挂 onremove/attacked 退出监听，最后 StartWorkout。[进入](../../../raw/current/scripts/components/mightygym.lua#L337-L395) |
| `CharacterExitGym(player)` | — | 还原实体与玩家动画/物理/影子；死亡、冻结、昏迷或睡眠玩家直接 Teleport 到侧向点，否则走 `jumpout` 状态（带 nointerrupt，退出后说状态台词）；最后 StopWorkout 并清 player.gym。[退出](../../../raw/current/scripts/components/mightygym.lua#L404-L475) |
| `SetSkinModeOnGym(doer, skin_mode)` | — | 用 SetSkinsOnAnim 把玩家皮肤套到器械动画；mightiness 三态切换时反向调用它保持外观一致。[皮肤同步](../../../raw/current/scripts/components/mightygym.lua#L278-L281) [mightiness 消费](../../../raw/current/scripts/components/mightiness.lua#L298-L301) |

## 典型接入

### 模组化配重或健身收益

- 前提与选择：自定义配重物品需 heavy 标签 + symbolswapdata 组件（符号替换）+ 可选 gymweight 字段；玩家侧收益不经本组件——健身房状态机按节拍调 `CalculateMightiness` 后 `DoDelta(..., fromgym=true)`（可超上限），健身期间 strongman 组件还会暂停体格自然下降。[收益调用](../../../raw/current/scripts/stategraphs/SGwilson_gymstates.lua#L127-L130) [暂停](../../../raw/current/scripts/components/strongman.lua#L7-L10)
- 生效与撤销：装载状态随器械存档（inventory 常规持久化）；健身会话不存档——存档时强退由 OnSave/LoadPostPass 之外的常规实体保存兜底，重载后器械恢复无人在用状态。卸载全部配重用 UnloadWeight。
- 原版依据：器械 prefab 用 heavyobstacleusetarget 接收重物放置，onbuilt 时监听重物交互；资格校验的字符串 reason 直接映射动作失败台词。[器械接线](../../../raw/current/scripts/prefabs/mighty_gym.lua#L118-L121)

```lua
-- 服务器端：查询某台健身器当前配重与是否可用（模组 HUD/任务用）
local function GetGymStatus(gym)
    local comp = gym.components.mightygym
    if comp == nil then
        return nil
    end
    return comp:CalcWeight(), comp:InUse()
end
```

- 调整与验证：`InUse()` 读 strongman 字段；改 GYM_RATE 需要理解 perfect 参数由状态机节拍判定传入，组件自身不判定完美。[使用中](../../../raw/current/scripts/components/mightygym.lua#L322-L324)

## 常见陷阱与限制

- **单件配重不算重量：** CalcWeight 在 `#items > 1` 才累计，双槽各一片才出力；只放一片时健身收益按最低档且 CanWorkout 会以 UNBALANCED 拒绝。[单件分支](../../../raw/current/scripts/components/mightygym.lua#L62-L66)
- **满槽换片丢的是"最旧槽"：** full_drop_slot 1↔2 轮换，被替换的配重直接 DropItem 落地（不返还背包）；自动化流程要接住掉落物。[换片](../../../raw/current/scripts/components/mightygym.lua#L162-L169)
- **gymweight 缺省 2、上限由物品决定：** `item.gymweight or 2`；想造超重器械必须给物品字段，组件不读 weight 数值（物理重量与健身重量是两套）。[权重来源](../../../raw/current/scripts/components/mightygym.lua#L64
- **入健身房期间实体被藏：** Hide + Physics:SetActive(false)，期间对 gym 的 FindEntities 不可见（碰撞也没了）；依赖器械碰撞的逻辑在会话中会失效。[隐藏](../../../raw/current/scripts/components/mightygym.lua#L340-L343)
- **退出监听绑定在器械上：** onremove/attacked 监听挂玩家（trytoexitgym 读 strongman.gym），玩家被打或被删都会强退；模组强制传送玩家也可能绕过 jumpout 正常路径，注意状态残留。[监听](../../../raw/current/scripts/components/mightygym.lua#L363-L364)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 mightygym.lua，核验配重装载/换片、资格校验链、收益换算、进出器械的实体/动画/物理交换、皮肤同步与退出路径；核对 mighty_gym prefab 安装、mightiness 的健身房联动与 tuning 常量段。strongman 状态机的节拍判定未展开。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；节拍收益与异常退出表现未实测。

相关：[[reference/components/mightiness]]、[[reference/components/strongman]]。

## 相关链接

- [[reference/components/mightiness]]：体格值与三态皮肤，器械收益的落点。
- [[reference/components/strongman]]：健身会话的状态机持有者。
