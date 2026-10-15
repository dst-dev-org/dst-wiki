---
title: 显示与表现组件索引
created: 2026-10-28
updated: 2026-10-29
type: index
tags: [overview]
---

# 显示与表现

HUD 指示、地图显示、特效挂载、缩放朝向存读与实体视觉表现类组件；引擎侧 Light／动画／UI API 缺口另行记录。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## HUD 指示与离屏标记

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/hudindicatable]] | 实体注册进客户端 HUD 指示器管理器，shouldtrackfn 过滤，仅非专用端 | HUD 指示器注册、离屏目标指示 / active |
| [[reference/components/hudindicatablemanager]] | 非专用端世界 HUD 指示物登记表：items 只写不读 | HUD 指示物登记表 / active |
| [[reference/components/playertargetindicator]] | 屏外玩家 HUD 标记；全库 grep 无挂载点 | 玩家目标指示器 / active |
| [[reference/components/hudindicatorwatcher]] | 本地玩家屏外目标 HUD 标记扫描增删 | HUD 标记监视器 / active |


## 地图图标与揭示

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/discoverable]] | 小地图图标点击后发现切换，原版无消费者 | 地图发现 / active |
| [[reference/components/globaltrackingicon]] | 单玩家全图追踪图标：globalicon＋revealable 双图标 | 全局追踪图标 / active |
| [[reference/components/maprevealer]] | 周期揭示周边地图给全体/私人拥有者，tag 供 maprevealable 扫描 | 地图揭示器 / active |
| [[reference/components/maprevealable]] | 全图图标挂载：公共/私人揭示来源、globalmapicon 跟随 | 小地图标记 / active |

## 颜色、光照与视觉状态

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/colouraddersync]] | RGBA 附加色打包 net_uint 跨端同步，双端回调应用 | 附加颜色同步 / active |
| [[reference/components/highlightchild]] | FX 子实体登记宿主高亮列表，属主可网络同步 | 子实体高亮 / active |
| [[reference/components/colourtweener]] | RGBA 线性补间 SetMultColour，带回调用 | 颜色补间 / active |
| [[reference/components/colouradder]] | 按来源堆叠加法色求和应用于 AnimState，可带子实体联动 | 附加色堆叠 / active |
| [[reference/components/highlight]] | 悬停/攻击前动态加的高亮加色与闪光，用后自拆 | 悬停高亮 / active |
| [[reference/components/bloomer]] | 泛光优先级堆叠：栈顶生效、子实体联动、来源移除自动弹 | 泛光堆叠 / active |
| [[reference/components/oceancolor]] | 客户端按相位渐变海面清屏色与纹理混合 | 海面颜色 / active |
| [[reference/components/playervision]] | 玩家视觉开关与 CC 表优先级：装备标签、强制栈、梦魇区域联动 | 玩家视觉 / active |
| [[reference/components/ambientlighting]] | 双通道环境光插值：相位/满月/洞穴选色、夜视视觉通道 | 环境光 / active |
| [[reference/components/colourcube]] | 三通道 colour cube 后处理混合与鱼眼/月狂强度 | 后处理调色 / active |
| [[reference/components/transparentonsanity]] | 客户端按理智渐隐宿主并同步压音量 | 理智透明 / active |
| [[reference/components/transparentondrones]] | 客户端专用：多组无人机重叠时淡出他人无人机 | 无人机淡出 / active |

## 渐变、缩放与相机

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/fader]] | 通用回调式数值渐变补间，可级联与挂结束回调 | 数值渐变 / active |
| [[reference/components/sizetweener]] | 缩放等比补间，结束派发事件并回调 | 缩放渐变 / active |
| [[reference/components/spawnfader]] | 生成淡入/淡出：net 同步渐隐值驱动 OverrideMultColour 与收尾事件 | 生成淡入 / active |
| [[reference/components/lighttweener]] | 灯光半径/强度/衰减/颜色时长补间，结束回调 | 灯光补间 / active |
| [[reference/components/camerafade]] | 按镜头距离/中心符号/高度逐帧计算 alpha 淡化实体 | 镜头淡化 / active |
| [[reference/components/focalpoint]] | 客户端相机焦点 | 相机焦点 / active |
| [[reference/components/spectatorcorpse]] | 尸体观战相机：CorpseCam 焦点吸附近距存活玩家，双端组件 | 尸体观战 / active |
| [[reference/components/uianim]] | UI 控件四类 outCubic 补间（移动/缩放/旋转/着色），Widget 自动挂载 | UI 动画 / active |

## 粒子与环境特效

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/groundshadowhandler]] | 抛射物贴地影子 FX，随高度自动缩放 | 贴地影子 / active |
| [[reference/components/emitter]] | 面积粒子发射：配 area_emitter 与密度后一次 Emit 交 EmitterManager 帧驱动 | 粒子发射器 / active |
| [[reference/components/moonstormlightningmanager]] | 月风暴客户端落雷：相机距离调密度，纯表现无伤害 | 月风暴闪电管理器 / active |
| [[reference/components/projectedeffects]] | 侵蚀着色器浮现/消散过渡（全息投影渐隐） | 投影特效 / active |
| [[reference/components/freezefx]] | 多档火焰式光效变体（类名 FireFX）；原版零消费者（死组件） | 火焰光效变体 / active |
| [[reference/components/firefx]] | 按等级表驱动火焰动画/灯光/音效的表现组件 | 火焰特效 / active |
| [[reference/components/floater]] | 落水漂浮表现：前后水面特效＋bank 切换，服务端附加湿/降温 | 漂浮 / active |

## 音频与台词演出

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/nis]] | 过场演出执行器：PlayNIS 入口＋脚本表协程，六键跳过；当前无调用者 | 过场演出 / active |
| [[reference/components/lavaarenamusic]] | 熔炉世界 net 实体 BGM，netvar 关卡号驱动 | 熔炉音乐、竞技场 BGM / active |
| [[reference/components/quagmire_music]] | 大亨世界 net BGM，按 level netvar 换曲 | 大亨音乐 / active |
| [[reference/components/vaultmusiclistener]] | 洞穴玩家宝库音乐：区块 ID＋守卫仇恨推 triggeredevent | 宝库音乐监听 / active |
| [[reference/components/playerhearing]] | 玩家闷音滤波：装备标签/外部开关推 push/popdsp | 闷音滤波 / active |
| [[reference/components/npc_talker]] | NPC 台词队列：Say/Chatter 入队，宿主驱动 DoNextLine 播出 | NPC 说话队列 / active |
| [[reference/components/maxwelltalker]] | 剧本化台词过场演出组件，DST 无挂载点的休眠遗留代码 | 麦克斯韦台词演出 / active |
| [[reference/components/grottowaterfallsoundcontroller]] | 洞穴瀑布声发射器调度，定位最近 3 水池淡入淡出 | 洞穴瀑布声控制器 / active |
| [[reference/components/dsp]] | 客户端混音器滤镜栈：季节/温度基线＋pushdsp/popdsp 叠加 | 音频滤镜、dsp / active |
| [[reference/components/dynamicmusic]] | 监听玩家事件在 busy/danger/pirates 三通道切音乐并管优先级 | 动态音乐 / active |

## 活动状态显示

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/lavaarenaeventstate]] | 熔炉世界 net 比赛状态，客户端只读轮次/进度 JSON | 熔炉比赛状态 / active |
| [[reference/components/quagmire_hangriness]] | 暴食 Gnaw 饥饿值：阈值加速、渴望对错、HUD net 同步 | Gnaw 饥饿度 / active |

返回 [[reference/components/index|组件主题导航]]。
