# 地图／UI／行为节点明细（30 个修改文件，全部已读 diff）

> 条目核验状态：标"已核验"者为逐项源码核对；未标注者为待审（已读文件级 diff，接入面未逐项展开），查询时只能作候选结论。

| 文件 | 变化 | 级别 | 要点 |
|---|---|---|---|
| `map/customize.lua` | +3/−0 | 新增功能 | 世界设置新增 `bat_boss` 与 `rocky_boss` 两个洞穴专属选项（默认 default）。 |
| `map/layouts.lua` | +7/−0 | 新增功能 | 注册 `bat_boss_cave1/2/3` 三个静态布局，供蝙蝠 Boss 洞 set piece 使用。 |
| `map/maptags.lua` | +20/−0 | 新增功能 | 新增 `BatBossCave_Spawner` map tag：每图仅放一次，随机三选一 bat_boss_cave 静态布局，`level.overrides.batbosscave == "never"` 时跳过。 |
| `map/rooms/cave/bats.lua` | +1/−2 | **M-01** 行为变化 | FernyBatCave 房间 tags 追加 `BatBossCave_Spawner`，蝙蝠 Boss 洞只能落在蕨类蝙蝠洞。**接入**：自定义洞穴房间 tags 或屏蔽 FernyBatCave 的世界生成模组 **判断**：已有用法兼容（新增标记仅影响新世界投放） **证据**：map/rooms/cave/bats.lua:56 **核验**：已核验（本次重读 diff） |
| `map/rooms/cave/ruins.lua` | +1/−1 | 数值调整 | RuinedCity 房间地皮由 WORLD_TILES.CAVE 改为 WORLD_TILES.MUD。 |
| `map/rooms/cave/toadstoolarena.lua` | +11/−2 | 数值调整 | ToadstoolArenaCave distributepercent 0→0.1 并新增散布表（洞花、石笋、裂缝、洞穴池塘、batcave 0.01），蛤蟆竞技场不再空旷。 |
| `map/static_layouts/rooms/atrium_end/atrium_end.lua` | +11/−0 | 新增功能 | 布局新增 `atriummarker_gate_center` 标记点(288,288)，为新剧情提供定位。 |
| `map/static_layouts/vault_lobby.lua` | +15/−0 | 新增功能 | 大厅布局新增 `vault_teleporter` 标记（virtualroomteleporter 属性：direction/shuffleddirection=IN、roomsetname=VAULT），金库大厅接入虚拟房间传送体系。 |
| `map/storygen.lua` | +1/−0 | **M-02** 行为变化 | START 节点 data.tags 改为 ArrayUnion(randomStartNode.data.tags, …)，把随机起始节点携带的 map tag 合并进起始节点数据。**接入**：自定义任务链／起始节点 tags 的世界生成模组 **判断**：建议调整（START 节点现合并随机起始节点的 map tags，tag 传播语义变化） **证据**：map/storygen.lua:967 ArrayUnion **核验**：已核验（本次重读 diff） |
| `map/tasksets/caves.lua` | +1/−0 | 数值调整 | caves taskset 的 required_prefabs 增加 `batbosscave`，强制该 Prefab 必须在洞穴世界生成。 |
| `screens/mapscreen.lua` | +2/−2 | **M-03** 行为变化 | ProcessRMBDecorations_TOSS_MAP 改用 `rmb.invobject` 取地图装饰物品，非手持使用的自定义地图物品也可提供 InitMapDecorations/CalculateMapDecorations。**接入**：提供 InitMapDecorations／CalculateMapDecorations 自定义地图装饰物品的模组 **判断**：已有用法兼容（非手持使用的 invobject 现在也生效，为扩展） **证据**：screens/mapscreen.lua:895 rmb.invobject **核验**：已核验（本次重读 diff） |
| `screens/playerhud.lua` | +31/−0 | 新增功能 | 新增 ShroudenOver 全屏覆盖层（查理 Boss/雾罩效果）；新增 POI 指示器 API `AddPOIIndicator`/`RemovePOIIndicator`。 |
| `screens/quickjoinscreen.lua` | +1/−1 | **M-04** 接口变更 | `TheNet:JoinServerResponse` 调用新增第 4 参 `sel_serv.row`（行号），引擎接口签名变化。**接入**：自建服务器浏览器或包装 TheNet:JoinServerResponse 的模组 **判断**：待核验（传参变化已核验；引擎绑定对缺省第 4 参的处理未验证，缺省后果不下结论） **证据**：screens/quickjoinscreen.lua:329 **核验**：已核验（本次重读 diff） |
| `screens/redux/mainscreen.lua` | +8/−1 | 资源文案 | Quit 确认弹窗描述按 `TheGenericKV:GetKV("charlie_defeated")` 切换为 ASKQUITDESC_NOCHARLIE。 |
| `screens/redux/multiplayermainscreen.lua` | +11/−5 | 资源文案 | 默认横幅由 dst_menu_rift7 换成 rift8；IS_BETA 扩展含 dev 分支；Quit 弹窗同样按 charlie_defeated 切换文案。 |
| `screens/redux/optionsscreen.lua` | +27/−8 | 新增功能 | 设置页新增"崩溃报告"开关（`TheSim:Get/SetCrashReportsEnabled`）；automodsSpinner 改为仅在 `CanDownloadWorkshopMods()` 时创建。 |
| `screens/redux/pausescreen.lua` | +17/−0 | 新增功能 | 暂停界面在 `TheSim:FeedbackEnabled()` 时显示"F8 提交 beta 反馈"提示文本。 |
| `screens/redux/quickjoinscreen.lua` | +1/−1 | **M-05** 接口变更 | 同 legacy 版：`TheNet:JoinServerResponse` 追加 `sel_serv.row` 参数。**接入**：自建服务器浏览器或包装 TheNet:JoinServerResponse 的模组 **判断**：待核验（传参变化已核验；引擎绑定对缺省第 4 参的处理未验证，缺省后果不下结论） **证据**：screens/redux/quickjoinscreen.lua:293 **核验**：已核验（本次重读 diff） |
| `screens/redux/scrapbookdata.lua` | +32/−8 | 数值调整 | 图鉴大更新：新增 bat_boss 系列、rocky_boss 系列、charlie_boss、shrouden、仪式器官、king_cane、queen_torch 及 5 个新棋子条目；数值修正：alterguardian_phase4_lunarrift sanityaura 1.67→0.67、worm_boss damage 14→10、bat 增 nitre/sanityaura、wanderingtrader type thing→creature 等。 |
| `widgets/bloodover.lua` | +10/−0 | **M-06** 行为变化 | 监听 start/stopbatcorpsehatdrain 事件；UpdateState 检测头部装备带 `bathat` 标签即强制显示血渍屏效，为新蝙蝠尸体帽服务。**接入**：制作 bathat 标签头部装备或监听 start/stopbatcorpsehatdrain 事件的模组 **判断**：已有用法兼容（血渍屏效与衰减事件为增量） **证据**：widgets/bloodover.lua:46 事件监听、:73 bathat 判定 **核验**：已核验（本次重读 diff） |
| `widgets/eventannouncer.lua` | +1/−1 | **M-07** 行为变化 | 黑暗死亡（cause "NIL"）播报判定由 `theDead == "WAXWELL"`（实体比字符串恒 false，旧版从不指认查理）改为 `CHARACTER_KNOWS_CHARLIE[theDead.prefab] 且 not charlie_tracker:IsCharlieDefeated()`：击败查理前 winona/waxwell 的黑暗死亡改报 CHARLIE，击败后回落 DARKNESS。**接入**：解析死亡播报 cause 或覆盖 eventannouncer 判定的模组 **判断**：建议调整（击败查理前 winona/waxwell 黑暗死亡播报由 DARKNESS 改 CHARLIE，击败后回落） **证据**：widgets/eventannouncer.lua:156 **核验**：已核验（本次重读 diff） |
| `widgets/healthbadge.lua` | +9/−1 | **M-08** 行为变化 | 血量徽章警示判定新增 IsBatCorpseHatDrain：装备 `bathat` 标签帽子时徽章进入警示形态。**接入**：覆盖血量徽章警示判定或制作 bathat 装备的模组 **判断**：已有用法兼容（bathat 帽触发警示形态为增量） **证据**：widgets/healthbadge.lua:378 IsBatCorpseHatDrain **核验**：已核验（本次重读 diff） |
| `widgets/hoverer.lua` | +5/−5 | 内部重构 | 悬停文本健壮化：target/invobject 加 IsValid 检查，玩家判定由 HasTag("player") 改 `target.isplayer`，语义不变。 |
| `widgets/redux/modstab.lua` | +8/−5 | **M-09** 行为变化 | "清理全部"按钮仅在 `CanDownloadWorkshopMods()` 时加入按钮菜单，否则挂载但隐藏；焦点链调整。**接入**：扩展模组页 UI 或依赖"清理全部"按钮的模组 **判断**：已有用法兼容（按钮仅在可下载构建加入菜单，否则隐藏） **证据**：widgets/redux/modstab.lua:151 **核验**：已核验（本次重读 diff） |
| `widgets/redux/templates.lua` | +11/−4 | **M-10** 内部重构 | GetBuildString 重写并缓存（VERSION_CACHE）：新格式含 versioning.CURRENTVERSION、分支名与架构信息。**接入**：解析 GetBuildString 返回文本的模组 **判断**：建议调整（返回格式改为"版本 - 更新名"＋分支/构建号/架构两行并加缓存） **证据**：widgets/redux/templates.lua:190-202 VERSION_CACHE **核验**：已核验（本次重读 diff） |
| `widgets/textedit.lua` | +86/−0 | 新增功能 | 文本编辑大改：鼠标点击定位光标、支持 HOME/END/Ctrl+Home/Ctrl+End 及上下方向键移动光标（多行）、新增 `SetChanged()`/`SetFn(onchange)` 变更回调。模组自定义输入框可直接受益。 |
| `behaviours/chattynode.lua` | +12/−5 | **M-11** 接口变更 | ChattyNode 构造新增第 8 参 data（`noanim`、`nonpctalker`）；chatlines 回调签名变为 `(inst, initial_enter)`——首参仍为实体，新增第二参区分首次进入。**接入**：在自定义大脑中使用 ChattyNode 或实现 chatlines 回调的模组 **判断**：已有用法兼容（第 8 参 data 为可选增量；chatlines 第二参 initial_enter 为新增回调信息） **证据**：behaviours/chattynode.lua:1/:48 **核验**：已核验（本次重读 diff） |
| `behaviours/findclosest.lua` | +3/−19 | 内部重构 | 三段 tag 校验循环改用 `HasAllTags`/`HasAnyTag` 原子方法，逻辑等价。 |
| `behaviours/useshield.lua` | +40/−12 | **M-12** 行为变化 | 新增 `shouldemergefn`/`shouldemergefromshieldendfn` 选项控制出盾时机；attacked 事件先经 `IsEquipmentOnAttackedOrBlocked` 过滤；damagetaken 与 projectileincoming 仅在**非** shield／ignoredamageforshield 状态时累计（盾内受击不计入），电击/成功出盾时清零。**接入**：使用 UseShield 行为的自定义生物或调整其出盾／受击计数逻辑的模组 **判断**：建议调整（盾内受击不再累计、attacked 先经 IsEquipmentOnAttackedOrBlocked 过滤——判定含 `nodangermusic` 与 GetGrandOwner 条件，计数语义变化） **证据**：behaviours/useshield.lua:28/:115 **核验**：已核验（本次重读 diff） |
| `util/hitbox.lua` | +5/−0 | 新增功能 | HitBox 新增 `AddRectangle(x1,z1,x2,z2)`，由两个三角面拼成矩形命中区。 |
