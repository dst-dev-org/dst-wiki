# FX 新增页独立检视：第 3 轮

日期：2026-11-05。源码：`raw/current/scripts`，实读 `steam-buildid:26335394`；知识仓 HEAD：`e271fae`。

本轮对应原生成批 4（F11～F13），共 **4 页、24 个通用表注册**；不重复前两轮及 U15～U17。原批次台账的自报复审仅作定位线索，本报告独立核对当前文件与源码。

首次检视结论（已更正，最新状态见文末）：**需局部修复；当时报 P1×2、P2×8、P3×2**。其中 R01、R04 经源码复验确认是本次检视者的误报，均已撤回；海獭窝船缩放和 world Hook 不需要按原建议修改。其余原始问题与建议保留用于追溯。

本轮只新增报告、追加日志；不修知识页、不 record、不改 accepted／PLAN／索引、不提交或推送。所有行为结论均为源码推导，未运行游戏。

## 范围与逐页结论

路径相对 `wiki/reference/prefabs/`；行数按物理行统计，字节为本次实际输入。

| 页面 | 注册数 | 行／字节 | 核验范围与结论 |
|---|---:|---:|---|
| `boat_grass_erode.md` | 7 | 112／15,447 | 七条目、两船 postsinkfn、漏点事件、钻船周期任务、水草补丁正常到期与 LongUpdate、示例；R01～R03（P2）、R12（P3） |
| `fx_boat_crackle.md` | 5 | 107／14,236 | 五条目、boat_crackle 动态选名、船与冰船 SG、码头／海冰管理器、蟹王炮塔复用、示例 Hook；R04（P1）、R12（P3） |
| `boat_mast_sink_fx.md` | 8 | 102／16,350 | 桅杆 death／onremove／平台销毁链、防撞条 hit／changegrade／death、小龙舟灯耗尽与读档；R05～R08（P2）、R11（P3） |
| `cannonball_used.md` | 4 | 88／12,605 | 四条目、两投射物 OnHit、撞墙回调可达性、命中面选名、发射链、FX 失败与后续结算顺序；R09（P2）、R10（P1） |

24 名均在当前 `fx.lua` 注册表存在，已进入所属页 aliases 与正文；反查当前非归档 Prefab 页，未发现这 24 名跨页重复归属。当前简中 `.po` 无这 24 个精确 `STRINGS.NAMES.<注册名大写>` 键；效果描述未冒充对应注册名的官方译名。

## 待修问题

### R01 · 已撤回（误报）：海獭窝船两枚残影没有草船的 0.75 缩放

- 页面：[boat_grass_erode.md:31](../../../wiki/reference/prefabs/boat_grass_erode.md#L31)，第 32 行也写“同 fn”。第 29 行定义的草船 fn 含缩放 0.75，因此该复用表述把缩放带到了海獭窝船两名。
- 依据：[草船两条目](../../../raw/current/scripts/fx.lua#L2766-L2799) 都调用 `SetScale(0.75,0.75,0.75)`；[海獭窝船两条目](../../../raw/current/scripts/fx.lua#L3396-L3429) 未调用 SetScale，其余排序／朝向／层和 water 淡出逻辑相同。
- 建议：写“同排序／朝向／层，未设置草船的 0.75 缩放”；water 条目另保留相同淡出逻辑，避免继续用“同 fn”概括全部字段。

### R02 · P2：侵蚀页正文新增的 13 个证据文件没有进入 sources

- 页面：[boat_grass_erode.md:10](../../../wiki/reference/prefabs/boat_grass_erode.md#L10) 与 [第 51 行](../../../wiki/reference/prefabs/boat_grass_erode.md#L51)。17 处事件源的数量复核成立，但其中以下 13 个文件未被本页哈希基线追踪：
  `prefabs/cannonballs.lua`、`prefabs/mortarball.lua`、`prefabs/crabking.lua`、`stategraphs/SGcrabking_cannontower.lua`、`stategraphs/SGeyeofterror.lua`、`stategraphs/SGalterguardian_phase2.lua`、`prefabs/gnarwail.lua`、`prefabs/trident.lua`、`prefabs/waterplant_seed.lua`、`prefabs/waterpump.lua`、`prefabs/winch.lua`、`prefabs/fish_box.lua`、`debugkeys.lua`。
- 影响：strict 引用检查与已声明来源的哈希检查都能通过，但这些事件源变化不会使本页进入来源变化队列。`trident.lua` 同行引用两次，仍按一个漏项文件计数。
- 建议：保留实际核验的事件源证据时补齐 sources，再按现有生成器规则只刷新本页基线；若精简为代表入口，须同步缩窄“17 处全量核验／无遗漏”的正文范围，不能只删 sources 来消除差异。

### R03 · P2：水草补丁到期遗漏 LongUpdate 的直接生成路径

- 页面：[boat_grass_erode.md:57](../../../wiki/reference/prefabs/boat_grass_erode.md#L57)～第 58 行：SetRepairedTime 被概括为仅修复分支和存档恢复调用，到期链被统一写为 `pre_timeout → 动画长＋2 帧 → FX`。
- 依据：[BoatLeak:LongUpdate](../../../raw/current/scripts/components/boatleak.lua#L323-L337) 也是 SetRepairedTime 调用方：剩余时间大于零时重排任务，否则直接调用 `set_repair_timeout_state` 生成 FX、恢复 `med_leak`，不经过 [start_repair_timeout 的预动画与延迟](../../../raw/current/scripts/components/boatleak.lua#L48-L60)。
- 建议：把现有链限定为正常定时到期，补一句 LongUpdate 补算到期可直接落 FX；同步验证记录的覆盖范围。动态拼名仍只得到本注册名，不需另建页面。

### R04 · 已撤回（误报）：码头示例 Hook 的 world 不是正常生成的 forest 实例

- 页面：[fx_boat_crackle.md:78](../../../wiki/reference/prefabs/fx_boat_crackle.md#L78)。示例用 `AddPrefabPostInit("world", ...)` 安装自定义事件监听，但正常地表世界构造的是 `forest`，这段监听不会因此安装到该实例。
- 依据：[forest 的返回注册](../../../raw/current/scripts/prefabs/forest.lua#L745-L748) 是 `MakeWorld("forest", ...)`；[MakeWorld](../../../raw/current/scripts/prefabs/world.lua#L405-L418) 仅把 world 加入依赖，最后 [返回当前 name 的 Prefab](../../../raw/current/scripts/prefabs/world.lua#L623-L629)，并没有额外生成 world 实体。[PopulateWorld](../../../raw/current/scripts/gamelogic.lua#L389-L395) 按存档 map.prefab 生成；[PostInit 派发](../../../raw/current/scripts/mainfunctions.lua#L392-L410) 按实例／请求注册名查表，不按依赖或工厂来源派发。
- 建议：若保留地表码头示例，直接 Hook `forest`，明确服务端和已有真实码头坐标前提；无需改成全局通用 Hook。同步实际宿主证据，并核对坐标接口，不能仅凭 Lua 语法通过宣称示例可用。

### R05 · P2：快速拆船不能保证桅杆仍由 boat 非空分支生成沉没 FX

- 页面：[boat_mast_sink_fx.md:48](../../../wiki/reference/prefabs/boat_mast_sink_fx.md#L48)～第 49 行，及第 78、90 行、[索引第 37 行](../../../wiki/reference/prefabs/fx-table-world-index.md#L37)。文中认为 InstantlyBreakBoat 跳过解绑、且无 death 时仍靠 `boat ~= nil` 出沉没动画，遗漏了船的 onremove 监听。
- 依据：[mast 构造](../../../raw/current/scripts/components/mast.lua#L36-L44) 定义 `OnBoatRemoved = function() self.boat = nil end`，[SetBoat](../../../raw/current/scripts/components/mast.lua#L108-L127) 将其监听到船的 onremove。[EntityScript:Remove](../../../raw/current/scripts/entityscript.lua#L1730-L1751) 先推 onremove，再调用组件 OnRemoveEntity；[PushEvent](../../../raw/current/scripts/entityscript.lua#L1312-L1326) 同步执行普通监听。因此在平台组件 [销毁桅杆](../../../raw/current/scripts/components/walkableplatform.lua#L24-L42) 前，mast.boat 已可被清空；桅杆的 [最终选名](../../../raw/current/scripts/components/mast.lua#L59-L74) 不再能靠 boat 非空命中。
- 具体反例：[蟹王对 leakproof 平台直接调用 InstantlyBreakBoat](../../../raw/current/scripts/prefabs/crabking.lua#L1037-L1043)，该函数 [直接 Remove、不推 death](../../../raw/current/scripts/prefabs/boat.lua#L463-L480)。此前未置 boat_death 的正常已绑定桅杆，不应被概括为必出沉没动画。
- 建议：以移除时 `boat_death`／`boat` 的真实状态区分路径。正常先 death 的船仍可凭 boat_death 出 FX；也不要反向写成“所有 InstantlyBreakBoat 都不出”，因为 [漩涡传送门先把 health 降到死亡再快速拆船](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L235-L240)。同步案例、陷阱和索引。

### R06 · P2：船件页引用公共加载器却未声明该来源

- 页面：[boat_mast_sink_fx.md:22](../../../wiki/reference/prefabs/boat_mast_sink_fx.md#L22)～第 23 行引用 `prefabs/fx.lua`，但 [sources](../../../wiki/reference/prefabs/boat_mast_sink_fx.md#L10) 未列它。
- 建议：补入 sources 并定向刷新该页基线；或只链接已核验的公共协议页、删掉本页不再直接采用的源码论据。不能把目前 19 组来源 unchanged 当作已覆盖这个文件。

### R07 · P2：death 状态不生成 FX，不等于致死碰撞不生成 FX

- 页面：[boat_mast_sink_fx.md:92](../../../wiki/reference/prefabs/boat_mast_sink_fx.md#L92) 写“health 到 0 走 death 状态无 FX”；[索引第 37 行](../../../wiki/reference/prefabs/fx-table-world-index.md#L37) 缩成“死亡不出”。第 59 行仅描述 death 状态自身，范围较准确，但不足以纠正前两处概括。
- 依据：[hullhealth 碰撞分支](../../../raw/current/scripts/components/hullhealth.lua#L168-L183) **先**向 bumper 推 boatcollision，**后**扣血。此前不 busy 且存活时，[onhit](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L198-L208) 会进入 [hit 并立即生成 FX](../../../raw/current/scripts/stategraphs/SGboatbumper.lua#L40-L51)，之后即使同次扣血归零、进入 [death](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L20-L36)，先前生成的 FX 也不会因此撤销。
- 建议：改为“death 状态不额外生成；致死碰撞仍可能先在 hit 生成”。busy 守卫只阻挡 onhit 入口；另一路 onhealthchange 降级没有相同 busy 守卫，勿扩写成 busy 时所有 FX 都被阻挡。

### R08 · P2：灯笼读档分支与 POPULATING 的解释相互矛盾

- 页面：[boat_mast_sink_fx.md:67](../../../wiki/reference/prefabs/boat_mast_sink_fx.md#L67) 写“读档不触发（OnLoad 走 nofuel 分支才生成）”，第 68 行又说无燃料读档重走 nofuel，容易被理解为无燃料存档例外生成气球。
- 依据：[OnLoad](../../../raw/current/scripts/prefabs/miniboatlantern.lua#L195-L201) 确会调用 nofuel，但 [SpawnBalloonFX](../../../raw/current/scripts/prefabs/miniboatlantern.lua#L104-L124) 只有 `not POPULATING` 才 Spawn；隐藏符号等恢复动作位于该门禁外。常规世界加载在 [PopulateWorld 入口置 true](../../../raw/current/scripts/gamelogic.lua#L358-L364)，[遍历 SpawnSaveRecord](../../../raw/current/scripts/gamelogic.lua#L613-L620) 后到 [加载结束才置 false](../../../raw/current/scripts/gamelogic.lua#L645-L655)；[SpawnSaveRecord](../../../raw/current/scripts/mainfunctions.lua#L494-L503) 同步 SetPersistData／OnLoad。
- 建议：明确正常世界读档期间 nofuel 复原状态但不生成气球；运行时耗尽仍生成。不要把这个范围扩大到任何 POPULATING=false 的手工 SetPersistData 场景。

### R09 · P2：船炮撞墙生成点不只包括其他船上的墙

- 页面：[cannonball_used.md:42](../../../wiki/reference/prefabs/cannonball_used.md#L42) 将飞行碰撞对象写为“其他船的墙”。
- 依据：[OnUpdateProjectile](../../../raw/current/scripts/prefabs/cannonballs.lua#L143-L169) 的 `on_other_boat` 实际为 `selfboat == nil or target:GetCurrentPlatform() ~= selfboat`；发射船存在时，陆地墙的当前平台 nil 同样不同于发射船；无有效发射船时更直接满足。其后才检查墙、health 与未死亡并生成 FX。
- 建议：写为“排除发射者所在船上的墙；陆地墙和其他船上的墙均可能命中”，并保留目标过滤／健康门禁，不把变量名直接译为完整适用范围。

### R10 · P1：删除注册可能中断后续码头伤害，不能保证结算不受影响

- 页面：[cannonball_used.md:77](../../../wiki/reference/prefabs/cannonball_used.md#L77) 断言“删除／替换这 4 个注册不影响溅射、船血、漏水与码头伤害”。“FX 自身不负责伤害”成立，但不能推出可任意删除注册。
- 依据：[船炮落地分支](../../../raw/current/scripts/prefabs/cannonballs.lua#L127-L139) 的顺序是 `SpawnPrefab("cannonball_used").Transform:SetPosition(...) → DamageDockAtPoint → inst:Remove()`。若移除注册／令生成返回 nil，未判空的 Transform 访问会报错，后面的码头伤害和移除无法完成。[迫击炮](../../../raw/current/scripts/prefabs/mortarball.lua#L154-L163) 虽先结算码头伤害，也仍在 FX 之后才移除炮弹。[SpawnPrefab](../../../raw/current/scripts/mainfunctions.lua#L429-L437) 不提供必定成功的保证。
- 建议：保留“伤害由投射物结算”，将无影响的范围限定为保留有效实体／Transform、正常返回的纯表现替换；移除生成调用须保留后续结算与清理顺序，不推荐删注册来关闭效果。同步第 47 行“全部先于／并列于本族生成”的概括：船炮码头伤害实际上位于 FX 生成之后。

### R11 · P3：八名中四名带声，剩余应为四名

- 页面：[boat_mast_sink_fx.md:22](../../../wiki/reference/prefabs/boat_mast_sink_fx.md#L22) 写“防撞条四名带撞击音效，其余五名无音效”。
- 建议：改为“其余四名”；三桅杆＋一灯笼，注册表本身没有漏第九名。

### R12 · P3：两处“Wiki 待建”已有实际页面

- [boat_grass_erode.md:71](../../../wiki/reference/prefabs/boat_grass_erode.md#L71) 的 `degrade_fx_grass`／`degrade_fx_ice` 已由 [junk_break_fx.md](../../../wiki/reference/prefabs/junk_break_fx.md#L29) 收录，应补到该主要页的链接。
- [fx_boat_crackle.md:65](../../../wiki/reference/prefabs/fx_boat_crackle.md#L65) 的 `ice_crack_grid_fx` 已有 [独立页](../../../wiki/reference/prefabs/ice_crack_grid_fx.md#L19)，应移除待建字样并链接。该页第 35 行只说 fx_ice_pop 不在本批范围，不等于声称缺页，无需误改其范围说明；可就近补 junk_break_fx 链接方便导航。
- 本项只修一层关系，不要求递归复审两个关联页或扩大本批范围。

## 本轮实际校验

以下均为本次运行，不沿用原批交付数字。

| 检查 | 实际范围／结果 |
|---|---|
| validate_page --check-indexes | 4 页＋fx-table-index／fx-table-world-index，6／6 passed，导航 passed，issue 0 |
| validate_source_links --strict | 4 页，183 处行引用、164 个独立目标、55 个源码文件；error 0、warning 0、skipped_unsupported 0 |
| validate_source_hashes check --fresh-hours 0 | 4 页，65 组已声明来源比较全部 unchanged；changed／sources_changed／invalid／unrecorded 均为 0，未 record |
| 正文引用与 sources 对照 | 两页漏项：侵蚀页 13 文件、船件页 1 文件；另两页未发现直接引用漏声明。哈希通过不覆盖这些漏项 |
| 名称与归属 | 24 名注册／aliases／正文齐全；目标名无跨页主要归属冲突；简中精确 NAMES 键均未命中 |
| 生成点反查 | scripts 全域 Lua 字面名与共享前缀检索，未截断；补查 boat_crackle／sink_fx 字段、boat_bumper_hit_／_timeout_fx 拼名、两炮弹变量选名；17 处 PushEvent("spawnnewboatleak") 数量与逐处上下文复核 |
| 章节读取 | 四页各一章，使用标题原文（含中文标点／引号／箭头）调用 query_wiki read，均退出 0；只验证入口可读，不证明章节结论正确 |

章节读取对象：`boat_grass_erode#水草补丁到期（repaired_kelp_timeout_fx）`、`fx_boat_crackle#复刻"预警 → 延迟破坏"节奏`、`boat_mast_sink_fx#桅杆沉没三名：随宿主销毁的选名与沉船链`、`cannonball_used#常见陷阱与限制`。本轮未重建查询缓存，也未用 stale search 结果作检索验收。

开始固定的 56 个输入（四页、已声明来源、BUILD、基线和两个索引）到报告收口时未变；补充采用的 12 个源码输入在收口前后二次核对，合计 68 个输入。没有改动他人的知识页或基线。报告本地 Markdown 链接路径／物理行区间及 `git -c core.safecrlf=false diff --check` 另作收口检查。

未覆盖：游戏运行、专服／多客户端实测、二进制动画资源内容及引擎内部行为；未重跑原有四段 Lua 示例的语法探针，也没有执行这些示例。第 R04 的坐标接口仍应随修例核验，本报告没有凭脚本调用外观断言引擎 Map 接口的重载签名错误。关联机制页不在整页复审范围。

## 后续轮次

本轮四页计入“已独立检视”，不等于问题已修或重新 accepted。累计已检视 **27／158 页**，尚余 **131 页**；修后复验不重复扣数。

下一轮按原生成批 5，**5 页、20 注册**：`ink_puddle_land`（4）、`flotsam_puddle`（2）、`merm_splash`（3）、`bee_poof_big`（4）、`crab_king_bubble1`（7）。本轮不提前复审该批。

## 修后复验（2026-11-05）

上次复验结论（历史记录，最新状态见文末“最终收口复验”）：**原 12 项中，2 项误报撤回、8 项修复闭环、2 项仍有表述残留；该次修订另引入 2 项精度／来源问题。当时待修 P1×0、P2×4、P3×0，见 R13～R16。**

### 误报更正

- **撤回 R01**：[fx.lua L3401](../../../raw/current/scripts/fx.lua#L3396-L3408) 与 [L3415](../../../raw/current/scripts/fx.lua#L3410-L3429) 明确设置 `SetScale(0.75,0.75,0.75)`，海獭窝船的两条目确实具有该缩放。原页面“同 fn”没有这项参数错误。首次检视误读了已经读取的源码，不是快照变化。
- **撤回 R04**：[world.lua L555](../../../raw/current/scripts/prefabs/world.lua#L551-L555) 明确执行 `inst:SetPrefabName("world")`，注释直接说明用于 postinitfns；[mainfunctions.lua](../../../raw/current/scripts/mainfunctions.lua#L388-L407) 因而先按 world 派发，再在请求名不同的情况下按原请求名（正常地表为 forest）派发。原 `AddPrefabPostInit("world", ...)` 有效，不需要换 Hook。首次检视遗漏了工厂内部显式改名。

此次复验对照前次固定输入：源码与 BUILD 均未变化，只有四篇知识页、基线、fx-table-world-index 发生修订，不能将这两项撤回解释为游戏更新。

| 原项 | 当前状态 | 核对结果 |
|---|---|---|
| R01 | 撤回 | 两海獭残影均有 0.75 缩放，原页正确 |
| R02 | 闭环 | 13 个事件源已补入 sources，本页 27 组基线一致 |
| R03 | 部分修复 | 第 57～58 行及验证记录已补 LongUpdate，但第 103 行仍是旧说法，见 R13 |
| R04 | 撤回 | world Hook 有效；新增解释仍需说明双重派发，见 R15 |
| R05 | 闭环 | 正常 death 与无 death 快速拆船的状态／时序已区分，案例、陷阱、索引已同步 |
| R06 | 闭环 | 原缺项 prefabs/fx.lua 已补；新增另一引用漏项单列 R16 |
| R07 | 闭环 | death 状态不额外生成与致死碰撞先出 hit 已区分，索引已同步 |
| R08 | 闭环 | nofuel 恢复状态与 POPULATING 抑制生成已分开 |
| R09 | 闭环 | 已说明陆地墙与其他船上的墙均可能命中 |
| R10 | 部分修复 | 第 77 行及验证记录已撤回删注册的安全保证，第 47 行仍有过强限制，见 R14 |
| R11 | 闭环 | “其余五名”已改四名 |
| R12 | 闭环 | degrade_fx 与冰裂纹已链接实际页面，fx_ice_pop 也补了主要归属链接 |

### R13 · P2：LongUpdate 正文修正后，陷阱仍保留“只有两处调用”

- 页面：[boat_grass_erode.md:103](../../../wiki/reference/prefabs/boat_grass_erode.md#L103) 仍写“SetRepairedTime 只有 repaired_kelp 分支与存档恢复调用”，与已经修正的第 57～58 行、[LongUpdate 的重排调用](../../../raw/current/scripts/components/boatleak.lua#L323-L337) 冲突。
- 建议：只改这一处旧断言，写“原版到期任务对应 repaired_kelp，LongUpdate 可重排或补算直接到期”；保留其他修复态没有该任务的边界，不用再复制整条流程。

### R14 · P2：不能删注册，不等于不能安全省略 FX 生成语句

- 页面：[cannonball_used.md:47](../../../wiki/reference/prefabs/cannonball_used.md#L47) 新写“本族不承载结算，但生成调用不可省略”。第 77 行对删注册导致 nil 访问的修正是正确的，但这句又把限制扩大到了生成语句本身。
- 依据：[船炮 OnHit](../../../raw/current/scripts/prefabs/cannonballs.lua#L127-L139) 的 DamageDockAtPoint 和 Remove 是后续独立语句，不消费 FX 实例；安全跳过 FX 生成语句、保留后续结算与清理，与保留调用却删除注册导致报错是两种操作。
- 建议：改为“生成失败不得中断后续结算与清理”；不用追加实现示例或改动游戏源码。

### R15 · P2：新增 world Hook 解释遗漏按原请求名的第二次派发

- 页面：[fx_boat_crackle.md:91](../../../wiki/reference/prefabs/fx_boat_crackle.md#L91) 新增“PostInit 按该名派发，不按存档 map.prefab（地表为 forest）查表”。这足以说明 world Hook 有效，但排除了另一个实际存在的入口。
- 依据：[mainfunctions.lua L394-L406](../../../raw/current/scripts/mainfunctions.lua#L392-L407) 先取实例名的函数；若实例名不同于 SpawnPrefab 请求名，还会执行请求名对应的函数。正常地表请求 forest、实例名 world，所以两种 Hook 都可到达。
- 建议：压成一句“MakeWorld 将实例名设为 world，派发先走 world，再走不同的请求名（地表 forest）”；保留现有 world 示例。若直接引用派发实现，按现有规则补 sources 与相应页基线。

### R16 · P2：船件页修订新增漩涡引用，sources 尚未同步

- 页面：[boat_mast_sink_fx.md:49](../../../wiki/reference/prefabs/boat_mast_sink_fx.md#L49) 新增对 [prefabs/oceanwhirlbigportal.lua](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L235-L240) 的直接证据引用，但第 10 行 sources 仍缺该文件。原 R06 的 prefabs/fx.lua 已修，不是同一文件重复报错。
- 建议：补入这个实际采用的证据文件，并只刷新该页来源基线；当前 20 组 unchanged 不包含它。

### 本次复验实际检查

- `validate_page --check-indexes`：四页＋两个索引，6／6 passed，navigation passed，issue 0。
- `validate_source_links --strict`：191 处行引用、169 个独立目标、55 个源码文件，error 0、warning 0、skipped_unsupported 0。
- `validate_source_hashes check --fresh-hours 0`：80 组已声明来源全部 unchanged，changed／sources_changed／invalid／unrecorded 均为 0；未 record。
- 正文直接引用与 sources 对照：侵蚀／crackle／炮弹页无漏项；船件页仅上述新增漩涡文件未声明。
- 原四个标题原文章节再次 read，全部退出 0；未重建查询缓存，未声称完成基于缓存的 search 验收。
- 本次固定 67 个输入（四页、已声明与直接引用来源、必要派发实现、BUILD、基线、两个索引），收口前后复核未变。新增报告链接路径／物理行区间及 diff --check 定向检查。

本次只修正／追加本报告与日志；不修知识页、不刷新基线、不改索引或 accepted 状态、不提交推送。未运行游戏，未执行 Lua 示例或重跑语法探针。待独立检视仍为 131 页，下一轮范围不变。

## 最终收口复验（2026-11-05）

**第 3 轮可以收口：R13～R16 均已闭环，本批无待修 P1／P2／P3。** R01、R04 两项误报继续保持撤回，不计为页面修复成果。此结论限于本批四页与已报告问题的源码／文档验收，不代表游戏运行验证或全库通过复审。

| 项目 | 当前修订与核验 |
|---|---|
| R13 | 侵蚀页第 57、103、108 行同步说明 LongUpdate 重排／直接到期，并区分 boatleak 动态漏点与 HullHealth 碰撞漏点两条存档恢复；对照 boatleak.lua L306-L337、hullhealth.lua L272-L288，旧调用范围断言已清除 |
| R14 | 炮弹页第 47、77 行区分“安全移除／替换生成调用”与“只删注册、留下 nil 访问”；后续码头伤害与 Remove 顺序明确保留，与 cannonballs.lua L127-L139 一致 |
| R15 | crackle 页第 91 行明确先按实例名 world、再按不同请求名 forest 派发；保留有效 world Hook，新增 mainfunctions.lua 已进入 sources 与基线 |
| R16 | 船件页 sources 已加入 prefabs/oceanwhirlbigportal.lua，当前该页 21 组来源一致；原新增引用不再漏声明 |

本次实际复跑：

- 四页＋两个索引 `validate_page --check-indexes`：6／6 passed，navigation passed，issue 0。
- 四页 `validate_source_links --strict`：193 处行引用、171 个独立目标、56 个源码文件，error 0、warning 0、skipped_unsupported 0。
- 四页 `validate_source_hashes check --fresh-hours 0`：82 组已声明来源全部 unchanged，changed／sources_changed／invalid／unrecorded 均为 0，未 record。
- 正文直接源码引用与 sources 对照：四页均无漏项。sources 分别为 27／17／21／17 项。
- 本次固定的 66 个输入在收口前后核对未变；报告本地链接路径／物理行区间及 diff --check 定向复核。

本次只追加／澄清检视报告与日志，没有修知识页、索引、基线或 accepted 状态。未运行游戏、未重跑 Lua 示例与章节读取；章节标题未变，四章入口此前复验均退出 0。未提交或推送。累计已独立检视仍为 27／158 页，余 131 页；下一轮为原批 5 的五页／20 注册，按既定轮次继续。
