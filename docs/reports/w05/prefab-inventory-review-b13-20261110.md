# B13 远古、裂隙和机械材料外部源码审核

- 2026-11-10 初检。Steam 安装清单与 `raw/current/BUILD.txt` 同为 **26474873**，证据取自 `raw/current/scripts`。按[最新认领](prefab-inventory-ledger-20261108.md#L164)核对 11 页、13 个物品注册、1 个同文件关联 FX，以及[制作材料索引 B13 十一行](../../../wiki/reference/prefabs/crafting-materials-index.md#L24-L34)。
- 初检结论：**待修订，暂不收口。P1 × 3、P2 × 10、P3 × 1。** 报告末尾按原编号回填，未运行游戏。
- 当前结论：**R01–R14 全部通过，可以收口。** 见[第二轮外部定向复验](#第二轮外部定向复验)。

## P1

### R01 三颗紫宝石只会激活法杖焦点，不会挂地图传送目的地

[宝珠碎片 L64、L94](../../../wiki/reference/prefabs/vault-orb.md#L64-L94)和[索引](../../../wiki/reference/prefabs/crafting-materials-index.md#L30-L30)把「三颗同种」写成添加目的地的充分条件，与同页 L87 矛盾。正常装填时，任一空座让 `validteleporttarget` 失败，混装会被宝石座提前用 `WRONGGEM` 拒绝。三颗紫宝石能让状态为 `VALID`、供法杖使用，但 `OnGemChange` 只在全座有效且 `requiredgem == "vault_orb_refined"` 时添加地图目的地。目的地座上的碎片不会被法杖传送打碎。验收须分别给出三颗紫宝石、三颗宝珠碎片、缺一座或尝试混装三组结果，并同步正文、导航、陷阱和索引。证据见[目的地与状态](../../../raw/current/scripts/prefabs/telebase.lua#L34-L76)、[焦点状态](../../../raw/current/scripts/prefabs/telebase.lua#L116-L125)、[插入与防碎](../../../raw/current/scripts/prefabs/telebase_gemsocket.lua#L6-L18)、[碎片防碎](../../../raw/current/scripts/prefabs/telebase_gemsocket.lua#L56-L68)。

### R02 陀螺传导核心示例漏掉客户端目标回调和服务端目标校验

[示例 L74–99](../../../wiki/reference/prefabs/wagdrone-parts.md#L74-L99)把 `UseableTargetedItem_ValidTarget` 放进 `ismastersim`，与原版在 pristine 前公共段接线不同。远端客户端无法用该回调筛出例子宣称的 `my_repairable`。服务端的 `ACTIONS.USEITEMON` 也不会重跑候选回调，只执行 `OnUseFn`，示例直接解引用 `target.components.health`。目标有示例标签却无生命组件，或在请求到达时已失效，仍可能报错。将可复制的候选判断放在公共路径，并在服务端验证有效目标、标签和生命组件后才改状态、扣料。末件被删除时先判实例有效再排复位，或用组件的「使用不切换可用性」配置省去复位任务。同步 L49 的动作链：`USEITEMON` 不自动 `StopUsingItem`。验收至少覆盖远端候选、标签存在但无 health、堆叠剩一件。证据见[原版端分界及末件守卫](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L13-L61)、[动作候选](../../../raw/current/scripts/componentactions.lua#L1900-L1913)、[动作执行](../../../raw/current/scripts/actions.lua#L3351-L3375)、[组件开关与结算](../../../raw/current/scripts/components/useabletargeteditem.lua#L105-L137)。

### R03 自定义生物只注册扫描定义，仍可能完全扫不到

[生物数据的示例 L74–87](../../../wiki/reference/prefabs/scandata.md#L74-L87)给出的唯一操作是调用 `AddCreatureScanDataDefinition`，但没有把候选标签写入示例前提。扫描仪先按 `SCAN_CAN` 和 `SCAN_CANT` 查世界实体，再查 Lua 表。自定义实体没有任何白名单标签，或带 `DECOR`／`FX`／`INLIMBO`／`NOCLICK`，即使已成功注册也不会进入后续筛选。`SCAN_CAN` 是 Lua 列表名，给实体加同名字符串也无效。示例明确要求目标已有适用白名单标签和可用的配方名，或展示在目标工厂正确加标记。验收用同一注册定义对照「无任一白名单标签」与「带 `animal` 且无排除标签」两个目标。证据见[过滤表与配方名](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L100-L109)、[世界查找先于定义校验](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L480-L517)、[注册函数](../../../raw/current/scripts/wx78_moduledefs.lua#L12-L18)。

## P2

### R04 亮茄死亡表的 2 外壳和 2 植物肉是基础名单

[亮茄外壳 L42–44、L86、L94](../../../wiki/reference/prefabs/lunarplant-husk.md#L40-L94)与[索引](../../../wiki/reference/prefabs/crafting-materials-index.md#L27-L27)将四个条目写成死亡时稳定原样落地。宿主带 `burnable`，死亡时调用 `DropLoot`。实际调用时若带 `burnt` 或满足燃烧／燃料门禁，先查熟制变体。普通火下无熟态外壳变灰，植物肉命中 `plantmeat_cooked`。受控火保留外壳原名，外壳本身无 `burnable`，不会触发落地闷烧；有熟态的肉仍先换名。`plant` 标签不在灰烬豁免的 `tree`／`boulder` 集合内。同步基础掉落、死亡条件和索引，避免把火烧写成与死亡掉落无关。证据见[宿主标签](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L443-L452)、[可燃与事件接线](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L487-L516)、[实际掉落](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L196-L217)、[换名与闷烧门禁](../../../raw/current/scripts/components/lootdropper.lua#L398-L445)。

### R05 扫描可以只教配方，零余额不一定阻止扫描

[生物数据 L37–41、L65–68、L93–95](../../../wiki/reference/prefabs/scandata.md#L35-L95)把 `GetData > 0` 写成所有候选的硬门禁，并含混地称「成功件」拾取时教配方。候选循环另外收集未学配方的目标，能用该列表替换普通候选，即使余额为 0 仍能扫描。出数据要等 `SpendData > 0`，教配方则在收取成功状态的扫描仪时触发。另一个数值边界：落点是 X、Z 分别加 0–1 单位，径向上界是 `sqrt(2)`，不能说总偏移小于四分之一格。验收写清「零数据且有未知配方」与「零数据且已会配方」的区别，明确偏移按轴。证据见[候选与教学优先](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L503-L547)、[记录配方](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L574-L592)、[出数据和偏移](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L736-L755)、[收取时教学](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L1019-L1035)。

### R06 收编会撤掉残骸锤拆，耐久用尽也不把它变回残骸

[陀螺传导核心摘要、L41、L54、L105](../../../wiki/reference/prefabs/wagdrone-parts.md#L35-L105)称能变成跟随者，又说反悔可以再锤成残骸且核心未必回本。实际 `ChangeToFriendly` 移除 `workable`，加 `companion`、`inventoryitem`、耐久和部署点组件。该链不设置玩家 leader；描述成可部署的友方无人机更准确。耐久归零只清部署点并停用。真正仍在残骸态并执行锤拆时，核心在该表里是 1.00，与「不一定回本」无关。还要区分无效／不持久目标直接失败与有效但非滚动无人机返回 `CANNOT_FIX_DRONE`。证据见[移除残骸组件](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L41-L47)、[残骸掉落与转态](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L125-L169)、[耗尽与收编](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L206-L253)、[使用返回值](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L13-L31)。

### R07 灵体传输模块的 12 是生物数据，废料只用 2

[废料配方 L57](../../../wiki/reference/prefabs/wagpunk-bits.md#L54-L59)把 `wx78_gestalttrapper` 的两种材料串位。[生物数据页](../../../wiki/reference/prefabs/scandata.md#L47-L55)已正确区分：灵体传输模块 12 生物数据＋2 废料，电刑机控制器 10 生物数据＋8 废料＋1 晶体管。给设备行补各自的 `builder_skill`（`wx78_allegiance_lunar`／`wx78_zapdrone_1`），避免沿用 23 个模块的通用科技门禁。证据见[两条实际配方](../../../raw/current/scripts/recipes.lua#L788-L789)。

### R08 修理请求值、锤拆返还量和零耐久不能直接当成实际结果

[绝望石 L81–84、L95](../../../wiki/reference/prefabs/dreadstone.md#L79-L95)、[铥矿 L89–96](../../../wiki/reference/prefabs/thulecite.md#L87-L96)与[废料 L83–100](../../../wiki/reference/prefabs/wagpunk-bits.md#L81-L100)应沿用同一个 `Repairable` 结算口径。绝望石 80、铥矿 100、废料 80 是生命请求，最后按上限截断；废料的 +1 工作量是另一分支，示例只接已有 health 的目标，不能写成一次同时 +80 和 +1。废料示例还不必强制 `SetHealthRepairable(true)`，构造和生命值变化会按是否受损维护；强行设真会让满血初始目标出现候选而在结算失败。绝望石墙锤拆实际是 `max(1, floor(2 × 当前生命比例))`，L95 的固定 2 块和[索引 L25](../../../wiki/reference/prefabs/crafting-materials-index.md#L25-L25)的回收表述要区分墙体锤拆与配方拆解。

[噩梦燃料 L110](../../../wiki/reference/prefabs/nightmare-fuel.md#L108-L111)从 0 耐久算 5 块应标纯算术或换成尚未耗尽的实例，懒人护符耐久耗尽会移除。证据见[修复分支](../../../raw/current/scripts/components/repairable.lua#L106-L124)、[修理组件构造](../../../raw/current/scripts/components/repairable.lua#L35-L54)、[生命值截断](../../../raw/current/scripts/components/health.lua#L565-L574)、[锤拆公式](../../../raw/current/scripts/prefabs/walls.lua#L167-L173)、[护符耗尽接线](../../../raw/current/scripts/prefabs/amulet.lua#L556-L564)。

### R09 蚁狮的贡品值只是材料门禁，160 秒还受计时器上限约束

[尘土块用途和接入](../../../wiki/reference/prefabs/refined-dust.md#L50-L89)说蚁狮只看 `rocktribute > 0`，索引把 `1` 等同于固定 160 秒。蚁狮若有待发奖励或已进入索战条件，`AcceptTest` 就拒绝；交易者还会先走 `AbleToAccept`。普通进贡才按 `rocktribute × ANTLION_TRIBUTE_TO_RAGE_TIME` 计算增量，累计和现有计时器都受最大值限制。自定义示例可保留设置字段的代码，标题、前提和自验写明「具备贡品资格」及蚁狮状态。同步[索引 L31](../../../wiki/reference/prefabs/crafting-materials-index.md#L31-L31)。证据见[交易入口](../../../raw/current/scripts/components/trader.lua#L103-L134)、[蚁狮验收与计时](../../../raw/current/scripts/prefabs/antlion.lua#L115-L151)。

### R10 纯粹辉煌嬗变在索引反向，正文的记录数混合了产出和拆解

[纯粹辉煌索引 L28](../../../wiki/reference/prefabs/crafting-materials-index.md#L28-L28)写成「2 换 3 注能碎片」，与[正文 L49–52](../../../wiki/reference/prefabs/purebrilliance.md#L49-L52)和实际配方矛盾。准确方向是 3 注能碎片 → 1 纯粹辉煌，1 纯粹辉煌 → 2 注能碎片。正文 L61 的 22 条若保留，应改作「涉及记录」，拆成 19 条 `Recipe2` 消耗（含反向嬗变）、1 条正向产出、2 条 `DeconstructRecipe` 回收。验收时分别按 `Ingredient("purebrilliance"`、`product="purebrilliance"` 搜本版 `recipes.lua`。证据见[两个方向](../../../raw/current/scripts/recipes.lua#L1499-L1500)、[锻造配方](../../../raw/current/scripts/recipes.lua#L848-L859)、[两条拆解回收](../../../raw/current/scripts/recipes.lua#L1660-L1662)。

### R11 共享概率表的小数是基础概率，不能直接当作实战期望

[暗影燃料 L62](../../../wiki/reference/prefabs/nightmare-fuel.md#L60-L73)、[绝望石 L46](../../../wiki/reference/prefabs/dreadstone.md#L44-L54)、[暗影碎布 L46](../../../wiki/reference/prefabs/voidcloth.md#L44-L54)把共享表解释成直接按条目概率掷骰，[废料 L48](../../../wiki/reference/prefabs/wagpunk-bits.md#L39-L48)和[铥矿 L55–56](../../../wiki/reference/prefabs/thulecite.md#L47-L57)也没给出期望的条件。两种机会表的概率低于 1 时都先走 `GetChance`，取死亡归因实体或最后工作者的正向幸运；1.00 条目直接入列。基础概率可以相加求无修正期望，有幸运时对修正后的概率求和。加权单抽与直接 `SpawnLootPrefab` 分开。反例：暗影碎布 0.33 的第四条在正向幸运时高于 33%。证据见[幸运来源与分支](../../../raw/current/scripts/components/lootdropper.lua#L163-L227)、[幸运公式](../../../raw/current/scripts/componentutil.lua#L2092-L2097)、[公式接线](../../../raw/current/scripts/componentutil.lua#L2151-L2151)。

### R12 工位筛选和配方记忆要按是否有 nounlock 分开

[暗影碎布 L60、L84、L92](../../../wiki/reference/prefabs/voidcloth.md#L58-L92)把 10 条消耗全部写成始终要在基座旁，实际九条标 `nounlock=true`，弹弓握把配方 L245 只有 `force_hint`、站点筛选和沃尔特技能，允许正常学会。[铥矿 L83](../../../wiki/reference/prefabs/thulecite.md#L78-L83)反过来把两条带 `station_tag` 的配方写成唯二必须站内，实际弹药和框均可记忆，多条仅有 `nounlock` 的远古配方才不会被普通解锁写入。同理，亮茄外壳和纯粹辉煌的两条弹药与九／十一条锻造配方应分开。同步涉及页和[索引 L26–29](../../../wiki/reference/prefabs/crafting-materials-index.md#L26-L29)，用「有技能、在站旁首次解锁、离站再次制作」作反例，保留对应的科技等级和技能门禁。证据见[铥矿与月系弹药](../../../raw/current/scripts/recipes.lua#L225-L229)、[框与握把](../../../raw/current/scripts/recipes.lua#L244-L245)、[基座配方](../../../raw/current/scripts/recipes.lua#L866-L875)、[记忆条件](../../../raw/current/scripts/components/builder.lua#L463-L473)、[已学先判](../../../raw/current/scripts/components/builder.lua#L883-L907)。

### R13 裂隙矿苗 480 秒是管理器冷却，回填目标取到时仍受控的裂隙

[绝望石 L50](../../../wiki/reference/prefabs/dreadstone.md#L48-L54)和[索引 L25](../../../wiki/reference/prefabs/crafting-materials-index.md#L25-L25)写成挖完后原矿苗固定在 480 秒再生。挖完通知管理器，管理器重排全局冷却。回调不使用 `OnFissureMinedFinished(fissure)` 的参数，而读当时的 `_fissure`，无受控裂隙时不会当场放回。写明 480 秒是管理器冷却，原裂隙是否仍受控是另一条件。挖完固定调用三次 `SpawnLootPrefab` 已核对。证据见[挖完](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L341-L358)、[管理器回调](../../../raw/current/scripts/components/shadowthrallmanager.lua#L470-L490)、[冷却常量](../../../raw/current/scripts/tuning.lua#L7773-L7774)。

## P3

### R14 纯粹辉煌的关联 FX 已有专页，交付统计应显式标为复用

[纯粹辉煌 L30–31、L82](../../../wiki/reference/prefabs/purebrilliance.md#L26-L33)已经正确区分物品与 FX 工厂，但[台账](prefab-inventory-ledger-20261108.md#L164)和日志把「＋1 个 FX 注册」与新落地注册并列，容易被下游当成新增主归属。[设备 FX 页](../../../wiki/reference/prefabs/winona_battery_high_shatterfx.md#L26-L31)已有该注册的精确别名与完整槽位生命周期。B13 写作侧只保留关联构造，并补回已有 FX 页入口。主代理把统计写成「13 个物品主注册，关联复用既有 1 个 FX」，便于合并总覆盖数据。证据见[工厂双返回](../../../raw/current/scripts/prefabs/purebrilliance.lua#L102-L129)和[已有 FX 导航](../../../wiki/reference/prefabs/fx-structure-index.md#L20-L20)。

## 源码事实与页面命中

| 对象页 | 已核验链与结果 |
|---|---|
| 噩梦燃料与纯粹恐惧 | 两注册及 180／360 秒、50／100 次的赋值匹配。追取燃料计算与橙护符修理，以及薇诺娜低阶发电机的角色技能分支。掉落基础概率和零耐久边界见 R08、R11。 |
| 绝望石 | 裂隙是矿苗载体，挖完固定生成 3 块。抽核墙体回血与锤拆、威尔逊双向嬗变。冷却条件见 R13。 |
| 暗影碎布 | 核四张裂影仆从表和憎恶第二张表，九件套的 `nounlock` 与单独的沃尔特握把配方，以及成品修理走 forge 组件。见 R11、R12。 |
| 亮茄外壳 | 致命亮茄固定名单与普通／受控换名链已对照；抽核亮茄修补套件与沃尔特弹药。见 R04、R12。 |
| 纯粹辉煌 | 追亮喙鸟喂三份与冷却、沃特随从施法、发电机槽位 FX，并计正反嬗变。见 R10、R12、R14。 |
| 铥矿与铥矿碎片 | 两注册同为 ELEMENTAL，两个材料修理参数相差六倍。抽核祭坛初值与锤后初值、弹药与弹弓框。见 R08、R11、R12。 |
| 传送宝珠碎片 | 断链路标原料、1→3 精炼、地图选点到执行，以及焦点装填和防碎链。见 R01。 |
| 尘土块 | 档案馆蓝图发放、配方、捕虫网与琥珀美食消费和蚁狮验收。蚁狮状态与封顶见 R09。 |
| 生物数据 | 核候选表、首次余额／取整／回复、生成和取回扫描仪时教学。23 个模块及 2 个设备的计数对上。见 R03、R05、R07。 |
| 陀螺传导核心 | 残骸表 1.00、候选与服务端执行、满耐久不耗料，及收编和耗尽。见 R02、R06。 |
| 废料 | 抽核残骸掉落、废料墙／棋子的材料接线，以及 WX-78 两条设备配方。见 R07、R08、R11。 |

同版导出逐项核对 13 个物品主注册与原版简中名称，关联 FX 有独立工厂返回。严格源码链接检查覆盖 11 页＋叶索引，**756 处引用、0 错、0 警**。机械结果单独记录。未实测游戏，未穷举裂隙刷怪节奏、档案馆世界生成分布、所有科技组合和全套配方下游行为。

## 修订回执与定向复验

同一事实若在摘要、正文、表格、陷阱、验证记录、关联页或索引复述，回执列明已同步位置及自验结果。R14 的交付计数由主代理同步。

| 编号 | 原写作者修订位置与自验 | 外部定向复验 |
|---|---|---|
| R01 | 待回填 | 通过（第二轮）。区分空座 `GEMS` 和异种插入 `WRONGGEM`。 |
| R02 | 待回填 | 通过。公共候选、服务端校验与末件守卫已就位。 |
| R03 | 待回填 | 通过。扫描标签前提和配方名已明确。 |
| R04 | 待回填 | 通过（第二轮）。索引已补普通／受控燃烧与闷烧条件。 |
| R05 | 待回填 | 通过（第二轮）。导航、验证与正文已区分出数据和教学。 |
| R06 | 待回填 | 通过（第二轮）。导航已区分收编／修满扣料、满耐久免扣及残骸转态。 |
| R07 | 待回填 | 通过。两种设备的材料数量与技能门禁一致。 |
| R08 | 待回填 | 通过（第二轮）。绝望石摘要与废料示例已限定实际分支。 |
| R09 | 待回填 | 通过。蚁狮受理状态、160 秒增量与封顶已对齐。 |
| R10 | 待回填 | 通过。正反嬗变与 19＋1＋2 的记录分类已对齐。 |
| R11 | 待回填 | 通过（第二轮）。碎布索引已区分手／角／口、翅膀及无幸运基准。 |
| R12 | 待回填 | 通过（第二轮）。索引与月系弹药正文均已区分站点科技和配方记忆。 |
| R13 | 待回填 | 通过（第二轮）。矿苗回填以冷却到点时的受控裂隙为准。 |
| R14 | 待回填 | 通过。物品主注册与既有 FX 的复用归属已分开。 |

## 第一轮外部定向复验

2026-11-10，Steam build **26474873**。定向核对 R01–R14 及修订触及的正文、索引和直接关联页。**6 项通过、8 项部分修复，暂不收口。** 三项原 P1 的核心行为和接入示例已纠正。R01 余下的是索引里的状态区分。写作方修订以当前页面及[日志](../../../wiki/log.md#L75-L76)为对照，表中的作者回执保留给写作方填写。

### 仍待同步的原编号

- **R01**：宝珠正文已区分三碎片、三紫宝石和插入拒绝。[索引 L30](../../../wiki/reference/prefabs/crafting-materials-index.md#L30-L30)的「空座或异种为 GEMS」仍把两层写成同一状态。分开写「空座时焦点状态 `GEMS`」「异种插入返回 `WRONGGEM`」。混装被拒后，焦点状态仍按当前已有的座子计算。证据见[座子的交易检查](../../../raw/current/scripts/prefabs/telebase_gemsocket.lua#L6-L18)及[焦点状态](../../../raw/current/scripts/prefabs/telebase.lua#L116-L118)。
- **R04**：[外壳正文 L42–45](../../../wiki/reference/prefabs/lunarplant-husk.md#L42-L45)已经给出正确分支，[索引 L27](../../../wiki/reference/prefabs/crafting-materials-index.md#L27-L27)仍只写「外壳随 `burnt` 转灰」。用「普通燃烧改写时外壳转灰，受控火保留且不闷烧」同时覆盖燃烧当下与受控例外。植物肉熟态分支排在受控判定前。证据见[改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)和[闷烧门禁](../../../raw/current/scripts/components/lootdropper.lua#L404-L407)。
- **R05**：[生物数据获取段 L37–42](../../../wiki/reference/prefabs/scandata.md#L37-L42)和索引已补零余额教学，落点也改为 X／Z 各自偏移。[导航 L69](../../../wiki/reference/prefabs/scandata.md#L69-L69)把「数据 > 0」同时套到生成与教学，[验证 L97](../../../wiki/reference/prefabs/scandata.md#L97-L97)还写「余额回到 1 点才能再扫」。改为只有**出数据**需要正余额，未学配方可走教学候选。L41 的无数据提示由 15 秒计时器限频，若保留「只播一次」须限定到一次计时窗口。证据见[候选替换与提示](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L503-L542)、[出数据](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L736-L755)及[收取时教学](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L1019-L1035)。
- **R06**：[陀螺传导核心 L41–46、L105](../../../wiki/reference/prefabs/wagdrone-parts.md#L39-L46)已修正收编、归零和退回残骸的结论，[导航 L63、L66](../../../wiki/reference/prefabs/wagdrone-parts.md#L62-L67)仍写「每次用掉 1 个」和「锤成残骸后带 `HAMMER_workable`」。满耐久分支成功但不扣料，且由 `ChangeToLoot` 先挂可锤组件，玩家锤击才兑现掉落。同步导航，保持和[陷阱](../../../wiki/reference/prefabs/wagdrone-parts.md#L103-L105)一致。证据见[扣料条件](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L13-L28)和[残骸配置](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L136-L169)。
- **R08**：[绝望石摘要](../../../wiki/reference/prefabs/dreadstone.md#L6-L6)的「按 +80 生命修」同步为正文的「请求回复 80、按上限截断」。[废料示例前提 L83–84](../../../wiki/reference/prefabs/wagpunk-bits.md#L83-L99)仍把仅有 `workable` 的目标纳入可照抄的范围，但示例的接线守卫只放行已有 `health` 的目标。限定为生命型示例即可，工作面作为独立分支说明，且 `workrepairable` 后续由 `workable` 状态维护。铥矿、橙护符、墙体锤拆公式及废料的 80／1 分支已核对通过。证据见[组件构造](../../../raw/current/scripts/components/repairable.lua#L35-L54)、[实际两分支](../../../raw/current/scripts/components/repairable.lua#L106-L124)、[工作状态回调](../../../raw/current/scripts/components/workable.lua#L1-L13)及[生命上限](../../../raw/current/scripts/components/health.lua#L565-L574)。
- **R11**：五页正文已补无幸运基础值。[暗影碎布索引 L26](../../../wiki/reference/prefabs/crafting-materials-index.md#L26-L26)仍把裂影仆从统一成 `3×1.00＋0.33`。该表只适用于手／角／口，翅膀是 `2×1.00＋0.67`；两组并列并标基础概率。无需重展开各页掉落表。证据见[碎布正文表](../../../wiki/reference/prefabs/voidcloth.md#L46-L54)、[手](../../../raw/current/scripts/prefabs/shadowthrall_hands.lua#L17-L24)及[翅膀](../../../raw/current/scripts/prefabs/shadowthrall_wings.lua#L18-L27)。
- **R12**：[制作材料索引 L26、L28–29](../../../wiki/reference/prefabs/crafting-materials-index.md#L26-L29)仍把 `nounlock` 写成「站旁首次解锁」。它会跳过 `UnlockRecipe`，通常靠当次站点科技制作，不能靠一次普通解锁记住。反过来，[亮茄外壳 L65](../../../wiki/reference/prefabs/lunarplant-husk.md#L65-L65)、[纯粹辉煌 L58](../../../wiki/reference/prefabs/purebrilliance.md#L58-L58)仍把「月之锻造台」并列成持续硬门禁；两条弹药都无 `nounlock`，学会后离站仍能做，技能门禁继续生效。纯粹辉煌索引的「两条弹药」跨到外壳页，可拆回各自一条。证据见[两条月系弹药](../../../raw/current/scripts/recipes.lua#L228-L229)、[解锁与跳过](../../../raw/current/scripts/components/builder.lua#L463-L473)和[技能及已学检查](../../../raw/current/scripts/components/builder.lua#L869-L887)。
- **R13**：[绝望石 L49](../../../wiki/reference/prefabs/dreadstone.md#L49-L49)已写 480 秒是管理器冷却，却仍写「挖完时没有受控裂隙就不会当场放回」。判空时点是**冷却回调执行时**，挖完后也可能在 480 秒内释放或换掉裂隙。索引已写成取当时受控裂隙，与源码一致。证据见[排程和回调](../../../raw/current/scripts/components/shadowthrallmanager.lua#L470-L490)。

### 已通过

| 编号 | 定向复验 |
|---|---|
| R02 | [陀螺核心示例](../../../wiki/reference/prefabs/wagdrone-parts.md#L74-L99)在公共 postinit 赋候选回调，服务端验证有效目标、标签与生命组件才改状态和扣料。末件删除后先判实例有效，动作说明也已区分独立的停止使用动作。对应[候选](../../../raw/current/scripts/componentactions.lua#L1900-L1913)、[执行](../../../raw/current/scripts/actions.lua#L3351-L3375)及[原版末件守卫](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L13-L27)。 |
| R03 | [自定义扫描前提和代码](../../../wiki/reference/prefabs/scandata.md#L75-L89)已明确实体须具备任一支持标签且无排除标签，示例用已存在的 `wx78module_maxhealth`。对应[标签表与配方名](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L100-L109)、[查实体](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L480-L517)。 |
| R07 | [废料配方](../../../wiki/reference/prefabs/wagpunk-bits.md#L57-L57)、[生物数据设备行](../../../wiki/reference/prefabs/scandata.md#L53-L55)与[索引](../../../wiki/reference/prefabs/crafting-materials-index.md#L34-L34)一致。灵体传输模块 12 生物数据＋2 废料，电刑机控制器 10 生物数据＋8 废料＋1 晶体管，各自技能门禁齐备。对应[实际配方](../../../raw/current/scripts/recipes.lua#L788-L789)。 |
| R09 | [尘土块正文与示例](../../../wiki/reference/prefabs/refined-dust.md#L53-L89)已区分材料资格、交易状态、160 秒增量与 6 天双封顶。[索引](../../../wiki/reference/prefabs/crafting-materials-index.md#L31-L31)也保留状态与上限。对应[蚁狮受理及计时](../../../raw/current/scripts/prefabs/antlion.lua#L115-L151)和[常量](../../../raw/current/scripts/tuning.lua#L3321-L3323)。 |
| R10 | [纯粹辉煌正文](../../../wiki/reference/prefabs/purebrilliance.md#L49-L61)与[索引](../../../wiki/reference/prefabs/crafting-materials-index.md#L28-L28)的正反向统一，19 条消耗、1 条产出、2 条拆解分开计数。对应[嬗变](../../../raw/current/scripts/recipes.lua#L1499-L1500)和[拆解](../../../raw/current/scripts/recipes.lua#L1660-L1662)。 |
| R14 | [纯粹辉煌](../../../wiki/reference/prefabs/purebrilliance.md#L26-L33)已把 FX 标成关联复用并链接既有主归属，[台账](prefab-inventory-ledger-20261108.md#L164-L164)与[执行表](prefab-inventory-batches-20261108.md#L5-L5)也区分 13 个物品主注册和 1 个既有 FX。收口同步 PLAN 时沿用该统计口径。 |

独立重跑 11 主页面＋叶索引的严格源码链接检查，**781 处引用，0 错 0 警**。本轮只定向核验原问题及修订触及的断言，未运行游戏。

## 第二轮外部定向复验

2026-11-10，Steam build **26474873**。复验第一轮余下的 R01、R04、R05、R06、R08、R11、R12、R13，八项全部通过。加上已通过的六项，**R01–R14 全部通过，可以收口**。

| 编号 | 本轮核验结果 |
|---|---|
| R01 | [索引 L30](../../../wiki/reference/prefabs/crafting-materials-index.md#L30-L30)已区分空座的焦点状态 `GEMS`、异种插入的交易拒绝 `WRONGGEM`，并保留三碎片才挂地图目的地、三紫宝石供法杖使用的条件。对应[插入检查](../../../raw/current/scripts/prefabs/telebase_gemsocket.lua#L6-L18)、[目的地挂载](../../../raw/current/scripts/prefabs/telebase.lua#L43-L68)和[焦点状态](../../../raw/current/scripts/prefabs/telebase.lua#L116-L118)。 |
| R04 | [索引 L27](../../../wiki/reference/prefabs/crafting-materials-index.md#L27-L27)已补普通改写转灰、受控燃烧保留外壳且不闷烧，与[正文](../../../wiki/reference/prefabs/lunarplant-husk.md#L42-L45)一致。植物肉的熟态匹配先于受控分支，落地闷烧另查掉落物自身是否可燃。对应[改写与闷烧](../../../raw/current/scripts/components/lootdropper.lua#L404-L442)。 |
| R05 | [生物数据导航](../../../wiki/reference/prefabs/scandata.md#L69-L69)和[验证段](../../../wiki/reference/prefabs/scandata.md#L97-L97)都把正余额限定到出数据，未学配方仍可进入教学候选。[获取段](../../../wiki/reference/prefabs/scandata.md#L41-L42)已补无数据提示的 15 秒限频。对应[候选与计时](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L503-L542)及[生成门禁](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L736-L755)。 |
| R06 | [陀螺传导核心导航](../../../wiki/reference/prefabs/wagdrone-parts.md#L62-L67)已写明收编或补满才扣 1 个，满耐久成功免扣。残骸由 `ChangeToLoot` 先配置可锤组件，锤击执行掉落。对应[使用分支](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L13-L28)和[残骸链](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L136-L169)。 |
| R08 | [绝望石摘要](../../../wiki/reference/prefabs/dreadstone.md#L6-L6)已标 80 是受上限截断的请求值。[废料示例](../../../wiki/reference/prefabs/wagpunk-bits.md#L81-L99)明确只接生命型目标，工作面是另一个分支，并按对应组件维护标签。对应[修理初始化与分支](../../../raw/current/scripts/components/repairable.lua#L35-L69)、[实际结算](../../../raw/current/scripts/components/repairable.lua#L106-L124)及[生命上限](../../../raw/current/scripts/components/health.lua#L565-L574)。 |
| R11 | [暗影碎布索引](../../../wiki/reference/prefabs/crafting-materials-index.md#L26-L26)已区分手／角／口的 `3×1.00＋0.33` 与翅膀的 `2×1.00＋0.67`，并统一标无幸运基础值。对应[手的表](../../../raw/current/scripts/prefabs/shadowthrall_hands.lua#L17-L24)、[翅膀的表](../../../raw/current/scripts/prefabs/shadowthrall_wings.lua#L18-L27)和[概率处理](../../../raw/current/scripts/components/lootdropper.lua#L173-L227)。 |
| R12 | [索引 L26–29](../../../wiki/reference/prefabs/crafting-materials-index.md#L26-L29)已区分 `nounlock` 跳过普通解锁和可记忆的握把／弹药／弹弓框。[外壳](../../../wiki/reference/prefabs/lunarplant-husk.md#L65-L66)与[纯粹辉煌弹药](../../../wiki/reference/prefabs/purebrilliance.md#L58-L58)均写明学会后可离站，技能门禁仍生效。对应[月系配方](../../../raw/current/scripts/recipes.lua#L228-L229)、[解锁分支](../../../raw/current/scripts/components/builder.lua#L463-L473)及[技能与已学判断](../../../raw/current/scripts/components/builder.lua#L869-L888)。 |
| R13 | [绝望石 L49](../../../wiki/reference/prefabs/dreadstone.md#L49-L49)和[索引](../../../wiki/reference/prefabs/crafting-materials-index.md#L25-L25)已把 480 秒写成管理器冷却，回填时读取回调执行当下的受控裂隙。对应[排程和回调](../../../raw/current/scripts/components/shadowthrallmanager.lua#L470-L490)。 |

同一范围的 12 文件严格源码链接检查通过，**784 处引用，0 错 0 警**。本轮覆盖八项定向断言与相关入口，未运行游戏。
