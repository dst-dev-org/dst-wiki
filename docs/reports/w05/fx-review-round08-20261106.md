# FX 独立检视第 8 轮：原生成批 10＋11

日期：2026-11-06。实读 Steam buildid **26335394**；知识仓 HEAD `97afb4e9e6ca05390558769926f1f9c8598d0f70`。按用户扩大每轮范围的安排，合并 F30、F32～F41，共 **11 页／69 个通用表注册名**；逐页审阅全文、表项和必要消费者，不以原批次 accepted 或既有复审记录代替本次判断。

**当前结论：第 8 轮已收口。初检 P1×0、P2×0、P3×1；R01 已修正并独立复验通过，未发现新 P1/P2。此前并行装备页造成的导航问题已消失，修后全库导航通过。初检情况与范围保留在下文，最新验收见文末。**

本轮只新增本报告并追加日志，不修改知识页、索引、基线、PLAN、台账、Skill 或工具；不提交、不推送。第 5 轮 R10 不在本轮范围，未代为收口。

## 范围与逐页结论

页面合计 608 个物理行、62,204 字节，均在当前 standard 档位内。表中行数沿用页面校验器口径。

| 页面 | 注册数 | 行／字节 | 本次重点与结论 |
|---|---:|---:|---|
| [sugarwood_leaf_fx](../../../wiki/reference/prefabs/sugarwood_leaf_fx.md) | 15 | 69／10,234 | 逐项字段、紫叶动态表字段、砍伐门禁、棕榈松拼名、海树落叶概率、催长与死亡烟调用；糖木事件包缺口保留。未发现 P1/P2。 |
| [spell_fire_throw](../../../wiki/reference/prefabs/spell_fire_throw.md) | 4 | 53／4,463 | 火杖失败目标条件、三种月化宠物结束路径、暗影火伤害归属、tintalpha 与施火名的声明／生成分界。未发现 P1/P2。 |
| [dreadstone_spawn_fx](../../../wiki/reference/prefabs/dreadstone_spawn_fx.md) | 4 | 53／4,689 | 裂隙加载与释放分支、寄生面具装备前提、Follower 与父挂区别、诅咒 150／162 帧及加载分支。未发现 P1/P2。 |
| [wagpunksteam_hat_up](../../../wiki/reference/prefabs/wagpunksteam_hat_up.md) | 4 | 52／4,268 | 帽甲阶段与重置条件差异、任务取消、0～0.3 秒延迟与执行时有效／存活／冰冻检查、AddChild 的实际对象。未发现 P1/P2。 |
| [wagdrone_rolling_collide_small_fx](../../../wiki/reference/prefabs/wagdrone_rolling_collide_small_fx.md) | 3 | 52／5,514 | running 事件、工作对象 spark、同类／硬甲分支、零距离公式边界；灵药攻击来源与头槽充能、debuff 门禁和放电顺序。未发现 P1/P2。 |
| [junk_break_fx](../../../wiki/reference/prefabs/junk_break_fx.md) | 11 | 65／8,875 | 退化循环与侵蚀对象、冰船共享 helper、海冰 silent、地皮落点、废料拾取分支、箱子升级与读档、雪人打蜡、pinnable 动态消费。未发现 P1/P2。 |
| [wurt_terraformer_fx_shadow](../../../wiki/reference/prefabs/wurt_terraformer_fx_shadow.md) | 8 | 53／5,298 | 瓦片筛选、改造／撤销、随机周期光点、湿度抵伤计算与父挂、鱼人 14／15 帧事件；正文两个注册名缩写见 R01。未发现 P1/P2。 |
| [wortox_soul_spawn_fx](../../../wiki/reference/prefabs/wortox_soul_spawn_fx.md) | 7 | 57／5,751 | 心脏 selfuse／他人传送分支、排箫 52 帧延迟、分身荆棘、18／22 帧事件及 expire 选名、月亮装备 resistance 回调安装。未发现 P1/P2。 |
| [voidcloth_boomerang_launch_fx](../../../wiki/reference/prefabs/voidcloth_boomerang_launch_fx.md) | 3 | 49／3,973 | 发射与命中落点、OnMiss 分界、牛铃→牛复活入口→SG 的 SpawnChild、无敌及声音归属。未发现 P1/P2。 |
| [abigail_attack_shadow_fx](../../../wiki/reference/prefabs/abigail_attack_shadow_fx.md) | 7 | 55／5,649 | 光环名单进出、murdered 技能门禁、额外生命调用缺口、两条启迪命中路径、纯辉标记的位面攻击与自触发排除。未发现 P1/P2。 |
| [fallingswish_clouds](../../../wiki/reference/prefabs/fallingswish_clouds.md) | 3 | 50／3,490 | 两种下坠状态、skip_vfx 条件、VOID 数组的 SinkEntity／旋涡消费者与地图外回退。未发现 P1/P2。 |

## R01 · P3：水盾第 2、3 个注册名在正文缩写

- 位置：[沃特页第 31 行](../../../wiki/reference/prefabs/wurt_terraformer_fx_shadow.md#L31)。表格写成 `` `wurt_water_splash_1`／`2`／`3` ``，正文没有完整的 `wurt_water_splash_2` 和 `wurt_water_splash_3`。
- 依据：[三个实际表项](../../../raw/current/scripts/fx.lua#L3468-L3489)及[动态选名与 AddChild](../../../raw/current/scripts/prefabs/wurt.lua#L499-L508)。这三个注册名都已正确写入 frontmatter aliases，动画对应也正确，故不构成事实错误或整页检索缺失。
- 建议：同一表格单元格写全三个注册名，保留合并行即可。这样正文 grep 和只读“注册与构造”章节时也可直接定位；无需拆页或改别名／来源。
- 处置：非阻塞建议，本次未修页。

## 检索与证据复核

- 对 current/scripts 全树 4,095 个 Lua 文件检索 69 个完整名，候选共 262 行，逐项区分表项、依赖声明、注释、实际生成和动态消费；未截断候选后下负向结论。
- 追加 `palmcone_leaf_fx_`、`wurt_water_splash_`、`wagpunksteam`、`splashfxlist`、`GetSinkEntityFXPrefabs` 等字段／前缀检索，复核树木拼名、装备调度、束缚碎裂和 VOID 数组消费。跟读 `SpawnFragment`、`SpawnChild`／`AddChild`、`GetPositionAdjacentTo`、`SetupLunarResists` 等必要实现，未无限展开消费者内部主题。
- `spell_fire`／`fire_throw` 共 5 行：1 处表项、1 处依赖、3 处 UI 动画字段，没有可据以推翻页面范围说明的生成候选。`sugarwood_leaf` 及相关组合检索共 8 行，只见表项与依赖；事件包不可见的限制保留。`AddBonusHealth` 及名称变体检索共 3 行，为定义、注释、导出；保留“未找到原版调用，不排除动态／外部调用”，不升级为不存在。
- 简中 `.po` 精确检索 69 个注册名的 `STRINGS.NAMES` 键均无命中；[Wurt](../../../raw/current/scripts/languages/chinese_s.po#L352520-L352522)与[Wortox](../../../raw/current/scripts/languages/chinese_s.po#L352480-L352482) 的官方角色名称与页面一致。这不等于对其他可能的名称覆盖机制作全域否定。
- 69 个注册名在当前 Prefab aliases 中均只有一个主要归属；正文直接引用文件都列入各自 sources。除 R01 的两处缩写外，负责注册名均在正文完整出现。

## 实际校验及并行边界

| 检查 | 本轮实际结果 |
|---|---|
| `validate_page --check-indexes` | 所选 11／11 页均 passed、issues 为空；全库导航两次均失败，退出 1，唯一问题是范围外 `reference/prefabs/nightsword.md` 的 `registration: ... got 0`。复跑时为 1,126 知识页、34 索引。该装备批正在并行写入，本轮保留其改动，不代其登记。 |
| 本轮叶级登记定向核对 | 11 页各恰有一个索引表格登记：world 索引 2 页、character 索引 4 页、table 索引 5 页；没有把范围外导航失败归给本轮页面。 |
| `validate_source_links --strict` | 224 处行引用、217 个唯一目标、73 个源码文件；error 0、warning 0、skipped_unsupported 0、未截断，退出 0。 |
| `validate_source_hashes check --fresh-hours 0` | 11 页／87 组来源比较均 unchanged，changed／sources_changed／unrecorded／invalid 均为 0，退出 0；未 record。 |
| 章节读取 | 11 页分别以标题原文 `#关联机制导航` 调用 query_wiki read，11 次均退出 0。未重建查询缓存或用 stale search 代替验收。 |
| 输入稳定性 | 开始与结束比较所选 11 页及其 73 个不同来源文件，84 个文件哈希均未变；BUILD 仍为 26335394。 |
| 差异检查 | 写报告前 `git -c core.safecrlf=false diff --check` 通过；写后再核对本报告链接、日志与 diff。 |

未运行 DST；页面无 Lua 示例，本次不声称 Lua 示例语法或运行时验证通过。格式、行号及哈希通过不替代上述技术核验；本报告不验收范围外的并行装备页，也不表示重新检视了全库。

## 进度与下一轮

固定 158 篇新增 Prefab 页队列累计已独立检视 **60／158 页，剩 98 页**；“已检视”不等于所有历史问题都已收口，仍不替第 5 轮 R10 作独立修后结论。按每轮约 10 页，剩余初检约 10 轮，修后复验另计。

下一轮建议原生成批 12 的 F42～F46 五页，加外围 C 的 `emberlight`、`lightning`、`staffcastfx` 三页，共 **8 页／45 个本轮页面负责的注册名**；外围 C 历史 17 名含既有宿主页的 3 名，不能全计入这三篇新页。随后继续按台账顺序合并相邻外围批。

## 2026-11-06 修后独立复验与收口

- R01 已覆盖：[沃特页第 31 行](../../../wiki/reference/prefabs/wurt_terraformer_fx_shadow.md#L31)在原合并行中写全 `wurt_water_splash_1`、`wurt_water_splash_2`、`wurt_water_splash_3`；动画对应、aliases、sources 和证据锚点均未变。与三个源码表项及 `wurt.lua` 动态选名一致，未发现新 P1/P2。该页仍为 53 行，字节数由 5,298 增至 5,334。
- `validate_page --check-indexes`：所选沃特页通过；全库导航通过，1,143 篇知识页／34 个索引／0 问题，退出 0。初检两次出现的范围外 `nightsword` 未登记问题已消失；本次没有代为修改装备页或其索引，也不把导航通过称为装备页语义验收。
- `validate_source_links --strict`：18 处行引用、18 个唯一目标、5 个来源文件，0 错误／0 警告，退出 0；哈希 check 为 5 组全部 unchanged，sources_changed／unrecorded／invalid 均为 0，未 record。
- 以标题原文 `#注册与构造` 调用 query_wiki read，退出 0，返回章节包含三个完整水盾名；无需依赖 frontmatter 或整页补读。其余 10 页在与初检相同的 HEAD 下没有工作区差异，沿用本轮初检，不冒充重新审阅全部 11 页。
- BUILD 实读仍为 26335394；只更新本报告与日志，不改知识页、基线、索引、PLAN、台账、Skill 或工具，不提交、不推送。未运行游戏，不替第 5 轮 R10 收口。

第 8 轮可以收口。累计仍为已检视 60／158 页、余 98 页；复验不重复计入初检进度，下一轮范围沿用上节建议。
