---
title: "dryer"
created: "2026-10-24"
updated: "2026-10-28"
type: component
summary: "识别旧单件晾晒器的输入消费、干制后腐败、雨天暂停与存读逻辑，迁移时优先使用当前容器式 dryingrack。"
aliases: ["dryer", "旧单件晾晒", "旧晾肉架", "旧晾晒存档", "StopDrying"]
tags: [component, food, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/dryer.lua", "components/dryable.lua", "prefabs/meatrack2.lua", "prefabs/hermitcrab.lua", "prefabs/staff.lua", "prefabs/staff_common.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# dryer

## 快速判断

- 职责与适用：旧单挂点处理器，用字符串保存一份原料／产物并删除输入，先计晾干时间，再计成品腐败时间；不持有容器中的活物品。[构造](../../../raw/current/scripts/components/dryer.lua#L17-L39) [开始](../../../raw/current/scripts/components/dryer.lua#L176) [完成](../../../raw/current/scripts/components/dryer.lua#L162)
- 优先使用：当前新架子选 [[reference/components/dryingrack]]，原料配置仍选 [[reference/components/dryable]]；原版普通架实际安装 container＋dryingrack，hermitcrab 对 dryer 的兼容分支明确标为 deprecated。[当前工厂](../../../raw/current/scripts/prefabs/meatrack2.lua#L325-L334) [旧兼容注释](../../../raw/current/scripts/prefabs/hermitcrab.lua#L1074-L1076)
- 重要副作用：`StartDrying` 成功会直接 Remove 输入实体；`StopDrying` 不是无损取消，可能立即完成或腐坏。`Harvest`／`DropItem` 即使产物 Spawn 失败也继续清空记录。[输入消费](../../../raw/current/scripts/components/dryer.lua#L176-L217) [停止分支](../../../raw/current/scripts/components/dryer.lua#L220-L248) [收获](../../../raw/current/scripts/components/dryer.lua#L314-L346)
- 修改边界：保留实现和旧动作仍可被旧模组使用，但不能据此推定当前 meatrack 挂载 dryer；组件自身无端校验或网络字段，兼容接入须由目标模组核验服务端宿主、资源和实际调用。[旧动作](../../../raw/current/scripts/actions.lua#L2171-L2186) [当前注册](../../../raw/current/scripts/prefabs/meatrack2.lua#L584-L591)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `CanDry(item)` | 仅查 product=nil、item 非 nil 且有 dryable；**不检查 perishable**、堆叠、资源或原料配置。StartDrying 随后无条件读 perishable:GetPercent，不能把 CanDry=true 当完整输入验证。[初筛](../../../raw/current/scripts/components/dryer.lua#L112-L113) [实际读取](../../../raw/current/scripts/components/dryer.lua#L181-L188) |
| `StartDrying(item)` | 返回 true／false；记录 prefab、两种 build、新鲜度、foodtype、产物与秒数。配置关键值缺失时清状态返回 false；成功删除传入实体，再按天气安排任务并回调。[实现](../../../raw/current/scripts/components/dryer.lua#L176-L217) |
| `IsDrying()`／`IsDone()` | 前者只查 ingredient 非 nil；后者查 product 非 nil 且 ingredient=nil。暂停的原料仍算 IsDrying。字段属性同时管理 candry／drying／dried 标签。[状态查询](../../../raw/current/scripts/components/dryer.lua#L116-L121) [标签](../../../raw/current/scripts/components/dryer.lua#L1-L14) |
| `GetTimeToDry()`／`GetTimeToSpoil()`／`IsPaused()` | 运行时取截止时刻减 GetTime，暂停时取 remainingtime；干制或腐坏阶段不相符时返回 0。IsPaused 仅查 remainingtime 是否非 nil，不表示外部所有时间推进均停。[读取](../../../raw/current/scripts/components/dryer.lua#L124-L133) |
| `Pause()`／`Resume()` | 互换任务和剩余秒数，适用于干制或成品腐坏。Resume 不自行复核天气，Pause 不撤销天气监听；不能据此构建独立于天气的永久禁用策略。[实现](../../../raw/current/scripts/components/dryer.lua#L251-L270) |
| `protectedfromrain` | 初始 nil；开始／恢复时决定是否忽略雨与注册监听。它是普通字段，没有自动刷新现有任务／监听的 setter；当前新架子的免雨方案见 dryingrack。[开始门禁](../../../raw/current/scripts/components/dryer.lua#L205-L210) [监听](../../../raw/current/scripts/components/dryer.lua#L44-L81) |
| `SetStartDryingFn`／`SetDoneDryingFn`／`SetOnHarvestFn` | 前两者回调 `(rack, prefab, build)`，最后为 `(rack)`；无现代 dryingrack 的 slot 参数。读档会重放开始／完成回调，不应无条件奖励玩家。[注册](../../../raw/current/scripts/components/dryer.lua#L98-L107) [加载回调](../../../raw/current/scripts/components/dryer.lua#L430-L437) |
| `StopDrying(reason)` | product=nil 时无操作；reason="fire" 时生成 product 后清空，即使还在干制；否则原料态立即 DoDry，成品态立即 DoSpoil。不是“取消并归还原料”。[实现](../../../raw/current/scripts/components/dryer.lua#L220-L248) |
| `DropItem()`／`Harvest(harvester)` | DropItem 可弹出原料或成品，按剩余时间重算新鲜度；Harvest 要求已完成且 harvester 有 inventory，再 GiveItem。两者最终取消任务、清状态、调 onharvest；不保存原物品实例身份。[弹出](../../../raw/current/scripts/components/dryer.lua#L273-L311) [收获](../../../raw/current/scripts/components/dryer.lua#L314-L346) |

### 干制、腐坏与天气怎样衔接

开始时若下雨且无 rainimmunity／protectedfromrain，保留剩余干制时间但不启动任务；未保护者监听 israining、gainrainimmunity、loserainimmunity 来暂停或恢复。雨的判断针对宿主，不是原料物品的湿度。[雨判定与事件](../../../raw/current/scripts/components/dryer.lua#L44-L71) [开始时接线](../../../raw/current/scripts/components/dryer.lua#L207-L211)

DoDry 清 ingredient、停止看雨，启动 `TUNING.PERISH_PRESERVED` 的成品腐坏计时，再调完成回调。DoSpoil 清空状态、在世界生成 spoiled_food，并调 onharvest；没有把成品作为真实 perishable 实体一直挂在架上。[完成](../../../raw/current/scripts/components/dryer.lua#L162-L173) [腐坏](../../../raw/current/scripts/components/dryer.lua#L138-L159)

## 典型接入

### 先判断正在维护旧架还是当前容器架

当前 `meatrack`、`meatrack_hermit`、`meatrack_hermit_multi` 均由 MakeMeatRack 安装 dryingrack，分别使用各自容器布局；新造架子的完整天气／显示／存读组装复用 [[reference/components/dryingrack#典型接入]]，不要将其示例中的组件名直接改成 dryer。[统一安装](../../../raw/current/scripts/prefabs/meatrack2.lua#L325-L334) [实际注册](../../../raw/current/scripts/prefabs/meatrack2.lua#L584-L591)

仍需兼容 dryer 的旧模组，可按原版保留的动作识别前提：dryable 的 USEITEM 收集器要求目标 candry、原料 dryable 且目标未 burnt；DRY 动作先从人物库存取料，StartDrying 拒绝时尝试归还，成功后推世界成就事件。已完成的 dried 标签走 HARVEST。组件直接调用不会代做库存移出／单件拆分、动作候选或成就事件。[候选](../../../raw/current/scripts/componentactions.lua#L1140-L1144) [取料与执行](../../../raw/current/scripts/actions.lua#L2171-L2186) [收获候选](../../../raw/current/scripts/componentactions.lua#L361-L364)

本快照 scripts 的 `.lua` 中，精确双／单引号 AddComponent("dryer") 挂载检索未命中；仍有上述动作、冰杖 DropItem 与 hermitcrab 兼容读取。不排除动态挂载和外部模组，本页不提供重建旧架的 demo。[法杖兼容](../../../raw/current/scripts/prefabs/staff_common.lua#L696-L697) [NPC兼容](../../../raw/current/scripts/prefabs/hermitcrab.lua#L1074-L1076)

### 旧存档如何进入现行架子

组件 OnSave 保存原料／产物名、build、foodtype、新鲜度和剩余时间；OnLoad 重建计时，并按原料／成品状态调用对应显示回调。宿主须在加载前装好回调，不能把回调重放当一次新操作。[组件存读](../../../raw/current/scripts/components/dryer.lua#L390-L439)

当前 meatrack2 自己的 OnLoad 另识别旧 `data.dryer`：生成原料或成品、放到第 2 槽；原料附 dryingrack_drytime，成品可恢复 dried_buildfile。它把旧记录迁入新容器机制，并非重新安装旧组件。新鲜度恢复与具体兼容条件以该分支为准，不能把两个组件的数据结构直接互换。[宿主迁移](../../../raw/current/scripts/prefabs/meatrack2.lua#L176-L203)

## 常见陷阱与限制

- **直接传整堆／任意 dryable：** StartDrying 删除整个入参实体，只记录一个产物；没有 perishable 还会直接访问失败。原动作先移出库存原料，组件本身不保证拆成一件。[实际消费](../../../raw/current/scripts/components/dryer.lua#L181-L205) [动作取料](../../../raw/current/scripts/actions.lua#L2177-L2179)
- **暂停后离屏时间完全冻结：** LongUpdate 仍扣剩余时间，可能完成干制后继续推进成品腐败，最后再按当前雨况安排任务；不能据 Pause 推导离屏语义。[LongUpdate](../../../raw/current/scripts/components/dryer.lua#L353-L384)
- **DropItem／Harvest=true 保证收到产物：** SpawnPrefab 失败仍清内部状态并回调；true 只表示通过前提并完成处理，非物品交付保证。[弹出](../../../raw/current/scripts/components/dryer.lua#L273-L311) [收获](../../../raw/current/scripts/components/dryer.lua#L314-L346)
- **旧档缺新鲜度就补1：** dryer.OnLoad 实际回填 `ingredientperish=100`，而正常开始取 GetPercent；这是保留的旧兼容代码，不作为新模组的新鲜度字段约定，也不默默改写它。[兼容赋值](../../../raw/current/scripts/components/dryer.lua#L406-L415)
- **删组件就归还食材：** 移除只取消任务／雨监听、去状态标签，不执行 DropItem 或 Harvest。旧宿主应在销毁前明确决定返还或丢失；当前新架的容器清理另有协议。[移除钩子](../../../raw/current/scripts/components/dryer.lua#L86-L94)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 变更文件 actions.lua／componentactions.lua／staff.lua／hermitcrab.lua 对照更新报告 R-01/R-04/P-112 与 hunk 地图逐 hunk 核验。锚点重定位 7 处：DRY 动作 3 处 +3 行移（actions.lua:2168-2183、:2174-2176），dryable／HARVEST 候选与 hermitcrab 兼容读行号不变；staff.lua 的 destroystructure 整体迁出至 staff_common.lua（P-112），"冰杖 DropItem" 锚点跨文件移至 staff_common.lua:696-697，sources 随之增补。DRY／dryable／HARVEST 候选无语义变化，R-01 燃烧 torch 拾取放宽不触及本页引用。未覆盖：运行验证。

- 源码审阅：2026-10-24，Steam buildid 25643519；完整核验 dryer、原料契约、保留动作与旧读取、当前架子安装及旧 data.dryer 迁移。当前 dryingrack／dryable 的已核验新机制只复用对应页，不扩称整套新架子本次重审。
- 结构检查：本页格式、来源、本地链接和源码锚点严格检查通过；本批两页共84处源码引用无错误／警告。无新增完整 Lua 示例，导航登记由主代理整合检查。
- 游戏运行：未运行；旧模组动态挂载、堆叠消费、雨况切换、离屏、产物生成失败及旧档新鲜度边界未实测。

相关：[[reference/components/dryingrack]]、[[reference/components/dryable]]、[[reference/components/perishable]]。
