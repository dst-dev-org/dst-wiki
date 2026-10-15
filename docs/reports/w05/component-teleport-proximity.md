# 传送与距离触发组件检视

2026-10-17；Steam buildid 25643519。按 dst-wiki Skill 完成第十五批五页；源码准备返回 Skip，未解压。文本与临时 Lua 均通过 apply_patch 写入／清理。

## 交付与关键边界

| 页面 | 核验与完整 modmain 示例 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [entitytracker](../../../wiki/reference/components/entitytracker.md) | 名称关系／移除监听／GUID 恢复；木箱记录建造者 | 75／5,380 |
| [playerprox](../../../wiki/reference/components/playerprox.md) | 五模式／近远过滤／生命周期；虫洞只响应活玩家并扩大距离 | 73／6,710 |
| [positionalwarp](../../../wiki/reference/components/positionalwarp.md) | 位置采样／消费／标记与旺达协作；年龄回退点数翻倍 | 73／6,496 |
| [teleportedoverride](../../../wiki/reference/components/teleportedoverride.md) | 薄回调与紫法杖选择顺序；猪人选当前位置 | 68／5,558 |
| [teleporter](../../../wiki/reference/components/teleporter.md) | 配对／接收／SG／存档迁移；虫洞接收偏移 | 77／8,485 |

- entitytracker 不查找、控制或同步目标；同名直接覆盖保留旧监听，须 Forget 后再 Track。GUID 恢复不保证玩家重连身份或跨世界所有权。
- playerprox 默认 AnyPlayer 为整体开关，AllPlayers 才逐人通知；指定目标和锁定后分支不统一使用生死过滤。无效目标可能不发 far，离屏停止轮询。丢目标路径读 initialtargetmode，而 setter 写 originaltargetmode，本页保留实现边界，不承诺自动恢复。
- positionalwarp 按移动距离采样，回退点数不等于秒数／固定距离；取点与实际传送、冷却、SG 及历史消费分开。历史不跨迁移恢复，marker 需对应接口，不把任意 FX 当标记。
- teleportedoverride 是供消费者查询的回调，不拦截所有瞬移。nil 交回默认选择，保持当前位置仍可能消耗法杖并触发闪电／地震与隐藏恢复；位置优先于目标坐标，不取消目标后续通知。
- teleporter 接收端 offset、临时出口、迁移与 SG 合作分别说明；active／Activate true 不证明安全落地，busy 不自动阻止激活。enabled／migration_data 的保存恢复及 data.items 从索引 0 写入的实际边界保留，不把实现观察升级为已复现游戏 bug。

## 检查与按需读取

七个 Wiki 文件（五页、世界主题与组件分类）通过格式、标签、来源文件、大小、链接和标题锚点检查。全库导航为 97 篇知识页、12 篇导航、22 条 Top 30，无遗漏／重复／父入口问题；不表示全库语义审阅。

五页 117 处源码行引用、93 个唯一目标、18 份来源文件 strict 检查为 0 error／0 warning，结果无截断／省略。首次检查发现 teleporter 九处引用端点为空行／结构行，已按实际函数区间修正并复查。五条覆盖记录的源码 SHA-256 与原清单逐项一致；798 个候选现为 89 已建／709 未建，另 19 replica／3 helper／1 empty 单列。

五段临时 Lua 与页面完整示例逐项一致，通过已有 Full Moon／Lua 5.1 syntax-only 探针；byte／position 明确跳过，不声称类型、字节往返或游戏效果验证。临时文件完成后删除，未安装依赖或构建工具。

作者受控读取各页“典型接入”，均包含快速判断、完整示例与验证记录，truncated=false、omitted=0：

| 页面 | JSON stdout 实际 UTF-8 字节（换行按 LF 计） |
|---|---:|
| entitytracker | 4,258 |
| playerprox | 3,879 |
| positionalwarp | 4,001 |
| teleportedoverride | 4,012 |
| teleporter | 3,990 |

未知名称需求可由组件分类进入世界主题“实体关联、距离与传送”，筛五条再读所需页；已知名称直达。主题 65 行／4,616 字节，分类 25 行／1,333 字节，没有新增必读层级。这是受控章节读取，不是独立模型自主发现或 token 收益评测；未重建缓存。

同步 PLAN、README、总体设计、文档入口、覆盖清单与追加日志；工程 Markdown 的本地链接／锚点单独检查，历史记录与已有暂存／工作区变更保留。

## 后续与限制

原第 6～15 批队列完成。下一批拟从农作扩展池选 fertilizer／fertilizable／farmtiller／farmplanttendable，执行前核验职责与消费者；Prefab、注册 API 与场景骨架队列保留，不等待全部组件完成。

本批未改 Skill、工具箱、游戏源码或输入模组，未提交／推送。页面保持 source-reviewed，未运行游戏；多人／专服、鬼魂与目标离场、旺达年龄／连续倒走、平台／水陆、传送中断／存档和分片迁移仍需游戏验证。
