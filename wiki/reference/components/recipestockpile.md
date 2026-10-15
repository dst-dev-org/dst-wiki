---
title: recipestockpile：配方库存记账与逐次补货
created: 2026-10-27
updated: "2026-10-28"
type: component
summary: 按配方名维护可消费次数、单个补货任务与满缺回调，并明确它不自动阻止制作。
aliases: [recipestockpile, 配方库存, 制作配额, 定时补货]
tags: [component, recipe, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: [components/recipestockpile.lua, components/builder.lua]
reliability: verified
validation: source-reviewed
status: active
---

# recipestockpile

## 快速判断

- 职责与适用：宿主按配方名保存库存次数；收到自身的 `builditem` 后取 `data.recipe.name` 扣一次。这里的库存是数值记录，不是物品实体或材料仓库。[recipestockpile.lua:2-11](../../../raw/current/scripts/components/recipestockpile.lua#L2-L11) [recipestockpile.lua:113-124](../../../raw/current/scripts/components/recipestockpile.lua#L113-L124)
- 优先使用：需要“消耗一次后定时补一个”的记账，可看 `SetupItem` 与 `OnItemCrafted`；需要禁止缺货制作，必须另接制作检查。组件没有注册门禁；Builder 的制作前入口另有 `recipe.canbuild`。[recipestockpile.lua:32-52](../../../raw/current/scripts/components/recipestockpile.lua#L32-L52) [builder.lua:657-688](../../../raw/current/scripts/components/builder.lua#L657-L688)
- 重要副作用：配置、补货和补满会调用 `onrestockfn`，清空或消费到零会调用 `onemptyfn`；回调实际效果完全由调用方提供。[recipestockpile.lua:17-52](../../../raw/current/scripts/components/recipestockpile.lua#L17-L52) [recipestockpile.lua:90-124](../../../raw/current/scripts/components/recipestockpile.lua#L90-L124)
- 修改边界：组件无网络字段、客户端分支或自动安装逻辑；本快照按 `recipestockpile` 全名检索未找到原版宿主／外部调用点，不将它描述为已启用的通用制作限额系统。[recipestockpile.lua:6-15](../../../raw/current/scripts/components/recipestockpile.lua#L6-L15)

## 关键接口

### 配置与回调

`SetupItem(data, start_restock_timer)` 仅在 `stock[data.recipe]` 尚不存在时工作，深拷贝配置；重复调用不会更新已有配方。`data` 至少需要 `recipe` 键和可计算的 `max`，可配置 `num`、`restocktime`、两个回调。`num` 缺省取 `max`，显式 `0` 会保留；没有范围校验或钳制。[recipestockpile.lua:32-52](../../../raw/current/scripts/components/recipestockpile.lua#L32-L52)

- 初始数量大于零时调用 `onrestockfn(inst, recipe, num, max)`，否则调用 `onemptyfn(inst, recipe)`；前者不只表示“刚补货”。
- `start_restock_timer` 为真、存在补货间隔且当前数量小于上限时，才在配置阶段启动任务；参数为假不会自动为初始缺口补货。
- `stock` 以 `data.recipe` 索引，保存时又使用记录中的 `recipe` 字段；应保持键与配置名一致。

以上均见 [recipestockpile.lua:32-52](../../../raw/current/scripts/components/recipestockpile.lua#L32-L52)、[recipestockpile.lua:126-135](../../../raw/current/scripts/components/recipestockpile.lua#L126-L135)。

| 接口 | 条件与副作用 | 依据 |
|---|---|---|
| `OnItemCrafted(recipe)` | 已登记且 `num > 0` 才减一；有间隔且无任务才启动补货，到零调用缺货回调。零库存再次收到事件会直接跳过。 | [recipestockpile.lua:113-124](../../../raw/current/scripts/components/recipestockpile.lua#L113-L124) |
| `FullyRestockItem(recipe)` | 已登记就赋值 `num = max`、取消当前任务，并调用补货回调，即使原本已满。 | [recipestockpile.lua:90-102](../../../raw/current/scripts/components/recipestockpile.lua#L90-L102) |
| `HasAnyStock()` | 任意记录 `num > 0` 返回真，否则返回假；不是指定配方的制作许可。 | [recipestockpile.lua:104-111](../../../raw/current/scripts/components/recipestockpile.lua#L104-L111) |
| `RemoveStock(recipe, allow_restock)`／`RemoveAllStock(allow_restock)` | 仅处理当前数量大于零的记录：置零、取消原任务，再尝试补货分支与缺货回调；没有删除记录。补货分支存在下述源码疑点。 | [recipestockpile.lua:54-88](../../../raw/current/scripts/components/recipestockpile.lua#L54-L88) |
| `GetDebugString()` | 按记录展示配方、数量／上限与任务剩余时间。 | [recipestockpile.lua:152-162](../../../raw/current/scripts/components/recipestockpile.lua#L152-L162) |

## 关键流程

### 每次任务只补一个

`OnRestock` 将数量设为 `min(num + 1, max)`；未满就再排一次完整 `restocktime`，满了将 `timer` 置空，然后通知补货回调。连续制作共享已有任务，不会每次消费重置计时；也没有按离线时间一次性批量补满的代码。[recipestockpile.lua:17-30](../../../raw/current/scripts/components/recipestockpile.lua#L17-L30) [recipestockpile.lua:113-124](../../../raw/current/scripts/components/recipestockpile.lua#L113-L124)

### 保存、恢复与组件卸载

`OnSave` 只保存每个配方的 `num` 与任务剩余时间，不保存上限、补货间隔或回调。`OnLoad(data)` 只恢复已由宿主配置好的键：覆盖数量、取消已有任务，有保存计时才重建任务；不会重新执行满缺回调，也不会为“有缺口但无保存计时”的条目主动启动任务。因此宿主须先恢复配置，并自行处理加载后展示或权限的刷新。[recipestockpile.lua:126-150](../../../raw/current/scripts/components/recipestockpile.lua#L126-L150)

`OnRemoveFromEntity` 只撤销宿主的 `builditem` 监听，没有遍历或取消补货任务。单独移除组件时，不应把这个钩子当成完整的任务清理协议；实体整体删除时的调度器行为不由本页推断。[recipestockpile.lua:13-15](../../../raw/current/scripts/components/recipestockpile.lua#L13-L15)

## 典型接入

### 对接制作事件时区分事件计数与制作门禁

Builder 在生成可入背包产品、确认宿主有 inventory 并移除材料后，向制作者及有效的当前原型站各发一次 `builditem`。因此组件应装在实际接收所需事件的对象上，不能仅凭配方名假设任意实体都会收到事件。[builder.lua:723-744](../../../raw/current/scripts/components/builder.lua#L723-L744)

这条事件按一次制作发出，随后才处理 `recipe.numtogive` 等产品数量；组件每个事件扣一，不按产品堆叠数量扣除。非 inventoryitem 产品走 `buildstructure`，本组件没有监听它；manufactured 分支也提前返回，不经过该 `builditem` 发出点。[builder.lua:717-747](../../../raw/current/scripts/components/builder.lua#L717-L747) [builder.lua:817-835](../../../raw/current/scripts/components/builder.lua#L817-L835)

库存为零时，组件只是停止继续扣数，不会撤销已制作产品。真正的制作限制需在制作前独立实现并同步所需展示；Builder 的 `HasIngredients` 有 `getlimitedrecipecount` 检查，`DoBuild` 也调用 `recipe.canbuild`，但它们没有自动读取此组件。[builder.lua:657-688](../../../raw/current/scripts/components/builder.lua#L657-L688) [builder.lua:918-948](../../../raw/current/scripts/components/builder.lua#L918-L948)

## 常见陷阱与限制

- **清空后允许补货的源码疑点：** 两个清空接口都读取 `time.restocktime`，但本文件未定义 `time`；若该全局为 nil，`allow_restock` 为真时将在置零／取消任务后报错，缺货回调也不会执行。`RemoveAllStock` 的调度实参又使用未在该函数定义的 `recipe`，而非循环键或 `stock.recipe`。这是实际源码写法，不替换成推测中的正确实现，也不推荐该分支为可靠补货入口。[recipestockpile.lua:54-88](../../../raw/current/scripts/components/recipestockpile.lua#L54-L88)
- **已空记录不参与清空：** 两接口均在 `num > 0` 内才取消任务；因此不能依靠它们统一取消所有补货任务，也不能据此构造安全卸载流程。[recipestockpile.lua:54-88](../../../raw/current/scripts/components/recipestockpile.lua#L54-L88)
- **恢复不等于重新配置：** 保存数据没有完整配置，未知配方键被忽略；`OnLoad` 也不校验 `data` 是否为 nil。调用者应遵守有效数据及预先配置的前提。[recipestockpile.lua:126-150](../../../raw/current/scripts/components/recipestockpile.lua#L126-L150)

## 验证记录

- 2026-10-27，`steam-buildid:26148036`：全文审阅组件，核对 Builder 的 `DoBuild`、`HasIngredients` 及事件分支；覆盖配置、回调、逐次补货、保存恢复、卸载与上述疑点。全名定向检索未找到原版安装点，不把无命中扩大为其他版本的结论。
- 游戏运行：未运行；未验证定时任务在单独移除组件后的实际调度、异常分支现场或模组接入。
- 结构检查：页面格式与严格源码链接检查通过；无新增完整 Lua 示例，未执行 Lua 解析测试。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。