---
title: "container_proxy"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "将实体的开箱入口代理到共享主容器，处理绑定、打开者同步与先关闭后解绑。"
aliases: ["container_proxy", "共享容器入口", "远端容器代理", "代理绑定解除"]
tags: [component, lifecycle, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/container_proxy.lua", "components/container.lua", "components/inventory.lua", "prefabs/container_opener.lua", "prefabs/magician_chest.lua", "prefabs/chester.lua", "prefabs/hats.lua", "componentactions.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# container_proxy

## 快速判断

- 职责与适用：实体提供开箱位置／会话，实际格子在 `master.components.container`；多个入口可共享同一主容器，代理没有自己的物品槽。[绑定](../../../raw/current/scripts/components/container_proxy.lua#L72-L98) [打开](../../../raw/current/scripts/components/container_proxy.lua#L110-L140)
- 优先使用：普通箱子先选 [[reference/components/container]]；需要原版魔术箱、暗影切斯特的共享入口才使用代理。注册名分别是 `magician_chest` 与 `chester`，后者按形态切换。[魔术箱工厂](../../../raw/current/scripts/prefabs/magician_chest.lua#L187-L198) [切换](../../../raw/current/scripts/prefabs/chester.lua#L386-L407)
- 重要副作用：打开先在主实体发 `onopenother`，其他代理据此关闭同一玩家的旧会话；成功后生成定向给该玩家的 `container_opener`，登记人物库存的 `opencontainerproxies`。[打开](../../../raw/current/scripts/components/container_proxy.lua#L110-L140) [监听](../../../raw/current/scripts/components/container_proxy.lua#L83-L96)
- 修改边界：组件含网络变量，原宿主在双方、`SetPristine` 前添加；绑定和开关业务在服务端。不要在 postinit 中给任意新宿主补此组件或移除它。[原安装](../../../raw/current/scripts/prefabs/magician_chest.lua#L148-L168) [移除断言](../../../raw/current/scripts/components/container_proxy.lua#L28-L37)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetMaster(ent)`／`GetMaster()` | 服务端绑定具有 container 的实体；nil 解除监听。只替换绑定，不自动关闭旧会话，也不监听主实体删除；主实体有效性由宿主管理。[实现](../../../raw/current/scripts/components/container_proxy.lua#L67-L98) |
| `SetCanBeOpened(bool)`／`CanBeOpened()` | 网络布尔控制候选动作；`Open` 不读取此开关。直接调用仍需自己校验有效主容器、资格与业务条件。[属性](../../../raw/current/scripts/components/container_proxy.lua#L39-L45) [候选](../../../raw/current/scripts/componentactions.lua#L305-L311) [Open](../../../raw/current/scripts/components/container_proxy.lua#L110-L117) |
| `Open(doer)`／`Close(doer?)` | 服务端调用，无成功布尔返回；打开后用 `IsOpenedBy` 确认。Close 无参数关闭此代理的全部打开者，经主容器 `onclose` 回调清理；指定 doer 会直接请求主容器关闭该人。[打开](../../../raw/current/scripts/components/container_proxy.lua#L110-L140) [关闭](../../../raw/current/scripts/components/container_proxy.lua#L142-L173) |
| `SetOnOpenFn(fn)`／`SetOnCloseFn(fn)` | 回调 `(proxyinst)`；首次打开／最后关闭才调用，无 doer 参数。扩展已有宿主时保留原动画／声音回调。[设置](../../../raw/current/scripts/components/container_proxy.lua#L100-L108) [调用](../../../raw/current/scripts/components/container_proxy.lua#L130-L138) [关闭回调](../../../raw/current/scripts/components/container_proxy.lua#L165-L171) |
| `IsOpenedBy(guy)`／`AttachOpener(opener)` | 服务端查 openlist；客户端仅能确认本地玩家。AttachOpener 是客户端复制流程，不由模组手动创建会话。[判断](../../../raw/current/scripts/components/container_proxy.lua#L47-L62) [消费者](../../../raw/current/scripts/prefabs/container_opener.lua#L7-L18) |
| `OnRemoveEntity()`／`OnUpdate(dt)` | 删除代理实体先 Close 再解绑；有 Transform 时首开启动更新，用主 container 的 OnUpdate 函数、以代理自己作为 self 检查距离与视线。[删除](../../../raw/current/scripts/components/container_proxy.lua#L32-L37) [更新](../../../raw/current/scripts/components/container_proxy.lua#L175-L183) [检查](../../../raw/current/scripts/components/container.lua#L1059-L1084) |

普通 RUMMAGE 会先解析代理的 master，再处理主容器权限／开关；库存按 master 查当前代理用于位置等业务。主容器布局、内容转移与存档细节复用 [[reference/components/container]]。[动作入口](../../../raw/current/scripts/actions.lua#L1057-L1095) [库存查询](../../../raw/current/scripts/components/inventory.lua#L2747-L2753)

## 典型接入

### 服务端暂时停用一个原版魔术箱入口

目标模组 `modmain.lua`，复用 `magician_chest` 的资源、动画和共享 shadow 主容器。只为实际魔术箱增加私有事件；不改变其他入口。待该宿主原加载／绑定完成后，由服务端业务发送 `my_mod_suspend_proxy` 或 `my_mod_resume_proxy`。

```lua
AddPrefabPostInit("magician_chest", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_suspend_proxy", function(chest)
        local proxy = chest.components.container_proxy
        if proxy:GetMaster() ~= nil then
            proxy:Close()
        end
        proxy:SetCanBeOpened(false)
        proxy:SetMaster(nil)
    end)
    inst:ListenForEvent("my_mod_resume_proxy", function(chest)
        local proxy = chest.components.container_proxy
        if proxy:CanBeOpened() then return end
        local master = GLOBAL.TheWorld:GetPocketDimensionContainer("shadow")
        if master == nil or not master:IsValid()
            or master.components.container == nil then return end
        proxy:SetMaster(master)
        proxy:SetCanBeOpened(true)
    end)
end)
```

先 Close 才能借旧主容器事件删 opener、清库存登记并执行原最后关闭回调；再禁候选和解绑。恢复只重绑，不自动重开，原 onopenfn／onclosefn 未被覆盖。原版切斯特使用相同的关闭→禁用→解绑顺序。[原顺序](../../../raw/current/scripts/prefabs/chester.lua#L386-L404) [代理清理](../../../raw/current/scripts/components/container_proxy.lua#L153-L173)

此暂停状态仅当前实例／本次运行有效；组件没有 OnSave／OnLoad，原魔术箱在非 POPULATING 时绑定、读档在 OnLoadPostPass 重绑。停用模组并重载后回到原绑定。私有监听随宿主删除清理；不移除组件、不删除共享主实体。存档持久化暂停不在示例范围内。[原加载](../../../raw/current/scripts/prefabs/magician_chest.lua#L187-L193) [完整组件](../../../raw/current/scripts/components/container_proxy.lua#L1-L185)

原版另一模式为 `tophat_container`：帽子工厂注册的隐藏临时代理、无 Transform 且 persists=false；其会话由工具开始／停止管理，见 [[reference/components/magician]]，不能假定它靠距离更新自动关闭。[构造](../../../raw/current/scripts/prefabs/hats.lua#L7834-L7854) [注册](../../../raw/current/scripts/prefabs/hats.lua#L8032-L8035)

## 常见陷阱与限制

- **开着时直接 SetMaster(nil)／换主实体**：旧 opener、openlist 和库存登记不会因此清掉；以后关闭又依赖新 master。先 Close，确认会话清理后再换绑；原 master 已被外部删除的损坏场景不由此方法修复。[绑定](../../../raw/current/scripts/components/container_proxy.lua#L72-L98) [关闭](../../../raw/current/scripts/components/container_proxy.lua#L142-L173)
- **RemoveComponent 当作正常解绑**：OnRemoveFromEntity 明确断言失败；仅停用用关闭／禁用／解绑，销毁入口用实体原删除流程。[实现](../../../raw/current/scripts/components/container_proxy.lua#L28-L37)
- **禁用就等于已关闭**：SetCanBeOpened 只改网络开关，不清现有会话；直接 Open 也不检查它。关闭当前用户须显式 Close。[开关](../../../raw/current/scripts/components/container_proxy.lua#L39-L45) [打开](../../../raw/current/scripts/components/container_proxy.lua#L110-L117)
- **解绑后照常调 Open／Close(doer)**：非空 doer 的 Open（尚未登记）和 Close 都会解引用 master，nil 时可报错；Close() 仅在 openlist 为空时不会触碰 master。停用业务必须同时阻止外部直接开关入口，恢复有效绑定后才允许调用。[实现](../../../raw/current/scripts/components/container_proxy.lua#L110-L151)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的漂移文件（actions.lua、componentactions.lua、prefabs/chester.lua、prefabs/hats.lua）对照 build-26148036 更新报告（R-01/R-04/P-30/P-56/P-57）逐条核验：RUMMAGE fn 与 container_proxy 候选判定逐行未变，chester.lua 仅 :624 新增 `nolifedrainable` 标签（不动 386-407 的形态切换），hats.lua 的 tophat_container 构造与注册逐行未变、仅被新增帽子代码整体后移；锚点重定位 3 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖完整代理组件、opener 复制、RUMMAGE 入口、库存查询、魔术箱注册／绑定／读档与切斯特解除顺序，以及高礼帽临时代理构造。主容器通用内容操作复用同版本 container 页；未审校全部口袋维度存档和世界销毁顺序。
- 结构检查：本页元数据、链接、源码行号与大小通过；完整 modmain 示例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；多人同时开关、距离关闭、暂停恢复、读档与其他模组修改仍待游戏验证。
