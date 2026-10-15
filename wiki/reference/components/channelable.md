---
title: "channelable"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "建筑持续交互的参与者与 SG／回调桥接；准入、停止、多人集合和宿主清理分别管理。"
aliases: ["服务端中止沙漠传送阵调谐", "建筑持续交互", "channelable"]
tags: [component, building, actions, callback, lifecycle, state-machine]
dst_build: "steam-buildid:26230496"
sources: ["components/channelable.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua", "prefabs/townportal.lua", "prefabs/waterpump.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# channelable

## 快速判断

- 职责与适用：装在持续交互建筑，保存单人 channeler 或多人集合，通过玩家 prechanneling／channeling 状态启动、停止并调用宿主效果；原沙漠传送阵和水泵使用不同状态适配。[默认](../../../raw/current/scripts/components/channelable.lua#L30-L53) [开始](../../../raw/current/scripts/components/channelable.lua#L119-L147) [宿主配置](../../../raw/current/scripts/prefabs/townportal.lua#L213-L219)
- 优先使用：SetChannelingFn(startfn,stopfn) 保留效果与撤销；正常启动经 STARTCHANNELING／SG，主动中止用 StopChanneling(true)，不手改参与者引用。（26148036 核验：R-01 STARTCHANNELING／STOPCHANNELING 定义加 `invalid_hold_action=true`，不可长按连发，消费点 playercontroller.lua:4623）[接口](../../../raw/current/scripts/components/channelable.lua#L94-L117) [动作](../../../raw/current/scripts/actions.lua#L4297-L4309) [禁长按](../../../raw/current/scripts/actions.lua#L485-L486)
- 重要副作用：开始关联移除监听、可切 SG、调 startfn 并启动更新；停止先标记／切结束状态，再调用 stopfn，之后才清参与者。**清引用在 stopfn 存在的分支里**，不能为关闭效果删掉停止回调。[开始](../../../raw/current/scripts/components/channelable.lua#L126-L145) [停止](../../../raw/current/scripts/components/channelable.lua#L164-L209)
- 修改边界：SetEnabled 只改准入及 channelable 标签，不中止已在进行的交互；停止参数 aborted 只转交宿主，不自动结算成功／失败奖励。本文不覆盖玩家手持物品的 channelcast 流程。[开关](../../../raw/current/scripts/components/channelable.lua#L86-L97) [回调参数](../../../raw/current/scripts/components/channelable.lua#L189-L204)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetEnabled(bool)`／`GetEnabled()` | 默认 true，联动 channelable 标签；已有交互的 OnUpdate 不读 enabled。禁用与中止若都需要，按宿主协议分别处理。[属性](../../../raw/current/scripts/components/channelable.lua#L10-L16) [开关](../../../raw/current/scripts/components/channelable.lua#L86-L92) [更新](../../../raw/current/scripts/components/channelable.lua#L212-L216) |
| `SetChannelingFn(startfn,stopfn)` | startfn(target,channeler)，stopfn(target,aborted,channeler)；开始回调返回值不作为是否成功的判定，停止回调后才清引用／集合。保留原回调，不替换成空值。[设置](../../../raw/current/scripts/components/channelable.lua#L94-L97) [开始回调](../../../raw/current/scripts/components/channelable.lua#L136-L145) [停止回调](../../../raw/current/scripts/components/channelable.lua#L189-L204) |
| `IsChanneling(targetchanneler?)` | 要参与者 SG 有 channeling 状态标签；多人可查指定参与者或任一个，单人只看当前 channeler，不按传入对象过滤。原始引用与实际状态不等价。[查询](../../../raw/current/scripts/components/channelable.lua#L99-L117) |
| `StartChanneling(channeler)` | enabled、没有对应当前交互（ignore_prechannel 可绕过该项）、有效 SG 且有 prechanneling 标签或 skip_state_channeling 才进入；成功 true，不满足通常 nil。不是任意 idle 实体的自动启动器。[资格](../../../raw/current/scripts/components/channelable.lua#L119-L147) |
| `StopChanneling(aborted,targetchanneler?)` | 单人停当前，多人可停指定或全部；先注销移除监听，设置 SG statemem.stopchanneling 防退出回调重入，再进入 stopchanneling（可配置跳过）。没有“当前参与者才允许调用”的统一授权检查。[顺序](../../../raw/current/scripts/components/channelable.lua#L149-L209) |
| 多人模式 | SetMultipleChannelersAllowed 负责标记和集合；构造初始是 nil，不是 false／空表。方法先赋新模式再处理旧参与者，活跃期切换不视为安全迁移；未开启时也不要盲调 false，可能 next(nil)。[默认](../../../raw/current/scripts/components/channelable.lua#L30-L45) [切换](../../../raw/current/scripts/components/channelable.lua#L69-L84) |
| 特殊状态字段 | use_channel_longaction 联动标签；skip_state_channeling／skip_state_stopchanneling／ignore_prechannel 为宿主适配字段。原水泵组合这些字段并监听 channel_finished，不能只复制其中一个就套到普通传送阵。[水泵组装](../../../raw/current/scripts/prefabs/waterpump.lua#L247-L256) |

## 典型接入

### 为原沙漠传送阵提供服务端中止当前调谐事件

目标模组 `modmain.lua`，无新资源。调用方在服务端对目标阵发 `portal:PushEvent("my_mod_abort_townportal_channel")`；仅交付原单人宿主接收端，不提供 UI、RPC、权限、禁止重新调谐或立即传送。

```lua
AddPrefabPostInit("townportal", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_abort_townportal_channel", function(portal)
        local channel = portal.components.channelable
        if channel ~= nil and channel.channeler ~= nil then
            channel:StopChanneling(true)
        end
    end)
end)
```

保留原 stopfn：它通知世界停用、恢复图标并撤销人物精神持续修正；不回退开始时已经扣掉的一次精神。引用检查也允许清理已离开 channeling 状态但尚登记的单人；不把这段示例用于多人模式。结束后仍能按原条件重新开始，传送配对可由宿主再决定准入。[原开始／停止](../../../raw/current/scripts/prefabs/townportal.lua#L46-L82) [配对消费者](../../../raw/current/scripts/prefabs/townportal.lua#L84-L101)

本组件没有存读，事件不持久记录；宿主重建自己的效果和可用性，停用重载移除接收端，不恢复已中止过程。传送执行另见 [[reference/components/teleporter]]，精神修正另见 [[reference/components/sanity]]，不把中止当退费或配对清空。

### 生命周期与原版协作

正常玩家 SG 的 channeling 更新检查视线，退出时在未标 stop／keep 且目标有效时调用 StopChanneling(true,player)；组件先设置 stopchanneling 标志以避开重复退出调用。组件 OnUpdate 发现无人有实际 channeling 状态时也中止。（26148036 核验：channeling 状态本体未变，仅其 vault_teleport 事件处理随虚拟房间迁移新增 fxprefab／return true，与组件协作无关）[SG 协作](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20137-L20176) [更新](../../../raw/current/scripts/components/channelable.lua#L212-L216)

参与者 onremove 监听只清引用／集合，不立刻调宿主 stopfn；移除组件会尝试中止登记者、停更新并去标签。热移除、多人切换和玩家删除仍要核验宿主效果是否完整撤销。水泵的 stopfn 另取消 animover 与投射物任务，属于实例清理，不复制进通用组件。[引用移除](../../../raw/current/scripts/components/channelable.lua#L39-L45) [组件移除](../../../raw/current/scripts/components/channelable.lua#L55-L67) [水泵停止](../../../raw/current/scripts/prefabs/waterpump.lua#L195-L207)

## 常见陷阱与限制

- **SetEnabled(false) 自动结束并撤销收益**：只影响准入／标签；中止与宿主回调需另走 StopChanneling，不提前抹掉回调或引用。[开关](../../../raw/current/scripts/components/channelable.lua#L86-L97)
- **删掉 stopfn 只是关掉停止特效**：当前实现清引用也依赖该回调分支，可能留下 channeled 标签／旧引用；保留完整宿主协议。[清理分支](../../../raw/current/scripts/components/channelable.lua#L189-L209)
- **SetMultipleChannelersAllowed(false) 总是安全初始化**：初始表 nil，false 分支直接 next；启用／禁用和活跃期模式切换按具体实现处理，不能把它当无副作用 setter。[模式实现](../../../raw/current/scripts/components/channelable.lua#L69-L84)
- **开始回调 return false 就拒绝启动**：组件不消费该返回；拒绝条件在进入前处理，不能在已开始／切 SG 后只返回 false。[回调消费](../../../raw/current/scripts/components/channelable.lua#L126-L145)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中变更文件 actions.lua、componentactions.lua、stategraphs/SGwilson.lua（channelable.lua／townportal.lua／waterpump.lua 本版未变）对照 build-26148036 更新报告核验，条目 R-01：STARTCHANNELING／STOPCHANNELING 加 `invalid_hold_action=true` 禁长按连发；componentactions 无 channelable 相关处理器变化，SGwilson channeling 状态本体未变（仅 vault_teleport 事件随虚拟房间迁移变化）；锚点重定位 2 处、结论修订 1 处；未覆盖：运行验证。
- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、单人／多人引用、开始／停止／标签／更新、STARTCHANNELING／STOPCHANNELING、玩家 SG 相关状态、传送阵回调／配对和水泵特殊配置／停止清理。未全面审校传送系统配对、多人宿主、全部预测／SG 或模式切换迁移。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整中止事件例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；玩家中止／打断／视线丢失、精神修正清理、配对／存读、多人模式和热移除待实测。
