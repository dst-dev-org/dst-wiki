# FX 75 页全文复核（2026-10-30）

范围：Steam buildid 26230496；以现有 75 文件队列对应实际知识页逐页检查，不按文件名猜注册。工厂全文与必要调用／消费者定向复核；结构、链接、基线与语法检查另行记录，不替代行为核验。未运行游戏、未解包资源，不声称运行时验证。

状态只记录本次检查，不改变知识页验证等级。未完成项不会按已复核汇总；其他执行者的原文与修改保留。

| 序号 | 页面 | 工厂 | 复核状态与处理 |
|---|---|---|---|
| 1 | wiki/reference/prefabs/abigail_attack_fx.md | prefabs/abigail_attack_fx.lua | 已复核：双注册／宿主播放及 kill_fx 已核；未见阻塞问题。 |
| 2 | wiki/reference/prefabs/acidsmoke_fx.md | prefabs/acidsmoke_fx.lua | 已复核：核对工厂与 acidinfusible 全生命周期；修正动画网络同步与休眠立即清理描述。 |
| 3 | wiki/reference/prefabs/alterguardian_summon_fx.md | prefabs/alterguardian_summon_fx.lua | 已复核：核对光照三表、net_event 与双层退场；未见阻塞问题。 |
| 4 | wiki/reference/prefabs/bearger_swipe_fx.md | prefabs/bearger_fx.lua | 已复核：核对双注册参数、SG Reverse／animover 与退出清理；未见阻塞问题。 |
| 5 | wiki/reference/prefabs/boat_water_fx.md | prefabs/boat_water_fx.lua | 已复核：核对本地工厂、boattrail／mover；补齐延长寿命受 80 帧兜底限制。 |
| 6 | wiki/reference/prefabs/book_fx.md | prefabs/book_fx.lua | 已复核：核对六注册名及 SG 服务端生成／移交；纠正预测客户端说法与 waxwell 变体计数。 |
| 7 | wiki/reference/prefabs/brilliance_projectile_fx.md | prefabs/brilliance_projectile_fx.lua | 已复核：核对弹跳筛选、命中额度、颜色叠加、武器结算；修正压暗／耐久及排序表述，补 sources。 |
| 8 | wiki/reference/prefabs/cannon_aoe_range_fx.md | prefabs/cannon_aoe_range_fx.lua | 已复核：两本地工厂与消费者边界已核；无阻塞问题。 |
| 9 | wiki/reference/prefabs/cave_vent_ground_fx.md | prefabs/cave_vent_ground_fx.lua | 已复核：工厂存档／死代码与宿主依赖已核；补官方译名 sources。 |
| 10 | wiki/reference/prefabs/channel_absorb_fire_fx.md | prefabs/channel_absorb_fire_fx.lua | 已复核：四工厂／lighter 扫描与清理已核；补余烬 sources，收紧未验证末帧结论。 |
| 11 | wiki/reference/prefabs/confetti_fx.md | prefabs/confetti_fx.lua | 已复核：核对双形状、发射参数与专服早退；修正粒子寿命、位置、调度及专服清理示例。 |
| 12 | wiki/reference/prefabs/daywalker2_swipe_fx.md | prefabs/daywalker2_fx.lua | 已复核：核对单工厂、Reverse 与 SG 临时伤害／退出清理；未见阻塞问题。 |
| 13 | wiki/reference/prefabs/deer_ice_circle.md | prefabs/deer_fx.lua | 已复核：九注册与冰火玩法／清理全文已核；修正温度门槛、grog 增量、点燃计数及抽角不重复的误述。 |
| 14 | wiki/reference/prefabs/deerclops_icespike_fx.md | prefabs/deerclops_icespike_fx.lua | 已复核：池化工厂与场景进出已核；补译名来源、纠正客户端只等删除与取焰错词。 |
| 15 | wiki/reference/prefabs/deerclops_icelance_ping_fx.md | prefabs/deerclops_mutated_fx.lua | 已复核：四工厂／冰圈结算已核；纠正 aura 半径公式与冷度硬封顶的错误推断。 |
| 16 | wiki/reference/prefabs/electricchargedfx.md | prefabs/electric_charged_fx.lua | 已复核：双实体、闪白与还原已核；纠正客户端可见性推断与 day_time 时长解释。 |
| 17 | wiki/reference/prefabs/electrocute_fx.md | prefabs/electrocute_fx.lua | 已复核：全工厂与通用 SG 接线已核；替换非法 Lua，补 duration 前提，修正裸循环自删误导。 |
| 18 | wiki/reference/prefabs/eyeofterror_arrive_fx.md | prefabs/eyeofterror_arrive_fx.lua | 已复核：核对插值、色彩任务与退场；注明 TurnLightOff 为局部函数非实体方法。 |
| 19 | wiki/reference/prefabs/firepit_firebird_puff_fx.md | prefabs/firepit_firebird_puff_fx.lua | 已复核：代理与皮肤 takefuel 原实现已核；修正示例燃料分档、卸监听字段及同步推断。 |
| 20 | wiki/reference/prefabs/flamethrower_fx.md | prefabs/flamethrower_fx.lua | 已复核：布焰代理全文及余烬调用已核；收紧 PVP 排除组表述。 |
| 21 | wiki/reference/prefabs/float_fx_front.md | prefabs/float_fx.lua | 已复核：双本地工厂、floater 网络值与移除链已核；未见阻塞问题。 |
| 22 | wiki/reference/prefabs/fossilized_break_fx.md | prefabs/fossilized_fx.lua | 已复核：四注册与闭包已核；补 po，注明随机选择会改写工厂闭包变量。 |
| 23 | wiki/reference/prefabs/gelblob_attach_fx.md | prefabs/gelblob_attach_fx.lua | 已复核：全工厂与双减速／清理路径已核；明确减速到 Remove 才注销，纠正不能恢复推断。 |
| 24 | wiki/reference/prefabs/ghost_transform_overlay_fx.md | prefabs/ghost_transform_overlay_fx.lua | 已复核：全工厂光照时序已核；修正 Flash 覆盖而非累加。 |
| 25 | wiki/reference/prefabs/hitsparks_fx.md | prefabs/hitsparks_fx.lua | 已复核：三工厂、Setup 与闪色全文已核；纠正武器官方中文名、同步推断及 piercing 参数前提。 |
| 26 | wiki/reference/prefabs/ice_crack_grid_fx.md | prefabs/ice_crack_grid_fx.lua | 已复核：全工厂与管理器铺设／销毁已核；未见阻塞问题。 |
| 27 | wiki/reference/prefabs/lantern_crystal_fx_ground.md | prefabs/lantern_crystal_fx.lua | 已复核：双工厂与宿主协议已核；修正摘要误称无 KillFX 与同帧推断。 |
| 28 | wiki/reference/prefabs/lantern_flower_fx_held.md | prefabs/lantern_flower_fx.lua | 已复核：全文及与同族参数对照已核；补 insect sources，完善示例 i／CreatePetal 作用域前提。 |
| 29 | wiki/reference/prefabs/lantern_gothic_fx_held.md | prefabs/lantern_gothic_fx.lua | 已复核：同构工厂全文、glow 参数与宿主协议已核；未见阻塞问题。 |
| 30 | wiki/reference/prefabs/lantern_insect_fx_held.md | prefabs/lantern_insect_fx.lua | 已复核：同构工厂全文、混搭符号／动画与宿主协议已核；未见阻塞问题。 |
| 31 | wiki/reference/prefabs/lantern_tesla_fx_held.md | prefabs/lantern_tesla_fx.lua | 已复核：两拼名工厂与宿主清理已核；未见阻塞问题。 |
| 32 | wiki/reference/prefabs/lantern_winter_fx_held.md | prefabs/lantern_winter_fx.lua | 已复核：同构工厂、snow 参数与宿主清理已核；统一冬季盛宴提灯称谓。 |
| 33 | wiki/reference/prefabs/lavaarena_creature_teleport_small_fx.md | prefabs/lavaarena_creature_spawn_fx.lua | 已复核：六注册、计数、音量与清理已核；修正 0 延迟调度和外部生成点的证据边界。 |
| 34 | wiki/reference/prefabs/lavae_move_fx.md | prefabs/lavae_move_fx.lua | 已复核：双工厂量化／dirty 门禁与本地退场已核；未见阻塞问题。 |
| 35 | wiki/reference/prefabs/lunar_goop_cloud_fx.md | prefabs/lunar_goop_cloud_fx.lua | 已复核：全工厂与宿主 EnableCloud 已核；补 sources，纠正开关不影响宿主逻辑的误述。 |
| 36 | wiki/reference/prefabs/lunar_goop_trail_fx.md | prefabs/lunar_goop_trail_fx.lua | 已复核：全工厂与池调用已核；纠正默认停帧、duration 起点、共享池归属，补速率未重置边界。 |
| 37 | wiki/reference/prefabs/lunarrift_crystal_spawn_fx.md | prefabs/lunarrift_crystal_spawn_fx.lua | 已复核：全工厂及专服早退已核；修复清理示例与粒子恰好同时消散的错误结论，补 po。 |
| 38 | wiki/reference/prefabs/marionette_appear_fx.md | prefabs/marionette_fx.lua | 已复核：双工厂与定时／渐隐已核；纠正 dissappear 拼写说明。 |
| 39 | wiki/reference/prefabs/lunarmerm_thorns_fx.md | prefabs/merm_fx.lua | 已复核：双工厂与真实伤害全链已核；明确首次半径及到 3 后停止判伤。 |
| 40 | wiki/reference/prefabs/miasma_cloud.md | prefabs/miasma_cloud_fx.lua | 已复核：845 行工厂逐段及 debuff 已核；补三变体别名，修正子体计数、火半径、睡眠清源及受管分支条件。 |
| 41 | wiki/reference/prefabs/monkeyisland_portal_fxloot.md | prefabs/monkeyisland_portal_fxloot.lua | 已复核：全工厂与传送门吐物链已核；修正未满额也能加权抽中 FX 且会入 _loot 的关键误述。 |
| 42 | wiki/reference/prefabs/moonstorm_ground_lightning_fx.md | prefabs/moonstorm_ground_lightning_fx.lua | 已复核：本地工厂与链式门禁已核；纠正仅主模拟之外生成的误述。 |
| 43 | wiki/reference/prefabs/moose_nest_fx_idle.md | prefabs/moose_nest_fx.lua | 已复核：双代理与本地视觉工厂已核；改正服务端不能引用实体的过度结论。 |
| 44 | wiki/reference/prefabs/mossling_spin_fx.md | prefabs/mossling_spin_fx.lua | 已复核：全工厂与音效计数已核；修正拼写及客户端声效断言。 |
| 45 | wiki/reference/prefabs/nightsword_cotl_fx.md | prefabs/nightsword_cotl_fx.lua | 已复核：全工厂两发射器与攻击门禁已核；收紧骑乘端差异推断，避免把 mount=nil 当必发条件。 |
| 46 | wiki/reference/prefabs/nightsword_curve_fx.md | prefabs/nightsword_curve_fx.lua | 已复核：全工厂三发射器、方向与加速度已核；收紧骑乘门禁推断。 |
| 47 | wiki/reference/prefabs/nightsword_lightsbane_fx.md | prefabs/nightsword_lightsbane_fx.lua | 已复核：全工厂循环实发数及 sharp 重名已核；收紧骑乘门禁推断。 |
| 48 | wiki/reference/prefabs/nightsword_sharp_fx.md | prefabs/nightsword_sharp_fx.lua | 已复核：全工厂循环实发数及包络已核；修正相同颜色关键帧时点与骑乘结论。 |
| 49 | wiki/reference/prefabs/nightsword_wizard_fx.md | prefabs/nightsword_wizard_fx.lua | 已复核：全工厂两发射器已核；修正唯一无 bloom 的错误比较与骑乘结论。 |
| 50 | wiki/reference/prefabs/petrified_trunk_break_fx.md | prefabs/petrify_fx.lua | 已复核：五注册、染色编码及本地生命周期已核；注明灰度实际下限 0.5。 |
| 51 | wiki/reference/prefabs/player_float_hop_water_fx.md | prefabs/player_float_fx.lua | 已复核：双工厂与 SG 对帧／解绑已核；纠正 bank 名推断资源未声明的说法。 |
| 52 | wiki/reference/prefabs/pocketwatch_weapon_fx.md | prefabs/pocketwatch_weapon_fx.lua | 已复核：全工厂 PPS 与宿主字段已核；修复 owner=nil 会报错的前提遗漏，精确 blend 与角速度。 |
| 53 | wiki/reference/prefabs/rabbithouse_yule_glow_fx.md | prefabs/rabbithouse_yule_glow_fx.lua | 已复核：全工厂与皮肤常驻清理已核；未见阻塞问题。 |
| 54 | wiki/reference/prefabs/reviver_cupid_beat_fx.md | prefabs/reviver_cupid_beat_fx.lua | 已复核：双工厂与拍点／高亮已核；纠正 beat 也调用 PlayAnimation 的误述。 |
| 55 | wiki/reference/prefabs/rock_break_fx.md | prefabs/rock_break_fx.lua | 已复核：全工厂音效代理已核；纠正 1 帧／1 秒混淆。 |
| 56 | wiki/reference/prefabs/rose_petals_fx.md | prefabs/rose_petals_fx.lua | 已复核：全工厂 ErodeAway 路径已核；删除引擎声音随 Remove 必停的无证据断言。 |
| 57 | wiki/reference/prefabs/shadow_glob_fx.md | prefabs/shadow_glob_fx.lua | 已复核：全工厂与涟漪任务已核；明确 PERIOD 为秒却与帧号比较，纠正客户端固定 shad1 的推断。 |
| 58 | wiki/reference/prefabs/shadowthrall_projectile_fx.md | prefabs/shadowthrall_projectile_fx.lua | 已复核：全工厂真实伤害已核；纠正 padding 3 被误计入实际伤害半径。 |
| 59 | wiki/reference/prefabs/sharkboi_swipe_fx.md | prefabs/sharkboi_fx.lua | 已复核：五工厂各自注册／变体／定时退场已核；未见阻塞问题。 |
| 60 | wiki/reference/prefabs/spooked_spider_rock_fx.md | prefabs/spooked_fx.lua | 已复核：双工厂唤醒声效／单次生命周期已核；未见阻塞问题。 |
| 61 | wiki/reference/prefabs/collapse_big.md | prefabs/structure_collapse_fx.lua | 已复核：双代理、材质表与播放分支已核；纠正专服也创建动画实体的误述。 |
| 62 | wiki/reference/prefabs/superjump_fx.md | prefabs/superjump_fx.lua | 已复核：双工厂与 SetTarget 已核；纠正主 FX 缩放与外部调用方未经核验的断言。 |
| 63 | wiki/reference/prefabs/tophat_shadow_fx.md | prefabs/tophat_shadow_fx.lua | 已复核：三工厂及本地层／粒子池已核；补 prefablist 来源并区分 CLASSIFIED 层与 FX 粒子。 |
| 64 | wiki/reference/prefabs/pumpkincarving_swap_fx.md | prefabs/ugc_swap_fx.lua | 已复核：双工厂数据／昼夜／清理全链已核；纠正固定 swap_body 与宿主颜色监听的误述。 |
| 65 | wiki/reference/prefabs/vault_pillar_guard_swipe_fx.md | prefabs/vault_pillar_guard_fx.lua | 已复核：双工厂与首帧 PostUpdate 对时已核；未见阻塞问题。 |
| 66 | wiki/reference/prefabs/warg_mutated_breath_fx.md | prefabs/warg_mutated_fx.lua | 已复核：597 行双工厂与命中／冰烬已核；补 ConfigureDamage 的 planardamage 前提，修正半径、限频、降温和回退条件。 |
| 67 | wiki/reference/prefabs/weremoose_smash_fx.md | prefabs/weremoose_smash_fx.lua | 已复核：全工厂与预测去重已核；纠正快速判断的 bank／build 混写。 |
| 68 | wiki/reference/prefabs/willow_shadow_flame.md | prefabs/willow_flame_fx.lua | 已复核：384 行三工厂与链生／buff 已核；纠正 pre 被覆盖、life 限制与位面措辞。 追查 firefx:SetLevel，撤回动画自动退场承诺与不完整直接复用代码。 |
| 69 | wiki/reference/prefabs/woby_dash_shadow_fx.md | prefabs/woby_shadow_fx.lua | 已复核：双工厂与 WallUpdate 追踪已核；未见阻塞问题。 |
| 70 | wiki/reference/prefabs/wormwood_plant_fx.md | prefabs/wormwood_plant_fx.lua | 已复核：全工厂驻留条件与 PlantTick 已核；未见阻塞问题。 |
| 71 | wiki/reference/prefabs/wortox_eat_soul_fx.md | prefabs/wortox_eat_soul_fx.lua | 已复核：全工厂与 mounted 接入已核；收紧所有进食者自动生成及清理先后推断。 |
| 72 | wiki/reference/prefabs/wortox_portal_fx.md | prefabs/wortox_portal_fx.lua | 已复核：双工厂与动作／消耗实现已核；纠正 consumeall 不等于清空背包所有灵魂。 |
| 73 | wiki/reference/prefabs/wortox_soul_heal_fx.md | prefabs/wortox_soul_heal_fx.lua | 已复核：全工厂帧驱动与治疗调用已核；明确 colouradder 不自动挂载，完善示例符号前提。 追查 colouradder 自动监听颜色源移除，补来源与实际清理依据。 |
| 74 | wiki/reference/prefabs/wortox_soul_in_fx.md | prefabs/wortox_soul_in_fx.lua | 已复核：全工厂 net_entity／客户端 tint 与清理已核；未见阻塞问题。 |
| 75 | wiki/reference/prefabs/wurt_casting_fx.md | prefabs/wurt_casting_fx.lua | 已复核：四工厂与骑乘拼名已核；纠正缺 i 的拼写说明与基础 fxprefab 前提。 |

## 共享问题与最终验收

本次 75/75 页已逐篇复核正文及对应工厂，必要时回查组件、状态图或消费点；没有以高风险抽样代替页面覆盖。新增来源声明涉及 13 页，使用既有 record 生成器局部合并基线，未修改游戏源码或工具实现。

主要修复（具体页见上表）：

- 接入与清理：替换 electrocute 的非法 Lua；warg 的 ConfigureDamage 明确外部 combat.inst 必须有 planardamage；lunarrift 与 confetti 补专服清理前提；pocketwatch 明确父级不能为空。
- 间接行为：追查 firefx:SetLevel 会把薇洛火焰改成显式循环，撤回“播完自删”承诺与不完整直接复用代码；life 只限制链生，不负责删除已有焰。此处修复知识描述，没有修改游戏实现。
- 实际计算：区分 AOE 粗查 padding 与真实命中半径；修正 aura 半径公式、冷度阈值与增量、伤害 tick 严格比较、粒子寿命、duration 起点、共享池归属、Flash 覆盖与岩石音效壳 1 秒寿命。
- 交互与同步：修正传送门假掉落也能加权抽中并记入 _loot、瘴气休眠立即清源；区分没有客户端 Lua 回调与没有网络动画／颜色更新，不据缺 rider 推断骑乘必然发射。
- 名称与导航：命中火花消费者使用 po 核实的亮茄剑／暗影槌／暗影收割者／亮茄盔甲／嚎弹炮；补齐瘴气三个注册别名。fx-index 的 75 条截断摘要改为完整需求说明，并补必要变体标识符；仍保留原有六组与唯一登记。

| 定向验收 | 实际结果 |
|---|---|
| validate_page --check-indexes | 75 页＋两索引共 77 文件通过；导航 0 issue（全库导航盘点 919 知识页／29 索引） |
| validate_source_links --strict | 75 页、1,880 处引用，257 个源码文件；0 error／0 warning |
| 来源声明对齐 | 75 页正文的源码链接均包含在 sources；漏声明 0 |
| Lua 语法 | Full Moon 对全部保留的 48 段 Lua 围栏解析通过；临时文件经 apply_patch 创建／删除，残留 0 |
| 标题原文章节 read | electrocute 接入、lunarrift 粒子接入、warg 伤害火焰接入、willow 自动索敌接入，4/4 返回 0 |
| 别名检索 | 三个补齐的瘴气注册名经直接读取 frontmatter 均唯一指向 miasma_cloud；未使用 stale search 作验收 |
| 基线 | 13 页 sources 变化局部 record；75 页 431 组比较全部 unchanged，sources_changed=0 |
| 索引预算 | fx-index 为 116 物理行／13,807 字节，未超过 160 行／16 KiB |
| 文本检查 | 75 页无行尾空白；git diff --check 通过 |

查询缓存 output/wiki/index.json 仍为 62 篇知识页，当前导航盘点为 919 篇，缓存未重建。检索验证使用当前 Markdown／别名字段与 read，不能把它当新缓存的 search 验收。

未运行 DST、未解包动画／音频资源；引擎复制、粒子残留、晚加入、实际观感及音效听感保持未验证。语法解析不等于 Lua 示例在任意宿主可直接运行，页面前提仍须满足。保留其他 Agent 的组件回链、索引及日志增量；无提交或推送。

