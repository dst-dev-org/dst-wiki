# 打包与拆包组件检视

2026-10-17；Steam buildid 25643519。按 dst-wiki Skill 完成第十四批三页；源码准备返回 Skip，未解压。文本和临时 Lua 均通过 apply_patch 写入／清理。

## 交付与关键边界

| 页面 | 核验与完整 modmain 示例 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [bundlemaker](../../../wiki/reference/components/bundlemaker.md) | 材料配置／消费与成品选择；打包袋产出礼物 | 63／6,041 |
| [bundler](../../../wiki/reference/components/bundler.md) | 人物两阶段打包、取消／保存与 SG 清理；完成至少两格 | 86／7,968 |
| [unwrappable](../../../wiki/reference/components/unwrappable.md) | 内容记录／恢复、门禁／延迟／预览；关闭普通包裹预览 | 72／8,804 |

- bundlemaker 只保存配置并转发开始回调；原版消费在材料回调完成，取消时人物另 Spawn 材料，不能只移除消费步骤。改产 gift 也会采用礼物的预览／额外掉落差异，不只是换皮肤。
- bundler.CanStartBundling 要求已处于 bundle SG，不能在 idle 下当通用能力查询；动作上层另检查可见性。临时容器成功打开后才登记材料并消费，正常流程依赖人物／动作／SG，不直接给生物挂组件就完整可用。
- FinishBundling 只关闭容器并转 bundle_pst，OnFinishBundling 才生成与封装；两格示例保留原方法，但不修改按钮的非空可用条件，服务端拒绝一格时没有新增提示。
- StopBundling 归还内容并按材料名／皮肤重建，不保存任意材料实例状态；OnSave／OnLoad 保存／恢复中途记录，不等于自动恢复打包窗口。人物离场、组件移除与 SG 中断存在各自清理入口。
- WrapItems 对已有实体只保存、不删除；对字符串临时生成实体才在记录后删除。非空再次调用覆盖旧记录，空表不清空；输入无效／生成失败没有完整防护，不能直接包装任意未核验名称。
- Unwrap 清记录、重建落地物品并通知宿主；包裹删除与额外掉落在宿主回调。记录前后事件、OnLoad 重调 onwrapped、延迟重复请求和 canbeunwrapped 的动作边界均单独说明。
- GetSaveRecord 的 references 未由包装组件完整传递，拆包 SetPersistData 未提供 newents；不承诺任意关联实体完整恢复。预览使用只读临时副本，原内容记录不迁移，不把预览当真实容器取物。

## 检查与按需读取

五个 Wiki 文件（三页、物品主题与组件分类入口）通过格式、来源文件、标签、大小、本地链接和标题锚点检查。全库导航登记为 92 篇知识页、12 篇导航、22 条 Top 30，无遗漏／重复／父入口问题；不等于全库语义核验。

三页 93 处源码行引用、76 个唯一目标、13 份来源文件 strict 检查为 0 error／0 warning，输出未截断。三条组件覆盖记录保留并核对源码 SHA-256；798 个候选现为 84 已建／714 未建，另 19 replica／3 helper／1 empty 单列。

三段完整示例与页面 fenced Lua 逐项一致，通过已有 Full Moon 3.0.0／Lua 5.1 syntax-only 探针；byte／position 检查明确跳过，不声称类型、字节往返或游戏效果验证。三份临时 Lua 已删除，未安装依赖或构建工具。

作者受控读取各页“典型接入”，均包含快速判断、前提／完整代码和验证记录，truncated=false、omitted=0：

| 页面／需求 | JSON stdout 实际 UTF-8 字节 |
|---|---:|
| bundlemaker／打包袋产出礼物 | 4,357 |
| bundler／每包至少两格 | 4,912 |
| unwrappable／关闭普通包裹预览 | 4,417 |

未知名称的包装需求由组件分类入口进入物品主题“打包与拆包”，只筛三条再读所需页；已知名称直达。物品主题现为 68 行／4,695 字节，分类入口 25 行／1,306 字节，不加新的必读层。这是作者受控读取，不是独立模型自主发现评测，字节不换算 token 收益；未重建知识缓存。

同步 PLAN、README、总体设计、文档入口、覆盖清单与追加日志；保留历史记录与既有暂存／工作区变更。

## 后续与限制

第十四批完成；下一批 entitytracker／playerprox／positionalwarp／teleportedoverride／teleporter，第十五批剩 5 个候选。Prefab、注册 API 与场景骨架队列保留。

本批未改 Skill、工具箱、游戏源码或输入模组，未提交／推送。页面保持 source-reviewed，未运行游戏；多人／专服、UI、动画中断、保存／迁移、皮肤、燃烧、延迟重复请求、内容恢复和跨模组方法覆盖需实际游戏验证。
