---
title: linkeditem
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "按 userid 持久关联物品与玩家，随玩家重连恢复实体，并可限制其他玩家装备。"
aliases: [linkeditem, 物品绑定玩家, 记录物品制造者, 绑定物品主人, 专属装备, 重连恢复主人]
tags: [component, item, network-sync, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/linkeditem.lua, components/linkeditemmanager.lua, components/equippable.lua, components/equippable_replica.lua, components/inventoryitem.lua, components/slingshotmods.lua, prefabs/slingshot.lua, prefabs/wortox_reviver.lua, modutil.lua]
reliability: verified
validation: source-reviewed
status: active
---

# linkeditem

## 快速判断

- 职责与适用：按玩家 `userid` 记录物品归属，配合 [[reference/components/linkeditemmanager]] 找回当前玩家实体；适合物品离身后仍保留的身份关联，不是 [[reference/components/inventoryitem]] 的当前持有者。[管理器用途](../../../raw/current/scripts/components/linkeditemmanager.lua#L1-L10)
- 优先使用：服务端 `LinkToOwnerUserID(userid)` 绑定、传 `nil` 解绑；双端通过 `GetOwnerUserID()`／`GetOwnerName()` 读取。仅跟踪存档内普通实体时比较 [[reference/components/entitytracker]]。[绑定](../../../raw/current/scripts/components/linkeditem.lua#L61-L76)、[读取](../../../raw/current/scripts/components/linkeditem.lua#L26-L52)
- 重要副作用：解绑旧身份、登记新身份均向世界发事件；管理器同步触发主人移除／创建回调。限制装备需另外启用 flag，由 equippable 消费。[事件](../../../raw/current/scripts/components/linkeditem.lua#L67-L75)、[装备判断](../../../raw/current/scripts/components/equippable.lua#L147-L162)
- 修改边界：组件构造含 netvar，原版在双端 `SetPristine()` 前添加；不能用服务端 PostInit 给普通物品临时补装，也不支持动态移除组件。[构造与移除注释](../../../raw/current/scripts/components/linkeditem.lua#L2-L20)、[弹弓工厂](../../../raw/current/scripts/prefabs/slingshot.lua#L482-L529)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `GetOwnerUserID()` | 双端；空网络字符串转为 nil。与当前持有者／玩家实体是否在线无直接等价关系。[实现](../../../raw/current/scripts/components/linkeditem.lua#L45-L48) |
| `GetOwnerName()` | 双端；有 userid 时，每隔超过 10 秒尝试查客户端表，用 `set_local` 更新本端名字；查不到回退网络名字，空值返回 nil。不保证离线玩家仍有名字。[实现](../../../raw/current/scripts/components/linkeditem.lua#L26-L43) |
| `GetOwnerInst()` | 服务端；返回管理器维护的当前玩家实体或 nil。[读取](../../../raw/current/scripts/components/linkeditem.lua#L57-L59)、[玩家离开](../../../raw/current/scripts/components/linkeditemmanager.lua#L80-L97) |
| `LinkToOwnerUserID(userid?)` | 服务端；相同身份直接返回；旧身份先注销，再写 userid 并登记新身份。绑定传有效非空 userid，解绑传 nil，不用空字符串代替。[实现](../../../raw/current/scripts/components/linkeditem.lua#L61-L76) |
| `SetEquippableRestrictedToOwner(bool)`／`IsEquippableRestrictedToOwner()` | 前者服务端设置 net_bool，后者双端读取；setter 本身不执行卸装。[设置](../../../raw/current/scripts/components/linkeditem.lua#L78-L80)、[读取](../../../raw/current/scripts/components/linkeditem.lua#L50-L52) |
| `SetOnOwnerInstRemovedFn(fn)`／`SetOnOwnerInstCreatedFn(fn)` | 服务端回调 `(item, owner)`；实体切换先调用旧主人 removed，再更新内部引用，最后调用新主人 created。设置函数只替换回调，不补发当前状态。[注册](../../../raw/current/scripts/components/linkeditem.lua#L82-L92)、[触发顺序](../../../raw/current/scripts/components/linkeditem.lua#L94-L115) |
| `SetOnSkillTreeInitializedFn(fn)` | 服务端回调 `(item, owner)`；管理器在玩家技能握手完成或登记到已就绪玩家时触发，适合读取技能再配置物品；不代表每次技能选择变化都会重发。[回调](../../../raw/current/scripts/components/linkeditem.lua#L117-L121)、[管理器时机](../../../raw/current/scripts/components/linkeditemmanager.lua#L55-L78) |
| `OnSave()`／`OnLoad(data)` | 保存 userid／名字，读档重新登记非空 userid；不保存 owner 实体、限制 flag 或回调。限制和回调由 Prefab 工厂重配。[存取](../../../raw/current/scripts/components/linkeditem.lua#L123-L144) |

`SetOwnerInst`／`OnSkillTreeInitialized` 明确供管理器调用；模组应设置回调并通过正常绑定登记，不直接模拟玩家上线。[内部入口注释](../../../raw/current/scripts/components/linkeditem.lua#L94-L121)

## 典型接入

### 允许其他合资格玩家装备基础弹弓，保留原主人关联

完整代码放目标模组 `modmain.lua`，无需新资源或 GLOBAL 代理；只修改已经正确创建网络组件的原版基础 `slingshot`。工厂默认启用主人限制，另设 `slingshot_sharpshooter` 标签限制；下面只解除前者，不会让所有角色都能使用。[工厂设置](../../../raw/current/scripts/prefabs/slingshot.lua#L534-L544)、[Hook](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("slingshot", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local linkeditem = inst.components.linkeditem
    if linkeditem ~= nil then
        linkeditem:SetEquippableRestrictedToOwner(false)
    end
end)
```

每次该 Prefab 生成／读档重建时覆盖 flag；移除这段并重新加载后恢复工厂默认。未改其他弹弓变体、归属 userid 或原回调；如要推广到变体，先核对对应工厂注册，不用宽泛所有物品 Hook。[注册名](../../../raw/current/scripts/prefabs/slingshot.lua#L911-L915)

此例不立即卸下现有装备。客户端 equippable replica 也读取本组件的网络 flag；检查时区分“其他合资格玩家可装备”和“无角色标签仍被拒绝”。物品显示名仍可带原主人，弹弓 inspectable 的 `NOT_MINE` 判断也不会因解除装备 flag 自动消失。[双端限制](../../../raw/current/scripts/components/equippable_replica.lua#L32-L48)、[检查文案](../../../raw/current/scripts/prefabs/slingshot.lua#L467-L478)

### 新物品需要绑定／主人技能协作时，照工厂分端组装

以 `wortox_reviver` 为参照：`AddNetwork` 后、`SetPristine` 前双端添加 linkeditem；服务端才装 inventoryitem、回调与技能逻辑。其建造／入袋／读档初始化尝试为尚未绑定的物品关联合资格玩家；离线失去实体时禁用消耗，技能初始化后再按主人技能开放。不要整段复制成普通物品的服务端 AddComponent。[工厂与回调](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L200-L266)、[首次绑定入口](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L33-L62)、[技能协作](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L64-L78)

弹弓采用另一种策略：安装部件变化后，按当前 `GetGrandOwner()` 更新绑定；没有部件时解绑。**自动绑定谁、何时换绑由宿主决定**，linkeditem 本身不监听入袋事件。[UpdateLinkedItemOwner](../../../raw/current/scripts/prefabs/slingshot.lua#L415-L426)、[宿主事件](../../../raw/current/scripts/prefabs/slingshot.lua#L561-L563)

换绑条件与输入的边缘情况（26148036 核验）：换绑由部件安装／卸载／替换事件（`containerinstalleditem`、`containeruninstalleditem`、`installreplacedslingshot`）驱动，加载期（`IsLoading()`）跳过；`HasAnyParts()` 只要求**至少安装一个部件**，不要求装齐，一件不剩时才走无条件解绑。[监听](../../../raw/current/scripts/prefabs/slingshot.lua#L561-L563)、[加载门控](../../../raw/current/scripts/components/slingshotmods.lua#L28-L33)、[部件判断](../../../raw/current/scripts/components/slingshotmods.lua#L45-L49)。同轮 `GetGrandOwner()` 由严格递归改为"递归无主时回退返回中间持有者"（[实现](../../../raw/current/scripts/components/inventoryitem.lua#L501-L506)）：弹弓被玩家持有／装备时新旧等价（玩家实体无 inventoryitem 组件，两版都返回玩家）；至少装有一个部件且弹弓位于无人持有的容器（如落地背包）内触发换绑时，旧版拿到 nil 不动作、维持原绑定，新版拿到容器实体并以 `userid=nil` 解绑。依赖旧行为"容器内维持原绑定"的模组需要调整，或自行补挂宿主事件控制换绑时机。

## 常见陷阱与限制

- **把专属装备当私有物品权限：** equippable 的判断只作用于玩家／`possessedbody` 装备目标，且未绑定 userid 时不会因该 flag 拒绝；不提供禁拾取、禁放容器或禁交易门禁。实际交互限制另查对应消费者。[服务端判断](../../../raw/current/scripts/components/equippable.lua#L147-L162)
- **离线误判为解绑：** 玩家离开仅令 owner 实体为 nil，userid 仍保留；不要据此自动让他人认领。名字可能被 `SetOwnerInst(nil)` 清空，也可能由后续本端查询回填，不能用名字判权限。[离开](../../../raw/current/scripts/components/linkeditemmanager.lua#L80-L97)、[名称更新](../../../raw/current/scripts/components/linkeditem.lua#L94-L115)
- **覆盖原回调：** 三个 SetOn…Fn 都只有一个槽；原宿主已有技能／离线行为时不能直接替换而丢失。需要扩展时保存原函数并保持其参数、调用顺序。[回调注册](../../../raw/current/scripts/components/linkeditem.lua#L82-L92)、[复活心宿主](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L263-L264)
- **保存后忘记重配权限：** OnSave 不含限制 flag；自定义工厂每次初始化都要设置。物品移除通过 onremove 注销关联，不用 RemoveComponent 实现解绑。[移除](../../../raw/current/scripts/components/linkeditem.lua#L10-L20)、[存档](../../../raw/current/scripts/components/linkeditem.lua#L123-L144)

## 验证记录

- 版本增量复核：2026-10-25，`steam-buildid:26148036`；本页原 sources 全部哈希未变（含 slingshot.lua、wortox_reviver.lua），枢纽反查命中 C-19：`GetGrandOwner` 递归无主回退中间持有者（inventoryitem.lua:501-506），实读 slingshot.lua:415-426／561-565 核验换绑触发事件与落地容器解绑边缘条件，正文补入条件化说明并把 inventoryitem.lua 纳入 sources；其余结论按原始哈希等价沿用。
- 源码审阅：2026-10-23，`steam-buildid:25643519`；完整审阅组件及管理器，核验弹弓工厂注册／默认限制／绑定更新、复活心工厂／绑定／技能回调、双端装备消费者及 Hook。依据见各节；不承诺跨分片自动寻找主人。
- 游戏运行：未运行；示例的穿戴、读档与检查文案需在实际模组测试。
- 结构检查：页面格式／链接、源码锚点严格检查及完整 modmain 示例的 Lua 5.1 语法检查通过，不代表游戏效果通过。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/linkeditemmanager]]、[[reference/components/equippable]]、[[reference/components/inventoryitem]]、[[reference/components/entitytracker]]。
