---
title: "harvestable"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "管理蜂箱／蘑菇农场式产物累积、分次增长与整批收获，区分计时、宿主回调和采收门禁。"
aliases: ["蘑菇农场生长间隔减半", "蜂箱产蜜", "产物累积收获", "harvestable"]
tags: [component, lifecycle, callback, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/harvestable.lua", "prefabs/mushroom_farm.lua", "prefabs/beebox.lua", "componentactions.lua", "actions.lua", "standardcomponents.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# harvestable

## 快速判断

- 职责与适用：保留宿主、逐次累积 `produce`，一次收走当前批次；用于蜂箱、蘑菇农场。采后等一轮再长满优先看 [[reference/components/pickable]]；阶段外观成长看 [[reference/components/growable]]。[增长](../../../raw/current/scripts/components/harvestable.lua#L128-L143) [宿主](../../../raw/current/scripts/prefabs/beebox.lua#L280-L282)
- 优先使用：宿主初始化用 `SetUp` 或分开配置；普通收获走 `Harvest(picker)`。`SetProduct` 会清空库存，不能当无副作用的产物 setter。[配置](../../../raw/current/scripts/components/harvestable.lua#L39-L44) [重置](../../../raw/current/scripts/components/harvestable.lua#L69-L72)
- 重要副作用：Harvest 先清 produce、调用宿主回调，再逐个生成／发放原数量的产物并尝试重启成长；回调不应再重复发同批产物。[收获顺序](../../../raw/current/scripts/components/harvestable.lua#L176-L214)
- 修改边界：本页服务端实现；可收获标签只表达 enabled 且 produce>0，额外限制在 Harvest 才检查。蜂箱还有授粉归巢直接 Grow 的路径，单改 growtime 不等于控制总产量。[门禁](../../../raw/current/scripts/components/harvestable.lua#L79-L84) [归巢](../../../raw/current/scripts/prefabs/beebox.lua#L144-L150)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetUp(product,max,time,onharvest,ongrow)` | 产物名、上限、秒数或 nil、两个回调 | 重置产物／数量、配置回调后 StartGrowing；nil 时间可由宿主事件驱动。[实现](../../../raw/current/scripts/components/harvestable.lua#L39-L44) |
| `SetProduct(product,max?)`／`SetGrowTime(time)` | max 默认 1；time 可为 nil | 前者把 produce 归零；后者只改基准时间，不取消／重排当前任务，也不清 pausetime。[实现](../../../raw/current/scripts/components/harvestable.lua#L69-L76) |
| `Grow()` | 有增长 true，已满 false | 未满加 1、调用 ongrowfn(inst,produce)，再决定续长或停止；不检查 enabled。[实现](../../../raw/current/scripts/components/harvestable.lua#L128-L143) |
| `StartGrowing(time?)` | 无显式返回 | 先停原任务，取显式 time→pausetime→growtime，清 pausetime，有时长才调度；不检查 enabled 或是否已满。[实现](../../../raw/current/scripts/components/harvestable.lua#L146-L152) |
| `PauseGrowing()`／`StopGrowing()` | 无显式返回 | Pause 留正剩余秒数供恢复；Stop 取消任务但不清已有 pausetime，因此 Stop 后 Start 未必从完整时长开始。[实现](../../../raw/current/scripts/components/harvestable.lua#L156-L172) |
| `Disable()`／`Enable()` | 暂停并禁收／启动并启收 | enabled 通过属性回调更新标签；直接 Grow／StartGrowing 仍能绕过禁用。[实现](../../../raw/current/scripts/components/harvestable.lua#L87-L94) [标签](../../../raw/current/scripts/components/harvestable.lua#L1-L14) |
| `CanBeHarvested()`／`SetCanHarvestFn(fn)` | 基本可收获布尔值／fn(inst,picker)→允许,原因 | 查询不执行 fn；Harvest 额外检查，拒绝返回 false,reason；基本不可收获时无显式返回。[实现](../../../raw/current/scripts/components/harvestable.lua#L79-L84) [执行](../../../raw/current/scripts/components/harvestable.lua#L176-L186) |
| `SetOnGrowFn(fn)`／`SetOnHarvestFn(fn)` | fn(inst,produce)／fn(inst,picker,原数量) | 收获回调时组件 produce 已为 0；之后使用当时的 product 生成原数量，不能把回调理解为只通知完成。[setter](../../../raw/current/scripts/components/harvestable.lua#L61-L66) [调用](../../../raw/current/scripts/components/harvestable.lua#L185-L206) |
| `SetDoMagicGrowthFn`／`DoMagicGrowth(doer)` | fn(inst,doer) | 只委托宿主；不转发回调返回值，具体次数／禁用由宿主决定。[实现](../../../raw/current/scripts/components/harvestable.lua#L47-L57) [农场连续催长](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L315-L322) |

HARVEST 候选按 harvestable 标签加入动作，实际执行再调用 Harvest。收获有 inventory 的 picker 会收到 harvestsomething 事件并 GiveItem；其他情况向地面 LaunchAt。无有效 product 或生成失败也不回滚已清掉的 produce。[候选](../../../raw/current/scripts/componentactions.lua#L404-L406) [动作](../../../raw/current/scripts/actions.lua#L2655-L2656) [发放](../../../raw/current/scripts/components/harvestable.lua#L194-L214)

### 保存与卸载

组件保存 produce、正剩余时间和 pausetime，不保存 product／maxproduce／growtime／enabled。宿主必须恢复配置；蘑菇农场另存产品／上限／growtime，并直接赋值恢复。OnRemoveFromEntity 只清标签，不取消成长任务；动态移除前先 StopGrowing，原版烧毁清理也这样处理。[组件存读](../../../raw/current/scripts/components/harvestable.lua#L97-L113) [农场存读](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L283-L310) [钩子](../../../raw/current/scripts/components/harvestable.lua#L35-L36) [烧毁处理](../../../raw/current/scripts/standardcomponents.lua#L93-L95)

## 典型接入

### 蘑菇农场后续种植的每次生长间隔减半

放目标模组 `modmain.lua`，无新增资源／配方。包装现有组件 SetGrowTime，保留原场景传入的 nil 停长信号、产量、外观与收获回调；不主动启动空农场，不影响蜂箱。农场种植先配置产品与每次间隔，再立即 Grow 一次。[种植](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L53-L74) [空／耗尽停长](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L90-L101) [注册入口](../../../raw/current/scripts/modutil.lua#L593-L598)

```lua
AddPrefabPostInit("mushroom_farm", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local harvestable = inst.components.harvestable
    if harvestable == nil then return end
    local original = harvestable.SetGrowTime
    harvestable.SetGrowTime = function(self, time)
        return original(self, time ~= nil and time * 0.5 or nil)
    end
end)
```

`0.5` 是秒数乘数；对之后经过 setter 的新种植生效，不追溯重排现有倒计时。读档直接恢复旧 growtime／剩余时间，不能据此宣称旧批次加速；新种植技能倍率会继续叠乘。积雪清产物、寿命／维修与魔法连续 Grow 仍按原版，不保证每种情形总等待恰好减半。停用后重建实例不再包装，已保存的时长不自动回滚。[存读](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L295-L310) [积雪](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L115-L125) [魔法](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L315-L322)

### 原版模式选择

- **事件累积＋条件计时：蜂箱。** 初始 growtime=nil，授粉蜂回家直接 Grow；睡眠期间另按季节、蜂群及花朵条件设置计时，收获回调又清时间／暂停余量。保留 [[reference/components/pollinator]] 的归巢分工，不用固定定时器替换全部逻辑。[初始化](../../../raw/current/scripts/prefabs/beebox.lua#L280-L282) [条件](../../../raw/current/scripts/prefabs/beebox.lua#L31-L36) [收获](../../../raw/current/scripts/prefabs/beebox.lua#L122-L139) [睡眠计时](../../../raw/current/scripts/prefabs/beebox.lua#L160-L172)
- **投料启动＋分次增长：蘑菇农场。** 产量／间隔来自投料与人物技能，回调负责外观、可交易状态及剩余收获次数。需要此模式时复用上例，不改成 pickable 的单轮恢复。[投料](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L46-L74) [回调](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L138-L153)

## 常见陷阱与限制

- **改 product／SetProduct 想保留现有库存**：setter 清 produce；换产品是否允许继承库存须由需求另行定义，不能将原批次无声改成新产物。见关键接口。
- **禁用就不会增长**：Disable 只暂停当前任务并禁收；自定义回调仍可直接 Grow 或 StartGrowing。宿主需一并管住这些入口，魔法农场便会在禁收期间继续定时 Grow。
- **CanBeHarvested=true 就保证动作成功**：额外 can_harvest_fn 在 Harvest 才运行；回调须接受 picker=nil 的直接调用，失败原因由调用者处理。
- **收获回调再发一批物品或移除宿主**：默认发放还在后面，且随后 StartGrowing；会有重复产出／失效宿主风险。回调保留宿主，扩展额外奖励时明确与默认产物分工，勿把生成失败当已回滚。见收获顺序。

## 验证记录

- 版本沿用：2026-10-25，`steam-buildid:26148036`；原结论复用、锚点迁移，见[本轮依据](../../updates/build-26148036/index.md#wiki-沿用记录)。
- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖精选配置／增长／暂停／收获、动作候选、保存／卸载、蜂箱两类增长与蘑菇农场示例。未审校全部 harvestable 宿主、所有额外失败原因、客户端显示或存档迁移。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整 modmain 例通过已有 Full Moon Lua 5.1 syntax-only，不证明增长或收获的游戏效果。
- 游戏运行：未运行；新投料／旧存档、积雪清空、催长连发、蜂箱睡醒和其他模组包装叠加待实测。
