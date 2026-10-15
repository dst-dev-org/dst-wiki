# 投射物与爆炸组件检视

2026-10-17；Steam buildid 25643519。按 dst-wiki Skill 核验组件实现、必要发射链／工厂／触发入口，交付第十批三页。会话源码准备返回 Skip，未解压；写入因自动审批服务错误中断一次，恢复后继续，失败尝试未落盘。

## 交付范围

| 页面 | 核验重点 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [projectile](../../../wiki/reference/components/projectile.md) | 目标追踪、命中结算、两种 owner 语义、落空清理与吹箭速度 | 76／8,886 |
| [complexprojectile](../../../wiki/reference/components/complexprojectile.md) | 总初速、高低弧线、重力、坐标修改与落地替换 | 76／9,256 |
| [explosive](../../../wiki/reference/components/explosive.md) | 整堆普通伤害、工作量、点燃、归属筛选及燃尽销毁 | 81／8,930 |

组件索引唯一登记本批三页，现为 156 行／12,271 字节。当前 66 篇组件、3 篇 modinfo、5 篇 cookbook，共 74 篇知识页与 7 篇导航；798 个组件候选中 732 未建，覆盖清单仅更新本批三行并复核源码 SHA-256。原有页面正文与锚点未迁移。

## 关键边界

- projectile 的 Hit 先停组件／物理，再 prehit→combat:DoAttack→onhit；命中回调不证明净伤害。Miss 不自动停止马达，Stop 不等于删除／回收。
- 自身投掷由 combat 丢出武器后 Throw；独立弹体由 weapon 生成，owner 先是发射武器，命中时再取其直接持有者。Throw 第三参不固定最终攻击归属；has_damage_set 决定是否保留弹体自身 weapon。
- complexprojectile 的 horizontalSpeed 实际参与总初速求解；默认落地只调回调，不自动攻击。Launch 改输入 y、不可解退用 30 度；OnUpdate 回调返回 truthy 会取代默认更新。
- 小眼球案例保留原版物理／资源、落地创建实体／传 commander／删除空中对象，只改高低弧线选择，不发布未核验的新弹体工厂。
- explosive 的普通伤害和工作量按自身件数放大，特殊伤害在本文件没有同样的额外堆数乘法。lightonexplode 仅关闭点燃分支，不能当作无环境破坏保证。
- 爆炸回调在读取堆数／位置及扫描之前，返回不能取消；实际 GetAttacked 来源仍是爆炸实体。燃尽先运行普通回调再检查实体有效性，火药主动移除默认烧毁回调以保留爆炸入口。

## 读取与验证

作者受控按“典型接入”读取三页，均返回快速判断、完整 Lua 与加载／生效边界、验证记录，无截断／省略；不是独立模型自主发现评测，不将字节换算成 token 收益。

| 需求 | 目标 | JSON stdout 实际 UTF-8 字节 |
|---|---|---:|
| 普通吹箭速度改四十 | projectile | 4,355 |
| 小眼球抛射选低弧线 | complexprojectile | 5,523 |
| 火药降低伤害并关闭主动点燃 | explosive | 5,035 |

- 四个 Wiki 文件（含组件索引）格式、来源、标签、大小和本地链接／锚点通过；全库导航检查为 74 篇知识页、7 篇导航、22 条 Top 30，唯一登记／父入口通过。
- 118 处源码行引用、86 个唯一目标、10 份来源文件 strict 检查为 0 error／0 warning，报告未截断；初查发现两处结构／空行锚点，已改到实际赋值／函数入口后复跑通过。锚点检查不证明结论语义。
- 三段完整 modmain 示例与页面代码逐项一致，已有 Full Moon 3.0.0／Lua 5.1 探针 syntax-only 通过；byte/position 明确跳过，不宣称往返／类型／运行验证。临时 Lua 经 apply_patch 创建于本地 output，检查后删除。
- 同步 PLAN、README、总体设计、报告入口、覆盖 CSV 和追加日志；不改历史报告／日志，不写游戏或输入模组、Skill、工具箱，不重建缓存，不提交／推送。
- 页面均保持 source-reviewed。未运行游戏，未实测高低弧线／速度、多人物 PVP、物品归属变化、爆炸最终伤害、平台／离屏／同步及跨模组覆盖。

第十批完成；下批扩库前先拆组件主题导航，避免继续增长超过索引上限，然后推进第十一批定向技能与瞄准。heatrock Prefab、配方与骨架队列保留。
