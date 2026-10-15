# FX 第 14 轮独立检视：外围 U2～U6

检视日期：2026-11-07。源码快照：`steam-buildid:26335394`（实读 `raw/current/BUILD.txt`）；K HEAD：`97afb4e9e6ca05390558769926f1f9c8598d0f70`。

**当前结论：2026-11-07 四项补正独立复验通过，R01～R22 全部完成，第 14 轮可收口，详见文末。** 初检 P1×0、P2×22、P3×0；来源漏项与父挂清理证据分别合并计数，下文保留初检历史。全局导航另有两篇并行在建装备页未登记，不据此称全库导航通过。只维护检视报告和日志，不改知识页、索引、来源基线、PLAN、台账、Skill、工具或 raw；不 record、不提交推送，保留并行装备扩库的增量。

## 范围与口径

14 页共 **1709 个物理行／291,544 字节，56 个主要覆盖注册名**；声明 **161 组 sources／120 个不同文件**。逐页通读，核对注册工厂、必要消费者与参数传递、生成点、端、父子／跟随、清理、数值和案例。没有把 120 份来源全部全文重审；未运行游戏。

| 页面 | 覆盖注册数 | 行／字节 | 独立检视结论 |
|---|---:|---:|---|
| [fossilspike_base](../../../wiki/reference/prefabs/fossilspike_base.md) | 2 | 109／16,473 | R02、R03 |
| [lunarrift_terraformer_visual](../../../wiki/reference/prefabs/lunarrift_terraformer_visual.md) | 2 | 120／19,174 | R02 |
| [shatter](../../../wiki/reference/prefabs/shatter.md) | 8 | 149／31,736 | R01、R04、R05 |
| [cavein_dust_low](../../../wiki/reference/prefabs/cavein_dust_low.md) | 9 | 155／27,470 | R06 |
| [slingshotammo_purebrilliance_debuff_fx](../../../wiki/reference/prefabs/slingshotammo_purebrilliance_debuff_fx.md) | 3 | 126／21,804 | R07～R09 |
| [spear_gungnir_lungefx](../../../wiki/reference/prefabs/spear_gungnir_lungefx.md) | 3 | 123／19,952 | R01、R02、R10、R11 |
| [terrariumchest_fx](../../../wiki/reference/prefabs/terrariumchest_fx.md) | 2 | 118／17,579 | R01、R02 |
| [winona_battery_high_shatterfx](../../../wiki/reference/prefabs/winona_battery_high_shatterfx.md) | 5 | 129／24,195 | R01、R02、R12 |
| [forcefieldfx](../../../wiki/reference/prefabs/forcefieldfx.md) | 1 | 106／12,114 | R01、R21 |
| [wx78_big_spark](../../../wiki/reference/prefabs/wx78_big_spark.md) | 4 | 116／20,842 | R01、R02、R13～R16 |
| [wagboss_beam_fx](../../../wiki/reference/prefabs/wagboss_beam_fx.md) | 4 | 109／20,752 | R02、R17 |
| [wortox_soulecho_buff_fx](../../../wiki/reference/prefabs/wortox_soulecho_buff_fx.md) | 6 | 116／20,228 | R01、R02、R18 |
| [wurt_shadow_merm_planar_fx](../../../wiki/reference/prefabs/wurt_shadow_merm_planar_fx.md) | 5 | 119／23,327 | R01、R22 |
| [waterplant_pollen_fx](../../../wiki/reference/prefabs/waterplant_pollen_fx.md) | 2 | 114／15,898 | R19、R20 |

U2～U6 共 15 个新页中的 `oceantree_ripples_short` 已在第 13 轮整页核验，本轮不重复。第 05 轮对 `cavein_dust_low` 只处理跨页别名冲突，明确未审其 9 注册，因此这次计为首次整页检视。正文辨析用的宿主／debuff 别名不额外计入 56；已保留 `cavein_debris` 的 sinkhole 主要归属。

## P2 问题

### R01｜8 页的正文源码引用未全部进入 sources

逐页比对实际 Markdown 本地引用与 frontmatter，发现 **8 页、9 个页—来源漏项**：

| 页面及 sources 位置 | 正文已引用、但未声明的文件 | 正文例子 |
|---|---|---|
| [shatter:11](../../../wiki/reference/prefabs/shatter.md#L11) | `prefablist.lua`、`widgets/werebadge.lua` | 第 58、130 行 |
| [spear_gungnir_lungefx:10](../../../wiki/reference/prefabs/spear_gungnir_lungefx.md#L10) | `prefabs/lavaarena.lua` | 第 32 行 |
| [terrariumchest_fx:10](../../../wiki/reference/prefabs/terrariumchest_fx.md#L10) | `worldsettings_overrides.lua` | 第 39 行 |
| [winona_battery_high_shatterfx:10](../../../wiki/reference/prefabs/winona_battery_high_shatterfx.md#L10) | `mainfunctions.lua` | 第 82 行 |
| [forcefieldfx:10](../../../wiki/reference/prefabs/forcefieldfx.md#L10) | `fx.lua` | 第 51 行 |
| [wx78_big_spark:10](../../../wiki/reference/prefabs/wx78_big_spark.md#L10) | `tuning.lua` | 第 54 行 |
| [wortox_soulecho_buff_fx:10](../../../wiki/reference/prefabs/wortox_soulecho_buff_fx.md#L10) | `fx.lua` | 第 66 行 |
| [wurt_shadow_merm_planar_fx:10](../../../wiki/reference/prefabs/wurt_shadow_merm_planar_fx.md#L10) | `fx.lua` | 第 62 行 |

其中 wortox 声明的是 `prefabs/fx.lua`，不能代替根目录数据表 `fx.lua`；forcefield／wurt 则未声明这两份文件中的任何一份。建议核验后补实际来源；无用声明按实际阅读范围处理，修改来源清单的页再按现规则定向刷新基线。本次哈希 check 通过只证明既有声明未变，不证明来源完整；不批量 record 历史页。

### R02｜8 页把引擎父挂写成已证明的删除级联

- 位置：fossil [6](../../../wiki/reference/prefabs/fossilspike_base.md#L6)、22、65～72；lunarrift [22](../../../wiki/reference/prefabs/lunarrift_terraformer_visual.md#L22)、75、103；spear [77](../../../wiki/reference/prefabs/spear_gungnir_lungefx.md#L77)、104；terrarium [67](../../../wiki/reference/prefabs/terrariumchest_fx.md#L67)、84；winona [81](../../../wiki/reference/prefabs/winona_battery_high_shatterfx.md#L81)、82、112；wx78 [73](../../../wiki/reference/prefabs/wx78_big_spark.md#L73)、75；wagboss [66](../../../wiki/reference/prefabs/wagboss_beam_fx.md#L66)～68；wortox [73](../../../wiki/reference/prefabs/wortox_soulecho_buff_fx.md#L73)、75、76。
- 依据：已读工厂只证明相关实体的 `entity:SetParent` 关系，例如 [fossil 父挂](../../../raw/current/scripts/prefabs/fossil_spike.lua#L157-L158)、[天灯影层](../../../raw/current/scripts/prefabs/floatinglantern.lua#L415-L417)、[强抢袋本地层](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L269-L280)、[beam 本地环](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L453-L457)。[EntityScript:Remove](../../../raw/current/scripts/entityscript.lua#L1779-L1783) 明文删除的是 Lua `children` 表，不能拿它证明所有引擎父挂实体必然执行同一回收链。
- 建议：保留已证实的显式 Remove／ErodeAway／事件路径；仅靠 SetParent 的部分注明引擎层回收尚未核验。自定义宿主示例应明确由谁保存引用和清理，不能把“本体消失即整层回收”作为已验推荐。这里是**证据边界问题，不是已证明泄漏**；也不要求为此补不存在的引擎实现。wx78 扫描束实际有 `AddChild` 登记，不纳入该项的纯 SetParent 推断。

### R03｜化石基座的“失效时直接删除”与原分支不符

- 位置：[fossilspike_base:65](../../../wiki/reference/prefabs/fossilspike_base.md#L65)、66。
- 依据：[一代 KillSpike](../../../raw/current/scripts/prefabs/fossil_spike.lua#L126-L148) 在 `basefx ~= nil` 但 `not basefx:IsValid()` 时调用 `OnKill`，仍有灰烬与后续侵蚀链；只有引用为 nil 才直接 Remove。[二代](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L109-L129) 只判非 nil，不检查基座 IsValid。
- 建议：分别写“引用 nil”“引用非 nil 但实体失效”两种条件，删除两代均“失效则立即 Remove”的概括；不要连带断言未读取的动画必然未播完。

### R04｜南瓜碎块 PRNG 种子少了坐标取整偏移

- 位置：[shatter:49](../../../wiki/reference/prefabs/shatter.md#L49)。
- 依据：[fx_TossChunks](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L262-L270) 是 `floor(x + 0.5) * floor(z + 0.5) * networkid`，不是文中的 `floor(x) * floor(z) * 网络ID`。
- 建议：照实际公式记录；小数坐标与负坐标可能产生不同种子，不能省略偏移后仍称同一确定性序列。

### R05｜brokentool 把成功复活写成失败，把 hit 动画写成状态名

- 位置：[shatter:63](../../../wiki/reference/prefabs/shatter.md#L63)、99。
- 依据：[Revive_OnHaunt](../../../raw/current/scripts/prefabs/pocketwatch.lua#L135-L143) 仅在旺达作祟且 CastSpell **成功**时生成 brokentool 并删除怀表，失败只是 Launch；三处 SG 的状态名均为 `toolbroke`，其中播的动画才叫 `hit`（[wilson](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15045-L15057)、[hermitcrab](../../../raw/current/scripts/stategraphs/SGhermitcrab.lua#L3691-L3703)、[possessedbody](../../../raw/current/scripts/stategraphs/SGwx78_possessedbody.lua#L2349-L2361)）。
- 建议：生成表和玩法辨析同步改为成功后的怀表损毁，状态定位改 toolbroke；不要保留“作祟失败／SG hit 状态”的检索误导。

### R06｜scraphat 并非快照外工厂，漏掉 MakeHat 动态注册

- 位置：[cavein_dust_low:104](../../../wiki/reference/prefabs/cavein_dust_low.md#L104)、139、145～146。
- 依据：[MakeHat 的命名](../../../raw/current/scripts/prefabs/hats.lua#L16-L20) 为 `name.."hat"`；[fns.scrap](../../../raw/current/scripts/prefabs/hats.lua#L4721-L4741) 实际构造 armor／waterproofer；[scrap 分支](../../../raw/current/scripts/prefabs/hats.lua#L6936-L6939) 选择该工厂，[通用返回](../../../raw/current/scripts/prefabs/hats.lua#L6984-L6986) 注册 prefabname，[MakeHat("scrap")](../../../raw/current/scripts/prefabs/hats.lua#L7935-L7936) 得到 scraphat。
- 建议：删掉“工厂缺失／事件包外／不能核验装备属性”的负向结论，补实际入口。FX 的 6～20% 掉落结算仍由 [daywalker2_items](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L10-L17) 支持，保留；无需扩写完整帽子 Wiki。补来源后定向刷新本页基线。

### R07｜恐惧弹 FX 的实际攻击延迟为 25 帧，回收也不是 animover 监听

- 位置：[slingshotammo_purebrilliance_debuff_fx:37](../../../wiki/reference/prefabs/slingshotammo_purebrilliance_debuff_fx.md#L37)、60、77。
- 依据：[调用](../../../raw/current/scripts/prefabs/slingshotammo.lua#L477-L489) 传 `Restart(attacker,target,variation,data.pool,first)`，而 [定义](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L420-L435) 只接 `attacker,target,variation,quick`：quick 实收非 nil 的池表（空表在 Lua 中也为真），first 成为多余实参。原版这条链每次都跳到第 3 帧，攻击任务是 `(28-3)*FRAMES`。回收函数名虽叫 HorrorFuel_AnimOver，实际由第 435 行按动画完整长度排 **DoTaskInTime**，不是注册 animover 事件。
- 建议：区分函数可选参数语义与原版调用实效；写清 25 帧攻击、按完整动画长度计时回池／Remove。只修知识，不替游戏源码纠错。

### R08｜纯辉双层标记的体形不是统一由标签决定，跟符号也有门禁

- 位置：[slingshotammo_purebrilliance_debuff_fx:36](../../../wiki/reference/prefabs/slingshotammo_purebrilliance_debuff_fx.md#L36)、40。
- 依据：[GetBestSymbolAndSize](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L272-L294) 优先返回 burnable 的 follow＋fxlevel，其次 freezable 的 follow＋fxlevel−1；只有 combat 回退分支按 smallcreature／largecreature／epic 取档。[AttachTo](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L308-L325) 仅在有符号且 GetSymbolPosition 成功时 FollowSymbol；有符号时才设置 clamp 后的 size 与前后动画。
- 建议：把“体形档位按标签”限定到 combat 回退分支，保留前两条真实数值来源和符号有效性门禁，避免给换符号／尺寸的读者错误配置依据。

### R09｜纯辉 FX 双层壳没有页面所称的自定义 netvar

- 位置：[slingshotammo_purebrilliance_debuff_fx:109](../../../wiki/reference/prefabs/slingshotammo_purebrilliance_debuff_fx.md#L109)。
- 依据：[PureBrillianceMarkFxFn](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L334-L369) 建 Network／AnimState／Follower，主模拟 AttachTo 控制父挂、符号与动画，未定义本页所谓“双层壳的 netvar”。
- 建议：改为通过联网表现实体的实际接口控制；若模组另需自定义同步再自行设计。不要把 Network 或引擎动画同步统称为已有 Lua netvar。

### R10｜奔雷突刺光等待动画和灯光都完成，不是“两路取先”

- 位置：[spear_gungnir_lungefx:20](../../../wiki/reference/prefabs/spear_gungnir_lungefx.md#L20)、76；第 36 行较详细的协同描述反而正确。
- 依据：[FX_OnUpdate／FX_OnAnimOver](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L578-L598)：动画先结束只置 killfx，等待灯光降零；灯光先结束只取消 task，等待 animover。因此为两条件共同完成，不是任一先到即删。另外 [gungnir 工厂](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L135-L145) 只有动画长度＋2 帧的定时任务，没有 animover 删除监听。
- 建议：快速判断与清理表同步分开两种协议，消除同页正反说法。

### R11｜快照外熔炉消费方只能作为候选，不能写成已证实

- 位置：[spear_gungnir_lungefx:30](../../../wiki/reference/prefabs/spear_gungnir_lungefx.md#L30)、57、102。
- 依据：[本体装配入口](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L91-L98) 证明事件包提供 master_postinit，但该包不在本快照，不能据此证明里面必定生成 lungefx。全 scripts 的完整名／gungnir 前缀／_lungefx 后缀候选已复查，未找到可见生成调用。
- 建议：沿用第 22 行的保守口径：“事件包可能是消费方，具体生成调用未核验”；保留注册与 FastForward 接口的正向证据，不判死注册，也不把包内调用写成事实。

### R12｜新建 FX 上的动画比较不是跨调用“同帧判重”

- 位置：[winona_battery_high_shatterfx:113](../../../wiki/reference/prefabs/winona_battery_high_shatterfx.md#L113)。
- 依据：[LoseGem](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L149-L162) 每次先 SpawnPrefab 一枚新 FX，再和其默认 redgem_shatter 比较；相同就避免重启**这枚新实体的默认动画**，不同则切对应宝石动画。没有共享实体／帧标记可合并同帧多次调用。
- 建议：删“避免同帧重播”，改为“所需动画与新壳默认动画相同则无需重播”；不要暗示该函数会限制同帧 FX 数量。

### R13｜WX 灯点半径漏了 i＋2 偏移

- 位置：[wx78_big_spark:31](../../../wiki/reference/prefabs/wx78_big_spark.md#L31)、55。
- 依据：[CreateLight](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L7-L36) 先把传入序号 i 加 2，保存在 inst.i；实际第 j 个灯点半径为 `sqrt(light_radius*0.08)*(j+2)`。[位置偏移](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L102-L123) 才使用未偏移的循环序号 j。
- 建议：区分灯半径与位置计算的序号；若半径式仍用 i，须明示 i＝灯点序号＋2，不能让读者把首个灯按 i＝1 计算。

### R14｜WX 的 floating 停止阈值分支不能当成已启用行为

- 位置：[wx78_big_spark:98](../../../wiki/reference/prefabs/wx78_big_spark.md#L98)。
- 依据：[LightBeam_GetTargetRotation](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L48-L84) 第 60 行查的是 **beam 自身 inst.sg**，不是 owner.sg；[beam 工厂与挂接](../../../raw/current/scripts/prefabs/wx78_lightbeam.lua#L125-L190) 没装 StateGraph。
- 建议：保留实际的预测位置／2 秒回退条件；将 floating 低阈值写成“代码中有此门禁，但原版 beam 未安装 SG，未证实可达”，不能直接称 floating 状态就会降阈值。

### R15｜扫描目标死亡不等于实体已经移除，风险推演缺实际链

- 位置：[wx78_big_spark:99](../../../wiki/reference/prefabs/wx78_big_spark.md#L99)。
- 依据：[OnUpdateScanCheck](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L419-L443) 对 health:IsDead 目标先 StopScanFX，再 OnScanFailed；[AddChild](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L553-L563) 的删除链针对实体 Remove，不能把死亡直接替换成 Remove。页面据此断言“死亡→级联删 FX→再调用已删 FX 的 goAway”，证据不足。
- 建议：写明死亡分支的真实调用顺序；若保留已删除引用风险，另找实体移除时序的实证或列待核验，不把推演当原版必然表现。不据此反向保证所有目标移除路径都安全。

### R16｜WX 大火花原版生成链没有父挂或持续跟随目标

- 位置：[wx78_big_spark:50](../../../wiki/reference/prefabs/wx78_big_spark.md#L50)。
- 依据：[AlignToTarget](../../../raw/current/scripts/prefabs/wx78_big_spark.lua#L116-L132) 只取一次目标坐标定位壳，不 SetParent；[StartFX](../../../raw/current/scripts/prefabs/wx78_big_spark.lua#L49-L53) 仅复制壳已有父级（若有）与当时变换。已核三个 SpawnBigSpark 包装均只调用 AlignToTarget，没有补父挂。
- 建议：明确原版链是在目标当时脚底生成世界位置爆点；“壳已有父级时本地体复用该父级”是条件能力，不是原版已跟随宿主。不要从函数名 Align 推断持续对位。

### R17｜提取器的共享 erode 路径被漏审，导致清理和灯光结论过强

- 位置：[wagboss_beam_fx:69](../../../wiki/reference/prefabs/wagboss_beam_fx.md#L69)、93。
- 依据：[contained 工厂](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1716-L1724) 把共享 erode 方法装给提取器；[donpcerode](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L579-L591) 将 time／erodein／remove 同步传给 _device；[共享实现](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L306-L365) 包含 Light 启停及 removewhendone 条件下的 Remove。[SG 的 ERODEOUT_DATA](../../../raw/current/scripts/stategraphs/SGwagstaff_npc.lua#L63-L68) 设置 remove=true，[capture_emote](../../../raw/current/scripts/stategraphs/SGwagstaff_npc.lua#L536-L545) 实际推送它。
- 建议：不能继续写“无显式 Remove／只由续期重建”及“只有 docollect 才开灯”；补共享 erode 的条件、调用方与明确未运行范围。这里只证明存在该分支，不保证所有时间参数／路径均命中完成条件；不把计时数值边界顺手改成已测结论。

### R18｜灵魂粒子发射率单位与“仅三项差异”均不准确

- 位置：[wortox_soulecho_buff_fx:30](../../../wiki/reference/prefabs/wortox_soulecho_buff_fx.md#L30)、95。
- 依据：[soulecho 发射循环](../../../raw/current/scripts/prefabs/wortox.lua#L969-L990) 使用 `3*tick_time` 到 `30*tick_time`，配置对应约 **3～30 粒／秒**，不是“每 tick 3～30 个”。[soulecho 粒子](../../../raw/current/scripts/prefabs/wortox.lua#L915-L928) 与 [overloading 粒子](../../../raw/current/scripts/prefabs/wortox.lua#L1035-L1047) 的出生坐标和初速度也不同（例如横向速度系数 0.02 对 1，位置 x/z 系数 2 对 0.2，y＝0 对随机高度），并非只差尺度包络、寿命、Follower。
- 建议：改正单位，比较段写“共用纹理与相近骨架，另有出生分布／速度差异”；不必复制两套全量参数表，但不能保留排他性的“差异只有”。

### R19｜空中花粉云的自删只在主模拟监听，35 帧任务也不是独立保证

- 位置：[waterplant_pollen_fx:62](../../../wiki/reference/prefabs/waterplant_pollen_fx.md#L62)、99。
- 依据：[front_fn](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L57-L68) 在客户端早退之后才安排 35 帧生成任务与 animover 删除监听，所以“双方各自动画结束各自删”错误。[EntityScript:Remove](../../../raw/current/scripts/entityscript.lua#L1739-L1746) 会取消待执行任务；若把动画改短、提前删云，不能仍保证 35 帧后生成海面云／放鱼。
- 建议：按主模拟删除＋联网表现说明；把第二段生成前提写为“实体存活且该任务未被取消”，不要把同一实体上的延迟任务当独立兜底。

### R20｜persists=false 只挡即时补位，不会停周期补块

- 位置：[waterplant_pollen_fx:68](../../../wiki/reference/prefabs/waterplant_pollen_fx.md#L68)～69、91；结合第 47、62 行。
- 依据：[on_piece_removed](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L86-L98) 的即时补位才判断 persists；[spawn_chum_piece](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L100-L118) 只判断数量与海面，[1 秒周期任务](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L180-L188) 并未在 disperse 时取消。因此开始淡出后的 2 秒里仍可能周期补块，直到实体删除。
- 建议：原版事实写清两条补位入口。示例若意图“消散即停止所有生成”，需让周期入口也检查消散标志或持有并取消任务；“置 persists=false 或取消任务任选其一即可停补位”不成立。仍要区分停止新生成和清理现存子块。

### R21｜kill_fx 延迟 0.6 秒删除，不能据此称重入也绝无双层

- 位置：[forcefieldfx:90](../../../wiki/reference/prefabs/forcefieldfx.md#L90)。
- 依据：[tryproc](../../../raw/current/scripts/prefabs/hats.lua#L478-L483) 原版被 _task 门禁挡住，正常 proc 期间不会再次 proc；若讨论直接重入 [ruinshat_proc](../../../raw/current/scripts/prefabs/hats.lua#L455-L460)，它调用旧 FX 的 kill_fx 后立刻造新 FX，而 [kill_fx](../../../raw/current/scripts/prefabs/forcefieldfx.lua#L35-L40) 是 0.6 秒后才 Remove。
- 建议：说明正常调用的门禁；内部实现参照只能说“旧层进入退场再创建新层”，不要写“同一时间最多一层／不会叠两层”。这不表示原版正常 proc 会重复触发，也不把 local proc 推荐成公开 API。

### R22｜沃特页把 debuff 计时自终止与 FX 的自删混写

- 位置：[wurt_shadow_merm_planar_fx:22](../../../wiki/reference/prefabs/wurt_shadow_merm_planar_fx.md#L22)、81。
- 依据：[OnAttached](../../../raw/current/scripts/prefabs/wurt.lua#L712-L719) 启动 expire，[bufffn](../../../raw/current/scripts/prefabs/wurt.lua#L804-L811) 在 timerdone 调 debuff:Stop，[OnDetached](../../../raw/current/scripts/prefabs/wurt.lua#L749-L779) 最后 Remove debuff。因此“debuff 实体无自删”不准确；[月影 FX 工厂](../../../raw/current/scripts/prefabs/wurt.lua#L846-L872) 明确登记 animover 自删，也与案例中“两者都不要在 FX 工厂里自删”的正句冲突。
- 建议：分开三类：debuff 有计时到点／死亡终止；暗影持续层由 detach 播 pst 后删；月影入退场各自一次性自删。保留现有正确的月影差异表，不用新增协议页。

## 已核对但不构成新增问题

- 56 个主要注册均进入本页正文和 aliases，跨现存 Prefab aliases 无主要归属冲突；额外宿主／描述性别名未机械当成独立 FX 覆盖。
- 全 scripts **4095 个 Lua** 反查：4093 份 UTF-8/BOM 可读，2 份按 latin-1 读取，未用 errors=ignore；最长注册名优先扫描得到 521 个候选出现（含资源名、字段名和子串，不等于生成次数）。可见字面生成与页面主要入口一致；daywalker2 动态拼名／SG 护甲表、wurt chargedfx 字段选名、freezable 的 shatter helper 链另已定向追到实际消费。
- `splash_spiderweb` 全库 6 个字面出现都已列完：本文件资源／bank／build／注册＋prefablist＋spider_web_spit 依赖；splash_spider 与 spiderweb 前缀补查未见新生成候选。`spear_gungnir_lungefx` 完整名和 _lungefx 后缀均仅两行，gungnir 全域候选 52 行逐类区分为皮肤／名称／依赖／装配入口；没有据此宣布快照外消费者已证实。
- 56 个注册的精确 NAMES 键中命中 3 个：`ALTERGUARDIAN_CONTAINED`、`SLINGSHOTAMMO_HORRORFUEL_DEBUFF_FX`、`WX78_BACKUPBODY_INVENTORY`。另完整查看正文 **46 组不同 PO 行区间**（含宿主名称、动作与检查描述），本次未发现已引用译文／键值错配。
- 大火花代理无 AnimState 的染色门禁确实存在；AlignToTarget 当场调用一次 OnUpdateFlash，当前构造随即拆更新器，不把“注册了回调”当成染色真的发生。细小的“首次更新”措辞不另计 P3。
- wurt 的 OnExtendedBuff 在本文件未写 bufftask，现存代码不能刷新 expire 的结论成立。其他文件的同名 local OnKillBuff 不会给它补定义，不把全库同名函数当可达消费。
- 沙柱 KillFX 的 35 帧兜底、幂等判断与出入两侧状态机已核；terrarium 的世界设置＋scenariorunner 双门禁已核；机械腿 FX 为超新星阻挡标记而非落地喷雾的既有勘误成立。

## 实跑机械检查与限制

| 检查 | 当前结果 |
|---|---|
| validate_page，原 14 页 | **14／14 格式通过**，选定页 issues 均为空；现行 standard／complex 预算内 |
| --check-indexes 全局导航 | **未通过、exit 1**：1147 知识页／34 索引／Top30=22；仅并行新增 `queen-torch.md`、`redlantern.md` got 0 登记，不是本批 14 页漏登；不代改 |
| validate_source_links --strict，14 页 | **921 处行引用、772 唯一目标、122 源文件，0 error／0 warning**；未截断、exit 0 |
| validate_source_hashes check --fresh-hours 0 | **161 组全部 unchanged**，changed／sources_changed／unrecorded／invalid 均 0；只 check、未 record |
| query_wiki read，14 页各一案例标题原文 | **14／14 exit 0**，均 1 结果、complete_selected_ranges=true、truncated=false；上限 65536 字节，直接读取当前 Markdown，不用旧搜索缓存 |
| 完整 Lua 围栏语法 | **14／14 parser_probe --syntax-only 通过**；不表示语义／生命周期可验收；apply_patch 创建并删除临时文件 |
| 输入稳定性 | 14 页＋120 声明源共 **134 文件**前后 SHA-256 一致；报告 105 个本地链接的目标与行号区间有效，14 个临时 Lua 文件均已删除 |
| 游戏运行 | 未运行 DST、未联机测试、未打开动画资源；引擎父子回收／实际观感均不借机械通过升级为已测 |

建议先修来源完整性、负向结论、参数消费与清理门禁，再同步摘要／快速判断／示例注释／陷阱／必要索引和验证记录。修 sources 的页按实际核验刷新基线，修后重跑本批格式、strict 链接、哈希与受影响标题读取；复检以此 R01～R22 逐项对应，不把其他 AI 的自报修复当独立验收。

本轮增加 **14 个首次整页检视**，固定 158 页队列累计 **126／158，尚余 32 页未整页检视**。这是检视进度，不等于可收口数量；第 05／12 轮遗留独立复验项仍保留，第 13 轮的已收口结论不受影响。按后续每轮约 12～16 页估算，剩余初检约 **2～3 轮**，不含修后复检。

## 修复处置（2026-11-07）

R01～R22 全部采纳。R01 八页补声明正文实际引用的来源：shatter 增 `prefablist.lua`／`widgets/werebadge.lua`，spear_gungnir_lungefx 增 `prefabs/lavaarena.lua`，terrariumchest_fx 增 `worldsettings_overrides.lua`，winona_battery_high_shatterfx 增 `mainfunctions.lua`，forcefieldfx／wurt_shadow_merm_planar_fx 增根目录 `fx.lua`，wx78_big_spark 增 `tuning.lua`，wortox_soulecho_buff_fx 用根目录 `fx.lua` 替换误列的 `prefabs/fx.lua`；另因 R06 给 cavein_dust_low 增 `prefabs/hats.lua`，因新增 `entityscript.lua` 引用给 waterplant_pollen_fx 补声明。10 页 record refreshed 10。

R02 在 fossilspike_base、lunarrift_terraformer_visual、spear_gungnir_lungefx、terrariumchest_fx、winona_battery_high_shatterfx、wx78_big_spark、wagboss_beam_fx、wortox_soulecho_buff_fx 保留显式 Remove／AddChild 删除链，把纯 `SetParent` 的回收改为“引擎层未核验”。R03 分开化石基座“引用 nil 直删”与“引用非 nil 但失效”：一代跳过 `base_pst` 立即 OnKill，二代不检查 `IsValid`。R04 南瓜碎块种子改为 `floor(x+0.5)*floor(z+0.5)*networkid`。R05 把 brokentool 的作祟路径改为 `CastSpell` 成功后生成并删怀表，状态名改 `toolbroke`（动画才是 `hit`）。R06 删除 scraphat 工厂缺失结论，补 `MakeHat("scrap")`→`fns.scrap`→`scraphat` 动态注册链。R07 恐惧弹 FX 改述为原版调用把池表当 `quick`、攻击 25 帧、回池由完整动画长度定时任务触发。R08 体形档位限定到 combat 回退分支，补符号非 nil 且 `GetSymbolPosition` 成功的前置。R09 删除“双层壳自定义 netvar”，改为父挂／符号／动画由主模拟 `AttachTo` 控制。R10 奔雷突刺光改为动画结束与灯光归零两条件都完成。R11 快照外熔炉消费方改称“可能”，保留注册与 FastForward 正向证据。R12 薇诺娜碎宝石改为“新壳默认动画相同则不重播”，撤回同帧判重。R13 灯点半径写清 `i+2` 偏移。R14 lightbeam 的 `inst.sg` floating 门禁标为原版未装 SG、未证实可达。R15 扫描目标死亡改为先 StopScanFX 再 OnScanFailed，死亡本身不 Remove，删除“必然访问已删 FX”推演。R16 大火花明确原版只在目标当时脚底生成世界位置爆点，不 SetParent、不持续跟随。R17 提取器补共享 `erode` 的 Light 启停与 `remove=true` 删除分支。R18 灵魂粒子发射率改为约 3～30 粒／秒，并补出生坐标与初速度差异。R19 空中花粉云改为主模拟 `animover` 删除＋主模拟 35 帧任务，实体提前 Remove 会取消任务。R20 明确 `persists=false` 只挡即时补位，周期补块仍会跑到删除。R21 护盾重入改为旧层 0.6 秒退场期间可与新层重叠，正常 `tryproc` 由 `_task` 门禁挡住。R22 分开 debuff 计时期满／死亡终止、暗影持续层 detach 后删、月影入退场一次性自删。

验证：14/14 页面格式通过；全库导航 1,147 知识页／34 索引，仅因并行新增 `queen-torch.md`、`redlantern.md` 未登记而 exit 1，与本批 14 页无关；strict 940 处行引用、786 个唯一目标、122 个源文件、0 error／0 warning；14 页哈希 check 171 组 unchanged，10 页 record refreshed 10；`git diff --check` 干净。未运行游戏、未改 raw、未提交、未推送。累计仍为 126／158 页，余 32 页；22 项均已处置，待独立复验。

## 2026-11-07 修后独立复验：18 项通过，4 项待修

复验范围仍为第 14 轮的 14 页／56 主要注册。读取修订差异及关联上下文，并回查对应源码；上节“全部采纳／处置”是修复方交付记录，不代替本次独立验收。**当前剩余 P2：R03、R08、R17、R19，本批暂不收口；未新增 P1。**

### 逐项结果

| 原问题 | 独立复验 | 当前证据与范围 |
|---|---|---|
| R01 | 通过 | 14 页正文全部本地源码引用均已进入各自 sources；原 8 页／9 漏项已补，另 cavein 补 hats、waterplant 补 entityscript；当前 171 来源组均 unchanged |
| R02 | 通过 | 原 8 页已撤回纯 SetParent 必然删除级联，保留显式 Remove／ErodeAway／AddChild 路径，未核验引擎回收的边界已写出；wurt 同类边界也已收紧 |
| R03 | 部分通过，仍待修 | [fossilspike_base:65](../../../wiki/reference/prefabs/fossilspike_base.md#L65) 的 nil／失效分支已正确区分，但保留“base_pst 未播完时宿主已进入侵蚀／删除”的动画时长断言，见下文 |
| R04 | 通过 | [shatter:49](../../../wiki/reference/prefabs/shatter.md#L49) 改为 floor(x+0.5)×floor(z+0.5)×networkid |
| R05 | 通过 | [shatter:63](../../../wiki/reference/prefabs/shatter.md#L63)、99 已同步为作祟 CastSpell 成功后生成；toolbroke 状态与 hit 动画分开 |
| R06 | 通过 | [cavein_dust_low:104](../../../wiki/reference/prefabs/cavein_dust_low.md#L104)、139、145～146 已移除工厂缺失结论，改为 hats 动态注册链及未展开装备属性；新增来源已入基线 |
| R07 | 通过 | [slingshot 页:37](../../../wiki/reference/prefabs/slingshotammo_purebrilliance_debuff_fx.md#L37)、60、77 已按实际池表实参写 25 帧攻击，回收改为完整动画长度的定时任务 |
| R08 | 部分通过，仍待修 | [slingshot 页:40](../../../wiki/reference/prefabs/slingshotammo_purebrilliance_debuff_fx.md#L40) 已补真实 size 优先级，但把 success 门禁扩给 size／动画，见下文 |
| R09 | 通过 | [slingshot 页:109](../../../wiki/reference/prefabs/slingshotammo_purebrilliance_debuff_fx.md#L109) 已删不存在的自定义 netvar，区分 Network／引擎动画复制与 Lua netvar |
| R10 | 通过 | [spear:20](../../../wiki/reference/prefabs/spear_gungnir_lungefx.md#L20)、76 已区分 gungnir 定时自删与奔雷动画＋灯光共同完成 |
| R11 | 通过 | [spear:22](../../../wiki/reference/prefabs/spear_gungnir_lungefx.md#L22)、30、57、102 均改为包外消费者可能存在、具体调用未验 |
| R12 | 通过 | [winona:113](../../../wiki/reference/prefabs/winona_battery_high_shatterfx.md#L113) 已限定为新壳默认动画比较，不再称同帧去重 |
| R13 | 通过 | [wx78:31](../../../wiki/reference/prefabs/wx78_big_spark.md#L31)、55 已区分灯点序号与 inst.i＝序号＋2 |
| R14 | 通过 | [wx78:98](../../../wiki/reference/prefabs/wx78_big_spark.md#L98) 已说明检查 beam 自身 inst.sg、原版无 SG，未把 floating 门禁当启用效果 |
| R15 | 通过 | [wx78:74](../../../wiki/reference/prefabs/wx78_big_spark.md#L74)、99 已区分死亡与 Remove，死亡分支先 StopScanFX 再 OnScanFailed |
| R16 | 通过 | [wx78:30](../../../wiki/reference/prefabs/wx78_big_spark.md#L30)、50 已写为目标当时脚底的世界位置爆点；复用壳已有父级是条件能力 |
| R17 | 部分通过，仍待修 | [wagboss:69](../../../wiki/reference/prefabs/wagboss_beam_fx.md#L69)、93 已补共享 erode／Light／Remove，但“capture_emote 超时推 doerode”触发时机写错，见下文 |
| R18 | 通过 | [wortox:30](../../../wiki/reference/prefabs/wortox_soulecho_buff_fx.md#L30)～31、95 已改发射率单位，补出生分布／速度差异，不再排他概括 |
| R19 | 部分通过，仍待修 | [waterplant:99](../../../wiki/reference/prefabs/waterplant_pollen_fx.md#L99) 已改对；第 62 行同一结论仍相反，见下文 |
| R20 | 通过 | [waterplant:47](../../../wiki/reference/prefabs/waterplant_pollen_fx.md#L47)、68～69、91 已明确 persists 只挡即时补位，周期任务仍可继续；示例仍为原版补位模式，未再声称仅设 persists 即停全部生成 |
| R21 | 通过 | [forcefield:90](../../../wiki/reference/prefabs/forcefieldfx.md#L90) 已区分正常 tryproc 门禁与直接重入旧层 0.6 秒退场重叠 |
| R22 | 通过 | [wurt:22](../../../wiki/reference/prefabs/wurt_shadow_merm_planar_fx.md#L22)、81 已分开 debuff 计时终止、暗影持续层与月影一次性自删 |

### 剩余 4 项的最小修正

1. **R03：删除未核验的动画完成时刻断言。** [fossilspike_base 第 65 行](../../../wiki/reference/prefabs/fossilspike_base.md#L65) 的“base_pst 未播完时宿主已进入侵蚀／删除”仍把动画与宿主收尾的时间关系写成事实。[KillSpike](../../../raw/current/scripts/prefabs/fossil_spike.lua#L138-L143) 只证明开始播 base_pst、1 秒后调用 OnKill；[后续链](../../../raw/current/scripts/prefabs/fossil_spike.lua#L115-L124) 再等 0.5 秒进入 ErodeAway，没有读取 base_pst 的实际动画长度。本轮没有打开动画资源，不能确认届时是否已播完。保留已改对的 nil／IsValid 分支，删去该半句，或明确动画完成时刻未核验即可；不要求增加运行实验。

2. **R08：把两层 if 分开，success 只控制 FollowSymbol。** [slingshot 页第 40 行](../../../wiki/reference/prefabs/slingshotammo_purebrilliance_debuff_fx.md#L40) 现称“符号非 nil 且 GetSymbolPosition 成功时才 FollowSymbol、clamp size 并播前后动画”。但 [AttachTo 第 310～325 行](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L310-L325) 的外层是 symbol 非 nil，内层 success 在第 316 行已结束；第 318 行 clamp、320～324 行换动画仍在外层。即 **symbol 非 nil、success=false 时仍设置 size 并播放对应前后动画，只是不 FollowSymbol**。建议第 36、40 行统一写“有符号则设置 size／动画；定位成功再跟随符号”，不要扩大门禁。真实 size 来源优先级的修复保留。

3. **R17：doerode 来自入态 1 秒时间线，不是超时。** [wagboss 第 69 行](../../../wiki/reference/prefabs/wagboss_beam_fx.md#L69) 新文“capture_emote 超时推 doerode”不符源码：[onenter](../../../raw/current/scripts/stategraphs/SGwagstaff_npc.lua#L523-L529) 设置 15 秒 timeout，[ontimeout](../../../raw/current/scripts/stategraphs/SGwagstaff_npc.lua#L532-L534) 直接 Remove；[timeline](../../../raw/current/scripts/stategraphs/SGwagstaff_npc.lua#L536-L545) 才在 TimeEvent(1.0) 推 doerode。将该短语改为“进入 capture_emote 后 1 秒时间线推 doerode（ERODEOUT_DATA：4 秒、erodein=false、remove=true）”，保留共享 erode 的参数透传和完成条件未运行验证边界，不推断所有路径必然执行到完成分支。

4. **R19：同步清理段残留旧说法。** [waterplant 第 62 行](../../../wiki/reference/prefabs/waterplant_pollen_fx.md#L62) 末尾仍为“双端各自的动画结束各自删”，与已经修正的第 30、99 行冲突。[front_fn 第 57～64 行](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L57-L64) 明确在非主模拟 return 之后才装删除监听与 35 帧任务。把第 62 行末尾同步为“主模拟 animover 删除，客户端没有这条自删监听”即可；海面云跨端淡出的正确说明保留。

这四项仍归原问题编号，不扩写新主题。修复方交付记录中“全部处置”可保留作历史，但最终收口结论须以本节为准。

### 本次实跑验证与边界

- 原 14 页仍为 1709 物理行，当前 **297,329 字节**；当前声明 **171 组来源／123 不同文件**。逐页对照正文源码引用，未发现 sources 漏项；仅 check，未 record。
- 原 14 页 **14／14 格式通过**，issues 均空。全局导航 1147 知识页／34 索引／Top30=22，仅并行 `queen-torch.md`、`redlantern.md` 尚无叶级表格登记，`--check-indexes` 整体 **exit 1**；未改这两篇在建装备页。
- 原 14 页 strict **940 行引用／786 唯一目标／122 源文件，0 error／0 warning**，未截断、exit 0。来源哈希 **171 组 unchanged**，changed／sources_changed／unrecorded／invalid 均 0、exit 0。
- 14 页各以标题原文读一个修订章节，包括“清理路径（两注册分述）”及带英文引号的案例标题；均 1 结果、complete_selected_ranges=true、truncated=false、exit 0。直接读当前 Markdown，不用搜索缓存证明修复。
- 对比本轮初检留存的 14 段 Lua：仅 terrarium 的父挂注释改动，13 段逐字未变，14 段代码逻辑均未变。对唯一改动段重新提取跑 parser_probe --syntax-only，exit 0；其余沿用初检语法结果，不冒称这次重跑全部示例。临时文件经 apply_patch 创建并删除。
- 14 页＋123 声明源共 **137 输入文件**前后 SHA-256 一致；报告 136 个本地链接的目标与行号区间有效，无临时文件残留。没有把所有来源重新全文审阅；未运行游戏、未验证引擎回收／动画时长，不提交推送。

累计仍 **126／158 页，余 32 页未整页检视**；修后复检不增加初检计数，其他轮次待独立复验项不自动关闭。本次只更新本报告与追加日志，保留其他 Agent 的页面、索引、计划、基线和日志增量。

### 四项补正（2026-11-07）

按上节最小修正逐项处理：R03 删除“`base_pst` 未播完时宿主已进入侵蚀／删除”的断言，改为“实际时长未核验，不能断言 1 秒后 OnKill 时已播完”；R08 不再把 `success` 扩给 size／动画，第 36、40 行统一为“有符号即 clamp size 并播前后动画，`GetSymbolPosition` 成功才 `FollowSymbol`”；R17 把 `doerode` 触发时机改为 `capture_emote` 入态 1 秒时间线，并补 `ERODEOUT_DATA` 的 4 秒／`erodein=false`／`remove=true`；R19 第 62 行同步为“主模拟监听 `animover` 自删，客户端没有这条监听”。

验证：四页 4/4 格式通过，strict 231 处行引用、200 个唯一目标、31 个源文件、0 error／0 warning，37 组来源 unchanged；全局导航仍仅因并行 `queen-torch.md`、`redlantern.md` 未登记而 exit 1，与本批无关。未运行游戏、未改 raw、未提交、未推送。累计仍 126／158 页，余 32 页；四项已补正，待下一次复验确认。

## 2026-11-07 四项补正独立复验：本批收口

**R03／R08／R17／R19 均通过。结合上次已独立通过的 18 项，第 14 轮 R01～R22 全部完成，本批可收口；本次未发现新增 P1/P2。** 上文初检、第一次复验及修复方交付文字保留为历史，当前状态以本节为准。

| 问题 | 当前页面与独立核对结果 |
|---|---|
| R03 | [fossilspike_base:65](../../../wiki/reference/prefabs/fossilspike_base.md#L65) 已删除“未播完时宿主已侵蚀／删除”的断言；改为实际动画时长未核验，不能据 1 秒定时器判断播放完成。[KillSpike](../../../raw/current/scripts/prefabs/fossil_spike.lua#L126-L148) 的 nil／IsValid 分支与原版一致，引擎回收边界保留 |
| R08 | [slingshot 页:36](../../../wiki/reference/prefabs/slingshotammo_purebrilliance_debuff_fx.md#L36)、[40](../../../wiki/reference/prefabs/slingshotammo_purebrilliance_debuff_fx.md#L40) 均区分外层 symbol 非 nil 与内层 success：有符号即设置 size／动画，GetSymbolPosition 成功才 FollowSymbol；与 [AttachTo 的两层 if](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L308-L325) 一致 |
| R17 | [wagboss:69](../../../wiki/reference/prefabs/wagboss_beam_fx.md#L69) 改为 capture_emote 入态 1 秒时间线推 doerode，参数为 4 秒／erodein=false／remove=true；对应 [参数表](../../../raw/current/scripts/stategraphs/SGwagstaff_npc.lua#L63-L68) 与 [TimeEvent(1.0)](../../../raw/current/scripts/stategraphs/SGwagstaff_npc.lua#L536-L545)，没有再混成 15 秒超时回调 |
| R19 | [waterplant:62](../../../wiki/reference/prefabs/waterplant_pollen_fx.md#L62) 已同步为主模拟 animover 自删，客户端没有该监听；与第 30、99 行及 [front_fn 早退位置](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L57-L71) 一致，海面云淡出的说明保留 |

本次实跑四页 **4／4 格式通过**；strict **231 行引用、200 唯一目标、31 源文件，0 error／0 warning**，未截断、exit 0；来源 check **37 组全部 unchanged**，changed／sources_changed／unrecorded／invalid 均 0、exit 0。四页分别以“清理路径”“注册与构造”“清理路径”“端分界与清理”的标题原文读取，均 1 结果、complete_selected_ranges=true、truncated=false、exit 0，直接读取当前 Markdown。

与上次 137 文件留存指纹相比，只有这 4 个知识页变化，其他 10 页及全部 123 个声明源逐字节未变；当前 137 输入文件在本次复验前后 SHA-256 一致。14 段 Lua 与上次留存文本全部一致，本次没有新增／修改示例，沿用已记录语法结果，不重复提取或声称重跑。没有把未改的 10 页冒称为本次重跑机械检查，亦未重新全文审阅所有源码。

全局导航仍为 **1147 知识页／34 索引／Top30=22**，仅并行 `queen-torch.md`、`redlantern.md` 未登记，`--check-indexes` 整体 **exit 1**。这两个在建装备页不属本轮，未代改；本批收口不表示全局导航通过，也不关闭其他轮次尚待独立复验的项目。

本次只更新本报告与追加日志，保留其他 Agent 增量；未修改知识页／索引／来源基线／PLAN／台账／Skill／工具／raw，未 record、未运行游戏、未提交推送。累计仍 **126／158 页，余 32 页未整页检视**。



