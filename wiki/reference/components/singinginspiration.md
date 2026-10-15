---
title: "singinginspiration"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "Wathgrithr 战歌灵感值系统：按造成/受到的伤害与骑乘积累灵感，兑换常驻战歌槽位或瞬时歌，向范围内友军施加 debuff 形式的战歌。"
aliases: ["singinginspiration", "战歌灵感", "灵感值", "battlesong", "战歌槽位"]
tags: [component, energy]
dst_build: "steam-buildid:26230496"
sources: ["components/singinginspiration.lua", "prefabs/wathgrithr.lua", "components/singable.lua", "prefabs/battlesongs.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# singinginspiration

## 快速判断

- 职责与适用：Wathgrithr 的灵感计量条（0～100），只有 wathgrithr prefab 安装；驱动常驻战歌（AddDebuff 到友军）与瞬时歌（立即效果+冷却），其它角色没有该组件。[安装](../../../raw/current/scripts/prefabs/wathgrithr.lua#L182-L186)
- 优先使用：歌本唱歌入口是 singable 组件→`AddSong(songdata, skip_inspire?, inst)`；模组想查可否插歌用 `CanAddSong(songdata, inst)`；读当前槽位用 `GetActiveSong(slot)`。[唱歌入口](../../../raw/current/scripts/components/singable.lua#L20-L27)
- 重要副作用：灵感满槽自动顶掉最旧歌（PopSong）；常驻歌按 `SONG_REAPPLY_PERIOD=0.5s` 周期向范围内目标重新施加 AddDebuff；死亡立即清零；灵感值与槽内歌经 player_classified 同步。[溢出](../../../raw/current/scripts/components/singinginspiration.lua#L166-L170) [重施加](../../../raw/current/scripts/components/singinginspiration.lua#L266-L268) [死亡清零](../../../raw/current/scripts/components/singinginspiration.lua#L39
- 修改边界：可用槽位数完全由 wathgrithr 注入的 `CalcAvailableSlotsForInspirationFn` 计算（未设置会在 DoDelta 报错）；瞬时歌消耗灵感或消耗歌器耐久（rechargeable）。[槽位注入](../../../raw/current/scripts/prefabs/wathgrithr.lua#L184-L186) [DoDelta 依赖](../../../raw/current/scripts/components/singinginspiration.lua#L155-L177)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `current`／`max` | number（默认 0/100） | 经 player_classified `currentinspiration`（百分比 0-255）与 `inspirationdraining` 同步；`GetPercent()` 比例读取。[同步](../../../raw/current/scripts/components/singinginspiration.lua#L3-L16) [上限常量](../../../raw/current/scripts/tuning.lua#L4956-L4965) |
| `DoDelta(delta, forceupdate?)` | — | clamp 后重算可用槽位并推 `inspirationdelta {newpercent, slots_available}`；槽位减少时从栈顶逐个 PopSong；current 从 0 变正或 forceupdate 时启动 OnUpdate 消耗循环。[增量](../../../raw/current/scripts/components/singinginspiration.lua#L155-L177) |
| `AddSong(songdata, skip_inspire?, inst?)` | — | INSTANT 歌走 `OnAddInstantSong`（扣灵感或 discharge 歌器冷却）；常驻歌入 `active_songs` 栈、 Inspire 施加、启动 0.5s 重施加任务与特效任务、同步 classified `inspirationsongs[slot]`、推 `inspirationsongchanged {songdata, slotnum}`。[加歌](../../../raw/current/scripts/components/singinginspiration.lua#L254-L280) |
| `CanAddSong(songdata, inst?)` | → bool | REQUIRE_SKILL 需技能树激活；INSTANT 要求 current≥DELTA 且目标歌器已充能；常驻歌要求 `#active_songs < available_slots`。[资格](../../../raw/current/scripts/components/singinginspiration.lua#L179-L191) [消费方](../../../raw/current/scripts/prefabs/battlesongs.lua#L5-L6) |
| `PopSong()` | — | 移除栈顶歌并清对应 classified 槽位、推 `inspirationsongchanged {slotnum}`；清空后停重施加与特效任务。[弹歌](../../../raw/current/scripts/components/singinginspiration.lua#L282-L306) |
| `Inspire()`／`InstantInspire(songdata)` | 内部 | Inspire 对 `FindFriendlyTargetsToInspire()`（非 PVP 含 12 距离内玩家及其追随者、背包内 leader 物品的追随者，排除 critter/死敌）逐个 `AddDebuff(歌名)`；InstantInspire 对 12 距离内可目标敌人（排除 epic/structure/wall/companion 等）执行歌的 ONINSTANT。[友军搜索](../../../raw/current/scripts/components/singinginspiration.lua#L312-L350) [瞬时目标](../../../raw/current/scripts/components/singinginspiration.lua#L352-L383) |
| 灵感获取（监听） | onhitother／attacked／OnRidingTick | 受击与造成伤害（`from_doattack`）按 `伤害×0.024×(1-当前百分比)` 获取，epic 目标 ×3，再乘 `gainratemultipliers`；骑乘 tick 按 0.35/s 上限 50。[受击](../../../raw/current/scripts/components/singinginspiration.lua#L102-L110) [命中](../../../raw/current/scripts/components/singinginspiration.lua#L112-L132) [骑乘](../../../raw/current/scripts/components/singinginspiration.lua#L134-L153) |
| 自然消耗（OnUpdate） | — | 距上次攻击/受击超过 `INSPIRATION_DRAIN_BUFFER_TIME=7.5s` 后以 `INSPIRATION_DRAIN_RATE=-2/s` 消耗并置 is_draining。[消耗](../../../raw/current/scripts/components/singinginspiration.lua#L398-L407) |
| `SetValidVictimFn(fn)` | 回调 | 限定 OnHitOther 的有效受害者（wathgrithr 传入 IsValidVictim）；瞬时目标标签过滤与它独立。[注入](../../../raw/current/scripts/prefabs/wathgrithr.lua#L186 [使用](../../../raw/current/scripts/components/singinginspiration.lua#L117 |
| `OnSave()`／`OnLoad(data)` | — | 存 current 与激活歌（按 ITEM_NAME 从 song_defs 还原，重新 AddSong(skip_inspire=true)）。[存读](../../../raw/current/scripts/components/singinginspiration.lua#L409-L435) |

## 典型接入

### 给自定义战歌或歌器对接灵感系统

- 前提与选择：自定义歌需在 battlesongdefs 注册 songdata（INSTANT/DELTA/ONINSTANT/LOOP_FX 等字段），歌物品走 singable 组件由动作系统调 AddSong；装备侧资格检查（能否拿起歌）用 `CanAddSong`。[singable 接线](../../../raw/current/scripts/components/singable.lua#L20-L27) [资格消费](../../../raw/current/scripts/prefabs/battlesongs.lua#L1-L10)
- 生效与撤销：常驻歌的生命周期由 AddDebuff 目标侧管理，组件侧只在槽位溢出/PopSong 时停止施加；瞬时歌 ONINSTANT 立即结算，不占槽位。
- 原版依据：wathgrithr 在 master 初始化注入槽位计算与受害者校验；战歌物品（battlesongs.lua）在拿起时用 CanAddSong 决定可拾取性。[安装](../../../raw/current/scripts/prefabs/wathgrithr.lua#L182-L186)

```lua
-- 服务器端：从模组直接给 Wathgrithr 加灵感并插一首已注册的常驻战歌
local function GrantAndSing(player, songdata)
    local insp = player.components.singinginspiration
    if insp == nil then
        return false -- 只有 Wathgrithr 有该组件
    end
    insp:DoDelta(50)
    insp:AddSong(songdata, false, nil) -- 槽位不足时 AddSong 内部静默放弃
    return insp:IsSinging()
end
```

- 调整与验证：`AddSong` 对不满足 `CanAddSong` 的情况不报错直接忽略；验证用 `GetDebugString()`（current/active_songs/available_slots）。[加歌守卫](../../../raw/current/scripts/components/singinginspiration.lua#L255 [调试](../../../raw/current/scripts/components/singinginspiration.lua#L437-L439)

## 常见陷阱与限制

- **未注入 CalcAvailableSlotsForInspirationFn 会让 DoDelta 崩溃：** DoDelta 直接调用该回调无判空；只有 wathgrithr 原版注入。模组给其他角色强行加组件会在此报错。[调用点](../../../raw/current/scripts/components/singinginspiration.lua#L161
- **瞬时歌不等于免费：** 未激活 `wathgrithr_songs_instantsong_cd` 技能时扣灵感（DoDelta(-DELTA)），激活后改为消耗歌器 rechargeable 冷却；两条路径互斥。[瞬时扣费](../../../raw/current/scripts/components/singinginspiration.lua#L241-L252)
- **PVP 下 Inspire 只唱给自己：** 非友军目标搜索在 PVP 时不收集附近玩家，追随者仍收集；想给队友上歌需关闭 PVP。[PVP 分支](../../../raw/current/scripts/components/singinginspiration.lua#L313-L315)
- **灵感只在攻击/受击窗口内保持：** 7.5 秒无攻击交互即开始流失；`max_enemy_health`、`inspiration_gain_bonus` 等构造字段当前在组件内赋值但未见被读取（历史残留），不要基于它们写逻辑。[残留字段](../../../raw/current/scripts/components/singinginspiration.lua#L31-L32)
- **骑乘积累上限 50：** OnRidingTick 最多补到 `INSPIRATION_RIDING_GAIN_MAX=50`，不会推满 100；上限值绑定 BATTLESONG_THRESHOLDS[2]。[骑乘上限](../../../raw/current/scripts/components/singinginspiration.lua#L138-L142) [常量](../../../raw/current/scripts/tuning.lua#L4962

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 singinginspiration.lua，核验灵感获取三路径、槽位计算与溢出、常驻/瞬时歌全流程、友军与敌人目标过滤、classified 同步与存读；核对 wathgrithr 安装注入、singable/battlesongs/actions 消费与 tuning 常量。未审校各战歌 debuff 的具体效果实现。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；槽位换歌、瞬时歌冷却与 PVP 行为未实测。

相关：[[reference/components/singable]]、[[reference/components/battleborn]]。

## 相关链接

- [[reference/components/singable]]：歌本物品侧的唱歌入口组件。
- [[reference/components/battleborn]]：Wathgrithr 击杀回响与灵感并行的另一个战斗系统。
- [[reference/components/rideable]]：OnRidingTick 的骑乘驱动方。
