---
title: "itemstore"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "把物品存为记录队列，按批重新生成；区分活体库存、恢复失败与跨分片事务的责任。"
aliases: ["物品记录队列", "延迟生成物品", "迁移物品暂存", "漂浮物打捞", "itemstore"]
tags: [component, item, data, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/itemstore.lua", "components/shardtransactionsteps.lua", "prefabs/oceanwhirlbigportal.lua", "entityscript.lua", "mainfunctions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# itemstore

## 快速判断

- 职责与适用：服务端保存物品记录的 FIFO 数组；AddItem 会删除原实体，暂存期间没有原物品组件持续运行。需要格子／现存实体库存时用 [[reference/components/container]]。[存入实现](../../../raw/current/scripts/components/itemstore.lua#L36-L50)
- 优先使用：原版 oceanwhirlbigportalexit 暂存迁移物品，拾取时按批恢复并分发；GetNumberOfItems 是记录条数，一条堆叠记录不等于一件。[宿主拾取](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L510-L526)、[计数](../../../raw/current/scripts/components/itemstore.lua#L7-L9)
- 重要副作用：GetFirstItems 会创建新实体并删除队首记录，事件通知发生在返回物品表前；组件不负责送入人物库存。[恢复实现](../../../raw/current/scripts/components/itemstore.lua#L11-L34)
- 修改边界：itemstore 不负责迁移事务、失败补偿或通用实体引用恢复；原版宿主在 mastersim 分支添加组件和监听。[宿主配置](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L549-L574)、[迁移接收](../../../raw/current/scripts/components/shardtransactionsteps.lua#L138-L165)

## 关键接口

| 接口／字段 | 签名或数据约束 | 条件、作用与证据 |
|---|---|---|
| `storeditemdatas` | 连续数组，初始 `{}` | 每项 `{item_record=记录, migrationdata=可选数据}`；不要留下空洞，计数与移队依赖 `#`。[结构](../../../raw/current/scripts/components/itemstore.lua#L1-L9)、[添加](../../../raw/current/scripts/components/itemstore.lua#L36-L50) |
| `GetNumberOfItems()` | 返回数组长度 | 计记录数，不展开 stackable 件数。[实现](../../../raw/current/scripts/components/itemstore.lua#L7-L9) |
| `AddItem(item)` | 有效、可保存的实体；无显式返回 | GetSaveRecord 后 Remove，再追加记录；不查 inventoryitem、容量、persist 标志或过滤，不保留 GetSaveRecord 第二返回值 references。[实现](../../../raw/current/scripts/components/itemstore.lua#L36-L43)、[记录返回](../../../raw/current/scripts/entityscript.lua#L308-L315) |
| `AddItemRecordAndMigrationData(record, migrationdata)` | 有效存档记录及可选迁移表；无显式返回 | 直接保存传入表，不生成／删除实体，不校验、不深拷贝、不去重。[实现](../../../raw/current/scripts/components/itemstore.lua#L45-L51) |
| `GetFirstItems(count)` | 非负整数；返回新实体数组 | 调用者保证 count 合法，组件无校验；最多恢复现有数量，移除对应队首并压紧数组。0 也发变更事件。[实现](../../../raw/current/scripts/components/itemstore.lua#L11-L34) |
| `itemstore_changedcount` | 宿主事件，无 payload | 两种追加及每次 GetFirstItems 发出；OnLoad 仅在恢复后非空时发出。消费者须重新查询数量。[追加事件](../../../raw/current/scripts/components/itemstore.lua#L36-L51)、[载入事件](../../../raw/current/scripts/components/itemstore.lua#L63-L72) |
| `OnSave()`／`OnLoad(data)` | `{itemdatas=数组}`；空队列不保存 | 载入只恢复记录表，不生成活体；没有自动清空／吐出记录的移组件或删除钩子。[保存到文件结尾](../../../raw/current/scripts/components/itemstore.lua#L53-L74) |

恢复时仅传 prefab、skinname、skin_id，以及 migrationdata.sessionid 构造的 creator，再 SetPersistData(record.data)；不调用 SpawnSaveRecord，不恢复记录位置／alt_skin_ids，也不传 newents。需要 GUID 引用、位置或额外皮肤信息的对象不能据此承诺完整往返。[恢复路径](../../../raw/current/scripts/components/itemstore.lua#L13-L22)、[完整存档路径对照](../../../raw/current/scripts/mainfunctions.lua#L464-L509)

## 典型接入

### 让原版漂浮物出口每次打捞最多恢复一条记录

完整代码放已启用模组的 `modmain.lua`，不新增资源。只包装 oceanwhirlbigportalexit 自身的 itemstore 方法，把本次恢复量上限设为 1；保留原拾取回调、队列事件、动画与 GiveLoot 分发。注册名与服务端构造见[原版出口](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L549-L580)，postinit 见[注册入口](../../../raw/current/scripts/modutil.lua#L593-L599)。

```lua
AddPrefabPostInit("oceanwhirlbigportalexit", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local store = inst.components.itemstore
    if store == nil then return end
    local previous_get_first = store.GetFirstItems
    store.GetFirstItems = function(self, count)
        return previous_get_first(self, math.min(count, 1))
    end
end)
```

- 选择理由：原版 onpickedfn 用固定 tuning 数量取队首，再把每个结果交给 GiveLoot；改变该实例恢复量即可保留这条消费链。[原拾取回调](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L510-L522)
- 生效与撤销：1 的单位是记录／次，整堆仍作为一条；影响该出口任何 GetFirstItems 调用，不改队列内容或迁移发送。后续仍可继续打捞，停用并重载后恢复原批量，无额外持久字段。
- 原效果：数量变化事件重算动画与 caninteractwith；分发按人物 inventory 是否对本人打开决定 GiveItem 或抛出，示例不替换它们。[状态消费](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L523-L526)、[GiveLoot](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L482-L488)
- 调整：替换 1 时保持正整数，并保留 count 上界；此包装依赖原调用者传合法 count，不提供坏记录修复或其他模组兼容保障。postinit 在原工厂后执行，见[创建入口](../../../raw/current/scripts/mainfunctions.lua#L388-L404)。

### 接收迁移记录与恢复实体的分工

原版 shardtransactionsteps 先在发送初始化时取记录并删除实体；接收最终化找到有 itemstore 的出口才追加记录，否则直接恢复到迁移位置。找不到出口且缺坐标返回 false，由事务层安排重试；itemstore 的追加方法本身不提供该保证。[发送](../../../raw/current/scripts/components/shardtransactionsteps.lua#L126-L136)、[接收分支](../../../raw/current/scripts/components/shardtransactionsteps.lua#L138-L165)、[重试调用者](../../../raw/current/scripts/components/shardtransactionsteps.lua#L47-L64)

队列中的 migrationdata 到恢复时只用于 creator.sessionid；目标位置解析不在 GetFirstItems 内。自定义储存不要把该方法当成跨分片发送 API，也不要重复追加同一事务记录。[恢复实现](../../../raw/current/scripts/components/itemstore.lua#L13-L22)

## 常见陷阱与限制

- **像 container 一样持有实体**：AddItem 后原实体已 Remove，不能再对旧引用操作；队列不会运行其原定时任务，恢复后的状态取决于各组件 OnLoad，不能承诺所有物品统一暂停／补算时间。[存入](../../../raw/current/scripts/components/itemstore.lua#L36-L43)、[持久数据载入](../../../raw/current/scripts/entityscript.lua#L1979-L2012)
- **恢复失败会安全跳过**：SpawnPrefab 结果未经 nil 检查就 SetPersistData；整批恢复结束后才移除记录。中途异常可留下已生成实体和未出队记录，无回滚；简单捕获后重试可能重复生成，须另做恢复和去重策略。[恢复与出队顺序](../../../raw/current/scripts/components/itemstore.lua#L11-L34)
- **返回 items 即已送入库存**：恢复只生成并载入数据，交付／位置由消费者处理；自定义调用者若丢弃返回表，会留下已出队的新实体。[恢复返回](../../../raw/current/scripts/components/itemstore.lua#L20-L33)、[原版交付](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L518-L520)
- **移组件／删宿主会返还内容**：组件没有自动恢复／散落钩子；需要保留内容时业务必须先取出、安置再移除，不能靠 OnSave 当作实时撤销。[组件尾部](../../../raw/current/scripts/components/itemstore.lua#L53-L74)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 3 处（mainfunctions SpawnSaveRecord／SpawnPrefabFromSim、entityscript SetPersistData 段）；无结论修订，仅锚点迁移——依据：全局枢纽 diff 审阅（R-09 改动不涉 SpawnSaveRecord 与实体持久化分发）＋变更清单命中 mainfunctions.lua、entityscript.lua，itemstore.lua、oceanwhirlbigportal.lua、shardtransactionsteps.lua 未变。
- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖完整 itemstore 实现、记录保存与恢复差异、原版出口计数／拾取／分发、迁移最终化及失败重试边界、上述单条打捞案例。未审校整个迁移事务协议或所有物品 OnLoad。
- 结构／语法：本页元数据、链接、源码锚点和大小检查通过；一个完整 modmain.lua 示例通过已有 Full Moon Lua 5.1 语法检查。
- 游戏运行：未运行；跨分片重连、旧档缺失 Prefab、恢复异常去重、皮肤权限及多人打捞仍需运行验证。

相关：[[reference/components/container]]；[[reference/components/inventoryitemholder]]；[[reference/components/stackable]]。
