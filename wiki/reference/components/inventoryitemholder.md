---
title: "inventoryitemholder"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "让建筑持有一件实物或同类堆叠，接入交入、取回与存档；展示和腐败处理由宿主补足。"
aliases: ["单物品展示架", "交入和取回物品", "建筑持有物品", "inventoryitemholder"]
tags: [component, item, building, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/inventoryitemholder.lua", "prefabs/gelblob_storage.lua", "prefabs/carpentry_station.lua", "standardcomponents.lua", "componentactions.lua", "actions.lua", "modutil.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# inventoryitemholder

## 快速判断

- 职责与适用：服务端宿主持有一个真实 inventoryitem，可选同类堆叠；不是有格子／开包界面的 [[reference/components/container]]。[实现](../../../raw/current/scripts/components/inventoryitemholder.lua#L89-L163)
- 优先使用：插入配件用交入／取出回调；单件参照木工台，同类食物堆叠参照 gelblob_storage。[木工台](../../../raw/current/scripts/prefabs/carpentry_station.lua#L193-L196)、[食物宿主](../../../raw/current/scripts/prefabs/gelblob_storage.lua#L206-L210)
- 重要副作用：交入会脱离人物库存、挂为子实体、移出场景并加 outofreach；回调再决定显示和业务效果。[GiveItem](../../../raw/current/scripts/components/inventoryitemholder.lua#L117-L160)
- 修改边界：本组件不内建保鲜／展示同步；客户端动作依赖 give／take 标签，具名物品和拆一件还需宿主提供 takeitem 网络实体。[动作消费](../../../raw/current/scripts/componentactions.lua#L482-L503)、[原版 takeitem](../../../raw/current/scripts/prefabs/gelblob_storage.lua#L184-L189)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `SetAllowedTags(tags)` | 标签字符串列表或 nil；默认 nil | 任一标签匹配即可；nil 不限制，仍要求物品有 inventoryitem。不会重检已持有内容。[配置和判断](../../../raw/current/scripts/components/inventoryitemholder.lua#L65-L104) |
| `SetAcceptStacks(bool)` | 仅 true 开启；默认 false | 非空时只接收可合并且未满的同类 stackable；空架交入人物物品时决定 DropItem 是否取整堆。[配置](../../../raw/current/scripts/components/inventoryitemholder.lua#L77-L79)、[交入](../../../raw/current/scripts/components/inventoryitemholder.lua#L112-L137) |
| `CanGive(item, giver)` | true／false／nil | 只做物品和容量判断，不保证 GiveItem 后续成功；最终 owner 非 inventory 时 GiveItem 另行拒收。[判断与转移](../../../raw/current/scripts/components/inventoryitemholder.lua#L89-L137) |
| `GiveItem(item, giver)` | boolean | 合并余量直接送回 giver.inventory，自定义调用者须满足此前提；成功不等于输入实体仍有效。[交入实现](../../../raw/current/scripts/components/inventoryitemholder.lua#L112-L163) |
| `IsHolding()`／`CanTake(taker)` | boolean | 前者仅查 item 非 nil，后者另查有效性；均不验证操作者权限。[查询](../../../raw/current/scripts/components/inventoryitemholder.lua#L83-L108) |
| `TakeItem(taker?, wholestack?)` | boolean；默认整堆 | false 拆一件；有效 taker 有 inventory 则送入，否则落在宿主位置。返回成功标志，不返回物品。[取出](../../../raw/current/scripts/components/inventoryitemholder.lua#L165-L210) |
| `SetOnItemGivenFn(fn)` | `fn(inst, item, giver)` 或 nil | 单回调替换；叠入已有堆后 item 可为 nil，应读 holder.item 或检查有效性。[调用点](../../../raw/current/scripts/components/inventoryitemholder.lua#L153-L160) |
| `SetOnItemTakenFn(fn)` | `fn(inst, item, taker, removed)` 或 nil | removed 表示原持有实体整体离架；移入库存合并后 item 可失效。外部转移也以 taker=nil、removed=true 调用。[外部转移](../../../raw/current/scripts/components/inventoryitemholder.lua#L39-L54)、[取出回调](../../../raw/current/scripts/components/inventoryitemholder.lua#L193-L207) |

存档仅保存有效且 persists 的 item，同时返回其引用；读档 SpawnSaveRecord 后重新 GiveItem，并以宿主自身作 giver，因此回调会重跑。移组件会 TakeItem 落地并清交互标签。[保存／恢复／移组件](../../../raw/current/scripts/components/inventoryitemholder.lua#L214-L242)

## 典型接入

### 让 gelblob_storage 只接受新交入的肉类，保留旧存档内容

完整代码放已启用模组的 `modmain.lua`，无新增资源。仅在服务端包装该宿主的 CanGive，不替换原有食物展示、停腐败、取回和锤毁回调；注册名由原版工厂确认。[宿主配置与注册](../../../raw/current/scripts/prefabs/gelblob_storage.lua#L206-L223)、[postinit 注册](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("gelblob_storage", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local holder = inst.components.inventoryitemholder
    if holder == nil then return end
    local previous_can_give = holder.CanGive
    holder.CanGive = function(self, item, giver)
        if giver ~= self.inst and not item:HasTag("edible_MEAT") then
            return false
        end
        return previous_can_give(self, item, giver)
    end
end)
```

- 选择理由：直接缩窄 allowed_tags 会连旧食物读档也拒绝；此例保留原标签／堆叠限制，仅豁免以宿主为 giver 的恢复调用。原版 OnLoad 的 giver 约定见[恢复实现](../../../raw/current/scripts/components/inventoryitemholder.lua#L225-L233)；postinit 在原工厂之后执行，见[创建入口](../../../raw/current/scripts/mainfunctions.lua#L388-L405)。
- 生效与撤销：影响该模组加载期间新建和重载宿主的后续交入；旧存食物可继续取回。同类非肉旧堆也不能再补入；直接以宿主为 giver 的其他代码同样获豁免，这不是权限系统。停用后初始化恢复原规则，无额外存档字段。
- 原效果：宿主已有回调负责暂停／恢复原本运行的 perishable、显示物品、同步 takeitem 及恢复拾取性，不应简单替换。（26148036 核验：gelblob_storage 新增 OnSave／OnLoad 持久化 `_wasperishing`，交入时记录的"是否本来就在腐败"现跨存档保留。）[交入](../../../raw/current/scripts/prefabs/gelblob_storage.lua#L64-L103)、[取出](../../../raw/current/scripts/prefabs/gelblob_storage.lua#L105-L139)
- 调整：替换 edible_MEAT 前核对目标标签；服务端拒收不保证客户端预先隐藏 GIVE，原动作入口只查通用标签。[动作生成](../../../raw/current/scripts/componentactions.lua#L1443-L1445)

### 配件插拔与销毁归还

木工台以 carpentry_blade 过滤，交入后改科技树／覆写动画，取回时复原；不是把任意可拾取物都当作有效刀片。[业务回调](../../../raw/current/scripts/prefabs/carpentry_station.lua#L114-L136)

宿主销毁须接通实际归还路径：原木工台锤毁先 TakeItem 再 Remove；标准建筑烧毁函数也先取物再移组件，不能从组件注释推断任意自定义删除入口都安全落物。[锤毁](../../../raw/current/scripts/prefabs/carpentry_station.lua#L22-L41)、[标准烧毁](../../../raw/current/scripts/standardcomponents.lua#L120-L123)

## 常见陷阱与限制

- **把 holder 当保鲜箱**：默认只 HibernateLivingItem，不统一 StopPerishing；gelblob 的食品支持来自宿主回调。新增食品架需成对处理进入／取出，保鲜容器方案另见 [[reference/components/preserver]]。[基础交入](../../../raw/current/scripts/components/inventoryitemholder.lua#L139-L150)、[宿主补足](../../../raw/current/scripts/prefabs/gelblob_storage.lua#L83-L97)
- **回调永远收到有效物品**：叠入可传 nil，取出入栏可合并失效；外部删除物品只清 self.item，不触发 taken 回调。效果清理不能只押在一个回调上。[删除监听](../../../raw/current/scripts/components/inventoryitemholder.lua#L37-L54)、[取出](../../../raw/current/scripts/components/inventoryitemholder.lua#L193-L207)
- **收紧过滤自动迁移旧内容**：OnLoad 不检查 GiveItem 返回值，也无失败返还逻辑；SpawnSaveRecord 成功后若被新规则拒收，不能保证重新挂回或妥善安置。恢复规则应兼容旧数据，迁移须单独设计。[恢复](../../../raw/current/scripts/components/inventoryitemholder.lua#L225-L233)
- **直接调用等同动作门禁**：CanTake 不查火焰／人物权限，客户端动作另排除 fire；普通 GIVE／TAKEITEM 最终才调用组件，私有调用应自行核验条件。[取出动作](../../../raw/current/scripts/actions.lua#L3401-L3409)、[交入动作](../../../raw/current/scripts/actions.lua#L2341-L2342)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（componentactions.lua、actions.lua、mainfunctions.lua、prefabs/gelblob_storage.lua）对照 build-26148036 更新报告条目 R-01／R-04 逐条核验，锚点重定位 6 处，无结论修订；补记 gelblob_storage 新增 `_wasperishing` 存读（见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖精选接口、堆叠回调、实体持有／外部转移、存档重入、移组件、两种宿主及标准烧毁、上述窄范围 postinit 案例。引用见对应结论；未做全组件类型审校。
- 结构／语法：本页元数据、链接、源码锚点和大小检查通过；一个完整 modmain.lua 示例通过已有 Full Moon Lua 5.1 语法检查。
- 游戏运行：未运行；多人拒收提示、食物堆叠、旧存档非肉内容、外部删除及其他模组包装顺序仍需实际验证。

相关：[[reference/components/inventoryitem]]；[[reference/components/container]]；[[reference/components/stackable]]。
