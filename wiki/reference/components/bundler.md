---
title: "bundler"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "管理人物的临时打包容器、两阶段完成、取消返还与中途存档恢复。"
aliases: ["人物打包", "每包至少两格", "取消打包返还", "bundler"]
tags: [component, item, lifecycle, state-machine]
dst_build: "steam-buildid:26230496"
sources: ["components/bundler.lua", "components/bundlemaker.lua", "prefabs/bundle.lua", "prefabs/player_common.lua", "actions.lua", "stategraphs/SGwilson.lua", "containers.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# bundler

## 快速判断

- 职责与适用：挂在人物上的打包过程管理；材料配置看 [[reference/components/bundlemaker]]，序列化／拆包看 [[reference/components/unwrappable]]。原版人物已有本组件。[人物装配](../../../raw/current/scripts/prefabs/player_common.lua#L2716-L2721)
- 优先使用：正常交互沿用 BUNDLE／WRAPBUNDLE 与 SG；额外完成条件包裹 FinishBundling，取消用 StopBundling，不直接移除临时容器。[动作入口](../../../raw/current/scripts/actions.lua#L4347-L4370)
- 重要副作用：开始打开并挂接临时容器，材料消费由材料回调完成；FinishBundling 只请求动画阶段，OnFinishBundling 才生成成品并保存内容。[开始](../../../raw/current/scripts/components/bundler.lua#L20-L41)／[两阶段](../../../raw/current/scripts/components/bundler.lua#L96-L139)
- 修改边界：服务端，要求兼容的 sg／人物收纳及动作；给普通生物挂组件不等于拥有原版打包 UI／状态机。[状态要求](../../../raw/current/scripts/components/bundler.lua#L8-L17)／[SG 完成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19908-L19912)

## 关键接口

| 接口 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `CanStartBundling()` | bool | 当前 SG 必须为 bundle，且无临时实例／材料记录；不是 idle 时可直接询问的通用能力判定。[条件](../../../raw/current/scripts/components/bundler.lua#L8-L11) |
| `StartBundling(item)` | 成功 true，否则 nil | 读取材料两个 Prefab；先 Stop 旧过程，再尝试生成／打开容器，成功后保存材料名／皮肤及成品配置并切 SG；不自行调用 CanStart 或检查可见性。[开始](../../../raw/current/scripts/components/bundler.lua#L20-L45)／[上层门禁](../../../raw/current/scripts/actions.lua#L4347-L4357) |
| `IsBundling(container_inst)` | bool | 指定实体正是临时实例，且 SG 为 bundling。[判断](../../../raw/current/scripts/components/bundler.lua#L14-L17) |
| `FinishBundling()` | 请求成功 true，否则 nil | 容器非空、有成品名、处于 bundling；关闭容器并转 bundle_pst，不立即产出成品。[请求](../../../raw/current/scripts/components/bundler.lua#L96-L105) |
| `OnFinishBundling()` | SG 结算入口 | 生成带 unwrappable 的成品，WrapItems 后移除临时容器并清会话，再给人物或落地；不是任意网络请求可直接暴露的接口。[结算](../../../raw/current/scripts/components/bundler.lua#L109-L139) |
| `StopBundling()` | 取消／清理 | 临时内容逐格归还 inventory，无 inventory 则落地；材料按已记 Prefab 重建，非恢复原实体及任意自定义状态。[取消](../../../raw/current/scripts/components/bundler.lua#L60-L93) |

## 典型接入

### 人物每次打包至少放两格物品

目标模组 `modmain.lua`，复用原版人物与包装材料，不新增资源；在完成请求前补“占用格数”条件，再调用原方法保留状态检查与动画。AddPlayerPostInit 在人物工厂后接入，原版此时已有 bundler。[注册入口](../../../raw/current/scripts/modutil.lua#L586-L589)／[人物组件](../../../raw/current/scripts/prefabs/player_common.lua#L2720)／[原完成请求](../../../raw/current/scripts/components/bundler.lua#L96-L105)

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local bundler = inst.components.bundler
    if bundler == nil then return end

    local old = bundler.FinishBundling
    bundler.FinishBundling = function(self, ...)
        local pack = self.bundlinginst
        local container = pack ~= nil and pack.components.container or nil
        local occupied = 0
        if container ~= nil then
            for slot = 1, container:GetNumSlots() do
                if container:GetItemInSlot(slot) ~= nil then
                    occupied = occupied + 1
                end
            end
        end
        if occupied < 2 then return false end
        return old(self, ...)
    end
end)
```

计占用格，不计堆叠件数；同一格放两件仍拒绝。影响该人物正常完成请求（含礼物），不改变容器容量／过滤；按钮仍按原版“非空”显示可用，一格时服务端拒绝且本例不新增提示。它不是所有自定义 OnFinishBundling 调用的统一门禁。[按钮条件](../../../raw/current/scripts/containers.lua#L486-L487)

本例不替换 OnSave／OnLoad，不改变取消返还；停用后重新创建／读档的人物恢复原方法。其他模组若后续覆盖方法仍需协调；测试一格堆叠、两格、空包、取消、中途存档与动画被打断。

## 生命周期

原版 SG 从 bundle 动画执行 BUNDLE，Start 成功转 bundling；完成请求转 bundle_pst，超时回调结算。离开 bundling 且未标记转阶段时 Stop，完成动画在结算前中断也 Stop；人物离场另主动 Stop，组件移除／实体移除同样绑定 Stop。[执行动作](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19803-L19822)／[退出与结算](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19881-L19928)／[离场](../../../raw/current/scripts/prefabs/player_common.lua#L1434-L1437)／[移除](../../../raw/current/scripts/components/bundler.lua#L191-L192)

OnSave 保存材料／成品名、成品皮肤与非空临时容器记录；不保存原包装材料完整实体记录。OnLoad 重建临时容器并挂接人物，不自行打开 UI／恢复 bundling SG；有现行会话时先处理载入记录，再还原现行引用，不能简化成“重登继续操作同一个窗口”。[保存](../../../raw/current/scripts/components/bundler.lua#L143-L156)／[恢复](../../../raw/current/scripts/components/bundler.lua#L159-L186)

## 常见陷阱与限制

- **Finish 返回 true 就物品到账：** 它只进入动画，结算还有 SpawnPrefab、unwrappable 和临时内容条件；失败不等于完成。不要收到 true 就另发一份包裹。[请求](../../../raw/current/scripts/components/bundler.lua#L96-L105)／[结算](../../../raw/current/scripts/components/bundler.lua#L109-L139)
- **取消能保留材料任意属性：** Stop 用材料名／皮肤重新 Spawn，不还原原材料完整保存数据；自定义耐久／名字／状态需要另设计，不能直接套此取消契约。[记录与消费](../../../raw/current/scripts/components/bundler.lua#L33-L38)／[材料重建](../../../raw/current/scripts/components/bundler.lua#L80-L92)
- **临时容器没有输入限制：** 原版过滤 irreplaceable、_container、bundle、nobundling；改过滤涉及嵌套与不可替代物保存，不只增加一个 UI 格。[过滤](../../../raw/current/scripts/containers.lua#L474-L475)；布局另见 [[reference/components/container]]。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的漂移文件（actions.lua、prefabs/player_common.lua、stategraphs/SGwilson.lua、containers.lua）对照 build-26148036 更新报告（R-01/R-07/SB-17/P-83）逐条核验：BUNDLE／WRAPBUNDLE 动作、人物 bundler 装配与离场 Stop、SG bundle／bundle_pst 两阶段状态、bundle_container 过滤与按钮条件均逐行未变（player_common 删行仅 wormdigestionsound 迁移、SGwilson 增删集中在 Rifts8 新状态与事件）；锚点重定位 7 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖人物装配、动作／SG 两阶段、开始消费／取消／保存与移除、临时容器及两格示例；未完整审校客户端预测、角色形态与全部迁移调用。
- 结构／示例检查：本批格式、来源文件、链接／行锚点、导航登记及完整 Lua 5.1 示例语法检查通过。
- 游戏运行：未运行；UI 提示、多人／专服、动画中断、保存／迁移与模组方法叠加需验证。
