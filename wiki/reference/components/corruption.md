---
title: "corruption"
created: "2026-10-26"
updated: "2026-10-28"
type: "component"
summary: "物品上的服务端腐化值账本（原版宿主 king_cane）：0~TUNING.KING_CANE_MAX_CORRUPTION 计量与满值 corrupted 态、三回调、施法目标硬直登记与 CorruptEnt 按目标类型分发；腐化清除入口由 moonbase 消费。"
aliases: ["王者法杖腐化值", "腐化值组件", "corruption"]
tags: ["component", "callback", "network-sync"]
dst_build: "steam-buildid:26230496"
sources: ["components/corruption.lua", "prefabs/king_cane.lua", "prefabs/moonbase.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua", "stategraphs/commonstates.lua", "componentutil.lua", "components/shadowthrallmanager.lua", "prefabs/bunnyman.lua", "prefabs/monkey.lua", "prefabs/rabbit.lua", "prefabs/knight.lua", "prefabs/chesspieces.lua", "tuning.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# corruption

## 快速判断

- 职责与适用：挂在可腐化物品上的服务端组件，原版唯一宿主是王者法杖 king_cane（ismastersim 分支添加）：维护 0~`TUNING.KING_CANE_MAX_CORRUPTION`（100）的腐化值，满值进入 corrupted 态并触发回调；同时承担 CORRUPTNIGHTMARE 动作的目标硬直登记（OnTargetCorruptable/OnUntargetCorruptable）与 CorruptEnt 按目标类型分发。[components/corruption.lua:1](../../../raw/current/scripts/components/corruption.lua#L1 [prefabs/king_cane.lua:350](../../../raw/current/scripts/prefabs/king_cane.lua#L350
- 优先使用：数值存取用 `GetCurrent`/`SetCurrent`/`DoDelta`，状态判定 `IsCorrupted`，强制置位/复位用 `MakeCorrupted`/`MakeUncorrupted`，变化通知注册 `SetOnCurrentFn`/`SetOnCorruptedFn`/`SetOnUncorruptedFn`；见关键接口。
- 重要副作用：`MakeCorrupted` 给实体加网络同步标签 `corrupted` 并停止组件更新（corrupted 态不再自然衰减）；正 `DoDelta` 会重置衰减计时；`OnRemoveFromEntity` 先解除目标硬直再尝试复位。[components/corruption.lua:44](../../../raw/current/scripts/components/corruption.lua#L44 [components/corruption.lua:14](../../../raw/current/scripts/components/corruption.lua#L14
- 修改边界：仅服务端——king_cane 客户端分支不添加本组件，客户端只能看 `corrupted` 等标签与 king_cane_fx 的 level 净变量；回调由组件以 `(inst[, current])` 调用，状态需回调方自持。[prefabs/king_cane.lua:295](../../../raw/current/scripts/prefabs/king_cane.lua#L295 [prefabs/king_cane.lua:558](../../../raw/current/scripts/prefabs/king_cane.lua#L558

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `current` | number，构造默认 0 | 腐化值，范围 [0, TUNING.KING_CANE_MAX_CORRUPTION]。[components/corruption.lua:4](../../../raw/current/scripts/components/corruption.lua#L4 |
| `IsCorrupted()` | boolean | `current >= TUNING.KING_CANE_MAX_CORRUPTION`（100）。[components/corruption.lua:22](../../../raw/current/scripts/components/corruption.lua#L22 [tuning.lua:9740](../../../raw/current/scripts/tuning.lua#L9740 |
| `GetCurrent()` | number | 原始值读取；king_cane 用它算疯狂光环分档。[components/corruption.lua:26](../../../raw/current/scripts/components/corruption.lua#L26 [prefabs/king_cane.lua:224](../../../raw/current/scripts/prefabs/king_cane.lua#L224 |
| `SetCurrent(current)` | — | 已 corrupted 时直接 return；clamp 到 0~MAX；值变化触发 `oncurrentchanged(inst, current)`；达 MAX 转 `MakeCorrupted`，>0 启用更新，归零停更并清 timetodecay。[components/corruption.lua:65](../../../raw/current/scripts/components/corruption.lua#L65 |
| `DoDelta(delta)` | — | 已 corrupted 时直接 return；delta>0 时 `timetodecay = KING_CANE_CORRUPTION_DECAY_PERIOD`（12 秒）后转 SetCurrent。[components/corruption.lua:84](../../../raw/current/scripts/components/corruption.lua#L84 [tuning.lua:9744](../../../raw/current/scripts/tuning.lua#L9744 |
| `MakeCorrupted()` | — | 加标签 `corrupted`、current=MAX、timetodecay=0、停更，触发 `oncorruptedfn(inst)`；无前置条件，重复调用仍会触发回调。[components/corruption.lua:44](../../../raw/current/scripts/components/corruption.lua#L44 |
| `MakeUncorrupted()` | — | 仅当前已 corrupted 才生效：摘标签、current=0、触发 `onuncorruptedfn(inst)`；原版唯一调用方是 moonbase。[components/corruption.lua:54](../../../raw/current/scripts/components/corruption.lua#L54 [prefabs/moonbase.lua:466](../../../raw/current/scripts/prefabs/moonbase.lua#L466 |
| `SetOnCurrentFn`／`SetOnCorruptedFn`／`SetOnUncorruptedFn(fn)` | fn(inst[, current]) | 三个回调槽位；king_cane 全部使用（FX 分档/标签/暗影支配/交易开关）。[components/corruption.lua:30](../../../raw/current/scripts/components/corruption.lua#L30 [prefabs/king_cane.lua:351](../../../raw/current/scripts/prefabs/king_cane.lua#L351 |
| `OnTargetCorruptable(target, doer)`／`OnUntargetCorruptable(target, doer)` | — | 施法起止登记 `corruptiontarget`；开始时对 stalker 系目标 `PushEventImmediate("stalker_corruption_stun")`（可经 `RedirectStalkerCorruption` 重定向，batbosscave→bat_boss）、对 `chess_moonevent` 目标推 `forcechessstruggle=true`；结束时推 `interruptcorruption` 与 `forcechessstruggle=false`。由 SGwilson castspell 状态在 Frame 8／onexit 调用。[components/corruption.lua:97](../../../raw/current/scripts/components/corruption.lua#L97 [stategraphs/SGwilson.lua:16218](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16218 [stategraphs/SGwilson.lua:16272](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16272 |
| `CorruptEnt(target, doer)` | boolean\|nil | CORRUPTNIGHTMARE 的执行体，按下文分发链；命中即 `rechargeable:Discharge` 对应冷却并返回 true，fissure 分支返回 ControlFissure 成败，无命中返回 nil。[components/corruption.lua:124](../../../raw/current/scripts/components/corruption.lua#L124 [actions.lua:7122](../../../raw/current/scripts/actions.lua#L7122 |
| `OnSave()`／`OnLoad(data)` | {current, timetodecay} | 仅 current>0 时落档；OnLoad 经 SetCurrent 恢复（见陷阱：timetodecay 实际不恢复）。[components/corruption.lua:186](../../../raw/current/scripts/components/corruption.lua#L186 |
| `OnRemoveFromEntity`／`OnRemoveEntity` | — | 两者同体：先 `OnUntargetCorruptable()` 解硬直，再 `MakeUncorrupted()`。[components/corruption.lua:14](../../../raw/current/scripts/components/corruption.lua#L14 |
| `OnUpdate(dt)`／`LongUpdate` | — | 0<current<MAX 时每 12 秒 `DoDelta(KING_CANE_CORRUPTION_DECAY_AMOUNT = -1)`；current<=0 停更。[components/corruption.lua:170](../../../raw/current/scripts/components/corruption.lua#L170 [tuning.lua:9745](../../../raw/current/scripts/tuning.lua#L9745 |

动作接线：componentactions 的 EQUIPPED 级 `corruption` 处理器在「右键＋手持物带 `corrupted` 标签＋目标带 `nightmarecorruptable`＋目标无 `nomagic`」时给出 CORRUPTNIGHTMARE；动作 fn 再经 `nomagiccast` 标签与物品拟态检查后调 `CorruptEnt`。[componentactions.lua:2307](../../../raw/current/scripts/componentactions.lua#L2307 [actions.lua:7113](../../../raw/current/scripts/actions.lua#L7113

### CorruptEnt 分发链（按优先级短路）

1. `IsStalkerCorruptable(target, doer)`（SG 有 `stalker_corruption_pre` 且 `sg.mem.canstalkercorrupt`，或 batbosscave 的 `CanStalkerCorrupt`）→ 经 `RedirectStalkerCorruption` 重定向后 `PushEventImmediate("startcorruption")`，Discharge BLIGHTED 冷却；stalker 系硬直状态族由 commonstates 的 OnStalkerCorrupt 接入。[componentutil.lua:2473](../../../raw/current/scripts/componentutil.lua#L2473 [stategraphs/commonstates.lua:3006](../../../raw/current/scripts/stategraphs/commonstates.lua#L3006
2. `target.has_nightmare_state`（bunnyman/monkey/rabbit 设置并各自监听 `ms_forcenightmarestate`）→ 推 `{duration=99999}`，Discharge NIGHTMARE 冷却。[prefabs/bunnyman.lua:514](../../../raw/current/scripts/prefabs/bunnyman.lua#L514 [prefabs/monkey.lua:511](../../../raw/current/scripts/prefabs/monkey.lua#L511 [prefabs/rabbit.lua:436](../../../raw/current/scripts/prefabs/rabbit.lua#L436
3. `TheWorld.components.shadowthrallmanager:IsRegisteredFissure(target)` → `ControlFissure(target)` 成功再 `OnDreadstoneMineCooldown`，返回成败，Discharge FISSURE 冷却——即更新报告 C-37 的王者法杖强制腐化路径（thralltype 为空时随机 TRIO/MOUTH）。[components/shadowthrallmanager.lua:203](../../../raw/current/scripts/components/shadowthrallmanager.lua#L203 [components/shadowthrallmanager.lua:533](../../../raw/current/scripts/components/shadowthrallmanager.lua#L533
4. `chess_moonevent` 标签 → 推 `shadowchessroar`；5. `chess` 标签 → 推 `ms_becomeshadowchess`；两者同用 CHESSPIECE 冷却。发条棋子（knight/bishop/rook）监听 `ms_becomeshadowchess`，chesspieces 监听 `forcechessstruggle`/`shadowchessroar`。[prefabs/knight.lua:163](../../../raw/current/scripts/prefabs/knight.lua#L163 [prefabs/chesspieces.lua:363](../../../raw/current/scripts/prefabs/chesspieces.lua#L363

### 与 king_cane／moonbase 的分工

- 本组件＝数值账本＋目标登记＋目标类型分发，不含任何外观逻辑；king_cane＝宿主接线：三回调驱动 FX 分档（LOW 33/MED 67/HIGH 100 映射 king_cane_fx.level 净变量）、`shadow_item` 标签（LOW 起）、shadowdominance 组件与交易开关，并在 onspellcast/onblink/weapononprojectilelaunched 时按宝石喂 `DoDelta(TUNING.KING_CANE_CORRUPTION[gem])`（红蓝 3、橙 5、紫黄 15、绿 20）。[prefabs/king_cane.lua:189](../../../raw/current/scripts/prefabs/king_cane.lua#L189 [prefabs/king_cane.lua:355](../../../raw/current/scripts/prefabs/king_cane.lua#L355 [tuning.lua:9747](../../../raw/current/scripts/tuning.lua#L9747
- moonbase＝终局消费者：接受 king_cane 插入（`HasCorruptedKingStaff` 读 `IsCorrupted`），满月充能计时结束时若法杖已腐化，播放 shadow_despawn 并 `MakeUncorrupted()`——腐化王杖不转化为 opalstaff。[prefabs/moonbase.lua:31](../../../raw/current/scripts/prefabs/moonbase.lua#L31 [prefabs/moonbase.lua:459](../../../raw/current/scripts/prefabs/moonbase.lua#L459

## 典型接入

### 给自定义物品挂腐化值并监听三回调

- 前提与选择：模组 `modmain.lua`，目标物品 prefab 的服务端实例；组件与回调都只能在主端添加/注册。
- 生效与撤销：DoDelta/SetCurrent 立即生效；`corrupted` 标签网络同步可作客户端判定；卸载模组后存档数据 key 不再被读取。

```lua
AddPrefabPostInit("mystaff", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst:AddComponent("corruption")
    inst.components.corruption:SetOnCurrentFn(function(inst, current)
        print(("[mymod] corruption %d/%d"):format(current, GLOBAL.TUNING.KING_CANE_MAX_CORRUPTION))
    end)
    inst.components.corruption:SetOnCorruptedFn(function(inst)
        inst:AddTag("shadow_item")
    end)
    inst.components.corruption:SetOnUncorruptedFn(function(inst)
        inst:RemoveTag("shadow_item")
    end)
    inst:ListenForEvent("onspellcast", function()
        inst.components.corruption:DoDelta(10)
    end)
end)
```

- 调整与验证：上限、阈值、衰减与冷却全部读 `TUNING.KING_CANE_*`，改值走 AddGamePostInit/TUNING 覆盖；验证记录见下。

## 常见陷阱与限制

- **corrupted 态吸收一切写入**：`SetCurrent`/`DoDelta` 在已腐化时直接 return，`MakeUncorrupted` 只在原为腐化态时触发一次回调；连续 MakeCorrupted 则每次都触发 `oncorruptedfn`。[components/corruption.lua:65](../../../raw/current/scripts/components/corruption.lua#L65 [components/corruption.lua:44](../../../raw/current/scripts/components/corruption.lua#L44
- **OnLoad 实际不恢复 timetodecay**：守卫条件读的是新实例的 `self.timetodecay`（构造为 0），存档值恒被丢弃；读档后首个更新帧立即按周期结算一步 −1 衰减。[components/corruption.lua:191](../../../raw/current/scripts/components/corruption.lua#L191 [components/corruption.lua:5](../../../raw/current/scripts/components/corruption.lua#L5
- **客户端无组件**：客户端判定走网络化标签与 king_cane_fx 的 `level`（net_tinybyte，leveldirty 事件）；直接读 `components.corruption` 会得到 nil。[prefabs/king_cane.lua:558](../../../raw/current/scripts/prefabs/king_cane.lua#L558
- **负 delta 不重置衰减计时**：只有 delta>0 才把 timetodecay 重置为 12 秒；外部周期性扣减不会推迟自然衰减。[components/corruption.lua:88](../../../raw/current/scripts/components/corruption.lua#L88

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件 209 行全部逻辑、king_cane 三回调与三个喂入事件、moonbase 消费分支、componentactions/actions/SGwilson castspell 接线、componentutil.IsStalkerCorruptable、shadowthrallmanager 裂缝接口、tuning KING_CANE_* 键值与上列事件消费方锚点；依据见上文。复用 build-26148036 更新报告已核验条目 C-37（强制腐化路径）与 R-01/R-04（动作接线），引用前已对照当前源码。未覆盖：各消费方完整行为、多分片表现、游戏运行。
- 游戏运行：未运行。
- 结构检查：2026-10-26 执行 validate_page.py（standard 档）与 validate_source_links.py（--strict）通过；页内 Lua 示例经 parser_probe.exe --syntax-only 语法检查通过。

相关：[[reference/components/rechargeable]]、[[reference/components/blinkstaff]]（冷却与施法喂入方）。
