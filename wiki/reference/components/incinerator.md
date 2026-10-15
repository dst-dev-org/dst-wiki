---
title: "incinerator"
created: "2026-10-24"
updated: "2026-10-28"
type: component
summary: "按过滤器销毁容器槽内整堆物品，发送焚烧与杀生事件，并由炉具宿主管理动作、动画和开关。"
aliases: ["incinerator", "容器焚烧", "批量销毁物品", "龙鳞火炉焚烧", "onincinerated"]
tags: [component, item, callback, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/incinerator.lua", "components/container.lua", "prefabs/dragonfurnace.lua", "containers.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# incinerator

## 快速判断

- 职责与适用：处理宿主 container 的全部槽位，将符合过滤条件的**整堆实体**移出并删除；不处理 inventory，也不直接点燃物品、产生灰烬或燃料。原版 dragonflyfurnace 在服务端把容器和本组件接在一起。[执行](../../../raw/current/scripts/components/incinerator.lua#L40-L54) [逐槽删除](../../../raw/current/scripts/components/container.lua#L330-L364) [端界与安装](../../../raw/current/scripts/prefabs/dragonfurnace.lua#L177-L199)
- 优先使用：先配置 container，再设置物品过滤与完成回调，执行用 `Incinerate(doer)`；容器布局／网络和普通移除语义复用 [[reference/components/container]]，不靠焚烧组件生成 UI。[原版安装](../../../raw/current/scripts/prefabs/dragonfurnace.lua#L193-L199)
- 重要副作用：每个被移出的物品先播放可选声音、收到 onincinerated，操作者可能收到 murdered／killed，再由容器删除；所有槽处理后才回调炉具。不是普通攻击或逐点 Health 扣减。[预销毁回调](../../../raw/current/scripts/components/incinerator.lua#L16-L37) [删除顺序](../../../raw/current/scripts/components/container.lua#L351-L360)
- 修改边界：直接执行只检查 container 存在，不复查开箱者、距离、骑乘或烧毁状态；这些动作候选条件不能代替自定义服务端入口的权限核验。空容器或过滤全拒仍可回调并返回 true。[执行门禁](../../../raw/current/scripts/components/incinerator.lua#L40-L54) [候选](../../../raw/current/scripts/componentactions.lua#L442-L449)

## 关键接口

| 接口／字段 | 参数与约定 | 作用与证据 |
|---|---|---|
| `SetShouldIncinerateItemFn(fn)`／`ShouldIncinerateItem(item)` | `fn(inst,item)`；默认允许 | setter 只赋回调；查询有回调就返回原值，否则 true。执行把同一个 fn 直接交给容器筛槽，不通过查询方法再包一层。[设置](../../../raw/current/scripts/components/incinerator.lua#L12-L13) [查询](../../../raw/current/scripts/components/incinerator.lua#L57-L62) [执行](../../../raw/current/scripts/components/incinerator.lua#L44) |
| `SetOnIncinerateFn(fn)` | `fn(inst)`，无 doer 或处理数量参数 | 所有槽遍历完成后执行，无论是否真的删过物品；返回值不影响 Incinerate 的结果。[设置](../../../raw/current/scripts/components/incinerator.lua#L8-L9) [调用](../../../raw/current/scripts/components/incinerator.lua#L44-L51) |
| `Incinerate(doer)` | doer 可 nil；返回 bool | 有 container 就临时保存 incinerate_doer、执行条件删除、完成回调、清 doer 后 true；无容器 false。没有计数、冷却或防重入门禁。[实现](../../../raw/current/scripts/components/incinerator.lua#L40-L54) |
| `OnPreDestroyItemCallbackFn(inst,item)` | 导出的预销毁函数 | inst 是焚烧宿主，item 已从槽中移出。按 incineratesound→health.murdersound→murderable.murdersound 的顺序选择声音，支持 FunctionOrValue，并向物品推事件。[实现](../../../raw/current/scripts/components/incinerator.lua#L16-L27) |
| `incinerate_doer` | 执行期间的临时字段 | 供预销毁函数读操作者，不是持久所有权；完成回调结束后清 nil。自定义回调抛错或重入没有异常／状态栈保护，不依赖它支持嵌套焚烧。[读字段](../../../raw/current/scripts/components/incinerator.lua#L16-L17) [写入与清除](../../../raw/current/scripts/components/incinerator.lua#L40-L51) |

### 删除与事件的实际顺序

容器按槽号检查过滤器，允许后 RemoveItemBySlot（整堆），调用预销毁函数，再检查物品仍有效才 Remove；不递归筛选被处理物品内部内容。过滤器为 nil 时直接无条件 DestroyContents，默认没有 irreplaceable 保护。[容器实现](../../../raw/current/scripts/components/container.lua#L330-L364) [整堆移出](../../../raw/current/scripts/components/container.lua#L569-L575)

物品的 `onincinerated` 载荷为 `{incinerator=inst, doer=doer}`。doer 非 nil 且物品有 health **或** murderable 时，向 doer 发 murdered，stackmult 取整堆数量；在该分支内，物品还带 combat 才另发 killed。判断不要求 Health 尚未死亡，也不走 Combat:GetAttacked，因此不能用这些事件推算战斗伤害。[事件条件](../../../raw/current/scripts/components/incinerator.lua#L27-L35)

## 典型接入

### 沿用原版炉具的容器、保护过滤与动画锁

本页是已有服务端炉具的组装指引，不是含资源／RPC／布局的完整建筑 demo：

1. dragonfurnace 工厂先 `container:WidgetSetup("dragonflyfurnace")`，再加 incinerator 和两个回调。布局资源、2×2 槽位与按钮在 containers.lua，组件本身没有网络变量或布局代码。[安装](../../../raw/current/scripts/prefabs/dragonfurnace.lua#L193-L199) [布局](../../../raw/current/scripts/containers.lua#L2052-L2080)
2. 原炉具过滤器拒绝 winter_food4、irreplaceable、以及仍有内容的子容器；这些是**宿主策略**，不是通用组件默认。布局入槽检查另外拒绝 irreplaceable，仍应保留执行过滤，防止绕过入槽路径的数据被毁。[执行过滤](../../../raw/current/scripts/prefabs/dragonfurnace.lua#L126-L138) [入槽检查](../../../raw/current/scripts/containers.lua#L2083-L2084)
3. 完成回调播放 incinerate→hi、关容器、设 canbeopened=false，等当前动画剩余时间加1帧再开放。动画由删除完成后的宿主回调启动，不能把动画开场当作稍后才删除内容。[宿主回调](../../../raw/current/scripts/prefabs/dragonfurnace.lua#L108-L123) [先删后回调](../../../raw/current/scripts/components/incinerator.lua#L44-L47)
4. 右键候选要求未 burnt、未骑乘、容器非空且 doer 已打开；INCINERATE 动作则直接转调组件，人物两端映射 doshortaction。容器按钮还有自己的可用判断与 BufferedAction／RPC 分派，不把“按钮可点”当通用安全入口。[右键收集](../../../raw/current/scripts/componentactions.lua#L442-L449) [动作](../../../raw/current/scripts/actions.lua#L6046-L6051) [人物SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1698) [客户端SG](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L943) [按钮](../../../raw/current/scripts/containers.lua#L2087-L2096)

### 识别杀生反馈与实际资源依赖

需要自定义物品声音时设置 item.incineratesound，或复用 health／murderable 的 murdersound；有声音的分支会访问炉具 SoundEmitter，创建新宿主须提供对应实体组件与声音资源。要根据物品变化处理统计，可监听物品的 onincinerated；原组件没有“焚烧产物配方”，需要返还材料时应另设计明确的宿主流程。[声音与事件](../../../raw/current/scripts/components/incinerator.lua#L19-L27) [原宿主组件](../../../raw/current/scripts/prefabs/dragonfurnace.lua#L144-L149)

## 常见陷阱与限制

- **没有允许焚烧的物品却返回成功：** true 仅表示存在容器并走完流程，不证明删除数量大于0；空容器、全被过滤或不能移出的物品也可触发完成动画。按需求另查合格内容，不把 bool 当数量。[返回](../../../raw/current/scripts/components/incinerator.lua#L40-L54) [只读容器不能移出](../../../raw/current/scripts/components/container.lua#L1019-L1022)
- **覆盖 ShouldIncinerateItem 方法来保护内容：** Incinerate 传给容器的是 shouldincinerateitemfn 字段，执行不会调用该查询方法；使用 SetShouldIncinerateItemFn，修改已有炉具时保留其原保护规则。[执行](../../../raw/current/scripts/components/incinerator.lua#L44) [查询](../../../raw/current/scripts/components/incinerator.lua#L57-L62)
- **一次只毁一件或自动保留珍贵物品：** RemoveItemBySlot 取整堆，默认过滤 nil 毁所有可移出内容。整堆杀生事件也以 stackmult 表示，不应另按事件次数推物品数量。[整堆](../../../raw/current/scripts/components/container.lua#L569-L575) [无条件回退](../../../raw/current/scripts/components/container.lua#L344-L348) [事件数量](../../../raw/current/scripts/components/incinerator.lua#L30-L34)
- **回调读取 doer 参数或返回 false 取消：** 完成回调只传 inst，运行时内容已处理；返回值不被读取。需要操作者可在这段回调中读取临时 incinerate_doer，但必须接受 nil，并避免递归进入同一实例。[时序](../../../raw/current/scripts/components/incinerator.lua#L40-L51)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页漂移文件 actions.lua（R-01）、componentactions.lua（R-04）、containers.lua（R-07）、stategraphs/SGwilson.lua（SB-17）与 SGwilson_client.lua 对照 build-26148036 更新报告与 actions／componentactions hunk 地图逐条核验；INCINERATE 动作、SCENE.incinerator 候选、dragonflyfurnace 布局／按钮与两端 SG 映射均未在变更 hunk 内（SGwilson 变更为施法／腐化相关，containers.lua 变更为 socket_keystone 按钮与 quagmire_pot 过滤），结论不变，锚点重定位 3 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-24，Steam buildid 25643519；完整审阅 incinerator，核验容器条件删除／整堆移出、龙鳞火炉安装与过滤／动画锁、布局入槽与按钮、动作收集／执行及人物SG映射。容器普通使用复用已建页，未全面审校 RPC 通用权限、物品 onincinerated 外部监听或全部皮肤／声音资源。
- 结构检查：本页格式、来源、本地链接及源码锚点严格检查通过；无新增完整 Lua 示例，导航登记由本批统一检查。
- 游戏运行：未运行；多人并发、过滤器动态变化、整堆杀生统计、自定义声音与嵌套容器销毁未实测。
