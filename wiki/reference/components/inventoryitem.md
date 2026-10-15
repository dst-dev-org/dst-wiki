---
title: inventoryitem
created: "2026-10-11"
updated: "2026-10-28"
type: component
summary: "让实体接入拾取与物品栏流程，配置收纳限制、图标和持有者回调；容器与装备另行组合。"
aliases: ["可拾取物品", "物品栏图标", "禁止放入箱子", "持有者", "入栏回调", "携带物品", "inventoryitem"]
tags: [component, item, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/inventoryitem.lua", "components/inventoryitem_replica.lua", "components/inventory.lua", "components/container.lua", "actions.lua", "componentactions.lua", "class.lua", "entityscript.lua", "prefabs/flint.lua", "prefabs/backpack.lua", "prefabs/heatrock.lua", "modutil.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# inventoryitem

## 快速判断

- 职责与适用：服务端 `inst:AddComponent("inventoryitem")` 接入物品侧拾取；权限／状态仍受动作检查，入栏由持有者 `inventory:GiveItem` 等管理。见[拾取限制](#常见陷阱与限制)与[原版组合](#原版组合的不同模式)。
- 优先使用：收纳标志、`ChangeImageName` 换图标、`SetOnPutInInventoryFn` 入栏回调；按 owner 链区分人物／箱子／背包。见[关键接口与证据](#关键接口)。
- 重要副作用：设置 owner／父实体、移出场景后才入栏回调；`onpickup` 不证明入栏完成，`ondropped` 不覆盖所有转移。见[生命周期与证据](#生命周期)。
- 修改边界：服务端改组件，客户端读 replica；格子／装备由 `container`／`equippable` 组合，不提供完整随身效果。见[原版组合](#原版组合的不同模式)。

## 关键接口

| 接口／字段 | 签名或原版约定 | 条件、作用与证据 |
|---|---|---|
| `canbepickedup` | boolean；默认 true | 普通权限；grab override 可覆盖，不保证所有状态可拾取。赋值经属性回调更新 replica；[inventoryitem.lua:13](../../../raw/current/scripts/components/inventoryitem.lua#L13)、[actions.lua:894](../../../raw/current/scripts/actions.lua#L894 |
| `cangoincontainer` | boolean；默认 true | **人物主栏也检查此值**，不要用 false 表达“只禁箱子”；inventory 可显式忽略此限制。[inventory.lua:231](../../../raw/current/scripts/components/inventory.lua#L231)、[container.lua:269](../../../raw/current/scripts/components/container.lua#L269 |
| `canonlygoinpocket` | boolean；默认 false | true 时 container 拒收，人物主栏不因本标志拒收；不保证已在主栏，不禁止鼠标手持。[inventoryitem.lua:68](../../../raw/current/scripts/components/inventoryitem.lua#L68)、[container.lua:269](../../../raw/current/scripts/components/container.lua#L269)、[inventory.lua:1184](../../../raw/current/scripts/components/inventory.lua#L1184 |
| `canonlygoinpocketorpocketcontainers` | boolean；默认 false | 与上项择一；接收容器自身须有 inventoryitem 且 `canonlygoinpocket` 为真，不泛指背包。[inventoryitem.lua:69](../../../raw/current/scripts/components/inventoryitem.lua#L69)、[container.lua:274](../../../raw/current/scripts/components/container.lua#L274 |
| `atlasname`／`imagename` | `string\|nil`；默认 nil | atlas 为 XML 路径；图片名不含 `.tex`，replica 补后缀。nil 时回退默认图集／Prefab 图片；[inventoryitem.lua:1](../../../raw/current/scripts/components/inventoryitem.lua#L1)、[inventoryitem_replica.lua:131](../../../raw/current/scripts/components/inventoryitem_replica.lua#L131)、[:151](../../../raw/current/scripts/components/inventoryitem_replica.lua#L151) |
| `ChangeImageName(newname)` | 图片名 string；无显式返回 | 写 imagename 并在物品发 `imagechange`；不切换 atlas，换图须已有匹配资源。replica 的 `OverrideImage` 仅本地覆盖；[inventoryitem.lua:470](../../../raw/current/scripts/components/inventoryitem.lua#L470)、[inventoryitem_replica.lua:135](../../../raw/current/scripts/components/inventoryitem_replica.lua#L135 |
| `owner`／`IsHeld()`／`IsHeldBy(guy)` | 实体或 nil／boolean／boolean | owner 是直接持有者（人物／箱子／背包）；IsHeld 只查非 nil，IsHeldBy 比较相等，guy 须为实际实体。[inventoryitem.lua:462](../../../raw/current/scripts/components/inventoryitem.lua#L462 |
| `GetGrandOwner()` | 实体或 nil | 沿 inventoryitem.owner 递归到末端，末端非 nil 也未必是玩家。（26148036 核验：重写为 and/or 表达式，owner 持有 inventoryitem 但递归结果为 nil 时（如物品在放在地上的背包中）改为返回该 owner，旧版返回 nil；对返回值做判等的消费者受影响。）[inventoryitem.lua:501](../../../raw/current/scripts/components/inventoryitem.lua#L501 |
| `GetContainer()`／`GetSlotNum()` | 持有者组件或 nil／槽号或 nil | 前者返回直接 owner 的 container 或 inventory；槽号查询不能证明装备／activeitem 状态。[inventoryitem.lua:319](../../../raw/current/scripts/components/inventoryitem.lua#L319 |
| `SetOnPickupFn(fn?)`／`SetOnPutInInventoryFn(fn?)`／`SetOnDroppedFn(fn?)` | 回调或 nil；无显式返回 | 各为单回调，赋值替换、nil 清除；参数／返回／顺序见[生命周期](#生命周期)。[inventoryitem.lua:293](../../../raw/current/scripts/components/inventoryitem.lua#L293 |
| `RemoveFromOwner(wholestack?, keepoverstacked?)` | 原版返回移出的物品或 nil | 转交持有者 RemoveItem；完整移出传 true，省略／false 可能仅拆一件堆叠。普通移出与删除／丢落分开处理。[inventoryitem.lua:475](../../../raw/current/scripts/components/inventoryitem.lua#L475)、[inventory.lua:1390](../../../raw/current/scripts/components/inventory.lua#L1390 |

字段经 Class 属性回调／replica setter 同步；owner 更新还改 classified 可见目标，服务端 owner 不是客户端公开字段。客户端 `IsHeldBy` 仅查本地 `ThePlayer`，不支持任意玩家。[class.lua:36](../../../raw/current/scripts/class.lua#L36)、[inventoryitem.lua:96](../../../raw/current/scripts/components/inventoryitem.lua#L96)、[inventoryitem_replica.lua:171](../../../raw/current/scripts/components/inventoryitem_replica.lua#L171)、[:201](../../../raw/current/scripts/components/inventoryitem_replica.lua#L201)

## 生命周期

物品事件在物品监听；持有者事件在对应 inventory／container 宿主监听。

| 入口／事件 | 参数或 payload | 顺序与适用边界 |
|---|---|---|
| `OnPickup(pickupguy, src_pos)` → 物品 `onpickup` | 事件 `{owner = pickupguy}`；随后回调 `(inst, pickupguy, src_pos)` | GiveItem 在选槽／入栏前调用；回调返回 truthy 会让该调用者停止后续 GiveItem，被当作物品已消耗，不能返回 true 表达普通成功。（26148036 核验：OnPickup 内的拾取灭火逻辑抽为可复用方法 `OnPickupStopSmoldering(pickupguy)`，行为不变，`ACTIONS.WOBY_PICKUP` 亦调用该方法。）[inventoryitem.lua:447](../../../raw/current/scripts/components/inventoryitem.lua#L447)、[inventory.lua:1055](../../../raw/current/scripts/components/inventory.lua#L1055 |
| `OnPutInInventory(owner)` → 物品 `onputininventory` | 回调 `(inst, owner)`；随后事件 data **直接是 owner 实体** | 设置 owner／AddChild／RemoveFromScene 后执行；人物主栏、activeitem、装备和容器均可能走此入口，不只地面拾取。[inventoryitem.lua:348](../../../raw/current/scripts/components/inventoryitem.lua#L348)、[inventory.lua:1184](../../../raw/current/scripts/components/inventory.lua#L1184)、[:1364](../../../raw/current/scripts/components/inventory.lua#L1364)、[container.lua:542](../../../raw/current/scripts/components/container.lua#L542 |
| `OnRemoved()` | 无专用回调／归属变更事件 | 清 owner、脱父子关系、ReturnToScene；转移或取出可走此路径，不自动发 ondropped。[inventoryitem.lua:369](../../../raw/current/scripts/components/inventoryitem.lua#L369 |
| `OnDropped(...)` → 物品 `ondropped` | 回调 `(inst)`；随后事件无 payload | 有效实体先 OnRemoved，再抛落物理，再回调／事件；此时 owner 已清空。[inventoryitem.lua:378](../../../raw/current/scripts/components/inventoryitem.lua#L378 |
| 持有者 `itemget`／`itemlose`，普通槽位分支 | `{item, slot, src_pos}`／`{prev_item, slot}` | 入栏在登记槽位／OnPutInInventory 后发 itemget；移出在清槽后、物品 OnRemoved 前发 itemlose，事件内可能仍读到旧 owner。[inventory.lua:1149](../../../raw/current/scripts/components/inventory.lua#L1149)、[:1405](../../../raw/current/scripts/components/inventory.lua#L1405)、[container.lua:1040](../../../raw/current/scripts/components/container.lua#L1040 |

可拾取容器入栏／丢落还向内部物品发 `onownerputininventory`（data 为 owner）／`onownerdropped`，并非任意层数的全局归属变更通知。[inventoryitem.lua:362](../../../raw/current/scripts/components/inventoryitem.lua#L362)、[:394](../../../raw/current/scripts/components/inventoryitem.lua#L394)

移组件的 `OnRemoveFromEntity` 只清辅助组件与自身监听；实体删除的 `OnRemoveEntity` 则请求持有者移出物品。宿主删除先发 onremove，再调用各组件 OnRemoveEntity。不能把移组件、转移、丢落和删除当成同一收尾入口。[inventoryitem.lua:106](../../../raw/current/scripts/components/inventoryitem.lua#L106)、[:485](../../../raw/current/scripts/components/inventoryitem.lua#L485)、[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670)、[:1730](../../../raw/current/scripts/entityscript.lua#L1730)

## 典型接入

### 禁止燧石放入箱子或背包，保留人物主栏收纳

- 前提与选择：完整代码放已加载的目标模组 `modmain.lua`；沿用 flint 资源／组件，原版工厂后只改收纳策略，不覆盖回调。[flint.lua:36](../../../raw/current/scripts/prefabs/flint.lua#L36)、[:52](../../../raw/current/scripts/prefabs/flint.lua#L52)、[modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)、[mainfunctions.lua:388](../../../raw/current/scripts/mainfunctions.lua#L388
- 生效与撤销：所有后续初始化的 flint 禁入 container；主栏容量／堆叠、鼠标手持沿原版。不迁移已入容器的物品；停用后新实例恢复原版。[inventory.lua:231](../../../raw/current/scripts/components/inventory.lua#L231)、[container.lua:269](../../../raw/current/scripts/components/container.lua#L269

```lua
AddPrefabPostInit("flint", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local inventoryitem = inst.components.inventoryitem
    if inventoryitem == nil then
        return
    end
    inventoryitem.cangoincontainer = true
    inventoryitem.canonlygoinpocket = true
    inventoryitem.canonlygoinpocketorpocketcontainers = false
end)
```

- 调整：换目标先核对原收纳／装备限制；三个策略值均被覆盖，须协调其他模组改动。无新资源／数值单位；验证范围见[验证记录](#验证记录)。

### 原版组合的不同模式

| 模式 | 原版参照 | 各组件的分工 |
|---|---|---|
| 基础拾取物品 | [flint.lua:21](../../../raw/current/scripts/prefabs/flint.lua#L21)、[:36](../../../raw/current/scripts/prefabs/flint.lua#L36)、[:47](../../../raw/current/scripts/prefabs/flint.lua#L47) | 网络／物理／动画在公共初始化，SetPristine 后客户端返回；服务端分别加 stackable、inspectable、inventoryitem，堆叠与检查是独立能力。 |
| 可拾取、可装备的容器 | [backpack.lua:268](../../../raw/current/scripts/prefabs/backpack.lua#L268 | inventoryitem 管自身归属，container 管内部格子，equippable 管装备；原版背包另将 cangoincontainer 设 false，不等于所有便携容器都应如此。 |
| 携带温度与发光 | [heatrock.lua:232](../../../raw/current/scripts/prefabs/heatrock.lua#L232)、[:257](../../../raw/current/scripts/prefabs/heatrock.lua#L257)、[:153](../../../raw/current/scripts/prefabs/heatrock.lua#L153) | inventoryitem 不替代 temperature／heater；暖石另建灯光实体并追踪 owner 链。只参考当前需求的机制，不把它的两个事件监听视为完整自定义效果清理。 |

## 常见陷阱与限制

- **把有组件等同于始终可拾取**：普通动作还检查 INLIMBO、燃烧、投射状态和人物限制（26148036 核验：燃烧排除放宽——燃烧中带 `torch` 标签的物品可被拾取，服务端 PICKUP.fn 与客户端 SCENE 过滤两侧同步，客户端另放行 `lighter`）；非装备容器的右键不生成 PICKUP，被其他人打开时服务端也拒绝拾取。新增 container 后应核对左右键与占用流程。[componentactions.lua:474](../../../raw/current/scripts/componentactions.lua#L474)、[actions.lua:904](../../../raw/current/scripts/actions.lua#L904
- **用改 owner／直接 OnPutInInventory 代替入栏**：组件不登记持有者槽位；通过 inventory／container 的实际转移入口处理。onputininventory 不保证主栏或装备槽登记完成，activeitem／装备路径先回调后登记。[inventoryitem.lua:285](../../../raw/current/scripts/components/inventoryitem.lua#L285)、[inventory.lua:1149](../../../raw/current/scripts/components/inventory.lua#L1149)、[:1184](../../../raw/current/scripts/components/inventory.lua#L1184)、[:1364](../../../raw/current/scripts/components/inventory.lua#L1364)
- **只靠 GetGrandOwner 和 ondropped 管人物效果**：末端玩家不证明主栏；转移可能不发 ondropped，itemlose 时旧 owner 仍在。按[持有者接口](#关键接口)和[生命周期](#生命周期)核验清理；主栏／背包／手持／装备范围见 [[reference/components/inventory#携带范围的选择]]，本页无完整随身效果实现。
- **携带发光只开物品自身 Light**：入栏会 RemoveFromScene，不能据此保证携带照明；原版暖石用独立灯光并调整父实体／场景状态。具体网络显示仍需运行验证。[inventoryitem.lua:353](../../../raw/current/scripts/components/inventoryitem.lua#L353)、[heatrock.lua:174](../../../raw/current/scripts/prefabs/heatrock.lua#L174

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（components/inventoryitem.lua、actions.lua、componentactions.lua、entityscript.lua、mainfunctions.lua）对照 build-26148036 更新报告条目 R-01／R-04／C-19／C-27 逐条核验，锚点重定位 19 处、结论修订 2 处（GetGrandOwner 地上容器场景返回值 nil→owner、燃烧 torch 标签物可拾取，见正文"26148036 核验"标注），并在 OnPickup 行补记拾取灭火逻辑抽为 `OnPickupStopSmoldering`；未覆盖：运行验证。

- 源码审阅：2026-10-11，Steam buildid 25643519；覆盖本页拾取入口、收纳标志／图标、owner 查询、回调 payload／顺序、移出／删除及三种原版组合，依据见各项链接。仅精选行为知识；未进行全组件类型审校、正式 reviews 或 LuaCATS 导出。
- 结构检查：页面元数据／源码路径／行号、链接／章节、标签、大小和导航登记检查通过；上述单个 Lua 示例经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；拾取／收纳实际交互、客户端图标与照明、重载已有容器内容、模组冲突及完整随身效果清理未测试。

相关：[[reference/components/inventory#携带范围的选择|人物收纳范围]]；[[reference/components/container|内部格子]]；[[reference/components/stackable|件数]]；[[reference/components/equippable#典型接入|装备属性]]；[[reference/components/finiteuses#典型接入|按次消耗]]；[[reference/components/heater#携带范围与地面热源的分工|携带供热]]；[[reference/components/sanityaura#附近光环装备与随身效果如何选择|精神效果选型]]。
