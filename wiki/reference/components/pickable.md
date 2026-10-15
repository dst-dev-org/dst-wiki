---
title: "pickable"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "配置采收产物与数量、采后再生和枯萎边界，保留原版采收回调及状态。"
aliases: ["采收产量", "草再生时间", "植物采摘", "采收回调", "pickable"]
tags: [component, mechanism, callback, loot, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/pickable.lua", "prefabs/grass.lua", "standardcomponents.lua", "simutil.lua", "actions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# pickable

## 快速判断

- 职责与适用：采收目标的产物、可采状态和再生计时；多阶段外观／成长用 [[reference/components/growable]]，砍挖工作量用 [[reference/components/workable]]。[pickable.lua:33](../../../raw/current/scripts/components/pickable.lua#L33
- 优先使用：新目标初始化用 `SetUp`；修改既有草产量／未来再生用 `numtoharvest`／`baseregentime`，不重置状态。实际采收走 `Pick(picker)`。[pickable.lua:222](../../../raw/current/scripts/components/pickable.lua#L222 [pickable.lua:535](../../../raw/current/scripts/components/pickable.lua#L535
- 重要副作用：先产物生成／交付，再 onpicked，之后变不可采、排再生、向目标发 picked；回调里 canbepicked 仍可能为 true。[pickable.lua:551](../../../raw/current/scripts/components/pickable.lua#L551
- 修改边界：服务端；再生还受暂停、贫瘠、枯萎、季节和自定义计时影响，基础秒数不是所有条件下的实际等待时间。[pickable.lua:559](../../../raw/current/scripts/components/pickable.lua#L559

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetUp(product, regen, number?)` | Prefab 名、基础秒数或 nil、默认 1 | 设置产物／数量并强制可采；不用于读档植物的单纯调参。[pickable.lua:222](../../../raw/current/scripts/components/pickable.lua#L222 |
| `numtoharvest`／`baseregentime`／`getregentimefn` | 数量／秒数；回调 `(inst) -> 秒数` | 下一次 Pick／MakeEmpty 排再生时优先用回调，再应用 SpringGrowthMod；改字段不重排已有任务。[pickable.lua:438](../../../raw/current/scripts/components/pickable.lua#L438 [pickable.lua:559](../../../raw/current/scripts/components/pickable.lua#L559 |
| `ChangeProduct(name)` | Prefab 名 | 只改 product；不改可采、数量或计时。[pickable.lua:255](../../../raw/current/scripts/components/pickable.lua#L255 |
| `CanBePicked()`／`caninteractwith`／`SetStuck(bool)` | 可采标志／交互门禁／卡住标志 | CanBePicked 只返 canbepicked；Pick 还查后两项；canbepicked 与交互共同维护 pickable 标签。[pickable.lua:1](../../../raw/current/scripts/components/pickable.lua#L1 [pickable.lua:382](../../../raw/current/scripts/components/pickable.lua#L382 [pickable.lua:595](../../../raw/current/scripts/components/pickable.lua#L595 |
| `Pick(picker)` | 成功 `true, 实体数组或 nil`；失败无显式返回 | 普通产物路径 nil picker 不生成物品；无 inventory 的非 nil picker 掉地。droppicked 分支可实际掉物但返回 nil。[pickable.lua:450](../../../raw/current/scripts/components/pickable.lua#L450 [pickable.lua:581](../../../raw/current/scripts/components/pickable.lua#L581 |
| `onpickedfn`／`onregenfn` | `(inst, picker, loot)`／`(inst)` | 单槽；loot 按生成路径可能实体、数组或 nil，不能假设已在背包或一定有效。[pickable.lua:386](../../../raw/current/scripts/components/pickable.lua#L386 [pickable.lua:450](../../../raw/current/scripts/components/pickable.lua#L450 |
| `FinishGrowing()`／`Regen()` | 前者 boolean；后者直接恢复可采 | FinishGrowing 要存在计时且未可采／未枯萎；Regen 本身不恢复 cycles 或解除 paused。[pickable.lua:129](../../../raw/current/scripts/components/pickable.lua#L129 [pickable.lua:386](../../../raw/current/scripts/components/pickable.lua#L386 |
| `Pause()`／`Resume()` | 无命名原因参数 | 内部计时保存剩余并取消任务，恢复重新应用春季倍率；只有一个 paused 标志，不支持多来源独立解除。[pickable.lua:147](../../../raw/current/scripts/components/pickable.lua#L147 [pickable.lua:179](../../../raw/current/scripts/components/pickable.lua#L179 |
| `MakeEmpty()`／`MakeBarren()`／`IsBarren()` | 清空／贫瘠／cycles_left==0 | MakeEmpty 不生成产品或执行完整采收链；MakeBarren 置 cycles=0 并停计时，不等于单纯采空。[pickable.lua:378](../../../raw/current/scripts/components/pickable.lua#L378 [pickable.lua:398](../../../raw/current/scripts/components/pickable.lua#L398 [pickable.lua:426](../../../raw/current/scripts/components/pickable.lua#L426 |

## 典型接入

### 草每次采收两份与六十秒基础再生

目标模组 `modmain.lua`；沿用已注册 grass 的 cutgrass、资源、动画／草蜥蜴回调、移植耗肥与冬季暂停，不新增 Prefab。初始化与注册见 [grass.lua:245](../../../raw/current/scripts/prefabs/grass.lua#L245)、[grass.lua:321](../../../raw/current/scripts/prefabs/grass.lua#L321)，接入见 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)。

```lua
AddPrefabPostInit("grass", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local pickable = inst.components.pickable
    if pickable ~= nil then
        pickable.numtoharvest = 2
        pickable.baseregentime = 60
    end
end)
```

数量用于普通堆叠产物；秒数在以后排再生时使用，不清空现有草或缩短已排／读档剩余。冬季仍 Pause／Resume，春季仍用原版倍率；贫瘠或枯萎仍可阻止正常采后再生。[pickable.lua:500](../../../raw/current/scripts/components/pickable.lua#L500 [pickable.lua:559](../../../raw/current/scripts/components/pickable.lua#L559 [standardcomponents.lua:822](../../../raw/current/scripts/standardcomponents.lua#L822 [simutil.lua:563](../../../raw/current/scripts/simutil.lua#L563

调参不替换 onpickedfn；草仍可能沿原版回调变草蜥蜴。product／数量／基础时长不由组件存档，初始化每次应用；已采状态、剩余时间与移植循环另行恢复，停用后恢复原版初始化，现有剩余时间不回滚。[grass.lua:118](../../../raw/current/scripts/prefabs/grass.lua#L118 [pickable.lua:295](../../../raw/current/scripts/components/pickable.lua#L295 [pickable.lua:327](../../../raw/current/scripts/components/pickable.lua#L327

## 常见陷阱与限制

- **直接改 regentime 就缩短下次再生**：Pick 会从 getregentimefn 或 baseregentime 重算；改正确来源，已有任务另外处理，不能为调速重调用 SetUp。[pickable.lua:560](../../../raw/current/scripts/components/pickable.lua#L560
- **非堆叠产品也按 numtoharvest 生成多件**：普通路径只 SpawnPrefab 一次，数量只写 stackable；确需多实体须设计掉落路径，不能重复调用 Pick。[pickable.lua:460](../../../raw/current/scripts/components/pickable.lua#L460 [pickable.lua:493](../../../raw/current/scripts/components/pickable.lua#L493
- **onpicked 中再 Pick，或再次发产品**：回调在变不可采之前；会重入或重复产物。包装保留原函数，额外效果不要再调用采收链。[pickable.lua:551](../../../raw/current/scripts/components/pickable.lua#L551
- **动作成功就代表拿到产品**：ACTIONS.PICK 调组件后直接返回 true；组件有更多门禁，且产物可能掉地／生成失败。检查实际目标状态和发放路径。[actions.lua:1933](../../../raw/current/scripts/actions.lua#L1933
- **移组件自动清外部再生计时**：OnRemoveFromEntity 只取消内部 task 与标签，useexternaltimer 的外部任务需所属机制清理。[pickable.lua:83](../../../raw/current/scripts/components/pickable.lua#L83

## 验证记录

- 版本沿用：2026-10-25，`steam-buildid:26148036`；原结论复用、锚点迁移 2 处（actions.lua PICK、simutil.lua SpringGrowthMod）；依据：actions.lua／componentactions.lua 枢纽 diff 审阅＋变更清单命中 actions.lua、simutil.lua，PICK 动作与春季倍率代码同旧版，componentactions 的 SCENE.pickable 过滤仅 HasTag→HasAnyTag 等价改写且本页未引用。未运行游戏。
- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖采收门禁、产物／回调顺序、内部再生／暂停、保存恢复及草调参。外部计时器完整接入、农作物与全部枯萎／施肥消费者未覆盖。
- 结构检查：元数据、源码行、链接／章节、大小、登记通过；发布 Lua 与植物调参组合经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；产量、移植／枯萎、季节切换、存档剩余与其他模组覆盖待实测。

相关：[[reference/components/growable#关键接口]]；[[reference/components/lootdropper#关键接口]]；[[reference/components/stackable#关键接口]]。
