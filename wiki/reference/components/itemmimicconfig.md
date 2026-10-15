---
title: "itemmimicconfig：物品拟态强制揭示配置"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "为拟态物品登记无条件揭示的动作并记录最近动作，供 ShouldItemMimicBeRevealedFor 判定拟态是否现形。"
aliases: [物品拟态配置, itemmimicconfig, 强制揭示拟态动作]
tags: [component, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/itemmimicconfig.lua", "componentutil.lua", "components/itemmimic.lua", "components/socket_shadow_mimicry.lua", "components/builder.lua", "actions.lua", "prefabs/king_cane.lua", "prefabs/staff_common.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# itemmimicconfig（物品拟态强制揭示配置）

## 快速判断

- 职责与适用：挂在拟态物品上的纯数据组件：`forcedrevealactions` 记录"无条件揭示"的动作表，`lastaction` 记录最近一次判定动作；判定入口是全局 `ShouldItemMimicBeRevealedFor(item, user, action)`，组件自身无更新循环、事件或存档钩子。[完整定义](../../../raw/current/scripts/components/itemmimicconfig.lua#L1-L5)
- 优先使用：模组自制拟态物品要让某动作（如 GIVE）对任何使用者都现形 → 服务端 `AddComponent("itemmimic")`＋`AddComponent("itemmimicconfig")` 后 `AddActionForceMimicReveal(ACTIONS.X)`，见[典型接入](#典型接入)。
- 重要副作用：判定链每次命中本组件都无条件 `SetLastAction(action)` 覆盖记录（可为 nil）；返回 true 的动作被以 `ITEMMIMIC` 失败拦截，部分动作即时触发 `TurnEvil`，其余经 `itemmimic` 监听延迟现形。[判定链写入](../../../raw/current/scripts/componentutil.lua#L2356-L2359)
- 修改边界：判定链只在物品带 `itemmimic` 时才读本组件，单独挂载无效果；登记键为动作对象（`ACTIONS.*`），运行时状态不随存档保存。[门禁](../../../raw/current/scripts/componentutil.lua#L2354-L2356)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddActionForceMimicReveal(action)` | 写入 `forcedrevealactions[action] = true` | 键为调用方传入值；原版与判定链都用 `ACTIONS.*` 动作对象。([itemmimicconfig.lua:15-17](../../../raw/current/scripts/components/itemmimicconfig.lua#L15-L17) |
| `RemoveActionForceMimicReveal(action)` | 置 nil 撤销登记 | ([itemmimicconfig.lua:19-21](../../../raw/current/scripts/components/itemmimicconfig.lua#L19-L21) |
| `DoesActionForceMimicReveal(action)` | 返回登记值（true／nil） | 供判定链查询。([itemmimicconfig.lua:23-25](../../../raw/current/scripts/components/itemmimicconfig.lua#L23-L25) |
| `SetLastAction(action)`／`GetLastAction()` | 写／读最近动作 | 由判定链写入，供延迟响应的 nil 动作回查；运行时状态不存档。([itemmimicconfig.lua:7-13](../../../raw/current/scripts/components/itemmimicconfig.lua#L7-L13) |
| `forcedrevealactions` | 表，构造初始为空 | 构造唯一字段。([itemmimicconfig.lua:1-5](../../../raw/current/scripts/components/itemmimicconfig.lua#L1-L5) |

### 判定链：ShouldItemMimicBeRevealedFor(item, user, action)

([componentutil.lua:2331-2363](../../../raw/current/scripts/componentutil.lua#L2331-L2363) 全局函数，返回"是否应揭示拟态"：

1. `item` 为 nil，或处于只读容器内 → false。[:2331-2337](../../../raw/current/scripts/componentutil.lua#L2331-L2337)
2. 物品无 `itemmimic` → false；`user` 为 nil → 有 `itemmimic` 即 true。[:2339-2342](../../../raw/current/scripts/componentutil.lua#L2339-L2342)
3. 使用者带 `socket_shadow_mimicry` 组件（WX-78 影子插座形态挂载，逐容器推送拟态暂停刷新）→ 仅强制揭示动作揭示；普通使用者任何动作都会揭示。[:2352](../../../raw/current/scripts/componentutil.lua#L2352) [组件行为](../../../raw/current/scripts/components/socket_shadow_mimicry.lua#L1-L17)
4. 物品带 `itemmimicconfig` 时：以 `action or GetLastAction()` 查强制揭示表（第三参缺省回退最近动作），随后无条件 `SetLastAction(action)`。[:2356-2359](../../../raw/current/scripts/componentutil.lua#L2356-L2359)
5. 返回 `itemisamimic and (forcemimicprocs or userprocsmimics)`。[:2362](../../../raw/current/scripts/componentutil.lua#L2362)

消费者：`actions.lua` 33 处动作 fn 传实际动作，抽样核验如 [DoToolWork:1699](../../../raw/current/scripts/actions.lua#L1699-L1701)、[GIVE.fn:2289-2292](../../../raw/current/scripts/actions.lua#L2289-L2292)（即时 `TurnEvil`）、[CASTSPELL:3453-3455](../../../raw/current/scripts/actions.lua#L3453-L3455)；`components/itemmimic.lua` 4 处延迟响应传 nil（([itemmimic.lua:43-57](../../../raw/current/scripts/components/itemmimic.lua#L43-L57)、[112-119](../../../raw/current/scripts/components/itemmimic.lua#L112-L119)、[212](../../../raw/current/scripts/components/itemmimic.lua#L212)），依赖最近动作回查。

## 典型接入

### 模组自制拟态物品的强制揭示

前提与选择：物品 Prefab 服务端构造段（mastersim 分支内）；`itemmimic` 提供现形行为（`TurnEvil` 掉落并替换为 `itemmimic_revealed`），`itemmimicconfig` 提供动作级强制揭示，两者成对挂载。

```lua
local function master_postinit(inst)
    inst:AddComponent("inventoryitem")
    inst:AddComponent("itemmimic")         -- 判定链只对带 itemmimic 的物品读本组件
    inst:AddComponent("itemmimicconfig")
    inst.components.itemmimicconfig:AddActionForceMimicReveal(ACTIONS.GIVE)
end
```

生效与撤销：登记后该动作对任何使用者都返回"应揭示"——GIVE 在动作 fn 内即时 `TurnEvil` 并以 `ITEMMIMIC` 失败；撤销用 `RemoveActionForceMimicReveal(ACTIONS.GIVE)`。不登记时普通使用者依然揭示，仅带 `socket_shadow_mimicry` 的使用者被按住。原版参照：king_cane 构造登记 [ACTIONS.GIVE](../../../raw/current/scripts/prefabs/king_cane.lua#L361-L362)；绿宝石法杖实现登记 [ACTIONS.CASTSPELL](../../../raw/current/scripts/prefabs/staff_common.lua#L776-L777)。

## 常见陷阱与限制

- **登记键是动作对象**：判定链以 `act.action` 动作对象查表，原版也登记 `ACTIONS.*` 对象；传动作 id 字符串不会命中。[查表](../../../raw/current/scripts/componentutil.lua#L2357)
- **最近动作回退的时序**：`itemmimic` 的延迟响应（`DoTaskInTime` 数帧后 `turn_evil_redirect`）传 nil 第三参回查 `GetLastAction`；两次判定之间插入该物品的其他判定会覆盖记录，nil 判定结束后记录被清为 nil。[无条件写入](../../../raw/current/scripts/componentutil.lua#L2358) [延迟重定向](../../../raw/current/scripts/components/itemmimic.lua#L43-L47)
- **两参旧调用兼容**：第三参缺省走 `action or GetLastAction()` 回退，`itemmimic.lua` 四处即以此工作。[回退表达式](../../../raw/current/scripts/componentutil.lua#L2357)
- **制造消耗不在此判定内**：`builder` 对拟态配料与拟态折扣装备直接 `TurnEvil`，刻意不经 `ShouldItemMimicBeRevealedFor`（"ingredients must be earned"），登记动作不影响制造行为。([builder.lua:519-544](../../../raw/current/scripts/components/builder.lua#L519-L544)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/itemmimicconfig.lua` 全部 27 行、`componentutil.lua` 判定函数 2325-2363 行、`components/itemmimic.lua` 四处调用与 `TurnEvil`、`actions.lua` 33 处调用定位并抽样核验三处、`prefabs/king_cane.lua` 与 `prefabs/staff_common.lua` 原版登记段、`components/builder.lua` 绕过路径、`components/socket_shadow_mimicry.lua` 刷新行为。未覆盖：`itemmimic_revealed` 替换实体细节、客户端表现。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行；自制拟态物品强制揭示与 WX-78 插座按住行为待实测。
