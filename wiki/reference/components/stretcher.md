---
title: "stretcher：朝目标拉伸动画图像"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "按宿主到目标的三维距离设置 AnimState 两轴缩放并朝向目标，原版用于阴影手臂视觉。"
aliases: [stretcher, 朝目标拉伸, 阴影手臂伸缩, 动画长度缩放]
tags: [component, lifecycle, prefab, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/stretcher.lua, prefabs/shadowhand.lua, prefabs/charlie_hand.lua]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# stretcher：朝目标拉伸动画图像

## 快速判断

- 职责与适用：根据两实体坐标差调整宿主朝向和 `AnimState:SetScale`；没有移动宿主位置、拖动目标或约束 Physics 的逻辑，不是担架、绳索或物理关节。[stretcher.lua:38-52](../../../raw/current/scripts/components/stretcher.lua#L38-L52)
- 优先使用：参考 shadowhand_arm 把固定起点的手臂图像拉到移动手部；原版设置静止长度4.75、宽度变化比例0.35，不能把数值照搬到不同素材。[shadowhand.lua:159-173](../../../raw/current/scripts/prefabs/shadowhand.lua#L159-L173)、[shadowhand.lua:248-250](../../../raw/current/scripts/prefabs/shadowhand.lua#L248-L250)
- 重要副作用：设置非空目标立即 `OnUpdate(0)`，当场改变朝向与图像缩放；清目标、目标失效或休眠只停止更新，不恢复此前缩放／朝向。[stretcher.lua:16-52](../../../raw/current/scripts/components/stretcher.lua#L16-L52)
- 修改边界：原版手臂在服务端添加组件，宿主须有 Transform、AnimState，目标须提供有效性和坐标接口。组件没有存读、目标移除监听或自定义移除清理钩子。[shadowhand.lua:219-252](../../../raw/current/scripts/prefabs/shadowhand.lua#L219-L252)、[stretcher.lua:1-54](../../../raw/current/scripts/components/stretcher.lua#L1-L54)

## 关键接口

| 接口／字段 | 参数、默认值 | 条件、副作用与证据 |
|---|---|---|
| `SetRestingLength(length)`、`restinglength` | 初始1，距离口径 | 仅赋值，不校验正值；更新时用真实距离除以它得到第一个图像缩放分量。[stretcher.lua:4-10](../../../raw/current/scripts/components/stretcher.lua#L4-L10)、[stretcher.lua:49-51](../../../raw/current/scripts/components/stretcher.lua#L49-L51) |
| `SetWidthRatio(ratio)`、`widthratio` | 初始1，无量纲 | 第二个缩放分量为 `1 + ratio * (scale - 1)`；0保持该分量为1，1则与长度分量一致，无 clamp。[stretcher.lua:5-14](../../../raw/current/scripts/components/stretcher.lua#L5-L14)、[stretcher.lua:49-51](../../../raw/current/scripts/components/stretcher.lua#L49-L51) |
| `SetStretchTarget(target)`、`target` | 实体或 nil；初始 nil | 非空时注册组件更新并立即更新一次；nil 时停更。不预先 IsValid，但立即 OnUpdate 会检查并清除失效目标。[stretcher.lua:16-24](../../../raw/current/scripts/components/stretcher.lua#L16-L24)、[stretcher.lua:38-42](../../../raw/current/scripts/components/stretcher.lua#L38-L42) |
| `OnEntitySleep()`／`OnEntityWake()` | 宿主休眠／唤醒钩子 | 休眠停更但保留目标；唤醒按目标是否非空重启或停止，目标有效性由之后的 OnUpdate 检查。[stretcher.lua:26-42](../../../raw/current/scripts/components/stretcher.lua#L26-L42) |

## 关键流程

更新先检查目标是否存在且有效，失败就 SetStretchTarget(nil) 返回；否则取双方世界坐标构造 Vector3，使用差向量 `Length()` 除以 restinglength，并调用 FacePoint(targetpos) 与 AnimState:SetScale。长度包含 Y 差，不仅是地面 XZ 距离；dt 没有参与计算，因此没有平滑插值或弹簧过渡。[stretcher.lua:38-52](../../../raw/current/scripts/components/stretcher.lua#L38-L52)

两个配置 setter 不立即重算，改变后通常等下一轮更新；SetStretchTarget 则立即重算。组件不保存旧缩放，停用后若继续复用宿主，需由接入方恢复其应有的 AnimState 缩放与朝向；不能依赖清目标自动复原。[stretcher.lua:8-24](../../../raw/current/scripts/components/stretcher.lua#L8-L24)、[stretcher.lua:38-52](../../../raw/current/scripts/components/stretcher.lua#L38-L52)

## 典型接入

### 阴影手臂与查理手部：运动端和视觉起点分开

`shadowhand.lua` 的 MakeShadowArm 注册普通／遮蔽两种 arm prefab，共用地面动画和服务端 stretcher 配置，手臂 `persists=false`。手部 SetTargetFire 在自身起始位置创建对应手臂，然后把手臂的拉伸目标设为手部自身；追火和移动由手部完成。手部 OnRemoveEntity 另行删除 arm，失效停更不能替代这一步。[shadowhand.lua:159-173](../../../raw/current/scripts/prefabs/shadowhand.lua#L159-L173)、[shadowhand.lua:203-209](../../../raw/current/scripts/prefabs/shadowhand.lua#L203-L209)、[shadowhand.lua:215-260](../../../raw/current/scripts/prefabs/shadowhand.lua#L215-L260)、[shadowhand.lua:327-328](../../../raw/current/scripts/prefabs/shadowhand.lua#L327-L328)、[shadowhand.lua:346-349](../../../raw/current/scripts/prefabs/shadowhand.lua#L346-L349)

查理手部同样通过 SpawnShadowArm 创建 `shadowhand_arm`，设置起点和朝向，再 SetStretchTarget(inst)。它的 OnLoadPostPass 从保存的 origin 与关联 atrium 重建手臂，而非恢复 stretcher 内部状态；OnRemoveEntity 也负责删手臂。这是宿主重建与清理协议，组件自身不存目标引用或缩放历史。[charlie_hand.lua:118-128](../../../raw/current/scripts/prefabs/charlie_hand.lua#L118-L128)、[charlie_hand.lua:153-179](../../../raw/current/scripts/prefabs/charlie_hand.lua#L153-L179)、[charlie_hand.lua:181-185](../../../raw/current/scripts/prefabs/charlie_hand.lua#L181-L185)、[charlie_hand.lua:251-252](../../../raw/current/scripts/prefabs/charlie_hand.lua#L251-L252)

## 常见陷阱与限制

- **静止长度0或比例任意都安全**：公式没有零分母或负缩放保护；保持 restinglength>0，并根据素材和距离范围约束 widthratio。目标与宿主重合时长度缩放会为0，不保证图像维持最小长度。[stretcher.lua:8-14](../../../raw/current/scripts/components/stretcher.lua#L8-L14)、[stretcher.lua:44-51](../../../raw/current/scripts/components/stretcher.lua#L44-L51)
- **失效即还原或删除**：OnUpdate 只清目标和停更，没有重置 AnimState、删除宿主或解除其他宿主关系。长期视觉实体的恢复与生命周期由 prefab 负责。[stretcher.lua:16-24](../../../raw/current/scripts/components/stretcher.lua#L16-L24)、[stretcher.lua:38-52](../../../raw/current/scripts/components/stretcher.lua#L38-L52)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、普通／遮蔽阴影手臂工厂和手部接线／移除，以及查理手部建臂、读档重建与删除；未扩展到其他视觉宿主。
- 游戏运行：未运行；未实测高低差、零长度、负缩放、休眠与目标移除时序，未核验 AnimState 缩放的网络表现及完整追火／查理流程；无新增 Lua 示例。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；未运行语法探针。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。