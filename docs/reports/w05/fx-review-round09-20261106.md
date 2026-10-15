# FX 独立检视第 9 轮：原生成批 12＋外围 C

日期：2026-11-06。实读 Steam buildid **26335394**；知识仓 HEAD `97afb4e9e6ca05390558769926f1f9c8598d0f70`。本轮检查 F42～F46 五篇通用表页及外围 C 的三篇新页，共 **8 页／45 个注册名**。外围 C 历史统计中的 `stafflight`、`staffcoldlight`、`staffcoldlightfx` 属既有宿主页，不重复计入新页范围。

**当前结论：第 9 轮已收口。初检 P1×0、P2×1、P3×1；R01／R02 均已修正并独立复验通过，未发现新 P1/P2。** 初检依据保留在下文，修后结果见文末；原批次 accepted 和既有复审记录不代替本次独立判断。

本轮只新增本报告并追加日志，不修改知识页、索引、基线、PLAN、台账、Skill 或工具；保留并行装备页改动，不提交、不推送。未重新验收范围外页面，也不替第 5 轮 R10 作修后独立结论。

## 范围与逐页结论

页面合计 **457 个物理行／56,534 字节**，均在当前 standard 档位内。

| 页面 | 注册数 | 行／字节 | 本次重点与结论 |
|---|---:|---:|---|
| [hermitcrab_fx_small](../../../wiki/reference/prefabs/hermitcrab_fx_small.md) | 9 | 59／8,115 | 迁移大小映射、休眠／limbo 分支、号角两端与设施废弃、传送器动态字段、首领地面判断、中庭本地玩家父挂与屏幕事件；岩树倒塌的负向结论另作完整检索。未发现 P1/P2。 |
| [fumarole_ember](../../../wiki/reference/prefabs/fumarole_ember.md) | 4 | 50／4,058 | 温度档及半秒概率、余烬队列与位置种子、烹熟一件及父挂新熟食、仪式提前 71 帧与三组圆环、玩法与表现分工。未发现 P1/P2。 |
| [cavehole_flick](../../../wiki/reference/prefabs/cavehole_flick.md) | 8 | 57／6,820 | 重生／换肤／箱怪重置、弹飞双段计时与死亡／幽灵／变身门禁、鼹鼠六个帧点、表情动态名与眼泪延迟、迁移覆盖名消费、nameoverride 与查看者描述。未发现 P1/P2。 |
| [monkey_cursed_pre_fx](../../../wiki/reference/prefabs/monkey_cursed_pre_fx.md) | 6 | 55／5,201 | 诅咒计数与父挂表现、变形重试门禁、骑乘下骑、角色交换及四个前后状态的落点生成。未发现 P1/P2。 |
| [minotaur_blood1](../../../wiki/reference/prefabs/minotaur_blood1.md) | 4 | 48／4,699 | 力量变档与静默门禁、受击阈值与 chess 排除、周期随机拼名、未使用的 offset、大血滴与触手逻辑分界；正文缩写见 R02。未发现 P1/P2。 |
| [emberlight](../../../wiki/reference/prefabs/emberlight.md) | 1 | 55／7,790 | 技能菜单及标签、两余烬花费、共享热星工厂、玩法组件、光照同步、到期直接 kill_light 与主动熄灭分界、暗影手调用；官方名与法术名分开。未发现 P1/P2。 |
| [lightning](../../../wiki/reference/prefabs/lightning.md) | 5 | 68／11,553 | 四个工厂的端／雷声／屏闪／回收、阻挡与月暴选名覆盖、天气结算门禁、月玻璃副作用、自然近远雷与事件调用者。机制描述未发现 P1；来源声明缺项见 R01（P2）。 |
| [staffcastfx](../../../wiki/reference/prefabs/staffcastfx.md) | 8 | 65／8,298 | 八名动画资源与骑乘方向、仅主模拟 SetUp、各施法帧点与中断清理、回溯解父和放弃引用、客户端不生成本族、同名 bank 辨析。未发现 P1/P2。 |

## R01 · P2：闪电页正文引用 forest.lua，但 sources 未声明

- 位置：[lightning.md 第 10 行](../../../wiki/reference/prefabs/lightning.md#L10)的 sources 缺少 `prefabs/forest.lua`；[第 53 行](../../../wiki/reference/prefabs/lightning.md#L53)已引用该文件，断言月暴闪电由 forest 预载。
- 依据：[forest.lua 第 413 行](../../../raw/current/scripts/prefabs/forest.lua#L413)确有 `"moonstorm_lightning"` 依赖，正文结论正确，问题在声明和哈希依赖遗漏。现有哈希 check 的 94 组均 unchanged，也不能覆盖这项未声明来源。
- 建议：保留这条有用的依赖辨析，在 sources 补入 `prefabs/forest.lua`，按现有规则只更新该页来源基线，再跑该页格式、严格链接与哈希 check。无需批量重录其他页。
- 处置：待修；本次不改页、不 record。

## R02 · P3：血滴第 2、3 个注册名在正文缩写

- 位置：[minotaur_blood1.md 第 28 行](../../../wiki/reference/prefabs/minotaur_blood1.md#L28)写成 `` `minotaur_blood1`／`2`／`3` ``；正文没有完整的 `minotaur_blood2`、`minotaur_blood3`。
- 依据：[三个表项](../../../raw/current/scripts/fx.lua#L2674-L2710)分别注册三名；[实际生成](../../../raw/current/scripts/prefabs/minotaur.lua#L329-L338)用 `"minotaur_blood"..math.random(1,3)`。aliases 已写全，动画、音效和随机选择描述正确，因此不是整页搜索失效或机制事实错误。
- 建议：保持合并行，在同一单元格写全三个注册名，便于正文 grep 和独立读取“注册与构造”章节；不必拆行、拆页或改 sources。
- 处置：非阻塞建议；本次未修页。

## 检索与证据复核

- 对 current/scripts 全树 **4,095 个 Lua 文件**按 45 个完整注册名检索，共 **329 行候选**，完整查看，区分注册、资源、对白、依赖声明、实际生成和动态消费；命中数不是生成点数量。
- 追加前缀与共享入口检索：`tree_rock_` 68 行、`hermitcrab_fx_` 24 行、`minotaur_blood` 10 行；补看相关 SpawnPrefab／SpawnChild／GetPooledFx／ReplacePrefab 及施法、传送的选名字段。岩树两种特效的字面名及前缀结果未见新的 fall 生成候选：保留页面“在本快照全 scripts 检索未找到”的范围说明，不升级为不存在。
- `ms_sendlightningstrike` 共 16 处引用，已区分监听、自然天气和事件调用者；读 weather 选名顺序及 componentutil 结算门禁，确认 `thunder` 可经过结算函数调用但不会执行其内部仅针对 `lightning` 的触电／点火分支。月暴闪电自身生成玻璃，不等同普通雷击伤害。
- 简中 `.po` 精确查 45 个注册名的 `STRINGS.NAMES` 键，只有 [EMBERLIGHT](../../../raw/current/scripts/languages/chinese_s.po#L342479-L342482) 与 [LIGHTNING](../../../raw/current/scripts/languages/chinese_s.po#L344729-L344732) 命中；另核对 [HERMITCRAB](../../../raw/current/scripts/languages/chinese_s.po#L343780-L343782)、[MOLE_UNDERGROUND](../../../raw/current/scripts/languages/chinese_s.po#L345530-L345532) 和 [PYROMANCY.FIRE_BALL](../../../raw/current/scripts/languages/chinese_s.po#L354929-L354932)。名称覆盖键与宿主名称不从注册名机械推断。
- 45 个注册名在当前 Prefab aliases 中均只有一个主要归属；除 R02 两个缩写外，均在正文完整出现。正文直接源码引用与 sources 对照只发现 R01 的一个缺项；不把 sources 中已核验的辅助依赖文件一律判作冗余或自动删除。

## 实际校验与边界

| 检查 | 本轮实际结果 |
|---|---|
| `validate_page --check-indexes` | 8／8 页 passed；全库导航通过，1,143 篇知识页／34 个索引／0 问题，退出 0。导航通过不代表验收并行装备页语义。 |
| 叶级登记定向核对 | 8 页各恰有一个索引表格登记：table-world 3 页、table-character 2 页、fx-index 2 页、fx-world 1 页；页尾均能返回登记索引。 |
| `validate_source_links --strict` | 217 处行引用／210 个唯一目标／75 个源码文件；0 错误、0 警告、skipped_unsupported 0、未截断，退出 0。 |
| `validate_source_hashes check --fresh-hours 0` | 8 页／94 组来源比较均 unchanged；changed、sources_changed、unrecorded、invalid 均为 0，退出 0。未 record；R01 未声明来源不在此页的基线覆盖内。 |
| 标题原文章节读取 | 8 页分别以 `#关联机制导航` 调用 query_wiki read，均退出 0。注册名归属和正文检查直接读 Markdown，未重建查询缓存或用 stale search 作验收。 |
| 输入稳定性 | 检视期间比较所选 8 页及其 78 个不同声明来源，共 86 个文件，前后哈希均未变；BUILD 仍为 26335394。 |
| 差异检查 | 写报告前工作区 `git -c core.safecrlf=false diff --check` 通过；交付前再检查报告链接与日志追加。 |

未运行 DST，未解包动画素材；本批没有 Lua 示例，不声称完成示例语法或游戏表现验证。格式、锚点和哈希通过不能替代机制核验，也不能兜住遗漏的 sources。

## 进度与下一轮

固定 158 篇新增 Prefab 页队列累计已独立检视 **68／158 页，剩 90 页**。本轮初检问题已修并独立复验收口，见文末；复验不重复计入初检进度，历史轮次的待复验项另计。

下一轮可按台账进入外围 D：`reticuleaoe`、`rain`、`wave_med`、`staff_castinglight` 四页／64 注册。四页合计约 71 KiB，含 complex 准星族和两篇密集机制页，工作量高于普通四篇小页，宜按复杂度定范围；随后为外围 E 的三页／57 注册，不为凑页数把两组强行并批。

## 2026-11-06 修后独立复验与收口

- R01 已覆盖：[闪电页 sources](../../../wiki/reference/prefabs/lightning.md#L10)补入 `prefabs/forest.lua`；正文第 53 行与源码第 413 行的月暴预载依赖一致。该页现有 21 个声明来源，正文直接引用无漏声明，基线 check 未报 sources_changed 或未记录来源；本次没有再次 record。
- R02 已覆盖：[血滴注册表](../../../wiki/reference/prefabs/minotaur_blood1.md#L28)在同一行写全三个注册名，与 fx.lua 三个表项一致；aliases、sources、动画及调用逻辑未变。以标题原文 `#注册与构造` 读取，返回章节包含全部三个完整名；闪电页 `#关联机制导航` 读取同样退出 0，并保留 forest.lua 行锚点。
- 两页实际 diff 仅上述来源增项和注册名展开，未发现新 P1/P2。闪电页 68 行／11,575 字节，血滴页 48 行／4,727 字节，均在 standard 档位内。其余六页在与初检相同的 HEAD 下没有工作区差异，沿用初检，不冒充重新审阅八页全文。
- 定向 `validate_page --check-indexes`：2／2 页 passed；全库导航 1,143 知识页／34 索引／0 问题，退出 0。`validate_source_links --strict`：62 处行引用／58 个唯一目标／26 个源码文件，0 错误、0 警告、skipped_unsupported 0，退出 0。
- `validate_source_hashes check --fresh-hours 0`：两页 26 组来源均 unchanged；changed、sources_changed、unrecorded、invalid 均为 0，退出 0。BUILD 仍为 26335394；未运行游戏，不替范围外装备页或第 5 轮 R10 作独立语义验收。
- 本次只更新本报告并追加日志，保留他方页面、基线和台账改动；不修知识页、不改索引／PLAN／台账／Skill／工具，不提交推送。

第 9 轮可以收口。累计仍为 68／158 页、余 90 页；下一轮范围沿用上节外围 D 四篇机制族页。
