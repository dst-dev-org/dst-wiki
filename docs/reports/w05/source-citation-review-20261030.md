# Wiki 源码引用与章节读取检视修复（2026-10-30）

证据快照：Steam buildid 26230496。只修工具、名称证据规则和明确的编辑问题；raw 只读，保留其他 Agent 已有增量，不提交或推送。

## 范围与数字口径

- 当前 .po 引用为 **109 次出现／90 个物理行／77 篇页面**，引用次数按出现次数计；77 页 sources 仍保留 chinese_s.po。
- 新专属工厂批次 75 页中有 56 页声明 .po；将此前 5 篇冰箱 FX 页一起计入时，为 61／80。两种分母范围不同，不改写历史批次记录。
- 不把文件级 .po 变化当作 Lua 机制变化；实际名称证据的哈希依赖继续保留。没有 record，也没有为消漂移删除来源。

## 改前与改后

下列工具路径相对明确工具根 D:/Code/dst-wiki-develop/dst-toolbox；规则路径相对 D:/Code/dst-wiki-develop/dst-wiki-skill。

| 项目 | 改前 | 改后／位置 |
|---|---|---|
| 文本来源链接 | source_links 仅接受 .lua，.po 无统计地跳过 | scripts/tools/verify/validate_source_links.py：local_links 识别本地行内目标，source_links 接受 .lua／.po，scan 检查两类路径／行区间，其他后缀计入 skipped_unsupported |
| 编码信息 | UTF-8 失败回退 latin-1，结果不可见 | source_encodings 按成功读取文件列 encoding／fallback／references；info 不使 strict 失败。明确二进制头／大量 NUL 每文件一次 source-binary warning |
| 行号口径 | 链接工具按 CR／LF，页面／章节工具用 splitlines，U+2028 可变成幻影章节 | scripts/tools/wiki_text.py 共用 physical_lines；validate_page 的正文／标题／链接／大小、query_wiki 的单章与多章、wiki_data 的缓存切章均使用它；保留原始换行终止符 |
| anchor 归一 | 反复把标题包装成伪章节解析，空归一化可命中纯标点标题 | wiki_text.normalize_anchor 由 validate_page 导出；query_wiki.read_chapter 复用，空结果报 missing，重名仍报 ambiguous，明确后缀仍精确命中 |
| Lua 迁移范围 | 迁移预览复用 source_links | scripts/tools/maintenance/plan_source_link_updates.py 显式保留 .lua 筛选，避免本次 .po 校验修复扩大迁移工具范围 |
| 名称证据规则 | .po 是否纳入 sources 未明说 | references/maintain/source-types.md 名称条款仅补一句：实际用于核验页面名称的 .po 计入 sources，并保留正文锚点；verify-wiki 检查选择同步支持 .po |
| 回链位置 | boattrail／floater 职责句尾括号突兀 | 两页回链并入“优先使用”，updated 同步；保留其他 12 处回链，不批量重排 miasma 关系 |

工具箱 references/wiki-validation.md 同步实际后缀、编码／跳过统计与分行口径，docs/design.md 更新处理边界；工具箱 PLAN 补齐 query_wiki 的归一化修复和本轮验收记录。AST 提取仍不处理 .po，查询缓存格式不变。

## .po 校验发现的真实修订

新增检查发现 7 处区间末尾多含一行空白。逐条对照 msgctxt／msgid／msgstr 后，只收紧终点，不删除译名证据，不改变 sources：

| 页面（均在 wiki/reference/prefabs） | 改前 | 改后 |
|---|---|---|
| brilliance_projectile_fx.md | L349340-L349343 | L349340-L349342 |
| brilliance_projectile_fx.md | L344245-L344248 | L344245-L344247 |
| eyeofterror_arrive_fx.md | L350505-L350508 | L350505-L350507 |
| eyeofterror_arrive_fx.md | L350510-L350513 | L350510-L350512 |
| gelblob_attach_fx.md | L342545-L342548 | L342545-L342547 |
| shadowthrall_projectile_fx.md | L348605-L348608 | L348605-L348607 |
| weremoose_smash_fx.md | L351765-L351768 | L351765-L351767 |

## 实际验收

| 检查 | 结果 |
|---|---|
| unittest discover | 245 项被发现：244 通过、1 项环境受限跳过，0 failure／error；不是“245 全通过” |
| 新增回归 | 12 项：6 项链接／编码，4 类 anchor，1 项三工具行号＋缓存切章一致，1 项 Lua 迁移范围 |
| 页面／导航 | 全库 948 页 passed：919 知识页、29 索引，0 issue |
| 源码链接 strict | 20,525 引用，0 error／0 warning；其中 109 次 .po 引用；43 个已识别的其他后缀未读取并明确统计 |
| 编码信息 | 1,724 个成功读取的来源文件：1,723 UTF-8，easing.lua 1 个 latin-1 回退、被引用 1 次；回退作为 info |
| 来源基线 check | 948 页、5,294 组比较全部 unchanged；0 invalid_pages、0 sources_changed、0 affected_pages |
| 真实章节读取 | cane_chess_fx、willow_shadow_flame、deer_ice_circle、floater 四章均返回 0，无截断，读取当前 Markdown |
| Skill | quick_validate 通过；无新增 helper、模板或依赖 |
| diff | 三仓 diff --check 通过；不重建正式查询缓存，不写来源基线，不运行游戏／解包动画 |

完整回归启用 DST_SETUP_SCRIPT 指向现有 Skill 源码准备脚本，DST_TEST_CODEX_BINARY 指向已有原生 codex.exe，因此源码准备与真实 apply_patch 集成均实际执行。唯一未执行项为 test_script_and_game_on_different_drives：默认临时目录位于 D 盘；显式换至现有 C 盘目录后，Python 的 mkdir 返回 WinError 5，tempfile 创建重试持续占用 CPU，已终止该次尝试。最终使用该测试已有的未配置环境跳过分支，不改测试断言或产品代码来绕过权限。

中断测试留下的三个临时夹具均先核对路径、文件集合与脚本哈希，再清理：文本副本使用 apply_patch；ZIP 是生成的二进制，无法使用文本补丁，采用原生文件操作删除，空目录移除前确认都在指定工具箱测试目录内。未删除真实源码或其他 Agent 文件。

机械检查不证明全库技术结论、译名语义或游戏表现已重新审阅。
