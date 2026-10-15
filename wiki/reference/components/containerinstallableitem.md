---
title: "containerinstallableitem"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "让库存物品进入指定容器时安装到目标实体，并处理退出、替换顺序与弹弓配件通知。"
aliases: ["containerinstallableitem", "容器安装物品", "弹弓配件安装卸载", "延后卸载"]
tags: [component, item, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/containerinstallableitem.lua", "components/slingshotmods.lua", "prefabs/slingshotparts.lua", "prefabs/slingshotpart_defs.lua", "prefabs/slingshotmodscontainer.lua", "prefabs/slingshot.lua", "componentactions.lua", "actions.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# containerinstallableitem

## 快速判断

- 职责与适用：组件挂在可安装的物品上；服务端监听物品库存事件，将进入有效 owner 解释为安装，目标取 `owner.install_target or owner`。它不创建格子，也不自行把物品装入容器。[事件实现](../../../raw/current/scripts/components/containerinstallableitem.lua#L1-L61)
- 优先使用：弹弓配件复用原工厂；普通容器收纳使用 [[reference/components/container]]，物品归属／转移基础见 [[reference/components/inventoryitem#生命周期]]。[配件构造](../../../raw/current/scripts/prefabs/slingshotparts.lua#L67-L96)
- 重要副作用：先调用 `(item, target)` 回调，再在 target 上发 `containerinstalleditem`／`containeruninstalleditem`，事件 data 直接是 item；原弹弓据此刷新外观和关联持有者。[通知](../../../raw/current/scripts/components/containerinstallableitem.lua#L109-L123) [消费者](../../../raw/current/scripts/prefabs/slingshot.lua#L428-L442)
- 修改边界：资格函数在双方可用；原工厂在 SetPristine 前加组件／设置资格，服务端才设安装回调与 deferred。不要只在服务端补组件却期待原安装动作在客户端出现。[工厂顺序](../../../raw/current/scripts/prefabs/slingshotparts.lua#L67-L91) [动作候选](../../../raw/current/scripts/componentactions.lua#L2622-L2627)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetValidContainerFn(fn)`／`IsValidContainer(owner)` | `fn(item, owner)` 判断容器资格；nil 默认允许，不自动要求 container。库存事件中的 owner 也可能是人物；新业务必须限制实际目标。[判断](../../../raw/current/scripts/components/containerinstallableitem.lua#L66-L72) [入栏](../../../raw/current/scripts/components/containerinstallableitem.lua#L1-L13) |
| `GetValidOpenContainer(doer)` | 从 `doer.replica.inventory:GetOpenContainers()` 用 pairs 取首个有效、非只读候选；没找到无返回值。不是最近／唯一容器选择器，也不保证 candidate 一定具有服务端 container。[实现](../../../raw/current/scripts/components/containerinstallableitem.lua#L74-L84) |
| `SetInstalledFn(fn)`／`SetUninstalledFn(fn)` | 服务端设置 `(item, target)` 回调；setter 替换旧函数，扩展原配件须主动链调。[设置](../../../raw/current/scripts/components/containerinstallableitem.lua#L89-L95) |
| `SetUseDeferredUninstall(enable)` | true 移除 exitlimbo 监听；false 恢复。没有定时器／固定帧数，旧物品卸载等待后续换 owner 或掉落流程。[实现](../../../raw/current/scripts/components/containerinstallableitem.lua#L97-L107) |
| `OnInstalled(target)`／`OnUninstalled(target)` | 直接执行业务回调及通知，不移动物品、不更改 `_owner`，也不重新校验资格；不要用它们伪造完整安装。[实现](../../../raw/current/scripts/components/containerinstallableitem.lua#L109-L123) |
| `_owner`／`ignoreuninstall` | 内部归属缓存；ignoreuninstall 为真时跳过旧目标卸载，但照样清／换缓存。原 slingshotmods 临时设置它用于拆解／烧毁时掉配件。[缓存](../../../raw/current/scripts/components/containerinstallableitem.lua#L1-L35) [原业务](../../../raw/current/scripts/components/slingshotmods.lua#L190-L204) [触发](../../../raw/current/scripts/prefabs/slingshot.lua#L444-L454) |

### 进入、替换与退出

`onputininventory(owner)` 在 owner 改变时先卸旧目标，再记新 owner，并对有效新 owner 安装；`ondropped` 卸载并清缓存。默认 `exitlimbo` 会在物品已不被旧 owner 持有时先卸载，覆盖移出但未掉地的情形；deferred 则保留缓存到后续事件。[完整事件分支](../../../raw/current/scripts/components/containerinstallableitem.lua#L1-L35)

原 `CONTAINER_INSTALL_ITEM` 同时安装和卸载：已经在打开容器中则移出归人物库存；特定槽则先取新物品、移旧槽、放新件再归还旧件；普通空槽则取物后 GiveItem。完整转移／失败语义沿 [[reference/components/container]]，不直接赋 owner。[动作实现](../../../raw/current/scripts/actions.lua#L6372-L6417)

原弹弓的隐藏 `slingshotmodscontainer` 具有 container，`slingshotmods` 创建时设 `install_target=弹弓`；所以资格检查对象是隐藏容器，回调 target 是弹弓。容器转给替换弹弓时也会对全部配件再次调用 OnInstalled，不保证每次调用都来自新入栏。[容器](../../../raw/current/scripts/prefabs/slingshotmodscontainer.lua#L26-L55) [创建与转移](../../../raw/current/scripts/components/slingshotmods.lua#L91-L126)

## 典型接入

### 猪皮弹弓皮筋在原效果上再提高弹速 10%

目标模组 `modmain.lua`，无新资源；`slingshot_band_pigskin` 来自 defs 键，经 MakePart 注册为真实 Prefab。链调原安装回调以保留射程效果，再将弹速倍率设为原猪皮配置的 1.1 倍；原卸载回调、默认资格和卸载策略原样保留。[定义与效果](../../../raw/current/scripts/prefabs/slingshotpart_defs.lua#L7-L31) [注册链](../../../raw/current/scripts/prefabs/slingshotparts.lua#L89-L103)

```lua
AddPrefabPostInit("slingshot_band_pigskin", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local installable = inst.components.containerinstallableitem
    if installable == nil then return end
    local previous_install = installable.oninstalledfn
    installable:SetInstalledFn(function(item, target)
        if previous_install ~= nil then previous_install(item, target) end
        target.projectilespeedmult = GLOBAL.TUNING.SLINGSHOT_MOD_SPEED_MULT_1 * 1.1
    end)
    -- 保留原 onuninstalledfn：正常卸载时恢复射程并清除弹速倍率。
end)
```

`1.1` 为无单位倍率：当前原配置为 1.1，本例最终为 1.21，发射时由 OnProjectileLaunched 乘到弹丸原速度上，不改变伤害／射程。每次安装都从 TUNING 基值赋值，读档或转换重装不会逐次累计。其他模组若也写同一字段，需要明确合并策略。[配置](../../../raw/current/scripts/tuning.lua#L4918-L4921) [发射消费](../../../raw/current/scripts/prefabs/slingshot.lua#L361-L369) [重装](../../../raw/current/scripts/components/slingshotmods.lua#L107-L126)

实际流程仍用原沃尔特弹弓改装资格／打开流程，再在已打开改装容器时对配件执行原安装／卸载动作或槽位操作；组件自己不打开 UI，不绕过技能检查。正常卸载调用原回调，将射程恢复并把 projectilespeedmult 设 nil；拆解／烧毁的 ignoreuninstall 特例继续沿原宿主收尾。[资格](../../../raw/current/scripts/components/slingshotmods.lua#L10-L23) [动作](../../../raw/current/scripts/actions.lua#L6372-L6417) [原恢复](../../../raw/current/scripts/prefabs/slingshotpart_defs.lua#L28-L31) [破坏流程](../../../raw/current/scripts/prefabs/slingshot.lua#L444-L454)

停用并重载后恢复原倍率；无需单独保存额外效果。组件无独立 OnSave／OnLoad 和移组件卸载钩子；原弹弓由 slingshotmods 保存隐藏容器数据，先重建 install_target，再 SetPersistData 恢复配件，正常安装回调会重算倍率。不要另外保存 `_owner` 或在移组件时期待自动卸载。[存读](../../../raw/current/scripts/components/slingshotmods.lua#L207-L221) [组件全文](../../../raw/current/scripts/components/containerinstallableitem.lua#L1-L127)

## 常见陷阱与限制

- **validcontainerfn 留空**：任何库存 owner 都可能触发安装；用于原安装动作的候选又可能缺实际 container，动作实现会直接访问它。限制为真实容器；原配件只认 `slingshotmodscontainer`。[原资格](../../../raw/current/scripts/prefabs/slingshotparts.lua#L22-L24) [动作消费](../../../raw/current/scripts/actions.lua#L6372-L6378)
- **把 deferred 当成延迟任务**：只是禁掉 exitlimbo 卸载；原骨架配件需观察新槽位是否还是同类，再决定是否替换弹弓。换成无后续库存事件的自定义路径可能长期不卸载。[开关](../../../raw/current/scripts/components/containerinstallableitem.lua#L97-L107) [骨架配件](../../../raw/current/scripts/prefabs/slingshotpart_defs.lua#L151-L170)
- **认为安装／卸载必定一一配对**：ignoreuninstall 可跳卸载，弹弓替换可直接重调安装；效果应遵循实际宿主和原回调协议，不能用通知次数累计属性。[跳过](../../../raw/current/scripts/components/slingshotmods.lua#L190-L204) [重装](../../../raw/current/scripts/components/slingshotmods.lua#L107-L126)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的漂移文件（actions.lua、componentactions.lua、tuning.lua）对照 build-26148036 更新报告（R-01/R-04、tuning 数值调整）逐条核验：CONTAINER_INSTALL_ITEM 动作实现、containerinstallableitem 候选判定逐行未变，TUNING.SLINGSHOT_MOD_SPEED_MULT_1 仍为 1.1（本版 tuning 改动集中在 Rifts8 新数值与 boss 平衡）；锚点重定位 5 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖完整组件、原配件注册／资格／猪皮回调、弹速配置及发射消费者、骨架 deferred 用途、安装动作／候选、隐藏容器与 slingshotmods 的目标设置／转移／存读／忽略卸载，以及弹弓通知消费者。通用库存事件和容器转移复用同版本链接页；未完整审校全部配件、改装 SG 与技能树。
- 结构检查：本页元数据、链接、源码行号与大小通过；完整 modmain 示例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；槽位替换、满库存去向、读档恢复、主机／远端和模组回调兼容仍待游戏验证。
