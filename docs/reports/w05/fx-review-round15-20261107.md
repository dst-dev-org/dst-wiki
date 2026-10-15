# FX 第 15 轮独立检视：外围 U7～U10

核验日期：2026-11-07；实际源码快照：`steam-buildid:26335394`；知识仓 HEAD：`97afb4e9e6ca05390558769926f1f9c8598d0f70`。

**当前结论：修后复验尚未收口。原 28 项中 24 项正文事实修订通过，R06／R19／R23／R25 各余局部补正；另有证据对齐 P2（R29）及维护日期 P3（R30），详见文末。** 初检 P1×0、P2×26、P3×2 保留为历史。只维护检视报告与日志，不改知识页、索引、基线、PLAN、台账、Skill、工具或 raw，不 record、不提交推送；未运行游戏。

## 范围与逐页结论

12 页、**1389 个物理行／233,424 字节、57 个主要覆盖注册名**，声明 120 组 sources、96 个不同文件。57 名包含 `voidcloth_scythe` 本体的表现链切片和 `tophat_container` 技术实体，不等于 57 个纯 FX，也不表示魔镰玩法全文已完成。通读全部页面，逐注册核工厂、必要调用、端、门禁、父子／跟随、清理、数值和示例；没有把 96 份来源的全部玩法全文重审。

| 页面（均位于 wiki/reference/prefabs） | 注册数 | 行／字节 | 独立检视结论 |
|---|---:|---:|---|
| [trap_fumarole_burn_fx](../../../wiki/reference/prefabs/trap_fumarole_burn_fx.md) | 2 | 104／17,651 | R02～R05 |
| [boatrace_start_flag](../../../wiki/reference/prefabs/boatrace_start_flag.md) | 4 | 99／18,021 | R02、R06～R08；P3：R27 |
| [shadow_bishop_fx](../../../wiki/reference/prefabs/shadow_bishop_fx.md) | 4 | 108／21,597 | R02；P3：R27 |
| [voidcloth_boomerang_fx](../../../wiki/reference/prefabs/voidcloth_boomerang_fx.md) | 5 | 119／22,692 | R02、R09～R12 |
| [vault_compass_visual](../../../wiki/reference/prefabs/vault_compass_visual.md) | 4 | 121／21,987 | R02、R13～R15 |
| [wendy_gravestone_rebirth_fx](../../../wiki/reference/prefabs/wendy_gravestone_rebirth_fx.md) | 2 | 101／14,779 | 未发现 P1/P2 |
| [hats](../../../wiki/reference/prefabs/hats.md) | 12 | 153／28,416 | R16、R17 |
| [mushtree_tall_burntfx](../../../wiki/reference/prefabs/mushtree_tall_burntfx.md) | 7 | 106／13,932 | R01、R26；P3：R28 |
| [moon_altar_break](../../../wiki/reference/prefabs/moon_altar_break.md) | 3 | 104／13,859 | R18；P3：R28 |
| [attackfx_handpillow_petals](../../../wiki/reference/prefabs/attackfx_handpillow_petals.md) | 7 | 138／22,783 | R24、R25 |
| [bramblefx_armor](../../../wiki/reference/prefabs/bramblefx_armor.md) | 3 | 109／17,286 | R19、R20 |
| [burntground](../../../wiki/reference/prefabs/burntground.md) | 4 | 127／20,421 | R21～R23 |

本轮属于首次整页独立检视。第 05 轮虽引用 hats.lua 的蚁狮帽段，并未检视 hats.md；不据交叉引用重复计入已检视页。

## P2：事实、条件和证据边界

### R01｜蘑菇树页正文来源有一项未声明

- 位置：[mushtree_tall_burntfx:11](../../../wiki/reference/prefabs/mushtree_tall_burntfx.md#L11)、[第 59 行](../../../wiki/reference/prefabs/mushtree_tall_burntfx.md#L59)。
- 正文引用 `prefabs/mushtree_webbed.lua`，sources 未含该文件；验证记录第 93 行又自报核验过它。本轮 120 组 unchanged 不覆盖这个漏项。
- 建议：核验后补入并按既有规则刷新本页基线，或同步移除该证据及相应断言；不要为消除提示批量删其他来源。

### R02｜五页的清理说明仍混淆 SetParent 与 AddChild

共同依据：[AddChild](../../../raw/current/scripts/entityscript.lua#L921-L935) 会登记 Lua children，[实体移除](../../../raw/current/scripts/entityscript.lua#L1779-L1796) 显式遍历该表；单独 SetParent 的原生级联不能由此证明。

| 页面位置 | 当前问题 | 实际依据及建议 |
|---|---|---|
| [trap_fumarole_burn_fx:61](../../../wiki/reference/prefabs/trap_fumarole_burn_fx.md#L61) | 藤蔓底座直接写“宿主被移除时靠引擎父挂级联” | [OnInit](../../../raw/current/scripts/prefabs/trap_vines.lua#L178-L186) 仅 SetParent；正常 DespawnTrap 有显式脱父、KillFX，但直接删宿主的原生级联未核验，须分开。 |
| [shadow_bishop_fx:62](../../../wiki/reference/prefabs/shadow_bishop_fx.md#L62) | 裂隙粒子写成关门／拆门走“引擎父子级联” | [CreateParticleFx](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L206-L210) 实际用 AddChild，已有 Lua 子项删除路径。第 22 行也应统一：巨口替身的 animover 仅发事件，不能先称“其余三个自删”再用括号否定。 |
| [boatrace_start_flag:56](../../../wiki/reference/prefabs/boatrace_start_flag.md#L56) | 浮标删除归因于引擎级联，并由此称无显式清理路径 | [SERVER_DeployBobber](../../../raw/current/scripts/prefabs/boatrace_start.lua#L791-L797) 用 AddChild，塔的 Remove 会进入 Lua children 循环；应写“无专门清环函数，宿主删除有通用 Lua 回收”。 |
| [voidcloth_boomerang_fx:73](../../../wiki/reference/prefabs/voidcloth_boomerang_fx.md#L73) | 装备／卸装阶段的父挂均被概括为随持有者或本体级联 | [SetFxOwner](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L93-L115) 仅 SetParent；已证实的 FxRemoveAll 是骨架被 Remove 时清本地层，不证明骨架何时被宿主带走。保留实际挂接和显式清理，其他标未核验。 |
| [vault_compass_visual:78](../../../wiki/reference/prefabs/vault_compass_visual.md#L78) | 净化器本体直接删除被写成雾效必随父挂删除 | [StartMisting](../../../raw/current/scripts/prefabs/vault_decon_mister.lua#L31-L44) 仅 SetParent；[StopMisting](../../../raw/current/scripts/prefabs/vault_decon_mister.lua#L47-L67) 显式删除成立，[本体工厂](../../../raw/current/scripts/prefabs/vault_decon_mister.lua#L69-L96) 未另接删除回调。不要把正常停雾等同所有宿主删除。 |

同步相关摘要、快速判断和陷阱。**本项不反向断言实际泄漏，也不要求修改原版游戏来消除未知。**

### R03｜移除灼烧宿主不等于调用组件 OnRemoveFromEntity

- 位置：[trap_fumarole_burn_fx:44](../../../wiki/reference/prefabs/trap_fumarole_burn_fx.md#L44)、第 60 行。
- [TrapFumaroleBurning](../../../raw/current/scripts/components/trapfumaroleburning.lua#L15-L22) 只实现 OnRemoveFromEntity；[RemoveComponent](../../../raw/current/scripts/entityscript.lua#L670-L684) 会调用它，而[实体 Remove](../../../raw/current/scripts/entityscript.lua#L1748-L1757) 调的是组件 OnRemoveEntity。
- 建议：保留“离区后移除组件显式删 FX”；撤回“宿主移除时同样走组件移除路径”。宿主直接删除的原生父挂结果保留未核验，并补实际采用的 EntityScript 证据。

### R04｜煤炭火伤的 10 是每秒基数

- 位置：[trap_fumarole_burn_fx:46](../../../wiki/reference/prefabs/trap_fumarole_burn_fx.md#L46)。
- 文中“每 14 帧结算 10 火伤”漏掉倍率和时间累计。[实际调用](../../../raw/current/scripts/components/trapfumaroleburning.lua#L93-L103) 是 `10 * ignitemult * firedamagedt`；[14 帧](../../../raw/current/scripts/components/trapfumaroleburning.lua#L53-L54) 是最低结算间隔，[tuning](../../../raw/current/scripts/tuning.lua#L9501-L9512) 也注明每秒火伤。
- 建议：写成“每隔至少 14 帧，按基数 10／秒 × 温度倍率 × 实际间隔结算”，不把 10 当单跳固定值。

### R05｜藤蔓的 variation 并未按同名字段正确往返

- 位置：[trap_fumarole_burn_fx:62](../../../wiki/reference/prefabs/trap_fumarole_burn_fx.md#L62)。
- [OnSave](../../../raw/current/scripts/prefabs/trap_vines.lua#L189-L192) 写的是原版拼写 `data.varation`；[OnLoad](../../../raw/current/scripts/prefabs/trap_vines.lua#L194-L206) 却读 `data.variation`。当前“OnSave 记 variation”掩盖了字段不配对，普通往返不能据此保留随机外观。
- 建议：只记实际字段与默认回落 1 的源码事实；不改原版拼写，不扩展为已运行复现。

### R06｜龙舟旗图案用选手编号，名次用挂点

- 位置：[boatrace_start_flag:20](../../../wiki/reference/prefabs/boatrace_start_flag.md#L20)、第 39、84 行。
- [testforwinpresentation](../../../raw/current/scripts/prefabs/boatrace_start.lua#L181-L194) 调 `setflag(名次位置, beacon._index)`；[setflag](../../../raw/current/scripts/prefabs/boatrace_start.lua#L634-L654) 用 position 选 flag1～3、id 选 swapflag 与 winid。烟花也用这个选手 id 换符号（[winOver](../../../raw/current/scripts/prefabs/boatrace_start.lua#L573-L579)），不是按第一名图案 1、第二名图案 2 固定映射。
- 建议：分清“前三名挂点”与“选手／船编号图案”。第 39 行也收窄为门禁允许时才出下一面旗：处于 win/prize 会暂缓，[prizeOver](../../../raw/current/scripts/prefabs/boatrace_start.lua#L200-L211) 再处理排队名次，并非每个完赛事件立即切 win。

### R07｜烟花并未跟随起点塔符号

- 位置：[boatrace_start_flag:6](../../../wiki/reference/prefabs/boatrace_start_flag.md#L6)、第 22 行。
- [winOver](../../../raw/current/scripts/prefabs/boatrace_start.lua#L573-L579) 对烟花只 SetPosition、换符号；没有 SetParent／FollowSymbol。工厂有 Follower 不等于消费者启用了跟随。
- 建议：摘要拆成“旗帜符号跟随；烟花在塔当时的位置独立播放”，避免把三种 FX 都归为宿主符号跟随。

### R08｜浮标不是整片竞赛区域的禁建判定

- 位置：[boatrace_start_flag:22](../../../wiki/reference/prefabs/boatrace_start_flag.md#L22)、第 56、80 行。
- 两个 Map 消费点只检查[目标瓦片上的实体](../../../raw/current/scripts/components/map.lua#L491-L498)、[另一入口同样如此](../../../raw/current/scripts/components/map.lua#L533-L536)；浮标是半径 12 上的 10 个离散点（[铺环](../../../raw/current/scripts/prefabs/boatrace_start.lua#L799-L813)）。
- 建议：写“有 dockjammer 的目标瓦片会拒绝部署”；不能由这些点推出整个圆内或整个竞赛区禁止修码头，也不反向保证其他瓦片一定可建。

### R09｜虚布页“5 名都无官方名”与自身引用冲突

- 位置：[voidcloth_boomerang_fx:24](../../../wiki/reference/prefabs/voidcloth_boomerang_fx.md#L24)、第 108 行。
- 本页把 `voidcloth_scythe` 本体纳入 5 注册；它恰有 [NAMES.VOIDCLOTH_SCYTHE](../../../raw/current/scripts/languages/chinese_s.po#L351359-L351362)＝暗影收割者。第 24 行前半已经正确引用，后半又写 5 名均无键。
- 建议：改为其余四个附属 FX／弹体名无独立 NAMES 键，本体有官方名；范围数字保持一致。

### R10｜不能把回旋镖 OnEquip 也说成 GetSkinBuild 样板分支

- 位置：[voidcloth_boomerang_fx:67](../../../wiki/reference/prefabs/voidcloth_boomerang_fx.md#L67)。
- [回旋镖 OnEquip](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L16-L24) 无 GetSkinBuild，且只清 swap_object；有皮肤样板分支的是[魔镰](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L171-L184)与[暗影伞](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L19-L26)。本页第 22、97 行原已说明回旋镖外观全部由帧层承担。
- 建议：把该句限制到实际两个函数；当前 prefabskins 未登记装备皮肤，不代表三个装备有相同的换装代码。

### R11｜暗影伞 wave／dome 是淡出并扩张

- 位置：[voidcloth_boomerang_fx:63](../../../wiki/reference/prefabs/voidcloth_boomerang_fx.md#L63)。
- [WaveFxOnUpdate](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L154-L166)：`q=(1-t/0.5)^2` 用于 alpha；scale＝`(2-1.7*q)*scalemult`，从约 `0.3*scalemult` 增至 `2*scalemult`，并非收缩。
- 建议：写“0.5 秒内按平方包络淡出、向外扩张”，不要把透明度包络直接当缩放。

### R12｜虚布 FX 的非专服分支不等于仅客户端

- 位置：[voidcloth_boomerang_fx:98](../../../wiki/reference/prefabs/voidcloth_boomerang_fx.md#L98)。
- [FxToggleEquipped](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L628-L635) 在非专服主模拟直接建本地层；[FollowSymbolFxFn](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L638-L667) 即使在专服也建 AnimState、Follower、highlightchild、colouraddersync，不能称“Transform/Network 空壳”。魔镰同构。
- 建议：本地帧层存在于客户端及非专服主模拟；专服不建本地帧层，但保留联网骨架的实际组件。

### R13｜父挂 marker 不因 persists 默认 true 就独立入档

- 位置：[vault_compass_visual:45](../../../wiki/reference/prefabs/vault_compass_visual.md#L45)、第 77、112 行。
- [OnEquip](../../../raw/current/scripts/prefabs/vault_compass.lua#L87-L98) 把 marker SetParent 到持有者；[世界存档入口](../../../raw/current/scripts/mainfunctions.lua#L1113-L1115) 还要求 `entity:GetParent()==nil`。所以不能由 marker 工厂未设 persists=false 推出装备期它会写入世界存档、读档留下无指针 marker。
- 建议：撤回该具体跨会话推导，保留未实测范围；本体正常卸装的 ClearMarker 显式删两者仍成立。若保留存档分析，补所依据的 mainfunctions 来源与锚点。

### R14｜地纹朝向的折算与镜像条件写反了

- 位置：[vault_compass_visual:60](../../../wiki/reference/prefabs/vault_compass_visual.md#L60)、第 100 行。
- [SetOrientation](../../../raw/current/scripts/prefabs/vault_ground_pattern.lua#L22-L36)：orientation>2 且 variation<3 且未 HideCenter 时先减 2，得到 1/2，走正常缩放与 0/180°。其余保留的 3/4 才镜像；原版 else 中 `orientation==1` 不可达，3/4 都落 -90°，不能写 ±90°。
- 建议：按这两层判定正向描述；不要把“无中心变体”与 `not nocenter` 混为同一条件。正文与陷阱同步。

### R15｜罗盘不是任一分支找不到方向都会变 fail

- 位置：[vault_compass_visual:42](../../../wiki/reference/prefabs/vault_compass_visual.md#L42)。
- [VAULT 分支](../../../raw/current/scripts/prefabs/vault_compass.lua#L25-L49) 中，当前非 key1 且 direction 为 nil 时没有 else，不调 ChangeToAnim，保留旧动画／方向。direction 有值但未找到匹配传送器才置 fail。
- 建议：去掉“任何分支找不到目标→fail”的全称，把 VAULT 无路径与找不到匹配传送器分开。

### R16｜舞台剧结束不保证演员实体退场删除 FX 壳

- 位置：[hats:100](../../../wiki/reference/prefabs/hats.md#L100)。
- [do_mask_blink](../../../raw/current/scripts/play_commonfn.lua#L65-L70) 只给演员生成并 AttachToOwner；[CollectCast](../../../raw/current/scripts/components/stageactingprop.lua#L107-L127) 收集现有穿戏服实体，[EndPerformance](../../../raw/current/scripts/components/stageactingprop.lua#L240-L270) 解除 acting／stage、推 stopstageacting，没有普遍删演员或 actor.fx；继续追到 [StopStageActing](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L1026-L1030) 也只恢复动作显示。本地 blink 层的 animover 删除不删除联网壳。
- 建议：保留“本地层播完自删、壳未见演出结束显式 Remove”；删除“演员随后退场连带”的保证。不能据此断言实际永久泄漏或原生级联行为。

### R17｜帽外观与跟随符号不能一概按 swap_hat

- 位置：[hats:130](../../../wiki/reference/prefabs/hats.md#L130)、第 133 行；第 39 行也应补齐条件。
- [lunarplant_onequip](../../../raw/current/scripts/prefabs/hats.lua#L3814-L3824) 已明确注释 swap_hat 为空、外观全靠 follow symbols；[蝙蝠帽装备](../../../raw/current/scripts/prefabs/hats.lua#L6533-L6539) 也清 swap_hat，不能概括为“帽可见外观仍靠该 override”。
- [注册数据](../../../raw/current/scripts/prefabs/hats.lua#L7970-L7984) 中 lunarplant／voidcloth 和 pumpkin 一样 isfullhelm=true；[实际选择符号](../../../raw/current/scripts/prefabs/hats.lua#L7755-L7763) 还要求 owner.isplayer 及 BuildHasSymbol。第 133 行“其余八个都跟 swap_hat”错误。
- 建议：说明外观有静态符号与全跟随层两类；按完整条件说明三类 fullhelm 层可选 headbase_hat，其他情况退回 swap_hat。

### R18｜祭坛破碎读档“静止第一帧”缺乏依据

- 位置：[moon_altar_break:59](../../../wiki/reference/prefabs/moon_altar_break.md#L59)、第 84 行相关存档结论。
- 未置 persists=false、只监听 animover 属实；但[SpawnSaveRecord](../../../raw/current/scripts/mainfunctions.lua#L464-L474) 会重新 SpawnPrefab，[本工厂](../../../raw/current/scripts/prefabs/moon_altar_break.lua#L18-L31) 再次 PlayAnimation("break") 并装 animover。当前证据不能推出读档固定为静止第一帧。
- 建议：保留符合存档门槛时可能入档、工厂重建会重新启动动画的源码路径；离屏睡眠对播放／退场的实际影响未运行核验，不写“正常一定不入档”或“读档一定静止”。

### R19｜trapdamage 对出生保护的作用方向反了

- 位置：[bramblefx_armor:92](../../../wiki/reference/prefabs/bramblefx_armor.md#L92)。
- [危险标签与判定](../../../raw/current/scripts/components/playerspawner.lua#L106-L127) 是在相应世界设置、非 loading 且世界天数等条件满足时，附近危险实体数量 >1 可使 `_ShouldEnableSpawnProtection` 返回 true。并非“附近放陷阱会阻止重生保护”。
- 建议：写“该标签参与启用出生保护的危险计数”，保留设置、loading、天数及数量门槛，不能把一枚瞬时 FX 等同必然启用。

### R20｜荆棘选目标还受 owner 战斗门禁和 owner 类型限制

- 位置：[bramblefx_armor:41](../../../wiki/reference/prefabs/bramblefx_armor.md#L41)、第 91 行。
- [有 owner 的分支](../../../raw/current/scripts/prefabs/bramblefx.lua#L25-L37) 还要求 owner 有 combat、CanTarget 且非 IsAlly；不是只过距离、ignore、可见和装备免疫就必结算。[SetFXOwner](../../../raw/current/scripts/prefabs/bramblefx.lua#L64-L69) 的 canhitplayers 为“owner 非玩家或 PVP”，不是所有 armor 用途都受 PVP 开关限制。
- 建议：在结算节补 owner 有效／失效后的分流与战斗门禁；陷阱写为“玩家穿戴时非 PVP 不打其他玩家”，不要推广到非玩家 owner。本页两档 armor 的正常 proc 门槛保留，不因这个修正删除冷却或技能条件。

### R21｜蜂蜜周期减速取的是初始化位置

- 位置：[burntground:48](../../../wiki/reference/prefabs/burntground.md#L48)、第 63 行。
- [OnInit](../../../raw/current/scripts/prefabs/honey_trail.lua#L52-L60) 只取一次 x/y/z/scale，并作为 DoPeriodicTask 的固定参数；[OnUpdateServer](../../../raw/current/scripts/prefabs/honey_trail.lua#L5-L16) 没有重读 FX 位置。文中两处“每 tick 重读当前位置／位置随 FX 更新”均不成立。
- 建议：写“以初始化坐标及半径周期判定”；模组后续移动／缩放 FX 不会自动更新该闭包参数，须重新安排判定。原版固定落点用途不受此误述影响。

### R22｜焦地周期渐隐不是持续发送服务端 netvar

- 位置：[burntground:70](../../../wiki/reference/prefabs/burntground.md#L70)，第 35、37 行应保持同一口径。
- [UpdateFade](../../../raw/current/scripts/prefabs/burntground.lua#L13-L21) 两端都用 `_fade:set_local` 推进；真正 `set` 在[构造](../../../raw/current/scripts/prefabs/burntground.lua#L78-L80)与[加载](../../../raw/current/scripts/prefabs/burntground.lua#L30-L40)。dirty 可接收权威赋值，但不是每个服务端 fade tick 的广播。
- 建议：保留两端周期相位不同与构造／加载同步，删去暗示“周期 α 始终由服务端覆写”的描述；不臆测原生 netvar 的额外行为。

### R23｜蜂蜜生成参数有反向阈值与范围漏项

- 位置：[burntground:58](../../../wiki/reference/prefabs/burntground.md#L58)、第 59、104 行。
- 暗影憎恶[速度分档](../../../raw/current/scripts/prefabs/shrouden.lua#L207-L227) 为 speed<1 不落蜜、1≤speed<4 选 threshold=6、speed≥4 选 threshold=3（[表](../../../raw/current/scripts/prefabs/shrouden.lua#L128-L140)）；当前“快时 6／慢时 3”写反。
- 弹弓弹[OnUpdate_Honey](../../../raw/current/scripts/prefabs/slingshotammo.lua#L215-L220) 是 `k=(1-elapsed/8)^2*0.6+0.3`，传入 min=k、max=k+0.2；[实际生成](../../../raw/current/scripts/prefabs/slingshotammo.lua#L204-L209) 的 duration 也有 +[0,0.5) 随机值。当前把尺寸压成 0.9→0.3 单线、时长固定 2 秒漏掉范围。
- 建议：给公式／区间而非固定曲线；正常蜂王已有的 duration 随机量也保持一致，不要求重审 Boss 全部移动玩法。

### R24｜挖起会收获的正确子集被标成了错误命题

- 位置：[attackfx_handpillow_petals:121](../../../wiki/reference/prefabs/attackfx_handpillow_petals.md#L121)。
- 陷阱标题“误区：挖起会收获作物”在成熟／枯萎子集中本来为真；正文第 82 行也正确说明。[OnDigUp](../../../raw/current/scripts/prefabs/plant_normal.lua#L100-L127) 确实先 Harvest，失败后才按生长阶段移除。
- 建议：改为“误区：挖起总会收获产物”，或正向“边界：仅成熟／枯萎时挖起会收获”。保留现有正确裁决表，不改成从不收获。

### R25｜冲锋尘土的随机倒数值不是生成间隔

- 位置：[attackfx_handpillow_petals:66](../../../wiki/reference/prefabs/attackfx_handpillow_petals.md#L66)。
- [骑枪周期回调](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L1176-L1193) 与[驼鹿周期回调](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22096-L22113) 都是在生成后设 delay=4～6，之后各回调先减到 0，再下一次回调才生成；相邻生成隔 5～7 次回调，不能直接写成每 4～6 帧生成。
- 建议：写实际倒数协议；若换算帧数，注明 0 周期按通常逐 tick 调度推导，未运行测量。两个复用者同改。

### R26｜读档焦壳实际来自保存时仍在燃烧的树

- 位置：[mushtree_tall_burntfx:87](../../../wiki/reference/prefabs/mushtree_tall_burntfx.md#L87)，第 56～57 行的“读档 burnt”说明也应完整。
- 常规树 [OnSave](../../../raw/current/scripts/prefabs/mushtree.lua#L353-L356) 与月树 [OnSave](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L94-L97) 的 data.burnt 取的是 IsBurning()；[OnLoad](../../../raw/current/scripts/prefabs/mushtree.lua#L635-L643) 据此直接进入 tree_burnt／stump_burnt。已正常烧尽的本体早已 Remove，并非通常等待读档的“已烧尽树”。
- 建议：说明“保存时处于燃烧中的树，读档按烧尽分支收尾”，别把这个字段名误读成保存了一棵已经烧尽但尚未清理的树。保留焦壳生成、掉落、树桩分流的正确内容。

## P3：证据区间与未验证数字

### R27｜两处译名锚点停在 msgid，未覆盖中文 msgstr

- [boatrace_start_flag:24](../../../wiki/reference/prefabs/boatrace_start_flag.md#L24) 的“龙蝇船”锚点止于 L342176，译文在 [L342177](../../../raw/current/scripts/languages/chinese_s.po#L342174-L342177)。
- [shadow_bishop_fx:21](../../../wiki/reference/prefabs/shadow_bishop_fx.md#L21) 的“狞笑”锚点止于 L349216，译文在 [L349217](../../../raw/current/scripts/languages/chinese_s.po#L349214-L349217)。
- 名称本身正确，路径／行号合法；建议延长到 msgstr，使读取该证据区间就能看到译文。严格校验通过不证明译文已在区间内。

### R28｜两页的“约 1～2 秒”不是已核验动画长度

- 位置：[mushtree_tall_burntfx:71](../../../wiki/reference/prefabs/mushtree_tall_burntfx.md#L71)、[moon_altar_break:58](../../../wiki/reference/prefabs/moon_altar_break.md#L58)。
- 页面验证记录均未核验动画包时长。蘑菇树用 GetCurrentAnimationLength()+FRAMES、祭坛只用 animover；Lua 依据并没有给出 1～2 秒常数。
- 建议：删去估计或标明未核验，按实际动画完成／定时表达式描述；不为填这个数字额外扩展资源审计。

## 本轮实际验证

| 检查 | 结果与边界 |
|---|---|
| 12 页 validate_page | 12/12 格式通过，issues 为空；hats／attackfx 为 complex，其余 standard，均在当前上限内。 |
| 全局 --check-indexes | 初跑整体退出 1，仅并行装备页 queen-torch、redlantern 各缺一次叶级登记；他方补齐后本轮重新运行，**12 页格式＋全局导航均通过，退出 0，issues 为空**：1147 知识页／34 索引／Top30 入口数 22。本轮没有代改两页或索引，保留先后检查口径。 |
| validate_source_links --strict | **632 个行引用／578 个唯一目标／91 个来源文件；0 error、0 warning，未截断，退出 0**。未验证正文是否受证据支持，也不检查 sources 漏列。 |
| validate_source_hashes check --fresh-hours 0 | **120 组 unchanged**，changed／sources_changed／unrecorded／invalid 等均 0，退出 0；只 check、不 record。R01 的漏项另记，不以这项通过掩盖。 |
| 标题原文 read | 12/12 成功：每页返回 1 个结果，complete_selected_ranges=true、truncated=false，退出均 0；读取当前 Markdown，不用 search 缓存。 |
| Lua 示例语法 | 12 个完整围栏各包独立 do/end，合成 4047 字节临时探针，经现有 parser_probe --syntax-only 通过；不执行示例。不安装／构建工具，临时文本用 apply_patch 建立并删除。 |
| 输入稳定性 | 最初 12 页＋96 声明来源共 **108 个字节指纹，收尾复比无变化**；检查结论没有跨用修改前后的页面。 |
| 报告与工作区 | 报告 113 个本地 Markdown 链接的文件及行区间检查通过；git diff --check 退出 0；无 .review-r15-* 临时残留。日志追加后 10,996 字节／13 行，未触发归档阈值。 |
| 名称与生成点检索 | 扫描 current/scripts 的 4095 个 Lua 文件，4093 个按 UTF-8/BOM、easing.lua 与 nis/xbnyn.lua 按 Latin-1 读取；57 名按完整标识符边界共 214 次候选出现，不当作生成点数量。另查枕头／plant_dug 动态前缀、蘑菇树 treeset、祭坛映射和地纹 SpawnTrackedPrefabAtXZ 等包装入口。候选收齐后分段核对，没有用截断结果证明不存在。 |
| .po | 逐名查 57 个直接 NAMES 键，仅 voidcloth_scythe 命中；其余显示名覆盖／宿主名另查。通读正文引用的 54 组不同区间，名称正确但有 R27 两处范围未含译文。 |

已核对温蒂两个工厂、SG 两状态／角色分流、墓碑退场；未发现 P1/P2。对帽子逐类跟随层／灯／容器、枕头动态注册与 crop/grower 收获、荆棘 owner/follower 归属、蜂蜜权威与本地预测分别对照；语法或引用通过均不替代这些语义核对。动画包观感、原生 SetParent 级联及游戏运行未验证。

当前还有一个**批外归属观察**：并行新建的 `voidcloth_scythe.md` 与本批族页都含 ASCII 别名 voidcloth_scythe。族页明确只展开表现切片，装备页属于他方工作范围；该新重叠不记作本批创建时的错误，也不代改。装备批维护时应确定本体主要查询归属并互链，不能仅靠别名机械冲突就删掉有效内容。

## 进度与后续

固定 158 页队列经 Git 新增列表与前 14 轮范围表、U15～U17 九页逐项对账：本轮初检后 **138/158 页，余 20 页未完成本对话整页独立检视**。这是检视覆盖数，**不是 138 页全部已收口**；第 05、12 轮的历史待复验不在本轮自动关闭。

后续可分两轮：

1. U11～U14 的 12 页：archive_ambient_sfx、atrium_gate_activatedfx、balloon_held_child、wintersfeastbuff_fx、witchbroom_fx、beargerfur_sack_frost_fx、bat_bosshat_fx、bishop_targeting_fx、itemmimic_revealed_shadow、positronbeam_back、stalker_shield1、toadstool_cap_absorbfx。
2. 余下 8 页：abigailsummonfx、abysspillar_fx、acidraindrop、diseaseflies、explode_small、grottopool_sfx、impact、ruins_cavein_obstacle_rubble_fx。

本轮修订后按 R01～R28 定向复验，核对摘要、快速判断、相关陷阱与记录同步；来源增补再按规则刷新对应基线。本报告不翻转台账的 accepted 或 PLAN 认领。

## 修复处置（2026-11-07）

R01～R28 全部采纳。R01 给 mushtree_tall_burntfx 补 `prefabs/mushtree_webbed.lua` 并 record；R02 五页分开 SetParent 与 AddChild：trap_vines 底座、boatrace 浮标、shadowrift 粒子（AddChild）、voidcloth 装备层、vault 净化雾，显式删除保留、纯父挂回收标未核验；R03 把 trap_fumarole_burn_fx 的“宿主 Remove 走组件移除”改为“RemoveComponent 才走 OnRemoveFromEntity，实体 Remove 走 OnRemoveEntity”；R04 煤炭火伤改为“每至少 14 帧按 10/秒×倍率×实际间隔”；R05 藤蔓存读改记 `data.varation` 写／`data.variation` 读的字段不配对，缺失回落 1；R06 龙舟旗改“挂点按名次、图案按选手编号”，并补 win/prize 暂缓到 prizeOver 的门禁；R07 烟花改塔位独立播放、不跟随符号；R08 浮标改“目标瓦片拒绝部署”，不推整个竞赛区；R09 虚布页改“scythe 本体有官方名，其余四个附属名无 NAMES”；R10 把 GetSkinBuild 分支限定到魔镰与暗影伞，回旋镖只清 swap_object；R11 伞 wave/dome 改为平方包络淡出并向外扩张；R12 虚布专服不建本地帧层但骨架保留实际组件；R13 撤回 marker 装备期必入世界存档的推导，补 parent==nil 门禁与 mainfunctions 来源；R14 地纹朝向改为“>2 且 variation<3 且未 HideCenter 先减 2；其余 3/4 镜像 -90°”；R15 罗盘 VAULT 分支 direction 为 nil 时保留旧动画，不统称 fail；R16 删除舞台剧演员退场连带删壳的保证；R17 帽外观分静态符号与全跟随层，fullhelm 补 owner.isplayer＋BuildHasSymbol 条件；R18 祭坛读档改为重新 SpawnPrefab 并重播 break，不再写“静止第一帧”；R19 trapdamage 改为参与出生保护危险计数，补设置／loading／天数／数量门槛；R20 荆棘有 owner 分支补 CanTarget／非 IsAlly，PVP 规则限定到玩家 owner；R21 蜂蜜减速改初始化坐标与半径固定参数，不逐 tick 重读；R22 焦地渐隐改两端 set_local，set 只在构造／加载；R23 暗影憎恶阈值改为 speed<1 不落、1≤speed<4→6、≥4→3，弹弓蜜补 k 公式、min/max 与 duration 随机；R24 作物陷阱改为“仅成熟／枯萎挖起才收获”；R25 冲锋尘土改为 4–6 delay 递减、到 0 的下一次回调才生成；R26 焦壳读档说明改为“保存时仍在燃烧的树”，不是已烧尽未清理；R27 两处 PO 锚点延长到 msgstr；R28 删除“约 1～2 秒”动画时长估计，改按动画完成／定时表达式。

验证：12/12 页面格式通过；全局导航 1,147 知识页／34 索引／0 问题；strict 643 处行引用、586 个唯一目标、92 个源文件，0 error／0 warning；12 页哈希 check 122 组 unchanged，mushtree／vault 两页 record refreshed 2；`git diff --check` 干净。未运行游戏、未改 raw、未提交、未推送。累计仍 138／158 页，余 20 页；28 项均已处置，待独立复验。

## 2026-11-07 修后独立复验：尚有局部补正

直接复核本次修订，不补做前次中断的收尾。实读 build 仍为 26335394，HEAD 未变；本批 11 页有正文改动，温蒂双层页未改。前面的初检意见、其他执行者“全部处置”的自报均保留为历史，**本节才是最新独立结论**。

**原 R01～R28 中，24 项正文事实修订通过；R06、R19、R23、R25 各余局部问题。另发现一组 P2 证据迁入缺口 R29，以及 P3 维护日期问题 R30。未发现新增 P1，本批暂不能收口。** 已通过的事实不因证据组 R29 再计一遍错误；证据齐备前也不称整批完成。

### 已通过的 24 项

| 编号 | 本次复核结论 |
|---|---|
| R01 | 蘑菇树已补 mushtree_webbed 来源，正文引用与 sources 对齐；本次哈希检查包含补录后的依赖。 |
| R02 | 五页正文已区分 SetParent 未验证回收与 AddChild 的 Lua children 回收；暗影页快速判断也已明确替身仅推事件。新通用回收结论的证据尚待 R29 迁入。 |
| R03 | 煤炭灼烧页已正确分开 RemoveComponent → OnRemoveFromEntity 与实体 Remove → 组件 OnRemoveEntity；通用实现证据待 R29。 |
| R04 | 已改成至少 14 帧、按 10／秒 × 温度倍率 × 实际间隔结算，符合组件代码。 |
| R05 | varation 写／variation 读及回落 1 的事实正确；但原存读锚点在修改中被删，待 R29 恢复。 |
| R07、R08 | 烟花已改塔当时位置独立播放；dockjammer 也已限定为目标瓦片，没有继续推广到整片竞赛区。 |
| R09～R12 | 魔镰本体官方名、两装备皮肤样板分支、伞淡出扩张、非专服主模拟本地层及专服实际骨架均已正确修订。 |
| R13～R15 | marker 增 parent==nil 存档门槛并撤回原推导；地纹折算／镜像条件和 -90°、罗盘 direction=nil 留旧状态均正确。 |
| R16、R17 | 已撤回演员必退场带走壳的保证；帽外观分静态符号与全跟随层，fullhelm 的玩家及 build 条件已同步。 |
| R18 | 已撤回祭坛读档静止第一帧，改为重调工厂播放 break；加载器证据待 R29，原生离屏行为仍未实测。 |
| R20～R22 | 荆棘补 owner 战斗门禁及玩家／非玩家 PVP 区别；蜂蜜固定初始化坐标、焦地两端 set_local 也已修正。 |
| R24 | 陷阱标题改为正向“只有成熟／枯萎时挖起才收获产物”，与裁决表一致。 |
| R26～R28 | 读档 burnt 来自保存时 IsBurning 的条件已补；两处 .po 锚点已含 msgstr；未核验的 1～2 秒估计已移除。 |

### R06 · P2 仍有旧表格：图案还被限制在名次 1～3

- 当前 [boatrace_start_flag:30](../../../wiki/reference/prefabs/boatrace_start_flag.md#L30) 仍写旗面来源为 `swapflag_1..3`；[第 31 行](../../../wiki/reference/prefabs/boatrace_start_flag.md#L31) 仍写“由生成方按名次 override 成 fx_spark_parts_01..03”。这与已修好的第 20、39 行冲突。
- [编号池](../../../raw/current/scripts/prefabs/boatrace_start.lua#L869-L874) 是 1～8，[给船分配编号](../../../raw/current/scripts/prefabs/boatrace_start.lua#L315-L326) 后，[setflag](../../../raw/current/scripts/prefabs/boatrace_start.lua#L634-L652) 分别用 position 选挂点、id 拼图案；[烟花](../../../raw/current/scripts/prefabs/boatrace_start.lua#L573-L578) 也拼 winid。
- 补正：第 30～31 行统一成 `swapflag_..id`／`fx_spark_parts_0..winid`，说明 id 是选手编号、只有挂点限前三名。无需为了这项读取动画资产或声称已核验资源全集。

### R19 · P2 方向已修正，但 always 与默认分支被错误合并

- 当前 [bramblefx_armor:92](../../../wiki/reference/prefabs/bramblefx_armor.md#L92) 把“always 或默认分支、非 loading、天数 >1、16 格危险数 >1”串成共同必要条件。
- [真实分支](../../../raw/current/scripts/components/playerspawner.lua#L109-L127)：dev 直接 false；非 loading 且 overrides 存在后，always 直接 true、never 直接 false；**仅其他设置的默认分支**才检查 cycles>1，随后“危险数门槛 **或** 四档结构／阻挡物门槛”任一成立即可。
- 补正：只需两三句话区分分支；若本页只讨论 trapdamage，可写“它参与默认分支的危险计数，达到该门槛是启用条件之一”，不必抄全出生保护机制。但不能说 always 也要求天数和危险数，更不能把危险数说成所有启用路径的必要条件。

### R23 · P2 正文公式正确，陷阱仍保留旧尺寸范围

- [burntground:58](../../../wiki/reference/prefabs/burntground.md#L58) 已正确写 k、min=k、max=k+0.2 和 duration 的随机量；[第 104 行](../../../wiki/reference/prefabs/burntground.md#L104) 却仍写弹弓尺寸“0.3～0.9”。
- [实际公式](../../../raw/current/scripts/prefabs/slingshotammo.lua#L215-L220) 的 0.3～0.9 是 k 的理论范围，不是最终随机尺寸全部范围；早期上界 k+0.2 可超过 0.9。建议第 104 行直接复用公式说明或链接正确段，避免另一份简化数字。
- 同行生成表第 58 行的蜂王 duration 还写 1.2～4 秒；[TrySpawnHoney](../../../raw/current/scripts/prefabs/beequeen.lua#L85-L89) 另加随机 [0,0.5)，第 44 行已有“普通档 1.2～4.5 秒”。将基数与实际传入时长区分，或统一到含随机量的区间。无需改正确的 shrouden 速度阈值。

### R25 · P2 间隔已改，首次初值还需明确为 0

- 当前 [attackfx_handpillow_petals:66](../../../wiki/reference/prefabs/attackfx_handpillow_petals.md#L66) 写“每次回调先递减 delay（初始 4～6）”，会把首次生成也描述成要先等一个倒数。
- 两条任务的实际初始参数都是 `{delay=0}`：[骑枪](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L1176-L1193)、[驼鹿](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22096-L22113)。首次回调即进入生成分支，**每次生成时**才重设 delay=4～6；后续间隔为已修正的 5～7 次回调。
- 补正：把“初始 4～6”改成“首次 delay=0；生成时重设为 4～6，后续递减到 0 的下一次回调再生成”。帧数仍保留源码推导、未运行测量的边界。

### R29 · P2 新增证据组：新结论未完整迁入正文证据与 sources

目前所有 **Markdown 已写出的来源链接** 都列在 sources 中；问题是部分新论断仍只靠本检视报告提供证据，正文没带入实际全局实现。机械工具不会把函数名自动识别成来源漏项。

| 当前页面位置 | 仍缺的支撑 |
|---|---|
| [trap_fumarole_burn_fx:44](../../../wiki/reference/prefabs/trap_fumarole_burn_fx.md#L44)、第 60 行 | 正文讲两个删除入口的分工，但 sources 没 entityscript.lua，引用仍只指灼烧组件。对应 [RemoveComponent](../../../raw/current/scripts/entityscript.lua#L670-L684)、[实体移除回调循环](../../../raw/current/scripts/entityscript.lua#L1748-L1757) 应随结论给出。 |
| [boatrace_start_flag:56](../../../wiki/reference/prefabs/boatrace_start_flag.md#L56)、[shadow_bishop_fx:62](../../../wiki/reference/prefabs/shadow_bishop_fx.md#L62) | AddChild 的调用点已给，但“宿主 Remove 遍历 Lua children”的通用实现还未引用，也未声明 entityscript.lua。对应 [AddChild](../../../raw/current/scripts/entityscript.lua#L921-L935) 与 [children 删除循环](../../../raw/current/scripts/entityscript.lua#L1779-L1783)。 |
| [moon_altar_break:59](../../../wiki/reference/prefabs/moon_altar_break.md#L59) | 新写 SpawnSaveRecord 重调工厂，但 sources 仍无 mainfunctions.lua，正文也无加载器锚点。对应 [SpawnSaveRecord](../../../raw/current/scripts/mainfunctions.lua#L464-L474)。 |
| [trap_fumarole_burn_fx:62](../../../wiki/reference/prefabs/trap_fumarole_burn_fx.md#L62) | varation／variation 的事实已正确，但原本指向 trap_vines 存读区间的链接被删除；该行仅剩 trap_fumarole 的另一个存读链接。恢复 [trap_vines OnSave／OnLoad](../../../raw/current/scripts/prefabs/trap_vines.lua#L189-L213) 的就近证据即可，该文件已在 sources。 |

补正只需必要锚点与实际采用的来源；新增 sources 后刷新受影响页的既有基线。若改为引用已有、已核验的通用机制章节，也可避免逐页复制通用实现；本项不要求新建机制页、不扩展 Skill。不要只加 sources 却不补正文定位，也不要为缩减依赖删除正确结论。

### R30 · P3 维护日期：11 篇实质改页仍写 2026-11-05

本次 11 篇实质修订页的 updated 全仍为 2026-11-05，正文验证记录也只有原建页审阅日期；新修订目前只见于报告自报。按现有 Ingest 收尾约定，补实际修订／核验日期与简短修订记录即可。温蒂页未改，不要求同步改日期；不把未执行的游戏验证补成已执行。

### 本次实际校验与限制

- 12 页格式全部通过；**全局 --check-indexes 本次退出 1**：1171 知识页／35 索引，只有并行新建 ranged-weapons.md、staffs-more.md 各缺叶级登记。本批 12 页没有登记错误，不代改他方页，也不以写作者此前 1147 页的通过记录声称当前全局通过。
- strict：**643 行引用、586 唯一目标、92 来源文件，0 error／0 warning、未截断，退出 0**。这不覆盖 R29 尚未写入的证据。
- 哈希：**122 组 unchanged**；changed、sources_changed、unrecorded、invalid 均 0，退出 0；本次只 check，没有 record。
- 12 次标题原文 read 均为 1 个结果、complete_selected_ranges=true、truncated=false、退出 0；读取当前 Markdown，不用 search 缓存。
- 本次修订的 diff 未改变 12 个 Lua 围栏；另与 HEAD 对比，12 段也全相同。沿用上次语法检查，本次未执行、未重建语法探针。
- 已重新通读所有改动行及必要上下文，定向回看上述门禁、编号、回调、存读、缩放／计时实现。未运行游戏，未实测引擎父挂级联。
- 本次 12 页及 98 个声明来源共 110 个输入指纹，复验前后未变；报告 139 个本地链接的文件／行区间核对通过。
- 本轮只更新本报告及维护日志；知识页、索引、基线、PLAN、台账、Skill、工具和 raw 保持本会话只读，不提交推送。累计仍为 138/158 已初检，20 页未整页独立检视；本次复验不增加覆盖数，也不自动关闭其他轮次。

### R06／R19／R23／R25 补正（2026-11-07）

R06 第 30 行改为 `swapflag_..id`，并注明 `id` 是选手编号 1～8、只有挂点 `flag1..3` 限前三名；第 31 行同步为 `fx_spark_parts_0..winid`，不再按名次。R19 把出生保护分支拆开：`always` 在非 loading 时直接 true、`never` 直接 false，只有默认分支才要求天数 >1，且“危险计数 >1”或“结构／阻挡物四档门槛”任一成立。R23 第 58 行蜂王 duration 改为基数 1.2／2／4 秒再 `+[0,0.5)`，第 104 行弹弓尺寸改为 0.3～1.1（`min=k`、`max=k+0.2`，k 从 0.9 降到 0.3）。R25 第 66 行改为 `{delay=0}` 首次立即生成、生成时再重设 4～6，后续 5～7 次回调；两处复用者同改。

二次复核（2026-11-07）：R06 第 30 行再写明 `id` 是分给各船的竞赛编号、**不是名次**，名次只决定挂点 `flag1..3`（仅前三名有挂点）；R19 补一句两个覆盖分支只在非 loading 时生效、loading 或 overrides 缺失时回落 false；R23 蜂王尺寸改为按档分列 0.5–0.8／0.5–1.1／1–1.3（duration 基数仍 1.2／2／4 秒，再 `+[0,0.5)`），第 104 行同步；R25 明写首次延迟 0、4–6 是生成后重设值，不是首次延迟。

验证：四页 4/4 格式通过；strict 230 处行引用、206 个唯一目标、41 个源文件，0 error／0 warning；50 组来源 unchanged；`git diff --check` 干净。全局导航 1,172 知识页／35 索引，仅并行 `misc-equipment-2.md`、`ranged-weapons.md`、`staffs-more.md` 未登记，与本批无关。未运行游戏、未改 raw、未提交、未推送。累计仍 138／158 页，余 20 页；本批 R29（证据迁入）与 R30（维护日期）仍待处理，暂不收口。

## R29／R30 处置（2026-11-07）

- R29 证据迁入：trap_fumarole_burn_fx 第 44、60 行补 `entityscript.lua` 的 `RemoveComponent`（L670-L684）与实体 Remove 组件回调（L1748-L1757）锚点，第 62 行补 `trap_vines.lua` 的 OnSave／OnLoad（L189-L213）；boatrace_start_flag 第 56 行、shadow_bishop_fx 第 62 行补 `AddChild`（L921-L935）与 Lua `children` 循环（L1779-L1783）；moon_altar_break 第 59 行补 `mainfunctions.lua` 的 `SpawnSaveRecord`（L464-L474）。四页 sources 分别补 `entityscript.lua`／`mainfunctions.lua` 并 record。
- R30 维护日期：trap、boatrace、shadow_bishop、vault、hats、mushtree、moon_altar、attackfx、bramble、burntground、voidcloth 共 11 页 `updated` 改 `2026-11-07`，验证记录各补一条本轮修订摘要；wendy_gravestone_rebirth_fx 未实质修改，保持 `2026-11-05`。
- 验证：11 页 `validate_page` 11/11；strict 622 处行引用／565 唯一目标／89 源文件，0 error／0 warning；四页 record 后 88 组来源 unchanged、0 changed（12 页范围）；`git diff --check` 干净。未运行游戏、未改 raw、未提交推送。R29／R30 收口，累计覆盖仍 158／158。

