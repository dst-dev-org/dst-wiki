---
title: "digester"
created: "2026-10-24"
updated: "2026-10-28"
type: component
summary: "按周期从宿主 inventory 主栏选择物品类型并消耗一件，复用食人花的消化筛选与任务启停边界。"
aliases: ["digester", "食人花消化", "库存定时消耗", "消化筛选", "TryDigest"]
tags: [component, item, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/digester.lua", "components/inventory.lua", "prefabs/lureplant.lua", "entityscript.lua", "scheduler.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# digester

## 快速判断

- 职责与适用：定期枚举宿主的 `inventory.itemslots`，选一个候选 prefab 后消耗一件；不是箱子的 container 处理器，也不调用 eater 或生成产物。原版食人花在服务端安装。[完整消化逻辑](../../../raw/current/scripts/components/digester.lua#L14-L38) [宿主端界与库存](../../../raw/current/scripts/prefabs/lureplant.lua#L358-L377) [挂载](../../../raw/current/scripts/prefabs/lureplant.lua#L396-L397)
- 优先使用：已有食人花规则用 `itemstodigestfn(inst,item)` 扩展候选资格；实际收纳范围与移除协议复用 [[reference/components/inventory]]，container 的批量删除另见 [[reference/components/incinerator]]。[筛选入口](../../../raw/current/scripts/components/digester.lua#L18-L26)
- 重要副作用：构造立即注册周期任务，默认每 20 秒尝试；有 inventory 却没有合格主栏候选时取消任务，后续 `gotnewitem` 才重新安排。首次尝试等一个周期，不是添加即吞物品。[构造／重启](../../../raw/current/scripts/components/digester.lua#L1-L10) [自动停用](../../../raw/current/scripts/components/digester.lua#L29-L36) [初次调度](../../../raw/current/scripts/scheduler.lua#L295-L299)
- 修改边界：筛选记录的是 prefab 名，真正消费又按名字查找；不能将筛选回调当成最终逐实例权限保护。组件没有产物、存档或移除清理钩子。[选型与消费](../../../raw/current/scripts/components/digester.lua#L18-L31) [消费查找](../../../raw/current/scripts/components/inventory.lua#L1772-L1797)

## 关键接口

| 接口／字段 | 参数或约定 | 作用与证据 |
|---|---|---|
| `TryDigest()` | 无参数，无显式成功返回 | 有 inventory 才遍历其主栏；先排除 `irreplaceable`，再执行可选筛选，按候选数组随机抽一个 prefab，调用 ConsumeByName(prefab,1)。不报告实际消耗对象。[实现](../../../raw/current/scripts/components/digester.lua#L14-L38) |
| `itemstodigestfn` | `fn(inst,item)` 返回真值允许 | 默认 nil，即除 irreplaceable 外均可入候选。只决定该实例是否贡献一个 prefab 候选，不传操作者、不覆盖最终消费查找。[筛选](../../../raw/current/scripts/components/digester.lua#L18-L26) |
| `digesttime` | 初值 20，单位秒 | 只在创建周期任务时读取；运行中修改字段不会重排现有任务。空候选取消后，下次 gotnewitem 建任务才使用新值。[任务创建](../../../raw/current/scripts/components/digester.lua#L3-L9) |
| `task` | 周期任务句柄或 nil | TryDigest 空候选分支取消并置 nil；缺 inventory 时函数直接结束，既不取消任务也不将句柄置 nil。[分支范围](../../../raw/current/scripts/components/digester.lua#L15-L38) |

候选权重是**合格槽位数**：同 prefab 在多个合格槽位会重复入数组，堆叠数量本身不增加该槽位权重。消费端按主栏槽序查同 prefab，堆叠足够时减一，否则移除相应实例；并非移除刚通过回调的那个对象。[候选](../../../raw/current/scripts/components/digester.lua#L18-L31) [减堆／移除](../../../raw/current/scripts/components/inventory.lua#L1756-L1766) [槽序](../../../raw/current/scripts/components/inventory.lua#L1778-L1785)

## 典型接入

### 参考食人花保存展示诱饵，消化其余库存

这是原版宿主组装指引，不是可独立加载的新生物 demo：

1. 食人花主模拟工厂先安装 shelf、inventory，再加 digester 并绑定 CanDigest；组件不替宿主创建库存、表现或生物行为。[配置顺序](../../../raw/current/scripts/prefabs/lureplant.lua#L373-L397)
2. CanDigest 对非展示物品返回真；若物品恰是 `shelf.itemonshelf`，只有可堆叠且数量大于 5 才允许。这个规则约束候选，不是全部同 prefab 实例的独占保护。[宿主规则](../../../raw/current/scripts/prefabs/lureplant.lua#L207-L212)
3. 收集眼球草库存属于 lureplant 的 CollectItems：先移出子体库存，再交入食人花 inventory。digester 只消费已有内容，不能单靠它获得远处拾物或眼球草行为。[收集入口](../../../raw/current/scripts/prefabs/lureplant.lua#L152-L167)
4. 原版 plantmeat 生成在 SelectLure 的另一个分支；不要把“消化一件”当作按材料转换成植物肉的配方系统。[原版生成](../../../raw/current/scripts/prefabs/lureplant.lua#L178-L194)

自定义筛选若只按 prefab 判断，通常与按名字消费的模型一致；若按皮肤、实例标签、品质或展示对象判断，必须另外设计按实际实例消费的机制，不能只替换 itemstodigestfn 就声称这些实例受到保护。

## 常见陷阱与限制

- **保护了某实例却仍被吃掉：** 同 prefab 的另一个实例可入候选，最终 ConsumeByName 按槽序消费，并不重跑筛选或检查 irreplaceable。不同实例具有不同资格时，原筛选不能提供最终授权保证。[记录名字](../../../raw/current/scripts/components/digester.lua#L18-L31) [最终消费](../../../raw/current/scripts/components/inventory.lua#L1778-L1785)
- **给箱子加组件期待消化容器内容：** 组件只读 inventory；只有 container 时每次都直接跳过，而且原周期任务继续存在。先选正确的持有模型，不把两种组件当成同名接口。[入口](../../../raw/current/scripts/components/digester.lua#L14-L18)
- **改 digesttime 就立即变速：** 已创建任务保留原周期；组件没有专用重启 API。直接 TryDigest 是立即尝试消费，不是更改任务周期的 setter。[构造](../../../raw/current/scripts/components/digester.lua#L1-L10) [直接执行](../../../raw/current/scripts/components/digester.lua#L14-L31)
- **移组件或取消一次 task 就完成清理：** 没有 OnRemoveFromEntity；构造的匿名 gotnewitem 监听仍捕获旧组件，task=nil 时还会新建任务。动态替换应设计完整任务和监听清理，不只 RemoveComponent 或一次 Cancel；整个宿主删除才会清事件和 pending tasks。[匿名监听](../../../raw/current/scripts/components/digester.lua#L5-L10) [移组件](../../../raw/current/scripts/entityscript.lua#L670-L683) [移实体](../../../raw/current/scripts/entityscript.lua#L1741-L1746)
- **存档恢复保留剩余消化秒数：** 组件无存读方法，工厂重新创建任务；库存持久化与消化计时是两件事，不承诺恢复上次周期进度。[完整定义](../../../raw/current/scripts/components/digester.lua#L1-L41)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 3 处（lureplant 装配段 +1、entityscript OnRemoveEntity 段 +3）；无结论修订，仅锚点迁移——digester.lua、inventory.lua、scheduler.lua 未变。依据：全局枢纽 diff 审阅＋变更清单命中 prefabs/lureplant.lua（本版 fn 内 -1+2 行）、entityscript.lua。
- 源码审阅：2026-10-24，Steam buildid 25643519；完整审阅 digester，核验食人花安装／展示诱饵规则／库存转交与植物肉生成、Inventory 按名消费、首次周期调度和宿主清理。库存其他接口复用已建 inventory 页，未全面审校食人花 AI／所有物品移除副作用。
- 结构检查：本页格式、来源、本地链接及源码锚点严格检查通过；无新增完整 Lua 示例，导航登记由本批统一检查。
- 游戏运行：未运行；同 prefab 实例筛选差异、重新入栏调度、动态卸载和存读时序未实测。
