---
title: "attuner"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "查询玩家当前调谐来源，通过专属代理同步远距离类别，并处理关系变化和角色实体转移。"
aliases: ["attuner", "玩家调谐查询", "是否绑定复活雕像", "调谐状态监听", "远距离绑定状态", "调谐迁移"]
tags: [component, event, network-sync, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/attuner.lua", "components/attunable.lua", "prefabs/attunable_classified.lua", "prefabs/player_common.lua", "actions.lua", "modutil.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# attuner

## 快速判断

- 职责与适用：玩家侧保存“来源 GUID → attunable_classified”映射；服务端可查各玩家，客户端只保证本地玩家的数据。目标侧创建／撤销关系见 [[reference/components/attunable]]。[实现与端约定](../../../raw/current/scripts/components/attuner.lua#L1-L24)
- 优先使用：已知目标查 `IsAttunedTo(target)`，只问类别查 `HasAttunement(tag)`；确需实际来源实体时在服务端用 `GetAttunedTarget(tag)`。[查询](../../../raw/current/scripts/components/attuner.lua#L13-L46)
- 重要副作用：代理注册／注销在玩家上推 `gotnewattunement`／`attunementlost`，`data.proxy` 是代理，不是建筑。收到失去事件须重查类别，不能直接断言“再无同类调谐”。[事件](../../../raw/current/scripts/components/attuner.lua#L64-L79) [原版重查](../../../raw/current/scripts/prefabs/player_common.lua#L454-L468)
- 修改边界：本组件原版在玩家公共工厂的 pristine／服务端返回之前添加，**不要套通用“组件只在服务端存在”假设**；客户端远处来源实体可能不可见，仍可查代理类别。[添加](../../../raw/current/scripts/prefabs/player_common.lua#L2508-L2509) [端分界](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2615) [代理约定](../../../raw/current/scripts/components/attuner.lua#L58-L65)

## 关键接口

| 接口／字段 | 签名、返回与条件 | 作用与证据 |
|---|---|---|
| `IsAttunedTo(target)` | 布尔；target 必须为可访问实体 | 服务端按 GUID 查表；客户端比较目标的 `ATTUNABLE_ID_<GUID>` tag。[实现](../../../raw/current/scripts/components/attuner.lua#L13-L24) |
| `HasAttunement(tag)` | 布尔；两端，但客户端只保证本地玩家 | 遍历代理检查精确 tag，不自动扩展等价 tag；查两类应显式 OR。[实现](../../../raw/current/scripts/components/attuner.lua#L26-L33) [原版两类查询](../../../raw/current/scripts/prefabs/player_common.lua#L863-L866) |
| `GetAttunedTarget(tag)` | 来源实体或 nil；服务端 | 找到该 tag 的代理后返回 `Ents[GUID]`；客户端无返回。不是给客户端跨距离取实体的 API。[实现](../../../raw/current/scripts/components/attuner.lua#L38-L46) |
| `attuned` | GUID → proxy 表 | 由 classified 注册／注销维护，勿直接写表模拟绑定；会跳过目标的成本、事件和存档。[索引维护](../../../raw/current/scripts/components/attuner.lua#L67-L79) [绑定过程](../../../raw/current/scripts/components/attunable.lua#L93-L118) |
| `TransferComponent(newinst)` | 服务端，已有新玩家实体与 attuner | 逐源对旧玩家 Unlink(true)，对新玩家 Link(true)；两边回调仍执行，成本跳过。[转移](../../../raw/current/scripts/components/attuner.lua#L49-L56) |
| `RegisterAttunedSource(proxy)`／`UnregisterAttunedSource(proxy)` | 内部协议，按 proxy.source_guid 维护去重 | 由 classified 调用；业务监听玩家事件，不手动生成事件或代理。[注册／注销](../../../raw/current/scripts/prefabs/attunable_classified.lua#L1-L22) |

### 代理与恢复的分工

`attunable_classified` 的 `source_guid` 为 net_uint，服务端设专属网络接收者并父接玩家；客户端用零延迟任务等初值反序列化后注册，代理移除则向玩家注销。代理 `persists=false`，绑定 userid 的存读由目标 attunable 负责，本组件没有自己的 OnSave／OnLoad。[代理构造](../../../raw/current/scripts/prefabs/attunable_classified.lua#L28-L56) [连接](../../../raw/current/scripts/prefabs/attunable_classified.lua#L17-L22) [目标存读](../../../raw/current/scripts/components/attunable.lua#L137-L164)

角色实体替换时，`SwapAllCharacteristics` 遍历具有 TransferComponent 的旧组件触发转移；不等于任意重生／跨分片都走这个入口。加载免成本和旧新回调区别见 [[reference/components/attunable#关键接口]]。[分发](../../../raw/current/scripts/prefabs/player_common.lua#L2328-L2334)

## 典型接入

### 为自定义逻辑维护“玩家已有远程复活调谐”标志

放目标模组 `modmain.lua`，无额外资源。这是完整的**服务端状态标志**示例：给玩家增删 `mod_has_effigy` tag，供本模组其他服务端规则查询；不添加复活能力、不重写原版调谐监听。原版也用两类 HasAttunement 判断恢复后的状态，并在失去事件时重查。[原版消费者](../../../raw/current/scripts/prefabs/player_common.lua#L442-L468) [恢复后查询](../../../raw/current/scripts/prefabs/player_common.lua#L852-L866)

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local function RefreshAttunement(player)
        local attuner = player.components.attuner
        local linked = attuner ~= nil
            and (attuner:HasAttunement("remoteresurrector")
                or attuner:HasAttunement("gravestoneresurrector"))
        if linked then
            player:AddTag("mod_has_effigy")
        else
            player:RemoveTag("mod_has_effigy")
        end
    end
    inst:ListenForEvent("gotnewattunement", RefreshAttunement)
    inst:ListenForEvent("attunementlost", RefreshAttunement)
    inst:DoTaskInTime(0, RefreshAttunement)
end)
```

初始延迟查询覆盖既有状态，后续代理增删都重算，避免仅收一次事件就把布尔状态写死。没有人为替换 attunable／attuner 的回调；玩家移除时清监听并取消待执行任务。卸载模组并重启停止添加该标志；存量实例热卸载不在本例范围。`mod_has_effigy` 可替换为本模组命名空间下的 tag；本例不承诺客户端初始化时序或 UI 适配。[玩家 Hook](../../../raw/current/scripts/modutil.lua#L585-L590) [tag 接口](../../../raw/current/scripts/entityscript.lua#L556-L562) [任务入口](../../../raw/current/scripts/entityscript.lua#L1547-L1555) [移除清理](../../../raw/current/scripts/entityscript.lua#L1741-L1746)

### 需要实际建筑时沿用服务端查找

原版 `ACTIONS.REMOTERESURRECT` 在玩家是 ghost 时，分别查 `remoteresurrector` 和 `gravestoneresurrector`，有目标才推 `respawnfromghost`；这是后续业务消费者，不是查询方法自动触发的效果。自定义用途应另行定义行为与门禁，不给 `HasAttunement` 附加修改关系副作用。[动作实现](../../../raw/current/scripts/actions.lua#L4117-L4129)

## 常见陷阱与限制

- **客户端查其他玩家：** 只有本地玩家被保证拥有自己的代理，空结果不能证明他人未绑定。需要权威结果在服务端查对应玩家。[端约定](../../../raw/current/scripts/components/attuner.lua#L1-L11) [专属目标](../../../raw/current/scripts/prefabs/attunable_classified.lua#L17-L22)
- **用失去事件直接清状态：** 新旧来源替换时可同时存在；客户端同帧 gained／lost 顺序不可靠。每次重新 HasAttunement，需要类别检查用 `data.proxy:IsAttunableType(tag)`，不要把 proxy 当目标建筑。[顺序说明](../../../raw/current/scripts/components/attuner.lua#L58-L79) [代理类型方法](../../../raw/current/scripts/prefabs/attunable_classified.lua#L24-L26)
- **以为类别等价等于查询别名：** 两种复活 tag 在目标侧互斥，但 HasAttunement 仅精确匹配当前代理 tag；查询两类须逐一 OR。[精确查询](../../../raw/current/scripts/components/attuner.lua#L26-L33) [目标替换条件](../../../raw/current/scripts/components/attunable.lua#L20-L30)
- **手工构造 proxy／改 GUID 映射：** 注册函数只更新本地索引和事件，不建立目标端关系与存档。实际绑定走目标 `attunable:LinkToPlayer`，恢复见其生命周期说明。[注册函数](../../../raw/current/scripts/components/attuner.lua#L67-L79) [真实链接](../../../raw/current/scripts/components/attunable.lua#L93-L118)
- **把 TransferComponent 当无条件复制：** 实现直接访问 Ents 的来源和 attunable，且要求新玩家满足 LinkToPlayer 门禁；不包含缺实体、无 userid 或缺组件的防御处理，不供任意对象调用。[实现](../../../raw/current/scripts/components/attuner.lua#L49-L56) [目标门禁](../../../raw/current/scripts/components/attunable.lua#L86-L91)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：actions.lua（R-01）、prefabs/player_common.lua（P-83）、entityscript.lua（R-08）对照 build-26148036 更新报告逐条核验，锚点重定位 6 处、结论修订 0 处：REMOTERESURRECT（actions.lua:4107-4119）、attuner 装配／pristine 端分界、SwapAllCharacteristics、DoTaskInTime 与 onremove 清理均逐行无变化（R-08 改动集中于 GetWet 系与距离函数，P-83 仅删 worm_boss 音效 netvar，均不涉及本页语义）；未覆盖：运行验证。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；完整核验 attuner、classified，核对 attunable 的关系／存读、player_common 双端装配与变化监听／激活顺序／角色转移、远程复活动作；demo 按实际玩家 Hook、查询、事件与实体接口审阅。
- 游戏运行：未运行；未验证客户端远距离显示、替换帧事件排列和自定义 tag 的实际消费者。示例仅维护服务端标志。
- 结构检查：定向页面格式／链接与源码引用 strict 检查通过；上例通过 Lua 5.1 语法检查，不代替 API／时序运行测试。
