# 外围 FX 扩库交接 v3（2026-11-05，待建清单复核）

> 本文取代 [fx-handover-20261104.md](fx-handover-20261104.md)，是续跑摘要；唯一当前状态在 [fx-coverage-ledger-20261101.md](fx-coverage-ledger-20261101.md) 的「阶段交接」与两张页组表，验收依据在「外围批 U 验收检查点」。本次只核对页面、别名归属、批次记录与候选分诊，不重做全文技术复审或全库校验；既有校验数字保留历史时点。


> **2026-11-05 收尾更新：** U15～U17 九页已完成，51 页组全部 accepted，无待建。§2 的“待建 9 组”与 §4 的排程已被 [fx-final-report-20261105.md](fx-final-report-20261105.md) 与台账「阶段交接」取代；续跑先读最终报告。

## 1 项目目标

外围 FX 扩库收尾：380 个外围候选文件逐文件闭环（建页／并入／排除／blocked），全部新页独立复审＋record，全库校验通过，产出最终报告。**不提交、不推送、不动 raw。**

## 2 当前状态（2026-11-05，U14 后、U15 待续）

### 2.1 权威数字 [实核]

- 当前受管知识／导航范围 **1137 页**（根 index＋reference＋cookbook，不含日志归档与 updates）。U14 交付记录记载全库来源 check 7360 组比较、0 漂移、0 未登记页；本次未重跑该检查。
- 380 候选定稿口径＝**187 清单内闭环＋101 needs-page（51 页组）＋26 blocked＋66 excluded**（`output/tmp/fx_closure_audit.py` 与 `fx_remaining_classify.py` 可重跑验证）。
- 51 页组中 **42 组已有页面及 U1～U14 验收记录**（41 新页＋1 增修页）；**剩 9 组、19 个计划注册，另有 1 个 placer**（见 §4）。组 11 的 spellmasterybuff 三注册未建页，旧 U4 编号误作完成；旧“43 组／剩 8 组 16 注册”作废。“110 注册”不再作为完成累计数，待另按注册归属逐名对账。
- 54 个"台账未提及"候选已全部实核分诊（blocked 15／excluded 26／needs-page 13 文件），结论在台账「未分诊 54 候选之实核」三节。
- 工作区有多方未提交改动；项数随写入变化，接手实读各仓 git status，不沿用历史计数。**不要回退、不要提交、不要推送。**
- 本会话对台账的历史勘误已全部回写（15 条，散见台账批 U 验收检查点各段与"未分诊 54 候选之实核"三节）；批 2/6/7/8/9/10/21 分诊结论中仍以页内实证为准。

### 2.2 U15 中断说明

U15 写作代理（ruinsnightmare_horn_attack／shadowhand_arm／shrouden_eye_fx 三页）曾被用户中途终止，本次复核三页仍无落盘文件。其余待建组也均无拟建页，19 个计划注册在现有非归档 Prefab 页 aliases 中无归属。U15～U17 只是后续排程，不视为当前认领；写入前核对最新工作区与占用。

## 3 协议（必须逐条遵守）

1. **本项目分工**：按当前会话实际授权分派写作，主会话集中处理对账、共享文件与验收；每页技术复审不抽样。历史分工不构成新会话启动子代理的授权。
2. **写作者预热包**：write-format.md、source-types.md Prefab 节、assets/prompts/new-pages.md、2 篇近期验收范例页、目标叶级索引（只读）。参照 U14 批的派发任务书（本会话实际使用的模板文本可在会话记录或 [fx-skill-retro-20261105.md](fx-skill-retro-20261105.md) §2.2 找到要点）。
3. **写作者自检**：指定草稿页面运行 validate_page，不加 --check-indexes；源码引用按 Skill 规则处理，注册名核对 aliases＋正文归属。统一登记后再验导航，未登记不能算最终通过；不将 warning 机械清零代替引用意图判断。
4. **交付报告**：按当前 new-pages 模板；本批涉及注册、生成与名称的页面，补对应源码入口、核验结果与缺口，不另设通用八字段要求。
5. **主对话复审**：对报告的"关键验证声明"逐条回源码实证＋"存疑清单"逐条核查；发现问题派回写作代理修 P1/P2/P3；零发现也要复核（防代理幻觉）。
6. **台账技术分诊不能替代源码证据**：按需回源码核验，已有有效依据可按 Skill 规则复用；不符时页内按实证写，交付报告列勘误建议，主代理当批复核回写台账。
7. **record 与索引登记是主代理职责**：record 用 `validate_source_hashes.py record --write <页>`；索引登记后必须对该页复跑 validate_page --check-indexes 确认 registration 消失。
8. **尺寸**：standard 240 行/24 KiB、complex 480/48、simple 120/12；complex 须页内简记理由；历史页验证记录中的旧预算（200/16、400/32）按写入当时规则理解，不回填。
9. **每批完成**：record＋索引＋台账验收段＋PLAN 条目＋log 条目，再进下一批。

## 4 剩余工作（9 组、19 个计划注册，另有 1 个 placer；计划 3 批）

以下规格来自台账分诊结论与 54 候选实核报告（均已回源码初核，写页时仍须全文复核）：

### 批 U15（3 页，重派）

| 页（拟名） | 规格 |
|---|---|
| `ruinsnightmare_horn_attack.md` | 1 注册（ruinsnightmare_horn_attack.lua:206）；穿地犄角投射 FX，本体无玩法组件、伤害经 owner combat 结算；生成 SGruinsnightmare.lua:117-118 双发；返回链 fx-creature-index |
| `shadowhand_arm.md` | 2 注册（shadowhand.lua:347/349）；NOCLICK+FX、highlightchild、stretcher 拉长表现；生成 shadowhand.lua:170 动态 armprefab＋charlie_hand.lua:119；本体 shadowhand/shadowhand_shrouded 玩法辨析；`shadowhand_fx` 是 fx.lua 注册、已由 waterplant_destroy 页承载（互链不重复）；返回链 fx-creature-index |
| `shrouden_eye_fx.md` | 1 注册（shrouden.lua:1621）；Boss 死后遗留之眼，AnimState+persists=false+inspectable("PUDDLE") 可检视；生成 SGshrouden.lua:240；与批 S 的 shrouden_optic_blast_fx 页互链辨析；返回链 fx-creature-index |

### 批 U16（3 页）

| 页（拟名） | 规格 |
|---|---|
| `hotcold_fx.md` | 1 注册（smallghost.lua:573）；冷热小游戏指示 FX；生成 smallghost.lua:51；本体 smallghost NPC 辨析；返回链 fx-creature-index |
| `terrarium_fx.md` | 1 注册（terrarium.lua:647）；召唤光柱 FX（DECOR、persists=false）；生成 terrarium.lua:217；与批 U8 terrariumchest_fx 互链（同名前缀不同注册）；返回链 fx-world-index |
| `wagboss_missile_target_fx.md` | 5 注册（组 49）：wagboss_missile_target_fx（纯 FX，注册 wagboss_missile.lua:900、生成 :392）＋4 个带 combat 真实结算的混合弹体——wagdrone_laserwire_fx（:356，生成 wagdrone_rolling.lua:83）、wagpunk_cagewall_fx（:357，**快照内无生成点诚实缺口**）、wagdrone_projectile_fx（:308，生成 SGwagdrone_flying.lua:680）、wx78_drone_zap_projectile_fx（:309，生成 SGwx78_drone_zap.lua:518）；伤害归属逐个辨析；返回链 fx-creature-index |

### 批 U17（3 页，含本次找回的组 11）

| 页（拟名） | 规格 |
|---|---|
| `wx78_taser_projectile_fx.md` | 1 注册（wx78_taser_projectile.lua:312）；生成 components/wx78_taserbuildup.lua:121＋SetFXOwner，组件由 wx78_moduledefs.lua:691-692 挂载；联网主实体 shock_fx_low 常驻＋showbase/baseradius netvar、爆炸态脱父挂定爆点＋crackle 环、FX 自带 combat（TASER_BUILDUP_DAMAGE=50 电击，伤害经 owner combat:GetAttacked 归属宿主玩家，命中窗 18 帧）；po 无 NAMES 键、死亡通告 SetPrefabNameOverride("wx78")；返回链 fx-index |
| `yotr_fightring_torch.md` | 4 注册（yotr_fightring.lua:1105-1108）＋placer(:1109)；生成点全在快照：kit deploy L730、torch L528、bell L556、recipes.lua:1120；torch 为非持久 FX 壳（事件驱动 turnon→torchfire_yotrpillowfight Follower "wick"＋AttachLightTo、turnoff→Remove）；官方名：枕头大战坑/钟/套装/兔人火炬（po 353064-353079）；消费方 cozy_bunnyman.lua:76-87；返回链 fx-world-index |
| `spellmasterybuff.md` | 组 11，3 注册：spellmasterybuff／spellmasteryorb／spellmasteryorbs（spellmasterybuff.lua:101-103）。buff_postinit／orbs_postinit 调用缺失的 lavaarena 事件包；按可见工厂与实际核验范围写清缺口，不宣称事件包行为已验证。旧 U4 标记无对应页与验收记录；返回链待建页时按实际主题选择。 |

### 收尾（全部建页完成后）

1. 全库 `validate_page.py --check-indexes`（数页数、0 failed）；全库 `validate_source_links.py --strict`（存量警告如实记录，新增页须 0 警）；全库 `validate_source_hashes.py check` 0 漂移；三仓 `git diff --check`。仅实际核验过的页按需局部 record，不做全量 record 掩盖漂移。
2. 台账/PLAN/log 数字对齐（页数、注册数、组数口径），51 页组全部有 accepted 状态和实际验收批次；仅写计划编号不算完成。
3. **最终报告**落盘 docs/reports/w05/（新增/增修页清单、覆盖注册数、各校验命令结果、未覆盖注册名与原因——blocked 26 文件与 excluded 66 文件引台账、完成条件核对）。
4. goal 标 complete。

## 5 常用命令（PowerShell；页面参数使用绝对路径）

```powershell
$taskK = 'D:/Code/dst-wiki-develop/dst-wiki'
$taskS = 'D:/Code/dst-wiki-develop/dst-wiki-skill'
$taskT = 'D:/Code/dst-wiki-develop/dst-toolbox'
$taskPython = "$taskK/.venv/Scripts/python.exe"
$taskPage = "$taskK/wiki/reference/prefabs/<页>.md" # 替换页名后执行
& $taskPython "$taskT/scripts/tools/verify/validate_page.py" --wiki-root "$taskK/wiki" --format "$taskS/references/maintain/write-format.md" --source-root "$taskK/raw/current/scripts" $taskPage
# 共享索引登记完成后，上一命令追加 --check-indexes 验导航
& $taskPython "$taskT/scripts/tools/verify/validate_source_links.py" --wiki-root "$taskK/wiki" --strict $taskPage
# 实际完成核验后，只 record 本批页面；check 不加页面参数才是全库检查
& $taskPython "$taskT/scripts/tools/verify/validate_source_hashes.py" record --knowledge-root $taskK --write $taskPage
& $taskPython "$taskT/scripts/tools/verify/validate_source_hashes.py" check --knowledge-root $taskK --fresh-hours 0 $taskPage
git -C $taskK -c core.safecrlf=false diff --check
# 以下脚本只统计旧文件分诊，不读取 U 批页组状态，不能单独判定当前剩余
& $taskPython -B -I "$taskK/output/tmp/fx_closure_audit.py"
& $taskPython -B -I "$taskK/output/tmp/fx_remaining_classify.py"
```

## 6 续跑注意事项（区分工作区约定与本批线索）

- **编辑与验收**：本工作区文本修改统一使用 apply_patch。索引按实际目标小节和表格上下文补丁编辑，不靠返回句或“最后一个 active 行”猜位置；登记后验导航。草稿检查与最终导航验收的区别见 §3。
- **证据与归属**：分诊、预核与抽取 JSON 都是定位线索；写页按现有 Skill 核验，已有有效证据可复用。发现清单外注册先查现有页面归属；不能用扫描命中、台账状态或来源哈希替代技术结论。
- **并发与分工**：本批曾遇并发限制，属于当时宿主状态，不形成通用“上限 2”。按当前可用额度和实际授权派发，避免重叠写共享文件；本文不构成新会话启动子代理的授权。
- **本批数据限制**：fx_perfile_triage.json 曾漏包装工厂、循环与 MakePlacer；旧对账脚本只消费文件分诊状态。本次找回 spellmasterybuff 待建组，续跑须同时看页组状态和落盘页面。
- **本批定位线索**：fx.lua 除通用表外还有散件注册；如遇 shadow_shield、singingshell 名族先查已有归属。arhivepoweron、ballon_held_child、iconffset 等是本快照的具体拼写，按实际源码引用，不推广为命名规则。
- **名称依据**：沿用 source-types 的实际显示键规则；NAMES、SKIN_NAMES 和 DESCRIBE 用途不同。宿主名称、效果描述、函数拼写分别核验，不能把后两者当官方名称键。

## 7 可直接粘贴的新会话提示词

```text
任务：接手并完成 DST Wiki「外围 FX 扩库」最后收尾。跨轮持续执行，不要每批结束就交还。

【第一步：读与核】
1. K/docs/reports/w05/fx-handover-20261105.md（主地图，取代 20261104 旧版）
2. K/docs/reports/w05/fx-coverage-ledger-20261101.md 的「阶段交接」「外围 needs-page 建页清单（含组 41-51）」「外围批 U 验收检查点」「未分诊 54 候选之实核（三节）」
3. K/PLAN.md 的 FX 区（看最后几条 [x] U1-U14）与 K/wiki/log.md 顶部 6 条
4. S/references/maintain/{write-format,source-types}.md、S/assets/prompts/{new-pages,source-review}.md
5. 按 §5 命令复核候选分诊基数，并按页组状态、实际页面／别名和验收记录核对当前剩余；旧脚本 remaining 不代表待建。基线检查用 check，不做全量 record。

【环境】
K=D:/Code/dst-wiki-develop/dst-wiki，S=D:/Code/dst-wiki-develop/dst-wiki-skill，T=D:/Code/dst-wiki-develop/dst-toolbox，Python=K/.venv/Scripts/python.exe，快照=K/raw/current/scripts（接手实读 BUILD；本次为 26335394；raw 只读）。三仓大量未提交改动为有意状态：不回退、不提交、不推送。

【本项目分工（仅在当前会话有对应授权时沿用）】
按实际授权与当前宿主额度派发写作；共享台账、record 与索引集中收口，每页独立复审，不抽样。本文描述分工，不构成新会话启动子代理的授权。

【剩余工作（9 组、19 个计划注册，另有 1 个 placer；计划 3 批，每批 3 页）】
- U15（3 页）：ruinsnightmare_horn_attack（1 注册，生成 SGruinsnightmare.lua:117-118）、shadowhand_arm（2 注册，生成 shadowhand.lua:170 动态＋charlie_hand.lua:119；shadowhand_fx 已由 waterplant_destroy 页承载勿重复）、shrouden_eye_fx（1 注册，生成 SGshrouden.lua:240）——三页均返回链 fx-creature-index
- U16（3 页）：hotcold_fx（1，生成 smallghost.lua:51，返回链 fx-creature-index）、terrarium_fx（1，生成 terrarium.lua:217，返回链 fx-world-index）、wagboss_missile_target_fx（5 注册＝纯 FX 1＋混合弹体 4，规格见交接文档 §4，返回链 fx-creature-index）
- U17（3 页）：wx78_taser_projectile_fx（1 混合，生成 wx78_taserbuildup.lua:121，返回链 fx-index）、yotr_fightring_torch（4 注册＋placer，范围见 §4，返回链 fx-world-index）、spellmasterybuff（组 11 找回，3 注册，事件包缺口见 §4）。批次编号只是计划，开始前核对占用。

【每批闭环】写作者按当前 new-pages 模板交付，草稿自检不验未登记导航 → 主会话逐锚点复审（报告声明逐条实证、存疑逐条核查）→ 修复 → record＋索引登记（登记后复验 --check-indexes）→ 按本项目约定留痕 → 下一批。台账技术分诊不能替代源码证据，勘误当批复核回写。

【收尾】全库 validate_page --check-indexes、全库 strict links（存量警告如实记录）、全库来源 check 0 漂移、三仓 git diff --check；台账/PLAN/log 数字对齐（51 组全部 accepted 且有实际验收批次）；最终报告落盘 docs/reports/w05/（覆盖数、命令结果、blocked 26/excluded 66 引台账、完成条件核对）。不提交不推送。

【完成条件】380 候选全部有结论且 51 页组全部建完验收；全部新页独立复审修完 P1/P2/P3 且 record 0 漂移；全库校验通过；台账/PLAN/log/最终报告数字一致。

【禁止】不改 raw、不声称已运行游戏、不把结构校验当技术核验、不用"暂未找到消费者"当偷懒结论、不新建 maintain-fx.md 类重复规范、不改 validate_page.py 契约。

开工前先读【第一步】的五项并把实核数字对不上之处报出来，然后直接开 U15。
```
