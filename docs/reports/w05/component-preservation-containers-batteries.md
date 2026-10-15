# 通用组件：保鲜、容器协作与充电

2026-10-22，Steam buildid 25643519。用户要求在 DeepSeek 增量基础上继续建设，本轮启动三个子代理，分别交付 C04～C06，主代理审阅并统一登记。准备脚本确认同 build，返回 Skip，未更换源码。

## 基线与本轮交付

开始时核对 246 个组件页与覆盖清单一致，已登记来源 SHA-256 均匹配当前快照；没有重复生成其他 AI 的页面。该核对不代表对既有页逐项语义审校。

| 组 | 页面 | 物理行数／UTF-8 字节 | 完整 modmain 接入 |
|---|---|---|---|
| C04 | [preservative](../../../wiki/reference/components/preservative.md) | 63／5,735 | 盐的总新鲜度恢复量改为 25 个百分点，保留目标堆叠分摊 |
| C04 | [compostingbin](../../../wiki/reference/components/compostingbin.md) | 69／8,146 | 原版堆肥桶基础周期减半，保留计时恢复和宿主产物 |
| C05 | [container_proxy](../../../wiki/reference/components/container-proxy.md) | 84／9,433 | 原魔术箱入口暂停／恢复，先关闭再解绑，不删除共享容器 |
| C05 | [containerinstallableitem](../../../wiki/reference/components/containerinstallableitem.md) | 80／10,157 | 猪皮皮筋在原配置上提高弹速 10%，保留射程和卸载恢复 |
| C06 | [battery](../../../wiki/reference/components/battery.md) | 77／7,873 | 避雷针拒绝晨星取电，保留其他接收者和原消费 |
| C06 | [batteryuser](../../../wiki/reference/components/batteryuser.md) | 78／8,556 | 晨星每次最多补 25 个百分点，保留部分充电与源消费 |

六页共 49,900 字节，均在页面上限内；采用五个主节，协作细节归入对应主节的小节。未改模板、Skill 或工具箱。`container_proxy` 标识符不变，新页按格式使用连字符文件名，覆盖清单记录源码到页面的映射。

## 核验与修正

- preservative 是一次性恢复材料，区别于 preserver 的持续倍率；动作先改变目标新鲜度，再尝试扣材料，失败返回不保证无副作用。
- compostingbin 的容量门禁依赖宿主，周期实际固定耗料两份；运行中 Refresh 按剩余比例平方重算。移组件只注销监听，不自动停止 timer；示例不在 postinit 强制刷新当前周期。
- container_proxy 的 SetMaster 只换监听，不清旧会话；先 Close 再解绑。开关影响候选，直接 Open 仍需有效 master；RemoveComponent 会触发断言。
- containerinstallableitem 的资格对象与 install_target 分开；deferred 仅禁用 exitlimbo 卸载，ignoreuninstall 和弹弓转换使通知不保证成对。初稿打印示例已改为实际弹速扩展，以原配置绝对赋值避免重装累计，并核验发射消费者和原卸载恢复。
- battery／batteryuser 是电源协议，接收成功后才消费源；倍率可为 nil，接收实体可为物品。WX-78 单格电击充能、晨星燃料比例与薇诺娜持续电路供电分别说明。

各组审阅实现、必要动作与消费者、实际宿主及注册／恢复清理；已建同版本 inventory、container、timer、perishable、upgrademoduleowner 等机制按需复用。主代理通读六页、复核关键顺序、公式和消费者，未替其他 AI 重审全部原版宿主。

## 检查结果

- 六页与四张改动导航的元数据、来源存在性、大小、Wiki／Markdown 链接通过；全库导航为 260 篇知识页、13 篇索引、22 条 Top 30，唯一叶级登记和父入口通过，Top 30 未扩增。
- 六页共 190 处源码行号引用，144 个唯一目标、27 份来源文件；strict 校验 0 error／0 warning，报告未截断。
- 三个代理检查六个最终完整 Lua 示例，Full Moon Lua 5.1 syntax-only 均通过；安装物示例修订后重新检查，其旧打印稿不作为交付案例。临时 Lua 已由各代理用 apply_patch 删除。
- 六次“典型接入”读取均为 source_version=match、complete_selected_ranges=true、truncated=false、omitted=0；返回章节与原 Markdown 逐字一致，保留前提、完整 Lua、快速判断和验证记录。
- 合并后覆盖清单的 252 个 built 记录与实际组件页面集合一致，252 个主源码 SHA-256 全部匹配。未自动重写旧记录的路径风格或来源证据。

| 页面 | 章节读取 JSON stdout UTF-8 字节 |
|---|---:|
| preservative | 4,004 |
| compostingbin | 4,305 |
| container-proxy | 5,997 |
| containerinstallableitem | 6,384 |
| battery | 4,508 |
| batteryuser | 4,532 |

字节是读取规模代理，不是模型 token 测量；代理并行不证明总 token 成本下降。所有页面保持 source-reviewed；未运行 DST，未验证多人、读档、跨模组回调和完整技能／动画表现。

## 登记与后续

preservative、container-proxy、containerinstallableitem 登记 items；compostingbin 登记 world；battery／batteryuser 登记 abilities。组件分类入口只更新主题范围，未增加查询必读层级。同步 [覆盖清单](component-coverage.csv)、[计划](../../../PLAN.md)、README、设计与工程报告入口，日志只追加一次。

累计 252 个组件已建、546 个未建；另有 3 篇 modinfo、5 篇 cookbook，共 260 篇知识页与 13 篇导航。按历史路线重算剩余扩展 318、专项 225、职责待确认 3；分类不是实际使用频率或职责核验。

下一轮从剩余通用候选按任务缺口选题，先重查认领与已有页面；不预建空页，不重复 DeepSeek 的天气／施工／垂钓／航海等交付。未安装、构建、重建缓存、运行独立模型评测、提交或推送，未修改游戏源码与历史报告。

收尾核对：五份工程 Markdown 的 164 处本地引用／锚点有效，git diff --check 通过。覆盖 CSV 无重复 source 或多余列，C04／C05／C06 各登记两页；日志 16 行／2,099 字节，无需归档。
