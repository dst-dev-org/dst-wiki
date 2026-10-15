---
title: "transformer"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "说明生成替代 Prefab、保存旧对象数据再恢复的实现，以及离屏排队、事件重绑和存档引用的已知限制。"
aliases: ["实体替换变形", "替换 Prefab 后恢复", "离屏变形排队", "transformer"]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/transformer.lua", "entityscript.lua", "prefabs/flower.lua", "prefabs/flower_evil.lua", "prefabs/mushrooms.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# transformer

## 快速判断

- 职责与适用：生成另一 Prefab、删除原实体，把旧组件保存数据留在替代体上供以后重建原 Prefab；不是保留同一 GUID 的原地换外观。[transformer.lua:150](../../../raw/current/scripts/components/transformer.lua#L150 [transformer.lua:195](../../../raw/current/scripts/components/transformer.lua#L195
- 优先使用：用于理解已有接入／审阅历史实现；当前定向命中的花、恶魔花与蘑菇安装代码均为注释，本页不提供可直接套用的通用变身 demo。[flower.lua:178](../../../raw/current/scripts/prefabs/flower.lua#L178 [flower_evil.lua:76](../../../raw/current/scripts/prefabs/flower_evil.lua#L76 [mushrooms.lua:241](../../../raw/current/scripts/prefabs/mushrooms.lua#L241
- 重要副作用：成功替换会 Remove 旧体；只显式复制位置，组件引用与 Prefab 数据恢复存在下述缺口。设置 onTransform／onRevert 并不会得到实际通知。[transformer.lua:55](../../../raw/current/scripts/components/transformer.lua#L55 [transformer.lua:150](../../../raw/current/scripts/components/transformer.lua#L150
- 修改边界：组件无服务端断言，也不补网络、SG 或物理配套；原版注释接线位于主模拟分支。事件／世界状态／离屏字段仅构成此组件自己的替换流程，不能代表其他变身机制。[flower.lua:161](../../../raw/current/scripts/prefabs/flower.lua#L161 [transformer.lua:25](../../../raw/current/scripts/components/transformer.lua#L25

## 关键接口

| 接口／字段 | 参数或作用 | 条件与证据 |
|---|---|---|
| `transformPrefab`／`transformed` | 默认 rabbit／false | 前者是生成目标，后者标记当前体是否为替代体；手改标记不生成恢复数据，StartRevert 仍需有效 objectData。[transformer.lua:25](../../../raw/current/scripts/components/transformer.lua#L25 |
| `transformOffScreen` | 默认 true | StartTransform／StartRevert 在实体 IsAsleep 时执行，否则等 entitysleep；false 允许可见时排替换任务。低层 Transform／Revert 与 LoadPostPass 检查不受此门禁约束。[transformer.lua:182](../../../raw/current/scripts/components/transformer.lua#L182 [transformer.lua:221](../../../raw/current/scripts/components/transformer.lua#L221 |
| `StartTransform()`／`StartRevert()` | 启动或排队，无显式返回 | 先 RemoveSleepEvents；前者仅对未 transformed 的体采集旧数据，后者仅对 transformed 的体回退。无“替换任务进行中”互斥，不宜反复调用。[transformer.lua:182](../../../raw/current/scripts/components/transformer.lua#L182 [transformer.lua:221](../../../raw/current/scripts/components/transformer.lua#L221 |
| `GetObjectData()`／`SetObjectData(data)` | 采集并写 self.objectData／直接赋值 | GetObjectData 无返回值；保存原 prefab、component_data、prefab_data，不是完整引擎 SaveRecord。[transformer.lua:63](../../../raw/current/scripts/components/transformer.lua#L63 |
| `SetTransformEvent(event,target?)`／`SetRevertEvent(event,target?)` | 普通事件，默认监听 TheWorld | 0.1 秒后尝试注册；字段已存在便返回，不是替换已有监听的 setter。事件到达后调用 Start*，不按事件载荷再过滤。[transformer.lua:106](../../../raw/current/scripts/components/transformer.lua#L106 |
| `SetTransformWorldEvent(var,value)`／`SetRevertWorldEvent(var,value)` | 世界状态监视 | 同样延迟 0.1 秒且只安装一次；value 为真值时才作相等过滤，false／nil 都接受任意变化值。[transformer.lua:11](../../../raw/current/scripts/components/transformer.lua#L11 [transformer.lua:126](../../../raw/current/scripts/components/transformer.lua#L126 |
| `SetOnLoadCheck(check)` | `check(inst)` | LoadPostPass 排下一任务检查是否需变／回退，直接走低层替换；同次检查可能调用两次，不能依赖一次调用只产生一个副作用。[transformer.lua:146](../../../raw/current/scripts/components/transformer.lua#L146 [transformer.lua:285](../../../raw/current/scripts/components/transformer.lua#L285 |
| `SetOnTransformFn(fn)`／`SetOnRevertFn(fn)` | 只保存字段 | 本文件 Transform／Revert 未消费这两个字段；不能作为已实现的前后回调接口推荐。[transformer.lua:55](../../../raw/current/scripts/components/transformer.lua#L55 [transformer.lua:150](../../../raw/current/scripts/components/transformer.lua#L150 [transformer.lua:195](../../../raw/current/scripts/components/transformer.lua#L195 |

## 关键流程

### 新实体替换与旧数据恢复

GetObjectData 遍历所有有 OnSave 的组件，只保留首返回的数据表；组件返回的 refs 虽被局部接收，却未合并。再调用宿主 OnSave(inst,p_data)，保存为 prefab_data。原实体自身的存档记录、旋转、皮肤、父子关系或引擎层身份没有在这里完整采集。[transformer.lua:67](../../../raw/current/scripts/components/transformer.lua#L67

Transform 延迟 math.random() 秒后 SpawnPrefab(transformPrefab)，定位到旧体当前坐标。新体有 transformer 才复制回退事件／世界状态配置、onLoadCheck、objectData，标记 transformed=true 并转移 transformOffScreen；这些数据不会被普遍加载到替代体其他组件。即使新体无 transformer，仍删除旧体，此时没有本组件提供的自动回退通道。[transformer.lua:150](../../../raw/current/scripts/components/transformer.lua#L150

Revert 也随机延迟，按 objectData.prefab 生成新的原形，在替代体当前位置逐个调用仍存在组件的 OnLoad(v)，再调用宿主 OnLoad。它不传 ents，不执行组件／宿主的 LoadPostPass，也不重建已忽略的 GUID refs，不能承诺恢复库存、关联实体或外部追踪关系。[transformer.lua:195](../../../raw/current/scripts/components/transformer.lua#L195

还有明确字段错位：采集写的是 **prefab_data**，Revert 却给宿主 OnLoad 传 **objectData.item_data**。默认采集路径没有 item_data，原 Prefab 的保存数据因此不会按该字段传回；例如花的 OnLoad 读取 anim／planted 时无法从默认快照得到它们。[transformer.lua:84](../../../raw/current/scripts/components/transformer.lua#L84 [transformer.lua:207](../../../raw/current/scripts/components/transformer.lua#L207 [flower.lua:42](../../../raw/current/scripts/prefabs/flower.lua#L42

Transform 对 SpawnPrefab 返回 nil 没有保护，随后直接访问 Transform；Revert 保护了 obj=nil，生成失败时保留当前替代体。但 Revert 在此前仍直接索引 objectData.prefab，缺失／畸形恢复数据不安全。两种延时任务都未保存可取消句柄或剩余时间，不能作为可重入事务使用。[transformer.lua:150](../../../raw/current/scripts/components/transformer.lua#L150 [transformer.lua:195](../../../raw/current/scripts/components/transformer.lua#L195

### 世界状态与休眠排队

Set*WorldEvent 的 self:WatchWorldState 是有效的组件方法：EntityScript 的 LoadComponent 会把该方法注入组件类，并以组件自身为世界回调目标；不能只因 transformer 文件未定义它就判断调用无效。不过过滤条件写的是 not value or newvalue==value，所以 value=false 不表示“仅 false 时触发”。[entityscript.lua:42](../../../raw/current/scripts/entityscript.lua#L42 [entityscript.lua:52](../../../raw/current/scripts/entityscript.lua#L52 [transformer.lua:11](../../../raw/current/scripts/components/transformer.lua#L11

TransformOnSleep／RevertOnSleep 设 queued 标志并监听自身 entitysleep；事件包装函数先清对应 queued，再调 Start*。但 sleepTransformEvent／sleepRevertEvent 被赋为 ListenForEvent 的返回值，而该宿主方法**无返回值**。按这套默认安装，两个字段仍是 nil，RemoveSleepEvents 中的条件不会撤监听，只会清 queued 标志，不能视为可靠取消已排队事件。[transformer.lua:1](../../../raw/current/scripts/components/transformer.lua#L1 [transformer.lua:91](../../../raw/current/scripts/components/transformer.lua#L91 [transformer.lua:177](../../../raw/current/scripts/components/transformer.lua#L177 [transformer.lua:216](../../../raw/current/scripts/components/transformer.lua#L216 [entityscript.lua:1214](../../../raw/current/scripts/entityscript.lua#L1214

组件没有 OnRemoveFromEntity，移除组件不主动撤普通事件／世界 watcher，也不取消延时任务；回调里又会访问 inst.components.transformer。删除整个实体时，EntityScript:Remove 才统一停止世界监听、事件回调与 pending tasks。热移除组件与完成实体替换的清理边界不同。[transformer.lua:25](../../../raw/current/scripts/components/transformer.lua#L25 [entityscript.lua:1730](../../../raw/current/scripts/entityscript.lua#L1730

### OnSave／OnLoad／LoadPostPass 的限制

OnSave 保存排队标志、目标 Prefab、transformed、objectData、事件及世界状态字段，事件目标 GUID 另进入 refs；这里的 refs 只包括两个事件目标，**不包括 objectData 中各组件的引用**。transformOffScreen、两个变／回退回调和延迟任务剩余时间未保存。onLoadCheck 被直接放入保存表，函数能否跨实际保存器保留，本页未核验，不作为可靠存档能力承诺。[transformer.lua:233](../../../raw/current/scripts/components/transformer.lua#L233

OnLoad 恢复基础数据、transformEvent／revertEvent、onLoadCheck，并按 queued 标志重新监听睡眠；却没有恢复保存的 transformWorldEvent／revertWorldEvent 及对应 value。若宿主没有重新配置这些字段，后面的世界状态重绑分支不会仅凭存档数据自行执行。[transformer.lua:266](../../../raw/current/scripts/components/transformer.lua#L266 [transformer.lua:321](../../../raw/current/scripts/components/transformer.lua#L321

LoadPostPass 尝试从 ents 找事件目标，再调用 Set*Event；但 OnLoad 已先写入 event 字段，setter 的延迟回调见字段非 nil 就 return，所以这条恢复路径通常不会真正安装监听。事件目标引用缺失时还会跳过该目标分支，不凭 GUID 创建实体。[transformer.lua:106](../../../raw/current/scripts/components/transformer.lua#L106 [transformer.lua:297](../../../raw/current/scripts/components/transformer.lua#L297

LoadPostPass 的 onLoadCheck 另排 0 秒任务，按检查结果直接 GetObjectData+Transform 或 Revert；这会绕开 Start* 的离屏门禁／睡眠清理。本页据此描述当前实现，不把保存字段完整当作恢复协议完整。[transformer.lua:285](../../../raw/current/scripts/components/transformer.lua#L285

## 典型接入

### 原版花与蘑菇：当前仅保留注释参照

flower 的服务端 commonfn 中保留了满月变恶魔花、非满月回退及加载检查的注释代码；flower_evil 的 AddComponent 也被注释。mushrooms 同样把组件、世界状态及变体配置全部注释。因此这几处不能作为当前运行的满月变形机制依据，也不能直接取消注释就宣称数据与事件恢复可用；上面的字段、refs、false 过滤与监听问题仍需独立处理。[flower.lua:178](../../../raw/current/scripts/prefabs/flower.lua#L178 [flower_evil.lua:76](../../../raw/current/scripts/prefabs/flower_evil.lua#L76 [mushrooms.lua:241](../../../raw/current/scripts/prefabs/mushrooms.lua#L241

## 常见陷阱与限制

- **把 objectData 当 GetSaveRecord 的替代**：它只收自定义保存表，忽略组件 refs，回退也不跑引用后处理；需要关联实体的场景必须另核完整恢复协议。[transformer.lua:67](../../../raw/current/scripts/components/transformer.lua#L67 [transformer.lua:195](../../../raw/current/scripts/components/transformer.lua#L195
- **把已保存的字段视作已重绑事件／已执行回调**：普通事件与世界字段恢复各有上文缺口，onTransform／onRevert 根本未调用。这里的静态接口名不等于可靠接入能力。[transformer.lua:55](../../../raw/current/scripts/components/transformer.lua#L55 [transformer.lua:266](../../../raw/current/scripts/components/transformer.lua#L266

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、组件世界 watcher 注入、ListenForEvent 返回／实体移除清理，以及花／恶魔花／蘑菇的注释状态和花的保存字段消费。未找到可据本轮证据发布为有效接入的原版示例；未审实际存档序列化器或所有实体引用恢复机制。
- 结构检查：validate_page 及源码引用 strict 检查通过；无新增完整 Lua 示例。机械检查不代替游戏运行。
- 游戏运行：未运行；替换失败、排队取消、事件读档重绑、函数序列化及复杂组件恢复均未实测，不提供完整可用 demo。

相关：同一存档内 GUID 关联恢复可比较 [[reference/components/entitytracker]]；关联对象集合见 [[reference/components/objectspawner]]。
