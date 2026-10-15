---
title: "slipperyfeet"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "在角色跑动或旋转时累计冰面打滑量，按来源、状态标签与防滑砂判定是否发出 feetslipped；跌倒动作交给状态机。"
aliases: ["slipperyfeet", "角色冰面打滑", "跑步滑倒", "打滑累计", "feetslipped"]
tags: [component, lifecycle, state-machine, event, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/slipperyfeet.lua", "components/slipperyfeettarget.lua", "components/areaaware.lua", "util/sourcemodifierlist.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "prefabs/wx78_possessedbody.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwx78_common.lua", "stategraphs/SGwx78_possessedbody.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# slipperyfeet

## 快速判断

- 职责与适用：挂在会打滑的角色上；组合海冰瓦片与 [[reference/components/slipperyfeettarget]] 冰面实体来源，在 `running`／`spinning` 且无 `noslip` 的 SG 状态累计滑度。[来源接口](../../../raw/current/scripts/components/slipperyfeet.lua#L92-L101) [状态门禁](../../../raw/current/scripts/components/slipperyfeet.lua#L117-L125)
- 优先使用：造滑面配置 `slipperyfeettarget`；临时打滑效果才对已有宿主配对调用 `StartSlipperySource(src, key)`／`StopSlipperySource(src, key)`。随身防滑复用 [[reference/components/nonslipgritsource]]，不把滑度当移动倍率。[来源实现](../../../raw/current/scripts/components/slipperyfeet.lua#L92-L101) [防滑分支](../../../raw/current/scripts/components/slipperyfeet.lua#L234-L255)
- 重要副作用：`SetCurrent(val)` 在 `val >= threshold` 时立即给宿主发 `feetslipped`，组件本身不切状态、不播放动画；`SGwilson` 的事件处理进入 `slip`，入场停止移动并清零滑度。[阈值事件](../../../raw/current/scripts/components/slipperyfeet.lua#L149-L158) [事件消费者](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2062-L2064) [slip 入场](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24287-L24304)
- 修改边界：原版玩家在服务端构造添加；海冰进出还依赖 `areaaware` 的瓦片事件，自定义生物必须自行提供有效 SG、Transform、Physics 与事件处理，单加组件不产生完整跌倒效果。[玩家端界](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2614) [添加](../../../raw/current/scripts/prefabs/player_common.lua#L2827) [瓦片订阅](../../../raw/current/scripts/prefabs/player_common.lua#L2664-L2666)

## 关键接口

| 接口／字段 | 参数或初值 | 条件、作用与证据 |
|---|---|---|
| `StartSlipperySource(src, key?)` | 来源对象或非空名称；可选 key | 布尔来源表登记并启动 `newstate` 监听；多来源为 OR，不叠加强度。[slipperyfeet.lua:92-94](../../../raw/current/scripts/components/slipperyfeet.lua#L92-L94) [布尔合并](../../../raw/current/scripts/util/sourcemodifierlist.lua#L33-L34) |
| `StopSlipperySource(src, key?)` | 原来源与 key；省 key 删除该来源全部键 | 最后一个来源移除才停止状态监听／累计；已有滑度继续衰减，不立即归零。[slipperyfeet.lua:97-101](../../../raw/current/scripts/components/slipperyfeet.lua#L97-L101) [停止累计](../../../raw/current/scripts/components/slipperyfeet.lua#L129-L145) [键语义](../../../raw/current/scripts/util/sourcemodifierlist.lua#L103-L120) |
| `SetCurrent(val)` | 数值；不钳制负数 | 直接写 `slippiness`；正值启动 decay 更新，达阈值立刻发事件；非正值停止 decay。[slipperyfeet.lua:149-158](../../../raw/current/scripts/components/slipperyfeet.lua#L149-L158) |
| `DoDelta(delta)` | 正数增加／非正数减少 | 增加会重置衰减速度；减少会钳制到 0；都经过 `SetCurrent` 的事件判定。[slipperyfeet.lua:162-167](../../../raw/current/scripts/components/slipperyfeet.lua#L162-L167) |
| `threshold`／`decay_accel` | `WILSON_RUN_SPEED * 4`／`WILSON_RUN_SPEED * 2` | 构造初值；阈值单位是累计滑度，不是秒数或速度倍率；衰减加速度使停止跑动后逐渐清零。[初值](../../../raw/current/scripts/components/slipperyfeet.lua#L53-L62) [衰减](../../../raw/current/scripts/components/slipperyfeet.lua#L201-L205) |
| `CalcAccumulatingSpeed()` | 读取宿主 Physics、SG | 普通为 `motor_speed² / WILSON_RUN_SPEED`；旋转有 WX 专用修正，再由更新乘秒数 `dt`、冰面倍率和 `0.7～1.0` 随机系数。[速度曲线](../../../raw/current/scripts/components/slipperyfeet.lua#L171-L176) [实际累计](../../../raw/current/scripts/components/slipperyfeet.lua#L254) |
| `GetSlipperyAndNearbyEnts()` | 返回命中冰面、任意近邻 | 半径 12 查 `slipperyfeettarget` 标签，逐一调用几何判定；首个命中的目标提供倍率，多个冰面不相乘。[查询](../../../raw/current/scripts/components/slipperyfeet.lua#L104-L114) [倍率](../../../raw/current/scripts/components/slipperyfeet.lua#L219-L230) |

### 来源累计与防滑顺序

1. 构造安排零延时初始化；读档若任务未执行就取消并立即初始化，保证在恢复位置后检查海冰。初始化订阅宿主 `on_OCEAN_ICE_tile`；已在海冰上直接启用 `ocean_ice`，否则安排 1 秒近邻慢查。[初始化](../../../raw/current/scripts/components/slipperyfeet.lua#L24-L32) [构造与读档](../../../raw/current/scripts/components/slipperyfeet.lua#L64-L74)
2. 慢查发现冰面或附近候选后切每帧精查 `checkiceentity`；进入实际几何范围才启用 `ice_entity`，离开范围撤销来源，附近无候选才退出精查。任意更新原因启动时会取消慢查，全部原因停止后重新安排慢查。[慢查](../../../raw/current/scripts/components/slipperyfeet.lua#L35-L50) [调度](../../../raw/current/scripts/components/slipperyfeet.lua#L179-L196) [范围撤销](../../../raw/current/scripts/components/slipperyfeet.lua#L219-L230)
3. `ocean_ice` 收到离开瓦片事件后不立即失效：先每帧检查海洋瓦片与视觉陆地重叠，保留冰沿悬伸区域，直到不满足该条件才撤销。[海冰事件](../../../raw/current/scripts/components/slipperyfeet.lua#L7-L20) [冰沿检查](../../../raw/current/scripts/components/slipperyfeet.lua#L209-L215)
4. 有来源时由 `newstate` 判定是否累计；先查地面防滑池，再查 `nonslipgrituser`。有池或 user 都跳过新增滑度并衰减旧值；只有 user 且不在池上才把 `dt` 转交砂物品消耗。实现与消耗限制见 [[reference/components/nonslipgrituser]]。[slipperyfeet.lua:234-257](../../../raw/current/scripts/components/slipperyfeet.lua#L234-L257)

## 典型接入

### 给已有玩家增加一种临时打滑来源

- 在目标模组的服务端效果开始／结束回调中取得玩家当前的 `components.slipperyfeet`，使用独有来源名和效果 key 配对增删；来源只允许累计，实际累计仍受跑动／旋转与 `noslip` 门禁，不能保证一启用就跌倒。[来源接口](../../../raw/current/scripts/components/slipperyfeet.lua#L92-L101) [状态判定](../../../raw/current/scripts/components/slipperyfeet.lua#L117-L125)
- 原版玩家已经有组件，通常不重装；死亡变幽灵移除、复活重新添加，所以不要跨此过程缓存旧组件或假定临时来源保存。组件没有 `OnSave`，复活／读档后的模组效果要由自己的持久状态重新接线。[玩家注册](../../../raw/current/scripts/prefabs/player_common.lua#L2827) [幽灵移除](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L642) [复活添加](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L298) [组件读档](../../../raw/current/scripts/components/slipperyfeet.lua#L71-L74)
- 事件响应边界：现成 `SGwilson` 仍只在 `running` 且无 `noslip` 时消费全局事件；直接给站立者 `SetCurrent(threshold)` 不等于会播放摔倒动画。WX 旋转另由状态内事件处理转入 `slip`。[玩家门禁](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2062-L2064) [WX 旋转分支](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L813-L819)

### 原版非普通玩家宿主：WX 附身躯体

`wx78_possessedbody` 服务端构造设置专用 SG，添加 `areaaware` 并订阅海冰瓦片，再添加 `slipperyfeet`。其 SG 消费 `feetslipped`，`slip` 入场同样停移动、清滑度。这是自定义生物接入时的协作参照，不能仅复制 `AddComponent` 一行。[服务端界](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L567-L570) [SG](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L634) [瓦片观察](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L646-L647) [组件](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L661) [事件](../../../raw/current/scripts/stategraphs/SGwx78_possessedbody.lua#L635-L637) [入场](../../../raw/current/scripts/stategraphs/SGwx78_possessedbody.lua#L3373-L3381)

## 常见陷阱与限制

- **把 `noslip` 当实体标签：** 判定读的是 SG state tag；原版骑乘、搬重物、变身与谨慎行走在跑动配置中添加它。给实体 `AddTag("noslip")` 不命中本组件门禁；动态改状态标签还须注意累计标志只在 `newstate` 时重算。[门禁](../../../raw/current/scripts/components/slipperyfeet.lua#L117-L118) [骑乘／重物／变身](../../../raw/current/scripts/stategraphs/SGwilson.lua#L445-L475) [谨慎行走](../../../raw/current/scripts/stategraphs/SGwilson.lua#L500-L502)
- **实体来源自动清理不等于停止累计：** `SourceModifierList` 在来源实体移除时只重算布尔值，本组件构造没有 dirty callback；这条路径不会调用 `StopSlipperySource`／`Stop_Internal`。临时效果结束须显式配对撤销，不能仅靠来源实体销毁，避免剩余 `started`／accumulate 状态（源码推导，未运行）。[来源表创建](../../../raw/current/scripts/components/slipperyfeet.lua#L53-L56) [自动移除](../../../raw/current/scripts/util/sourcemodifierlist.lua#L84-L91) [显式停止](../../../raw/current/scripts/components/slipperyfeet.lua#L97-L101)
- **阈值不是一次性开关：** 每次 `SetCurrent` 写入达到阈值的正数都会发事件，无独立冷却；原版 SG 入场负责清零。自定义消费者若不清零，后续累计可反复触发。[事件](../../../raw/current/scripts/components/slipperyfeet.lua#L149-L155) [清零](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24295-L24296)
- **没有瓦片观察只检测初始海冰：** 组件自己监听事件但不添加 `areaaware`／订阅 tile；`areaaware:UpdatePosition` 只对已观察瓦片发事件。自定义宿主需仿照上面的 WX 接线，不依赖自身每帧重新读全部瓦片。[事件订阅](../../../raw/current/scripts/components/slipperyfeet.lua#L24-L31) [事件生产](../../../raw/current/scripts/components/areaaware.lua#L62-L67) [观察接口](../../../raw/current/scripts/components/areaaware.lua#L113-L117)
- **移除边界：** 移除组件取消初始化／慢查任务、瓦片与状态监听；框架先停止组件更新。本组件不清理 `nonslipgrituser`，也没有保存滑度或临时来源的 `OnSave`。[组件清理](../../../raw/current/scripts/components/slipperyfeet.lua#L78-L89) [框架停止更新](../../../raw/current/scripts/entityscript.lua#L670-L679)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 10 处（SGwilson／player_common／SGwx78_possessedbody 行号迁移，区间文本逐段比对一致）；结论修订：无，仅锚点迁移。
- 依据：全局枢纽 hunk 地图（R-08）＋变更清单命中 stategraphs/SGwilson.lua、prefabs/player_common.lua、prefabs/player_common_extensions.lua、stategraphs/SGwx78_possessedbody.lua、entityscript.lua。

- 源码审阅：2026-10-23，Steam buildid 25643519；完整核验 `slipperyfeet`，定向核验来源表、滑面契约、areaaware 事件生产、玩家幽灵／复活、WX 附身宿主与三个 SG 的事件／清零链；相关证据见上文。未把防滑页的全部历史结论作为本页新核验范围。
- 结构检查：本页 `validate_page.py` 字段、大小与本地链接检查通过；`validate_source_links.py --strict` 源码锚点检查通过。本页无完整 Lua 示例，无语法检查项。
- 游戏运行：未运行；临时来源实体销毁后的残留累计、冰沿边界、不同网络延迟与 WX 旋转效果待实测；不代表全组件或所有宿主已覆盖。

- 2026-10-28 build 26230496：SGwilson ConfigureRunState 加 flyingbroom 判定与 inventory 局部变量，noslip 分支区间微移；判定逻辑未变。
相关：[[reference/components/slipperyfeettarget]]；[[reference/components/nonslipgritsource]]；[[reference/components/nonslipgrituser]]；[[reference/components/locomotor]]。
