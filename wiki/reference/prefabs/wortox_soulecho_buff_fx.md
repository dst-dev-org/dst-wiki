---
title: "沃拓克斯灵魂增益与装备跟随特效族（wortox_soulecho_buff_fx 等 6 注册）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "沃拓克斯技能与装备外围的 6 个 FX 注册：灵魂回响增益的粒子跟随层与灵魂过载警告粒子（都由 debuff/掉魂链生成、经引擎父挂跟随玩家）、强抢袋的三层结构（联网 body 载体＋本地机体动画层＋本地灵魂粒子层，尺寸与灵魂档位走净变量）与双尾心施法期间的联网跟随体。含未注册死代码 wortox_panflute_buff_fx_fn 的辨析。"
aliases: ["wortox_soulecho_buff_fx", "wortox_overloading_fx", "wortox_nabbag_body", "wortox_nabbag_body_fx", "wortox_nabbag_body_soulfx", "wortox_reviver_body", "灵魂回响增益特效", "灵魂过载粒子", "强抢袋机体层", "强抢袋灵魂粒子", "双尾心跟随体", "强抢袋"]
tags: [prefab, fx, item, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/wortox.lua", "prefabs/wortox_nabbag.lua", "prefabs/wortox_reviver.lua", "stategraphs/SGwilson.lua", "fx.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 沃拓克斯灵魂增益与装备跟随特效族（wortox_soulecho_buff_fx 等 6 注册）

## 快速判断

- 职责与适用：6 个注册分三类载体——①`wortox_soulecho_buff_fx`（灵魂回响增益的白色灵魂粒子层，debuff 实体生成）、②`wortox_overloading_fx`（顽皮性情灵魂过载前的警告粒子层，掉魂判定链生成）、③⑤⑥`wortox_nabbag_body`／`wortox_nabbag_body_fx`／`wortox_nabbag_body_soulfx`（强抢袋装备期的三层结构：联网 body 载体→本地机体动画层→本地灵魂粒子层）、④`wortox_reviver_body`（双尾心施法期间挂在持有者 `swap_remote` 符号上的联网跟随体）。台账所记 5 注册经回源确认实为 **6 注册**（nabbag 家族是 3 个）。
- 优先使用：查"灵魂回响buff的粒子/加速"用 soulecho 链；查"灵魂太多冒白烟警告"用 overloading 链；查"强抢袋怎么随走路/攻击换动画、灵魂档位发光"用 nabbag 三层；查"捏双尾心传送时手里的跟随体"用 reviver_body。
- 重要副作用：两个粒子 FX **都无自删**，由生成方引用移除（回响经 debuff Detach、过载经 DestroyOverloadingFX）；nabbag 的 body 载体在换装/卸装时被整只重建；reviver_body 的显隐完全由 SGwilson 施法状态帧事件控制。
- 修改边界：只覆盖 6 注册的工厂、生成点、父子/跟随与清理；沃拓克斯技能树、灵魂经济（wortox_soul/souljar）、强抢袋捕网玩法与双尾心传送玩法不展开。
- 名称：宿主官方名 `WORTOX_NABBAG`＝"强抢袋"（[po](../../../raw/current/scripts/languages/chinese_s.po#L352494-L352497)）、`WORTOX_REVIVER`＝"双尾心"（[po](../../../raw/current/scripts/languages/chinese_s.po#L352499-L352502)）；6 个注册名 `WORTOX_SOULECHO_BUFF_FX`／`WORTOX_OVERLOADING_FX`／`WORTOX_NABBAG_BODY`／`WORTOX_NABBAG_BODY_FX`／`WORTOX_NABBAG_BODY_SOULFX`／`WORTOX_REVIVER_BODY` 均无 `STRINGS.NAMES.*` 键，标题中文为已核验效果描述，**非官方译名**。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wortox_soulecho_buff_fx` | `soulecho_buff_fx_fn`：Transform／Network（**无 Follower**）、标签 `FX`；`persists=false`；非专服 VFXEffect 粒子（`fx/soul.tex` 10 帧 UV、200 上限、寿命 1 秒、白→透明包络、泛光）；发射率按宿主位移插值（约 3～30 粒/秒，源码写作 `3*tick_time`～`30*tick_time`） | 粒子层靠引擎父挂跟随目标；尺度包络 0.4→0.2 收缩；专服只剩空壳（本地发射）。[工厂](../../../raw/current/scripts/prefabs/wortox.lua#L931-L941)、[发射器](../../../raw/current/scripts/prefabs/wortox.lua#L974-L999)、[注册](../../../raw/current/scripts/prefabs/wortox.lua#L1602) |
| `wortox_overloading_fx` | `overloading_fx_fn`：Transform／Network／Follower、标签 `FX`；`persists=false`；同纹理同包络方向的粒子，但尺度 0.5→1 放大、寿命 0.5 秒、发射率同样按位移插值（约 3～30 粒/秒） | 与 soulecho 共用纹理与相近骨架，但另有出生分布／速度差异（见陷阱），不是只差尺度包络／寿命／Follower；粒子从 `py+random` 高处散落。[工厂](../../../raw/current/scripts/prefabs/wortox.lua#L1051-L1065)、[发射器](../../../raw/current/scripts/prefabs/wortox.lua#L1095-L1113)、[注册](../../../raw/current/scripts/prefabs/wortox.lua#L1603) |
| `wortox_nabbag_body` | `bodyfn`：Transform／Follower／Network、标签 `FX`；净变量 `bodyvfx_souls`（net_tinybyte 档位）＋`bodysize_netvar`（net_tinybyte 桶位，dirty 事件名 sizedirty）；非专服挂 updatelooper 每帧 `OnUpdate_body`；主模拟 `persists=false` | **body 载体**：自身无 AnimState，负责把袋子的桶位/灵魂档位同步给客户端并驱动本地两层。[工厂](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L352-L384)、[注册](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L510) |
| `wortox_nabbag_body_fx` | `bodyfxfn`：Transform／AnimState／Follower；标签 `CLASSIFIED`（非联网实体）；四向；bank/build `wortox_nabbag`、`idle_body_empty` 循环、FinalOffset 1 | **机体动画层**：按宿主状态播 `idle_body<size>`／`loop_swing<size>`／`attack<size>`／`pst_swing<size>` 拼名动画；换桶位时保帧切换。[工厂](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L323-L344)、[注册](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L511) |
| `wortox_nabbag_body_soulfx` | `bodysoulfxfn`：Transform／Network、标签 `FX`；`persists=false`；VFXEffect 粒子（同 `fx/soul.tex`、发射率查表 [0, 0.2, 0.6, 1]×tick） | **灵魂粒子层**：发射器读父级（body 载体）的 `bodyvfx_souls` 净变量定速率，父级无效时不发射。[工厂](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L443-L507)、[速率表](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L436-L441) |
| `wortox_reviver_body` | `bodyfn`：Transform／AnimState／Network、标签 `FX`；bank/build `wortox_reviver`、`idle_body` 循环、FinalOffset 1；主模拟 `persists=false` | 双尾心捏在手里施法时的**联网跟随体**；跟随符号/偏移由生成方 `OnStartBody` 挂接。[工厂](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L269-L292)、[注册](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L295) |

## 关联机制导航

### soulecho／overloading 的 buff 链生成点

| 生成链 | 触发与条件 | 证据 |
|---|---|---|
| 灵魂回响 | `TryToPortalHop` 免费跳未花完灵魂配额且技能 `wortox_liftedspirits_1` 激活 → `AddDebuff("wortox_soulecho_buff", ..., {duration})` → debuff 实体（同文件注册 [wortox_soulecho_buff](../../../raw/current/scripts/prefabs/wortox.lua#L1601)，master-only 无网络）的 `OnAttached_soulecho` 父挂目标、启动 buffover 计时、加 1.2 倍速并生成 fx | [AddDebuff](../../../raw/current/scripts/prefabs/wortox.lua#L614-L645)、[OnAttached](../../../raw/current/scripts/prefabs/wortox.lua#L821-L835)、[速度调值](../../../raw/current/scripts/tuning.lua#L7415-L7415) |
| 灵魂过载警告 | `CheckForOverload` 超出容量（基础 20，souljar 扩容另计）且性情为 naughty 且非强制复查 → `CreateOverloadingFX`：生成 fx 父挂宿主、`FollowSymbol(hiteffectsymbol)`、起 10 秒强制复查任务 | [CheckForOverload](../../../raw/current/scripts/prefabs/wortox.lua#L313-L345)、[CreateOverloadingFX](../../../raw/current/scripts/prefabs/wortox.lua#L304-L311)、[停止时限](../../../raw/current/scripts/tuning.lua#L7391-L7391) |
| 过载落魂 | 10 秒后 `ForceCheckOverload` 置 `_forceoverloading` 复查，或非 naughty 直接结算：掉魂＋理智扣减（警告层由 `DestroyOverloadingFX` 移除） | [ForceCheckOverload](../../../raw/current/scripts/prefabs/wortox.lua#L297-L303)、[DestroyOverloadingFX](../../../raw/current/scripts/prefabs/wortox.lua#L285-L293) |

回响 buff 的结束路径汇于 `OnDetached_soulecho`：移除速度乘数、`bufffx:Remove()`、debuff 实体自删（[L837-L846](../../../raw/current/scripts/prefabs/wortox.lua#L837-L846)）；进入路径是 `FinishPortalHop` 的 `RemoveDebuff`（[L580-L594](../../../raw/current/scripts/prefabs/wortox.lua#L580-L594)）或 buffover 计时到点（[L856-L860](../../../raw/current/scripts/prefabs/wortox.lua#L856-L860)）或目标死亡监听（[L827-L829](../../../raw/current/scripts/prefabs/wortox.lua#L821-L835)）。

### 强抢袋三层的父子与渲染分工

| 层 | 载体角色 | 挂接与数据 | 证据 |
|---|---|---|---|
| `wortox_nabbag`（宿主物品，本页不覆盖注册） | 可装备捕网物品；netvar 写入方 | `OnEquip` 移除旧 body 后重建并父挂持有者（[L115-L140](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L115-L140)）；`UpdateStats` 把库存占比折成三桶（`_empty/_medium/_full`）写 `bodysize_netvar`，灵魂数折成 0–3 档写 `bodyvfx_souls`（[L22-L68](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L22-L68)、[伤害调值](../../../raw/current/scripts/tuning.lua#L7433-L7434)） |
| `wortox_nabbag_body` | 联网载体：同步桶位/灵魂档位 | 父挂持有者；`OnUpdate_body`（仅非专服）首次更新时生成 body_fx 并 `FollowSymbol(owner, "swap_object", 54, -182)`，再在 body_fx 上挂 soulfx（[L269-L321](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L269-L321)，生成 [L273-L281](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L269-L321)） |
| `wortox_nabbag_body_fx` | 本地机体动画层：走路/攻击/停步拼名动画 | 父挂 body 载体（引擎子级）；每帧按持有者 `moving/nopredict/attacking` 状态切换动画，桶位 dirty 时保帧换尺寸（[OnSizeDirty_body](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L249-L267)） |
| `wortox_nabbag_body_soulfx` | 本地灵魂粒子层：档位发光 | 父挂 body_fx；发射率查 `bodyvfx_souls` 净变量（0=不发光，[L495](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L484-L504)）；`wortox_souljar_3` 技能下按总灵魂数分 3 档（[L36-L45](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L22-L68)、[上限调值](../../../raw/current/scripts/tuning.lua#L7449-L7449)） |

持握符号分工：nodangle/骑乘等状态由 `ToggleOverrideSymbols` 把袋子切到持有者 `swap_object` 覆盖（body 隐藏），否则覆盖成 `swap_wortox_nabbag_rope` 并显示 body 层（[L99-L114](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L99-L114)）——**袋子本体与 body 层不同时可见，绳子符号替代**。

### wortox_reviver_body 的跟随与显隐

生成/跟随：双尾心（物品页见 [[reference/prefabs/wortox-reviver]]）的 `OnStartBody` 生成 body、父挂持有者并 `FollowSymbol(owner.GUID, "swap_remote", 25, -42, 0)`（[L183-L193](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L183-L193)）。触发：普通 `CASTSPELL` 由 ActionHandler 对带 `crushitemcast` 的物品选 `crushitemcast` 状态（[选态 L1303-L1310](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1303-L1310)），该状态**第 9 帧**调 `OnStartBody` 挂 body（[L16676-L16682](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16676-L16682)），**第 18 帧只播** `crushitemcast_sound`（[L16683-L16689](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16683-L16689)）；`crushitemcast_holding`／`_trigger`（[L16462-L16646](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16462-L16462)）属 `STARTVAULTORBTELEPORT` 链，与本件无关。移除：`OnStopBody` 由 `crushitemcast` 的非 `crushcasting` onexit（[L16705-L16713](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16705-L16713)）、`crushitemcast_fail` 的 onexit（[L16739-L16745](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16739-L16745)）、传送状态 onexit（[L16823-L16824](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16823-L16824)）、自用状态 onenter（[L16758-L16759](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16758-L16759)）调用。反向监听方向：双尾心监听的是 **body 的 `onremove`**，body 被移除时清掉双尾心上的 `wortox_reviver_body` 引用（[OnRemove_Body L180-L182](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L180-L182)），不是双尾心本体移除时触发；引擎父子回收仍单列待核验。

### wortox_panflute_buff_fx_fn 未注册死代码辨析

`wortox.lua:1254-1277` 定义了 `wortox_panflute_buff_fx_fn`（bank/build `wortox_soul_heal_fx`、anim `heal`、FinalOffset 3、缩放 1×0.75、半速播放），但文件返回块的注册清单（[L1600-L1606](../../../raw/current/scripts/prefabs/wortox.lua#L1600-L1606)）只注册 `wortox_panflute_buff`（debuff 实体，[L1231](../../../raw/current/scripts/prefabs/wortox.lua#L1231) 带 FIXME 注释）——**该 fx_fn 无任何 `Prefab()` 引用，全库 grep 无 `wortox_panflute_buff_fx` 注册与生成**，属于未完成的 VFX 占位。实际的音乐性情增益图标走 `player_classified.wortox_panflute_buff` 净变量（[L1190-L1199](../../../raw/current/scripts/prefabs/wortox.lua#L1190-L1199)）。清单外连带：沃拓克斯与排箫另生成 `wortox_soul_spawn_fx`（注册于 [fx.lua:3597](../../../raw/current/scripts/fx.lua#L3597)，wortox.lua:1437、panflute.lua:59 生成），不在本页范围。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `wortox_soulecho_buff_fx` | 无自删：debuff Detach 时 `inst.bufffx:Remove()`（[L837-L846](../../../raw/current/scripts/prefabs/wortox.lua#L837-L846)） | debuff 实体 `keepondespawn=true`，玩家下线重连后由重新附建立重建跟随。 |
| `wortox_overloading_fx` | 无自删：`DestroyOverloadingFX` 移除并取消 10 秒复查任务（[L285-L293](../../../raw/current/scripts/prefabs/wortox.lua#L285-L293)） | 复查落魂后警告层同步消失；宿主被移除时是否随引擎父挂回收未核验。 |
| `wortox_nabbag_body` | `OnUnequip` 显式 Remove（[L148](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L142-L160)）；换装时 `OnEquip` 先删旧再建新；onremove 反登记清宿主引用（[L112-L114](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L112-L114)） | 本体强抢袋用尽 `inst:Remove()`（[L168-L174](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L168-L174)）是否必然经卸装路径触发 OnUnequip 未逐路径核验（见存疑）。 |
| `wortox_nabbag_body_fx`／`_soulfx` | 无独立 Remove，是否随 body 载体回收未核验（引擎子级）；body 的 `OnUpdate_body` 在引用失效时重建 body_fx（[L273-L281](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L269-L321)） | 两层都是本地实体，`bodyfx` 单槽引用在 body 载体上。 |
| `wortox_reviver_body` | `OnStopBody` 显式 Remove（`crushitemcast` 非 crushcasting onexit、`crushitemcast_fail` onexit、传送状态 onexit、自用状态 onenter）；持有者／双尾心移除时是否随引擎父子回收未核验 | 只在施法状态机持有期间存在，`persists=false`；反向监听由 body 的 `onremove` 清双尾心引用，不是双尾心移除触发。 |

## 典型接入

### 给 buff 实体挂"跟随宿主并按位移变速"的粒子层

- 前提与选择：buff/debuff 实体多为 master-only 无网络实体，客户端表现需另挂联网 FX：soulecho 模式是 FX 父挂目标（引擎父挂即可跟随，可不加 Follower），发射器按父级每帧位移在低/高速率间插值。
- 生效与撤销：由 buff 的 AttachedFn 生成、DetachedFn 移除；不要在 FX 工厂里自删——生命周期归 buff 链管理。
- 原版依据：[OnAttached 生成](../../../raw/current/scripts/prefabs/wortox.lua#L821-L835)、[发射器](../../../raw/current/scripts/prefabs/wortox.lua#L974-L999)。

```lua
local fx = SpawnPrefab("wortox_soulecho_buff_fx")
fx.entity:SetParent(target.entity) -- 引擎父挂跟随；buff 结束时 fx:Remove()
```

- 调整与验证：替换点是注册名与包络参数；若需要"停在符号上"而非"跟人走"，参考 overloading_fx 的 `Follower:FollowSymbol(hiteffectsymbol)` 写法（[L307-L310](../../../raw/current/scripts/prefabs/wortox.lua#L304-L311)）。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：两个灵魂粒子工厂共用纹理与相近骨架，但差异不止三项。** `soulecho_buff_fx` 的出生坐标为 `px*2, 0, pz*2`、横向初速 `px*0.02`／`pz*0.02`；`overloading_fx` 为 `px*0.2, py+random, pz*0.2`、横向初速 `px`／`pz`，并带 y 方向初速。尺度包络方向、寿命与 Follower 也不同；改其一不会影响另一个，两份包络名各自独立。[soulecho 发射](../../../raw/current/scripts/prefabs/wortox.lua#L915-L928)、[overloading 发射](../../../raw/current/scripts/prefabs/wortox.lua#L1035-L1047)
- **边界：粒子层在专用服务器上是空壳。** 两工厂的 `IsDedicated` 分支直接返回（[soulecho](../../../raw/current/scripts/prefabs/wortox.lua#L931-L941)、[overloading](../../../raw/current/scripts/prefabs/wortox.lua#L1051-L1065)），专服只有 Transform/Network；同理 nabbag 的 OnUpdate_body 只在非专服挂。
- **误区：强抢袋的动画在袋物品上。** 袋物品只有 `idle_empty` 静态动画与持握符号覆盖；走/打/停的动态机体全在 `wortox_nabbag_body_fx` 本地层，且 nodangle/骑乘时 body 层隐藏、只留绳子符号（[ToggleOverrideSymbols](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L99-L114)）。
- **边界：灵魂档位净变量是 body 载体上的。** `bodyvfx_souls`（0–3）与 `bodysize_netvar`（0–2）写在 body 载体，soulfx 粒子层隔着 body_fx 一层读（`parent.entity:GetParent()`，[L488-L489](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L484-L504)）；跨层改档位要走 `UpdateStats` 而非直接写 netvar。
- **边界：reviver_body 只认 swap_remote 符号。** 跟随偏移 (25, -42) 按该符号设计，换持握符号会错位；它不随双尾心耐久/腐烂状态换动画（工厂恒播 `idle_body`）。
- **边界：overloading 警告只代表"顽皮缓冲"。** 非 naughty 性情不生成警告层、直接掉魂；`_forceoverloading` 复查窗口（10 秒）内警告层持续存在，落魂结算后才移除——警告层存在不等于马上掉魂（[CheckForOverload](../../../raw/current/scripts/prefabs/wortox.lua#L313-L345)）。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/wortox_nabbag.lua`、`prefabs/wortox_reviver.lua` 全文，`prefabs/wortox.lua` 的 soulecho/overloading/panflute 工厂与 buff 链段（soulecho 全函数族、overloading 全函数族、panflute 两个 fn、FinishPortalHop/TryToPortalHop 链），按需读取 `stategraphs/SGwilson.lua` 的 crushitemcast 状态族与 teleport_reviver 状态名、`tuning.lua` 相关调值。对 6 个注册名以 `grep -rn "注册名" --include=*.lua` 在 `raw/current/scripts` 全域反查：`wortox_soulecho_buff_fx` 命中注册（wortox.lua:1602）、依赖（:808）、生成（:831）——**生成点唯一**；`wortox_overloading_fx` 命中注册（:1603）、依赖（:30）、生成（:307）——**生成点唯一**；`wortox_nabbag_body` 命中注册（wortox_nabbag.lua:510）、依赖（:8）、生成（:122）——**生成点唯一**；`wortox_nabbag_body_fx` 命中注册（:511）、依赖（:11）、生成（:274）；`wortox_nabbag_body_soulfx` 命中注册（:512）、依赖（:12）、生成（:279）——各**唯一**；`wortox_reviver_body` 命中注册（wortox_reviver.lua:295）、依赖（:12）、生成（:187）——**生成点唯一**。`wortox_panflute_buff_fx` 全域无注册与生成（死代码）。
- 名称核验：`languages/chinese_s.po` 定向检索——`WORTOX_NABBAG`＝"强抢袋"、`WORTOX_REVIVER`＝"双尾心"（锚点见"快速判断"）；6 个 FX 注册名均无 `STRINGS.NAMES.*` 键，标题使用效果描述。
- 分档说明：`size_class: standard`——6 注册三条链（buff 链×2、装备三层、施法跟随），nabbag 三层用表格压缩；未选 complex 因各链可独立小节承载。
- 未覆盖：`wortox_soulecho_buff`／`wortox_panflute_buff`／`wortox_forget_debuff`／`wortox_decoy` 等同文件其余注册的完整机制、强抢袋捕网（NET 动作）与 nabbag 组件玩法、双尾心传送 SpellFn 全流程、`wortox_soul_spawn_fx`（fx.lua 注册，另页主题）、技能树门禁细节、动画包内容。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。
- 外部检视 R17（2026-11-10）：`wortox_reviver_body` 的触发改为普通 `CASTSPELL` 选 `crushitemcast`（第 9 帧挂 body、第 18 帧只播音），`crushitemcast_holding`／`_trigger` 标为 `STARTVAULTORBTELEPORT` 链；`OnStopBody` 改列 `crushitemcast` 非 crushcasting onexit、`crushitemcast_fail` onexit、传送 onexit、自用 onenter；反向监听方向改为「body 的 `onremove` 清道具引用」，引擎父子回收继续单列待核验。

## 相关链接

- [[reference/prefabs/wortox_eat_soul_fx|进食灵魂特效]]（沃拓克斯灵魂主题的另一表现链）
- [[reference/prefabs/wortox_soul_heal_fx|灵魂治疗特效]]（`wortox_panflute_buff_fx_fn` 死代码借用的同款动画库）
- [[reference/prefabs/wortox_portal_fx|灵魂跳跃出入场特效]]（soulecho 免费跳所属的传送链）
- 返回 [[reference/prefabs/fx-character-index|角色技能、施法与读书]]（角色技能/装备）
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 4 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
