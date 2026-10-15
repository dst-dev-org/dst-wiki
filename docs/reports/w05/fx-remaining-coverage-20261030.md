# 剩余 FX 覆盖盘点与下一批候选（2026-10-30）

Steam buildid：26230496；知识仓 HEAD：ad42b8fc793ce5a4ec74f67c63c0138e025b20a4。此报告是工程盘点，不是游戏机制 Wiki，也不升级任何页面的验证等级。

## 结论

上一批完成的是 `prefabs/*_fx*.lua` 专属工厂的 75 篇新页，连同此前 12 篇皮肤 FX 页覆盖 87 个匹配文件。这个文件口径不能推成全游戏 FX 完成。

本轮核对的通用表有 **504 个唯一注册名：453 个有效静态条目＋51 个循环变体**。当前 Prefab 文件名／标题／aliases 没有为这些名字提供专门的元数据入口；其中 **30 个在 reference 正文或元数据中按完整标识符出现，474 个未出现**。全文出现只是定位信号，可能是消费者、资源或局部说明，不等于有完整产物页；504 也不等于要建 504 页。

| 范围 | 实查结果 | 交付 |
|---|---|---|
| `fx.lua` 通用注册表 | 504 名，已展开 6 组循环；无重名 | [逐注册覆盖表](fx-table-coverage-20261030.csv) |
| 上述静态条目 | 453 名；忽略注释与字符串中的伪代码 | 覆盖表的 literal 行 |
| 上述循环条目 | 51 名，展开值与当前循环／枚举核对 | 下表与覆盖表 |
| 其他 Prefab 词法候选 | 380 个文件；混合宿主、游戏实体、独立 FX、本地子体与 helper | [外围候选表](fx-other-candidates-20261030.csv) |
| 外围候选已有来源引用 | 51 文件在 Prefab 页 sources 中，另 137 文件只在其他知识类型中引用；共 188 | 表内记录具体页面，不据 sources 判完整覆盖 |
| 外围候选注册未解析 | 14 文件没有提取到字面 Prefab 或受限包装调用名 | 保留待审，不据此判断没有注册 |
| 下一批候选 | 10 个族、80 个注册名，尚未生成或认领其知识页 | 后文队列 |

## 方法与边界

1. 只读当前源码与 reference Markdown，跳过导航页；Prefabs 文件名与 aliases 按完整英文标识符匹配，全文定位采用标识符边界。中文俗名没有被当成注册覆盖证据。
2. 对 `fx.lua` 做词法分词，忽略行注释、长注释和字符串内的标点，再按主表直接子项取 name；未执行游戏 Lua。主表 453 项均为唯一字面名称；后置 6 组循环按当前常量／枚举展开。
3. 实读 [通用加载器](../../../raw/current/scripts/prefabs/fx.lua#L161-L172)：require("fx") 后逐项 MakeFx，最终返回 Prefab(t.name)。因此这里统计的是加载器产物名，不把 bank/build 或动画名当注册名。
4. 扫描其他 prefabs Lua 文件中的显式 AddTag("FX")、字面 Prefab 名含 fx，以及受限的首参 name 包装调用；排除上一批 `*_fx*` 文件与通用加载器。仅做候选发现，包装调用没有完整跨函数求值、返回可达性或逐注册类型证明；动态标签／其他包装可能漏掉。**380 不是全游戏 FX 总数或待建页数。**
5. 来源引用、正文提及、同名文件是三种不同信号；表格保留区别。开发需求的实际覆盖还须读取对应章节及消费者，缺少 aliases 也不能单独证明缺知识页。
6. 工程 CSV 不进入 Wiki 索引或查询缓存；它们供下一轮选题和防重复认领，不作为游戏行为证据。新增页面前仍须复查 PLAN、现文件和必要调用链。

## 循环变体核对

| 构造 | 注册数 | 范围／枚举 | 源码 |
|---|---:|---|---|
| crater steam | 4 | crater_steam_fx1～4 | [fx.lua:4047-4055](../../../raw/current/scripts/fx.lua#L4047-L4055) |
| slow steam | 5 | slow_steam_fx1～5 | [fx.lua:4057-4065](../../../raw/current/scripts/fx.lua#L4057-L4065) |
| shadow shield | 6 | shadow_shield1～6，j=0/3 与 i=1/2/3 组合 | [fx.lua:4067-4079](../../../raw/current/scripts/fx.lua#L4067-L4079) |
| slingshot hit | 18 | SHOT_TYPES 逐项拼接，不用弹药名称猜完整集 | [fx.lua:4081-4134](../../../raw/current/scripts/fx.lua#L4081-L4134) |
| lunarflame puff | 12 | tiny/small/med/large × _low/空/_high | [fx.lua:4136-4157](../../../raw/current/scripts/fx.lua#L4136-L4157) |
| wx78 shield | 6 | full、half 与四种转场；不虚构 empty 常驻项 | [fx.lua:4159-4197](../../../raw/current/scripts/fx.lua#L4159-L4197) |

注释里的 spawn_fx_large、spawn_fx_huge、splash_teal、splash_black 不计入注册。以上两组蒸汽可同族列两个子表；shadow_shield 六名继续留后续队列，未挤入本次 10 族。

## 已有承载与暂缓归类

| 对象／来源 | 当前实际承载 | 后续处理 |
|---|---|---|
| backpack_swap_fx | [backpack](../../../wiki/reference/prefabs/backpack.md) 注册表、皮肤与拥有／释放接线 | 不重复建页 |
| heatrocklight | [heatrock](../../../wiki/reference/prefabs/heatrock.md) 注册表与灯光管理 | 不重复建页 |
| staffcoldlightfx | [staffcoldlight](../../../wiki/reference/prefabs/staffcoldlight.md) 有独立变体小节，仍有玩法组件 | 不因 fx 后缀再拆纯视觉页；emberlight 的缺口另议 |
| rain／caverain | [rain-fx-gating](../../../wiki/reference/mechanisms/rain-fx-gating.md) 已记录门控链与驱动 | 复用既有机制；完整工厂覆盖另判 |
| shock_fx／shock_arc_fx | [electrocute_fx](../../../wiki/reference/prefabs/electrocute_fx.md) 的生成／消费关系 | 新族页只补其工厂与差异，并互链 |
| wortox_soul_spawn | [魂入 FX](../../../wiki/reference/prefabs/wortox_soul_in_fx.md) 与 [治疗 FX](../../../wiki/reference/prefabs/wortox_soul_heal_fx.md) 记录部分宿主调用 | 不是本体全覆盖；其投射物、灵魂发放和 SoulSpearTick 伤害不能归成普通表现差异 |
| sparks／electrichitsparks／electrichitsparks_electricimmune | [sparks.lua](../../../raw/current/scripts/prefabs/sparks.lua#L139-L179) 已核三个 MakeSparks 返回 | 通用表以外的下一轮独立工厂候选，不与 shock_fx 混同 |

## 下一批候选：10 族而非 80 页

先维护一份通用工厂协议，再用少量同族 Prefab 页承载变体。候选分组依据已读构造与配置差异，生成点、宿主与回收仍须在各族建页时完整核验；本表不表示 80 个产物都已经完成机制复审。这里的中文是选题描述，不冒充官方译名。

| 组 | 机制族 | 注册数 | 建议主文件名（尚未创建） | 组织与待核边界 |
|---|---|---:|---|---|
| G01 | 蒸汽编号变体 | 9 | `crater_steam_fx1.md` | 9 个编号放同页变体表；先查实际生成名与调用条件。 |
| G02 | 基础水花与雪花溅落 | 5 | `splash.md` | 5 个条目合表，区分音效／build；splash_ocean 有旧海洋注释，不能推定当前仍生成。 |
| G03 | 沙尘与暗影烟团 | 9 | `sand_puff.md` | 9 个条目合表；前后层、透明度、缩放与 itemmimic 使用条件单列，不按资源相同认定协议一致。 |
| G04 | 四季树叶飘落与砍伐 | 8 | `green_leaves.md` | 8 个条目同页；chop／fall 与四色分两维表，purple_leaves 另属资源族，暂不合并。 |
| G05 | 生成烟雾 | 5 | `spawn_fx_small.md` | 5 个条目合表；medium_static 的暂停更新需保留端与时钟边界。 |
| G06 | 电击附着表现 | 6 | `shock_fx.md` | 6 个条目合表；与现有 electrocute_fx 互链，查 autorotate／面向，不把表现当电伤结算。 |
| G07 | 位面命中与抵抗表现 | 2 | `planar_hit_fx.md` | 2 个条目同页分职责小节；复用 planarentity 既有机制，仅核表现生成、父挂与清理。 |
| G08 | 弹弓弹药命中表现 | 18 | `slingshotammo_hitfx_rock.md` | 18 种弹药用变体表；核特定动画／覆符号／音效，四个 AOE 条目另候选，不随 bank 自动合并。 |
| G09 | 月焰消退烟团 | 12 | `lunarflame_puff_small.md` | 12 个条目用尺寸×高度矩阵；与 warg_mutated_breath_fx 主体互链，不复制判伤。 |
| G10 | WX78 护盾动画 | 6 | `wx78_shield_full.md` | 6 个条目同页；破盾转场与 11 帧解除父挂单列，先核消费者再写清理承诺。 |

G01～G05 共 36 名，可先交付 5 篇同族页作第一小批；G06～G10 共 44 名随后做。各组的精确成员在覆盖 CSV 的 proposed_group 列，不能仅按名称前缀或共享 bank 外扩。

公共协议的建议归属为 mechanisms 下的通用 FX 工厂页：记录网络代理与本地动画的分工、父挂／位置传递、声音与暂停时钟、fn／fntime 的回调对象、animover／animqueueover 回收和一秒代理寿命。尚未创建该页；不把一秒代理回收误写成全部本地表现一秒内清空，也不据动画名判断循环或自动退场。

首批实际内容落地时再新增非空的通用表 FX 叶级索引，挂入 Prefab 索引；专属工厂 fx-index 保留原有 75 页入口。正文一份，差异用表；只有职责／接入／清理明显不同才设产物小节，不默认一个注册名一页。共用 bank 只是分组线索，不能证明生成或清理协议相同。

## 验证与未覆盖

- CSV 回读通过：通用表 504 名唯一、外围 380 文件唯一；453 个静态 name 行逐项匹配源码，6 组动态名称集合逐项核对，4 个注释内负例均未计入。
- 已提及的 30 名按完整标识符逐项复查文件存在与文本匹配；CSV 所列来源与既有页面路径全部存在。10 组计数与精确成员无重复，共 80 名，其余 424 名维持 pending。
- 工程报告 17 个本地链接与其中 8 组 Lua 行号区间通过存在性／范围检查；这是报告路径检查，不替代正文机制语义审阅。
- 未新增或改写游戏知识页，不运行知识页校验、Lua 示例语法、record 或检索缓存重建；工程 diff 检查通过。通用表与加载器操作前后 SHA-256 一致，raw 只读。
- 只完成注册与覆盖信号盘点及候选排队；尚未逐一审阅 504 个条目的所有消费者，也未解析其他工厂的所有动态注册。未运行游戏、未解包动画，未提交或推送。
