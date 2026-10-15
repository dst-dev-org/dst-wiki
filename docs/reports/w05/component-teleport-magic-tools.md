# 传送与魔术工具组件检视

2026-10-20 完成第 57～61 批五页，依据 Steam buildid 25643519。沿用 dst-wiki Skill，游戏源码与输入模组只读；文本及临时 Lua 通过 apply_patch 写入／清理，复用已有快照、工具与环境。

## 交付范围

| 批次／页面 | 实际行数／UTF-8 字节 | 完整 modmain 案例 |
|---|---|---|
| 57：[blinkstaff](../../../wiki/reference/components/blinkstaff.md) | 64／6,517 | 懒人魔杖关闭组件的出发／落地声音，保留 SG 声音、FX 和即时消费 |
| 58：[recallmark](../../../wiki/reference/components/recallmark.md) | 71／7,139 | 服务端私有事件把原回溯表重设到当前玩家持有者位置，保留许可／标记刷新 |
| 59：[remoteteleporter](../../../wiki/reference/components/remoteteleporter.md) | 68／7,766 | 原传送伞物品收集半径设为 3，保留技能、供电、耗能及 SG 收集／恢复 |
| 60：[magician](../../../wiki/reference/components/magician.md) | 70／7,648 | 原麦斯威尔服务端结束当前工具会话，保留物品归还与人物事件 |
| 61：[magiciantool](../../../wiki/reference/components/magiciantool.md) | 69／7,491 | 原 tophat 在事件执行时查询动态组件，从工具侧请求完整停止链 |

五页合计 36,561 字节，均在页面大小上限内；采用五主节，远程传送的 SG 协作归接口下三级小节，没有新增类型、必填章节或空分类。各例沿用原宿主资源，不交付完整新传送系统、回溯技能或魔术容器工程。

- blinkstaff 的 true 表示延迟任务已开始，不是已抵达；0.25 秒后复查落点，原 onblinkfn 却在开始时即时扣精神／耐久。原宿主是 orangestaff，显示、无敌、阴影和中断恢复另有 quicktele 协作；关闭组件声音不等于关闭整个 SG 音效。
- recallmark 保存坐标和分片，GetMarkedPosition 不返回其他分片坐标；MarkPosition(nil) 不是清除，Copy 也不返回标记结果。原 marker、库存／死亡／复活监听和实际回溯分工明确；示例成功后的标记会存档，移除示例不会自动还原旧位置。
- remoteteleporter 从世界管理器取台，按距离阈值选台，不依实际相机。CanActivate、物品收集／隐藏、Teleport 和阶段回调分离；NODEST 仍调用原耗能回调，不能据失败断言免费。移动后的物品还需 SG ReturnToScene，集合与失败残留不当作通用迁移协议。
- magician 暂存物品时改父实体、场景与 persists，停止先清人物引用再调用工具并恢复／归还。held／equip 不记录原箱子格子，不能保证放回原位置；嵌套存档和人物等待库存加载后的 StopUsing 分别核验。
- magiciantool 的 OnStart／OnStop 是工具握手，不能替代人物取物／归还；开始回调 false 不回滚。原高礼帽按配方／读档动态转换，容器关闭也会请求停止；实际物品名仍为 tophat。waxwelljournal 的 TopHatSpellFn 位于整段注释，不作为当前运行转换方案。

未全面审校客户端预测、全部跨分片迁移、世界管理器登记、容器代理、所有物品删除／满库存去向或其他模组损坏引用的恢复。已有 inventoryitem 的末端持有者接口与 magician 页的 SG 停止结论按匹配版本／条件复用，不重复铺开完整依赖链。

## 检查与受控读取

逐批页面格式、来源文件、本地链接／锚点、大小和全库导航登记通过。五份完整 Lua 示例与临时文件逐字核对，使用已有 Full Moon 二进制执行 Lua 5.1 syntax-only 后清理；没有执行示例或启动游戏。

五页合并 strict 源码引用检查：145 处引用、103 个唯一目标、15 份来源文件，0 error／0 warning，truncated=false、omitted=0、by_page_omitted=0。全部 158 个已建组件的 CSV 页面集合与实际文件一致，原字节 SHA-256 与当前快照匹配；这些检查不证明全库语义、运行调用或游戏行为通过。

| 页面 | “典型接入”JSON stdout 实际 UTF-8 字节 |
|---|---:|
| blinkstaff | 4,440 |
| recallmark | 5,335 |
| remoteteleporter | 4,312 |
| magician | 4,956 |
| magiciantool | 4,997 |

五次最终章节读取均完整返回元数据、快速判断、完整案例／条件与验证记录；complete_selected_ranges=true、source_version=match、truncated=false、omitted=0，示例代码往返一致。magician 已补工具页互链，表中为补链后的输出；字节只作读取规模代理，不换算模型 token，不称独立模型自主评测，未重建缓存。

## 时间检查点

工具 UTC 时钟换算为 Asia/Shanghai，均为 2026-10-20：开始点 23:20:16，第 57／58／59／60／61 批记录结束点依次为 23:22:38、23:24:14、23:26:21、23:28:28、23:30:55。结束点包含相应页面／登记／适用检查和临时文件清理。

检查点跨度 10 分 39 秒，存在跨批预读与共享上下文复用，不是五个独立冷启动作者样本；工程文档、报告、日志、最终互链及复核在此后完成，另计，不外推全库工期或 token 收益。

## 登记与后续

五页均唯一登记在交互技能主题：人物 16／物品 30／战斗 13／世界 66／交互技能 33，共 158 篇组件；加 3 篇 modinfo、5 篇 cookbook，合计 166 篇知识页、12 篇导航、22 条 Top 30。

798 个组件候选中 158 已建、640 未建，另 19 replica／3 helper／1 empty 单列。magician／magiciantool 来自原专项池，其余三项来自扩展池；现余扩展 409／专项 228／职责待确认 3。历史路线、报告和日志保留。交互技能主题 61 行／6,326 字节，组件入口 25 行／1,885 字节，均未超限，正文地址稳定。

同步覆盖 CSV、主题／组件导航、PLAN、README、设计及 docs 入口，并追加一次日志。按本次要求完成五批后收尾，下次第 62 批候选 key／lock，先核验钥匙／目标、匹配／执行与消费，再决定案例和拆批；heatrock Prefab、配方 API 和 cookbook 原队列保留。

本轮临时 Lua 已清理；未安装依赖、编译解析器、运行游戏或独立模型评测、重建缓存、提交或推送。五页保持 source-reviewed，不升级 runtime-tested。

收尾复核：五页及两张导航的格式／来源／本地链接／大小与全库登记通过；五份工程 Markdown 的 137 处本地链接／锚点、git diff --check 通过。日志追加后为 234 行／34,800 字节，未触发归档；第 57～61 批临时 Lua 为零。
