# 持续施法与物品使用协议组件检视

2026-10-20 完成第 47～51 批五页，依据 Steam buildid 25643519。沿用 dst-wiki Skill，游戏源码与输入模组只读；页面、工程说明及临时 Lua 均以 apply_patch 写入或清理。

## 交付范围

| 批次／页面 | 实际行数／UTF-8 字节 | 完整 modmain 案例 |
|---|---|---|
| 47：[channelcastable](../../../wiki/reference/components/channelcastable.md) | 66／6,238 | 原打火机服务端取消吸火事件，保留技能动态挂载与双方停止链 |
| 48：[channelcaster](../../../wiki/reference/components/channelcaster.md) | 75／7,288 | 人物持续施法时增加每分钟三点精神贡献，停止后撤销独立来源 |
| 49：[chargingreticule](../../../wiki/reference/components/chargingreticule.md) | 64／6,548 | 原本地蓄力 FX 的手柄旋转插值系数改为 12，不改变攻击范围或鼠标更新 |
| 50：[toggleableitem](../../../wiki/reference/components/toggleableitem.md) | 70／5,607 | 给原剃刀新增私有双态标记，对新组件局部适配 USEITEM 返回值 |
| 51：[useableequippeditem](../../../wiki/reference/components/useableequippeditem.md) | 68／7,123 | 原 WX78 遥控器服务端停止事件，保留技能／装备链和无人机业务收尾 |

五页合计 32,804 字节，均采用五个主节；必要生命周期和原版参照归入接口／案例，没有新增类型、必填章节或空分类。

- channelcastable 的 OnStart／OnStop 是物品握手，不替代人物完整状态切换；活跃卸装停止并移组件。物品 SetStrafing(false) 不取消人物组件的基础移动倍率。案例执行时取得当前组件，不覆盖原技能条件或效果回调。
- channelcaster 支持 nil 物品模式，替换时先停止旧流程再验证新物品；SG 标签、人物回调与事件共同决定清理。精神贡献复用同版本 sanity 页，不把贡献值当最终净增长，不覆盖原人物回调。
- chargingreticule 是本地 HUD 创建链，ease／smoothing 只影响带 dt 的旋转插值；鼠标路径不传 dt。重复 Link 不先清旧输入 handler，按宿主删除旧 FX／新建的模式清理，不把移组件视为等价操作。
- toggleableitem 先翻转 on 后调用回调，没有失败回滚、存读或业务停止实现。原 ToggleItem 不返回成功值，USEITEM 分支却直接返回它；本例只包装自己新增的组件，不全局修改动作。未核验到静态原版挂载实例，不把模组新增示例称为原版用法。
- useableequippeditem 启动前置 inuse，明确 false 拒绝并回滚状态，nil 兼容为成功；停止回调仅在使用中执行，移组件只移标签。原遥控器的组件由技能动态安装，原 OnStopUse 不消费 doer；示例省略 doer 仅适用于这个宿主，不推广为所有装备契约。

没有全面审校所有 SG／预测、无人机 AI、RPC 权限、物品热迁移、输入切换或其他模组直接改字段的行为。三套使用协议与建筑 channelable 分别保留边界，源码审阅不等同游戏运行。

## 检查与读取

逐批完成页面格式／来源／本地链接／锚点／大小及导航登记检查；五份完整示例与临时 Lua 逐字核对，已有 Full Moon 二进制的 Lua 5.1 syntax-only 通过后清理，没有执行示例。

五页合并 strict 源码引用检查：106 处引用、75 个唯一目标、13 份来源文件，0 error／0 warning，truncated=false、omitted=0、by_page_omitted=0。全部 148 个已建组件的 CSV 页面集合和原字节 SHA-256 与当前快照一致；这不证明锚点语义、运行调用或全库游戏行为已验证。

| 页面 | “典型接入”JSON stdout 实际 UTF-8 字节 |
|---|---:|
| channelcastable | 4,351 |
| channelcaster | 4,873 |
| chargingreticule | 4,368 |
| toggleableitem | 4,353 |
| useableequippeditem | 5,451 |

五次按已知页选择章节，均保留元数据、快速判断、完整案例／条件及验证记录；complete_selected_ranges=true、source_version=match、truncated=false、omitted=0。stdout 按 LF 计实际字节，不换算模型 token，不称独立模型自主评测；没有重建缓存。

## 时间检查点

工具 UTC 时钟转换为 Asia/Shanghai，以下均为 2026-10-20。开始点 22:37:33，第 47／48／49／50／51 批的记录结束点依次为 22:41:31、22:41:56、22:45:14、22:49:33、22:51:05；结束点包含相应页面／登记／适用检查及临时 Lua 清理。

检查点墙钟跨度 13 分 32 秒，包含跨批准备、共享上下文复用和续接核对，并非五个独立冷启动作者样本；不能用相邻检查点差值估算单篇写作成本。统一工程文档、报告、日志与最后核对在此后完成，不混入这段跨度，不外推全库工期或 token 收益。

## 登记与后续

五页均唯一登记在交互技能主题：人物 16／物品 30／战斗 13／世界 66／交互技能 23，共 148 篇组件；加 3 篇 modinfo、5 篇 cookbook，合计 156 篇知识页、12 篇导航及 22 条 Top 30。

798 个组件候选中 148 已建、650 未建，另 19 replica／3 helper／1 empty 单列。五项均来自原扩展池，现余扩展 415／专项 232／职责待确认 3；历史路线快照未改写。交互技能索引 51 行／4,512 字节，组件入口 25 行／1,810 字节，均未超限，正文地址不变。

同步覆盖 CSV、主题与组件导航、PLAN、README、总体设计和 docs 入口，并追加一次日志；历史报告／日志保留。按本次要求完成五批后收尾，下次第 52 批候选 book／reader，先核验实际读写、动作／SG 与消费／撤销，再确定页面和案例；heatrock Prefab、配方 API 与 cookbook 原队列保留。

本次临时 Lua 已清理；未安装依赖、编译解析器、运行游戏或独立模型评测、提交或推送。五页均保持 source-reviewed，不升级为 runtime-tested。

收尾复核：五页及两张导航的格式／来源／本地链接／大小与全库登记通过；五次完整章节读取再次通过，输出字节与表中一致。五份工程 Markdown 的 133 处本地链接／锚点及 git diff --check 通过；日志追加后为 220 行／32,789 字节，未触发归档，第 47～51 批临时 Lua 为零。
