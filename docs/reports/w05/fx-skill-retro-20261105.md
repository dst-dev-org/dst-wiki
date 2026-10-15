# 外围 FX 扩库 Skill 使用复盘与改进建议（2026-11-05）

> 写作背景：接手会话在一天内完成批 T 复审、54 个未分诊候选实核、14 个建页批次（U1～U14，41 新页＋1 增修页、110 注册）与对账定稿。本文是该过程中对 `dst-wiki-skill`（S）规范资产、`dst-toolbox`（T）校验工具与协作流程的使用复盘，供 Skill/工具维护者参考。所有结论均有本会话实例支撑，不做泛泛而谈。
> 数据：14 个建页批次中，主对话逐锚点复审发现 P1×0、P2×3、P3×5（其余批次零发现）；写作代理主动上报台账勘误 15 条；结构校验（validate_page／strict links）在写作者自检后全部 0 错 0 警入库；全库 record 7360 组比较 0 漂移。

## 1 运转良好的部分（保持不动）

1. **台账作为唯一续跑状态源**。`fx-coverage-ledger-20261101.md` 的「阶段交接」节＋批次检查点结构，让新会话在上下文被摘要后仍能精确定位进度。本会话所有关键决策（批 T 冲突裁决、剩余清单重算）都以台账＋可重跑脚本（`fx_closure_audit.py`、`fx_remaining_classify.py`）为准，而不是交接文档的叙述。**结论：台账模式成立，应作为跨会话项目的标配；且台账数字必须由脚本生成，不得手估。**
2. **写作代理的预热包**。每批派发时要求代理先读 write-format、source-types Prefab 节、new-pages 提示词＋2 篇近期验收范例页＋目标叶级索引，产出结构一致性极高（frontmatter 12 字段、验证记录七段式、返回链、互链）。14 批无一例外合格。
3. **写作者自检三件套**（validate_page --check-indexes、validate_source_links --strict 逐页、grep 双覆盖）把锚点空行/结构行、slug 规则、表格管道符等问题在代理侧消化，主对话复审只需对事实断言。初跑 warning 平均每页 2～5 处，全部由代理自行修复到 0/0 才交付。
4. **复审分工（写作与校验用子代理、复审用主对话）**。主对话逐锚点核验压住了两类风险：写作代理对台账既有结论的盲从（15 条勘误全部由主对话实证后回写台账）、以及写作代理自建声明的真实性。零 P1 说明现有 write-format＋范例页已足够约束事实层。
5. **诚实缺口惯例**。「快照内无生成点（写明检索模式与范围）、不判死代码」的格式（goosplash.md 先例）被所有代理正确复用，未出现以"没找到"当"不存在"的偷懒结论。
6. **record/check 基线**。只改正文不触发漂移、增修页重 record 即刷新——语义清晰，本会话 41 页＋多次增修全程 0 漂移。

## 2 痛点与改进建议

### 2.1 规范文档（S/references/maintain/*）

| # | 现象 | 实例 | 建议 |
|---|---|---|---|
| 1 | **分档预算变更后，旧页验证记录引用旧数值**。本会话接手时 standard 已从 200 行/16 KiB 提到 240/24，但批 T 页与 13 个历史页验证记录仍写旧预算 | fx_book_mount「standard 档 200 行／16 KiB 内」 | write-format 已加"旧预算保留 standard"条款，建议再补一句："验证记录中的预算数值按写入当时规则理解，规则升级不回填历史页"——把这一处理写成明文，避免每个接手会话重新纠结 |
| 2 | **validate_page 单页模式的 navigation 报错语义**。未登记索引的页报 `registration: expected one leaf table registration, got 0`，14 个写作代理初次都当成错误来问 | 每批交付报告都有此疑问 | 单页模式输出里加一句人话提示（"未在本仓叶级索引登记：若由主代理统一登记可忽略"），或提供 `--expect-unregistered` 开关 |
| 3 | **ledger 状态词不统一导致脚本分诊漏计**。批 6 用「excluded＋FX 子注册待合并」而非「needs-page」，初版分类脚本漏掉 10 文件 | 本会话对账时发现，补了 `待合并` 特判 | 在台账口径节固定状态枚举（accepted／covered-existing／needs-page／excluded／blocked＋可选后缀），后续分诊一律用枚举词，修饰语放括号 |
| 4 | **fx.lua 的双重角色无明文**。它既是 504 名通用表数据表，又承载零散注册（singingshell_creature_*、shadow_shield1-6、wortox_soul_spawn_fx 等在本会话被反复"新发现" | U2/U13/U14 三批都踩到 | source-types 或 write-format 补一句：fx.lua 除通用表条目外还有散件注册，反查命中 fx.lua 时先查该名是否已有归属页 |
| 5 | **「一个注册名一个主要页」与既有页归属的核对动作缺提示**。多批次遇到清单外注册时先问"要不要并进来"，实际规则是先 grep 全库 aliases 找归属页 | U8 launch/impact fx、U14 shadow_shield1-6 | new-pages 提示词里把"发现清单外注册→先全域 grep aliases→有归属则互链不重复承载→无归属才并入"写成固定步骤 |

### 2.2 提示词资产（S/assets/prompts/*）

| # | 现象 | 建议 |
|---|---|---|
| 1 | **写作派发无官方模板**。source-review.md 有复审模板，但写作派发的任务书（环境路径、预热包清单、写作规则、自检三件套、交付报告格式）每批由主代理手写，14 批约 60% 内容逐字重复 | 在 assets/prompts/ 增加 `page-writing-dispatch.md`（对齐 source-review.md 的体例：派发前确认＋可填写任务文本＋收口），主代理只填页组规格段。本次会话的任务书文本可直接作为底稿 |
| 2 | **交付报告格式靠约定**。每批报告的字段（尺寸/注册/生成点/关键声明/po/自检/缺口/存疑）由主代理逐批要求，前两批有遗漏后补齐 | 把这 8 字段写进上述模板的"最终报告"节，写作者按固定结构回传，主代理复审可直接对表 |
| 3 | **"台账结论须实证"没有明文**。15 条勘误里大多数是代理复核台账旧结论时发现与源码不符（10Hz→6/s、feather_loop 一帧→一轮、落地喷雾→阻挡标记、经代理→直接 attacker、同 bank→不同 bank 等） | 在 new-pages 或 source-types 加一句：台账/既有分诊结论是待验假设，写页时必须回源码复核；与台账不符时在页内按实证写，并在交付报告单列"台账勘误建议" |
| 4 | 复审模板 source-review.md 已好用，但缺"对写作代理报告的存疑清单逐条实证"这一步的提醒 | 复审模板"本批重点"加一条：代理自报存疑清单是必查项，逐条回源码 |

### 2.3 工具箱（T/scripts/tools/verify/*）

| # | 现象 | 建议 |
|---|---|---|
| 1 | **叶级索引登记无工具**。本会话 41 页登记全靠主代理手写脚本，出过两类事故：bash 双引号吞反引号（一次，已修）；文本标记 replace 命中 fx-creature-index 引言段（引言与末尾都含"返回 [[reference/prefabs/index|…]]"，U5/U7 两行插错位，U11 发现后修复） | 做 `add_index_row.py --index fx-world-index --page X --summary … --aliases …`：定位该索引的表格末行（最后一个 `/ active |` 行）后插入、校验行落在表格内（上一行也是表格行）、幂等 |
| 2 | **锚点质量可以前移**。每页初跑 strict 平均 2～5 个锚点警告（空行/结构行起止），全靠代理按报错修 | validate_source_links 的 warning message 里直接给出建议修正（"起点落在空行，建议 L57→L58"），或提供 `--suggest-fixes`；另可在 write-format 锚点节加两个正反例 |
| 3 | **verify_page_coverage 的别名覆盖判定**。正文提及不算覆盖（D 检查只认 aliases）——设计合理，但"正文已提及、缺 aliases"的 15 个候选需要人工甄别 | 可加 `--listed-in-body` 输出层：把"正文有字面名但无 aliases"单列为一类 info，缩小人工甄别面 |
| 4 | 基线 check 全库模式很快（7360 组秒级），但**没有汇总页数/类型分布的轻量命令**，每批交接要靠整跑 validate_page（数分钟） | validate_source_hashes check 全库输出里加 pages 分类计数，或 validate_page 加 `--summary-only` |

### 2.4 流程与协作惯例（本次会话自创且验证有效的，建议固化进 Skill）

1. **每批五步闭环**：写作代理（含自检 0/0）→ 主对话逐锚点复审（对代理"关键验证声明"逐条实证＋存疑清单逐条回源码）→ P1/P2/P3 修复（派写作代理）→ record＋索引登记（主代理）→ 台账/PLAN/log 留痕。本会话 14 批零返工。
2. **预核**：派发写作代理前，主代理对页组做 3～5 分钟的注册行/生成点轻量 grep，形成"预核发现"写进任务书（如 dust_low 双生成点、pb_fx 双挂点）。复审时对证，能提前暴露清单错误。
3. **并发上限 2**：4 个并行子代理触发 `user concurrency limit exceeded`；1 写作＋1 分诊的 2 并发全程稳定。
4. **台账勘误即时回写**：代理上报的勘误当批复核后写进台账对应节（本会话累计 15 条），不做"收尾时统一改"——跨会话项目里延迟回写等于丢失。
5. **对账脚本先于结论**：接手第一件事跑 `fx_closure_audit.py`／`fx_remaining_classify.py` 重算，不信任何文档数字。本次交接文档"约 20 组合并页"实为 88→101 文件、批 T 复审状态两处矛盾，全靠脚本裁决。
6. **文档维护的 bash 卫生**：向仓库 markdown 写入含反引号/管道符的中文文本时，用 `python - <<'PYEOF'`（单引号 heredoc）而不是 `python -c "…"`——双引号内反引号会被 shell 当命令替换吞掉（本会话实际发生一次并已修复）。
7. **主代理自改文本文件时的标记选择**：`str.replace(marker, …, 1)` 的 marker 必须在全文唯一——fx-creature-index 的引言段与末尾都含同一"返回 …"句子，教训是先 `grep -c` 验证 marker 唯一性再 replace。

## 3 数字备查

- 本会话产出：41 新页＋1 增修页（oceantree_ripples_short 并入 watertree 两注册）；覆盖 110 个注册名；54 个未分诊候选全部实核（blocked 15／excluded 26／needs-page 13 文件）；380 口径定稿＝187 闭环＋101 needs-page（51 页组）＋26 blocked＋66 excluded。
- 剩余：51 页组中已完成 43 组，剩 8 组（U15 被中断：ruinsnightmare_horn_attack／shadowhand_arm／shrouden_eye_fx；U16：hotcold_fx／terrarium_fx／wagboss_missile_target_fx＋无人机弹体混合；U17：wx78_taser_projectile_fx／yotr_fightring 全族），约 16 注册。
- 质量抽样：14 批复审发现 P1×0、P2×3、P3×5；代理上报台账勘误 15 条（全部实证回写）；结构校验入库 0 错 0 警；record 7360 组 0 漂移。
