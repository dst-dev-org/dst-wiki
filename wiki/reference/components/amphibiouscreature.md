---
title: "amphibiouscreature"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "维护生物水陆状态、动画 bank 与进出水回调，并向 locomotor／跳跃状态图提供转换判定。"
aliases: ["amphibiouscreature", "生物水陆切换", "游泳动画切换", "两栖生物岸边转换"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/amphibiouscreature.lua", "components/locomotor.lua", "prefabs/grassgator.lua", "stategraphs/SGgrassgator.lua", "stategraphs/commonstates.lua", "prefabs/shark.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# amphibiouscreature

## 快速判断

- 职责与适用：维护 `in_water`、`swimming` 标签、两套动画 bank 和进出水回调；不自行实现水上寻路、岸边跳跃或游泳物理。原版草鳄由服务端安装，另配 locomotor、embarker 和状态图。26230496 起构造检查 `TheWorld.has_ocean`：无海洋世界置 `no_ocean`，不启动更新、不监听 death。[amphibiouscreature.lua:5-26](../../../raw/current/scripts/components/amphibiouscreature.lua#L5-L26) [grassgator.lua:174-178](../../../raw/current/scripts/prefabs/grassgator.lua#L174-L178) [grassgator.lua:222-260](../../../raw/current/scripts/prefabs/grassgator.lua#L222-L260)
- 优先使用：已有完整移动／动画体系的生物需要水陆状态回调时，使用 SetBanks 和进出水回调；需要岸边运动时必须核对 locomotor 与 SG 接线，不能只添加组件。[amphibiouscreature.lua:42-53](../../../raw/current/scripts/components/amphibiouscreature.lua#L42-L53) [amphibiouscreature.lua:83-117](../../../raw/current/scripts/components/amphibiouscreature.lua#L83-L117)
- 重要副作用：实际状态变化时切 bank、增删 `swimming`，再调用对应 `fn(inst)`，最后即时推送 `ms_enterwater`／`ms_exitwater`（PushEventImmediate，SG 击退态等同步消费）；重复进入同一侧不再调用回调。设置回调本身不补发当前状态。[amphibiouscreature.lua:83-117](../../../raw/current/scripts/components/amphibiouscreature.lua#L83-L117)
- 修改边界：自动状态采样用 `IsPassableAtPoint`，预判转换用 `IsVisualGroundAtPoint`；二者不是同一种地图判断。SG 有 `jumping` 标签时跳过自动采样，由跳跃流程明确切换状态；水陆采样逻辑已抽为公开 `CheckSwimming()`（击退态等外部流程可调用，`no_ocean` 时早退）。[amphibiouscreature.lua:55-73](../../../raw/current/scripts/components/amphibiouscreature.lua#L55-L73) [commonstates.lua:1118-1134](../../../raw/current/scripts/stategraphs/commonstates.lua#L1118-L1134)

## 关键接口

| 接口／字段 | 签名或约束 | 条件、作用与证据 |
|---|---|---|
| `in_water` | 初始 false 的状态字段 | 供宿主读取，建议通过进出水方法改变，以保持标签、bank 和回调一致；构造不立即采样当前位置。[amphibiouscreature.lua:5-26](../../../raw/current/scripts/components/amphibiouscreature.lua#L5-L26) [amphibiouscreature.lua:83-109](../../../raw/current/scripts/components/amphibiouscreature.lua#L83-L109) |
| `SetBanks(land,ocean)` | 动画 bank 名；可不配置对应项 | 只存名称，不立即 SetBank；以后发生进出水才切对应 bank，不切 build，也不主动播放动画。[amphibiouscreature.lua:42-45](../../../raw/current/scripts/components/amphibiouscreature.lua#L42-L45) [amphibiouscreature.lua:83-109](../../../raw/current/scripts/components/amphibiouscreature.lua#L83-L109) |
| `SetEnterWaterFn(fn)`／`SetExitWaterFn(fn)` | 可选 `fn(inst)`，各一个槽 | 覆盖回调；只有实际状态翻转时调用（回调后紧跟即时 `ms_enterwater`／`ms_exitwater` 事件），组件不保存或自动撤销回调施加的速度／阴影等副作用。[amphibiouscreature.lua:111-117](../../../raw/current/scripts/components/amphibiouscreature.lua#L111-L117) |
| `OnEnterOcean()`／`OnExitOcean()` | 无参数；不返回位置或运动结果 | 分别从陆态切水态／从水态切陆态；不重查地图、不移动实体，因此直接调用可以暂时与当前位置不一致。[amphibiouscreature.lua:83-109](../../../raw/current/scripts/components/amphibiouscreature.lua#L83-L109) |
| `SetTransitionDistance(distance)`／`GetTransitionDistance()` | 默认2.5，地图距离单位 | 设置／返回前向探测距离参数；本组件不拿它计时或移动。locomotor 将其与物理半径相加作为转换探测距离。[amphibiouscreature.lua:16](../../../raw/current/scripts/components/amphibiouscreature.lua#L16 [amphibiouscreature.lua:47-53](../../../raw/current/scripts/components/amphibiouscreature.lua#L47-L53) [locomotor.lua:1719-1734](../../../raw/current/scripts/components/locomotor.lua#L1719-L1734) |
| `ShouldTransition(x,z)` | 返回地图判定的布尔结果 | 当前在水中时要求目标为视觉地面；当前在陆上时要求目标非视觉地面，y固定0。不含可达性、阻挡、平台或跳跃状态检查。[amphibiouscreature.lua:75-81](../../../raw/current/scripts/components/amphibiouscreature.lua#L75-L81) |

## 关键流程

### 自动采样与显式转换

构造在实体未休眠且未死亡时开始组件更新，并监听宿主 `death` 停止更新；26230496 起仅当 `TheWorld.has_ocean` 才挂监听（无海洋世界 `no_ocean` 短路，唤醒也不重启更新）。每次更新若无 SG 或 SG 不在 `jumping` 状态，取实体当前位置，以 `Map:IsPassableAtPoint` 作为 `is_on_land`；与当前 `in_water` 不匹配时调用进／出水方法。这里没有用 `IsOceanAtPoint`，也没有在构造当场推断“所有不可通行点都是安全水域”。水陆采样已抽为公开 `CheckSwimming()`。[amphibiouscreature.lua:1-26](../../../raw/current/scripts/components/amphibiouscreature.lua#L1-L26) [amphibiouscreature.lua:55-67](../../../raw/current/scripts/components/amphibiouscreature.lua#L55-L67)

转换顺序是：有对应 bank 时 SetBank → 更新 `in_water` → 增删 `swimming` → 调用回调。直接 OnEnterOcean／OnExitOcean 不受 `jumping` 门槛限制，原版跳跃状态正是用它们在动画边界明确改变水陆表现。[amphibiouscreature.lua:83-109](../../../raw/current/scripts/components/amphibiouscreature.lua#L83-L109) [commonstates.lua:1083-1104](../../../raw/current/scripts/stategraphs/commonstates.lua#L1083-L1104) [commonstates.lua:1118-1134](../../../raw/current/scripts/stategraphs/commonstates.lua#L1118-L1134)

### 岸边跳跃由移动与状态图协作

locomotor 仅在已有正向移动速度、允许平台跳跃且动作没有禁用跳跃时进入相关检测。普通平台跳跃不成立、目标无平台且 SG 不在 jumping 时，才读取本组件的转换距离与 ShouldTransition；外部还进行平台扫描／阻挡测试，满足后发 `onhop`。因此 ShouldTransition 返回真不是“可跳且必到达”的承诺。[locomotor.lua:1658-1679](../../../raw/current/scripts/components/locomotor.lua#L1658-L1679) [locomotor.lua:1697-1740](../../../raw/current/scripts/components/locomotor.lua#L1697-L1740)

`CommonHandlers.OnHop` 再按生死、moving／idle、jumping 等 SG 条件决定进入 hop_pre 或 hop_antic；`AddAmphibiousCreatureHopStates` 提供 jumping 状态、embarker／运动协作及碰撞切换。hop_pre 的相应动画／登船事件可先 OnExitOcean，hop_pst 根据传入的 `land_in_water` 决定进水或出水。本组件只消费这些明确的切换调用。[commonstates.lua:769-784](../../../raw/current/scripts/stategraphs/commonstates.lua#L769-L784) [commonstates.lua:1020-1114](../../../raw/current/scripts/stategraphs/commonstates.lua#L1020-L1114) [commonstates.lua:1118-1152](../../../raw/current/scripts/stategraphs/commonstates.lua#L1118-L1152)

### 生命周期与保存边界

实体休眠停止组件更新，醒来且未死亡（且非无海洋世界）才重启；死亡事件停止更新，不自动退出水态。组件无 `OnSave`／`OnLoad`，不保存 `in_water`、bank 或回调；重建时从 false 起，依靠后续采样或宿主显式调用重新建立状态。[amphibiouscreature.lua:1-40](../../../raw/current/scripts/components/amphibiouscreature.lua#L1-L40)

`OnRemoveFromEntity` 只撤死亡监听，不调用 OnExitOcean，也不恢复 bank、`swimming` 标签或宿主回调产生的外部效果。若运行时要撤销组件，应先按宿主需要完成状态与副作用恢复，不能将删除组件等同于回陆。[amphibiouscreature.lua:28-30](../../../raw/current/scripts/components/amphibiouscreature.lua#L28-L30) [amphibiouscreature.lua:97-109](../../../raw/current/scripts/components/amphibiouscreature.lua#L97-L109)

## 典型接入

### 草鳄：配置完整的岸边转换体系

原版草鳄在服务端构造 locomotor、设置可水上寻路，安装 SG／Brain 和 embarker，开启平台跳跃，再安装本组件并配置陆水 bank。进水回调保存陆地跑速和跳跃距离，改用水上跑速、跳跃距离4；出水回调恢复对应值。其 pathcaps 另设 `allowocean=true`，这些移动能力不是本组件自动赋予的。[grassgator.lua:174-178](../../../raw/current/scripts/prefabs/grassgator.lua#L174-L178) [grassgator.lua:222-260](../../../raw/current/scripts/prefabs/grassgator.lua#L222-L260)

SGgrassgator 接 `CommonHandlers.OnHop`，并调用公共两栖跳跃状态构造器补入状态，另附水花／停止移动时间线。这是原版内部接线参照；自定义生物需要自己的资源、碰撞、embarker 与 SG 配合，本页不提供只加组件的通用两栖 demo。[SGgrassgator.lua:23-30](../../../raw/current/scripts/stategraphs/SGgrassgator.lua#L23-L30) [SGgrassgator.lua:550-578](../../../raw/current/scripts/stategraphs/SGgrassgator.lua#L550-L578)

### 鲨鱼：进出水回调还要协调动画可用性

鲨鱼的服务端接线设置 `shark`／`shark_water` bank；进水保存陆上跑／走速并改水速、关动态阴影，出水恢复速度／阴影。若出水时 SG 仍有 moving 标签，宿主主动切 `leap`，因为陆上鲨鱼没有对应走跑动画。不要把 SetBanks 当成已经解决所有状态图动画差异。[shark.lua:239-243](../../../raw/current/scripts/prefabs/shark.lua#L239-L243) [shark.lua:273-305](../../../raw/current/scripts/prefabs/shark.lua#L273-L305)

## 常见陷阱与限制

- **改 in_water 就完成切换：** 直接改字段会漏掉标签、bank 和回调；用进出水方法，并明确是否需要立即纠正地图状态。[amphibiouscreature.lua:83-109](../../../raw/current/scripts/components/amphibiouscreature.lua#L83-L109)
- **transitiondistance 是实际跳远：** 它只被 locomotor 用于前方转换探测；真实运动、碰撞、跳跃距离与动画由其他组件及 SG 负责。[locomotor.lua:1719-1734](../../../raw/current/scripts/components/locomotor.lua#L1719-L1734) [commonstates.lua:1029-1073](../../../raw/current/scripts/stategraphs/commonstates.lua#L1029-L1073)
- **组件自带客户端同步：** 实现没有网络变量或 replica；原版以服务端宿主接线为准，不能仅在客户端添加它便获得一致的权威水陆状态。[amphibiouscreature.lua:5-23](../../../raw/current/scripts/components/amphibiouscreature.lua#L5-L23) [grassgator.lua:174-178](../../../raw/current/scripts/prefabs/grassgator.lua#L174-L178)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件全文、locomotor 转换分支、公共 OnHop 与完整两栖跳跃状态构造器、草鳄安装／回调及 SG 接线、鲨鱼服务端安装与进出水回调，覆盖状态判定、运动分工、回调顺序、休眠／死亡／移除与无组件存读边界。
- 游戏运行：未运行；未测地图原生判定在各类平台／地形上的结果、跳跃打断、读档初始化时序或自定义动画兼容性。未穷举所有两栖宿主。
- 结构检查：`validate_page.py` 通过；两页合并执行 `validate_source_links.py --strict`，87个行号引用、0错误／0警告。本页无新增 Lua 示例；机械检查不代替游戏测试。

- 2026-10-28 build 26230496：has_ocean 短路、CheckSwimming 公开、ms_enterwater/ms_exitwater 即时事件按报告 C-9 逐条核验并修订正文；锚点全部重定位。
相关：[[reference/components/locomotor]] · [[reference/components/walkableplatform]]
