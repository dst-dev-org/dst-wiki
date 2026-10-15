---
title: "弹弓元素弹药附带表现（slingshotammo_purebrilliance_debuff_fx 等 3 注册）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "slingshotammo_debuffs.lua 的 3 个弹弓命中后表现注册：纯辉标记的前后双层跟随标记（载体 mark 为 CLASSIFIED 非联网逻辑实体）、恐惧燃料 DoT 的每跳恐怖幻影（混合载体：FX 自带 planardamage 并池化复用）与减速贴片的纯视觉随层；生成链、伤害归属与端模型分档辨析。"
aliases: ["slingshotammo_purebrilliance_debuff_fx", "slingshotammo_horrorfuel_debuff_fx", "slingshotammo_slow_debuff_fx", "slingshotammo_purebrilliance_debuff", "辉煌闪现", "群聚恐怖", "弹弓纯辉标记", "弹弓恐惧DoT", "弹弓减速", "纯粹辉煌弹药", "纯粹恐惧弹药", "减速弹药"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/slingshotammo_debuffs.lua", "prefabs/slingshotammo.lua", "components/debuffable.lua", "components/debuff.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 弹弓元素弹药附带表现（slingshotammo_purebrilliance_debuff_fx 等 3 注册）

## 快速判断

- 职责与适用：`slingshotammo_debuffs.lua` 返回 4 个注册（[返回](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L595-L599)），本页覆盖其中 **3 个 FX 注册**：`slingshotammo_purebrilliance_debuff_fx`（纯辉标记的双层跟随标记）、`slingshotammo_horrorfuel_debuff_fx`（官方名"群聚恐怖"，恐惧燃料 DoT 的每跳幻影）、`slingshotammo_slow_debuff_fx`（减速弹药命中贴片）；第 4 个 `slingshotammo_purebrilliance_debuff`（官方名"辉煌闪现"）是 mark **玩法载体**，见[载体辨析](#本体辨析辉煌闪现-mark-载体不计-fx-覆盖)。
- 三种载体模式：①**两层跟随标记**（purebrilliance fx——front FinalOffset 1／back FinalOffset -1，各自 Follower 跟目标符号，声音循环在 front 上）；②**池化 DoT 幻影**（horrorfuel fx——自带 planardamage／damagetypebonus／deflectable，按目标池化复用，**伤害由 FX 实体结算**）；③**纯视觉贴片**（slow fx——零玩法组件，减速改在宿主 locomotor 上）。
- 重要副作用：三个 FX 都 `persists=false` 不入存档；horrorfuel fx 挂 `deflectable`——被弹反一次后整个 DoT 剩余跳数不再结算（[标记位](../../../raw/current/scripts/prefabs/slingshotammo.lua#L487-L488)）；purebrilliance 标记只在目标**受到平面伤害攻击**时反击。
- 修改边界：只覆盖 3 注册与 mark 载体的工厂、生成点、端模型与清理；弹药投掷／命中主流程、弹弓本体、`purebrilliance_mark_hit_fx`（另有页面）、`slingshot_aoe_fx`（另有页面）不展开。
- 名称：官方名 `STRINGS.NAMES.SLINGSHOTAMMO_HORRORFUEL_DEBUFF_FX`＝"群聚恐怖"（[po](../../../raw/current/scripts/languages/chinese_s.po#L349499-L349502)）、`STRINGS.NAMES.SLINGSHOTAMMO_PUREBRILLIANCE_DEBUFF`＝"辉煌闪现"（[po](../../../raw/current/scripts/languages/chinese_s.po#L349529-L349532)）；弹药名"纯粹辉煌弹药"／"纯粹恐惧弹药"／"减速弹药"（[锚点见验证记录](#验证记录)）；`SLINGSHOTAMMO_PUREBRILLIANCE_DEBUFF_FX` 与 `SLINGSHOTAMMO_SLOW_DEBUFF_FX` 无 `STRINGS.NAMES.*` 键，正文相应效果描述**非官方译名**。

## 注册与构造

### 本体辨析（辉煌闪现 mark 载体，不计 FX 覆盖）

`slingshotammo_purebrilliance_debuff`（[工厂](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L227-L268)）是纯辉弹命中后挂到目标身上的**玩法载体**：非联网（客户端生成即删，[早退](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L230-L235)）、标签 `CLASSIFIED`／`attacktriggereddebuff`／`notarget`，带 `planardamage`（基础 5，[L251-L252](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L251-L252)）、`damagetypebonus`（对 `shadow_aligned` 目标 ×1.1）与 `debuff` 组件（keepondespawn，[L257-L261](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L257-L261)）。挂载（`OnAttached`）时把本页主 FX 生成并父挂、监听宿主 `attacked`；超时 `TUNING.SLINGSHOT_BRILLIANCE_MARK_TIMEOUT`＝30 秒或宿主死亡时 `debuff:Stop` 解绑；`OnExtended`（重复命中）只刷新超时任务。

### 三个 FX 注册

| 注册名 | 工厂壳 | 差异与证据 |
|---|---|---|
| `slingshotammo_purebrilliance_debuff_fx` | Transform／AnimState／SoundEmitter／Follower／Network；bank／build `slingshotammo_purebrilliance_mark_fx`；初始播 `fx_front_pre_2`＋`fx_front_loop_2`；泛光 shader、LightOverride 0.3、FinalOffset 1；标签 `FX`；`persists=false`；`OnRemoveEntity` 移除 back 层 | 双层结构：`AttachTo` 时各生成一枚 front（本体）＋back（FinalOffset -1）、都父挂目标并 `FollowSymbol`，按目标体形 clamp 1–3 选 `fx_front/back_pre/loop_<size>` 动画，front 播 `ammo_pstfx_purebrilliance_lp` 声音循环。[工厂](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L334-L369)、[AttachTo](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L296-L326) |
| `slingshotammo_horrorfuel_debuff_fx` | Transform／AnimState／SoundEmitter／Follower／Network；bank／build `slingshot_shadowcurse`、FinalOffset 7、`parts_red` 符号 LightOverride 1；标签 `FX`／`NOCLICK`／`notarget`／`pseudoprojectile`；显示名 override 指向 `slingshotammo_horrorfuel`；主模拟加 `planardamage`（基础 10）、`damagetypebonus`（`lunar_aligned` ×1.1）、`deflectable`；`persists=false` | **混合载体**：视觉之外还承载 DoT 伤害与弹反判定；`Restart` 方法按变体重播 `idle<1–6>` 动画、排 28 帧攻击任务与动画长度退场任务、Follower 跟 `combat.hiteffectsymbol`、播 `ammo_pstfx_purehorror`。[工厂](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L462-L504)、[Restart](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L420-L460) |
| `slingshotammo_slow_debuff_fx` | Transform／AnimState／Network（Follower 注释掉）；bank／build `slingshotammo_slow_fx`、FinalOffset 7；标签 `FX`／`NOCLICK`；主模拟 `_level=1`、0 帧兜底自删任务、`StartFX`／`SetFXLevel`／`KillFX` 三方法；`persists=false` | 纯视觉：动画名 `slow_<pre/loop/pst>_<size>_<level>` 按目标体形（small／med／large）与层数 1–3 拼名；`StartFX` 可带 0–0.3 秒随机延迟（非主目标错峰）；`SetFXLevel` 重播 loop 并随机起帧。[工厂](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L561-L591)、[StartFX](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L512-L538) |

符号选择协议（purebrilliance fx 专属，[GetBestSymbolAndSize](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L272-L294)）：依次取目标 `burnable.fxdata[1].follow`、`freezable.fxdata[1].follow`、`combat.hiteffectsymbol`；体形档位按 `smallcreature`→1、`largecreature`/`epic`→3、其余→2。

## 关联机制导航

### 生成点与宿主调用

| 注册 | 生成语句（file:line） | 消费文件与条件 |
|---|---|---|
| `slingshotammo_purebrilliance_debuff_fx` | front：[debuffs.lua#L194](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L193-L198)（mark `OnAttached`）；back：[debuffs.lua#L297](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L296-L301)（fx `AttachTo` 内部再生成一枚） | 弹药链：纯辉弹命中 `OnHit_PureBrilliance` → `DoHit_PureBrilliance` → `target:AddDebuff("ammo_purebrilliance_mark", "slingshotammo_purebrilliance_debuff")`（[slingshotammo.lua#L845](../../../raw/current/scripts/prefabs/slingshotammo.lua#L841-L851)）→ `debuffable` 生成 mark 并 `AttachTo`（[debuffable.lua#L95-L108](../../../raw/current/scripts/components/debuffable.lua#L95-L108)）→ 上表两个 SpawnPrefab |
| `slingshotammo_horrorfuel_debuff_fx` | [slingshotammo.lua#L482](../../../raw/current/scripts/prefabs/slingshotammo.lua#L477-L489)（池空时才 Spawn，池里有货则取出 `ReturnToScene` 复用） | 恐惧弹命中 `OnHit_HorrorFuel` → `DoHit_HorrorFuel` 起周期任务 → 每跳 `OnUpdate_HorrorFuel` 取变体、取池／生成、父挂、设弹反回调、`Restart`（[L465-L501](../../../raw/current/scripts/prefabs/slingshotammo.lua#L465-L501)） |
| `slingshotammo_slow_debuff_fx` | [slingshotammo.lua#L352](../../../raw/current/scripts/prefabs/slingshotammo.lua#L347-L367)（目标首次中减速时生成并父挂） | 减速弹命中 `OnHit_Slow` → `DoHit_Slow`（`target._slingshot_slow` 数据表，叠加刷新） |

弹药归属（[定义](../../../raw/current/scripts/prefabs/slingshotammo.lua#L1415-L1520)）：`slingshotammo_slow`（[L1415](../../../raw/current/scripts/prefabs/slingshotammo.lua#L1415-L1424)）、`slingshotammo_purebrilliance`（[L1479](../../../raw/current/scripts/prefabs/slingshotammo.lua#L1479-L1494)，skill `walter_ammo_lunar`，canmagicamp）、`slingshotammo_horrorfuel`（[L1495](../../../raw/current/scripts/prefabs/slingshotammo.lua#L1495-L1520)，skill `walter_ammo_shadow`，canmagicamp）；三者的 `prefabs` 依赖表都同时声明 `slingshot_aoe_fx`（魔法增幅圈，另有页面）与本页注册——声明不等于生成，实际生成都在上表。

### 恐惧燃料 DoT 链（horrorfuel）

[常量](../../../raw/current/scripts/prefabs/slingshotammo.lua#L453-L456)：6 个动画变体、单目标最多 4 跳并发（`MAX_HORRORS`，超出淘汰最旧任务）、跳间隔 1 秒、首跳随机延迟 0–0.35 秒；跳数 `TUNING.SLINGSHOT_HORROR_TICKS`＝7，暗夜剑套装 bonus 时 9（[tuning](../../../raw/current/scripts/tuning.lua#L4891-L4893)）。

1. `DoHit_HorrorFuel`（[L503-L528](../../../raw/current/scripts/prefabs/slingshotammo.lua#L503-L528)）：目标红闪、建/续 `_slingshot_horror` 数据表（变体洗牌池＋任务列表＋FX 对象池），按跳数排 `DoPeriodicTask`。
2. 每跳 `OnUpdate_HorrorFuel`（[L465-L501](../../../raw/current/scripts/prefabs/slingshotammo.lua#L465-L501)）：目标存活且可被攻击时取一个未用变体，从对象池取 FX（`RemoveFromScene` 状态回收，[RecycleHorrorDebuffFX](../../../raw/current/scripts/prefabs/slingshotammo.lua#L458-L463)）或新生成，父挂目标、`deflectable:SetOwner(attacker)`＋弹反回调置 `data.deflected`，然后 `Restart(attacker, target, variation, ...)`。
3. `Restart`（[debuffs.lua#L420-L460](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L420-L460)）：播 `idle<variation>`（quick 时直接跳到第 3 帧）、排 28 帧后的 `HorrorFuel_DoAttack` 与动画长度后的 `HorrorFuel_AnimOver`。
4. `HorrorFuel_DoAttack`（[L375-L396](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L375-L396)）：**由 FX 实体发起攻击**——`target.components.combat:GetAttacked(inst, 0, nil, nil, spdmg)`（普通伤害 0＋planar 10，对 `lunar_aligned` 目标 ×1.1），目标无仇恨目标时再以射手名义推 `attacked` 事件；随后目标红闪。`inst` 已失效（被弹反）则跳过闪红。
5. `HorrorFuel_AnimOver`（[L398-L418](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L398-L418)）：有池且仍属同一 DoT → `CancelFlash`＋`StopFollowing`＋回池；否则 `Remove`。全部跳数结束后池内 FX 逐一 `Remove`（[L492-L499](../../../raw/current/scripts/prefabs/slingshotammo.lua#L492-L500)）。

### 减速链（slow）

`DoHit_Slow`（[L347-L373](../../../raw/current/scripts/prefabs/slingshotammo.lua#L347-L373)）：每层压入一个 `TUNING.SLINGSHOT_AMMO_MOVESPEED_DURATION`＝30 秒的任务，层数上限 `TUNING.SLINGSHOT_AMMO_MOVESPEED_MAX_STACKS`＝3；`SetSpeed_Slow`（[L316-L327](../../../raw/current/scripts/prefabs/slingshotammo.lua#L316-L327)）按层数设 locomotor 外部速度倍率 `(2/3)^层数`（[tuning](../../../raw/current/scripts/tuning.lua#L4863-L4865)）并同步 `fx:SetFXLevel`；与黏弹（gelblob）叠乘时取 min(1, …)（[L318-L324](../../../raw/current/scripts/prefabs/slingshotammo.lua#L318-L324)）。层数归零时 `KillFX`＋移除外部倍率。

### 辉煌闪现反击链（purebrilliance mark）

mark 挂上后监听宿主 `attacked`（[L197](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L193-L198)）；`PureBrilliance_OnOwnerAttacked`（[L140-L180](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L140-L180)）的触发条件：攻击者或武器带**平面伤害**（`SpDamageUtil.CollectSpDamage` 含 `planar`）、非 redirect、非同类 debuff 自触发。触发时 mark 以自身为攻击源结算 `GetAttacked(inst, 0, nil, nil, spdmg)`（planar 5，对 `shadow_aligned` 目标 ×1.1），再生成 `purebrilliance_mark_hit_fx`（[L112](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L107-L136)，按攻击者方位环绕布点，已有页面覆盖）并给目标白闪。

### 清理路径

| 注册 | 退场 | 边界与证据 |
|---|---|---|
| purebrilliance fx（front／back） | mark 解绑（超时／死亡）时 front 播 `fx_front_pst_<size>`、back 追播 `fx_back_pst_<size>`，`animover` 自删；front 被移除时 `OnRemoveEntity` 直接删 back | [OnDetached](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L201-L216)、[OnRemoved](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L328-L332)；声音循环随 front 移除停止 |
| horrorfuel fx | `animover` → 回池（`RemoveFromScene` 不销毁）或 `Remove`；DoT 结束清池 | [AnimOver](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L398-L418)、[清池](../../../raw/current/scripts/prefabs/slingshotammo.lua#L492-L499)；目标被移除时 FX 脱离父级留在原地（[脱离](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L448-L456)） |
| slow fx | `KillFX` 播 `slow_..._pst` 后按动画长度定时自删（睡眠也能删）；从未 StartFX 则 0 帧兜底自删 | [KillFX](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L550-L559)、[兜底](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L583-L583) |
| mark 载体 | `debuff:Stop` → `RemoveDebuff` → `debuff:OnDetach` → `OnDetached`（解绑监听＋自删）；`keepondespawn`＝跟随下线的宿主 | [debuff.lua](../../../raw/current/scripts/components/debuff.lua#L28-L32)、[OnDetached 尾](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L211-L216) |

## 典型接入

### 给目标挂一套"命中后持续表现＋数值"效果

- 前提与选择：想要"挂件持续存在、宿主侧持有逻辑"就复刻 mark＋fx 两件套（mark 是非联网逻辑实体，fx 是联网表现壳）；想要"周期跳数、池化省开销"就复刻 horrorfuel 模式；纯视觉叠层用 slow 模式即可。
- 生效与撤销：mark 走 `debuffable:AddDebuff`（重复命中自动 `Extend` 刷新超时）；horrorfuel 池化 FX 必须置 `inst.pool`／`inst.onrecyclefn` 才能回池，否则 `animover` 直接 `Remove`。
- 原版依据：[AddDebuff 写法](../../../raw/current/scripts/prefabs/slingshotammo.lua#L845-L845)、[池化写法](../../../raw/current/scripts/prefabs/slingshotammo.lua#L477-L489)。

```lua
local function ApplySlowVisual(target)
    local fx = SpawnPrefab("slingshotammo_slow_debuff_fx")
    if fx ~= nil then
        fx.entity:SetParent(target.entity)
        fx:StartFX(target, nil)     -- nil＝主目标立即播；数值 0~0.3 为错峰延迟
        fx:SetFXLevel(1)            -- 层数 1~3，改播对应 loop 动画
    end
end
```

- 调整与验证：替换点为注册名与层数；`StartFX`／`SetFXLevel`／`KillFX` 是 slow fx 的实例方法。减速数值不在 FX 上，需要另调 `locomotor:SetExternalSpeedMultiplier`。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：horrorfuel 的伤害源是 FX 实体，不是弹药。** 每跳 planar 10 由 FX 上的 planardamage 组件结算（攻击源＝FX）；普通伤害为 0，仇恨事件才以射手名义推送。给 FX 上的伤害做加成要动 `planardamage`／`damagetypebonus`，不是改弹药伤害。[攻击](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L375-L396)
- **边界：弹反一次即中断整个 DoT 的伤害。** FX 带 `deflectable`，弹反回调把 `data.deflected` 置真后，后续所有跳都跳过生成与伤害（任务本身还会走完计时）。[标记位](../../../raw/current/scripts/prefabs/slingshotammo.lua#L466-L490)
- **边界：purebrilliance fx 的 back 层是 fx 自己生成的。** mark 只生成 front；back 由 front 的 `AttachTo` 内部 `SpawnPrefab`——直接 `SpawnPrefab("slingshotammo_purebrilliance_debuff_fx")` 不调 `AttachTo` 只会得到一枚 front 壳（无 back、无跟随、无声音循环语义）。[AttachTo](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L296-L326)
- **边界：slow fx 的减速不在 FX 上。** FX 只是随层换动画的贴片；真正的减速是宿主 locomotor 外部倍率，且只对**有 locomotor 的目标**生效（`DoHit_Slow` 的入口判定）。[判定](../../../raw/current/scripts/prefabs/slingshotammo.lua#L347-L348)
- **边界：horrorfuel fx 会脱离目标。** 目标实体被移除时 FX `StopFollowing`＋脱离父级留在原地（池引用清空），动画走完后正常退场——不是随目标消失。[脱离](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L448-L456)
- **边界：mark 是客户端不可见的。** `CLASSIFIED`＋非联网＋客户端生成即删；任何"让客户端看到 mark"的改法都要走 fx 双层壳的 netvar，不要直接改 mark。[早退](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L230-L235)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/slingshotammo_debuffs.lua` 全文（4 工厂＋共享 flash／expire 函数）；按需读取 `prefabs/slingshotammo.lua` 的 slow／horror／purebrilliance 命中函数、DoT 常量与池化、三个弹药定义段，`components/debuffable.lua` 的 AddDebuff／RemoveDebuff，`components/debuff.lua` 的 AttachTo／OnDetach／Stop，`tuning.lua` 的 `SLINGSHOT_AMMO_MOVESPEED_*`／`SLINGSHOT_HORROR_*`／`SLINGSHOT_BRILLIANCE_MARK_*`／`SLINGSHOT_AMMO_VS_*_BONUS`。全 `raw/current/scripts` 反查（`grep -rn "注册名" --include=*.lua`）：`slingshotammo_purebrilliance_debuff_fx` 命中 6 行（debuffs.lua 依赖 L15、生成 L194/L297、注册 L597；slingshotammo.lua 弹药依赖 L1490）——两处生成均在 debuffs.lua 内；`slingshotammo_horrorfuel_debuff_fx` 命中 4 行（slingshotammo.lua 生成 L482、依赖 L1519；debuffs.lua 注册 L598）；`slingshotammo_slow_debuff_fx` 命中 4 行（slingshotammo.lua 生成 L352、依赖 L1420；debuffs.lua 注册 L599）；`slingshotammo_purebrilliance_debuff` 另命中 slingshotammo.lua:845（AddDebuff）与 L1490（依赖表）。清单外连带：`purebrilliance_mark_hit_fx`（debuffs.lua:112 生成）由 [[reference/prefabs/abigail_attack_shadow_fx|温蒂、阿比盖尔与纯辉标记特效]] 页覆盖，本页不展开；`slingshot_aoe_fx` 由 [[reference/prefabs/slingshot_ice_aoe_fx|弹弓 AOE 命中圈]] 页覆盖。
- 名称核验：`languages/chinese_s.po` 定向检索——`SLINGSHOTAMMO_HORRORFUEL_DEBUFF_FX`＝"群聚恐怖"（[L349499-L349502](../../../raw/current/scripts/languages/chinese_s.po#L349499-L349502)）、`SLINGSHOTAMMO_PUREBRILLIANCE_DEBUFF`＝"辉煌闪现"（[L349529-L349532](../../../raw/current/scripts/languages/chinese_s.po#L349529-L349532)）、`SLINGSHOTAMMO_PUREBRILLIANCE`＝"纯粹辉煌弹药"（[L349524-L349527](../../../raw/current/scripts/languages/chinese_s.po#L349524-L349527)）、`SLINGSHOTAMMO_HORRORFUEL`＝"纯粹恐惧弹药"（[L349494-L349497](../../../raw/current/scripts/languages/chinese_s.po#L349494-L349497)）、`SLINGSHOTAMMO_SLOW`＝"减速弹药"（[L349544-L349547](../../../raw/current/scripts/languages/chinese_s.po#L349544-L349547)）；`SLINGSHOTAMMO_PUREBRILLIANCE_DEBUFF_FX`／`SLINGSHOTAMMO_SLOW_DEBUFF_FX` 无 `STRINGS.NAMES.*` 键，正文相应描述为效果描述。
- 分档说明：`size_class: standard`——3 注册同文件、无多阶段生命周期；复杂度集中在 horrorfuel 池化 DoT 与 mark 载体辨析，一个 DoT 流程小节可承载，不需要整体拆页。实测行数与字节见结构检查。
- 未覆盖：弹药投掷／装填与弹弓本体玩法、`magicamplified` 魔法增幅圈（另一页面）、`purebrilliance_mark_hit_fx` 工厂（另一页面）、Wurt 的 `purebrilliance/horrorfuel_castfx` 施法动画（另一页面）、horrorfuel 弹药自身悬浮 FX（`CreateFX_HorrorFuel`）、动画资源内容与观感。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/slingshot_ice_aoe_fx|弹弓 AOE 命中圈]]（魔法增幅命中的地面指示圈；与本页 debuff 表现分工不同）
- [[reference/prefabs/slingshotammo_hitfx_rock|弹弓弹药命中表现]]（常规命中的撞击 FX）
- [[reference/prefabs/abigail_attack_shadow_fx|温蒂、阿比盖尔与纯辉标记特效]]（`purebrilliance_mark_hit_fx` 的工厂与通用表条目）
- [[reference/prefabs/wurt_casting_fx|沃特施法特效]]（同名 fuel 前缀的施法动画，注册名不同）
- 返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]
