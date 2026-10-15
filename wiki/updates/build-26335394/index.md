---
title: "源码更新影响说明：build 26230496 → build 26335394"
created: "2026-11-01"
dst_build: "steam-buildid:26335394"
status: active
---

# 源码更新影响说明 build 26335394

快照 26230496 → 26335394（2026-11-01），万圣夜怪物化内容更新。全库 84 个源码文件变动：80 改 / 4 增 / 0 删；哈希全检 954 个带 sources 页面、5,528 组对比中 893 组真变化，受影响页面 573（行尾 0、大小写 0、删除 0、来源集合变化 0）。

**版本与快照身份**：起始 build 26230496（旧快照 `raw/archive/build-26230496`，即先前 current，2026-11-01 归档）；目标 build 26335394（新快照 `raw/current`，BUILD.txt 同值，4095 个 .lua）。本报告只覆盖这一起止对；其他起点另建报告。

**阅读方式（渐进式）**：本页只放主题与决策规则，逐文件明细在 5 个 detail 文件。兼容性相关条目带稳定编号（C／R／P／SB／M-序号），适配结论可直接引用编号。适配判断三步：① 读下方适配检查清单，命中即得动作；② 未命中但涉及变化机制的，按主题尾部指针打开对应 detail 文件，按组件名／函数名／事件名定位；③ detail 条目均出自实际 diff（全部已核验，无待核验条目）。机器事实（changed-files.txt）在本目录；页面处理队列用 `validate_source_hashes.py check --fresh-hours 0` 的 affected_pages 重建，页面核验状态以各页 dst_build 为准。

## 本轮主题（模组影响分诊）

- **万圣夜怪物化三件套**：普通 pigman／merm／bunnyman 不新增注册名，而是活动期按 30% 概率怪物化——科学怪猪（Frankenswine／科学怪猪）、木乃伊鱼人（Mermummy／鱼人木乃伊）、吸血兔（Vamphare／吸血兔），统一模板：600 血＋回血、伤害 40／攻速 2、免睡眠、无尸体、`sg.mem.noparasitehost`、负理智光环、掉糖果与南瓜弹，服务端字段 `inst.franken`／`inst.mummy`／`inst.vampire` ＋ 客户端 `monster` 标签换名。触发链各异：猪在地面活动期掷点、鱼人由 `ismummyhouse` 鱼人屋生成、兔仅限洞穴，存档均走 OnPreLoad 且活动关闭后不还原表现。巢穴作息反转（猪屋／兔屋新增 `EnableFrankenPigHouse`／`EnableVampireBunnyHouse`，打死屋内怪自动复原）。索敌排除连锁：卫兵／猪群／鱼人／兔群／触手／Wurt 全部跳过怪物化个体。 （明细：[detail-prefabs.md](detail-prefabs.md)、[detail-sg.md](detail-sg.md)、[detail-components.md](detail-components.md)）
- **SG 怪物化状态族 `hn_*`**：SGpig／SGmerm／SGbunnyman 旁路 CommonHandlers.OnLocomote/OnAttack 换自定义事件，新增 hn_walk/hn_run/hn_attack 状态（先走 3 秒再冲刺、lunge 位移攻击）；werepig／moonpig 的 attack 态同款冲刺化。 （明细：[detail-sg.md](detail-sg.md)）
- **击退框架收尾**：尸体三态加 `noknockback`；击退对带 inventoryitem 实体正确落地（SetLanded），普通生物落地声移除（SGbird 自补）；bat_boss 免击退；ticoon／bird_mutant 首次接入可击退。 （明细：[detail-sg.md](detail-sg.md)、[detail-prefabs.md](detail-prefabs.md)）
- **施法目标门禁（仅 USEITEM）**：`spellcaster` 组件动作的持物右键处理器拒绝背包内目标；EQUIPPED／POINT／INVENTORY 入口未加此门。 （明细：[detail-root.md](detail-root.md)）
- **道具武器 PVP 标签（双路径异构）**：新 `propweapon_nopvp`（witchbroom 首用）——`IsValidTarget` 路径退出 propweapon 豁免、`CanBeAttacked` 路径反为独立放行资格；witchbroom 双标签净效果为退出友伤豁免。 （明细：[detail-components.md](detail-components.md)、[detail-prefabs.md](detail-prefabs.md)）
- **世界设置档位改名（破坏性，范围有限）**：`applyoverrides_pre` 中 40 个 `few`＋39 个 `many` 档改名 `rare`／`often`（petrification 等未改名）；按旧档名设置静默失效；Balatro 机 BEES 键随改 `WASPHIVE_ENABLED`，显式 false 现在真正禁用（明细：[detail-root.md](detail-root.md)、[detail-prefabs.md](detail-prefabs.md)）
- **扩展点与组件 API**：`homeseeker_onsethome(inst, home, oldhome)` 换家钩子；`carefulwalker:Enable(b)` 开关（含 OnRemoveFromEntity 崩溃修复）；`sweeper` 音效 API（SetSweepSound/OnSweep，Sweep 前置调用）；`NAUGHTY_VALUE` 支持函数值（franken/werepig/vampire 淘气豁免）；combat 放行 companion 标签仇恨；king_cane 降级撤销 CASTSPELL 强制拟态揭示。 （明细：[detail-components.md](detail-components.md)、[detail-root.md](detail-root.md)、[detail-prefabs.md](detail-prefabs.md)）
- **万圣活动经济**：双王（猪王／鱼人王）糖果交易（鱼人王排除 11 号巧克力猪）；南瓜弹猪王掉率 0.33→0.2、抛弹上限 %6→%8；落叶堆可燃（烧毁只掉灰）且 treat 有声；4 个新装饰。 （明细：[detail-prefabs.md](detail-prefabs.md)、[detail-root.md](detail-root.md)）
- **杂项行为**：暗影香炉即死 stalker_npc（旧监听本被注释，属首个受击行为）；trap_fumarole 布置后无物理＋传送刷新；wobster 无海洋沉没；penguin/monkey/molebat 补 drownable＋落水坠虚态；mermking 对任意交易物清仇恨；薇洛暗焰目标追踪重写（2D 近距判定、近距放宽偏转上限，爆炸位置无版本差异）；balatro false 判定修复；pigking 测试钩子移除；versioning 子版本位。 （明细：[detail-prefabs.md](detail-prefabs.md)、[detail-root.md](detail-root.md)、[detail-sg.md](detail-sg.md)）
- **资源与表现（一笔带过）**：pumpkin_bomb／leafpile FX 音效替换（fx.lua 本体仅 6 处音效字串、行号零位移）；本地化 3 文件各 +630 行、17 角色语音新条目；官方名 Frankenswine／Mermummy／Vamphare（简中 科学怪猪／鱼人木乃伊／吸血兔）。 （明细：[detail-skins-loc.md](detail-skins-loc.md)）

## 适配检查清单（条件 → 动作）

| 如果你的模组… | 判断／动作 |
|---|---|
| 世界设置按 `few`/`many` 档名接线 | **必须适配（逐设置核对）**（[R-7](detail-root.md)）：40+39 个设置改名 `rare`/`often`，旧名静默无效；`petrification` 等未改名，**勿全局替换** |
| 持物右键对背包内物品施法（USEITEM spellcaster） | **必须适配**（[R-1](detail-root.md)）：仅此入口拒绝物品栏内目标，改对世界目标；EQUIPPED／POINT／INVENTORY 不变 |
| 消费 `VERSIONING.CURRENTVERSION` 字符串 | 建议调整（[R-5](detail-root.md)）：可能带 `.N` 子版本段 |
| 依赖 propweapon 非 PVP 友伤 | 检查武器是否挂 `propweapon_nopvp`（[C-2](detail-components.md)）；自制道具武器想退豁免就挂它 |
| 想给实体"换家"回调 | 用 `inst.homeseeker_onsethome`（[C-4](detail-components.md)） |
| 依赖 NPC 对 minhealth 实体不仇恨 | 自身或其跟随者首领带 `companion` 标签即放行仇恨（[C-1](detail-components.md)）；维持旧行为＝不加该标签 |
| 自制生物想接入击退 | 尸体态自动免击退（[SB-2](detail-sg.md)）；带 inventoryitem 的生物击退会自动落地（[SB-3](detail-sg.md)） |
| 自制"不可寄生"实体 | 设 `sg.mem.noparasitehost = true`（[P-22](detail-prefabs.md)） |
| 自制糖果／万圣交易物 | 挂 `tradable.halloweencandyvalue` 即接入双王（[P-6](detail-prefabs.md)） |
| 直调 `inst.TestGame`/`inst.CancelGame`（猪王） | 已移除（[P-9](detail-prefabs.md)） |
| 拦截 kramped 淘气计算 | 狼猪特判挪到 `NAUGHTY_VALUE.pigman` 函数；函数返回 0 跳过入账、**返回 nil 会报错**；旧版 0 值入账的掷阈值／警告音不再发生（[C-5](detail-components.md)、[R-2](detail-root.md)） |
| 对 leafpile 做焚化/掉落拦截 | leafpile 可燃，烧毁只掉 ash；treat 音效从拾取者播（[P-17](detail-prefabs.md)） |
| 拾取判定依赖击退落地 | 带 inventoryitem 实体击退中 SetLanded 状态正确（[SB-3](detail-sg.md)） |

## 分类明细导航

| 明细文件 | 覆盖 | 建议检索词 |
|---|---|---|
| [detail-root.md](detail-root.md)（R-1～R-8） | 根脚本 6 文件 | spellcaster、IsHeld、NAUGHTY_VALUE、HN_MONSTER、versioning_prefabs、rare/often、cave_vent_mite |
| [detail-components.md](detail-components.md)（C-1～C-8） | components/* 8 文件 | ShouldAggro、companion、propweapon_nopvp、carefulwalker Enable、homeseeker_onsethome、sweepsound |
| [detail-prefabs.md](detail-prefabs.md)（P-1～P-28） | prefabs/* 28 文件 | MakeFrankenPig、MakeMummyMerm、MakeVampireBunny、EnableFrankenPigHouse、ismummyhouse、hn candy、leafpile burnable、shadowlure、WASPHIVE_ENABLED |
| [detail-sg.md](detail-sg.md)（SB-1～SB-18） | stategraphs 15＋brains 5 | hn_walk、hn_attack、noknockback、SetLanded、animqueueover、fadeout、frankenpigbrain |
| [detail-skins-loc.md](detail-skins-loc.md)（M-1～M-4） | 语音 17＋本地化 3＋strings | Frankenswine、Mermummy、Vamphare、科学怪猪 |

## 新增与删除

- **新增（4）**：brains/frankenpigbrain.lua、brains/mummymermbrain.lua、brains/vampirebunnybrain.lua（SB-14～16）、versioning_prefabs.lua（R-6）。
- **删除（0）**：无。三怪物不新增 prefab 注册名（沿用 pigman／merm／bunnyman ＋ SetPrefabNameOverride）。

## 覆盖与审校状态

- 文件覆盖：**84/84**（A/M/D 全登记，逐文件条目见五份 detail；资源类按 M-1～M-4 归并）。
- 语义审校：**完成（2026-11-01）**——逻辑文件 63 个逐条读 diff、全部条目已核验；资源类（21 文件：strings＋17 语音＋3 本地化）按变更量与代表样例审校。
- 知识页迁移：**完成（2026-11-01，批次-1～3）**——573 受影响页升版并 record（批次-1 238＋批次-2 335）；381 个未受影响页按哈希等价批量沿用升版（批次-3，不重录基线）；全库 954 个带 sources 页 dst_build 均为 26335394，hash check 5,529 组对比 0 变化、affected_pages 0。批次页面清单见本目录 page-migration-batch1／2／3 三个 txt。

## Wiki 沿用记录

**批次-1（2026-11-01，26230496 → 26335394）**：迁移 238 页（波一零位移 25＋机械锚点 203＋语义修订 10）。

- **输入与快照**：旧 `raw/archive/build-26230496/scripts`，新 `raw/current/scripts`（BUILD.txt 26335394）。全检基线：5,528 组对比 893 变化、受影响 573 页（行尾/大小写/删除 0）。
- **波一（25 页）**：全部 sources 为零位移文件（fx.lua 6 处音效字串 1:1、combat.lua、combat_replica.lua、virtualroomset.lua、balatro_machine.lua、vaultcollision.lua、wurt.lua、bird_mutant_brain.lua 均行号不变）。三查：未决项标记 6 页均为边界声明（动画内容/未运行游戏）不阻断；符号反查（ShouldAggro/minhealth/companion/leaf_rustle/mummy）命中 5 页逐条排除（health.md 的 minhealth 为封顶语义、green_leaves.md 引 L364 树叶段未变区等）；升版＋record。
- **机械锚点（203 页）**：`plan_source_link_updates.py` 全 573 页规划——14,615 处引用：13,702 未变、740 机械提案全部应用（evidence=unique-equal-block，脚本按目标 URL 精确替换、复验 0 待处理）；review 级人工处理。批次内行为变化符号反查（MakeFrankenPig/homeseeker_onsethome/SetSweepSound/propweapon_nopvp/HN_MONSTER/cave_vent_mite/WASPHIVE_ENABLED/shadowlure/hn_* 等 20 符号）在本批 212 页零命中——无未声明依赖。升版＋record（237 刷新＋1 跳过）。
- **语义修订（10 页）**：carefulwalker（C-3：Enable/TrackTarget 门/OnRemoveFromEntity 修复；纠正上轮把 flyingbroom 判定记到 26230496 的注记——实为本 build）、sweeper＋sweepable（C-6/C-7：OnSweep 调用序与音效 API）、homeseeker（C-4：homeseeker_onsethome 钩子；修正 GetHomePos/GoHome 两处上版错位锚点）、kramped（C-5/R-2：淘气函数值＋>0 门）、spellcaster（R-1：IsHeld 拒绝）、virtualroomset（C-8 等价注记）、itemmimicconfig（P-10：king_cane 撤销揭示）、deflectable＋willow_shadow_flame（P-11：2D 近距判定/目标位置爆炸/ReduceAngle；修全库唯一 1 错——注册尾 L381-383→L378-380）。全部 validate_source_links 0 错（存量结构/空行软警告按引用意图保留）。
- **待续（余 335 页）**：① review 级单项目 6 页——spawner（pighouse 作息分流 P-2）、slipperyfeet（SGwilson careful 分支）、container-proxy（hats noparasitehost P-22）、piratespawner（worldsettings few/many 改名 R-7）、generic-fx-factory（fx.lua L4203 单行核验）、icebox_coffin_bat_fx（easing.lua 引用 source-read 异常）；② 其余约 329 页引用了变化文件但锚点未移（tuning／componentactions／SGwilson 等大文件命中页的未迁移部分）——planner 输出与逐批三查按本记录复用，下批延续。

**批次-2（2026-11-01，同起止）**：余量清零——上项待续全部落地。

- **review 单项 6 页**：spawner（pighouse 四处引用迁 L469-L479／L475-L479／L298-L321／L224-L256，补 SetOnKilledFn／OnStartDusk／IsFrankenPig 分流注记）；slipperyfeet（careful 分支补 flyingbroom 豁免注记，锚点同位）；piratespawner（pirate_raids 覆盖块补 rare/often 改名注记，行号 1:1 不变）；container-proxy（hats 段 +3 迁 L8035-L8038）；generic-fx-factory（fx.lua L4203 表尾单行核验内容未变，零修改）；icebox_coffin_bat_fx（早前 source-read 错误为快照切换瞬间暂态，复验 29 引用 0 错 0 警，零修改）。
- **机械沿用 335 页**：行为变化符号反查 30 项（含 NAUGHTY_VALUE／bodyfall_dirt／noknockback／CommonStates.AddIdle／SetLanded／IsCarefulWalking 等扩展符号）零命中；未决项标记（review-needed／待核验问题／证据不足／证据冲突）零命中、全部 active；planner 对此批 0 提案（引用区间全部未移）；升版 dst_build／updated＋record（334＋1 页此前已升）。
- **收尾核验**：全库 hash check 5,529 组对比 0 变化、affected_pages 0；animqueueover／bodyfall 全库命中页均为未变代码路径上下文（fx.lua 加载器、maxwelltalker 等自有逻辑），无需修订。

**批次-3（2026-11-01，同起止）**：381 个未受影响页（sources 所引文件本轮原始字节全部未变）哈希等价批量沿用。

- **机器确认**：全库 hash check（5,529 组）对其全部对比 0 变化——sources 集合与完整旧基线一致、字节未变；按批量沿用分支不重录基线。
- **筛查**：status 全部 active；review-needed／待核验问题／证据不足／冲突标记仅 2 页且均为边界声明（moisture、playertargetindicator，保留原记录）。sources 未变不等于无行为接入面——已对本轮行为变化符号做**全库定向反查**：`NAUGHTY_VALUE` 第二消费方 rabbitkingmanager.lua:448 经 `FunctionOrValue` 接入本轮改为函数的 bunnyman 键（吸血兔受害者传入 0——淘气值不增加但阈值检查与警告音照常执行，不等同 kramped 的直接跳过；正常兔人仍 3；其知识页已补语义注记并把 constants.lua 补入 sources 并重录基线，为本批定向修订例外），其余 30 个行为符号在 381 页零命中；结论已记入 R-2 条目与该页。
- **落地**：dst_build／updated 升版 26335394（381 页）＋验证记录插入沿用注记；清单见 `page-migration-batch3-unaffected.txt`。并行会话 5 个 icebox 页按当前快照建页与基线，不在本队列。
- **页面清单与页内注记**：本批（及批次-1）逐页清单见本目录 `page-migration-batch1.txt`（238 页）／`page-migration-batch2.txt`（335 页）；**全部 954 个带 sources 页的验证记录都含指向本节固定锚点的可读回链**（仓内 wikilink `[[updates/build-26335394/index#wiki-沿用记录]]` 短式，兼容 8KB／16KB 页面字节预算；sweepable、itemmimicconfig 以各自专项修订注记内的回链替代通用行，避免重复记录）。批次-2 筛查中 2 页有"待核验问题"章节（moisture、playertargetindicator）——均为探索范围边界声明，不影响已采纳结论，按协议保留原记录不阻断。
- 机器事实：`output/tmp/`（hashcheck/plan/link 中间件，不入库）；报告 changed-files.txt 为全集清单。
