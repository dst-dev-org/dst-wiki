---
title: "bait"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "诱饵物品与 trap 的双向关联、被吃／偷／拾取通知；不内置吸引生物或鱼饵机制。"
aliases: ["服务端解除熟胡萝卜诱饵关联", "陷阱诱饵", "诱饵被吃与偷走", "bait"]
tags: [component, item, event, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/bait.lua", "components/trap.lua", "prefabs/veggies.lua", "componentactions.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# bait

## 快速判断

- 职责与适用：保存所属 Trap **组件**引用，监听被吃／偷／拾取／删除并通知该陷阱；不直接规定生物食谱、诱惑范围或海钓鱼饵。[完整组件](../../../raw/current/scripts/components/bait.lua#L1-L46)
- 优先使用：建立关联走 trap:SetBait(item)，解除走同一 trap:RemoveBait()；仅赋 bait.trap 会漏目标端、位置与显示层序。捕获逻辑见 [[reference/components/trap]]。[设置与解除](../../../raw/current/scripts/components/trap.lua#L400-L424)
- 重要副作用：oneaten 把 data.eater、onstolen 把 data.thief 交 BaitTaken；无绑定陷阱的偷窃事件还会尝试把本实体 GiveItem 给 thief。不能把 onstolen 当无效果通知。[被吃／偷](../../../raw/current/scripts/components/bait.lua#L7-L17)
- 修改边界：服务端关联；动作候选靠目标 canbait 标签，执行移出执行者库存后 SetBait，不复查全部状态。吸引／进食与标签可见性需要实际食物和消费者协作。[候选](../../../raw/current/scripts/componentactions.lua#L1046-L1048) [动作](../../../raw/current/scripts/actions.lua#L1600-L1603)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `trap` | 默认 nil，类型是 Trap 组件而非陷阱 Entity；由 trap:SetBait 写入，RemoveBait 清空。不调用 bait:SetTrap，因为无此方法。[默认](../../../raw/current/scripts/components/bait.lua#L21-L28) [写入](../../../raw/current/scripts/components/trap.lua#L412-L424) |
| `IsFree()` | 仅 trap==nil，不证明物品在地面、未被其他系统占用或可食。[方法](../../../raw/current/scripts/components/bait.lua#L42-L43) |
| `oneaten` | 绑定时 BaitTaken(data.eater)；目标带 trap.targettag 或 baitstealer 时发触发链，否则只解关联。事件数据不能省 eater 后期待自动找附近对象。[监听](../../../raw/current/scripts/components/bait.lua#L7-L10) [消费者](../../../raw/current/scripts/components/trap.lua#L427-L432) |
| `onstolen` | 绑定时 BaitTaken(data.thief)；未绑定且 thief 有 inventory 时 GiveItem(inst)。组件不提供自定义 OnStolenFn setter。[监听](../../../raw/current/scripts/components/bait.lua#L13-L17) |
| `onpickup`／`onremove` | 原 OnRemove 回调调用 trap:RemoveBait，不删除另一个实体。移除 bait **组件**只解绑四个监听，不额外调用这一回调，不能当正常解除关联入口。[事件](../../../raw/current/scripts/components/bait.lua#L1-L4) [组件移除](../../../raw/current/scripts/components/bait.lua#L31-L35) |

### 关联、消费与保存的分工

trap:SetBait 先解除旧诱饵，关联新 bait、置于陷阱位置并调 onbaited；RemoveBait 只复原层序／清双向引用，不销毁物品。DoSpring 才按目标类型／baitstealer 消费或转交饵物，甚至可从周边找一件合适的自由 bait。独立 bait 没有存读方法；保存 GUID 和 postpass 重新关联归 trap。[关联](../../../raw/current/scripts/components/trap.lua#L400-L424) [捕获消费](../../../raw/current/scripts/components/trap.lua#L313) [保存](../../../raw/current/scripts/components/trap.lua#L439) [恢复](../../../raw/current/scripts/components/trap.lua#L483)

## 典型接入

### 服务端撤下熟胡萝卜诱饵，保留实体与原陷阱状态

目标模组 `modmain.lua`，不新增资源；原蔬菜 cooked 工厂已挂 bait。让实际服务端逻辑对已取得的 carrot_cooked 发私有事件，不凭事件名发起拾取或新 RPC。[原组件](../../../raw/current/scripts/prefabs/veggies.lua#L748-L765) [解除接口](../../../raw/current/scripts/components/trap.lua#L400-L410)

```lua
AddPrefabPostInit("carrot_cooked", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_detach_bait", function(item)
        local bait = item.components.bait
        local trap = bait ~= nil and bait.trap or nil
        if trap ~= nil and trap.inst:IsValid() and trap.bait == item then
            trap:RemoveBait()
        end
    end)
end)
```

调用 `carrot:PushEvent("my_mod_detach_bait")` 后物品仍在原位置；陷阱的就绪／捕获／产物不被 Reset，也没有返还给指定人物。回调检查双方指向一致，只处理合法现有关系，不声称修复其他模组的坏引用。[解除效果](../../../raw/current/scripts/components/trap.lua#L400-L410)

撤下不改变 edible 或 bait 组件，附近生物仍可能吃它，后续 DoSpring 也可能把周边合适 bait 当食物处理；若要保证安全回收，还需独立的库存／放置流程。例子只保留关联解除，不销毁物品。[周边饵物路径](../../../raw/current/scripts/components/trap.lua#L313)

不新增存档字段，后续 trap 保存反映当前无关联状态；停用并重载不再安装监听，已解除的关联不自动恢复。真实新增物品还须补双端资源／注册与食物前提，不能只复制这一 PostInit 当新诱饵工程。

## 常见陷阱与限制

- **bait.trap 是陷阱实体**：它是组件，直接在其上调 PushEvent／Transform 会用错对象；实体在 trap.inst。[写入](../../../raw/current/scripts/components/trap.lua#L412-L424)
- **RemoveComponent("bait") 会安全脱钩／RemoveBait 会销毁食物**：两者都不具备该组合语义；按目标 RemoveBait 解除再由明确业务决定去向。[组件清理](../../../raw/current/scripts/components/bait.lua#L31-L35) [目标解除](../../../raw/current/scripts/components/trap.lua#L400-L410)
- **有 bait 就自动吸引任意动物**：这里只转事件，类型和能否捕获由目标标签／trappable／eater 等消费者决定；海钓诱饵应查其实际系统，不套此协议。[触发](../../../raw/current/scripts/components/trap.lua#L184-L199) [饵物消费](../../../raw/current/scripts/components/trap.lua#L313)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 变更文件 actions.lua／componentactions.lua 对照更新报告 R-01/R-04 与 actions-ca hunk 地图逐 hunk 核验。锚点重定位 2 处（BAIT 动作移至 actions.lua:1597-1600，canbait 候选 componentactions.lua:1046-1048 行号不变）；BAIT 定义与候选无语义变化，R-01 拾取放宽（燃烧 torch，actions.lua:901）不触及本页引用。未覆盖：运行验证。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖完整 bait、trap 关联／饵物消费／保存恢复、BAIT 动作／候选及蔬菜 cooked 工厂。未全面审校动物找食物／偷窃 Brain、所有食物变体、海钓机制或坏关联恢复。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整撤下事件例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；关联后吃／偷／拾取、撤下后周边再消费、层序与 GUID 恢复、满库存转移待实测。
