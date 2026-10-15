---
title: "源码更新影响说明：build 25643519 → build 26148036"
created: "2026-10-24"
dst_build: "steam-buildid:26148036"
status: active
---

# 源码更新影响说明 build 26148036

快照 25643519 → 26148036（2026-10-24），大型内容更新。全库 491 个源码文件变动：427 改 / 59 增 / 5 删；哈希基线全检受影响知识页 318/345，git diff 独立对账与工具判定一致。

**版本与快照身份**：起始 build 25643519（旧快照 `raw/archive/build-25643519-20261024-190304-a7b132afb0e94d2bacc12d2688c4b098`，即先前 current）；目标 build 26148036（新快照 `raw/current`，BUILD.txt 同值）。本报告只覆盖这一起止对；其他起点另建报告。

**阅读方式（渐进式）**：本页只放主题与决策规则，逐文件明细在 5 个 detail 文件。兼容性相关条目带稳定编号（C／SB／R／M／P-序号），适配结论可直接引用编号。适配判断三步：① 读下方适配检查清单，命中即得动作；② 未命中但涉及变化机制的，按主题尾部指针打开对应 detail 文件，按组件名／函数名／事件名 grep 定位行；③ detail 条目均出自实际 diff（个别标"意图待核验"处除外）。机器事实（changed-files.txt、check-report.json）在本目录；页面处理队列用 `validate_source_hashes.py check --fresh-hours 0` 的 affected_pages 重建，页面核验状态以各页 dst_build 为准。

## 本轮主题（模组影响分诊）

- **vault → virtualroom 房间系统重做**：`vaultroommanager`/`vaultroom`/`vault_teleporter`/`vault_floor_helper` 组件删除，新 `virtualroommanager`/`virtualroomset`/`virtualroomteleporter` 组件族 + `worldstaticlayouts`；事件改名（`ms_register_vault_*` → `ms_register_virtualroom_entity`）；`vaultroom_defs` 整体重写为 19 房间布局声明式连接图；存档升级 5.25 自动迁移。挂接 vault 事件／组件／存档数据的模组必须迁移。 （明细：`detail-components.md`、`detail-prefabs.md`）
- **受击 proc 门控工具化**：componentutil 新增 `ShouldProcOnAttackedOrBlocked`（data 非空＋非重定向 `data.redirected`＋攻击者存在＋攻击者或其物品最终持有者 ≠ 受害者自身）、`IsEquipmentOnAttackedOrBlocked`（攻击者带 inventoryitem、GetGrandOwner()==owner 且带 `nodangermusic`——不查装备栏，勿与 combat/rider 的 IsItemEquipped 混同）、`IsRangedWeapon`、`IsLifeDrainable`；combat 事件新增 `from_doattack` 数据。**各消费者组合不同**：护甲/护符反伤与 itemmimic 走 ShouldProc…，singinginspiration 走 IsEquipment…，wolfgang 等的 OnHitOther 另要求 `from_doattack`（combat 内部结算标记，反伤也带）——自定义 proc 的模组按自身语义对齐对应判定，勿照抄他人条件。 （明细：`detail-components.md`、`detail-root.md`、`detail-prefabs.md`）
- **弹道偏转系统**：`complexprojectile` 用 `SetOnDeflect(fn)`，`projectile` 用 `SetOnDeflectFn(fn)`（各有 `SetKeepOnDeflect`/`Deflect`，**默认偏转后仍移除弹体**），新增 `deflectable` 组件供 debuff/fx 类实体声明可偏转；火把、哑铃、回旋镖、凝胶弹、影焰等接入。攻击类模组可接偏转回调。 （明细：`detail-components.md`、`detail-prefabs.md`）
- **eater 饮食接口 Deprecated**：`SetCanEatHorrible/Gears/Nitre/Raw` 源码标记弃用，迁移方向 `SetDiet`（本轮大量 prefab 已迁移，行为等价）；新增 `ResetEdibleTagsCache`。波及页 [reference/components/eater](../../reference/components/eater.md)。 （明细：`detail-components.md`）
- **新 boss 与腐化体系**：Charlie boss 试炼（atrium_gate 仪式状态机、双钥匙、charliearena）、shrouden 召唤体（四大 shadowthrall 新增 shrouden_* 状态）、rocky/bat boss 重做、worm_boss 暗影变体（吞噬/硬直）、NPC stalker 腐化系统（`stalker_corruption_*` 状态族 + `stalkerinspectable` 检视族）、king_cane／queen_torch。boss 战相关模组注意新硬直（stun_*）、`shadowboss` 排除标签与竞技场排除逻辑。 （明细：`detail-sg-brains.md`、`detail-prefabs.md`）
- **标签族扩展**：`plantcreature`（植物生物判定，十余生物）、`canwearhat`（可戴帽，slurper 帽化目标泛化）、`nightmarecorruptable`（可被王杖腐化）、`nolifedrainable`/`nolife`（吸血免疫）、`shadowboss`。 （明细：`detail-prefabs.md`）
- **模组 API 与兼容**：`CONTROL_CUSTOM_START` 由 102 上移到 103（引用常量自动跟随，硬编码旧起点 102 的才需 +1）；`EncodeSkillTreeData` 新增可选参（缺省行为与旧版等价，已核验）；`ChatHistory:AddToHistory`/`OnChatterMessage` 新增 silent 参数；`npc_talker:Say/Chatter` 新增回调；`JoinServerResponse` 新增第 4 参；`CanDownloadWorkshopMods()`；发行位新增 R43_SHADOWYDEPTHS；deciduous 世界频率设置修正对既有季节概率字段的写入（[R-18](detail-root.md)）；`SaveFileUpgrades` 5.24–5.27。 （明细：`detail-root.md`）
- **staff 法杖重构**：六系法杖逻辑迁至新 `staff_common.lua`（经典六杖行为基本不变；绿杖缠绕目标过滤新增 `no_deconstruction` 排除；覆写法杖逻辑的模组需改指向）。 （明细：`detail-prefabs.md`）
- **资源与表现（一笔带过）**：本地化 languages/ 约 2.1 万行、strings／skin 系列（release_group 186）、rifts8 音效包、牛音效路径迁移（`rifts4` → `dontstarve`）、postprocesseffects 屏幕扭曲框架、触发式音乐表。

## 适配检查清单（条件 → 动作）

模组适配判断从这里开始：命中即得动作；未命中再按主题指针查 detail。

| 如果你的模组… | 判断／动作 |
|---|---|
| 包装／覆盖 `Combat:GetAttacked` 做伤害拦截 | 必须适配（[C-10](detail-components.md)）：内部路径已绕过包装，事前拦截改挂 `GetAttacked_Internal`（改事件监听不等价）；依赖返回值的改调 `_Internal` |
| 按键位把旧 `CONTROL_CUSTOM_START`（102）作为起点硬编码 | 起点上移到 103，这组键位需要 +1；引用常量的自动跟随（`constants.lua`） |
| 调用或覆写 eater 的 `SetCanEat*` | 已 Deprecated，迁移到 `SetDiet`（旧接口暂有兼容包装）（[C-12](detail-components.md)） |
| 覆写 `staff.lua` 内部逻辑 | 逻辑迁至 `staff_common.lua`，改指向；经典六杖行为基本不变，绿杖缠绕目标过滤有新排除（`no_deconstruction`） |
| 直接调用 `blinkstaff:Blink` 并依赖旧 quicktele 拒绝规则，或把 `onblink` 当到达通知 | 前者需自行补门禁；事件是开始阶段通知，不能证明已经到达（[C-05](detail-components.md)） |
| 监听 `ms_register_vault_*` 事件、读 `vaultroommanager` 数据 | 必须迁移 `virtualroom*` 新接口；旧存档数据已由升级 5.25 自动迁移（[P-122](detail-prefabs.md)、[P-123](detail-prefabs.md)） |
| 装备／护甲／武器有受击触发效果（反伤、充能、诅咒、灵感等） | 新门控按消费者而异：`ShouldProcOnAttackedOrBlocked` 过滤重定向与"自己装备打自己"；`IsEquipmentOnAttackedOrBlocked` 过滤满足【攻击者带 inventoryitem、GetGrandOwner()==owner 且带 `nodangermusic`】的攻击（不查装备栏）；部分消费另要求 `from_doattack`。对照你的触发语义选对应判定 |
| 用 `weapon.projectile` 等手写判定远程武器 | 改用 `IsRangedWeapon`；伪远程可加 `pseudorangedweapon` 标签 |
| 吸血／生命吸取效果针对建筑或植物 | `lifedrainable` 大面积移除，判定改 `IsLifeDrainable` ＋ `nolifedrainable`／`nolife` 标签 |
| hook `player_common` 的虫胃音效／吞噬表现 | 字段迁至 `player_classified.wormdigestionsound`（net_bool 同步）（[P-82](detail-prefabs.md)、[P-83](detail-prefabs.md)） |
| 投掷物／攻击希望被格挡弹回 | 接 `deflectable` 组件或 `SetOnDeflect`／`SetKeepOnDeflect`（默认偏转后仍移除弹体，需保留弹体才设 keep） |
| 生成点／索敌／传送逻辑引用金库房间 | `IsPointInVaultRoom` 系改 `IsPointInVirtualRoomSet(VIRTUALROOMSETS.*)` |
| 自订 deciduousmonster（多汁树怪）世界频率 | 修正对既有季节概率字段的写入（旧写入键未被消费，自定义频率现按季节生效，[R-18](detail-root.md)）；customize 选项名未变 |
| 直接调用 `SkillTreeData:EncodeSkillTreeData` | 新增可选第二参，缺省行为与旧版等价（已核验），无需适配（[R-17](detail-root.md)） |
| 监听 `haunted` 事件 | 建议调整（[C-17](detail-components.md)）：现仅成功作祟后触发 |
| 自定义 SG 依赖 busy 期间不转向 | 行为已变化（[C-21](detail-components.md)）：locomotor 不再因 busy 跳过转向 |
| 调用／包装 charlie_npc 的 `StartCasting`／`Despawn` | 必须适配（[P-26](detail-prefabs.md)）：方法已删除，改走事件流或 StartCastingWithDelay |
| 包装 `Rechargeable.OnUpdate` 改充能 | 留意 LongUpdate 别名分叉（[C-32](detail-components.md)）：补算时间不走包装，需两路同改或换参数接口 |
| 直接构造 `ChattyNode` | 新增第 8 参 data 为可选，Lua 侧缺省等价（已核验） |
| 调用 `TheNet:JoinServerResponse` | 传参变化已核验（新增第 4 参 row）；缺省兼容性待核验（原生绑定处理未验证，不下结论） |
| 监听聊天历史（ChatHistory listeners） | `silent` 消息仍入历史但不通知 listeners（[R-03](detail-root.md)） |
| 湿度／降雨／淋湿相关逻辑 | 新增 `Map:CanPointHaveRain` 区域检查（虚拟房间、室内不再受潮、不再继承湿度） |
| boss 战相关（目标选择、碰撞、音效、竞技场） | 注意 `shadowboss` 排除标签、boss `stun_*` 硬直体系与查理竞技场各项排除 |
| 小地图／图鉴 | 图集新增第三张（minimap_data3）、`alwaysoutsidevrs` 字段、图鉴收录范围调整 |

## 分类明细导航

逐文件明细在 5 个 detail 文件（全部出自实际 diff），按名 grep 定位行，不必整读。条目核验状态：兼容性条目标"已核验"者为逐项核对；未标注者为**待审**（已读文件级 diff，接入面未逐项展开），只能作候选结论：

| 明细文件 | 覆盖 | 建议检索词 |
|---|---|---|
| [detail-components.md](detail-components.md) | 组件 54 | 组件名（combat／eater／map…） |
| [detail-sg-brains.md](detail-sg-brains.md) | 状态图 28 ＋ 大脑 13 | SG 名／boss 名／状态名 |
| [detail-root.md](detail-root.md) | 根脚本 47 | API／常量／动作／存档／tuning |
| [detail-misc.md](detail-misc.md) | 地图 10 ＋ UI 16 ＋ 行为节点 3 ＋ util 1 | 布局／界面／输入框 |
| [detail-prefabs.md](detail-prefabs.md) | 预制物 221 | prefab 名（大文件，务必 grep） |

## 资源文案文件（34 个，一笔带过）

`languages/` 下 10 个翻译 PO 文件约 2.1 万行；`clothing.lua`/`misc_items.lua`/`prefabskins.lua`/`prefablist` 皮肤注册（release_group 186）；`strings.lua` 新实体文案；`skin_*` 系列；`speech_wormwood.lua` 台词扩充。**审阅范围**：翻译与皮肤注册数据按低风险处理未逐条审；`skin_*` Lua 脚本与资源路径注册可能有兼容影响，需要时按 changed-files.txt 定向核对。

## 新增文件（59 个）

纯新增默认不影响旧代码；**承接既有逻辑迁移的共享模块除外**（`staff_common.lua`、`aoeutil.lua` 改变 hook 路径，见 detail 相应条目）。14 个新组件是知识扩库候选——**不属于旧版冻结的 98 项清单**（41/98、剩余 57 的口径不变），另立新版扩库队列认领，不自动建页：atriumritualorgantracker、autojiggle、charlie_tracker、charliearenawatcher、corruption、deflectable、rockybossspawner、shard_charlieinfo、stalkerinspectable、stalkermanager、virtualroommanager、virtualroomset、virtualroomteleporter、worldstaticlayouts。其余 45 个为新内容 prefab／SG／布局（charlie_boss 系列、shrouden、rocky_boss、bat_boss、lobbyvaultroom_defs 等），完整清单见 changed-files.txt。

## 删除文件（5 个）

`components/vault_floor_helper.lua`、`components/vault_teleporter.lua`、`components/vaultroom.lua`、`components/vaultroommanager.lua`、`stategraphs/SGwx78_possessedbody_no_package.lua`。均未被知识页引用，但**按接入影响评估**：模组若对 vault 系 prefab 调 `AddComponent("vaultroommanager")` 等将直接报错，需迁移 `virtualroom*` 组件族（见本轮主题第一条）。

## Wiki 沿用记录

| 日期／版本 | 页面 | 共享依据与范围 |
|---|---|---|
| 2026-10-25，25643519 → 26148036 | [cyclable](../../reference/components/cyclable.md)、[harvestable](../../reference/components/harvestable.md)、[upgradeable](../../reference/components/upgradeable.md) | [R-01／R-04](detail-root.md) 的动作变化不触及所用 CYCLE／HARVEST／UPGRADE；[P-111](detail-prefabs.md) 仅移除蜘蛛巢 lifedrainable，不触及所用升级机制。组件及其他字节未变的来源沿用；actions.lua／componentactions.lua 和 spiderden.lua 仅沿用所引相同代码块并重定位，不称这些文件哈希未变。未重审整页源码或进行新版游戏验证。 |
| 2026-10-25，25643519 → 26148036，批次标识「原始哈希等价批量沿用-24」 | [chargingreticule](../../reference/components/chargingreticule.md)、[curseditem](../../reference/components/curseditem.md)、[deployhelper](../../reference/components/deployhelper.md)、[domesticatable](../../reference/components/domesticatable.md)、[dryingracksaltcollector](../../reference/components/dryingracksaltcollector.md)、[entitytracker](../../reference/components/entitytracker.md)、[farmplantstress](../../reference/components/farmplantstress.md)、[groomer](../../reference/components/groomer.md)、[inspectable](../../reference/components/inspectable.md)、[insulator](../../reference/components/insulator.md)、[inventory](../../reference/components/inventory.md)、[named](../../reference/components/named.md)、[pethealthbar](../../reference/components/pethealthbar.md)、[propagator](../../reference/components/propagator.md)、[prototyper](../../reference/components/prototyper.md)、[recallmark](../../reference/components/recallmark.md)、[resistance](../../reference/components/resistance.md)、[saltlicker](../../reference/components/saltlicker.md)、[slipperyfeettarget](../../reference/components/slipperyfeettarget.md)、[temperatureoverrider](../../reference/components/temperatureoverrider.md)、[thief](../../reference/components/thief.md)、[tradable](../../reference/components/tradable.md)、[witherable](../../reference/components/witherable.md)（23 页机械沿用）＋[linkeditem](../../reference/components/linkeditem.md)（转常规修订） | 沿用分支「原始哈希等价的批量沿用」：机器确认 24 页 sources 集合与完整旧基线一致且所列每个文件原始字节哈希未变（sources_changed=0、line_ending_only=0），旧结论及行号直接沿用。页内未决项筛查与枢纽反查（组件／事件／被调接口 × 491 变更路径 × 报告条目，同名近似逐个以实际 diff 排除）中 23 页无命中；linkeditem 命中 [C-19](detail-components.md)（GetGrandOwner 回退中间持有者 × 弹弓换绑：至少装有一个部件且在无人持有容器内触发换绑时从维持原绑定变为解绑），转主流程修订并把 inventoryitem.lua／slingshotmods.lua 纳入 sources 后单页 record。其余 23 页批量升版 dst_build／updated，不逐页 record、不重写未变基线。本批 24 页锚点 669 处 0 错 0 警（`--strict`）；未重审整页源码、未运行游戏。旧新快照：`raw/archive/build-25643519-20261024-190304-a7b132afb0e94d2bacc12d2688c4b098` → `raw/current`。 |

## 页面处理

初次检查有 318 页进入哈希漂移队列；当前进度以各页 dst_build、来源基线及更新导航为准。完整候选、批量沿用与定向复查按知识 Skill 的版本增量维护规则执行，affected_pages 不单独代表全部维护候选。本报告不另设逐页重读／record 规则。初次检查时 beard／brushable 的 SGBeefalo.lua 引用大小写已修复，当时尚未核验内容；这不是当前两页状态保证。
