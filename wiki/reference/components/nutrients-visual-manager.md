---
title: "nutrients_visual_manager"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "管理本地农田养分标记的视野样式与登记，区分土壤数据、覆盖层同步和眼镜开关。"
aliases: ["调整养分标记绘制排序", "农田养分显示", "养分眼镜视野", "nutrients_visual_manager"]
tags: [component, lifecycle, callback, ui, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/nutrients_visual_manager.lua", "prefabs/world.lua", "prefabs/nutrients_overlay.lua", "components/farming_manager.lua", "components/playervision.lua", "prefabs/hats.lua", "widgets/nutrientsover.lua", "modutil.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# nutrients_visual_manager

## 快速判断

- 职责与适用：非专服世界上的本地养分标记样式管理器；客户端和有画面的主机均可有，不能用 ismastersim 排除客户端。专服构造会断言失败。[宿主分支](../../../raw/current/scripts/prefabs/world.lua#L557-L569) [断言](../../../raw/current/scripts/components/nutrients_visual_manager.lua#L8-L10)
- 优先使用：改标记样式包装 `UpdateVisualAnimState(visual)`；改实际养分走 [[reference/components/farming-manager]]。显示模式由世界 nutrientsvision 事件更新，登记不负责改养分／创建网络覆盖层。[样式与登记](../../../raw/current/scripts/components/nutrients_visual_manager.lua#L43-L62) [实际写入](../../../raw/current/scripts/components/farming_manager.lua#L384-L390)
- 重要副作用：模式改变会刷新全部已登记 visual；关闭时仍设黑色乘色／暗红加色及地面层，并非 Hide 或删除。绕过原方法改颜色可能破坏眼镜开关。[模式事件](../../../raw/current/scripts/components/nutrients_visual_manager.lua#L30-L35) [两种样式](../../../raw/current/scripts/components/nutrients_visual_manager.lua#L43-L53)
- 修改边界：管理对象是本地 nutrients_overlay_visual，不是保存三养分精确值的对象；营养等级和水分的网络／动画处理在父覆盖层，直接调用样式方法不会施肥或广播给其他玩家。[本地对象](../../../raw/current/scripts/prefabs/nutrients_overlay.lua#L57-L82) [网络对象](../../../raw/current/scripts/prefabs/nutrients_overlay.lua#L87-L110)

## 关键接口

| 接口／输入 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `UpdateVisualAnimState(visual)` | 有 AnimState 的显示实体；无显式返回 | 按私有 nutrients_vision 设置乘色／加色／图层及排序号 1，不改养分数量、符号等级或登记集合。[实现](../../../raw/current/scripts/components/nutrients_visual_manager.lua#L43-L53) |
| `RegisterNutrientsVisual(visual)` | 实体作为集合键；无显式返回 | 仅置 true，重复登记同一键不叠加；不初始化样式、监听移除或创建 visual，宿主另行负责。[实现](../../../raw/current/scripts/components/nutrients_visual_manager.lua#L57-L58) [原宿主顺序](../../../raw/current/scripts/prefabs/nutrients_overlay.lua#L69-L78) |
| `UnregisterNutrientsVisual(visual)` | 移除集合键；无显式返回 | 不 Remove 对象；原 visual 在 onremove 时调用。自建对象须对称清登记，防止后续切模式访问已失效实体。[实现](../../../raw/current/scripts/components/nutrients_visual_manager.lua#L61-L62) [宿主清理](../../../raw/current/scripts/prefabs/nutrients_overlay.lua#L75-L78) |
| 世界 `nutrientsvision` 事件 | `{enabled = boolean}` | 监听对象为世界实例；enabled 改变时才遍历刷新，不检查 payload 是否存在。私有模式初值 false，无公开 getter／setter；不写不存在的 self.nutrients_vision。[私有状态](../../../raw/current/scripts/components/nutrients_visual_manager.lua#L22-L35) [监听](../../../raw/current/scripts/components/nutrients_visual_manager.lua#L78) |

### 数据、视野与生命周期分工

1. **土壤数据与网络父对象：** farming_manager 在农田地块建立 nutrients_overlay，养分写入时调用父对象 UpdateOverlay。它以 0／1／25／50／100 阈值压成各占 3 位的等级并写 net_ushortint；不是把三个精确数值直接交给本管理器。[地块建立／移除](../../../raw/current/scripts/components/farming_manager.lua#L153-L178) [等级编码](../../../raw/current/scripts/prefabs/nutrients_overlay.lua#L39-L50) [网络字段](../../../raw/current/scripts/prefabs/nutrients_overlay.lua#L106-L111)
2. **本地子对象与样式：** 非专服才 SpawnChild nutrients_overlay_visual；visual 创建时先应用当前样式再登记，父对象的 dirty 回调按等级 ShowSymbol／HideSymbol。水分动画在父对象，不由本管理器开关。[子对象初始化](../../../raw/current/scripts/prefabs/nutrients_overlay.lua#L69-L78) [等级消费](../../../raw/current/scripts/prefabs/nutrients_overlay.lua#L17-L34) [水分](../../../raw/current/scripts/prefabs/nutrients_overlay.lua#L53-L54)
3. **本地玩家视野：** 原 nutrientsgoggles 的共同初始化加入 nutrientsvision 标签；playervision 在装备变化时检查 replica.inventory，只有该人物是 ThePlayer 且没有强制模式覆盖时才向 TheWorld 发事件。需要能力接入时沿这条路径核验，勿仅在服务端 PushEvent 就称全客户端生效。[眼镜标签](../../../raw/current/scripts/prefabs/hats.lua#L3126-L3142) [装备检查](../../../raw/current/scripts/components/playervision.lua#L56-L59) [初始化监听](../../../raw/current/scripts/components/playervision.lua#L82-L95)
4. **视野与 HUD 分开：** `playervision:ForceNutrientVision(force)` 名称为单数 Nutrient，修改的是单个强制布尔值，不是多来源堆栈；HasNutrientsVision 合并装备与强制状态。屏幕遮罩 nutrientsover 另听同一世界事件，并在创建时查人物当前状态；只改土壤 visual 不会改 HUD。[强制入口](../../../raw/current/scripts/components/playervision.lua#L297-L301) [查询](../../../raw/current/scripts/components/playervision.lua#L151-L152) [遮罩](../../../raw/current/scripts/widgets/nutrientsover.lua#L18-L34)

本地 visual 不联网、不保存；有画面的主模拟在父对象 entitysleep 时移除子对象，entitywake 时重建并刷新等级。重建会重新走样式／登记，移除会注销。组件本身未定义存读或专用卸载恢复，动态删除整个世界管理器会破坏宿主后续调用，不作为普通显示开关。[对象属性](../../../raw/current/scripts/prefabs/nutrients_overlay.lua#L60-L82) [主机睡醒](../../../raw/current/scripts/prefabs/nutrients_overlay.lua#L108-L124)

## 典型接入

### 将养分标记排序号设为 2，保留眼镜视野切换

放目标模组 `modmain.lua`，无需新增资源／Prefab；只包装原世界组件的样式入口，不手工创建管理器。ComponentPostInit 在组件构造后调用，此时方法已建立；原视觉对象创建和模式变化都会走该入口。不要加“非主模拟 return”，否则远端客户端不会安装包装。[Hook 定义](../../../raw/current/scripts/modutil.lua#L566-L572) [调用顺序](../../../raw/current/scripts/entityscript.lua#L655-L662) [创建入口](../../../raw/current/scripts/prefabs/nutrients_overlay.lua#L69-L75)

```lua
AddComponentPostInit("nutrients_visual_manager", function(self)
    local original = self.UpdateVisualAnimState
    self.UpdateVisualAnimState = function(manager, visual)
        original(manager, visual)
        visual.AnimState:SetSortOrder(2)
    end
end)
```

`2` 是替换原排序号 1 的显示参数；保留原模式的颜色和图层，不承诺跨图层遮挡优先级，也不改变水分父对象或屏幕遮罩。模式切换及主机子对象重建后仍应用包装。代码须在要改变显示的客户端／有画面主机加载，仅专服执行不会改变远端显示；具体分发沿用目标模组配置。停用并重新加载后恢复原方法，不改土壤数值或存档。本例不提供自定义贴图／视觉对象骨架或强制视野功能。

## 常见陷阱与限制

- **只在 ismastersim 分支包装，或给专服挂组件**：前者漏掉远端显示，后者触发断言。沿原非专服宿主生命周期安装，不复制服务端 farming_manager 的端检查。
- **登记就自动按当前模式显示，注销就销毁**：这两个方法只改集合；原宿主先 Update 再 Register、在 onremove 注销。自建对象要复用这一协作，不只抄登记调用。
- **切 nutrientsvision 就改变实际养分或人物能力**：它只通知本地显示消费者，不改变养分网格或 playervision 的装备／强制字段；直接发事件还可能与下一次装备切换冲突。显示、能力、数据分别用各自入口。
- **读取覆盖层等级就得到三养分精确值**：网络数据经过阈值量化；精确服务端数值读 farming_manager。改配色或符号并不施肥，也不证明作物压力解除。

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 4 处（world 宿主分支、hats 眼镜初始化、playervision 查询与强制入口两段）；无结论修订，仅锚点迁移——依据：变更清单命中 prefabs/world.lua（新增段在 MakeWorld 其他区域）、prefabs/hats.lua（大量改动均在其他帽型，nutrients_custom_init 内容未变仅行移）、components/playervision.lua（本版改动在 OnAreaChanged，养分入口仅行移）；nutrients_visual_manager.lua、farming_manager.lua、nutrients_overlay.lua 未变。
- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖非专服世界宿主、样式／登记与世界事件、覆盖层量化同步及子对象睡醒清理、眼镜／本地玩家事件和独立遮罩，以及排序包装例。playervision 仅核验养分相关入口，未审校全部视觉能力、网络引擎或自定义资源。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整 modmain 例通过已有 Full Moon Lua 5.1 syntax-only，不证明实际渲染顺序或联网显示效果。
- 游戏运行：未运行；专服连接／有画面主机、眼镜穿脱／强制视野、主机睡醒、土壤量化变化、分片和其他模组包装叠加待实测。
