---
title: "debuffable"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "管理目标的具名 Buff 集合、施加门禁与保存恢复，区分实体便捷接口和组件返回值。"
aliases: ["吃食物获得Buff", "吃胡萝卜加速", "Buff重复施加", "Buff管理", "debuffable"]
tags: [component, mechanism, callback, lifecycle, food, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/debuffable.lua", "components/debuff.lua", "components/edible.lua", "prefabs/veggies.lua", "prefabs/player_common.lua", "entityscript.lua", "mainfunctions.lua", "componentutil.lua", "mods.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# debuffable

## 快速判断

- 职责与适用：挂在**承受效果的目标**上，按管理键保存效果实体；效果工厂用 [[reference/components/debuff]]，倒计时用 [[reference/components/timer]]。原人物已具备管理器，不重复添加或覆盖原回调。[debuffable.lua:9](../../../raw/current/scripts/components/debuffable.lua#L9 [player_common.lua:2816](../../../raw/current/scripts/prefabs/player_common.lua#L2816
- 优先使用：目标:AddDebuff，默认检查启用／死亡／幽灵；Has／Get 查管理键，RemoveDebuff 成对撤销。实体／组件签名与返回不同，见[关键接口与证据](#关键接口)。
- 重要副作用：同键延展旧效果、忽略新 prefab；Enable(false) 移除全部当前效果，影响原版／其他模组。见[关键接口](#关键接口)。
- 修改边界：本页服务端集合／效果，不自动同步 UI；新网络目标需要时在 pristine 阶段显式加 debuffable tag，已有人物保留初始化／符号与通知。[debuffable.lua:16](../../../raw/current/scripts/components/debuffable.lua#L16 [player_common.lua:2544](../../../raw/current/scripts/prefabs/player_common.lua#L2544

## 关键接口

| 接口 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `target:AddDebuff(name, prefab, data?, skip_test?, pre_buff_fn?, buffer?)` | **boolean** | 自动加管理器；通过外层条件后返回 true，即使 Spawn 失败。pre_buff_fn 无参，先于组件施加；skip_test 跳外层条件，组件仍检查 enable。[entityscript.lua:2140](../../../raw/current/scripts/entityscript.lua#L2140 [debuffable.lua:95](../../../raw/current/scripts/components/debuffable.lua#L95 |
| `component:AddDebuff(name, prefab, data?, buffer?)` | 效果实体或 nil | 仅 enable 门禁，没有死亡／幽灵检查；新键 Spawn＋Attach，旧键 Extend。缺 debuff 的实体被删，返回的实体不证明附着有效。[debuffable.lua:72](../../../raw/current/scripts/components/debuffable.lua#L72 [debuffable.lua:95](../../../raw/current/scripts/components/debuffable.lua#L95 |
| `target:HasDebuff(name)`／`target:GetDebuff(name)` | boolean／实体或 nil | 查管理键，不查 Prefab 名；目标无管理器也可调用。需要确认生成后的有效效果，用 Get 后检查有效性。[entityscript.lua:2130](../../../raw/current/scripts/entityscript.lua#L2130 |
| `target:RemoveDebuff(name)` | 缺管理器／键无动作 | 先移集合和 onremove 监听，再宿主 ondebuffremoved，最后效果 OnDetach。有效 debuff 的实体清理由 Detached 决定。[debuffable.lua:110](../../../raw/current/scripts/components/debuffable.lua#L110 [debuff.lua:44](../../../raw/current/scripts/components/debuff.lua#L44 |
| `Enable(bool)`／`IsEnabled()` | 设置／boolean；默认 true | false 不只是拒绝新效果，还删除当前全部键；直接写 enable 不执行该删除循环。[debuffable.lua:1](../../../raw/current/scripts/components/debuffable.lua#L1 [debuffable.lua:28](../../../raw/current/scripts/components/debuffable.lua#L28 |
| `SetFollowSymbol(symbol, x, y, z)` | 定位参数 | 更新共享 offset 并通知效果 ChangeFollowSymbol；不自行挂动画／移动实体。[debuffable.lua:51](../../../raw/current/scripts/components/debuffable.lua#L51 |
| `ondebuffadded`／`ondebuffremoved` | `(target, name, effect, data, buffer)`／`(target, name, effect)` | 添加通知晚于 Attached；主动删除通知早于 Detached。单字段，已有人物回调不能直接覆盖。[debuffable.lua:86](../../../raw/current/scripts/components/debuffable.lua#L86 [debuffable.lua:115](../../../raw/current/scripts/components/debuffable.lua#L115 |

## 保存恢复与移除

OnSave 按键取效果 GetSaveRecord，并返回 add_component_if_missing。OnLoad 对不存在的键 SpawnSaveRecord，效果自身组件先 OnLoad，之后再 AttachTo；**不重传施加时的 data／buffer，也不检查 enable**，目标已有同键保留。可变状态保存到效果自己的数据，附着时重施来源，计时不能重置保存剩余。[debuffable.lua:126](../../../raw/current/scripts/components/debuffable.lua#L126 [debuffable.lua:139](../../../raw/current/scripts/components/debuffable.lua#L139 [mainfunctions.lua:503](../../../raw/current/scripts/mainfunctions.lua#L503 [[reference/components/debuff#三十秒加速效果并成对撤销]]

原人物 OnDespawn 在保存／删除前调用 RemoveOnDespawn，移除未设 keepondespawn 的效果；这不是死亡清理。效果自己监听 death 并 Stop；自建目标若需要同策略，也须接入实际宿主清理路径。[debuffable.lua:39](../../../raw/current/scripts/components/debuffable.lua#L39 [player_common.lua:1415](../../../raw/current/scripts/prefabs/player_common.lua#L1415

组件没有移除时逐项 Detached 的钩子；主动 RemoveComponent("debuffable") 前先逐键 RemoveDebuff，或按玩法接受 Enable(false) 清空全部，不删管理器来撤销一个 Buff。TransferComponent 用保存数据在新目标复制／恢复，不清旧集合，目标同键跳过；OnSave 丢弃效果 GetSaveRecord 的 refs，不能据此承诺任意外部实体引用都正确迁移。[debuffable.lua:110](../../../raw/current/scripts/components/debuffable.lua#L110 [debuffable.lua:133](../../../raw/current/scripts/components/debuffable.lua#L133 [debuffable.lua:152](../../../raw/current/scripts/components/debuffable.lua#L152

## 典型接入

### 吃胡萝卜施加或刷新加速效果

已有目标模组入口；先取得 [[reference/components/debuff#三十秒加速效果并成对撤销]] 的完整 `scripts/prefabs/my_mod_speedbuff.lua`，以下追加到 `modmain.lua`。PrefabFiles 合并已有列表，不覆盖；各端使用相同模组版本，运行端声明按 [[reference/api/modinfo/fields#运行端与用户配置]] 确认。工厂无需新增美术资源。[mods.lua:654](../../../raw/current/scripts/mods.lua#L654

```lua
PrefabFiles = PrefabFiles or {}
GLOBAL.table.insert(PrefabFiles, "my_mod_speedbuff")

local BUFF_KEY = "my_mod_speed"

AddPrefabPostInit("carrot", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local edible = inst.components.edible
    if edible == nil then return end
    local old_oneaten = edible.oneaten
    edible:SetOnEatenFn(function(food, eater)
        if old_oneaten ~= nil then old_oneaten(food, eater) end
        if eater ~= nil and eater:IsValid() and eater:HasTag("player")
            and eater.components.locomotor ~= nil then
            eater:AddDebuff(BUFF_KEY, "my_mod_speedbuff")
        end
    end)
end)
```

只扩展原 carrot 成功食用后的效果，保留原回调／三维／腐败／堆叠消耗，不影响 carrot_cooked；[[reference/components/edible#常见陷阱与限制]] 解释单回调与额外删除风险。原 veggies 工厂按名称注册，oneaten 收 `(food, eater)`；默认实体施加入口拒绝死亡／幽灵，不使用 skip_test 绕门禁。[veggies.lua:1096](../../../raw/current/scripts/prefabs/veggies.lua#L1096 [edible.lua:149](../../../raw/current/scripts/components/edible.lua#L149 [edible.lua:183](../../../raw/current/scripts/components/edible.lua#L183 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593 [componentutil.lua:15](../../../raw/current/scripts/componentutil.lua#L15

同管理键刷新到三十秒、不叠倍数，由所链接效果的 Extended 保证。主动撤销用 `eater:RemoveDebuff(BUFF_KEY)`；停用本功能只撤销这个键，不全局禁用管理器。若业务要基于施加成功继续奖励，不能判断 AddDebuff 的 true；读 GetDebuff 并检查有效性／效果条件。同键换 prefab 要先移旧效果，否则只延展旧对象。[entityscript.lua:2149](../../../raw/current/scripts/entityscript.lua#L2149 [debuffable.lua:95](../../../raw/current/scripts/components/debuffable.lua#L95

## 常见陷阱与限制

- **把第四参数当 skip_test 传给组件方法**：组件第四参是 buffer，实体便捷方法第四参才是 skip_test；调用前分清接收者和返回类型。[entityscript.lua:2140](../../../raw/current/scripts/entityscript.lua#L2140 [debuffable.lua:95](../../../raw/current/scripts/components/debuffable.lua#L95
- **禁用即保证读档无效果**：Enable(false) 清空当前集合，但 OnLoad 能重新登记存档效果；永久免疫策略需核验保存恢复时序并在正确位置处理，不只在工厂禁用。[debuffable.lua:28](../../../raw/current/scripts/components/debuffable.lua#L28 [debuffable.lua:139](../../../raw/current/scripts/components/debuffable.lua#L139
- **直接删效果等于正常撤销**：onremove 分支移集合但不 Detached；清理来源的具体要求在 [[reference/components/debuff#常见陷阱与限制]] 维护。[debuffable.lua:77](../../../raw/current/scripts/components/debuffable.lua#L77
- **配置符号就会显示／跟随**：SetFollowSymbol 只传参数，效果必须实现定位／回调；本页隐藏速度效果不使用符号，也不提供客户端图标。[debuffable.lua:51](../../../raw/current/scripts/components/debuffable.lua#L51

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 8 处（player_common debuffable 构造／pristine 标签段 -12~-14、entityscript AddDebuff/HasDebuff 段 +3、mainfunctions SetPersistData +26、mods.lua PrefabFiles -39）；无结论修订，仅锚点迁移——debuffable.lua、debuff.lua、edible.lua、veggies.lua 未变；componentutil.lua 本版改动在 100 行之后，L15 引用未变。依据：全局枢纽 diff 审阅（R-11 不涉 PrefabFiles；R-08 不涉 AddDebuff 签名）＋变更清单命中 player_common.lua、entityscript.lua、mainfunctions.lua、componentutil.lua、mods.lua。
- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖两种 Add 签名／返回／门禁、同键与通知顺序、保存恢复／移除／复制、人物消费者和食用接入。未全量审校分片／重选、所有原版 Buff 或外部实体引用保存。
- 结构检查：元数据、证据行、链接／章节、大小与登记；发布 modmain 与所链接工厂分别经 Full Moon 3.0.0／Lua 5.1 语法和原字节往返检查，成对文件内容检视。
- 游戏运行：未运行；食用门禁、重复刷新／撤销、死亡／退出重连、剩余时长、多人预测与其他模组回调协作待实测。

相关：[[reference/components/debuff#关键接口]]；[[reference/components/timer#生命周期]]；[[reference/components/locomotor#关键接口]]。
