# 明细：状态图（stategraph.lua 与 stategraphs/*）

证据旧快照 `raw/archive/build-26148036/scripts`，新快照 `raw/current/scripts`。SB-1～SB-9 逐文件；SB-10 为 49 个生物 SG 统一接入击退框架的汇总（附表逐文件）。

| 编号 | 变化 | 影响接入／条件 | 证据位置 | 核验状态 |
|---|---|---|---|---|
| SB-1 | **`commonstates` 新增击退框架**：① `CommonHandlers.OnKnockback()`——"knockback" 事件 handler 工厂：dead／`nointerrupt`／`noknockback` 状态标签／`sg.mem.noknockback` 任一命中即忽略；有 `knockbacklanded` 状态且（`data.forcelanded` 或 swimming 或 `heavybody` 标签或装备 `heavyarmor`）走落地击退，否则有 `knockback` 状态且非游泳走普通击退。② `CommonStates.AddKnockbackStates(states, timelines, anims, fns, speedmult)`——添加 `knockback`＋`knockbacklanded` 两个 State：从 knocker 反向推离（MotorVelOverride，速度按距离衰减 10×k，k∈[-1,-0.7]），非 companion 且不在围墙 VRS 内临时清 LIMITS 碰撞；`knockback` 态时间线 12 帧落体声/溺水检查/两栖检查、14 帧解 nofreeze/noelectrocute、32 帧 WakeUp＋caninterrupt、35 帧解 busy；入水（ms_enterwater，见 C-9）按 sleeping 转 sleep/idle；fns 钩子 knockback_onenter/onanimover/onexit 与 knockbacklanded_*。③ `onsleep` 睡眠触发守卫：旧"死亡或 electrocute 中不可睡"改"未死亡且（无 electrocute/knockback 标签或态有 caninterrupt）"——击退中不可入睡，除非已可打断。④ `AddAmphibiousCreatureHopStates` 的 `wantstomoveforward` 字段改 `WantsToMoveForward()` 方法（等价重构）。 | 想接官方击退的 SG：`CommonStates.AddKnockbackStates(states)` ＋ events 挂 `CommonHandlers.OnKnockback()`；`sg.mem.noknockback = true` 可实例级免疫（P-17～P-19）。爆炸方传 knockbackdata（C-7）。守卫③影响所有共享 onsleep 的生物——击退期间 Sleep 触发被抑制。 | 旧:stategraphs/commonstates.lua:27-31（onsleep）、:1065-1069（hop）；新:commonstates.lua:27-33、:3105-3407（knockback 框架） | 已核验 |
| SB-2 | `stategraph.lua`（框架文件）：`HasStateTag`/`HasAnyStateTag` 在 `BRANCH == "dev"` 下包参数断言（单参非 table／多参或 table），与 R-5 同模式。 | release 行为不变；dev 分支防误用显性化。 | 新:stategraph.lua:641-656 | 已核验 |
| SB-3 | **`SGwilson`**：① `ConfigureRunState` 新增 `statemem.normalflyingbroom`（EquipHasTag "flyingbroom"）；run/run_start 转入新 `run_flyingbroom_start`/`run_flyingbroom` 两态（flyingonbroom 标签、autopredict）：进入时 `ConfigureWitchBroomFlying(inst, true)` = 加 `notraptrigger` 标签＋`locomotor:EnableGroundSpeedMultiplier(false)`，退出还原；退出条件：卸下飞行扫帚、暴风/云雾中失去护目镜视野、沙尘暴满级、瘴气≥1、谨慎行走。`run_pst` 按 lasttags `flyingonbroom` 播 run_witchbroom；galloping 在 run_pst 补马蹄音。② `ActionHandler(ACTIONS.SWEEP, "doswipeaction")`。③ **`lyingdown` 状态标签族**：wakeup（16/70 帧移除）、bedroll（8+60 帧加）、knockout（32/54 帧按 dismounting 加）、washed_ashore（70 帧移除）、abyss_drop（14 帧加）、abyss_drop_pst/knockback_pst（13 帧移除）、bucked（11 帧加、33 帧移除）各态补齐——标记"玩家倒地"；`bucked` onenter 据此选 `knockback_low`（倒地起手）或 `knockback_high`（站立高抛），starthigh 优先（SB-8 喂入）。 | 玩家侧接入点：自制装备想改跑态抄 normalflyingbroom 判定链；bucked 动画选择依赖 lyingdown 标签准确性——覆盖这些态的 mod 记得维护该标签。SWEEP 玩家侧走 doswipeaction 既有动作态。 | 新:stategraphs/SGwilson.lua:446（ConfigureRunState broom 判定）、:890-901（ConfigureWitchBroomFlying）、:1799-1800（SWEEP handler）、:2660-2700（wakeup）、:14291-14300/:14434-14439（run 分流）、:14345-14351（bucked 动画选择）、:28816-28939（run_flyingbroom 两态） | 已核验 |
| SB-4 | `SGwilson_client`：与 SB-3 同构的客户端预测版——normalflyingbroom 判定、run_flyingbroom_start/run_flyingbroom 两态（无 unequip 事件、无 notraptrigger，只 EnableGroundSpeedMultiplier）、run_pst run_witchbroom、galloping 马蹄音补齐、SWEEP ActionHandler。 | 客户端预测与主 SG 保持一致；客户端 mod 改跑态需两侧同步改。 | 新:stategraphs/SGwilson_client.lua:164-166（broom 判定）、:261-263、:1028-1029（SWEEP）、:1214-1219/:1352-1357（run 分流）、:8712-8824（broom 两态） | 已核验 |
| SB-5 | `SGmole`：新增 "knockback" 事件 handler——非 forcelanded 且未死未 noattack 时转 `stunned` 态（鼹鼠无击退态，改为眩晕表达）。 | 鼹鼠被爆炸击退＝进眩晕；模组若依赖鼹鼠不吃击退需自行处理。 | 新:stategraphs/SGmole.lua:63-67 | 已核验 |
| SB-6 | `SGcharlie_boss`：① `ShouldVineCounter` 距离改 `(IsRangedWeapon(data.weapon or data.attacker) and 4 or 5) + attacker 半径`——远程/近战区别对待；② 反斩触发半径 `TUNING.CHARLIE_BOSS_COUNTERATTACK_RANGE + inst 半径` → `+ target 半径`（旧版用 Boss 自身半径，目标半径普遍更小——修正后实际范围随 R-22 的 8→14 显著扩大）。 | Charlie boss 反斩行为变化：远程武器贴身阈值 4、近战 5；范围计算基准修正。Boss 战 mod/攻略数值需同步。 | 旧:stategraphs/SGcharlie_boss.lua:31-36、:587-592；新同函数 | 已核验 |
| SB-7 | `SGshrouden`：两处反斩触发半径同 SB-6②（inst 半径 → target 半径）；配合 R-22 的 16→20。 | 同上。 | 旧:stategraphs/SGshrouden.lua:818-822、:843-849；新同函数 | 已核验 |
| SB-8 | `SGshadowthrall_horns` `DoSpitOut`：吐出事件数据新增 `starthigh = true`。 | 吞吐事件喂给玩家 bucked 态（SB-3③）：被影怪巨口吐出的玩家起手高抛动画。自定义"spitout"消费者不认识新字段则忽略（向后兼容）。 | 新:stategraphs/SGshadowthrall_horns.lua:188-197 | 已核验 |
| SB-9 | `SGcarnival_crowkid`：注释内 `wantstomoveforward` → `WantsToMoveForward()`（注释行）。 | 无实质变化（注释同步）。 | 新:stategraphs/SGcarnival_crowkid.lua:63 | 已核验 |
| SB-10 | **49 个生物 SG 接入击退框架**：各 SG 在 events 挂 `CommonHandlers.OnKnockback()` 并 `CommonStates.AddKnockbackStates(states, ...)`；另有若干 handler 的忙碌判断补 `HasAnyStateTag(..., "knockback")`。无框架行为之外的独立玩法变化（各 SG 配置差异见附表；SGpigelite/SGchester 删除各自旧版自制 knockback/knockbacklanded 态，由公共实现替代）。 | 生物击退表现逐物种配置：猪族 scream＋"smacked" 动画、飞行生物降/升、逃飞/惊吓衔接等。mod 覆写这些 SG 的击退相关态或监听其 timeline 事件的需对表自查；模组生物可直接复用 AddKnockbackStates。 | 各文件：AddKnockbackStates 调用段（统一在 SG 构造尾部） | 已核验 |

## SB-10 附表：49 文件接入配置

| 文件 | 配置要点 |
|---|---|
| SGMandrake | 无参 |
| SGbunnyman | 无参 |
| SGcarrat | 无参 |
| SGcatcoon | 无参 |
| SGfrog | 无参 |
| SGchest_mimic | 无参 |
| SGfruitdragon | 无参 |
| SGfruitfly | 无参 |
| SGglommer | 无参 |
| SGhound | 无参 |
| SGlightcrab | 无参 |
| SGbee | 无参 |
| SGlightflier | 无参 |
| SGmolebat | 无参 |
| SGmonkey | 无参 |
| SGmosquito | 无参 |
| SGmossling | 无参 |
| SGperd | 无参 |
| SGrabbit | 无参 |
| SGrabbitking | 无参 |
| SGrabbitking_bunnyman | 无参 |
| SGsalty_dog | 无参 |
| SGslurper | 无参 |
| SGsmallbird | 无参 |
| SGspider | 无参 |
| SGspider_water | 无参 |
| SGsquid | 无参 |
| SGwalrus | 无参 |
| SGwobsterland | 无参 |
| SGpowdermonkey | 无参 |
| SGbat | timelines：14 帧落地、33 帧起飞；fns：onenter SwitchToNoFaced、onexit TryRestoreSixFaced＋起飞 |
| SGbeeguard | timelines：13 落地、25 起嗡、28 起飞；fns：onenter 停嗡、onexit 起嗡 |
| SGbird | timelines：32 帧空事件；fns：两态 onanimover → _flyawayonanimover（受扰逃飞） |
| SGbuzzard | timelines：32 帧燃烧且该惊吓时转 distress_pre；fns：onanimover 自定义 |
| SGchester | fns：onenter/onexit 自定义（_knockbackonenter/_knockbackonexit，两态共用） |
| SGcrabking_mob | fns：knockbacklanded_onenter/onexit 隐/显 FX 符号 |
| SGeyeofterror_mini | timelines：14 降、28 升（飞行）；fns：onexit 起飞 |
| SGgingerbreadpig | timelines：35 帧受惊转 scared、landed 12 帧；timelines 与默认合并 |
| SGgrassgekko | timelines：33/10 帧转 scare；fns：两态 onanimover="scare" |
| SGpenguin | timelines：35/8 帧按 sg.mem.flyaway 转 flyaway |
| SGpig | timelines：3 帧尖叫；anims：knockback="smacked" |
| SGpigelite | 同 SGpig ＋ fns：onenter 处理 data.propsmashed；**删除旧自制 knockback 态** |
| SGpigelitefighter | 同 SGpig（尖叫＋smacked） |
| SGpolly_rogers | timelines：32 帧 flyaway；fns：onenter 自定义 |
| SGslurtle | timelines：32/8 帧按 sg.mem.dissolving 转 dissolving_pre |
| SGmerm | 单行：anims knockback="smacked"；另 attacked 守卫补 knockback 标签豁免 |
| SGmoonpig | 单行 smacked；attacked 守卫补 knockback；新增态带 "noknockback" 标签 |
| SGprimemate | 单行 smacked |
| SGwerepig | 单行 smacked；attacked 守卫补 knockback＋存活检查 |
