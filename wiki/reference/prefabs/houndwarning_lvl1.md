---
title: "威胁预警音效与暗影窥视表现（houndwarning_lvl1 等 FX 族）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "11 个工厂、29 个注册的纯表现 FX 族：威胁预警声的联网分发与本地随机方位、陨石与坠落物落点阴影、三类低理智暗影表现及声音调试图标。"
aliases: ["houndwarning_lvl1", "houndwarning_lvl2", "houndwarning_lvl3", "houndwarning_lvl4", "wormwarning_lvl1", "wormwarning_lvl2", "wormwarning_lvl3", "wormwarning_lvl4", "wormwarning_worm_boss", "deerclopswarning_lvl1", "deerclopswarning_lvl2", "deerclopswarning_lvl3", "deerclopswarning_lvl4", "beargerwarning_lvl1", "beargerwarning_lvl2", "beargerwarning_lvl3", "beargerwarning_lvl4", "krampuswarning_lvl1", "krampuswarning_lvl2", "krampuswarning_lvl3", "acidbatwavewarning_lvl1", "piratewarningsound", "yothknightwarningsound", "meteorwarning", "warningshadow", "creepyeyes", "shadowwatcher", "shadowskittish", "sounddebugicon", "猎犬预警音效", "怪物预警音效", "落点阴影", "暗影窥视"]
tags: [prefab, fx]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/houndwarning.lua", "prefabs/wormwarning.lua", "prefabs/monsterwarningsounds.lua", "prefabs/piratewarningsound.lua", "prefabs/yoth_knightwarningsound.lua", "prefabs/meteorwarning.lua", "prefabs/warningshadow.lua", "prefabs/shadowskittish.lua", "prefabs/shadowwatcher.lua", "prefabs/creepyeyes.lua", "prefabs/sounddebugicon.lua", "components/hounded.lua", "components/deerclopsspawner.lua", "components/beargerspawner.lua", "components/kramped.lua", "components/acidbatwavemanager.lua", "components/piratespawner.lua", "components/yoth_knightmanager.lua", "components/hallucinations.lua", "components/quaker.lua", "components/meteorshower.lua", "components/lunarhailmanager.lua", "components/area_trigger.lua", "prefabs/player_classified.lua", "prefabs/cave.lua", "prefabs/forest.lua", "prefabs/shadowmeteor.lua", "prefabs/moon_device.lua", "prefabs/ruins_cavein_obstacle.lua", "stategraphs/SGgrassgator.lua", "constants.lua", "tuning.lua", "easing.lua", "debugsounds.lua", "consolecommands.lua", "main.lua", "prefabs/world.lua", "prefablist.lua", "gamelogic.lua", "languages/chinese_s.po", "tuning_override.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 威胁预警音效与暗影窥视表现（houndwarning_lvl1 等 FX 族）

## 快速判断

- 本页是 11 个工厂、29 个注册的纯表现 FX 族入口：威胁预警声（猎犬、蠕虫、巨鹿、熊獾、坎普斯、酸蝠、海盗、Yoth 骑士）、陨石与坠落物落点阴影（`meteorwarning`、`warningshadow`）、三类低理智暗影（`creepyeyes`、`shadowwatcher`、`shadowskittish`）以及声音调试图标（`sounddebugicon`）。这些注册没有伤害、掉落、AI 或可交互玩法，播放或显示后由工厂或宿主清理。[猎犬工厂](../../../raw/current/scripts/prefabs/houndwarning.lua#L11-L40)、[怪物预警工厂](../../../raw/current/scripts/prefabs/monsterwarningsounds.lua#L121-L153)
- 三种构造：非联网的即时声源（猎犬／蠕虫 4 档，直接父挂 `TheFocalPoint.entity` 后自删）；联网代理＋各客户端本地声源（`*warning_lvlN`、`piratewarningsound`、`yothknightwarningsound`，代理只同步随机值或位置）；联网或非联网的动画实体（`meteorwarning`、`warningshadow` 与三类暗影）。[本地声源](../../../raw/current/scripts/prefabs/houndwarning.lua#L11-L20)、[联网代理](../../../raw/current/scripts/prefabs/monsterwarningsounds.lua#L121-L153)
- 优先使用：给 Boss／袭击“将至”加方向感时，参考 `monsterwarningsounds` 的“代理同步 `net_byte` → 客户端在本地焦点周围随机方位建声源”模式；只需落点阴影时用 `warningshadow` 并自行缩放、移除；需要淡变和落点声时用 `meteorwarning`，由 `shadowmeteor` 生成并驱动。[淡变](../../../raw/current/scripts/prefabs/meteorwarning.lua#L47-L54)、[宿主驱动](../../../raw/current/scripts/prefabs/shadowmeteor.lua#L293-L299)
- 重要副作用：预警声源父挂本地焦点并按半径与随机角摆位，动画阴影不自行消失（`warningshadow`、`meteorwarning` 都依赖宿主 `Remove`）；本族不修改世界状态。[阴影清理](../../../raw/current/scripts/prefabs/shadowmeteor.lua#L49-L51)、[宿主清理](../../../raw/current/scripts/components/quaker.lua#L359-L362)
- 修改边界：`houndwarning*`、`wormwarning*` 与三类暗影工厂没有 `AddNetwork`，在服务端生成不会同步到客户端；`sounddebugicon` 只由声音调试开关生成；所有注册名在简中 `languages/chinese_s.po` 的 `STRINGS.NAMES` 键均未命中，标题与别名是核验效果描述而非官方译名。[端分界](../../../raw/current/scripts/prefabs/shadowskittish.lua#L21-L55)、[调试钩子](../../../raw/current/scripts/debugsounds.lua#L26-L40)

动态循环展开后的 29 个注册名：`houndwarning_lvl1`、`houndwarning_lvl2`、`houndwarning_lvl3`、`houndwarning_lvl4`；`wormwarning_lvl1`、`wormwarning_lvl2`、`wormwarning_lvl3`、`wormwarning_lvl4`、`wormwarning_worm_boss`；`deerclopswarning_lvl1`、`deerclopswarning_lvl2`、`deerclopswarning_lvl3`、`deerclopswarning_lvl4`；`beargerwarning_lvl1`、`beargerwarning_lvl2`、`beargerwarning_lvl3`、`beargerwarning_lvl4`；`krampuswarning_lvl1`、`krampuswarning_lvl2`、`krampuswarning_lvl3`；`acidbatwavewarning_lvl1`；`piratewarningsound`、`yothknightwarningsound`、`meteorwarning`、`warningshadow`、`shadowskittish`、`shadowwatcher`、`creepyeyes`、`sounddebugicon`。

## 注册与构造

`prefablist.lua` 的 `PREFABFILES` 在启动时由 `gamelogic.lua` 逐个 `LoadPrefabFile("prefabs/"..file)`，因此上述 11 个工厂文件及其循环注册在启动即存在；world prefab 的 `prefabs` 数组只表达依赖／资产预载，不是注册或生成。[启动加载](../../../raw/current/scripts/gamelogic.lua#L203-L205)、[清单](../../../raw/current/scripts/prefablist.lua#L576-L576)

| 注册族／变体 | 工厂、端与位置 | 差异、随机与清理 | 证据 |
|---|---|---|---|
| 猎犬声 `houndwarning_lvl1`～`lvl4`（循环 4 档） | `makewarning(distance)` 闭包；无 Network、带 `FX`、`persists=false`、`SetCanSleep(false)`。0 秒任务中 `SetParent(TheFocalPoint.entity)`，按半径 60／50／40／30 与 `math.random()*TWOPI` 摆位，播 `dontstarve/creatures/hound/distant` 后立即 `Remove`。 | 等级由 `WARNING_LEVEL_DISTANCE` 下标决定：LVL1 最远（60），LVL4 最近（30，等于 `SPAWN_DIST`）。构造不存档，无持续任务。 | [播放](../../../raw/current/scripts/prefabs/houndwarning.lua#L11-L20)、[工厂与循环](../../../raw/current/scripts/prefabs/houndwarning.lua#L23-L46) |
| 蠕虫声 `wormwarning_lvl1`～`lvl4` | 与猎犬同构；`makewarning(distance)` 另接收 `soundoveride`，默认播 `dontstarve/creatures/worm/distant`。 | 距离仍为 60／50／40／30，等级与猎犬一一对应；洞穴 `wave_pre_upgraded` 时不使用这四档而用 boss。 | [播放](../../../raw/current/scripts/prefabs/wormwarning.lua#L11-L22)、[循环](../../../raw/current/scripts/prefabs/wormwarning.lua#L44-L47) |
| `wormwarning_worm_boss` | 循环外的第 5 个注册：`makewarning(SPAWN_DIST, "rifts4/worm_boss/distant")`。 | 半径固定 30，使用 boss 专属远声；同样带 `FX`、父挂焦点、随机角、播完自删。 | [注册](../../../raw/current/scripts/prefabs/wormwarning.lua#L49-L49) |
| 怪物预警 4 族：`deerclopswarning_lvl1`～`lvl4`、`beargerwarning_lvl1`～`lvl4`、`krampuswarning_lvl1`～`lvl3`、`acidbatwavewarning_lvl1`（共 12 名） | 共用 `makewarning(params, level)`：联网代理加 Transform、Network、`FX`，除代理自身变换外只同步 `net_byte("_rand")`；非专服客户端监听 `randdirty`；主模拟 `_rand:set(math.random(255))` 并 1 秒后移除。客户端在 0 秒任务里建本地非联网声源：父挂焦点、角度 `_rand/255*TWOPI`、半径按距离与超出 range 的缓动计算，播完 `Remove`。 | 参数表：巨鹿／熊獾 range 40、距离 25／20／15／5；坎普斯 range 30、三档距离均 0；酸蝠 range 12（`TUNING.ACIDBATWAVE_SPAWN_DISTANCE`）、唯一一档距离 0。距离 0 时随机角不产生位移。 | [参数表](../../../raw/current/scripts/prefabs/monsterwarningsounds.lua#L6-L84)、[常量](../../../raw/current/scripts/tuning.lua#L7760-L7766)、[本地声源与缓动](../../../raw/current/scripts/prefabs/monsterwarningsounds.lua#L86-L119)、[工厂](../../../raw/current/scripts/prefabs/monsterwarningsounds.lua#L121-L153) |
| `piratewarningsound` | 联网代理，带 `FX`；非专服 0 秒任务取 `ThePlayer`，`range = 40 + 当前平台 walkableplatform.platform_radius`；玩家与声源距离超过 15 时把声源沿两点连线拉回 15；`CreateSoundFxAt` 建非联网声源，在世界坐标播 `monkeyisland/primemate/announce` 后自删。主模拟 1 秒移除代理。 | 只对本地玩家做距离判定；没有船时范围 40。声源不父挂焦点，位置在世界坐标。 | [本地播放](../../../raw/current/scripts/prefabs/piratewarningsound.lua#L14-L35)、[代理与清理](../../../raw/current/scripts/prefabs/piratewarningsound.lua#L37-L63) |
| `yothknightwarningsound` | 同型联网代理，带 `FX`；`RANGE = PLAYER_CAMERA_SEE_DISTANCE + 20`（40＋20＝60，注释写作“+10”与代码不符）；超出 15 同样拉回；声源播 `yoth_2026/fanfare/announce`。 | 与海盗声的差别只在范围、声音与 clamp 距离；代理生命周期同样 1 秒。 | [范围](../../../raw/current/scripts/prefabs/yoth_knightwarningsound.lua#L14-L34)、[代理](../../../raw/current/scripts/prefabs/yoth_knightwarningsound.lua#L36-L62) |
| `meteorwarning` | 联网动画实体：bank `warning_shadow`、build `meteor_shadow`、`idle` 循环、`SetFinalOffset(3)`，带 `FX`／`NOCLICK`。三个网络字段 `_fade`（smallbyte）、`_fadeend`（smallbyte）、`_period`（float），默认 alpha 0.33→1、周期 1/42 秒。主模拟在 0 秒播 `dontstarve/common/meteor_spawn`，`persists=false`，无自删。 | 两端都在构造时启动周期任务；默认参数下各自按相同初值推进。非默认参数时 `startfn` 写网络值并触发 dirty，客户端重建周期任务；原版 `startfn(1,.33,1)` 与默认值相同，不改写。收尾只有宿主的 `Remove`。 | [网络与淡变](../../../raw/current/scripts/prefabs/meteorwarning.lua#L23-L54)、[工厂](../../../raw/current/scripts/prefabs/meteorwarning.lua#L60-L99) |
| `warningshadow` | 联网动画实体：bank／build 均为 `warning_shadow`、`idle` 循环、`SetFinalOffset(3)`、`FX`；无组件、无声、`persists=false`。 | 不再自带缩放、淡变或移除；由宿主设置 `Transform:SetScale` 并在移除时一并 `Remove`。 | [工厂](../../../raw/current/scripts/prefabs/warningshadow.lua#L6-L28) |
| `shadowskittish` | 非联网动画实体：`FX`／`NOCLICK`／`NOBLOCK`、bank `shadowcreatures`、build `shadow_skittish`、`idle_loop`。`playerprox:SetDist(5,8)`，近身回调 `Disappear`；非专服另加 `transparentonsanity`（`ForceUpdate`）与 `updatelooper`（每帧查女王火炬，命中即消失）。 | `Disappear` 取消死亡任务、播 `disappear`，`animover` 时移除；另有 `5+10*math.random()` 秒的死亡任务。 | [消失与任务](../../../raw/current/scripts/prefabs/shadowskittish.lua#L6-L13)、[构造](../../../raw/current/scripts/prefabs/shadowskittish.lua#L39-L52) |
| `shadowwatcher` | 非联网动画实体，带 `FX`／`NOCLICK`：地面朝向、背景层、`sort 3`、alpha 0.5；`LightWatcher` 阈值 0.2／0.19，`enterlight` → 播 `watcher_pst` 后移除。创建时 `Hide()`，`OnInit` 在暗处 `Show()`；玩家为空或 `CanEntitySeeInDark` 时立即移除，否则监听玩家 `nightvision` 并排 `5+10*math.random()` 秒死亡任务。 | 光线进入即消失；夜视开启也移除。动画顺序 `watcher_pre` → `watcher_loop`。 | [光与动画](../../../raw/current/scripts/prefabs/shadowwatcher.lua#L40-L51)、[玩家与清理](../../../raw/current/scripts/prefabs/shadowwatcher.lua#L55-L67) |
| `creepyeyes` | 非联网动画实体，带 `FX`／`NOCLICK`：bank／build `eyes_darkness`、LightOverride 1；`animname = tostring(math.random(3))`，播放 `appear_N` → `idle_N`。`LightWatcher` 阈值同上，`enterlight` 直接移除；`playerprox:SetDist(3,5)` 近身回调 `Disappear`。 | 隐藏后 `OnInit` 逻辑同 `shadowwatcher`；闪烁每 `0.5+math.random()` 秒一次（首次 `1+math.random()`），死亡任务 `10+5*math.random()` 秒。 | [眨眼与消失](../../../raw/current/scripts/prefabs/creepyeyes.lua#L6-L23)、[构造与计时](../../../raw/current/scripts/prefabs/creepyeyes.lua#L63-L76) |
| `sounddebugicon` | 非联网动画＋`Label` 实体：bank `sound`、build `sounddebug`、`idle`、`SetFinalOffset(1)`、`FX`；标签字号 20、颜色 0.73／0.05／0.02。`0.5` 秒自删；`debugsounds` 对循环声会取消该任务并在标签写声音名。 | 仅调试用；正常运行不生成，唯一生成点是 `SOUNDDEBUGUI_ENABLED` 为真时的 `SoundEmitter.PlaySound` 钩子。 | [工厂](../../../raw/current/scripts/prefabs/sounddebugicon.lua#L6-L31)、[生成钩子](../../../raw/current/scripts/debugsounds.lua#L26-L40) |

## 关联机制导航

| 类型／对象 | 本族如何使用 | 变体、端与条件 | 入口 |
|---|---|---|---|
| 事件／同步：`houndwarning` → `HOUNDWARNINGTYPE` | 世界组件 `hounded` 给目标玩家推 `houndwarning` 事件（值为 `LVL1`～`LVL4`／`WORM_BOSS`，洞穴另用 `LVL1_WORM`～`LVL4_WORM`）；`player_classified` 把值写入 `net_smallbyte("frontend.houndwarning")`，客户端 dirty 回调再按值选注册名生成。 | 主模拟推事件、客户端生成；`player_classified` 的映射直接决定九种名字，不在工厂内。 | [事件](../../../raw/current/scripts/components/hounded.lua#L890-L908)、[映射](../../../raw/current/scripts/prefabs/player_classified.lua#L1050-L1072)、[常量](../../../raw/current/scripts/constants.lua#L2462-L2473) |
| 组件：`playerprox` | `shadowskittish` 用 5／8、`creepyeyes` 用 3／5 的近身距离回调 `Disappear`。 | 两类暗影实体本地创建；组件行为见 [[reference/components/playerprox|playerprox]]。 | [skittish](../../../raw/current/scripts/prefabs/shadowskittish.lua#L39-L41)、[eyes](../../../raw/current/scripts/prefabs/creepyeyes.lua#L50-L52) |
| 组件：`transparentonsanity`／`updatelooper` | 非专服端为 `shadowskittish` 添加：前者按理智刷新透明度，后者每帧检查女王火炬并在命中时消失。 | 仅非专服分支；从工厂看不到服务端等价逻辑。 | [挂载](../../../raw/current/scripts/prefabs/shadowskittish.lua#L43-L50)、[[reference/components/transparentonsanity|transparentonsanity]]、[[reference/components/updatelooper|updatelooper]] |
| 引擎接口：`LightWatcher` | `shadowwatcher`、`creepyeyes` 用 `AddLightWatcher`、阈值 0.2／0.19 与 `enterlight` 事件决定显示或消失；两实体先隐藏，待 `OnInit` 再查光照。 | 实体是本地表现，光照判断在客户端；`CanEntitySeeInDark` 为真时整个实体不保留。 | [watcher](../../../raw/current/scripts/prefabs/shadowwatcher.lua#L40-L42)、[eyes](../../../raw/current/scripts/prefabs/creepyeyes.lua#L46-L48) |
| 网络字段：`net_byte`／`net_smallbyte`／`net_float` | `monsterwarningsounds` 代理只同步 `_rand`；`meteorwarning` 同步 `_fade`、`_fadeend`、`_period`；`houndwarning` 走 `player_classified` 的 `houndwarningevent`。 | 随机值 1～255，0 保留为未设置；`net_smallbyte` 上限 63 与 alpha 的 0～63 映射一致。 | [随机值](../../../raw/current/scripts/prefabs/monsterwarningsounds.lua#L108-L119)、[淡变字段](../../../raw/current/scripts/prefabs/meteorwarning.lua#L76-L83) |
| 资源：动画与声音 | 动画：`anim/warning_shadow.zip`（`warningshadow`；`shadowmeteor` 也声明）、`anim/meteor_shadow.zip`（`meteorwarning`）、`anim/shadow_skittish.zip`、`anim/shadow_creatures_ground.zip`、`anim/eyes_darkness.zip`、`anim/sounddebug.zip`。声音：上述 `dontstarve/...`、`dontstarve_DLC001/...`、`rifts4/worm_boss/distant`、`monkeyisland/primemate/announce`、`yoth_2026/fanfare/announce`。 | 声明不等于素材内容已验证；资源不随 Wiki 分发。 | [meteor 资源](../../../raw/current/scripts/prefabs/meteorwarning.lua#L1-L4)、[shadowmeteor 资源](../../../raw/current/scripts/prefabs/shadowmeteor.lua#L1-L7) |

### 原版生成点

| 注册族 | 生成入口与位置 | 端、条件与清理 | 证据 |
|---|---|---|---|
| `houndwarning_lvl1`～`lvl4` | `hounded` 的 `warning_sound_thresholds` 把 30／60／90／500 秒映射到 `LVL4`～`LVL1`，`DoWarningSound` 取首个满足 `剩余时间 <= time` 的条目向陆上玩家推事件（延迟玩家走 `DoDelayedWarningSound`，同样推事件）；`player_classified` 在客户端 `SpawnPrefab`，位置由工厂自己设为焦点周围的半径环。 | 仅主模拟推事件；客户端生成；每档声源播完自删，无宿主清理。 | [阈值](../../../raw/current/scripts/components/hounded.lua#L74-L80)、[推送](../../../raw/current/scripts/components/hounded.lua#L890-L898)、[延迟推送](../../../raw/current/scripts/components/hounded.lua#L916-L925)、[客户端生成](../../../raw/current/scripts/prefabs/player_classified.lua#L1050-L1072) |
| `wormwarning_lvl1`～`lvl4`、`wormwarning_worm_boss` | 洞穴 `hounded` 装配的 `warning_sound_thresholds`：普通波映射 `LVL4_WORM`～`LVL1_WORM`，升级波四档全部映射 `WORM_BOSS` 且带 `quake=true`；事件经同一 `player_classified` 通道生成。 | 洞穴世界；升级波在 `DoWarningSound` 里额外向世界推 `ms_miniquake`（目标为该玩家）；客户端生成后自删。 | [洞穴阈值](../../../raw/current/scripts/prefabs/cave.lua#L296-L312)、[地震分支](../../../raw/current/scripts/components/hounded.lua#L890-L902) |
| `deerclopswarning_lvl1`～`lvl4` | `deerclopsspawner:DoWarningSound` 按计时器剩余时间选档，在目标玩家世界位置 `SpawnPrefab`；每客户端再按自身与代理的距离摆本地声源。 | 主模拟生成、1 秒移除代理；客户端 0 秒建声源；声音半径 25／20／15／5，range 40。 | [生成](../../../raw/current/scripts/components/deerclopsspawner.lua#L319-L326)、[参数](../../../raw/current/scripts/prefabs/monsterwarningsounds.lua#L6-L30) |
| `beargerwarning_lvl1`～`lvl4` | `beargerspawner:DoWarningSound` 同构，按 `BEARGER_TIMERNAME` 剩余时间选档，在目标玩家位置生成。 | 主模拟生成；档位条件与巨鹿相同；range 40。 | [生成](../../../raw/current/scripts/components/beargerspawner.lua#L287-L294)、[参数](../../../raw/current/scripts/prefabs/monsterwarningsounds.lua#L31-L53) |
| `krampuswarning_lvl1`～`lvl3` | `kramped:_DoWarningSound` 在差值 `score<20` 时按 `score<5`／`<15`／其余选 3／2／1，延迟 `1+math.random()*2` 秒后在玩家位置生成。 | 主模拟生成；三层距离 0，客户端听感在焦点；代理 1 秒移除。 | [生成](../../../raw/current/scripts/components/kramped.lua#L175-L190)、[参数](../../../raw/current/scripts/prefabs/monsterwarningsounds.lua#L54-L72) |
| `acidbatwavewarning_lvl1` | `acidbatwavemanager:IssueWarningForPlayer` 在每次到期时于玩家位置生成，并重排下一次警告时间。 | 主模拟生成；range 12、距离 0；只在有酸蝠潮候选的玩家元数据上触发。 | [生成](../../../raw/current/scripts/components/acidbatwavemanager.lua#L375-L382)、[参数](../../../raw/current/scripts/prefabs/monsterwarningsounds.lua#L73-L83) |
| `piratewarningsound` | `piratespawner:spawnpirateship` 在返回海盗船前，用船生成点 `pt` 生成并设世界位置；该点距玩家 `RANGE=38`。 | 主模拟生成；客户端各自按本地玩家 40＋平台半径判断并播放；代理 1 秒移除。 | [生成](../../../raw/current/scripts/components/piratespawner.lua#L355-L365)、[范围常量](../../../raw/current/scripts/components/piratespawner.lua#L271-L271) |
| `yothknightwarningsound` | `yoth_knightmanager:TryToSpawnKnights` 找到玩家周围 6～20 的可走点后调用 `SpawnKnights(hat, pos+offset)`，后者先 `DoWarningSound(pos)` 生成。 | 主模拟生成；`RANGE=60`、clamp 15；代理 1 秒移除。 | [调用](../../../raw/current/scripts/components/yoth_knightmanager.lua#L155-L164)、[生成](../../../raw/current/scripts/components/yoth_knightmanager.lua#L69-L82) |
| `meteorwarning` | `shadowmeteor:SetSize` 先生成并把 `warnshadow` 缩放为陨石大小，再父挂陨石、调用 `startfn(1,.33,1)`；`shadowmeteor` 本身由 `meteorshower:SpawnMeteor` 或 `moon_device` 生成。 | 主模拟生成并驱动；`onexplode` 移除 `warnshadow`；工厂无自删。 | [生成](../../../raw/current/scripts/prefabs/shadowmeteor.lua#L228-L230)、[挂接与驱动](../../../raw/current/scripts/prefabs/shadowmeteor.lua#L293-L299)、[爆炸清理](../../../raw/current/scripts/prefabs/shadowmeteor.lua#L49-L51)、[陨石雨](../../../raw/current/scripts/components/meteorshower.lua#L157-L166)、[月石装置](../../../raw/current/scripts/prefabs/moon_device.lua#L596-L599) |
| `warningshadow` | 洞穴 `quaker` 在地震坠落物下方生成并按高度缩放；遗迹落石 `ruins_cavein_obstacle` 在 `fall` 中生成；草鳄 `SGgrassgator` 的 `fall` 状态在首次 `onupdate` 生成，并在宿主提供 `UpdateShadowSize` 时随高度缩放。 | 主模拟生成、联网显示；以上宿主都用 `onremove` 或自身移除回调清理；`cave.lua` 的 `prefabs` 数组只是预载声明。 | [quaker](../../../raw/current/scripts/components/quaker.lua#L359-L362)、[遗迹](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L225-L228)、[草鳄](../../../raw/current/scripts/stategraphs/SGgrassgator.lua#L273-L279) |
| `shadowskittish`、`shadowwatcher`、`creepyeyes` | 世界组件 `hallucinations` 的三个 `spawnfn` 以随机角在玩家附近生成（惊走者 15 距离、窥视眼 5～15、窥视者绕 60 内火焰 27 距离）；理智阈值分别为惊走者 0.8、窥视眼 0.6、窥视者 0.5。暗度条件各不相同：惊走者只查附近女王火炬、无光照判定；窥视眼要求 `light<=.05`；窥视者用 `TUNING.DARK_SPAWNCUTOFF`；窥视眼／窥视者另受 `nightonly`（夜晚、非满月、`CanEntitySeeInDark`）门控。 | `hallucinations` 同时挂在森林与洞穴世界 `common_postinit` 的非专服分支，洞穴也会生成；实体工厂无 Network，围绕本地 `_player` 生成，故是各端本地表现。 | [惊走者](../../../raw/current/scripts/components/hallucinations.lua#L138-L163)、[窥视者](../../../raw/current/scripts/components/hallucinations.lua#L107-L136)、[窥视眼](../../../raw/current/scripts/components/hallucinations.lua#L74-L102)、[夜晚门控](../../../raw/current/scripts/components/hallucinations.lua#L186-L192)、[森林挂载](../../../raw/current/scripts/prefabs/forest.lua#L621-L634)、[洞穴挂载](../../../raw/current/scripts/prefabs/cave.lua#L340-L346) |
| `sounddebugicon` | `debugsounds` 在 `SOUNDDEBUGUI_ENABLED` 为真时于被播放声音的实体位置生成图标；`c_sounddebugui()` 是打开该开关的控制台入口，默认关闭。 | 仅开发／调试；`world.lua` 的 `prefabs` 数组只是预载；正常游玩路径没有生成点。 | [钩子](../../../raw/current/scripts/debugsounds.lua#L26-L40)、[命令](../../../raw/current/scripts/consolecommands.lua#L1357-L1364)、[默认值](../../../raw/current/scripts/main.lua#L66-L67)、[预载](../../../raw/current/scripts/prefabs/world.lua#L68-L70) |

### 预载与反查边界

- 反查以整个 `raw/current/scripts` 为范围，检索了 29 个完整注册名、`warning_lvl`／`warningsound` 前缀、`SpawnPrefab` 调用与 `AddComponent("hallucinations")`、`AddComponent("quaker")`；`forest.lua`、`cave.lua` 的 `monsters` 循环只是把名字插入世界 `prefabs` 数组，供资产预载，不能当作生成。[森林预载](../../../raw/current/scripts/prefabs/forest.lua#L581-L592)、[洞穴预载](../../../raw/current/scripts/prefabs/cave.lua#L207-L216)
- `lunarhailmanager.lua` 里仍有 `SpawnPrefab("warningshadow")`，但该组件在 `forest.lua` 的挂载行已被注释为 deprecated，本快照没有其他 `AddComponent("lunarhailmanager")`；因此这条生成链在原版当前流程中不可达，只作历史实现看待。[遗留调用](../../../raw/current/scripts/components/lunarhailmanager.lua#L289-L292)、[弃用挂载](../../../raw/current/scripts/prefabs/forest.lua#L711-L711)
- `tuning_override.lua` 的 `creepyeyes = dummyfn` 是区域触发器覆盖表的键（`AreaTrigger:DoOverride` 按名取函数），和 `hallucinations` 的生成无关；`prefablist.lua` 也只列文件名。`warningshadow` 只被 `cave.lua` 的 `prefabs` 数组预载，但预载声明与工厂注册要分开看。[覆盖键](../../../raw/current/scripts/tuning_override.lua#L37-L37)、[覆盖消费](../../../raw/current/scripts/components/area_trigger.lua#L8-L16)、[洞穴预载](../../../raw/current/scripts/prefabs/cave.lua#L11-L11)

## 典型接入

### 为坠落物挂可缩放的落点阴影

- 前提与选择：在服务端为下落物创建 `warningshadow`，用宿主高度算缩放并在宿主移除时一并移除；不要选 `meteorwarning`，后者带淡变与陨石声，且只被 `shadowmeteor` 驱动。[工厂](../../../raw/current/scripts/prefabs/warningshadow.lua#L6-L28)
- 生效与撤销：实体联网、无组件、无自删；宿主 `onremove` 是唯一清理点。原版 `quaker`、`ruins_cavein_obstacle`、`SGgrassgator` 均按同一模式接。[quaker](../../../raw/current/scripts/components/quaker.lua#L359-L362)、[草鳄](../../../raw/current/scripts/stategraphs/SGgrassgator.lua#L273-L279)
- 原版依据：缩放公式取 `Lerp(.5, 1.5, height/35)`（`quaker`）；遗迹落石改用三倍区间 `Lerp(1.5, 4.5, height/35)`，说明缩放范围由宿主自定，工厂不内建。[quaker 缩放](../../../raw/current/scripts/components/quaker.lua#L89-L92)、[遗迹缩放](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L178-L181)

```lua
local function AttachFallShadow(host, x, z, height)
    local shadow = SpawnPrefab("warningshadow")
    shadow.Transform:SetPosition(x, 0, z)
    local scale = Lerp(.5, 1.5, height / 35) -- 与 quaker 同式；35 为源高度基准
    shadow.Transform:SetScale(scale, scale, scale)
    host:ListenForEvent("onremove", function()
        shadow:Remove()
    end)
    return shadow
end
```

- 调整与验证：示例是原版调用的组合，不是新增接口；未在游戏运行。若宿主会被池化或跨存档保留，需在回收时显式 `Remove` 阴影，不能依赖 `persists=false` 自动清理。

## 常见陷阱与限制

- **非联网工厂不能跨端补表现**：`houndwarning*`、`wormwarning*`、`shadowskittish`、`shadowwatcher`、`creepyeyes` 都没有 `AddNetwork`。在服务端 `SpawnPrefab("houndwarning_lvl4")` 不会让客户端听到声音；原版走 `player_classified` 的客户端生成，暗影三件套则由世界组件在本地端生成。[猎犬工厂](../../../raw/current/scripts/prefabs/houndwarning.lua#L23-L40)、[暗影工厂](../../../raw/current/scripts/prefabs/creepyeyes.lua#L33-L40)
- **预警声的距离衰减与注释不完全一致**：`PlayWarningSound` 把 `easing.inQuad(dist-range, radius, 30-radius, range+10)` 作为半径；分母是 `range+10` 而不是 10，距离刚超过 range 时半径只微增（range 40、radius 25 时上限约 0.2），更远时还可超过 30。注释所称“2 倍 range + buffer 时偏移到 30”与当前参数不符，实际听感未运行核验。[缓动调用](../../../raw/current/scripts/prefabs/monsterwarningsounds.lua#L95-L103)、[inQuad](../../../raw/current/scripts/easing.lua#L44-L47)
- **两类“落点阴影”清理责任不同**：`warningshadow` 完全靠宿主移除；`meteorwarning` 虽然同理，但原版只在 `shadowmeteor:onexplode` 中移除它，其他方式生成后若不手动清理会残留。[阴影宿主](../../../raw/current/scripts/components/quaker.lua#L359-L362)、[陨石清理](../../../raw/current/scripts/prefabs/shadowmeteor.lua#L49-L51)
- **坎普斯与酸蝠的距离 0 不是缺陷**：这两族把 `distance` 配成 0，本地声源落在焦点，随机角不产生位移；方向感只存在于巨鹿／熊獾的 25／20／15／5 档。[参数表](../../../raw/current/scripts/prefabs/monsterwarningsounds.lua#L54-L83)
- **`sounddebugicon` 是调试工具而非死代码**：唯一生成点在 `debugsounds` 的 `PlaySound` 钩子内，只有 `c_sounddebugui()` 把 `SOUNDDEBUGUI_ENABLED` 置真后才创建；`world.lua` 的预载和 `main.lua` 的默认关闭都不能当作正常生成。[钩子](../../../raw/current/scripts/debugsounds.lua#L33-L40)、[命令](../../../raw/current/scripts/consolecommands.lua#L1357-L1364)
- **官方中文名缺口**：`languages/chinese_s.po` 对 29 个注册名逐一检索 `STRINGS.NAMES.<注册名>` 均为 0 命中（检索范围 [NAMES 全段](../../../raw/current/scripts/languages/chinese_s.po#L338464-L353135)），因此本页标题、别名与表内中文都是效果描述，不能当官方译名引用。

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394。逐文件核对 11 个工厂的循环上限（`WARNING_LEVEL_DISTANCE` 4 档、`monster_params` 的 4／4／3／1 档）、29 个展开名、端分界、父挂与坐标、随机角度／延迟、自删与宿主清理；对 `raw/current/scripts` 全树反查注册名与前缀，确认 `hounded`、`player_classified`、各 spawner／manager、`hallucinations`、`quaker`、`ruins_cavein_obstacle`、`SGgrassgator`、`debugsounds` 等生成点，并区分预载声明、已弃用调用与同名覆盖键。未覆盖：音频素材内容与游戏内听感、网络延迟下的实际同步、模组动态注册。
- 分档理由：本页需同时保留 11 个工厂、29 个动态展开名、十余个生成／消费端及端／清理差异，若压到标准档会删减对照证据，故按“多状态、多变体、复杂生命周期或协作”选 `complex`（400 行／32 KiB）。
- 游戏运行：未运行。
- 结构检查：`AttachFallShadow` 示例经 `parser_probe.exe --syntax-only` 通过；`validate_page.py` 报告 121 行／30,911 字节（`complex` 上限 400 行／32 KiB 内）、无问题；`validate_source_links.py --strict` 报告 106 处源引用、0 错误、0 警告。

返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]。
