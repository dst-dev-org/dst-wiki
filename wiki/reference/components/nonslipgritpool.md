---
title: "nonslipgritpool"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "定义地面防滑砂池的覆盖几何，供 slipperyfeet 跑动累计时查询；原版强化砂铺出半径 3 的限时防滑区域。"
aliases: ["nonslipgritpool", "防滑池", "地面防滑砂", "防滑区域", "强化砂地面覆盖"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/nonslipgritpool.lua", "components/slipperyfeet.lua", "prefabs/nonslipgrit.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# nonslipgritpool

## 快速判断

- 职责与适用：挂在地面防滑区域实体上，回答某世界坐标是否有砂；不改变瓦片、不主动找玩家，也不修改移动速度。[组件几何](../../../raw/current/scripts/components/nonslipgritpool.lua#L18-L30) [实际消费者](../../../raw/current/scripts/components/slipperyfeet.lua#L234-L254)
- 优先使用：原版 `nonslipgritpool` Prefab 已有半径 3 的几何与到期流程；新防滑实体配置 `SetIsGritAtPoint(fn)`，让已有 [[reference/components/slipperyfeet]] 自动消费。随身砂的入包登记与消耗归 [[reference/components/nonslipgritsource]]／[[reference/components/nonslipgrituser]]，不需要复制到池组件。[原版几何](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L96-L99) [挂接与计时](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L136-L145)
- 重要副作用：跑动累计分支命中任一池就跳过新增滑度，并衰减已有滑度；即使同时带砂也不调 user 消耗。不立刻清零旧滑度，也不撤销正在进行的跌倒状态。[消费顺序](../../../raw/current/scripts/components/slipperyfeet.lua#L234-L254)
- 修改边界：原版组件在服务端添加，公共构造显式加 `nonslipgritpool` 标签。客户端看见池的外观不代表拥有几何组件；带标签但缺组件的实体会被消费者跳过。[标签与端界](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L127-L137) [双重检查](../../../raw/current/scripts/components/slipperyfeet.lua#L237-L240)

## 关键接口

| 接口／字段 | 签名／约定 | 条件、作用与证据 |
|---|---|---|
| 构造 | `NonSlipGritPool(inst)` | 添加 `nonslipgritpool` 标签；源码建议在 Prefab 公共构造阶段提前显式添加。无更新任务或自动寿命。[构造](../../../raw/current/scripts/components/nonslipgritpool.lua#L1-L7) |
| `SetIsGritAtPoint(fn)` | `fn(inst, x, y, z)` | 单槽覆盖，自定义几何返回真值表示覆盖；没有回调时才走默认 Physics 判定。传 nil 可回退默认。[设置](../../../raw/current/scripts/components/nonslipgritpool.lua#L14-L15) [优先级](../../../raw/current/scripts/components/nonslipgritpool.lua#L18-L24) |
| `IsGritAtPosition(x, y, z)` | 世界坐标 | 默认忽略 y，以 Physics 半径和实体世界中心做 XZ 圆内判定 `dx² + dz² < r²`；无 Physics 返回 false。[实现](../../../raw/current/scripts/components/nonslipgritpool.lua#L18-L30) |
| `OnRemoveFromEntity()` | 无参 | 只移除标签；不删除实体、不结束动画／timer。消费者以后不再按标签命中它。[实现](../../../raw/current/scripts/components/nonslipgritpool.lua#L10-L11) |

### 覆盖判定与查询范围

- `slipperyfeet` 仅在已有打滑来源、SG 为 `running`／`spinning` 且无 `noslip` 时启用累计；防滑池检测位于该累计分支。完整来源与 SG 处理复用 [[reference/components/slipperyfeet#来源累计与防滑顺序]]。[累计门禁](../../../raw/current/scripts/components/slipperyfeet.lua#L117-L125) [池消费](../../../raw/current/scripts/components/slipperyfeet.lua#L234-L251)
- 消费者先以宿主坐标、查询半径 12、必需标签 `nonslipgritpool` 取得候选，再调用每个组件的 `IsGritAtPosition`。自定义几何再大也不会扩展这一步查询预算；远离候选中心的覆盖不能仅靠回调保证被发现（源码推导，未运行）。[半径常量](../../../raw/current/scripts/components/slipperyfeet.lua#L4) [候选与精查](../../../raw/current/scripts/components/slipperyfeet.lua#L234-L242)
- 重叠池首个命中便停止查找，结果为布尔防滑，不叠加强度或加速；只在 user 存在且未命中池时才消耗随身砂。[消费分支](../../../raw/current/scripts/components/slipperyfeet.lua#L239-L254)

## 典型接入

### 原版强化砂铺出一个防滑池

1. `nonslipgritboosted` 与普通砂共用物品工厂，但 source 消耗回调选择 `OnDelta_Boosted`；每次回调在砂物品当前世界位置生成池并随机旋转，扣 `100 / NUMBER_OF_POOLS` 燃料。何时回调和多砂消耗问题见 [[reference/components/nonslipgritsource]]，不由池自行推动。[铺池回调](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L22-L29) [选择回调](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L69-L74) [Prefab 注册](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L377-L379)
2. `fn_pool` 创建 Transform、AnimState、SoundEmitter、Network，播放贴地的 `pool_pre → pool_idle`，加 `NOCLICK` 和池标签；服务端添加本组件并覆盖几何为 `distsq(center, point) <= 3 * 3`。原版没有 Physics，若漏掉回调，将永远返回 false。[表现构造](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L111-L130) [几何回调](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L96-L99) [服务端接线](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L132-L141)
3. timer 启动 `dissolve`，时长取 `TUNING.NONSLIPGRITBOOSTED_POOL_TIME`；到期设 `persists = false`、播放声音／`pool_pst`，等 `animover` 才删实体。期间未移除组件或标签，因而不是 timer 到点立刻停止防滑（源码推导，未运行）。[启动计时](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L139-L141) [到期处理](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L87-L92) [时长定义](../../../raw/current/scripts/tuning.lua#L9118)

### 自定义区域应改哪里

在目标防滑 Prefab 的服务端构造中添加组件并设置几何回调，公共构造保留池标签；回调按实体中心或自有形状判断世界坐标。资源、寿命与生成入口仍由 Prefab 管理，组件不提供 `SetRadius`、铺池、燃料或 timer 接口；想沿用原版资源与寿命，可针对已有 `nonslipgritpool` 修改几何，而不是给角色添加这个区域组件。[组件全部几何入口](../../../raw/current/scripts/components/nonslipgritpool.lua#L14-L30) [原版构造分工](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L111-L145)

## 常见陷阱与限制

- **边界差一圈：** 默认 Physics 圆是严格 `< r²`，原版半径 3 回调是 `<= 9`；替换回调时是否包含边缘需要主动决定，不把两者视为同一判定。[默认](../../../raw/current/scripts/components/nonslipgritpool.lua#L23-L27) [原版覆盖](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L96-L99)
- **默认无碰撞体不生效：** 无 Physics 且没回调直接 false；单加标签只有查询入口，单加组件也不会自动获得半径 3。[回退](../../../raw/current/scripts/components/nonslipgritpool.lua#L23-L30) [标签后的组件检查](../../../raw/current/scripts/components/slipperyfeet.lua#L237-L240)
- **防滑不是清除已发生的滑倒：** 池只阻断后续累计并调用 decay，不直接清零，也不处理 SG 的 `slip`；此效果不能当作治疗滑倒或改变海冰瓦片。[消费](../../../raw/current/scripts/components/slipperyfeet.lua#L245-L254)
- **形状不跟美术自动变化：** 原版覆盖回调固定半径 3、只看 XZ 距离，不读取动画尺寸、旋转或 y；缩放图像、旋转图像或叠放高度不会自动改变回调覆盖。[原版几何](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L96-L99) [随机旋转](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L24-L28)
- **组件没有保存或寿命：** 组件完整实现只有标签和几何接口；原版到期、表现与延迟移除属于 Prefab／timer 协作。移除组件只去标签，池图像与 timer 仍可能存在。[移除](../../../raw/current/scripts/components/nonslipgritpool.lua#L10-L11) [原版时序](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L87-L92) [计时装配](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L139-L145)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 1 处（tuning +17，区间文本比对一致）；结论修订：无（NONSLIPGRITBOOSTED_POOL_TIME 键值未变）。
- 依据：tuning 键级 hunk 地图＋变更清单命中 tuning.lua。

- 源码审阅：2026-10-23，Steam buildid 25643519；完整核验 `nonslipgritpool`，定向核验 `slipperyfeet` 查询／累计／消耗分支、`nonslipgrit.lua` 强化砂来源与池完整工厂／到期回调、时长常量。已核验 source／user 的登记与消耗结论直接复用对应页，不重复展开。
- 结构检查：本页 `validate_page.py` 字段、大小与本地链接检查通过；`validate_source_links.py --strict` 源码锚点检查通过。无完整 Lua 示例，不新增临时 Lua。
- 游戏运行：未运行；大半径查找、圆边缘、退场动画期间继续防滑、网络端表现与跨高度重叠待实测；不宣称所有防滑实体或模组已覆盖。

相关：[[reference/components/slipperyfeet]]；[[reference/components/nonslipgritsource]]；[[reference/components/nonslipgrituser]]；[[reference/components/timer]]。
