---
title: "wax"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "把物品标记为普通蜂蜡或蜡喷雾，接入目标 waxable 的打蜡动作与成功消耗。"
aliases: ["蜂蜡材料", "蜡喷雾", "打蜡工具", "wax"]
tags: [component, item, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/wax.lua", "components/waxable.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "prefabs/beeswax.lua", "prefabs/beeswax_spray.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wax

## 快速判断

- 职责与适用：挂在打蜡材料／工具上，只维护 `is_spray` 与 `waxspray` 标签；实际改造效果由目标的 [[reference/components/waxable]] 回调决定。[完整组件](../../../raw/current/scripts/components/wax.lua#L1-L27)
- 优先使用：普通蜂蜡保留默认 false；喷雾调用 `SetIsSpray()`。原版分别注册为 `beeswax`、`beeswax_spray`，无需自建打蜡动作。[蜂蜡](../../../raw/current/scripts/prefabs/beeswax.lua#L61-L78) [喷雾](../../../raw/current/scripts/prefabs/beeswax_spray.lua#L130-L143)
- 重要副作用：材料类型决定动作可见性及玩家动画；成功后由目标统一扣耐久／拆堆／删除，`wax` 自己不扣次数。[动作采集](../../../raw/current/scripts/componentactions.lua#L1931-L1935) [消耗](../../../raw/current/scripts/components/waxable.lua#L43-L58)
- 修改边界：原版服务端挂组件；普通／喷雾必须与目标 `needswaxspray` 标签严格匹配才出现交互。直接调用 `Wax` 的检查更宽，不能代替动作匹配规则。[蜂蜡端边界](../../../raw/current/scripts/prefabs/beeswax.lua#L53-L61) [喷雾端边界](../../../raw/current/scripts/prefabs/beeswax_spray.lua#L113-L131) [双向区分](../../../raw/current/scripts/componentactions.lua#L2563-L2567)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `is_spray` | 默认 false | 属性回调按真假添加／删除 `waxspray`；不保存次数或目标类型。[定义](../../../raw/current/scripts/components/wax.lua#L1-L17) |
| `SetIsSpray()` | 无参数、无返回 | 固定写 true；传 false 也不会关闭喷雾类型。[实现](../../../raw/current/scripts/components/wax.lua#L19-L21) |
| `GetIsSpray()` | 返回 `is_spray` | 目标 `Wax` 只在其要求喷雾且材料具有 wax 时读取。[getter](../../../raw/current/scripts/components/wax.lua#L23-L25) [消费者](../../../raw/current/scripts/components/waxable.lua#L38-L41) |
| `ACTIONS.WAX` | `target`＝被打蜡对象，`invobject`＝材料 | 执行前处理物品拟态揭露，再转发 `target.components.waxable:Wax(doer, invobject)`。[动作](../../../raw/current/scripts/actions.lua#L5547-L5554) |

## 典型接入

### 选择普通消耗材料还是多次使用的喷雾

以下为原版构造参照，不是完整新物品工程。新物品的加载与资源骨架见 [[cookbook/new-item]]，不要只挂 `wax` 就认为拥有可拾取物品的全部行为。

| 需求 | 原版组装 | 行为与证据 |
|---|---|---|
| 普通蜂蜡，一次用一枚 | `beeswax`：服务端 `wax`＋`inventoryitem`＋`stackable` | 保持默认非喷雾；成功时拆一枚删除。[构造／注册](../../../raw/current/scripts/prefabs/beeswax.lua#L53-L78) [拆堆](../../../raw/current/scripts/components/waxable.lua#L47-L54) |
| 可装备、多次喷涂 | `beeswax_spray`：服务端 `equippable`＋`wax:SetIsSpray()`＋`finiteuses` | 次数上限／初始值均为 `TUNING.BEESWAX_SPRAY_USES`，耗尽删除；装备动画由宿主提供。[装备](../../../raw/current/scripts/prefabs/beeswax_spray.lua#L77-L100) [组装／注册](../../../raw/current/scripts/prefabs/beeswax_spray.lua#L126-L143) |

对新网络 Prefab，保留网络共同构造、`SetPristine()` 与服务端返回边界；原版两种材料都在此门禁后挂 `wax`，喷雾通过属性 setter 添加类型标签。[普通材料](../../../raw/current/scripts/prefabs/beeswax.lua#L38-L61) [喷雾](../../../raw/current/scripts/prefabs/beeswax_spray.lua#L102-L131)

交互发现既支持拿材料作用于目标，也支持装备工具右键目标，两者都要求目标有 `waxable` 且两种喷雾标签相等。双端玩家 SG 按材料 `waxspray` 标签选择 `spray_wax`，否则 `dolongaction`；喷雾服务端在动画第 13 帧执行动作，客户端只预览与等待服务端状态。[采集一](../../../raw/current/scripts/componentactions.lua#L1931-L1935) [采集二](../../../raw/current/scripts/componentactions.lua#L2563-L2567) [服务端选择](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1600-L1605) [客户端选择](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L850-L855) [执行时点](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12374-L12390) [预览](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L2711-L2723)

成功消费与目标效果必须由 [[reference/components/waxable#典型接入]] 协调；要调整喷雾次数，复用 [[reference/components/finiteuses#典型接入]]，不要在材料回调额外扣一次。

## 常见陷阱与限制

- **喷雾并非普通蜂蜡的通用升级：** 菜单要求标签严格相等，所以喷雾不会给普通蜡目标提供 WAX。裸 `Wax` 只拒绝“要求喷雾＋材料有 wax＋非喷雾”，两层门禁不相同。[采集](../../../raw/current/scripts/componentactions.lua#L1931-L1935) [执行](../../../raw/current/scripts/components/waxable.lua#L38-L41)
- **误以为组件自带成功回调或消耗：** 本组件没有 `SetOnWaxFn`、耐久或物品转换；效果在目标 `waxfn`，只有其返回真值才消费。[wax 定义](../../../raw/current/scripts/components/wax.lua#L9-L27) [目标回调](../../../raw/current/scripts/components/waxable.lua#L43-L58)
- **移除组件以为会清掉喷雾标记：** `wax` 没有 `OnRemoveFromEntity`，动态移除不能依赖它清理 `waxspray`；若设计切换材料类型，需同步处理类型字段、标签及动作组件，不只删组件。[完整组件](../../../raw/current/scripts/components/wax.lua#L1-L27)
- **把类型当存档字段：** 组件无 OnSave／OnLoad；原版由每次 Prefab 构造重新确定类型。运行时自定义类型切换的持久化不由本组件提供。[组件](../../../raw/current/scripts/components/wax.lua#L9-L27) [固定喷雾初始化](../../../raw/current/scripts/prefabs/beeswax_spray.lua#L130-L131)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（componentactions.lua、actions.lua、SGwilson.lua、SGwilson_client.lua）对照 build-26148036 更新报告条目 R-01／R-04 逐条核验，引用区域内容均未变化，锚点重定位 10 处，无结论修订；未覆盖：运行验证。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；覆盖完整 wax／waxable、动作采集和执行、玩家双端 WAX 路由及喷雾状态、蜂蜡与喷雾的实际注册／服务端挂载／消费配置。未审校材料全部燃烧、漂浮及皮肤机制。
- 游戏运行：未运行；新材料与目标组合未实测。
- 结构检查：页面格式／链接与源码锚点严格检查通过；本页仅说明原版组装，无新增完整 Lua 示例。
