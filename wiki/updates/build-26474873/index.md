---
title: "源码更新影响说明：build 26335394 → build 26474873"
created: "2026-11-08"
dst_build: "steam-buildid:26474873"
status: active
---

# 源码更新影响说明 build 26474873

快照 26335394 → 26474873（2026-11-08），万圣夜怪物化收尾＋漂浮动画修复小型更新。全库 49 个源码文件变动：49 改 / 0 增 / 0 删；哈希全检 1,200 个带 sources 页面、9,756 组对比中 813 组真变化，受影响页面 561（行尾 0、大小写 0、删除 0、来源集合变化 0）。

**版本与快照身份**：起始 build 26335394（旧快照 `raw/archive/build-26335394`，即先前 current，2026-11-08 归档）；目标 build 26474873（新快照 `raw/current`，BUILD.txt 同值，4,095 个 .lua）。本报告只覆盖这一起止对；其他起点另建报告。

**阅读方式（渐进式）**：本页只放主题与决策规则，逐文件明细在 6 个 detail 文件。兼容性相关条目带稳定编号（C／R／P／SB／U／M-序号），适配结论可直接引用编号。适配判断三步：① 读下方适配检查清单，命中即得动作；② 未命中但涉及变化机制的，按主题尾部指针打开对应 detail 文件，按组件名／函数名／事件名定位；③ detail 条目均出自实际 diff（全部已核验，无待核验条目）。机器事实（changed-files.txt）在本目录；页面处理队列用 `validate_source_hashes.py check --fresh-hours 0` 的 affected_pages 重建，页面核验状态以各页 dst_build 为准。

## 本轮主题（模组影响分诊）

- **MakeInventoryFloatable 批量补 swap_data（26 文件，表现修复）**：amulet、armor_bramble、beargervest、beef_bell、books、glommerflower、hawaiianshirt、hermithouse_ornament、icepack、piggyback、portableblender、portablecookpot、portablespicer、raincoat、records、reflectivevest、saddle、slingshot、spicepack、trap、trap_teeth、trunkvest、wx78_scanner、winona_catapult、winona_spotlight、plantables 全部给 `MakeInventoryFloatable` 传第 7 参 `swap_data={bank,anim}`。floater 组件未改：物品换肤的 `basic_init_fn`／`basic_clear_fn`（仅物品正在漂浮时）与 fumarole 工具破损恢复路径此前用 prefab 名＋"idle" 强制恢复地面动画，皮肤物品显示错；现在用正确 bank/anim（`SetSkinsOnAnim` 属人物皮肤／服饰路径，与本改动无关）。数值与机制结论不变。 （明细：[detail-prefabs.md](detail-prefabs.md) P-1）
- **万圣夜怪物化收尾（4 文件）**：猪人（`OnInit`）与兔人（`OnInitHallowedNights`，且仅洞穴）的初始化掷骰新增 `not inventory:HasAnyEquipment()` 门（装备中不变身）；鱼人没有独立掷骰门，仍由鱼人屋 `OnSpawned` 按 `ismummyhouse` 调 `MakeMummyMerm`。三种怪物的 `AddChanceLoot` 六连都收进活动期门内，登记发生在变身（或读档重建变身态）时——活动期外变身不登记万圣掉落；死亡结算 `GenerateLoot` 直接消费已登记项、不再回调活动开关。pigman/bunnyman/merm 新增 `OnLoadPostPass` 在变身态读档后卸装；mermhouse `OnGoHome` 判定由 `and` 改 `or`（普通屋须在回调执行时屋外孩子数为 0 且非冬季才尝试重启——`TakeInChild` 先调 OnGoHome、后从屋外名单移除该孩子，回调瞬间仍会数到它）、旧存档 retrofit 只在存档缺 `data.ismummyhouse` 且当前实例已判为木乃伊屋时挂钩。 （明细：[detail-prefabs.md](detail-prefabs.md) P-3~P-6、[detail-root.md](detail-root.md) R-3）
- **鱿鱼群生成修正（3 文件）**：`squidspawner` 判相从 isday/isnight/isdusk 派生布尔改读 `phase` 字符串（phase 变更事件内派生布尔未更新，官方注释确认）；新 `TUNING.SQUID_MIN_COUNT`（默认 2）控单群投放的尝试次数下限（每次找到可游泳落点才排入生成任务，不是保证生成量），世界设置五档各补值（never 0/rare 1/default 2/often 3/always 3），always 档满月抽样上限 2→3。 （明细：[detail-components.md](detail-components.md) C-2、[detail-root.md](detail-root.md) R-1/R-2）
- **杂项行为修补**：hounded 存档恢复早退分支补 `_attackplanned=true`（存档标记为假／缺失而倒计时仍 >0 时，**旧版**读档后袭击不推进；26474873 已在该分支补记）；月盲鸦破坏玩家骨架半径 1.25→1.3＋行为树重组（无骨架跳过分支、移除 AttackWall）；werepig hit 态补 hit-recovery 延迟；mossling 补 drownable＋沉没/虚空态；暗影百家蚁恢复受击音效（去 noimpactsound）；witchbroom 起飞音效替换（双端）。 （明细：[detail-components.md](detail-components.md) C-1、[detail-sg.md](detail-sg.md) SB-1~SB-5、[detail-prefabs.md](detail-prefabs.md) P-2/P-7）
- **界面与图鉴**：地图开／关新增音效 map_open/map_close；图鉴白名单＋装饰 7-10、turf_vent，数据行补万圣掉落列、leafpile 可燃标记、witchbroom 耐久字段改 finiteuses=125。 （明细：[detail-ui.md](detail-ui.md) U-1/U-2、[detail-root.md](detail-root.md) R-5）
- **资源与表现（一笔带过）**：waterballoon 皮肤 build 名修正（swap_waterballoon→waterballoon）；stalker 台词句号补齐（strings.lua＋pot 同步）。 （明细：[detail-root.md](detail-root.md) R-4、[detail-skins-loc.md](detail-skins-loc.md) M-1/M-2）

## 适配检查清单（条件 → 动作）

| 如果你的模组… | 判断／动作 |
|---|---|
| 给猪／兔人／鱼人穿装备或监听其掉落 | **检查行为变化**（[P-3](detail-prefabs.md)~[P-6](detail-prefabs.md)）：猪／兔人装备中不参与初始化掷骰（鱼人无独立掷骰，由鱼人屋生成路径控制）；活动期外变身不登记万圣掉落，死亡按已登记项结算；读档后变身个体自动卸装（OnLoadPostPass 时序：持有者收 `unequip`／`dropitem`，物品收 `ondropped`） |
| 维护含木乃伊屋的旧存档逻辑 | 检查（[P-6](detail-prefabs.md)）：OnGoHome 判定放宽（木乃伊屋回屋即重启生成；普通屋仅当回调瞬间屋外孩子数为 0 且非冬季才重启——`TakeInChild` 先调 OnGoHome、后从屋外名单移除该孩子，回家这只回调时仍在名单）；retrofit 仅在存档缺 `data.ismummyhouse` 且当前判为木乃伊屋时挂钩，带装备孩子回退非木乃伊屋 |
| 自判昼夜相（在 phase 变更监听内） | 建议调整（[C-2](detail-components.md)）：改读 `_worldstate.phase` 字符串，派生布尔（isday 等）在监听时机未更新 |
| 消费 `TUNING.SQUID_*` 或鱿鱼世界设置档位 | 建议调整（[R-1](detail-root.md)／[R-2](detail-root.md)）：新键 `SQUID_MIN_COUNT` 是投放尝试次数下限（档位 0/1/2/3/3，非保证生成量）；always 档满月抽样上限 2→3 |
| 消费 `VERSIONING.CURRENTVERSION` 字符串 | 建议调整（[R-3](detail-root.md)）：子版本段 `.9`→`.10` |
| 给 waterballoon 类物品做皮肤或皮肤初始化 | 检查（[R-4](detail-root.md)／[P-1](detail-prefabs.md)）：皮肤 build 名修正；自制物品走 `basic_init_fn`／`basic_clear_fn` 换肤（或自建同类强制恢复）且 bank≠prefab 名／地面 anim≠"idle" 时应补 floater `swap_data` |
| 让自制生物落水／掉虚空有通用表现 | 参考接入（[SB-3](detail-sg.md)／[P-2](detail-prefabs.md)）：`drownable` 组件＋监听 `onsink`／`onfallinvoid`（处理器仍查存活、状态与 `ShouldDrown()`／`ShouldFallInVoid()` 环境判定）＋`AddSinkAndWashAshoreStates`/`AddVoidFallStates` |
| 依赖 hounded 读档后袭击推进 | 行为修复（[C-1](detail-components.md)）：读档后 `_attackplanned` 正确置位，`GetAttacking()` 恢复可用 |
| 替换或扩展地图界面 | 检查（[U-1](detail-ui.md)）：开／关音效直接 `PlaySound`，无事件可监听 |
| 读 scrapbookdata 表或 witchbroom 耐久数据 | 建议调整（[U-2](detail-ui.md)）：5 个新图鉴条目；witchbroom 耐久字段 fueled 族→`finiteuses=125` |

## 分类导航

| 明细文件 | 覆盖 | 建议检索词 |
|---|---|---|
| [detail-root.md](detail-root.md) | R-1~R-5：tuning／worldsettings_overrides／versioning_prefabs／prefabskin／scrapbook_prefabs | SQUID_MIN_COUNT、VERSION、swap_waterballoon、水球皮肤 |
| [detail-components.md](detail-components.md) | C-1~C-2：hounded／squidspawner | _attackplanned、PlanNextAttack、phase、鱿鱼判相 |
| [detail-prefabs.md](detail-prefabs.md) | P-1（26 文件 swap_data 批量）~P-7 | MakeInventoryFloatable、swap_data、怪物化、HasAnyEquipment、OnLoadPostPass、木乃伊屋 |
| [detail-sg.md](detail-sg.md) | SB-1~SB-5：SGwilson(+client)／SGwerepig／SGmossling／SGkrampus／bird_mutant_brain | run_witchbroom_pre、UpdateHitRecoveryDelay、AddSinkAndWashAshoreStates、月盲鸦 |
| [detail-ui.md](detail-ui.md) | U-1~U-2：mapscreen／scrapbookdata | map_open、map_close、图鉴数据、finiteuses |
| [detail-skins-loc.md](detail-skins-loc.md) | M-1~M-2：strings.lua／strings.pot | STALKER_NPC_PERFORM_CORRUPT |

## 资源／新增／删除

- **新增预制物**：0（`halloween_ornament_7~10`、`turf_vent` 的图鉴收录为既有 prefab 补登记，非新注册）。
- **删除**：0 文件；代码删除仅 shadowthrall_centipede 的 `noimpactsound` 一行与注释行。
- **资源**：本地化仅台词句号（M-1/M-2）；音效路径新增 `hallowednights2025/broom/broom_takeoff`（SB-1）、`dontstarve/HUD/map_open`／`map_close`（U-1）；皮肤数据 waterballoon build 名修正（R-4）。

## 审校状态

- **文件覆盖**：49/49（detail 条目路径 ∪ 资源清单 = 修改全集；对账 2026-11-08）。
- **语义审校**：全部条目已核验（逐文件读 diff；floater 消费链、versioning 拼接、怪物化门禁与掉落门均查证消费者与注册点；未运行游戏）。
- **知识页迁移**：完成 561/561（2026-11-08）：批V-01／02／08／09 117 页＋本轮升版批 V-03~V-07 与锚点批 V-09-rest／V-10／V-11／V-12 共 444 页；全库哈希 check affected_pages＝0、严格源码链接 0 error，详见下方沿用记录。

## Wiki 沿用记录

共享依据：本报告全部条目（2026-11-08，49 文件 diff 逐文件核验，全部已核验）；旧快照 `raw/archive/build-26335394`，新快照 `raw/current`（26474873）。迁移队列以 `validate_source_hashes.py check --fresh-hours 0` 的 affected_pages（561 页）为准；页面升版格式见各页「验证记录」末行。锚点预览输入：`output/update-26474873/anchors/`（page-index.json、review-resolutions.json、batch*.json）。

- **批V-01（2026-11-08，主代理直接执行，9 页）**：语义修订 2 页——squidspawner（判相改读 `phase` 字符串＋`SQUID_MIN_COUNT`（尝试次数下限）键与五档值，依据 C-2/R-1/R-2；锚点迁移 8 处）、hounded（`PlanNextAttack` 存档恢复补 `_attackplanned`，依据 C-1；锚点迁移 19 处）。特殊处理 7 页——5 个 CRLF FX 页（terrarium_fx、eye_charge、ruinsnightmare_horn_attack、wagboss_missile_target_fx、wx78_taser_projectile_fx，锚点共 10 处，全部按原字节格式宿主编辑、CRLF 保持）、houndwarning_lvl1（锚点 5 处；其 easing.lua 引用保留不动——该文件两快照逐字节一致，锚点工具 source-read 为 ISO-8859 解码限制）、pointofinterest（scrapbookdata 表 pigking 行 1116→1120，行内容未变）。validate_page 9/9 通过。
- **批V-02（2026-11-08，升版代理，60 页）**：reference/components 组件页（activatable～sweepable），机械沿用（引用区间未受行号位移影响），锚点零修改；validate_page 60/60 通过、0 issue。页清单 `output/update-26474873/batches/V-02.txt`。
- **批V-08（2026-11-08，锚点代理，40 页）**：reference/components 40 页，锚点迁移 79 处（77 条 proposals 全命中＋2 条复核裁决）；validate_page 40/40 通过。三项复核裁定（主代理）：dataanalyzer／linkeditem 的 review 区间内新增行均为 P-1 的 swap_data 局部变量（非 netvar、非组件挂载），页面断言不受影响；aoespell 两条先天截断链接（无右括号）补全并迁移（slingshot.lua L786／L859，内容经两快照核对等块）。
- **批V-09（2026-11-08，锚点代理，8 页完成）**：lunarhailmanager（锚点 2 处，LUNARHAIL 常量块 +1 等块平移）＋oar／oceanfishingrod／oceantrawler／pushable／regrowthmanager／repairable（实属 V-05 清单）＋schoolspawner（实属 V-06 清单）——后 7 页因主代理派发清单笔误提前处理，处理内容与升版批完全一致（均无需改动），已从 V-05/V-06 清单剔除，不重做。**笔误与回退说明**：V-09 派发文本含 24 个不存在的页名（主代理笔误），代理按纪律拒绝创建；另 7 页（moistureimmunity、playeractionpicker、playercontroller、quaker、rechargeable、scaler、shadowlevel）不引用任何变动文件、本不应升版，已由主代理完整回退（git 工作区对该 7 页零残留）。
- **批V-03（2026-11-08，升级执行，60 页）**：reference/components 11／mechanisms 1／prefabs 48，机械沿用（引用区间未受本次源码行号位移影响），锚点零修改；validate_page 60/60 通过。
- **批V-04（2026-11-08，升级执行，60 页）**：reference/components 47／prefabs 12／cookbook 1，机械沿用；validate_page 60/60 通过。
- **批V-05（2026-11-08，升级执行，54 页）**：reference/components 54，机械沿用；validate_page 54/54 通过。
- **批V-06（2026-11-08，升级执行，59 页）**：reference/components 32／prefabs 27，机械沿用；`farm-crops-fruit` 因追加沿用记录后越过 standard 档 32 KiB 字节预算，改判 `complex`（480 行／64 KiB）并在页内记分档依据；validate_page 59/59 通过。
- **批V-07（2026-11-08，升级执行，58 页）**：reference/prefabs 58，机械沿用；`farm-crops-staple` 同因 32 KiB 预算改判 `complex`；validate_page 58/58 通过。
- **批V-09-rest（2026-11-08，锚点执行，39 页）**：reference/components 39，锚点迁移 70 处，proposal 与复核裁决全部命中、0 未命中；validate_page 39/39 通过。
- **批V-10（2026-11-08，锚点执行，40 页）**：reference/components 9／prefabs 31，锚点迁移 142 处（含 5 页 changed-or-cross-hunk 裁决扩 1 行）；validate_page 40/40 通过。
- **批V-11（2026-11-08，锚点执行，40 页）**：reference/prefabs 40，锚点迁移 152 处（含 10 页复核裁决）；validate_page 40/40 通过。
- **批V-12（2026-11-08，锚点执行，34 页）**：reference/prefabs 34，锚点迁移 99 处；validate_page 34/34 通过。
- **需主代理复核（2 项，正文未改）**：① `icepack` 页「floater（`small`/0.15，无 swap_data）」与 P-1 新增 `swap_data = {bank = "backpack1", anim = "anim"}` 冲突；② `amulets` 页公共段断言 `MakeInventoryFloatable("med", nil, 0.6)`，新源码为 `MakeInventoryFloatable(inst, "med", nil, 0.6, nil, nil, swap_data)`（`{bank = "amulets", anim = anim}`）。另有 4 页（armor-boss／clothing／pouch-bags／ranged-weapons）引用区间覆盖 P-1 改动行但正文未就浮水参数断言，未改。`shadow_trap` 两处裁决串 `#L2193-L2186` 为倒序笔误（原页 `#L2186-L2186`），按同格式修正为 `#L2193-L2193`。
- **完成状态（2026-11-08）**：**561/561 全部迁移完成**。444 页 validate_page 444/444 通过、0 阻塞、0 未命中；收口基线 record（1,251 页／9,932 组对比）后 `validate_source_hashes.py check --fresh-hours 0` affected_pages＝0；全库严格源码链接 0 error（收口时 37,453 处引用；并行会话此后的新增页会使计数继续增长）（33 warning 均为既有 structural／blank 类，warning 页或本轮未改、或锚点零修改，非本轮引入）；抽查 5 页（monkey_cursed_pre_fx／oceanfishable／archive_ambient_sfx／cutless／staff_lunarplant_fx）锚点落点与正文对应正确。页清单 `output/update-26474873/batches/`，逐页执行记录 `result-V-*.json`、迁移计划 `plan.json`、改动行映射 `changed-hunks.json`、重叠复核 `overlap-review.json`。
- **迁移后语义修订（2026-11-08，主代理执行，3 页）**：按 [P-1](detail-prefabs.md) 修正浮水断言——`amulets`（commonfn 未下沉者调用补第 7 参 `swap_data = {bank = "amulets", anim = anim}`）、`icepack`（“floater（small/0.15，无 swap_data）”改为 `{bank = "backpack1", anim = "anim"}`，调用串两处）、`piggyback`（补 `{bank = "piggyback", anim = "anim"}`）；`swap_bank`／`float_index` 为 nil 时 floater 走 `SetSwapData`（standardcomponents.lua L1772-L1791，三页均加锚点）。三页 validate_page 通过、strict 178 引用 0 错 0 警；上方两项复核已闭环，另 4 页（armor-boss／clothing／pouch-bags／ranged-weapons）复核确认正文未就浮水参数断言、未改。
- **P2 复核修正（2026-11-08，外部复检 6 项）**：① `squidspawner` 页与 C-2／R-1／R-2 明示 `SQUID_MIN_COUNT` 是**投放尝试次数下限**（每次尝试找到可游泳落点才排生成任务，非保证数量），盈月 `max=2` 只是「附近已有 squid 数」门槛、单群抽样仍用 `moonphase_max`，并注明默认档满月等 `moonphase_max < SQUID_MIN_COUNT` 组合下强制入口会走到 `math.random(2, 0)` 空区间；② `hounded` 页与 C-1 收窄修复范围——旧版正常读取存档 `attackplanned=true`，仅「标记为假／缺失且倒计时仍 >0」的早退分支补记；③ 万圣摘要与 P-3~P-5 拆分装备门（仅猪人 `OnInit`／兔人 `OnInitHallowedNights` 且仅洞穴，鱼人由 mermhouse `OnSpawned` 控制）与掉落门（变身／读档重建时登记，死亡 `GenerateLoot` 直接消费已登记项），并补兔人仅洞穴；④ P-6 补普通屋的重启分支（该分支的回调时序判断经二次复核改正，见下条），retrofit 仅「存档缺 `data.ismummyhouse` 且当前实例已判为木乃伊屋」时挂钩、带字段走字段恢复；⑤ P-1 与摘要改指实际入口 `basic_init_fn`／`basic_clear_fn`（要求 `floater:IsFloating()`）与 fumarole 破损路径，`SetSkinsOnAnim` 属人物皮肤／服饰；⑥ SB-3 改为可监听事件 `onsink`／`onfallinvoid`，并写出存活／状态标记／`ShouldDrown()`／`ShouldFallInVoid()` 门禁。两页 validate_page 通过、strict 58 引用 0 error（2 处既有 `source-blank` warning 未动）。
- **P2／P3 二次复核修正（2026-11-08，3 项）**：① P-6 与摘要的鱼人屋回家时序改正——`TakeInChild` 先调 OnGoHome、后从屋外名单移除该孩子（childspawner.lua:671-677），故普通屋只有「回调执行时 `CountChildrenOutside() < 1`」才尝试重启（最后一只回家通常不触发），且 `StartSpawning` 仍受冬季限制（mermhouse.lua:122-126）；② 读档卸装的事件名写实——`DropEquipped → DropItem → RemoveFromOwner → RemoveItem → Unequip` 链上原持有者收 `unequip`（inventory.lua:1241）与 `dropitem`（:749），物品自身收 `ondropped`（inventoryitem.lua:392），摘要与 P-3 由「发 drop 事件」改为这三条；③ hounded 摘要与页内验证记录补「旧版」，避免读成 26474873 仍不推进。`hounded` 页 validate_page 通过、链接 0 error。
