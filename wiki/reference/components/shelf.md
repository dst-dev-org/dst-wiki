---
title: "shelf"
created: "2026-10-21"
updated: "2026-10-28"
type: "component"
summary: "为建筑提供单件展示槽与可取开关，派生 takeshelfitem 标签，供取物动作、猪人等消费者读取。"
aliases: ["展示槽", "展示台取物", "takeshelfitem", "PutItemOnShelf", "TakeItem"]
tags: ["component", "lifecycle", "callback"]
dst_build: "steam-buildid:26230496"
sources: ["components/shelf.lua", "prefabs/birdcage.lua", "prefabs/rat_gym.lua", "prefabs/yotc_carrat_scale.lua", "prefabs/lureplant.lua", "actions.lua", "componentactions.lua", "brains/pigbrain.lua", "components/inventory.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# shelf

## 快速判断

- 职责与适用：给建筑一个逻辑展示槽 `itemonshelf` 和可取开关 `cantakeitem`，并派生 `takeshelfitem` 标签；物品实体通常由宿主的 `inventory` 持有，shelf 不实现容器槽位。[shelf.lua:9-20](../../../raw/current/scripts/components/shelf.lua#L9-L20)
- 优先使用：需要“放一件、取一件”的展示或寄存建筑，参照鸟笼（单槽 inventory＋生命周期回调）或 rat gym（取物权限过滤）。[birdcage.lua:628-633](../../../raw/current/scripts/prefabs/birdcage.lua#L628-L633) [rat_gym.lua:305-309](../../../raw/current/scripts/prefabs/rat_gym.lua#L305-L309)
- 重要副作用：给 `cantakeitem`／`itemonshelf` 赋值会即时增删 `takeshelfitem` 标签，客户端据此显示 `TAKEITEM`；`TakeItem` 会把物品移入取者背包或丢到地面。[shelf.lua:1-7](../../../raw/current/scripts/components/shelf.lua#L1-L7) [actions.lua:3401-3407](../../../raw/current/scripts/actions.lua#L3401-L3407)
- 修改边界：字段与回调只在服务端有意义；组件没有 `OnSave`／`OnLoad`，重载后要由宿主从 inventory 重新 `PutItemOnShelf`，且取物的丢弃分支假定宿主有 inventory。[shelf.lua:42-66](../../../raw/current/scripts/components/shelf.lua#L42-L66) [birdcage.lua:535-542](../../../raw/current/scripts/prefabs/birdcage.lua#L535-L542)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `cantakeitem` | boolean，默认 false | Class 属性 setter 与 `itemonshelf` 共用 `ontakeshelfitem`：仅当两者都非空才 `AddTag("takeshelfitem")`，否则移除；任何赋值都会重算标签。[shelf.lua:9-20](../../../raw/current/scripts/components/shelf.lua#L9-L20) |
| `itemonshelf` | 物品实体或 nil | 同一属性 setter；只保存引用，不移动实体，也不校验是否可展示。[shelf.lua:18-19](../../../raw/current/scripts/components/shelf.lua#L18-L19) |
| `PutItemOnShelf(item)` | 无返回值 | 先写 `itemonshelf`，再回调 `onshelfitemfn(inst, item)`；不从 inventory／container 移入，宿主需自行把物品放进 inventory（通常先 `GiveItem`）。[shelf.lua:34-40](../../../raw/current/scripts/components/shelf.lua#L34-L40) |
| `TakeItem(taker)` | taker 实体或 nil | 仅 `cantakeitem` 且 `itemonshelf` 非空时继续；可选 `takeitemtstfn(inst, taker, item)` 返回 false 则静默返回；回调在移走前执行；taker 有 inventory 时宿主 `inventory:RemoveItem` 后 `GiveItem(pos)`，否则宿主 `inventory:DropItem`；最后清空引用。[shelf.lua:42-66](../../../raw/current/scripts/components/shelf.lua#L42-L66) |
| `SetOnShelfItem(fn)`／`SetOnTakeItem(fn)` | `fn(inst, item)`／`fn(inst, taker, item)` | 写入 `onshelfitemfn`／`ontakeitemfn`；构造里初始化的 `onitemtakenfn` 是另一个名字，不参与取物。[shelf.lua:26-32](../../../raw/current/scripts/components/shelf.lua#L26-L32) |
| `takeitemtstfn` | `fn(inst, taker, item) -> boolean` | 无 setter，只能直接赋值；原版 rat gym／carrat scale 用它限制非玩家或非训练者取物（训练者离线时放行）。[rat_gym.lua:142-164](../../../raw/current/scripts/prefabs/rat_gym.lua#L142-L164) [yotc_carrat_scale.lua:345-349](../../../raw/current/scripts/prefabs/yotc_carrat_scale.lua#L345-L349) |
| `GetDebugString()` | string | 空槽返回 `""`，否则返回 `"prefab: Can/Cannot be taken"`。[shelf.lua:68-75](../../../raw/current/scripts/components/shelf.lua#L68-L75) |

## 典型接入

### 鸟笼：单槽展示与生命周期钩子（原版宿主）

- 前提与选择：`prefabs/birdcage.lua`，注册名 `birdcage`；master sim 中先加 `inventory`（`maxslots = 1`）再加 `shelf`，展示槽与唯一 inventory 槽一一对应。[birdcage.lua:628-633](../../../raw/current/scripts/prefabs/birdcage.lua#L628-L633) [birdcage.lua:656](../../../raw/current/scripts/prefabs/birdcage.lua#L656
- 放入：饿死、中毒的掉落先 `GiveItem` 再 `PutItemOnShelf`；鸟腐坏时物品已在 inventory，只补一次 `Put`。Put 触发 `OnGetShelfItem`，移除 `occupiable`、置 `cantakeitem = true`，并监听物品 `perished`。[birdcage.lua:416-421](../../../raw/current/scripts/prefabs/birdcage.lua#L416-L421) [birdcage.lua:437-440](../../../raw/current/scripts/prefabs/birdcage.lua#L437-L440) [birdcage.lua:479-486](../../../raw/current/scripts/prefabs/birdcage.lua#L479-L486)
- 取出：`TAKEITEM` 调 `TakeItem`；`OnLoseShelfItem` 在物品移走前解绑 `perished`、恢复 `occupiable`，随后组件清空引用并移除标签。[birdcage.lua:488-500](../../../raw/current/scripts/prefabs/birdcage.lua#L488-L500) [shelf.lua:49-60](../../../raw/current/scripts/components/shelf.lua#L49-L60)
- 存档：组件不持久化，`OnLoadPostPass` 从 inventory 第 1 格重新 Put，恢复展示槽与标签。[birdcage.lua:535-542](../../../raw/current/scripts/prefabs/birdcage.lua#L535-L542)
- 端与撤销：单槽、整件（可整堆）；只服务端写字段。`RemoveComponent("shelf")` 会走 `OnRemoveFromEntity` 移除标签，但不会搬走 inventory 里的物品。[shelf.lua:22-24](../../../raw/current/scripts/components/shelf.lua#L22-L24)

### 取物权限：rat gym 与 carrat scale（原版宿主）

- 前提：`prefabs/rat_gym.lua` 生成 `yotc_carrat_gym_{direction,speed,reaction,stamina}` 四个注册名；`prefabs/yotc_carrat_scale.lua` 注册名 `yotc_carrat_scale`。均在加组件后设 `cantakeitem = true` 并挂 `takeitemtstfn`。[rat_gym.lua:305-309](../../../raw/current/scripts/prefabs/rat_gym.lua#L305-L309) [rat_gym.lua:374-388](../../../raw/current/scripts/prefabs/rat_gym.lua#L374-L388) [yotc_carrat_scale.lua:345-349](../../../raw/current/scripts/prefabs/yotc_carrat_scale.lua#L345-L349)
- 规则：`taker == nil` 允许（紧急落地），非玩家拒绝；训练者在线时只有其 `userid` 可拿，训练者不在服务器时任何人可取；拒绝时向玩家提示，`TakeItem` 直接返回，槽与标签不变。[rat_gym.lua:142-164](../../../raw/current/scripts/prefabs/rat_gym.lua#L142-L164) [shelf.lua:45-47](../../../raw/current/scripts/components/shelf.lua#L45-L47)
- 撤销：置 `cantakeitem = false` 即隐藏取物动作；若要保留物品，先自行 `DropItem` 再移除组件，否则物品留在 inventory 中。[shelf.lua:1-7](../../../raw/current/scripts/components/shelf.lua#L1-L7)

### 直接挂引用：食人花（原版宿主）

- `prefabs/lureplant.lua` 注册名 `lureplant`；把诱饵实体直接赋给 `itemonshelf`（仍经过属性 setter，只改引用），诱饵本身由 `SelectLure` 从 inventory 选出，或生成后 `GiveItem` 再返回。[lureplant.lua:44-50](../../../raw/current/scripts/prefabs/lureplant.lua#L44-L50) [lureplant.lua:189-194](../../../raw/current/scripts/prefabs/lureplant.lua#L189-L194)
- 取出时 `RemoveItem` 找不到该物品也不报错，随后 `GiveItem` 给取者；消化逻辑用 `item ~= shelf.itemonshelf` 豁免展示中的诱饵，只有堆叠大于 5 才消化。[lureplant.lua:208-212](../../../raw/current/scripts/prefabs/lureplant.lua#L208-L212) [inventory.lua:1390-1393](../../../raw/current/scripts/components/inventory.lua#L1390-L1393)
- 限制：只有 `taker` 为 nil 或没有 inventory 时才走 `DropItem` 的 else 分支，该分支直接读宿主 inventory；正常玩家取物走 `GiveItem`，无 inventory 的宿主只在这种空手路径下报错。展示实体被移除时要同步清空 `itemonshelf`，否则留下悬空引用。[shelf.lua:62-63](../../../raw/current/scripts/components/shelf.lua#L62-L63)

### 最小接线骨架

```lua
-- 目标：建筑 prefab 的 master_postinit（服务端）；只是接线顺序示意
inst:AddComponent("inventory")
inst.components.inventory.maxslots = 1
inst:AddComponent("shelf")
inst.components.shelf:SetOnShelfItem(function(inst, item)
    inst.components.shelf.cantakeitem = true
end)
inst.components.shelf:SetOnTakeItem(function(inst, taker, item)
    -- 在物品移走前解绑 perished 等监听
end)
inst.components.inventory:GiveItem(item)
inst.components.shelf:PutItemOnShelf(item)
```

- 调整与验证：骨架省略了鸟笼的 `occupiable` 与腐坏清理，不能直接替代原版宿主；单位是“一个引用／一个 inventory 槽”。[birdcage.lua:628-633](../../../raw/current/scripts/prefabs/birdcage.lua#L628-L633)

## 常见陷阱与限制

- **`cantakeitem = false` 不等于安全**：`TakeItem` 静默返回，但 `ACTIONS.TAKEITEM.fn` 调用后无条件 `return true`，动作层无法区分成功；权限应放在 `takeitemtstfn`，并用标签挡住动作入口。[actions.lua:3401-3407](../../../raw/current/scripts/actions.lua#L3401-L3407) [shelf.lua:43-47](../../../raw/current/scripts/components/shelf.lua#L43-L47)
- **回调字段名不一致**：构造初始化 `cantakeitemfn`、`onitemtakenfn`，而 `TakeItem` 实际读 `ontakeitemfn`；写 `onitemtakenfn` 不生效，公开入口只有 `SetOnTakeItem`。[shelf.lua:11-13](../../../raw/current/scripts/components/shelf.lua#L11-L13) [shelf.lua:30-32](../../../raw/current/scripts/components/shelf.lua#L30-L32) [shelf.lua:49-51](../../../raw/current/scripts/components/shelf.lua#L49-L51)
- **Put 不做搬运**：只赋值＋回调；不先 `GiveItem` 就 Put，会让引用指向不在宿主 inventory 的物品，`TakeItem` 的 `RemoveItem` 空操作，存档只剩 inventory 内容。[shelf.lua:34-40](../../../raw/current/scripts/components/shelf.lua#L34-L40) [inventory.lua:1390-1403](../../../raw/current/scripts/components/inventory.lua#L1390-L1403)
- **没有自身存档**：shelf 无 `OnSave`／`OnLoad`，重载后 `itemonshelf` 与标签都要宿主用 `OnLoadPostPass` 重建；只在客户端赋值不会同步。[birdcage.lua:535-542](../../../raw/current/scripts/prefabs/birdcage.lua#L535-L542) [shelf.lua:9-24](../../../raw/current/scripts/components/shelf.lua#L9-L24)
- **容器协作边界**：shelf 不提供容器槽位、堆叠或打开 UI，物品仍归 `inventory`／`container`；多槽、堆叠上限与合并规则见 [[reference/components/container]] 和 [[reference/components/stackable]]，不要给 shelf 增加槽位假设。
- **重复 Put 不清理旧监听**：Put 只覆盖引用并再触发一次 `onshelfitemfn`；宿主在回调里注册 `perished` 等监听时，要在取出或替换前自行解绑，否则旧物品监听残留。[shelf.lua:34-40](../../../raw/current/scripts/components/shelf.lua#L34-L40) [birdcage.lua:484-490](../../../raw/current/scripts/prefabs/birdcage.lua#L484-L490)
- **消费者依赖标签而非字段**：猪人找食物读 `takeshelfitem` 标签、`itemonshelf`、`cantakeitem` 与 edible；展示物品不可食用时不会成为目标，别把 shelf 当通用库存接口。[pigbrain.lua:69-79](../../../raw/current/scripts/brains/pigbrain.lua#L69-L79) [componentactions.lua:782-786](../../../raw/current/scripts/componentactions.lua#L782-L786)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的漂移文件（actions.lua、componentactions.lua、prefabs/lureplant.lua）对照 build-26148036 更新报告（R-01/R-04/P-67）逐条核验：TAKEITEM.fn 与 shelf 候选收集逐行未变，lureplant.lua 改动仅为 :349 处标签调整（移除 lifedrainable、新增 plantcreature／nolife，不影响本页展示槽与消化豁免结论）；锚点重定位 2 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-21，steam-buildid:25643519；通读 `components/shelf.lua`（77 行）及 `birdcage`、`rat_gym`、`yotc_carrat_scale`、`lureplant` 宿主接线，核对 `actions`、`componentactions`、`pigbrain`、`inventory` 的消费者；正文源码链接 49 处，结论只覆盖这些路径。
- 校验：`validate_page.py` 0 error／0 warning；`validate_source_links.py --strict` 0 issue；接线骨架经 `parser_probe --syntax-only` 通过。
- 游戏运行：未运行；多人同时取物、重载后标签恢复、皮肤／可堆叠物品的展示表现未实测。

相关：[[reference/components/container]]；[[reference/components/stackable]]；[[reference/components/inventoryitem]]。
