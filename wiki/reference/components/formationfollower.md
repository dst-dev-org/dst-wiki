---
title: formationfollower
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: 为环形编队成员保存队长与目标坐标，通过宿主回调移动，处理搜索、离队和休眠更新
aliases: [编队成员, 跟随编队位置, formationfollower, 发光飞虫搜索半径, active不停止更新]
tags: [component, ai, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: [components/formationfollower.lua, components/formationleader.lua, prefabs/lightflier.lua, brains/lightflierbrain.lua, prefabs/lightflier_flower.lua]
reliability: verified
validation: source-reviewed
status: active
---

# formationfollower

## 快速判断

- **职责与适用：** 成员侧编队引用与坐标桥接：队长写入 `formationleader`、`in_formation`、`formationpos`，本组件把坐标交给 `onupdatefn`。它不自带移动算法，也不自动周期搜队。[成员更新:93–98](../../../raw/current/scripts/components/formationfollower.lua#L93-L98)、[队长入队:155–172](../../../raw/current/scripts/components/formationleader.lua#L155-L172)
- **优先使用：** 与 [[reference/components/formationleader]] 配套实现旋转环形站位；原版 lightflier 由 Prefab 搜队／驱动，Brain 切换常规行为和编队运动。普通忠诚／主人关系仍由 [[reference/components/follower]] 负责。[Prefab接线:511–518](../../../raw/current/scripts/prefabs/lightflier.lua#L511-L518)、[Brain分派:108–125](../../../raw/current/scripts/brains/lightflierbrain.lua#L108-L125)
- **重要副作用：** 离队会经队长触发回调、清引用、`combat:DropTarget()` 和可选的普通跟随停止；`StopUpdating()` 只停本组件更新，不退出队伍。[显式离队:87–89](../../../raw/current/scripts/components/formationfollower.lua#L87-L89)、[队长清理:177–194](../../../raw/current/scripts/components/formationleader.lua#L177-L194)
- **修改边界：** `active` 默认 false，但 `OnUpdate` 实际不读它；不把 `active = false` 当停用 API。原版在服务端添加组件，客户端没有这一套普通 Lua 成员字段同步。[默认与属性表:23–42](../../../raw/current/scripts/components/formationfollower.lua#L23-L42)、[更新门控:93–98](../../../raw/current/scripts/components/formationfollower.lua#L93-L98)、[主端门控:452–454](../../../raw/current/scripts/prefabs/lightflier.lua#L452-L454)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `formation_type` | 字符串，默认 `"monster"` | 维护自身 `formation_<类型>` 标签，搜索标签为 `formationleader_<类型>`；原版改成 `"lightflier"`。[类型回调:1–9](../../../raw/current/scripts/components/formationfollower.lua#L1-L9)、[原版配置:513–518](../../../raw/current/scripts/prefabs/lightflier.lua#L513-L518) |
| `formationleader`／`in_formation`／`formationpos` | 队长组件或 nil／布尔值／坐标或 nil | `formationleader` 是组件，不是实体；回调第二参才是队长实体。状态由队长管理，不应只改 `in_formation` 伪造入队。[入队:155–169](../../../raw/current/scripts/components/formationleader.lua#L155-L169)、[坐标:199–209](../../../raw/current/scripts/components/formationleader.lua#L199-L209) |
| `searchradius` | 数值，默认 50 | 无覆盖候选时用于搜队；lightflier 覆盖为 8，并在宿主 `FindTarget` 中用它先查候选。[默认:23–31](../../../raw/current/scripts/components/formationfollower.lua#L23-L31)、[宿主搜索:227–231](../../../raw/current/scripts/prefabs/lightflier.lua#L227-L231) |
| `SearchForFormation` | `SearchForFormation(override_find_entities)` → `true` 或 `nil` | 候选列表可省略；选第一个未满队长、调用入队后立即返回 true，未检查入队是否真正完成。[59–66](../../../raw/current/scripts/components/formationfollower.lua#L59-L66) |
| `onupdatefn` | `fn(member_entity, formationpos)` | 更新中有队长且有坐标才调用；不传 `dt`，也不自动发移动动作。lightflier 依据距离设速度并朝目标点前进。[93–98](../../../raw/current/scripts/components/formationfollower.lua#L93-L98)、[原版运动:126–134](../../../raw/current/scripts/prefabs/lightflier.lua#L126-L134) |
| `onenterformationfn`／`onleaveformationfn` | `fn(member_entity, leader_entity)` | 由队长调用，分别先于本次登记／清理；原版处理停步、`NOBLOCK`、速度与离家关系。[入队:155–169](../../../raw/current/scripts/components/formationleader.lua#L155-L169)、[退队:177–194](../../../raw/current/scripts/components/formationleader.lua#L177-L194)、[宿主:143–165](../../../raw/current/scripts/prefabs/lightflier.lua#L143-L165) |
| `StartUpdating`／`StopUpdating` | `()` | 仅启停组件逐帧更新；离队用 `LeaveFormation()`，停止宿主搜队任务是另一件事。[51–56](../../../raw/current/scripts/components/formationfollower.lua#L51-L56)、[87–89](../../../raw/current/scripts/components/formationfollower.lua#L87-L89) |

### 搜队结果的真实含义

普通搜索按类型标签取附近队长，不额外匹配玩家目标或最近距离；传入候选列表时连这层半径／标签过滤也由调用者承担。列表元素需确有 `formationleader`，成员需满足队长资格；不要对任意实体数组直接调用。[搜索:59–66](../../../raw/current/scripts/components/formationfollower.lua#L59-L66)、[资格:110–117](../../../raw/current/scripts/components/formationleader.lua#L110-L117)

`true` 表示执行过一次入队调用，不是检查后的成员状态；需要确定已加入时读取 `formationleader`。例如成员已有队长会被 `ValidMember` 拒绝，但搜索接口本身仍可能返回 true。[返回位置:63–66](../../../raw/current/scripts/components/formationfollower.lua#L63-L66)、[已有队长门控:110–117](../../../raw/current/scripts/components/formationleader.lua#L110-L117)

## 典型接入

### 把发光飞虫加入附近已有编队的搜索半径改为 12

已有模组在 `modmain.lua` 追加下面完整注册片段。注册函数来自模组环境，游戏全局显式用 `GLOBAL`，不要求先安装读取代理；加载基础见 [[cookbook/mod-starter#全局名称与加载环境]]。原版在主端添加组件并初始化为半径 8，`FindTarget` 每次按当前字段取候选，因此这里仅改这项搜索距离，不改队长半径／容量或运动回调。[配置:513–518](../../../raw/current/scripts/prefabs/lightflier.lua#L513-L518)、[消费者:227–231](../../../raw/current/scripts/prefabs/lightflier.lua#L227-L231)

```lua
AddPrefabPostInit("lightflier", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.formationfollower.searchradius = 12
end)
```

- **生效与撤销：** 作用于安装该模组后创建的 lightflier，距离单位为世界坐标；需重载世界／重建实体应用注册改动。移除这段注册后重新创建的实体回到原版默认配置，不承诺热卸载会还原已有实例。
- **范围：** 没找到可加入编队后，另找持有 lightflier 的玩家这一过程仍用 `FIND_TARGET_RADIUS = 9`，未被本例改成 12；周期搜队由 Prefab 的 `DoPeriodicTask` 调度，不是组件字段自行启动。[玩家搜索常量:33–38](../../../raw/current/scripts/prefabs/lightflier.lua#L33-L38)、[分派:220–237](../../../raw/current/scripts/prefabs/lightflier.lua#L220-L237)、[周期任务:248–251](../../../raw/current/scripts/prefabs/lightflier.lua#L248-L251)
- **验证方法：** 游戏中分别比较已有编队在 8～12 距离范围时的加入行为、没有编队时的找玩家行为，检查收纳／丢出、受击后退出。本例只完成源码核验与语法探测，未执行这些游戏场景。

### 保留原版移动回调与 Brain 协作

lightflier 的 `FollowerOnUpdate` 在 Brain 未停止时，按距编队坐标的距离计算步速（上限 10.5），朝向该点，必要时 `WalkForward(true)`；`onenterformationfn` 先停步并加 `NOBLOCK`，`onleaveformationfn` 恢复速度、移除标签，并可能解除远处家园关系。[运动:126–134](../../../raw/current/scripts/prefabs/lightflier.lua#L126-L134)、[回调:143–165](../../../raw/current/scripts/prefabs/lightflier.lua#L143-L165)

Brain 常规分支先写 `active = false`、恢复步速并设 `directdrive = false`；进入编队运动分支才写 `active = true`、`directdrive = true`。这里的 `active` 还被花朵召回流程读取，但组件自身更新并不据此停下，不应直接删掉宿主这些字段便称等价。[Brain起始动作:87–94](../../../raw/current/scripts/brains/lightflierbrain.lua#L87-L94)、[编队分支:108–125](../../../raw/current/scripts/brains/lightflierbrain.lua#L108-L125)、[召回读取:194–206](../../../raw/current/scripts/prefabs/lightflier_flower.lua#L194-L206)

## 生命周期

- 构造时启动更新；没有 `onupdatefn` 则每帧直接返回，不会自行移动或搜索。休眠时先离队再停止更新，唤醒时若仍处于容器中则不启动，否则重启。[构造:23–33](../../../raw/current/scripts/components/formationfollower.lua#L23-L33)、[休眠／唤醒:71–84](../../../raw/current/scripts/components/formationfollower.lua#L71-L84)、[更新:93–98](../../../raw/current/scripts/components/formationfollower.lua#L93-L98)
- 原版收纳时停止宿主搜队任务和本组件更新，丢出时重新启动两者；入 limbo 的离队由队长的事件监听配合完成。单纯调用 `StopUpdating()` 不替代这些宿主操作。[收纳:254–258](../../../raw/current/scripts/prefabs/lightflier.lua#L254-L258)、[丢出:279–284](../../../raw/current/scripts/prefabs/lightflier.lua#L279-L284)、[队长监听:163–169](../../../raw/current/scripts/components/formationleader.lua#L163-L169)
- 离队清空队长引用与 `in_formation`，未清 `formationpos`；旧坐标不代表仍在编队。更新实际要求“队长非 nil 且坐标非 nil”同时满足。[清理:187–194](../../../raw/current/scripts/components/formationleader.lua#L187-L194)、[门控:93–98](../../../raw/current/scripts/components/formationfollower.lua#L93-L98)

## 常见陷阱与限制

- **`active` 并非启停开关。** 本文件虽然定义局部 `onactive`，属性回调表只登记 `formation_type`；`OnUpdate` 的 active 条件也是注释。真正停止用 `StopUpdating()`，需要退出还要 `LeaveFormation()`。[未接线函数:13–19](../../../raw/current/scripts/components/formationfollower.lua#L13-L19)、[属性表:40–42](../../../raw/current/scripts/components/formationfollower.lua#L40-L42)、[实际更新:93–98](../../../raw/current/scripts/components/formationfollower.lua#L93-L98)
- **`leashdistance = 70` 不构成自动脱队阈值。** 文件只初始化该字段，当前更新与搜索不读取它；原版离散距离判断在宿主，不按字段名推导行为。[初始化:23–33](../../../raw/current/scripts/components/formationfollower.lua#L23-L33)、[宿主队长距离检查:46–53](../../../raw/current/scripts/prefabs/lightflier.lua#L46-L53)
- **不要覆盖回调时丢失宿主清理。** 原版离队涉及家园、速度和碰撞标签；若只为调搜索范围，无需替换回调。自定义生物还需核对队长退出时要求的 `combat` 等组件，不能直接套此注册示例。[离队回调:143–159](../../../raw/current/scripts/prefabs/lightflier.lua#L143-L159)、[队长退出:177–194](../../../raw/current/scripts/components/formationleader.lua#L177-L194)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：核验成员组件、配套队长、lightflier Prefab、Brain 与花朵召回读取的所述接口／分支；模组注册环境复用 mod-starter 的已核验说明。覆盖搜队、回调、更新门控与收纳／丢出；未审校整条光照／花朵生长机制。
- 游戏未运行；完整注册片段仅验证语法，不代表已测加入距离、寻路、联机表现或热卸载。宿主与配套队长的其他限制见 [[reference/components/formationleader]]。
