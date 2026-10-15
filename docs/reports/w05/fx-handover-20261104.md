# 外围 FX 扩库交接（2026-11-04）

> 交接对象：下一个 AI 会话。本文只读核验后编写；未改页面、索引、基线，未提交、未推送。
> 事实标注来源：`[重算]` 为本次命令／脚本重算，`[台账]` 为 `K/docs/reports/w05/fx-coverage-ledger-20261101.md`，`[PLAN]` 为 `K/PLAN.md`，`[估计]` 为题面估算。

## 1 项目目标与范围

- 目标：对 `K/output/tmp/fx_perfile_triage.json` 的外围 380 个候选文件逐文件闭环——建页、并入既有页、排除或标 blocked；不以文件数直接换算页面数。
- 通用表 FX（`fx.lua` 及其动态展开）504 名已由批 1～12 闭环；本交接只处理外围批 C～T 与剩余候选。
- 不含：模组兼容、游戏版本迁移、全库组件扩库、运行游戏验证。
- 页面产物为 `K/wiki/reference/prefabs/` 下的 FX 族页；叶级索引为 `fx-index`／`fx-world-index`／`fx-creature-index`；通用表另用 `fx-table-index` 等三张索引。

## 2 当前状态（截至 2026-11-04）

### 2.1 页面与索引 [重算]

命令：`./.venv/Scripts/python.exe ../dst-toolbox/scripts/tools/verify/validate_page.py --wiki-root wiki --format ../dst-wiki-skill/references/maintain/write-format.md --source-root raw/current/scripts --check-indexes`（在 K 根执行）。

| 指标 | 数值 |
|---|---:|
| 页面总数 | 1096 |
| 索引页 | 34 |
| 知识页 | 1062 |
| Prefab 页 | 238 |
| component 页 | 812 |
| 页面／导航结果 | `passed true`，0 failed，0 nav issues |

三个外围 FX 索引现状 [重算]：`fx-index` 79 行／9,054 B；`fx-world-index` 66 行／11,614 B；`fx-creature-index` 48 行／6,947 B。通用表三索引：`fx-table-index` 59 行／16,156 B，`fx-table-world-index` 63 行／15,585 B，`fx-table-character-index` 25 行／2,346 B。

### 2.2 504 个通用表注册名 [重算]

以 `docs/reports/w05/fx-table-coverage-20261030.csv` 的 504 个名字对当前非 archived Prefab 页 frontmatter aliases 统计：

- 504／504 均恰好命中 1 个页面的 aliases；0 个跨页重复注册名，0 个无别名。
- 整个 Prefab 分类有 **21 组**描述性别名跨页重复（如“开包特效”“手杖雪花特效”），均非注册名；[台账] 写“20 组”，两者差 1，需对账。
- 结论：通用表注册名主体归属当前无重复；所有重复都是可重叠的检索措辞，不要机械去重。

### 2.3 外围闭环与剩余 [重算＋台账批次表]

按台账批次表逐项相加（既有页 10 文件／29 注册；批 C～R；S；T）：

| 口径 | 文件 | 注册 | 说明 |
|---|---:|---:|---|
| 既有 + C～R 已验收 | 180 | 349 | 批次表相加 |
| 再加 S（已验收） | 189 | 359 | S 验收见台账 409 行；阶段交接仍写“待复审”，冲突 |
| 再加 T（待复审） | 194 | 388 | T 按 5 文件／29 注册；表头写 8 文件 |
| 台账阶段交接声称 | — | 394 | 与逐批和 359／388 不一致 |

- 380 个候选文件扣除已验收 189 与 T 的 5，剩 **186** 个未闭环（若把 T 计入未闭环则 191）；[台账] 阶段交接写 191，未把 T 单列。
- **54** 个是“台账未提及、尚未分诊”的子集；其余 132 个已有 excluded／blocked／needs-page 结论但尚未统一收尾（按台账 191 口径则为 137）。两者口径不同，不要混用。

### 2.4 工作区状态 [重算]

- K：本文件新增前 236 项未提交（122 modified + 114 untracked），含 159 页返回链修复、台账、PLAN、未跟踪新页；加上本文件后 237 项（122 M + 115 ??）。**不要回退**。
- S：10 个已改文件（含 `scripts/setup_raw.ps1`、`verify-wiki.md`、`write-format.md`、`maintain-version.md` 等）＋2 个未跟踪；均为有意状态。
- T：`PLAN.md`／`README.md`／`references/wiki-validation.md`／`scripts/tests/python/test_setup_raw.py` 修改，新增 `verify_page_coverage.py` 与 `test_page_coverage.py`；均未提交。
- 开发 Skill：`setup_raw.ps1` 与 `bundled-helpers.json` 已同步；另有外部 LICENSE／NOTICE 改动，不要误认为本次任务。
- raw 只读；BUILD.txt 实读为 **26335394**。

## 3 已完成工作（批次 C～T）

| 批次 | 范围 | 文件／页／注册 | 状态 |
|---|---|---:|---|
| C | stafflight／lightning／staffcastfx＋3 个雷文件 | 6／3／17 | 已验收 [台账] |
| D | 瞄准圈＋天气降水＋海面波浪＋施法光源 | 19／4／64 | 已验收 |
| E | 火焰烟雾＋光源实体＋威胁预警 | 30／3／57 | 已验收 |
| F | 宿主附属层＋远古吊灯／石像鬼＋熔炉化石化碎裂 | 10／3／27 | 已验收 |
| G | 装备跟随光效＋天体英雄激光＋藏影女王 | 13／3／22 | 已验收 |
| H | 冲击爆点＋巨鹿激光＋贴地贴水 | 11／3／15 | 已验收 |
| I | 残骸植物＋实验台粒子＋月装置 | 12／3／16 | 已验收 |
| L | 地面贴片＋宿主挂接＋灯具光源 | 22／3／29 | 已验收 |
| M | 附着跟随＋小型一次性物件＋有表现载体 | 15／3／18 | 已验收 |
| N | 命中落点＋传送门投影＋碎屑皮肤 | 8／2／9 | 已验收 |
| O | 皮肤火星＋地面痕迹＋粒子降水 | 9／3／9 | 已验收 |
| P | 阿比盖尔子特效＋海树表现＋怀表传送门 | 8／3／20 | 已验收 |
| Q | 影柱法术＋梦魇鞍具 | 3／2／6 | 已验收 |
| R | 睡眠云＋装备持握层 | 4／2／11（分节列 7 行） | 已验收；文件数待核 |
| S | 影陷阱＋暗影消散＋散件贴地 | 9／3／10（分节列 8 行） | 已验收（台账 409 行） |
| T | 书页挂载＋goosplash＋暗影光源 | 5／3／29（表头 8 文件） | **生成待复审，未 record** |

工具行：只读 `verify_page_coverage.py` 已实现并测 24 项；全量 270 项通过（2 skip）；尚未提交；S `verify-wiki.md` 已加“可选”覆盖自查入口。`inspect_prefab_sources.py` **未实现**。

## 4 未完成工作（按可执行顺序）

### 4.1 先复审批 T

| 页面 | 注册 | 输入文件 | 台账状态 |
|---|---:|---|---|
| `fx_book_mount.md` | 12 | `prefabs/books.lua` | generated，待独立复审 |
| `goosplash.md` | 9 | `prefabs/goosplash.lua` | generated，待独立复审 |
| `nightmarelightfx.md` | 8 | `nightmarelightfx.lua`／`gestalt_head.lua`／`circlingbuzzard.lua` | generated，待独立复审 |

- 3 页已登记 `fx-world-index`，页面与 strict 校验通过，但**未独立复审、未 record**，不能计完成。
- T 表头写 8 文件、表体只列 5 文件；任务口径按 5 文件／3 页／29 注册，复审核对后再更新台账。

### 4.2 已识别未建合并页（约 20 组）

台账分诊批 3／6／8／9 的结论分别报 8／8／9／9 组（含重叠），去重约 20 组；下表列 14 个代表组。[估计/台账]

| 组名 | 代表注册 | 来源分诊批 |
|---|---|---|
| abigail 子特效 | `abigail_retaliation`／`abigail_vex_hit`／`abigail_murder_buff` | 6 |
| floatinglantern 影层 | `floatinglantern_shadow` | 6 |
| fossil spike 基座 | `fossilspike_base`／`fossilspike2_base` | 6 |
| oceanhorror 附层 | `oceanhorror_ripples`／`_attachpivot` | 6 |
| oceantree 表现 | 7 个 ripples／roots／falling 注册 | 6 |
| oceantree_pillar | 3 个 ripples／roots／leaves | 6 |
| pocketwatch 标记 | warp／recall marker | 6 |
| pocketwatch_portal | overlay／underlay／exit_fx | 6 |
| propsign 碎裂 | `propsignshatterfx` | 6 |
| pumpkin bomb 爆炸 | `pumpkin_bomb_explode_fx` | 6 |
| purebrilliance 光符号 | `purebrilliance_symbol_fx` | 6 |
| rift terraformer 视觉 | `lunarrift_terraformer_visual` | 6 |
| rocky boss 表现 | `rocky_snip_fx`／`rocky_dash_fx`／`rocky_boss_shadow` | 6 |
| rope bridge FX | `rope_bridge_fx` | 6 |

其余组（如 `watertree_pillar_ripples`、`mushtree` 焦壳、`forcefieldfx` 等）见台账分诊批 7／8／9 结论；不要直接按“约 20”批量建页，先按宿主/共享机制合并。

### 4.3 未分诊候选 54 文件

`fx_perfile_triage.json` 370 条减台账已提及文件后余 54；54 个路径**全部存在**，分类 [重算]：

| 类别 | 数量 | 代表／说明 |
|---|---:|---|
| 事件模式 → blocked | 15 | 8 个 `lavaarena_*`＋7 个 `quagmire_*`；其中 13 个显式调用 `event_*_data`，`lavaarena_rhinobumpfx`／`quagmire_cooking_buff` 仅按事件目录归类 |
| 非事件且含 FX | 2 | `wx78_taser_projectile_fx`、`yotr_fightring` |
| 其余（多为玩法／内部） | 37 | 逐个按 `fx_perfile_triage.json` 的 `regs[].fx/gp` 与源码结论分诊 |

不要把 54 与 186／191 混为一谈：54 只是台账未提及、尚未分诊的子集；已提及的 132 个（按台账 191 口径为 137）仍需统一收尾。

### 4.4 工具后续

- 候选审计：把 `fx_perfile_triage.json`、500／54 分类与台账状态做成可重复清单；动态注册名（变量、循环、`MakePlacer`、拼接）仍是静态下界。
- 覆盖脚本接入：当前 S 只有可选入口；是否作为批量验收/CI 检查由维护者决定；默认 `--scope prefab`，索引/别名需 `--scope all`。
- `inspect_prefab_sources.py`：未做；若需“按注册名定位源码候选”，另立任务。
- 台账对账：修阶段交接与批次表的数字冲突（S 状态、T 文件数、394 注册）。

### 4.5 收尾

全量 `validate_page --check-indexes`、`validate_source_links --strict`、`git diff --check`；台账与 PLAN／log 同步；最终报告。没有明确任务前不要开始新一批页面。

## 5 工具与校验现状

### 5.1 既有命令（K 根或页面参数相对 `K/wiki`）

```powershell
python ../dst-toolbox/scripts/tools/verify/validate_page.py --wiki-root wiki --source-root raw/current/scripts --format ../dst-wiki-skill/references/maintain/write-format.md --check-indexes
python ../dst-toolbox/scripts/tools/verify/validate_source_links.py --wiki-root wiki "<页面绝对路径>"
python ../dst-toolbox/scripts/tools/verify/validate_source_hashes.py check --knowledge-root . --fresh-hours 0 "<页面相对 wiki 的路径>"
& "../dst-toolbox/scripts/fullmoon-dst/target/debug/examples/parser_probe.exe" --syntax-only "<临时完整示例.lua>"
```

record 的页面参数相对 wiki 根；哈希基线只在来源集合／原始字节／归档变化时处理。

### 5.2 新覆盖脚本（已实现，未提交）

路径：`T/scripts/tools/verify/verify_page_coverage.py`。默认 `--scope prefab`；检查 A 别名归属（confirmed 精确匹配）、B 返回链（登记叶索引）、C 索引预算（仅 `--scope all`）、D 注册覆盖（仅非 archived Prefab 页、精确页面名／别名）。

CLI：`--wiki-root --format [--registrations] [--scope prefab|all] [--missing-scope prefab|all] [--changed] [--severity] [--code] [--page] [--fail-on error|warning|info] [--strict] [--json] [--limit]`。

输出字段：`snapshot／scope／checks／counts／unresolved／hits_total／shown／truncated／omitted／findings／groups／passed／exit_reason`；findings 含 `code／severity／page／name／line／message／pages`。退出 0＝无阻塞，1＝有阻塞或 `--fail-on` 命中，2＝输入失败。

本次实跑 [重算]：默认 `0／0／0`，`clean`；带 FX 清单 270 confirmed 时 `0 error／96 warning／0 info`，`shown 96／omitted 0／truncated false`。已知误报：96 个 coverage-gap 中有 15 个已在正文被提及但没有 aliases（如 `backpack_swap_fx`、`petrified_tree_fx`、`moon_altar_link_fx_spawner`），不能直接当缺页；正文提及不满足 D 的 aliases 判定。字面 `Prefab("...")` 还会漏变量／循环／`MakePlacer`／拼接名，清单只是下界。

S `verify-wiki.md` 已有“覆盖自查（可选）”入口；未接入 CI，也未强制。

## 6 关键口径

- 覆盖 = 页面文件名或 aliases **加正文**；一个注册名一个主要页；额外描述性别名可跨页重叠，不要求全部别名唯一。
- 生成点只认真实调用（`SpawnPrefab`／`GetPooledFx`／`SpawnChild`／动态拼名等）；依赖表、`prefabs` 数组、注释、scrapbook 不算生成；负向结论写检索范围。
- 父子与清理分别核验 Lua `children`、引擎父子、`Follower`／跟随关系；区分宿主实体、网络代理、本地动画；不能仅凭 `SetParent` 推断随父删除。
- 名称只认 `languages/chinese_s.po` 的实际显示名称键；FX 页不要求官方 NAMES，别名可用注册名＋效果描述。名称证据 .po 进 sources。
- 尺寸：standard 240 行／24 KiB，complex 480／48 KiB，simple 120／12，索引 240／24，根导航 80／8；按当前 `write-format.md` 读取，不以字节超限自动升档。
- 新建／拆分页的返回链指向登记叶索引，可另保留 hub；历史页不批量迁移（本轮 105 页是按用户要求的一次性例外）。

## 7 环境与命令

- 工作区 `D:/Code/dst-wiki-develop`；K／S／T 路径见首页；Python `K/.venv/Scripts/python.exe`。
- 快照 `K/raw/current/scripts`，buildid `26335394`；历史快照在 `K/raw/archive/build-26230496` 等，raw 只读。
- 回归：`python -B -m unittest discover -s scripts/tests/python`（T 根；本机全量 270 通过、2 skip；DSH 沙箱需正常权限，临时目录 ACL 会误报）。
- 覆盖脚本默认只报 Prefab；查看索引／非 Prefab／短语别名加 `--scope all`；历史缺返回链加 `--missing-scope all`。

## 8 已知坑与历史教训

1. 台账“阶段交接”把 S 写成待复审，但同文件 409 行已有 S 验收；交接前按批次表和验收段逐项对账。
2. 台账“394 注册”与逐批相加 359／388 不一致；不要用总数做完成率。
3. T 表头写 8 文件、表体 5 文件；以实际行和页面复核为准。
4. `fx_perfile_triage.json` 370 条与台账提及 371 个路径不是同一集合；剩余 54 个全部存在，不能按旧 CSV 的“不存在路径”处理。
5. 字面 `Prefab("...")` 漏 `local name = "..."`、循环、`MakePlacer`；249 个 spawn-only 名（94 FX）是补充信号，不是注册证明。
6. 组件页／归档页不再算注册覆盖；正文提及也不等于 aliases 覆盖；D 的 gap 可能是“无别名”而非“无页”。
7. 21 组重复别名都是中文描述性措辞，不是注册冲突；机械去重会降低检索。
8. 索引提醒按表格数据行，不等于 240 物理行硬限；16 KiB 的 `fx-table-index` 仍在 24 KiB 硬限内，不要仅因提醒拆页。
9. 默认 `--scope prefab` 不会打印索引／非 Prefab／短语别名；“0 发现”不等于全库完整核验。
10. 开发 Skill 的旧 `setup_raw.ps1` 可能污染调用方 `ProgressPreference`；当前修复已用 try/finally 恢复，若见旧副本先同步。
11. Windows Bash 的 `>nul` 会生成名为 `nul` 的残留文件；本次已删除，后续避免。
12. 历史页无差异时不要重读／刷新日期版本；版本维护只在 diff／受影响页范围内做。

## 9 风险与边界

- 所有页面均为 **source-reviewed**，未运行游戏；不得声称 runtime-tested 或行为已验证。
- 54 个剩余候选的事件模式部分是按目录／调用模式判定的估计；15／2／37 需逐文件复核。
- 合并页约 20 组是台账结论去重后的估计，组名／代表注册只列代表项。
- 工具未提交，路径在 T 工作区；不要假设另一 checkout 已有。
- K 236 项既有未提交＋本文件、S/T 未提交；**不要回退、不要提交、不要推送**，除非用户明确要求。
- raw 只读；不得以工具输出替代语义审阅或运行验证。

## 10 下一步建议（优先级）

1. 复审批 T：`fx_book_mount`／`goosplash`／`nightmarelightfx`，修 P1/P2，record，更新台账／PLAN／log。
2. 跑覆盖脚本默认与 FX 清单；先处理 15 个“正文已提及但无 aliases”的候选，再决定 6 个 `_sfx` 是否纳入。
3. 选 3 个高价值合并页先做（建议 oceanhorror／oceantree 组、pocketwatch＋portal、rocky_boss／rope_bridge 中的一个），按宿主/共享机制合并，不按文件一户一页。
4. 分诊剩余 54：15 事件→blocked 记录，2 个非事件 FX 建/并入，37 个玩法按 `fx_perfile_triage.json` 逐项排除或 needs-page。
5. 收尾：全量校验、台账与批次表对账、最终报告；再决定是否启动全量 prefab 覆盖。

## 11 参考入口

- 台账：`docs/reports/w05/fx-coverage-ledger-20261101.md`（批次表、分诊批、阶段交接）
- 计划：`K/PLAN.md` 的“FX 当前覆盖与续作”
- 日志：`K/wiki/log.md` 最近 20 条
- 索引：`wiki/reference/prefabs/fx-index.md`、`fx-world-index.md`、`fx-creature-index.md`
- 机器清单：`K/output/tmp/fx_perfile_triage.json`、`fx-table-coverage-20261030.csv`、`output/coverage/registrations_fx.json`
- Skill：`S/references/maintain/{write-format,source-types,maintain-index,maintain-wiki,verify-wiki}.md`、`S/assets/prompts/source-review.md`
- 工具：`T/scripts/tools/verify/verify_page_coverage.py`、`T/scripts/tests/python/test_page_coverage.py`、`T/references/wiki-validation.md`
