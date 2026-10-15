---
title: "鱼人水花（merm_splash 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "鱼人系统的落点式水花：王诞生跨世界增益、月变异变时鱼人自身位置 merm_splash，鱼人王诞生 merm_king_splash，召唤与召回守卫出退场 merm_spawn_fx；音效由生成方播放，条目无 sound。"
aliases: ["merm_splash", "merm_king_splash", "merm_spawn_fx", "鱼人水花", "鱼人王水花", "鱼人生成水花", "鱼人变异水花"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "stategraphs/SGmerm.lua", "stategraphs/SGmermking.lua", "prefabs/merm.lua", "prefabs/mermking.lua", "brains/mermguardbrain.lua", "components/mermkingmanager.lua", "components/shard_mermkingwatcher.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 鱼人水花（merm_splash 族）

## 快速判断

- 职责与适用：`fx.lua` 通用表的三注册水花，覆盖鱼人系统的出现／增益／变异／退场表现：`merm_splash`（鱼人本体水花，三处触发）、`merm_king_splash`（鱼人王诞生）、`merm_spawn_fx`（守卫召唤与回巢退场）。纯表现，全部落点式（生成在实体自身位置，不父挂）。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L2055-L2073)
- 优先使用：查"鱼人出现／变身／退场水花"选本页；鱼人王产生与守卫机制的完整流程另读鱼人相关机制页（Wiki 待建，源码入口见下表）。
- 重要副作用：无玩法作用；三名 fn 均为 `FinalOffset1`（`SetFinalOffset(1)`），条目均无 sound 字段——水花声由生成方播放。[helper](../../../raw/current/scripts/fx.lua#L1-L3)
- 修改边界：服务端生成（SG 状态与 Brain 节点内触发）；六种鱼人 prefab 共用 SGmerm，事件驱动的水花随事件全体播放；专用服务器无本地表现。[条目](../../../raw/current/scripts/fx.lua#L2055-L2073)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `merm_splash` | 通用表条目 → 加载器代理＋一帧后本地实体，fn=FinalOffset1 | bank/build/anim=`merm_splash`，无声。[条目](../../../raw/current/scripts/fx.lua#L2062-L2066) |
| `merm_king_splash` | 同上 | bank/build/anim=`merm_king_splash`，无声。[条目](../../../raw/current/scripts/fx.lua#L2055-L2059) |
| `merm_spawn_fx` | 同上 | bank/build=`merm_spawn_fx`、anim=`splash`（动画名与注册名不同），无声。[条目](../../../raw/current/scripts/fx.lua#L2069-L2073) |

三个注册名在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

六处生成点均为服务端、落点式 `SetPosition` 于实体自身位置；"伴随音效"一列只列生成方在 FX 生成同刻播放的 SoundEmitter 调用（同一触发内还有其他时点音效未逐一列出），与 FX 条目无关。

| 生成点 | 触发链与条件 | 伴随音效 | 证据 |
|---|---|---|---|
| `merm_splash`（王诞生增益） | SGmerm `buff` 态 onenter——`onmermkingcreated_anywhere` 世界事件（shard_mermkingwatcher 在任意世界首个王出现时推送，含跨世界同步）→ 六种鱼人（merm／mermguard 及 shadow／lunar 变体共用 SGmerm）各自播放 | `dontstarve/characters/wurt/merm/buff` | [事件处理](../../../raw/current/scripts/stategraphs/SGmerm.lua#L160-L162)、[buff 态](../../../raw/current/scripts/stategraphs/SGmerm.lua#L431-L458)、[推送](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L17-L24) |
| `merm_splash`（月变） | SGmerm `lunar_transform` 态 onenter——`DoLunarMutation` 生成 `merm_lunar`／`mermguard_lunar` 后向新体推 `mutated` | 同上 buff 声＋`meta4/lunar_merm/transform` | [态](../../../raw/current/scripts/stategraphs/SGmerm.lua#L917-L953)、[推送](../../../raw/current/scripts/prefabs/merm.lua#L798-L833) |
| `merm_splash`（月返） | SGmerm `lunar_revert` 态 onenter——`DoLunarRevert` 变回 `merm`／`mermguard` 后向新体推 `demutated` | 同上 | [态](../../../raw/current/scripts/stategraphs/SGmerm.lua#L955-L990)、[推送](../../../raw/current/scripts/prefabs/merm.lua#L835-L864) |
| `merm_king_splash`（王诞生） | SGmermking `oncreated` 态 onenter——mermkingmanager `CreateMermKing` 把候选鱼人 `ReplacePrefab` 成 mermking 后向新王推 `oncreated`（宿主为 forest／cave 世界的 mermkingmanager 组件） | `dontstarve/characters/wurt/merm/throne/spawn` | [态](../../../raw/current/scripts/stategraphs/SGmermking.lua#L86-L102)、[CreateMermKing](../../../raw/current/scripts/components/mermkingmanager.lua#L123-L129) |
| `merm_spawn_fx`（召唤守卫） | 王的 `HealthDelta`：血量跌破 75% 且下降、有目标、尚有 guards_available 且无在场守卫（另需未在召唤中、未死亡）→ 推 `call_guards` → SGmermking `call_guards` 态 30 帧处 `CallGuards`：召唤至多 guards_available 名 `mermguard`，每名落点一枚 | `dontstarve/characters/wurt/merm/throne/spawn` | [HealthDelta](../../../raw/current/scripts/prefabs/mermking.lua#L324-L338)、[call_guards 态](../../../raw/current/scripts/stategraphs/SGmermking.lua#L126-L148)、[CallGuards](../../../raw/current/scripts/prefabs/mermking.lua#L406-L438) |
| `merm_spawn_fx`（守卫回巢退场） | mermguardbrain `ShouldGoToThrone`：`return_to_king` 为真（`ReturnMerms` 在王失去目标时置真，读档也恢复）的守卫走到王座旁 ActionNode：落点一枚后守卫 `Remove` | `dontstarve/characters/wurt/merm/buff` | [Brain 节点](../../../raw/current/scripts/brains/mermguardbrain.lua#L366-L388)、[ReturnMerms](../../../raw/current/scripts/prefabs/mermking.lua#L440-L449) |

补充关系：

- SG 宿主：merm.lua 以 `MakeMerm` 批量返回六种鱼人 prefab，全部经 common／guard master 挂 `SetStateGraph("SGmerm")`。[宿主装配](../../../raw/current/scripts/prefabs/merm.lua#L1773-L1778)、[guard](../../../raw/current/scripts/prefabs/merm.lua#L1303-L1304)、[common](../../../raw/current/scripts/prefabs/merm.lua#L1467-L1468)
- 依赖声明：merm.lua 六 prefab 共表声明 `merm_splash`／`merm_spawn_fx`（[表](../../../raw/current/scripts/prefabs/merm.lua#L43-L45)）；mermking.lua 声明 `merm_king_splash`（[表](../../../raw/current/scripts/prefabs/mermking.lua#L10-L14)）。仅声明，生成点如上表。
- 资源：三名各用同 build 动画族（`merm_splash`／`merm_king_splash`／`merm_spawn_fx`）；`oceanwhirlportal_splash` 复用 `merm_king_splash` 的 bank/build/anim 但为独立注册、与鱼人无关，见 [[reference/prefabs/splash_sink|沉没与单点水花页]]。[辨析](../../../raw/current/scripts/fx.lua#L2030-L2038)

## 典型接入

### 实体退场水花

- 前提与选择：需要"实体消失前原地水花"的表现时，照原版守卫回巢的写法：先落点 FX、播提示声、再移除本体；FX 与移除在同一服务端逻辑内完成。
- 生效与撤销：FX 落点式自删（animover），本体 `Remove` 后无残留；位置取移除前的实体位置。
- 原版依据：[Brain 节点](../../../raw/current/scripts/brains/mermguardbrain.lua#L378-L387)。

```lua
local fx = SpawnPrefab("merm_spawn_fx")
fx.Transform:SetPosition(inst.Transform:GetWorldPosition())
inst.SoundEmitter:PlaySound("dontstarve/characters/wurt/merm/buff")
inst:Remove()
```

- 调整与验证：`inst` 为服务端实体；如把退场改成延时移除，水花位置取的仍是调用时刻的位置。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：条目不出声**：三名均无 `sound` 字段，所有水花声（buff／transform／throne spawn）由生成方在触发处播放；只 `SpawnPrefab` 不发声。[条目](../../../raw/current/scripts/fx.lua#L2055-L2073)
- **边界：王诞生水花是全体广播**：`onmermkingcreated_anywhere` 由 shard_mermkingwatcher 跨世界同步，任何分片出现首个王都会让本世界所有挂 SGmerm 的鱼人各落一枚水花；拦截表现要按多模拟语义处理。[推送](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L17-L33)
- **边界：`merm_splash` 不只属于普通鱼人**：月变／月返路径在新体（`merm_lunar`／`mermguard_lunar` 或变回的 `merm`／`mermguard`）位置播放，且这些体共用 SGmerm。[变异](../../../raw/current/scripts/prefabs/merm.lua#L828-L830)
- **边界：同名资源不同注册**：`oceanwhirlportal_splash` 用 `merm_king_splash` 的 bank/build/anim 做漩涡传送门表现，按注册名检索时勿混。[辨析](../../../raw/current/scripts/fx.lua#L2030-L2038)
- **边界：`merm_spawn_fx` 的 anim 是 `splash`**：按注册名找动画名会落空。[条目](../../../raw/current/scripts/fx.lua#L2069-L2073)
- **边界：动画是否重复未核验**：工厂只 `PlayAnimation` 一次，是否循环由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-11-01，steam-buildid:26335394。三注册配置与 `FinalOffset1` helper、六处生成点（SGmerm 三态、SGmermking 两态、mermguardbrain 节点）的完整触发链、六种鱼人 SG 宿主装配、依赖声明、`.po` 名称键核验完毕；对 `merm_king_splash|merm_splash|merm_spawn_fx` 在 scripts 全域 `rg -n`（根级、prefabs、components、stategraphs、brains，含拼名模式）仅命中上表位置与 fx.lua 内 `oceanwhirlportal_splash` 的资源字段（L2031-L2033，非生成点），无动态拼名。未覆盖：动画资源内容与循环性、晚加入时序。
- 游戏运行：未运行；水花视觉与跨世界广播未实测。
- 结构检查：`validate_page.py` 通过（0 issue；86 行、10168 字节，低于 200 行／16 KiB 标准档）；`validate_source_links.py --strict` 通过（errors 0、warnings 0；33 处源码引用，初版 9 处锚点起于 fx.lua 条目 `{` 行，已按 name 行起点约定修正清零）。正文 Lua 示例另存临时文件经 `parser_probe --syntax-only` 通过（只查 Lua 5.1 语法，未执行）。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/prefabs/index|Prefab 索引]]。
