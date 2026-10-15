---
title: "武神之魂与战歌特效（wathgrithr_spirit）"
created: "2026-11-01"
updated: "2026-11-05"
type: prefab
summary: "通用表 14 注册的武神战歌表现族：击杀升灵、常驻战歌 attach/detach 与目标侧谱号循环、歌手侧七首歌曲音符、瞬时歌引述；全部一次性动画自删，战歌玩法数值不在本族。"
aliases: ["wathgrithr_spirit", "battlesong_attach", "battlesong_loop", "battlesong_detach", "battlesong_durability_fx", "battlesong_healthgain_fx", "battlesong_sanitygain_fx", "battlesong_sanityaura_fx", "battlesong_fireresistance_fx", "battlesong_shadowaligned_fx", "battlesong_lunaraligned_fx", "battlesong_instant_electric_fx", "battlesong_instant_taunt_fx", "battlesong_instant_panic_fx", "武神之魂", "武神英灵", "战歌特效", "战歌buff特效", "战歌音符", "瞬时战歌特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/battlesongdefs.lua", "prefabs/battlesongs.lua", "components/singinginspiration.lua", "entityscript.lua", "prefabs/wathgrithr.lua", "prefabs/spawnprotectionbuff.lua", "prefabs/abigail.lua", "stategraphs/SGabigail.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 武神之魂与战歌特效（wathgrithr_spirit）

## 快速判断

- 职责与适用：`fx.lua` 通用表中武神（Wathgrithr）战歌体系的 14 个纯表现 FX：击杀升灵 1 名、常驻战歌 attach/loop/detach 3 名、按歌曲命名的音符 7 名、瞬时歌引述 3 名。全部无玩法数值；增减益本体在歌曲定义与 debuff，分工见下文导航。公共工厂协议（网络代理／本地表现、一帧延迟、animover 自删）只在 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]] 维护，本页不重复。
- 优先使用：查"击杀后升起的武神之魂／战歌 buff 头顶音符／歌起歌落的进场退场动画／瞬时歌引述"读本页；查战歌数值、灵感槽位与唱歌流程选 [[reference/components/singinginspiration|singinginspiration]]；查战歌纸物品本体另找 battlesong 条目页（Wiki 待建）。
- 重要副作用：仅 `wathgrithr_spirit` 有音效条目（valhalla，延迟 0.2 秒）；其余 13 条无声、无 fn、无染色，走默认四向。[条目](../../../raw/current/scripts/fx.lua#L973-L1060)
- 修改边界：生成点全在服务端，本地表现由工厂代理在非专服客户端一帧后创建；每条都是一次性动画（animover 自删、代理 1 秒自删），"循环"全靠调用方周期重播。[代理回收](../../../raw/current/scripts/prefabs/fx.lua#L156-L157)、[本地回收](../../../raw/current/scripts/prefabs/fx.lua#L98-L102)

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wathgrithr_spirit` | 通用表单条 → 加载器代理＋一帧后本地实体 | bank/build/anim 同名；`sound` valhalla、`sounddelay` 0.2。[条目](../../../raw/current/scripts/fx.lua#L973-L979) |
| `battlesong_attach` | 同上协议 | bank/build=`fx_wathgrithr_buff`，anim `attach`，歌起进场。[条目](../../../raw/current/scripts/fx.lua#L983-L987) |
| `battlesong_loop` | 同上协议 | anim `fx_trebleclef`（高音谱号）；名带 loop 仍是一次性动画。[条目](../../../raw/current/scripts/fx.lua#L989-L993) |
| `battlesong_detach` | 同上协议 | anim `detach`，歌落退场。[条目](../../../raw/current/scripts/fx.lua#L995-L999) |
| `battlesong_durability_fx` | 同上协议 | anim `fx_durability`；作为耐久歌 LOOP_FX 字段。[条目](../../../raw/current/scripts/fx.lua#L1002-L1006) |
| `battlesong_healthgain_fx` | 同上协议 | anim `fx_healthgain`。[条目](../../../raw/current/scripts/fx.lua#L1008-L1012) |
| `battlesong_sanitygain_fx` | 同上协议 | anim `fx_sanitygain`。[条目](../../../raw/current/scripts/fx.lua#L1014-L1018) |
| `battlesong_sanityaura_fx` | 同上协议 | anim `fx_sanityaura`。[条目](../../../raw/current/scripts/fx.lua#L1020-L1024) |
| `battlesong_fireresistance_fx` | 同上协议 | anim `fx_fireresistance`。[条目](../../../raw/current/scripts/fx.lua#L1026-L1030) |
| `battlesong_shadowaligned_fx` | 同上协议 | anim `fx_shadowaligned`；shadow 歌需 `wathgrithr_allegiance_shadow` 技能。[定义](../../../raw/current/scripts/prefabs/battlesongdefs.lua#L191-L218) |
| `battlesong_lunaraligned_fx` | 同上协议 | anim `fx_lunaraligned`；lunar 歌需 `wathgrithr_allegiance_lunar` 技能。[定义](../../../raw/current/scripts/prefabs/battlesongdefs.lua#L162-L189) |
| `battlesong_instant_electric_fx` | 同上协议 | anim `quote_revive`（引述样式，名字不含 anim 名）；快照内无运行时生成点，见陷阱。[条目](../../../raw/current/scripts/fx.lua#L1044-L1048) |
| `battlesong_instant_taunt_fx` | 同上协议 | anim `quote_taunt`。[条目](../../../raw/current/scripts/fx.lua#L1050-L1054) |
| `battlesong_instant_panic_fx` | 同上协议 | anim `quote_panic`；被出生保护与阿比盖尔惊吓复用。[条目](../../../raw/current/scripts/fx.lua#L1056-L1060) |

定向检索 `languages/chinese_s.po`：这 14 个注册名没有 `STRINGS.NAMES` 键（战歌名键 `BATTLESONG_DURABILITY` 等属于歌纸物品 prefab，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L339129-L339172)），标题与正文中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | `prefabs/fx.lua` `require("fx")` 后逐项 `MakeFx`；14 条同批注册 | 全族同协议，默认四向 | [[reference/mechanisms/generic-fx-factory]]；[装载](../../../raw/current/scripts/prefabs/fx.lua#L165-L172) |
| 生成者：击杀升灵 | 武神 master 端监听 `killed` → 有效受害者、`not health.nofadeout` 且（epic 或 10% 随机）→ `destroytime`（默认 2 秒）后 `spawnspirit` 落点生成，scale 按体型 0.5／0.7／1.1 | 服务端；火化受害者跳过；nofadeout 是 `onkilled` 内额外门禁，不在 `IsValidVictim` 里 | [spawnspirit](../../../raw/current/scripts/prefabs/wathgrithr.lua#L35-L39)、[onkilled](../../../raw/current/scripts/prefabs/wathgrithr.lua#L51-L66)、[监听](../../../raw/current/scripts/prefabs/wathgrithr.lua#L202) |
| 生成者：常驻战歌 attach | 战歌 buff（`battlesong_*_buff`，debuff 实体）附着后 `DoTaskInTime(math.random())` → `SpawnFx(ATTACH_FX, target, 0.7)`：生成 `battlesong_attach` 父挂受歌目标 | 服务端 buff 生命周期；scale 0.7，骑乘抬 2.3 | [附着](../../../raw/current/scripts/prefabs/battlesongs.lua#L233-L243)、[SpawnFx](../../../raw/current/scripts/prefabs/battlesongs.lua#L168-L191) |
| 生成者：目标侧谱号循环 | `CheckForLoopFx` 把 `battlesong_fx_task` 写在受歌目标的字段上，但任务实体由创建它的战歌 buff `inst:DoPeriodicTask` 持有（周期 4＋随机秒）；每拍生成固定名 `battlesong_loop`（scale 0.4，y 1.6＋随机）；目标开唱即取消任务 | 服务端；仅当歌有 LOOP_FX 字段且目标非演唱中歌手；创建者 buff 先移除会连任务一起取消，而目标字段可能仍非 nil，其余歌存续期间 `CheckForLoopFx` 因此不再重挂 | [循环](../../../raw/current/scripts/prefabs/battlesongs.lua#L198-L214)、[周期 1s 驱动](../../../raw/current/scripts/prefabs/battlesongs.lua#L216-L231)、[移除取消任务](../../../raw/current/scripts/entityscript.lua#L1730-L1746) |
| 生成者：detach | buff 脱离且 `battlesong_count` 归零才 `SpawnFx(DETACH_FX, target, 1)` 播一次退场，并取消目标侧循环任务 | 服务端；多层歌共享计数，逐层剥落不播 | [脱离](../../../raw/current/scripts/prefabs/battlesongs.lua#L272-L301) |
| 生成者：歌手侧音符 | `SingingInspiration:DisplayFx` 轮询 `active_songs`，逐首 `SpawnPrefab(songdata.LOOP_FX)` 父挂歌手（scale 0.4，y 1.2＋随机），0.4–0.9 秒随机轮转、整轮后歇 4 秒；插歌 4 秒后启动，清槽即取消 | 服务端；7 个 `battlesong_*_fx` 只出现在演唱中的武神身上 | [DisplayFx](../../../raw/current/scripts/components/singinginspiration.lua#L193-L239)、[启停](../../../raw/current/scripts/components/singinginspiration.lua#L270-L272)、[清槽](../../../raw/current/scripts/components/singinginspiration.lua#L294-L296) |
| 生成者：瞬时歌 | `InstantInspire` 执行 ONINSTANT：taunt 对非鸟敌人 `SetTarget` ＋ `AddEnemyDebuffFx`；panic 对可恐慌敌人 `Panic(6s)` ＋ 落点生成对应引述 FX（0–0.25 秒随机延迟，位置取生成时） | 服务端；落点生成不父挂，不随目标移动 | [taunt/panic](../../../raw/current/scripts/prefabs/battlesongdefs.lua#L224-L254)、[AddEnemyDebuffFx](../../../raw/current/scripts/prefabs/battlesongdefs.lua#L13-L23)、[目标过滤](../../../raw/current/scripts/components/singinginspiration.lua#L355-L383) |
| 复用：出生保护 | `spawnprotectionbuff` 附着即父挂 `battlesong_instant_panic_fx`（先 `SetNoFaced`） | 服务端 buff；与战歌无关的复用 | [SpawnFx](../../../raw/current/scripts/prefabs/spawnprotectionbuff.lua#L11-L17)、[调用](../../../raw/current/scripts/prefabs/spawnprotectionbuff.lua#L52-L52) |
| 复用：阿比盖尔惊吓 | 升级状态第 15 帧推 `do_ghost_scare` → 半径 10 内可恐慌目标逐个延迟 `DelayedGhostScare` → `apply_panic_fx` 落点生成 panic 引述 FX | 服务端 | [监听](../../../raw/current/scripts/prefabs/abigail.lua#L934-L934)、[链路](../../../raw/current/scripts/prefabs/abigail.lua#L486-L521)、[SG 推事件](../../../raw/current/scripts/stategraphs/SGabigail.lua#L830-L830) |
| 组件：debuff | 常驻歌载体 `battlesong_*_buff` 是服务端隐形实体，ONAPPLY/ONDETACH 把耐久倍率、回血回 san、火伤倍率、对月/暗抗性与增伤挂在目标组件上 | 服务端；`entity:Hide()` 无 Network | [[reference/components/debuff|debuff]]；[载体](../../../raw/current/scripts/prefabs/battlesongs.lua#L303-L330)、[歌曲定义](../../../raw/current/scripts/prefabs/battlesongdefs.lua#L46-L218) |
| 组件：singinginspiration／singable | 唱歌入口与灵感槽位：Inspire 周期给友军 `AddDebuff`，瞬时歌立即结算 | 服务端；仅武神安装 | [[reference/components/singinginspiration|关键接口]]、[[reference/components/singable|singable]]；[Inspire](../../../raw/current/scripts/components/singinginspiration.lua#L385-L391) |
| 数据：player_classified | buff 计数经 `hasinspirationbuff` 同步 HUD 图标，非本族 FX | 服务端写、客户端读 | [置位](../../../raw/current/scripts/prefabs/battlesongs.lua#L252-L256)、[清位](../../../raw/current/scripts/prefabs/battlesongs.lua#L280-L283) |
| 资源 | bank/build `fx_wathgrithr_buff`（13 条战歌条目）与 `wathgrithr_spirit`（1 条）；音效事件 `dontstarve_DLC001/characters/wathgrithr/valhalla` | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L973-L1060)、[音效播放](../../../raw/current/scripts/prefabs/fx.lua#L43-L50) |

## 典型接入

### 借用战歌表现给自定义实体

- 前提与选择：服务端脚本（组件回调／SG timeline 等）仿照 `battlesongs.lua` 的 `SpawnFx`：生成后 `SetNoFaced`、`AddChild` 父挂、按需设 scale；落点展示则学 `AddEnemyDebuffFx` 直接 `SetPosition`。
- 生效与撤销：一次生成播一遍动画后自删，无需手动清理；需要持续表现时自行周期重播（注意别堆叠生成）。
- 原版依据：[SpawnFx](../../../raw/current/scripts/prefabs/battlesongs.lua#L168-L191)、[工厂协议](../../../raw/current/scripts/prefabs/fx.lua#L121-L159)。

```lua
-- 服务器端：给任意实体挂一个战歌起手表现（仿 prefabs/battlesongs.lua SpawnFx）
local function PlayBattlesongAttachFx(target)
    local fx = SpawnPrefab("battlesong_attach")
    if fx ~= nil then
        fx.Transform:SetNoFaced()
        target:AddChild(fx)
        fx.Transform:SetScale(0.7, 0.7, 0.7)
    end
    return fx
end
```

- 调整与验证：改注册名即可换 attach/loop/detach/引述变体；scale 是三轴同比值（原版 0.4–1.0）；表现是否随宿主移动取决于是否 `AddChild`。自建周期重播要自带取消时机：目标侧 `target.battlesong_fx_task` 的句柄记在目标、任务实体归创建它的战歌 buff，buff 先移除会连任务一起取消；歌手侧 `display_fx_task` 的句柄记在 `SingingInspiration` 组件、任务归歌手实体（由 `self.inst:DoTaskInTime` 创建），不随战歌 buff 移除取消。

## 常见陷阱与限制

- **误区：`battlesong_loop` 是循环常驻特效**：名带 loop，但它与全族一样是 animover 自删的一次性动画；"循环"由受歌目标上的周期任务（4＋随机秒）反复重新生成实现。[循环重播](../../../raw/current/scripts/prefabs/battlesongs.lua#L198-L214)、[本地回收](../../../raw/current/scripts/prefabs/fx.lua#L98-L102)
- **边界：七首歌曲音符不出现在受歌目标身上**：`battlesong_durability_fx` 等 7 个 `LOOP_FX` 只被歌手侧 `DisplayFx` 生成，父挂演唱中的武神；受歌目标头上只有通用谱号 `battlesong_loop`。LOOP_FX 字段在目标侧仅当开关用。[DisplayFx](../../../raw/current/scripts/components/singinginspiration.lua#L206-L228)、[CheckForLoopFx](../../../raw/current/scripts/prefabs/battlesongs.lua#L201-L210)
- **边界：FX 名不是疗效保证**：`battlesong_fireresistance_fx` 的存在不代表火免生效——实际加减成在 `battlesongdefs.lua` 各歌 ONAPPLY/ONDETACH（如 `health.externalfiredamagemultipliers`、`weapon.attackwearmultipliers`、`damagetyperesist`/`damagetypebonus`），FX 只伴生显示。[耐久歌](../../../raw/current/scripts/prefabs/battlesongdefs.lua#L48-L80)、[火抗歌](../../../raw/current/scripts/prefabs/battlesongdefs.lua#L142-L160)
- **误区：`battlesong_instant_electric_fx` 会在复活歌里播放**：它只注册、被 revive 歌的 `ATTACH_FX` 字段引用并声明进 prefabs 依赖；但 INSTANT 歌不建 buff 实体（`AddSong` 跳过注册 buff prefab），唯一 ATTACH_FX 生成路径走 buff 附着，故覆盖不到它。检索范围：全域 `rg` 14 个注册名与 `ATTACH_FX`/`LOOP_FX`/`DETACH_FX`/`ONINSTANT` 消费点，命中仅 fx.lua、battlesongdefs.lua、battlesongs.lua、wathgrithr.lua、spawnprotectionbuff.lua、abigail.lua；未发现其他生成点，不等于引擎内无隐藏路径。[字段声明](../../../raw/current/scripts/prefabs/battlesongdefs.lua#L281-L281)、[INSTANT 跳过](../../../raw/current/scripts/prefabs/battlesongs.lua#L339-L347)
- **边界：特效与 buff 生命周期不同步**：attach FX 有 0–1 秒随机延迟才生成；detach 只在最后一层歌移除时播一次；已生成的音符会各自播完自删，不随 buff 提前消失。[附着延迟](../../../raw/current/scripts/prefabs/battlesongs.lua#L241-L243)、[计数门](../../../raw/current/scripts/prefabs/battlesongs.lua#L278-L294)
- **边界：瞬时歌引述落点不跟随目标**：taunt/panic FX 在生成时刻固定于敌人当时位置，不父挂；嘲讽把仇恨拉向歌手但表现留在原地。[落点](../../../raw/current/scripts/prefabs/battlesongdefs.lua#L13-L23)
- **边界：`SpawnFx` 判空顺序**：`Transform:SetNoFaced()` 在 `if fx then` 之前执行（L169-170），若 prefab 未注册会在服务端直接报错；复用这段代码前先确认注册名存在。[顺序](../../../raw/current/scripts/prefabs/battlesongs.lua#L168-L172)

## 验证记录

- 源码审阅：2026-10-31，Steam buildid 26335394。逐条核验 fx.lua 14 个表项（bank/build/anim/sound/fn 有无）；`prefabs/fx.lua` 装载与代理／本地双实体回收；`battlesongdefs.lua` 全文 10 首歌的 ATTACH_FX/LOOP_FX/DETACH_FX/ONINSTANT 字段；`battlesongs.lua` SpawnFx、attach/loop/detach 三段生命周期与 INSTANT 跳过；`singinginspiration.lua` DisplayFx 启停与 InstantInspire 目标过滤；wathgrithr onkilled 升灵、spawnprotectionbuff 与 abigail 惊吓两处复用；tuning 相关常量（SONG_REFRESH_PERIOD=1、PANIC_TIME=6 等）。全域反查 `rg` 14 注册名：命中上述 6 文件，无其他生成点。未覆盖：`fx_wathgrithr_buff`／`wathgrithr_spirit` 动画资源内容与时长、歌纸物品 prefab 家族、客户端音符 HUD widget。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（--strict）；Lua 示例经 parser_probe --syntax-only 语法检查。
- 游戏运行：未运行；升灵触发概率观感、音符轮播节奏与瞬时歌引述表现未实测。

相关：[[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]、[[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/components/singinginspiration|战歌灵感]]。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/prefabs/index|Prefab 索引]]。
