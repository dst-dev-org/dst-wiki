---
title: "unwrappable"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "把包内物品保存为记录并在拆包时重建，管理拆包门禁、回调与只读预览。"
aliases: ["包裹内容保存", "关闭包裹预览", "拆包恢复物品", "unwrappable"]
tags: [component, item, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/unwrappable.lua", "components/bundler.lua", "prefabs/bundle.lua", "actions.lua", "stategraphs/SGwilson.lua", "containers.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# unwrappable

## 快速判断

- 职责与适用：成品包裹保存物品记录、拆包重建与可选预览；不是仍存活的物品容器。交互打包过程看 [[reference/components/bundler]]，空材料看 [[reference/components/bundlemaker]]。[记录](../../../raw/current/scripts/components/unwrappable.lua#L47-L67)
- 优先使用：交互包装沿用 bundler；独立生成奖励包可用 WrapItems，但必须明确输入实体的转移／删除和成品回调，不将保存操作当搬运。[实际打包结算](../../../raw/current/scripts/components/bundler.lua#L117-L132)
- 重要副作用：WrapItems 仅删除其按字符串临时生成的实体，传入现有实体仍保留；Unwrap 重建落地物品后清记录，包裹移除由宿主回调负责。[输入处理](../../../raw/current/scripts/components/unwrappable.lua#L47-L67)／[拆包](../../../raw/current/scripts/components/unwrappable.lua#L116-L136)／[宿主删除](../../../raw/current/scripts/prefabs/bundle.lua#L297-L302)
- 修改边界：服务端内容与回调，动作标签在共同端声明；canbeunwrapped 控制标签／正常动作，但直接 Unwrap 不检查该字段。[属性](../../../raw/current/scripts/components/unwrappable.lua#L1-L23)／[动作门禁](../../../raw/current/scripts/actions.lua#L4374-L4380)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `WrapItems(items, doer)` | 非空顺序表，元素为实体或 Prefab 名 | 重建 itemdata／origin；每件先 GetSaveRecord 再发 wrappeditem，字符串输入另删除临时实体；无空／失效实体或 Spawn 失败的完整保护。[实现](../../../raw/current/scripts/components/unwrappable.lua#L47-L67) |
| `SetOnWrappedFn(fn)` | `fn(bundle, record_count, doer)` | 表项数不是堆叠总件数；OnLoad 也调用，但无 doer，不可当只发生一次的奖励通知。[设置](../../../raw/current/scripts/components/unwrappable.lua#L26-L27)／[恢复调用](../../../raw/current/scripts/components/unwrappable.lua#L191-L197) |
| `Unwrap(doer, nodelay)` | 生成落地物品，无成功 bool | 按记录 Prefab／皮肤生成，恢复 data 并通知掉落；失败的单件也不会保留记录供自动重试。[重建与清空](../../../raw/current/scripts/components/unwrappable.lua#L116-L136) |
| `SetOnUnwrappedFn(fn)` | `fn(bundle, pos, doer)` | 在 unwrapped 事件之后；即使没有 itemdata 也可能调用。原版额外掉落／特效与包裹删除在此实现。[通知](../../../raw/current/scripts/components/unwrappable.lua#L132-L136)／[宿主](../../../raw/current/scripts/prefabs/bundle.lua#L271-L302) |
| `SetUnwrapDelayFn(fn)` | `fn(bundle, doer) → 延迟秒数或nil` | 有 delay 时可能先从持有者物品栏丢出，再排任务；直接调用不会自动锁定重复请求，原版礼物回调另禁用拾取／拆包。[排任务](../../../raw/current/scripts/components/unwrappable.lua#L80-L114)／[宿主锁定](../../../raw/current/scripts/prefabs/bundle.lua#L155-L177) |
| `canbeunwrapped` | 默认 true | 更新 unwrappable 标签；燃烧时原版设 false，但烧毁仍可直接 Unwrap(nil,true)。[属性](../../../raw/current/scripts/components/unwrappable.lua#L1-L23)／[烧毁](../../../raw/current/scripts/prefabs/bundle.lua#L108-L118) |
| `SetPeekContainer(name)`／`PeekInContainer(doer)` | 预览容器名／成功 bool | 前者同步维护本端 canpeek 标签；后者生成只读副本并进入人物 SG，不取走 itemdata。不是普通可取件容器。[配置](../../../raw/current/scripts/components/unwrappable.lua#L38-L44)／[预览](../../../raw/current/scripts/components/unwrappable.lua#L141-L175) |

## 典型接入

### 关闭普通包裹的预览，保留正常拆包

目标模组 `modmain.lua`，复用原版 bundle 的内容、动画／掉落与回调，不新增资源。普通 bundle 默认声明 canpeek，服务端配置 bundle_container；本例共同端移除标签，再取消服务端预览配置。[共同标签](../../../raw/current/scripts/prefabs/bundle.lua#L338-L343)／[配置](../../../raw/current/scripts/prefabs/bundle.lua#L372-L375)／[普通包设置](../../../raw/current/scripts/prefabs/bundle.lua#L400-L405)／[PostInit](../../../raw/current/scripts/modutil.lua#L593)

```lua
AddPrefabPostInit("bundle", function(inst)
    inst:RemoveTag("canpeek")
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.unwrappable ~= nil then
        inst.components.unwrappable:SetPeekContainer(nil)
    end
end)
```

只禁原版 PEEKBUNDLE 路径，不是内容保密或权限系统；不关闭 UNWRAP、不修改保存数据／燃烧处理，也不影响 gift 等其他 Prefab。停用后新建／读档普通包裹恢复原版预览配置；测试持有／地面、读档和其他模组重新开启预览。[预览动作门禁](../../../raw/current/scripts/actions.lua#L4384-L4391)

## 生命周期

包装保存当前物品记录并记世界 session_identifier；OnSave 保存 items／origin，OnLoad 恢复记录后通知 onwrappedfn。拆包按记录生成实体、SetPersistData，再发每件 unwrappeditem；清空记录后给包裹发 unwrapped，最后执行 onunwrappedfn。事件是通知，不应被用来补写此前已生成的保存记录。[包装](../../../raw/current/scripts/components/unwrappable.lua#L47-L67)／[保存恢复](../../../raw/current/scripts/components/unwrappable.lua#L182-L197)／[拆包顺序](../../../raw/current/scripts/components/unwrappable.lua#L116-L136)

保存记录不等于任意对象关系完整迁移：GetSaveRecord 可另返回 references，但此组件只取第一返回值；拆包只恢复 v.data，未提供 newents 映射。不推荐用它打包任意关联实体，先核验目标组件的保存恢复需求。[记录返回](../../../raw/current/scripts/entityscript.lua#L312-L315)／[取记录](../../../raw/current/scripts/components/unwrappable.lua#L56-L57)／[恢复](../../../raw/current/scripts/components/unwrappable.lua#L125)／[组件加载](../../../raw/current/scripts/entityscript.lua#L2005-L2012)

## 常见陷阱与限制

- **对现有实体调用 WrapItems 后不处理原件：** 原件继续存在，拆包又生成记录副本，可能复制物品。交互路径由 bundler 接着移除临时容器；自定义流程明确资源所有权，不能看到 WrapItems 就盲目另删所有输入。[仅删字符串临时件](../../../raw/current/scripts/components/unwrappable.lua#L52-L63)／[正常清理](../../../raw/current/scripts/components/bundler.lua#L124-L132)
- **第二次 WrapItems 是追加：** 非空输入会重置 itemdata，覆盖旧内容；空表则什么也不清。只对明确未装内容的目标初始化，增删已有包裹另设计事务。[重置](../../../raw/current/scripts/components/unwrappable.lua#L47-L57)
- **预览窗口就是打开原件：** 它重建不持久化副本并设只读，原记录仍在；原版 SG 退出负责删除预览容器，不可把副本发给人物。[只读副本](../../../raw/current/scripts/components/unwrappable.lua#L150-L170)／[退出清理](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19881-L19888)／[按钮拒绝只读](../../../raw/current/scripts/containers.lua#L486-L487)
- **Unwrap 一定把自己删掉：** 无此内置保证，且空内容仍可能触发宿主额外掉落。保留原版成品回调，勿把 Unwrap 接成可反复触发的无门禁 RPC。[尾部](../../../raw/current/scripts/components/unwrappable.lua#L132-L136)／[宿主删除](../../../raw/current/scripts/prefabs/bundle.lua#L302)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页漂移文件 actions.lua（R-01）、stategraphs/SGwilson.lua（SB-17 相关）、containers.lua（R-07）、entityscript.lua（R-08）对照 build-26148036 更新报告与 actions／componentactions hunk 地图逐条核验，全部变更未触及本页 UNWRAP／PEEKBUNDLE 动作门禁、预览退出与按钮只读拒绝等结论，锚点重定位 4 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖输入所有权、保存／恢复、事件顺序、延迟／门禁／宿主清理、普通包只读预览与关闭示例；未核验全部特殊奖励包、皮肤跨世界权限或任意实体关系恢复。
- 结构／示例检查：本批格式、来源文件、链接／行锚点、导航登记及完整 Lua 5.1 示例语法检查通过。
- 游戏运行：未运行；预览副本清理、重复请求／延迟、堆叠与新鲜度恢复、燃烧、跨世界与其他模组兼容需验证。
