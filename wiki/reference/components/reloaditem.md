---
title: "reloaditem"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "标记可经 CHANGE_TACKLE 放入当前手持容器的物品，候选、容器转移和发射消耗由外部流程完成。"
aliases: ["装填物品标记", "弹药装入手持容器", "reloaditem", "CHANGE_TACKLE"]
tags: [component, item, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/reloaditem.lua", "componentactions.lua", "actions.lua", "entityscript.lua", "prefabs/houndstooth.lua", "prefabs/houndstooth_blowpipe.lua", "containers.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# reloaditem

## 快速判断

- 职责与适用：空标记组件，构造只保存 inst，没有 Reload 方法、弹药计数或回调。实际候选插入 **ACTIONS.CHANGE_TACKLE**，不应据组件名编造 RELOAD 调用接口。[reloaditem.lua:2](../../../raw/current/scripts/components/reloaditem.lua#L2 [componentactions.lua:2855](../../../raw/current/scripts/componentactions.lua#L2855
- 优先使用：给允许快捷装入手持容器的物品沿用原版标记；是否能装入由当前 HANDS 装备的容器过滤决定，组件不声明目标武器种类。[componentactions.lua:2855](../../../raw/current/scripts/componentactions.lua#L2855
- 重要副作用：执行的是物品从旧 owner 转入／移出容器，或按槽规则交换；**装填转移不等于发射消耗**，消耗由实际武器回调处理。[actions.lua:1450](../../../raw/current/scripts/actions.lua#L1450 [houndstooth_blowpipe.lua:69](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L69
- 修改边界：原版弹药在主模拟安装组件，两端候选用 inventory／container replica 查询；reloaditem_ammo 标签只参与动作文案，不是候选的通用准入条件。[houndstooth.lua:21](../../../raw/current/scripts/prefabs/houndstooth.lua#L21 [houndstooth.lua:34](../../../raw/current/scripts/prefabs/houndstooth.lua#L34 [actions.lua:1442](../../../raw/current/scripts/actions.lua#L1442

## 关键流程

INVENTORY 的 reloaditem 候选要求使用者有 inventory replica、未搬重物，当前 HANDS 装备存在且有容器，该容器对使用者打开并 CanTakeItemInSlot(inst)。本处理器未以 right 参数作额外分支；它也不直接检查 slingshotammo／blowpipeammo 等标签，那些属于容器过滤。[componentactions.lua:2855](../../../raw/current/scripts/componentactions.lua#L2855

CHANGE_TACKLE 定义为 priority=3、rmb=true、instant=true、mount_valid=true；权威 PushBufferedAction 的 instant 分支直接执行动作，不要求为 reloaditem 新建专用装填 SG。文案先看该物品是否已在当前手持容器内，命中为 REMOVE；否则带 reloaditem_ammo 才为 AMMO。[actions.lua:423](../../../raw/current/scripts/actions.lua#L423 [entityscript.lua:1660](../../../raw/current/scripts/entityscript.lua#L1660 [actions.lua:1442](../../../raw/current/scripts/actions.lua#L1442

动作执行重新取得当前 HANDS 装备，缺 invobject／装备／container 就失败；随后主要分三路：[actions.lua:1450](../../../raw/current/scripts/actions.lua#L1450

1. 物品已由该手持装备持有时，从容器移出并尝试交给使用者 inventory。
2. 专槽容器按 GetSpecificSlotForItem 找槽，处理空槽、可堆叠补入或可替换旧物；具体 GiveItem／回退逻辑由动作与容器完成。
3. 普通容器重新 CanTakeItemInSlot，再尝试移入；只能移部分或失败时处理余物／第一槽交换，超堆叠旧物另有拒换分支。

这些分支不是 reloaditem 方法；返回 true 也不能统一解释为“整组弹药全部装满”。直接执行动作时不重查 reloaditem 标记或“容器对我打开”的候选条件，最终容量／过滤仍应核对 container 行为。[actions.lua:1456](../../../raw/current/scripts/actions.lua#L1456 [actions.lua:1466](../../../raw/current/scripts/actions.lua#L1466 [actions.lua:1500](../../../raw/current/scripts/actions.lua#L1500

## 典型接入

### 原版犬牙与犬牙吹箭：标记、装入、发射各自负责

houndstooth 在网络 pristine 前加 blowpipeammo、reloaditem_ammo 标签，服务端加 reloaditem；houndstooth_blowpipe 的容器参数从单格 hand_inv 布局派生，过滤实际只接受 blowpipeammo。仅给任意物品加 reloaditem 不会使它通过这个过滤。[houndstooth.lua:21](../../../raw/current/scripts/prefabs/houndstooth.lua#L21 [houndstooth.lua:38](../../../raw/current/scripts/prefabs/houndstooth.lua#L38 [containers.lua:1463](../../../raw/current/scripts/containers.lua#L1463 [containers.lua:1944](../../../raw/current/scripts/containers.lua#L1944

吹箭装备时 Open(owner)，卸下／装备到模型时 Close；服务端安装容器并监听 itemget／itemlose，以设置或撤销 projectile、攻击距离与 rangedweapon 等标签。发射成功进入武器 OnProjectileLaunched 回调后，才从第一槽取一份并 Remove；reloaditem 自身完全不处理射击计数或伤害。[houndstooth_blowpipe.lua:30](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L30 [houndstooth_blowpipe.lua:69](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L69 [houndstooth_blowpipe.lua:80](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L80 [houndstooth_blowpipe.lua:200](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L200 [houndstooth_blowpipe.lua:316](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L316

## 常见陷阱与限制

- **寻找 SetAmmo、Reload 或消耗回调**：组件没有这些接口；应沿 CHANGE_TACKLE、container 与目标武器追踪，不能在组件上直接调用不存在的方法。[reloaditem.lua:2](../../../raw/current/scripts/components/reloaditem.lua#L2
- **只加 reloaditem_ammo 就有装填动作**：标签只控制文案；候选需要组件动作登记与当前容器资格。反之组件构造也不自动加该标签。[actions.lua:1442](../../../raw/current/scripts/actions.lua#L1442 [componentactions.lua:2855](../../../raw/current/scripts/componentactions.lua#L2855
- **组件保存弹药或卸载时退弹**：没有 OnSave／OnLoad、更新、监听或移除钩子；物品归属与容器内容保存／撤销由相应宿主负责。[reloaditem.lua:2](../../../raw/current/scripts/components/reloaditem.lua#L2

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、INVENTORY 候选、CHANGE_TACKLE 文案与全部执行分支、instant 执行入口，以及犬牙／吹箭的容器过滤、开关、装载通知与发射消费。未展开弹弓所有变体、全部容器堆叠内部或投射物伤害。
- 结构检查：validate_page 及源码引用 strict 检查通过；无新增完整 Lua 示例。机械检查不代替游戏运行。
- 游戏运行：未运行；换手持装备、开关容器、专槽／部分堆叠／交换、骑乘与发射消耗待实测。

相关：[[reference/components/container]]；[[reference/components/inventory]]；[[reference/components/weapon]]。
