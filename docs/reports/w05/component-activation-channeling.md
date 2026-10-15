# 激活、定向使用与建筑持续交互组件检视

2026-10-19 完成第 44～46 批源码审阅、页面生成与逐批检查，2026-10-20 续办工程收尾。依据 Steam buildid 25643519；沿用 dst-wiki Skill，游戏源码与输入模组只读，文本及临时 Lua 使用 apply_patch 写入和清理。

## 批次与测时

工具 UTC 时钟转为 Asia/Shanghai，以下均为 2026-10-19；每批到页面／主题登记／覆盖清单、适用检查和临时 Lua 清理完成。相关宿主可提前定位、跨批复用，不是三个独立冷启动样本。

| 批次 | 页面 | 开始—结束（北京时间） | 批内墙钟 | 源码引用数 |
|---|---|---|---|---:|
| 44 | activatable | 23:42:46—23:45:11 | 2 分 25 秒 | 22 |
| 45 | useabletargeteditem | 23:45:11—23:46:54 | 1 分 43 秒 | 28 |
| 46 | channelable | 23:46:54—23:49:58 | 3 分 04 秒 | 32 |

三批核心审阅／生成／检查共 7 分 12 秒。工程文档／总报告／日志和最终核对跨日续办，未记录为连续工作时长，不把中间等待算作生成耗时。时间、页面数与输出字节不证明 token 收益或游戏正确性。

## 交付与边界

| 页面 | 实际行数／UTF-8 字节 | 完整接入案例 |
|---|---|---|
| [activatable](../../../wiki/reference/components/activatable.md) | 74／6,792 | 原木栅栏门拒绝经 CanActivate 发起的非玩家激活，保留宿主开关与旧测试 |
| [useabletargeteditem](../../../wiki/reference/components/useabletargeteditem.md) | 77／8,059 | 原小猫命名牌 onuse 增加服务器目标／玩家检查，保留客户端候选与命名流程 |
| [channelable](../../../wiki/reference/components/channelable.md) | 76／9,187 | 原沙漠传送阵服务端中止事件，保留 SG、停止回调与精神修正撤销 |

三页合计 24,038 字节，均在页面上限内；采用五节结构，生命周期协作归入案例下的三级小节，未新增类型、必填章节或空导航。

- activatable 的 CanActivateFn 覆盖默认 inactive 判断，DoActivate 不再自检资格。执行先清 inactive，回调真值才发 onactivated；原 ACTIVATE 却将 nil 返回视为成功。宿主恢复、动作成功与事件通知不能混同，示例不宣称防止直接绕过调用。
- useabletargeteditem 的客户端 ValidTarget 与 Prefab 标签为 OR，服务端 StartUsingItem 不执行这套筛选，也不自动 CanInteract。开始回调后才写使用中，停止回调只收到物品；绑定引用、命名 UI 与监听清理归宿主。移除组件不是完整撤销，实际自目标标签拼写保留原文。
- channelable 的 SetEnabled 只改准入，SG 中止和宿主停止分别处理。停止回调分支还承担引用／集合清除；开始回调的 false 返回不构成拒绝。多人模式 setter 的初始 nil 表与先改模式后停止顺序有边界，不能当安全热迁移；示例只用于原单人 townportal。

未全面审校 RPC／权限、所有机关宿主、命名编辑器、全部服务端／客户端 SG、传送配对、多人迁移或热移除；没有修改游戏源码、Skill、工具箱或正式类型数据。

## 校验与受控读取

逐批页面格式／来源／本地链接／锚点／大小和全库导航登记通过。三份完整示例与临时 Lua 逐字核对，已有 Full Moon 执行 Lua 5.1 syntax-only 后清理；未执行示例或运行游戏。

三页合并 strict 源码检查：82 处引用、64 个唯一目标、12 份来源文件，0 error／0 warning，truncated=false、omitted=0、by_page_omitted=0。覆盖 CSV 与组件正文集合一致，143 个已建记录的原字节 SHA-256 已核对；结构／位置检查不代替语义审阅。

| 页面 | “典型接入”JSON stdout 实际 UTF-8 字节 |
|---|---:|
| activatable | 4,505 |
| useabletargeteditem | 4,913 |
| channelable | 6,127 |

逐批按已知页选择章节，均返回元数据、快速判断、完整案例／条件及验证记录；complete_selected_ranges=true、source_version=match、truncated=false、omitted=0。字节按 LF stdout 记录，不换算模型 token，不称独立模型自主评测；未重建缓存。

## 收尾与下次入口

本轮三条唯一登记均在交互技能主题：人物 16／物品 30／战斗 13／世界 66／交互技能 18，共 143 篇组件。加 3 篇 modinfo、5 篇 cookbook，合计 151 篇知识页、12 篇导航、22 条 Top 30。

798 候选中 143 已建／655 未建，另 19 replica／3 helper／1 empty 单列；本轮三项来自扩展池，现剩扩展 420／专项 232／职责待确认 3。交互技能主题 46 行／3,535 字节，组件入口 25 行／1,753 字节；未超限，未改正文地址或导航层级。

同步覆盖 CSV、主题索引、组件入口、PLAN、README、设计和 docs 入口，并追加日志；旧报告与历史日志不改写。按用户要求完成三批后在第 46 批收尾；下次候选为 channelcastable／channelcaster，先核验物品与玩家端协作，不将名称相近视为同一套建筑交互。heatrock Prefab、配方注册 API 和 cookbook 原队列保留。

本轮临时 Lua 已清理；未安装依赖、构建解析器、运行独立模型评测、提交或推送。三页保持 source-reviewed，不将格式／语法／受控读取通过升级为 runtime-tested。

2026-10-20 收尾复核：五份 Wiki 文件的格式／来源／本地链接／大小及全库登记通过；三页 82 处源码引用 strict、三次完整章节读取和全部 143 个已建记录指纹再次通过。五份工程 Markdown 的 129 处本地链接／锚点、git diff --check 通过；日志 213 行／31,762 字节，未触发归档，本轮临时 Lua 为零。
