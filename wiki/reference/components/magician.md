---
title: "magician"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "人物暂存魔术工具、成对启动／停止与归还装备；关联工具回调、原高礼帽 SG 和读档收尾。"
aliases: ["服务端结束麦斯威尔魔术工具会话", "人物魔术工具暂存", "magician"]
tags: [component, callback, lifecycle, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/magician.lua", "components/magiciantool.lua", "prefabs/waxwell.lua", "prefabs/hats.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# magician

## 快速判断

- 职责与适用：人物侧暂存一个带 [[reference/components/magiciantool]] 的物品，取出原持有／堆叠、设父实体、移出场景并调用工具回调；原麦斯威尔已有组件。不是通用施法或普通装备栏容器。[开始](../../../raw/current/scripts/components/magician.lua#L44-L76) [原安装](../../../raw/current/scripts/prefabs/waxwell.lua#L269-L283)
- 优先使用：正常交互走 USEMAGICTOOL／原 SG，服务端结束用 StopUsing；不要直接把 item=nil 或把物品重新 GiveItem，绕过工具业务收尾。[动作](../../../raw/current/scripts/actions.lua#L5892-L5907) [停止](../../../raw/current/scripts/components/magician.lua#L78-L117)
- 重要副作用：使用中物品 persists=false、在人物父实体下且不在场景；StopUsing 先清人物引用，再通知工具，恢复场景／持久化，尝试回穿或给人物物品栏，最后发 magicianstopped。[开始](../../../raw/current/scripts/components/magician.lua#L60-L75) [归还](../../../raw/current/scripts/components/magician.lua#L78-L117)
- 修改边界：原组件恢复存档只重建物品和 held／equip，不重新执行开始握手；麦斯威尔在库存恢复后的 OnLoad 主动 StopUsing。没有这个宿主步骤不能称为完整读档恢复。[存读](../../../raw/current/scripts/components/magician.lua#L119-L138) [宿主](../../../raw/current/scripts/prefabs/waxwell.lua#L173-L177)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `StartUsingTool(item)` | 已有 item 或目标缺 magiciantool 返回 false；有末端持有者时要求其 inventory／container 对本人 IsOpenedBy。目标需符合该宿主假定，方法不完整核验距离／角色技能或对端 user 空闲。[准入](../../../raw/current/scripts/components/magician.lua#L44-L58) |
| `item`／`held`／`equip` | item 可为从堆叠拆出的实体；held 记有持有来源，equip 记原装备状态，不是原格子／原箱子位置。属性维护 usingmagiciantool 两类标签。[属性](../../../raw/current/scripts/components/magician.lua#L1-L30) [取物](../../../raw/current/scripts/components/magician.lua#L60-L75) |
| `StopUsing()` | 无物品返回 false；有物品时调工具 OnStopUsing、恢复 parent／场景／persists。原本持有则尝试回穿空槽或归人物库存，其他分支用 inventoryitem 掉落；不保证还回原容器或原槽。[方法](../../../raw/current/scripts/components/magician.lua#L78-L117) |
| `DropToolOnStop()` | 仅清 held／equip，改变下一次停止的归还策略；不会立刻停止或扔出物品。原 SG 中断及工具删除使用它。[方法](../../../raw/current/scripts/components/magician.lua#L39-L42) [SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22998-L23017) [工具删除](../../../raw/current/scripts/components/magiciantool.lua#L15-L22) |
| `magicianstopped` | 在人物上发、无业务 data，发生在工具停止与物品归还之后；原使用状态监听它退出 idle，不等同完整停止动画。[事件](../../../raw/current/scripts/components/magician.lua#L114-L117) [消费者](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22978-L22981) |
| 移组件 | OnRemoveFromEntity 先 StopUsing 再去相关标签；清理仍依赖实际工具／库存链，不只是一组标签。[实现](../../../raw/current/scripts/components/magician.lua#L32-L37) |

原魔术高礼帽在工具开始时创建并打开 container_proxy、播放循环声，停止时关闭／移除代理并停声；人物组件不替它实现这些效果。容器关闭也反向请求工具停止，人物先清 item 的顺序有助于避免重复停止同一物品。[原业务](../../../raw/current/scripts/prefabs/hats.lua#L906-L934) [顺序](../../../raw/current/scripts/components/magician.lua#L78-L97)

## 典型接入

### 服务端结束麦斯威尔当前魔术工具会话

目标模组 `modmain.lua`，无新资源；保留原人物组件、帽子转换和工具回调，不改变普通玩家资格或伪造 USEMAGICTOOL。

```lua
AddPrefabPostInit("waxwell", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_stop_magic_tool", function(player)
        local magician = player.components.magician
        if magician ~= nil then
            magician:StopUsing()
        end
    end)
end)
```

服务端对实际玩家发 `player:PushEvent("my_mod_stop_magic_tool")` 才触发；不新增按键／RPC。沿原停止顺序处理工具、暂存物品与事件，已有高礼帽使用状态会监听 magicianstopped；重复调用在空状态返回 false。没有调用 DropToolOnStop，保留正常尝试归还／回穿策略；容量、已占装备槽及其他模组仍可能改变最终去向。[归还](../../../raw/current/scripts/components/magician.lua#L78-L117) [SG 消费](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22978-L22981)

停用并重载不再安装私有监听；组件原嵌套物品保存和原人物 OnLoad 收尾保持不变。这个入口不承诺播放手动停止动作的完整动画，也不修复其他模组已破坏的双向引用。[原存读](../../../raw/current/scripts/components/magician.lua#L119-L138) [人物加载](../../../raw/current/scripts/prefabs/waxwell.lua#L173-L177)

## 常见陷阱与限制

- **物品不在场景就可另存一份库存记录**：人物组件已嵌套 GetSaveRecord，工具又被设 persists=false；重复独立保存／生成可能破坏唯一物品关系，沿原保存链处理。[暂存](../../../raw/current/scripts/components/magician.lua#L62-L75) [保存](../../../raw/current/scripts/components/magician.lua#L119-L128)
- **DropToolOnStop 立即掉落并退出**：它只改后续停止策略；真正的回调、恢复、掉落和事件在 StopUsing。[两方法](../../../raw/current/scripts/components/magician.lua#L39-L42) [停止](../../../raw/current/scripts/components/magician.lua#L78-L117)
- **STOPUSINGMAGICTOOL 返回 true 就代表停了东西**：动作固定返回 true，组件无 item 时实际返回 false；判断业务状态不依赖这个动作结果。[动作](../../../raw/current/scripts/actions.lua#L5902-L5907) [组件](../../../raw/current/scripts/components/magician.lua#L78-L82)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中变更文件 actions.lua、componentactions.lua、stategraphs/SGwilson.lua、prefabs/waxwell.lua、prefabs/hats.lua（magician.lua／magiciantool.lua 本版未变）对照 build-26148036 更新报告核验：USEMAGICTOOL／STOPUSINGMAGICTOOL 及 magiciantool 动作候选无语义变化，SGwilson 魔术工具状态本体未变（整体行移），waxwell 变更（shadow_item 抗性限负值、stalkerinspectable）与 hats 变更均不触及魔术工具业务；锚点重定位 6 处、结论无修订；未覆盖：运行验证。
- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整人物／工具组件、麦斯威尔安装与 OnLoad、原帽子开始／停止业务、动作／候选及使用 SG 的停止事件／中断退出。未完整审校全部高礼帽 SG、容器代理实现、客户端预测、物品删除或库存不足的全部去向。
- 结构检查：本批格式、来源、源码引用／行号、大小和登记通过；完整人物停止事件例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；开始／重复停止、装备槽占用／满库存、容器关闭、存读／中断、主机／远端和其他模组回调待实测。
