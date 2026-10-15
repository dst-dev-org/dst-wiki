---
title: "upgrademoduleremover"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "WX-78 拆卸工具的空能力标记；候选、检视面板、玩家状态机和模块拔出分属其他入口。"
aliases: ["服务端结束拆卸工具持有者检视", "WX78模块拆卸工具", "upgrademoduleremover"]
tags: [component, item, state-machine, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/upgrademoduleremover.lua", "components/upgrademoduleowner.lua", "prefabs/wx78_moduleremover.lua", "prefabs/wx78.lua", "prefabs/wx78_classified.lua", "componentactions.lua", "actions.lua", "networkclientrpc.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# upgrademoduleremover

## 快速判断

- 职责与适用：组件只保存 inst，没有拆卸方法、配置、耐久或存读。原 wx78_moduleremover 挂此标记，原候选根据人物模块宿主、检视标签和 activeitem 选择动作。[完整组件](../../../raw/current/scripts/components/upgrademoduleremover.lua#L1-L7) [原工具](../../../raw/current/scripts/prefabs/wx78_moduleremover.lua#L28-L34) [候选](../../../raw/current/scripts/componentactions.lua#L3032-L3041)
- 优先使用：理解流程从原动作／宿主进入；能力撤销和返还看 [[reference/components/upgrademoduleowner]]，模块自身效果／耐久看 [[reference/components/upgrademodule]]，不编造 remover:RemoveModule 接口。
- 重要副作用：REMOVEMODULE 动作主要将工具置为 activeitem 或通知手柄交互，不在此动作直接弹出模块。实际拔出在后续选择／unplugmodule／人物 SG 中发生。[动作](../../../raw/current/scripts/actions.lua#L6719-L6736) [SG 消费](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28013-L28017) [实际弹出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28103-L28106)
- 修改边界：服务端业务与客户端 UI／SG 分开；给普通物品加此空标记不代表完成模块系统、工具动画或权限校验。当前样例仅复用原工具和人物入口。[原工厂](../../../raw/current/scripts/prefabs/wx78_moduleremover.lua#L6-L39)

## 关键接口

本组件没有公开业务方法，以下是已核验协作入口，不是组件 API 清单：

| 入口 | 条件、作用与证据 |
|---|---|
| `STARTREMOVINGMODULE` | target 或 doer 的模块宿主 StartInspecting(doer)，不等于已拔一枚；检视可涉及原备份身体，不能只根据工具名推断目标永远是自己。[动作](../../../raw/current/scripts/actions.lua#L6705-L6710) |
| `REMOVEMODULE` | 鼠标路径将工具移出原 owner 后 GiveActiveItem；手柄路径发 controller_removing_module，原玩家 SG／HUD 接后续流程。[动作](../../../raw/current/scripts/actions.lua#L6713-L6736) |
| `STOPREMOVINGMODULE`／`StopInspectingModules` | 动作更新目标 StopInspecting；原 WX 宿主 StopInspectingModules 在服务端发 stopinspectingmodule，交由当前 SG 退出，不能只清 inspecting 字段。[动作](../../../raw/current/scripts/actions.lua#L6739-L6744) [宿主](../../../raw/current/scripts/prefabs/wx78.lua#L113-L118) [SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28026-L28031) |
| `UnplugModule` 请求处理／`unplugmodule` | 请求处理检查指定模块及技能／末端顺序，向人物发事件；SG 后续 FindAndPopModule。组件标记本身没有这些检查，不把低层 PopOneModule 暴露给任意客户端参数。[请求入口](../../../raw/current/scripts/networkclientrpc.lua#L1325-L1349) [SG 弹出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28103-L28106) |
| owner `inspecter`／`inspecting` 与 classified | Start／StopInspecting 管检视者、标签和面板状态；原 classified dirty 回调显示／关闭本地 HUD。其他宿主的完整网络支持不由空标记提供。[宿主状态](../../../raw/current/scripts/components/upgrademoduleowner.lua#L83-L109) [客户端](../../../raw/current/scripts/prefabs/wx78_classified.lua#L379-L387) |

旧复数 REMOVEMODULES 调已废弃 NumModules()==0 的链，与当前候选的单数 REMOVEMODULE 不同。不要从旧代码拷出无参数 PopOneModule 或假定返回零就无耗能；当前单件成本由原 WX 回调处理。[旧路径](../../../raw/current/scripts/actions.lua#L5822-L5834) [兼容方法](../../../raw/current/scripts/components/upgrademoduleowner.lua#L165-L166) [原成本](../../../raw/current/scripts/prefabs/wx78.lua#L408-L426)

## 典型接入

### 服务端请求结束原工具持有者对自己的模块检视

目标模组 `modmain.lua`，复用原工具／角色资源；事件只找真实末端持有者及其自行发起的检视，随后调用已有 StopInspectingModules 保留 SG 退出。持有者接口按同版本 [[reference/components/inventoryitem#关键接口]] 复用。

```lua
AddPrefabPostInit("wx78_moduleremover", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_stop_self_module_inspect", function(tool)
        local inventoryitem = tool.components.inventoryitem
        local owner = inventoryitem ~= nil and inventoryitem:GetGrandOwner() or nil
        if owner == nil or not owner:IsValid() then return end
        local modules = owner.components.upgrademoduleowner
        if modules ~= nil and modules.inspecter == owner
            and owner.StopInspectingModules ~= nil then
            owner:StopInspectingModules()
        end
    end)
end)
```

服务端向实际工具发 `tool:PushEvent("my_mod_stop_self_module_inspect")`；地面／无主人、非模块宿主或其他检视者不会处理。这里的“自己的”限制不覆盖正在检视备份身体的情况；没有遍历其他实体或强关他人面板。[原停止入口](../../../raw/current/scripts/prefabs/wx78.lua#L113-L118)

该调用向当前 SG 请求退出，不直接扣电、移出模块、消费工具或清除模块效果；仅在原状态接收该事件时沿原停止动画流程完成。不要把调用方法存在或事件已发解释为任何自定义 SG 都已退出。[原接收](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28026-L28031)

无新保存字段，停用并重载后不再装额外监听；不取消此前已开始的拔出动画。扩展完整新拆卸工具仍需核验素材、候选、鼠标／手柄和网络链，本例不是完整新物品工程。

## 常见陷阱与限制

- **空组件有 Remove／PopAllModules 方法**：业务在 owner 与动作／SG，不在该标记；单独调用 RemoveComponent 也不等于取消正在进行的操作。[完整组件](../../../raw/current/scripts/components/upgrademoduleremover.lua#L1-L7)
- **选择 REMOVEMODULE 就立即扣模块耐久／电量**：该动作准备手持工具，实际拔出和宿主回调在后续链；不能另在按钮点击时重复扣费。[动作](../../../raw/current/scripts/actions.lua#L6719-L6736) [实际结算](../../../raw/current/scripts/prefabs/wx78.lua#L408-L426)
- **只 StopInspecting 或清标签就停全部人物状态**：UI、SG、工具归还分别有协作；复用原人物停止入口，不用字段改写替代退出链。[停止入口](../../../raw/current/scripts/prefabs/wx78.lua#L113-L118) [状态处理](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28026-L28031)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 13 处（componentactions／actions／SGwilson 行号迁移，区间文本逐段比对一致；REMOVEMODULE／检视链语义未变）；结论修订：无，仅锚点迁移。
- 依据：枢纽 diff 审阅（actions.lua／componentactions.lua hunk 地图）＋变更清单命中 componentactions.lua、actions.lua、stategraphs/SGwilson.lua、networkclientrpc.lua、prefabs/wx78.lua。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖空标记、原工具工厂、候选与检视／手持动作、WX 停止方法、classified 面板回调、UnplugModule 处理和 SG 实际弹出入口。未完整审校全部 UI 操作、手柄／预测／中断、影子插槽、备份身体或请求滥用防御。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整停止请求例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；鼠标／手柄、自身与备份身体、检视中切物／移动、拔出中请求停止、库存归还和远程 HUD 待实测。
