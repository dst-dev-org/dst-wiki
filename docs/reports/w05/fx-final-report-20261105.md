# 外围 FX 扩库最终报告（2026-11-05）

> 任务：DST Wiki 外围 FX 扩库收尾。380 个外围候选文件逐文件闭环，51 个页组全部建页／合并／排除／blocked，全部新页经主会话复审并 record；U15～U17 九页另经外部独立检视。全库页面与导航通过，strict links 0 error／27 条存量 warning（strict 未通过），来源哈希 0 漂移。**未提交、未推送、未改 `raw`、未运行游戏。**
>
> 当前状态以 [覆盖台账](fx-coverage-ledger-20261101.md) 的「阶段交接」「外围 needs-page 建页清单」「外围批 U 验收检查点」为准；本报告是收尾结论与校验记录。

## 1 结论

- 380 个外围候选全部有结论：**187 清单内闭环＋101 needs-page（51 页组）＋26 blocked＋66 excluded**，四数合计 380。
- 51 个页组全部落地：**50 个新页＋1 个增修页**（组 18 对应两页，组 36／37 合并到 `burntground`）。U1～U17 均有实际验收批次，不再有“计划编号冒充完成”。
- 本阶段 U15～U17 完成 **9 页／19 个计划注册＋1 个 placer**：U15 4 注册、U16 7 注册、U17 8 注册＋`yotr_fightring_kit_placer`。
- 全库页面与导航 0 failed；来源链接 0 error、27 条存量 warning（strict 未通过，均在既有组件页）；来源哈希 7,415 组比较 0 漂移。
- **外部独立检视**（[fx-independent-review-20261105.md](fx-independent-review-20261105.md)）报告 P1×2、P2×7、R-01 全部采纳并修复：U15 P2×4、U16 P1×1＋P2×3、U17 P1×1；九页现均 0 error 0 warning。
- 台账、PLAN、log 的数字已对齐；本报告落盘 `docs/reports/w05/`。未提交、未推送。

## 2 交付

### 2.1 本阶段九页

| 批次 | 页面 | 注册 | 文件 | 档位 | 行／字节 |
|---|---|---|---:|---|---:|
| U15 | `ruinsnightmare_horn_attack` | 1 | 1 | standard | 161／19,539 |
| U15 | `shadowhand_arm` | 2 | 2 | standard | 133／16,043 |
| U15 | `shrouden_eye_fx` | 1 | 2 | standard | 153／17,699 |
| U16 | `hotcold_fx` | 1 | 1 | simple | 88／9,724 |
| U16 | `terrarium_fx` | 1 | 1 | standard | 98／10,708 |
| U16 | `wagboss_missile_target_fx` | 5 | 4 | complex | 178／25,925 |
| U17 | `wx78_taser_projectile_fx` | 1 | 3 | standard | 132／14,700 |
| U17 | `yotr_fightring_torch` | 4＋placer | 1 | standard | 133／15,930 |
| U17 | `spellmasterybuff` | 3 | 1 | simple | 81／9,334 |

- 九页均通过 `validate_page.py` 与 `validate_source_links.py --strict`（0 error 0 warning），均在所选档位的行数／字节上限内。
- 索引登记：`fx-creature-index` 5 行（U15 3＋U16 2）、`fx-world-index` 2 行（U16 1＋U17 1）、`fx-index` 2 行（U17），共 9 行。索引均为 0 问题。
- 本阶段修复了 `fx-creature-index` 的两处既有表格断裂：表头前 6 行游离、表头后中段空行导致后 2 行无表头；修复后 46 行一张表。

### 2.2 覆盖口径

| 类别 | 数量 | 依据 |
|---|---:|---|
| 清单内闭环（既有页／批 A～T） | 187 | 台账批表体首次 accepted 去重 |
| needs-page（51 页组） | 101 | 台账「外围 needs-page 建页清单」，51 组全部 accepted |
| blocked | 26 | 旧 11＋事件组 15；见台账「外围候选」与「未分诊 54 候选之实核·终」 |
| excluded | 66 | 旧 40＋新增 26；见台账各分诊节 |
| **合计** | **380** | 四数相加 |

通用表 504／504 已覆盖，属另一口径；外围 380 与通用表不互相抵扣。51 页组共 50 新页＋1 增修，**页组数不等于页面数，也不等于注册数**。

### 2.3 本阶段关键结论

- **U15**：`ruinsnightmare_horn_attack` 是宿主 combat 结算的双角投射物（attacker 是潜伏梦魇，不是角；每角独立 `targets`；落点固定）；`shadowhand_arm` 两只手臂纯视觉、无自删、宿主清理，普通／遮蔽手部都监听 `enterlight`，`charlie_hand` 只生成普通手臂；`shrouden_eye_fx` 是死亡水洼的可检视眼睛，环是客户端本地 goop，`PUDDLE` 走专属检视文本。
- **U16**：`hotcold_fx` 按玩家到最近玩具的距离缩放，**任务完成会清理而放弃路径不清理**；`terrarium_fx` 夜间 5 秒延迟后出光柱，关闭 10 帧退场，读档可重建；`wagboss_missile_target_fx` 的落点环是客户端本地实体，四个混合注册（电束×2＋电球×2）都以 FX 自身作 attacker；30 基础伤害经电系倍率后非免疫干燥 45／满湿 75、电免疫 15；`SetBeam` 不重算采样缓存，`wagpunk_cagewall_fx` 全快照无生成点。
- **U17**：`wx78_taser_projectile_fx` 是父挂玩家的常驻电壳，爆发时直接 `GetAttacked`，attacker 正常为玩家、目标跟随玩家时改为 FX；原版模块会把默认 50／2 覆盖为 `n×50`／`2+n`（1 模块满蓄能 50／3、2 模块 100／4）；`yotr_fightring_torch` 有 8 支火炬（不是 4），`gosmall` 后 `turnon` 不恢复大火；`spellmasterybuff` 的 buff／orbs 主模拟初始化在缺失的 lavaarena 事件包，`spellmasteryorb` 是唯一不依赖事件包的可见注册。

## 3 校验

### 3.1 页面与导航

命令：

```text
python dst-toolbox/scripts/tools/verify/validate_page.py \
  --wiki-root dst-wiki/wiki \
  --format dst-wiki-skill/references/maintain/write-format.md \
  --source-root dst-wiki/raw/current/scripts \
  --check-indexes
```

结果（存档 `dst-wiki/output/tmp/full_validate_page.json`）：

| 项 | 结果 |
|---|---|
| 页面总数 | 1,146 |
| failed | **0** |
| 结构 issue | **0** |
| 导航 | **passed**，0 issue |
| 索引 | 34 |
| 知识页（不含索引等） | 1,112 |

### 3.2 来源链接

命令：

```text
python dst-toolbox/scripts/tools/verify/validate_source_links.py \
  --wiki-root dst-wiki/wiki --strict
```

结果（存档 `dst-wiki/output/tmp/full_links.json`）：

| 项 | 结果 |
|---|---:|
| 页面 | 1,146 |
| 引用总数 | 29,309（行级 29,301＋整文件 8） |
| error | **0** |
| warning | **27** |
| skipped unsupported | 987（工具不支持的链接形式，存量） |
| strict 结果 | **passed=false**（27 warning；工具退出 1） |
| 唯一目标／来源文件 | 23,046／2,147 |

27 条 warning 全部是**既有组件页**的锚点边界提示（`source-blank` 10、`source-end-blank` 11、`source-structural` 6），分布：`kramped` 7、`sweeper` 5、`sweepable` 4、`wintersfeasttable` 3、`blinkstaff`／`carefulwalker`／`golfable`／`migrationmanager`／`nonslipgritsource`／`shard_mermkingwatcher`／`tackler`／`wax` 各 1。**本阶段九页均为 0 error 0 warning**；全库 strict 因这 27 条既有 warning 未通过（工具退出 1），按人工接受范围记录，未为清零 warning 改既有页。必要时可按批次单独处理这 12 页。

### 3.3 来源哈希

命令：

```text
python dst-toolbox/scripts/tools/verify/validate_source_hashes.py check \
  --knowledge-root dst-wiki --fresh-hours 0
```

结果（存档 `dst-wiki/output/tmp/full_hashes.json`）：

| 项 | 结果 |
|---|---:|
| 页面 | 1,146 |
| 有 sources／无 sources | 1,112／34 |
| 比较组数 | 7,415 |
| unchanged | 7,415 |
| changed／line-ending-only | **0／0** |
| stale／invalid／missing | 0／0／0 |
| passed | **true** |

九页均以 `record --write` 建立基线后复跑 scoped check 为 0 漂移，再跑全库 check 仍 0 漂移。

### 3.4 工作区

```text
git -C dst-wiki        -c core.safecrlf=false diff --check   # 0
git -C dst-wiki-skill  -c core.safecrlf=false diff --check   # 0
git -C dst-toolbox     -c core.safecrlf=false diff --check   # 0
```

`git status --short`：K 170 untracked／122 modified；S 2／11；T 2／5。这些是任务期间的有意未提交状态；**未 commit、未 push**。九页逐页检查：无行尾空白、均以换行结尾。

## 4 台账／PLAN／日志对齐

- 台账「阶段交接」：51 组全部落地、50 新页＋1 增修、0 待建；页组表 51 行全部 accepted；新增 U15／U16／U17 三段验收检查点；尾部补收尾校验数字。
- PLAN：`[x] U15`／`[x] U16`／`[x] U17` 三条；原「外围续作进行中」改为「收尾进行中」，记录 0 待建与全库校验。
- `wiki/log.md`：追加 U15／U16／U17 三条，记录页数、复审修正、校验与索引。
- 交接文档 `fx-handover-20261105.md` 顶部加收尾横幅，指向本报告与台账；旧「待建 9 组」不再作为续跑依据。
- 外部独立检视报告保留在 [fx-independent-review-20261105.md](fx-independent-review-20261105.md)，作为修复前记录；其 P1×2／P2×7 已全部回写本报告、台账与页内。

## 5 遗留与风险

1. **未运行游戏**：所有页面 `validation: source-reviewed`；动画帧长、跨端观感、事件时序均未做运行时验证。
2. **既有 strict warning 27 条**：12 个组件页的锚点边界，非本阶段新增；全库 strict 因此未通过（R-01），本报告已按“0 error、27 存量 warning、strict 未通过”记录，未为清零改既有页。
3. **诚实缺口**：`wagpunk_cagewall_fx` 全快照无生成点；`spellmasterybuff`／`spellmasteryorbs` 的 `buff_postinit`／`orbs_postinit` 在缺失的 `lavaarena_event_server`；`hotcold_fx` 放弃任务不清理、实体移除不显式删 FX。均已在对应页和台账记录，未用“暂未找到消费者”代替结论。
4. **未纳入 380 口径的观察**：`torchfire` 12 个产物（含本页引用的 `torchfire_yotrpillowfight`）无专属页；`lavaarena`／`quagmire` 工厂自足但无快照生成点的素材留待独立任务。见台账「清单外发现」。
5. **工具未完成项**：`inspect_prefab_sources.py` 与候选审计脚本仍属工具箱后续；`verify_page_coverage.py` 未提交（任务要求不提交）。不影响本次知识页验收。

## 6 完成条件核对

| 条件 | 状态 |
|---|---|
| 380 候选全部有结论 | ✅ 187＋101＋26＋66＝380 |
| 51 页组全部建完／合并／排除／blocked 且有实际验收批次 | ✅ 51 行全部 accepted；U1～U17 |
| 全部新页独立复审并修完 P1／P2／P3 | ✅ 全部新页经主会话复审；U15～U17 九页另经外部独立检视，外部 P1×2／P2×7 全部采纳：U15 P2×4、U16 P1×1＋P2×3、U17 P1×1；另校正引用边界 |
| record 0 漂移 | ✅ 九页 record 后 scoped check 0；全库 7,415 组 0 |
| 全库页面与导航通过 | ✅ 1,146 页 0 failed，导航 0 issue |
| 全库 strict links | ⚠️ 本批九页 0 error／0 warning；全库 0 error、27 条存量 warning，strict 未通过（人工接受范围） |
| 台账／PLAN／log／最终报告数字一致 | ✅ 51 组、50 新页＋1 增修、0 待建 |
| 不提交、不推送、不改 raw | ✅ 三仓 `diff --check` 干净，工作区改动保留未提交 |

## 7 附录：命令与存档

```text
K=D:/Code/dst-wiki-develop/dst-wiki
S=D:/Code/dst-wiki-develop/dst-wiki-skill
T=D:/Code/dst-wiki-develop/dst-toolbox
PY=K/.venv/Scripts/python.exe

# 页面／导航
PY T/scripts/tools/verify/validate_page.py --wiki-root K/wiki \
  --format S/references/maintain/write-format.md \
  --source-root K/raw/current/scripts --check-indexes

# 来源链接
PY T/scripts/tools/verify/validate_source_links.py --wiki-root K/wiki --strict

# 来源哈希
PY T/scripts/tools/verify/validate_source_hashes.py check \
  --knowledge-root K --fresh-hours 0

# 三仓空白检查
git -C K -c core.safecrlf=false diff --check
git -C S -c core.safecrlf=false diff --check
git -C T -c core.safecrlf=false diff --check
```

存档：`K/output/tmp/full_validate_page.json`、`full_links.json`、`full_hashes.json`（生成物，不入库）。
