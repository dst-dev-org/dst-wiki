---
title: "源码更新影响说明：build 26148036 → build 26230496"
created: "2026-10-27"
dst_build: "steam-buildid:26230496"
status: active
---

# 源码更新影响说明 build 26230496

快照 26148036 → 26230496（2026-10-27），万圣夜 2026 内容更新。全库 154 个源码文件变动：147 改 / 7 增 / 0 删；哈希基线全检受影响知识页 350/420（869 处真变化对比，行尾/大小写/删除均 0，sources_changed 2，漂移队列即页面处理队列）。

**版本与快照身份**：起始 build 26148036（旧快照 `raw/archive/build-26148036`，即先前 current）；目标 build 26230496（新快照 `raw/current`，BUILD.txt 同值）。本报告只覆盖这一起止对；其他起点另建报告。

**阅读方式（渐进式）**：本页只放主题与决策规则，逐文件明细在 5 个 detail 文件。兼容性相关条目带稳定编号（C／R／P／SB／M-序号），适配结论可直接引用编号。适配判断三步：① 读下方适配检查清单，命中即得动作；② 未命中但涉及变化机制的，按主题尾部指针打开对应 detail 文件，按组件名／函数名／事件名定位；③ detail 条目均出自实际 diff（个别标"待核验"处除外）。机器事实（changed-files.txt、check-report.json）在本目录；页面处理队列用 `validate_source_hashes.py check --fresh-hours 0` 的 affected_pages 重建（本目录 check-report.json 即该次全检），页面核验状态以各页 dst_build 为准。

## 本轮主题（模组影响分诊）

- **通用击退（knockback）SG 框架**：commonstates 新增 `CommonHandlers.OnKnockback()` 事件工厂＋`CommonStates.AddKnockbackStates(...)` 双状态（普通击退/落地击退，物理推离＋LIMITS 碰撞临时清除＋入水/溺水/唤醒时间线）；`explosive:SetKnockbackData` 使爆炸可推人（伤害与击退解耦，`forcepvp` 时 PVP 只推不伤）；49 个生物 SG 接入，`sg.mem.noknockback` 可实例级免疫。击退中不可入睡（onsleep 守卫）。mod 自定义 SG 可直接复用框架；玩家侧 bucked 动画按新 `lyingdown` 状态标签选 low/high。 （明细：[detail-sg.md](detail-sg.md)、[detail-components.md](detail-components.md)）
- **拟态揭示配置化**：`ShouldItemMimicBeRevealedFor(item, user, action)` 三参化（第三参可缺省），删除 `castonrecipes` 标签特例，改为新组件 `itemmimicconfig` 按物品声明强制揭示动作（staff_common＝CASTSPELL、king_cane＝GIVE）；GIVE 作用于拟态目标（宝箱怪）时立即 TurnEvil；拟态失败台词统一归 GENERIC。 （明细：[detail-root.md](detail-root.md)、[detail-components.md](detail-components.md)）
- **万圣夜 2026 内容**：新动作 SWEEP（清扫，sweeper/sweepable 组件对）＋三件新物：leafpile 落叶堆（三档生长、trick/treat/debris 掉落、强制惊吓、桦树瓦限定、leafpilemanager 世界组件管理）、witchbroom 女巫扫帚（finiteuses、living 变体加速＋飞行跑态、livingtree_root 转正）、pumpkin_bomb 南瓜炸弹（complexprojectile＋爆炸击退）。`WORLD_SPECIAL_EVENT` 默认 HALLOWED_NIGHTS。新 tuning 键块可覆盖。 （明细：[detail-prefabs.md](detail-prefabs.md)、[detail-sg.md](detail-sg.md)）
- **VRS（虚拟房间集）围墙语义强化**：新全局 `IsPointInWalledOffVRS`；`AddIsTileInvalidForPathing_VirtualRoomSet` 改为只认内置 VIRTUALROOMSETS 名（**自定义名注册被静默丢弃——必须适配**）；新 `Map:GetVirtualRoomSetNameAtPoint`；海上找岸点排除围墙 VRS；ATRIUM lobby 态寻路豁免；两栖生物新增 `ms_enterwater`/`ms_exitwater` 即时事件（击退态消费）。 （明细：[detail-root.md](detail-root.md)、[detail-components.md](detail-components.md)、[detail-prefabs.md](detail-prefabs.md)）
- **防误用断言（仅 dev 分支）**：`HasTag`/`HasTags`/`HasOneOfTags`、`HasStateTag`/`HasAnyStateTag`、`net_event` 三参误用均在 dev 分支 assert；release/staging 行为不变。官方同步清理了 5 处 net_event 三参误用（监听名与 netvar 名统一）。 （明细：[detail-root.md](detail-root.md)、[detail-sg.md](detail-sg.md)、[detail-prefabs.md](detail-prefabs.md)）
- **Boss 平衡**：Charlie Boss 反斩范围 8→14 且远程/近战区分（4/5+半径）、Shrouden 16→20，反斩半径改按目标半径计算——实际覆盖显著扩大。 （明细：[detail-sg.md](detail-sg.md)、[detail-root.md](detail-root.md)）
- **杂项行为**：hounded 保底升级波无视警告期/年龄门且波级一次性消费；amphibiouscreature 无海洋世界短路；slingshotammo 无攻击者也派 attacked 事件；tumbleweed aggro 掉落按世界开关门控；rocky 酸雨组件按 CanPointHaveAcidRain 特判；南瓜/巨型南瓜腐烂翻倍（9600/38400）；ConcatArrays nil 容错。 （明细：[detail-components.md](detail-components.md)、[detail-prefabs.md](detail-prefabs.md)、[detail-root.md](detail-root.md)）
- **资源与表现（一笔带过）**：本地化 3 文件各约 532 行、17 角色语音新条目、皮肤 release_group 187（hallowed 建筑系列）、前端横幅/音乐切 rifts8、6 个 leafpile FX 注册。

## 适配检查清单（条件 → 动作）

| 如果你的模组… | 判断／动作 |
|---|---|
| 用自定义 roomset 名调 `AddIsTileInvalidForPathing_VirtualRoomSet` | **必须适配**（[R-3](detail-root.md)）：现只认内置 `VIRTUALROOMSETS` 名，自定义注册被静默丢弃 |
| 调用／包装 `ShouldItemMimicBeRevealedFor` | 两参调用仍兼容（缺省回退 lastaction）；但 `castonrecipes` 特例已移除，强制揭示改走 `itemmimicconfig`（[R-3](detail-root.md)、[C-4](detail-components.md)） |
| 监听 `musicdirty`／`killdirty` 等旧事件名（月石守护者/眼球/柳条火焰） | 建议调整：netvar 事件监听名改为 netvar 全名（[P-27](detail-prefabs.md)～[P-31](detail-prefabs.md)） |
| 自定义 SG 想响应爆炸击退／自制可被击退生物 | 接 `CommonStates.AddKnockbackStates`＋`CommonHandlers.OnKnockback()`；实例免疫用 `sg.mem.noknockback`（[SB-1](detail-sg.md)） |
| 自定义爆炸物想要击退/PVP 击退 | `explosive:SetKnockbackData`；`forcepvp` 时 PVP 目标只击退不掉伤（[C-7](detail-components.md)） |
| 依赖 hound 波次的新手保护（警告期/年龄门） | 保底升级波现在无视这些门且波级一次性消费（[C-8](detail-components.md)） |
| 从海上找上岸点／监听 ms_enterwater／ms_exitwater | 找岸点排除围墙 VRS（[R-11](detail-root.md)）；两栖生物进出水有新即时事件（[C-9](detail-components.md)） |
| 无海洋世界挂 amphibiouscreature | 不再启动更新/监听 death（[C-9](detail-components.md)） |
| 处理 pumpkin／巨型南瓜保质期 | 腐烂时长翻倍 4800→9600／19200→38400（[R-19](detail-root.md)） |
| 弹弓命中监听 `attacked` 事件 | 现在无有效攻击者也派发（data 无 attacker 字段，判空）（[P-11](detail-prefabs.md)） |
| dev 分支开发误用 HasTag 多参／net_event 三参 | dev 分支会 assert；release 不变（[R-5](detail-root.md)、[SB-2](detail-sg.md)、[R-10](detail-root.md)） |
| 皮肤硬编码 versioning_skins VERSION 或 release_group | 版本位 187（[M-6](detail-skins-loc.md)） |
| 覆写查理 Boss／Shrouden 反斩逻辑或数值 | 范围与半径基准变化（[SB-6](detail-sg.md)、[SB-7](detail-sg.md)、[R-22](detail-root.md)） |

## 分类明细导航

| 明细文件 | 覆盖 | 建议检索词 |
|---|---|---|
| [detail-components.md](detail-components.md)（C-1～C-13） | components/* 13 文件 | sweeper、sweepable、leafpilemanager、itemmimicconfig、spooked、explosive、knockbackdata、hounded、amphibiouscreature、ms_enterwater、GetVirtualRoomSetNameAtPoint |
| [detail-root.md](detail-root.md)（R-1～R-23） | 根脚本＋map/＋screens/ 23 文件 | SWEEP、ShouldItemMimicBeRevealedFor、IsPointInWalledOffVRS、VIRTUALROOMSETNAMETOHASH、WORLD_SPECIAL_EVENT、dev assert、ocean_util、tuning、ConcatArrays、scrapbookdata |
| [detail-prefabs.md](detail-prefabs.md)（P-1～P-36） | prefabs/* 36 文件 | leafpile、witchbroom、pumpkin_bomb、livingtree_root、staff_common、king_cane、pigking、tumbleweed、rocky、slingshotammo、useabletargeteditem、_musicdirty |
| [detail-sg.md](detail-sg.md)（SB-1～SB-10＋49 文件附表） | stategraph 框架＋58 文件 | AddKnockbackStates、OnKnockback、knockbacklanded、onsleep、SGwilson、lyingdown、bucked、SGcharlie_boss |
| [detail-skins-loc.md](detail-skins-loc.md)（M-1～M-6） | 语音 17＋本地化 3＋皮肤 4 | speech、strings.pot、skins_defs_data、versioning_skins |

## 新增与删除

- **新增（7）**：components/itemmimicconfig.lua、components/leafpilemanager.lua、components/sweepable.lua、components/sweeper.lua、prefabs/leafpile.lua、prefabs/pumpkin_bomb.lua、prefabs/witchbroom.lua —— 全部逐条展开于 C-1～C-4 与 P-1～P-3。C-1～C-4 四组件已建知识页（2026-10-28 建页批：reference/components/ 的 sweeper、sweepable、leafpilemanager、itemmimicconfig，行号证据以页面为准）；P-1～P-3 prefab 页仍待维护任务认领。
- **删除（0）**：无。

## 覆盖与审校状态

- **文件覆盖**：detail 条目 95 条覆盖 154/154 修改文件（多文件合并条目见 SB-10 附表与 M-1/M-2）；changed-files.txt 对账无遗漏。
- **语义审校**：本轮全部条目基于实际 diff 通读；1 条标待核验（P-16 AOEUtil.Work 意图）；强结论（击退框架行为、拟态揭示条件、VRS 注册收紧）均有新旧对照证据。
- **知识页迁移**：**完成**（2026-10-28）。漂移队列 350 页全部处理——沿用-70 批＋常规迁移批A 38 页＋批B 312 页（见下方沿用记录）；收尾全库 check 2,996 组对比全部一致、affected 0、sources_changed 0。

## Wiki 沿用记录

- **常规迁移-批B（枢纽域 312 页）**（2026-10-28）：26148036 → 26230496。**范围**：漂移队列余量全部 312 页（actions.lua／componentactions.lua／entityscript.lua／SGwilson.lua／tuning.lua 等全部枢纽引用页群，含 cookbook 各页与 sources_changed 2 页 childspawner／health）。**流程**：plan_source_link_updates 全批预览（10,751 引用：9,409 未变／1,144 机械迁移——581 hunk 补丁 259 文件全应用＋38 处 CRLF 建议手工）；review 50 项逐条人工处理——actions.lua 25 锚按 +10 偏移（SWEEP 定义 +3、GIVE +7、动作函数拟态三参化均不换行）迁移，amorphous（SGchester 重构锚点）／flotationdevice（washed_ashore +66）／sleeper（onsleep 守卫改写）／slipperyfeet／virtualroommanager（constants 哈希表块）逐个 grep 定位修正，netvars:29-32／lightflier:46-67／componentutil:1／constants:3049-3053 经源码核对确认未变保留。**标签一致性**：312 页扫描修复 437 处。**校验**：validate_page 312/312；严格链接 10,751 处 0 错 0 警。**record**：312 页集中 record（sources_changed 2 页基线重建，状态消除）。**收尾全检**：全库 check 2,996 组对比全部一致、affected 0——漂移队列清零。**未覆盖项**：新行动作语义（SWEEP/GIVE 拟态）未在无记载面的页面展开，见报告 R-1/R-3；静态迁移不证明语义完整，行为级问题随各域后续复查。
- **常规迁移-批A（组件域 38 页）**（2026-10-28）：26148036 → 26230496。**范围**：受影响页中 sources 含 explosive／spooked／amphibiouscreature／hounded／locomotor／map／playercontroller／brushable 变化的 38 页（组件自身页＋消费页，含 cookbook/equipment/hand）。**流程**：plan_source_link_updates 全批预览（1,419 引用：1,158 未变／219 机械迁移补丁全应用／46 review 逐项人工处理）；语义修订 5 页——explosive（C-7 SetKnockbackData＋伤害击退解耦＋流程行）、explosiveresist（抗性消费先于 notpvp 门禁的 PVP 积累语义）、spooked（C-6 两新接口行＋ShouldSpook 条件改写）、amphibiouscreature（C-9 has_ocean 短路＋CheckSwimming＋ms_enterwater/ms_exitwater 即时事件）、vault-virtualroom-migration（VRS 锚点重定位，语义未变）；其余 33 页纯锚点迁移。**标签一致性**：38 页扫描修复 111 处滞留标签。**校验**：validate_page 38/38；严格链接 1,424 处 0 错（CRLF 4 页走建议手工应用）。**record**：38 页基线集中 record；未覆盖：常规迁移余量（280−38）页与 sources_changed 2 页。
- **原始哈希等价批量沿用-70**（2026-10-28）：26148036 → 26230496；旧快照 `raw/archive/build-26148036`、新快照 `raw/current`（本目录 check-report.json 即本次全检）。**分支**：原始哈希等价——70 页 sources 所列文件在本起止对内原始字节全部未变（均不在 affected_pages，行尾／大小写／删除均 0），按批量沿用协议不 record、不重写基线。**共享依据**：本报告 5 个 detail 文件全部条目（154 文件 diff 通读）。**资格筛**：active＋verified＋有 dst_build；21 个无 dst_build 导航页（分类索引／主题索引）不适用升版，另 1 页无基线（oldager）单独 record。**页内筛查**：review-needed 标记 0。**枢纽反查**：38 个本轮变化符号（AddKnockbackStates、OnKnockback、knockbacklanded、noknockback、ms_enterwater、ms_exitwater、TryForcingCustomSpook、IsSpookImmune、SetKnockbackData、itemmimicconfig、IsPointInWalledOffVRS、GetVirtualRoomSetNameAtPoint、VIRTUALROOMSETNAMETOHASH、sweepable、witchbroom、leafpile、pumpkin_bomb、starthigh、heavybody、AOEUtil、useabletargeteditem、shouldavoidaggrofn、WantsToMoveForward、ConcatArrays、TryLuckRoll、flyingbroom、lyingdown、WORLD_SPECIAL_EVENT、HALLOWED_NIGHTS、musicdirty、killdirty、FindRandomPointOnShoreFromOcean、PUMPKIN_BOMB、LEAFPILE、WITCHBROOM、SWEEP、sweep、knockback）对 70 页精确检索：仅 mechanisms/spike-launcher.md 命中 "knockback"（sharkboi 冰刺击退事件推送侧，事件名与参数本轮未变；本轮变化在 SG 消费侧框架＝SB-1），不影响页面记载面，排除。**引用检查**：70 页严格链接 2,019 处 0 错 0 警。**页内记录**：各页验证记录补「版本沿用」行链接本节；dst_build 统一升 26230496、updated 2026-10-28。**未覆盖项**：sources_changed 2 页（childspawner、health）与 350 页漂移队列余量走常规迁移；静态无命中不作动态依赖完整证明。
- **页面清单（70）**：api/modinfo/fields.md、components/aoeweapon-base.md、components/armor.md、components/autojiggle.md、components/burnable.md、components/chargingreticule.md、components/charlie_tracker.md、components/circler.md、components/complexprojectile.md、components/cooldown.md、components/curseditem.md、components/custombuildmanager.md、components/damagetyperesist.md、components/deployhelper.md、components/domesticatable.md、components/dryable.md、components/dryingrack.md、components/dryingracksaltcollector.md、components/edible.md、components/entitytracker.md、components/epicscare.md、components/farmplantstress.md、components/followermemory.md、components/fuel.md、components/groomer.md、components/grouptargeter.md、components/hatchable.md、components/heavyobstaclephysics.md、components/insulator.md、components/inventory.md、components/linkeditem.md、components/minionspawner.md、components/named.md、components/objectspawner.md、components/oldager.md、components/perishable.md、components/pethealthbar.md、components/planardamage.md、components/planardefense.md、components/planarentity.md、components/plantregrowth.md、components/preserver.md、components/projectile.md、components/propagator.md、components/prototyper.md、components/rampingspawner.md、components/recallmark.md、components/rechargeable.md、components/recipestockpile.md、components/replayproxy.md、components/researchpointconverter.md、components/resistance.md、components/roseinspectable.md、components/saltlicker.md、components/shard_charlieinfo.md、components/shop.md、components/simplemagicgrower.md、components/skinner-beefalo.md、components/slipperyfeettarget.md、components/squadmember.md、components/stalkermanager.md、components/stretcher.md、components/temperatureoverrider.md、components/thief.md、components/treasuremarked.md、components/unevenground.md、components/waterproofer.md、components/weapon.md、components/witherable.md、mechanisms/spike-launcher.md。
