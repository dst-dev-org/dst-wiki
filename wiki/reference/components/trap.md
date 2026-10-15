---
title: "trap"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "捕捉陷阱的就绪、SG 触发、战利品记录／重建、诱饵与饥饿计时；不同于伤害型地雷。"
aliases: ["普通捕捉陷阱搜索范围改为二", "兔子陷阱捕获流程", "陷阱收获", "trap"]
tags: [component, callback, lifecycle, loot, state-machine, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/trap.lua", "prefabs/trap.lua", "prefabs/birdtrap.lua", "stategraphs/SGtrap.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# trap

## 快速判断

- 职责与适用：捕捉后记录产物名／有限 lootdata，移除原目标；收获重新 Spawn 实体并推进腐败时间，不是把完整活体放进隐藏容器。伤害型触发见 [[reference/components/mine]]。[捕获](../../../raw/current/scripts/components/trap.lua#L270-L304) [收获](../../../raw/current/scripts/components/trap.lua#L350-L376)
- 优先使用：Set 就绪，DoTriggerOn 通知 SG，再由 SG 调 DoSpring；扩展捕获效果用 onspring，收获用 onharvest，诱饵协作见 [[reference/components/bait]]。[触发](../../../raw/current/scripts/components/trap.lua#L189-L199) [SG](../../../raw/current/scripts/stategraphs/SGtrap.lua#L18) [回调设置](../../../raw/current/scripts/components/trap.lua#L71-L76)
- 重要副作用：Set／Reset 会清战利品、目标、诱饵关联和 foodspoil 计时，不适合用来“刷新有货陷阱”；Harvest 先发事件／调宿主，可能先耗尽并删除陷阱，再发放缓存位置的战利品。[重置](../../../raw/current/scripts/components/trap.lua#L126-L146) [收获警告](../../../raw/current/scripts/components/trap.lua#L350-L376)
- 修改边界：服务端；候选标签、目标可捕捉字段、SG 和资源需协作。仅 AddComponent 不构成完整捕捉物品；CHECKTRAP 动作返回 true 不证明获得战利品。[动作](../../../raw/current/scripts/actions.lua#L1686-L1690) [原宿主](../../../raw/current/scripts/prefabs/trap.lua#L66-L87)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `range`／`targettag`／`checkperiod` | 默认 1.5／smallcreature／0.75 秒；原普通 trap 改 targettag=canbetrapped，birdtrap 改 bird。搜索排 INLIMBO／untrappable、检查非死亡，命中标签仍可能只是安全拆陷阱。[默认](../../../raw/current/scripts/components/trap.lua#L33) [搜索](../../../raw/current/scripts/components/trap.lua#L180-L208) [宿主](../../../raw/current/scripts/prefabs/trap.lua#L77-L79) [鸟类](../../../raw/current/scripts/prefabs/birdtrap.lua#L140-L143) |
| `Set()`／`Reset(sprung?)` | Set 先 Reset，再 isset=true、排周期；Reset 停周期、清产物／魂／诱饵／目标／饥饿，并按参数设 issprung。Disarm 是已废弃的 Reset 别名。[方法](../../../raw/current/scripts/components/trap.lua#L126-L162) |
| `DoTriggerOn(target)` | 看 inventoryitem.trappable 或 lootdropper.trappable 决定保存 target，停更新并发 springtrap；可捕捉目标收到 trapped，否则收到 safelydisarmedtrap。不在此移除目标或产生物品。[方法](../../../raw/current/scripts/components/trap.lua#L184-L199) |
| `DoSpring()`／`onspring` | 有效目标先 fn(trap,target,bait)，再按可捕捉类型保存 Prefab／lootdata 或 GenerateLoot，启动饥饿；有 lootprefabs 才发 ontrapped 并 Remove 目标，最后处理诱饵／触发状态。[方法](../../../raw/current/scripts/components/trap.lua#L270) |
| `SetOnHarvestFn(fn)`／`Harvest(doer?)` | fn(trap) 没有 doer 参数；doer 在 harvesttrap 事件数据中。产物重建可调用 restoredatafromtrap，发给人物库存或放在缓存位置；存活且有剩余使用次数才重新 Set。[实现](../../../raw/current/scripts/components/trap.lua#L350-L395) |
| `StartStarvation()`／`OnTrappedStarve()` | 用目标剩余腐败秒数或两天，timer 名 foodspoil；到期走 starvedlootprefabs／魂事件、Reset(true) 并切 SG empty。不是把活体原地饥饿模拟。[计时](../../../raw/current/scripts/components/trap.lua#L245-L260) [到期](../../../raw/current/scripts/components/trap.lua#L213) |
| `SetBait(item)`／`RemoveBait()` | 关联 bait 组件、位置和显示层序；移除关联不销毁物品。设置本身不检查 isset／食谱／动作授权。[接口](../../../raw/current/scripts/components/trap.lua#L400-L424) |
| `HasLoot()`／`IsSprung()`／`IsBaited()` | HasLoot 仅判 lootprefabs 非 nil，空表也算；IsSprung 仅状态；IsBaited 还要求已设置、未触发、bait 非 nil。[查询](../../../raw/current/scripts/components/trap.lua#L114-L124) [状态](../../../raw/current/scripts/components/trap.lua#L346) |
| `OnSave`／`OnLoad`／`LoadPostPass` | 保存状态、产物／lootdata、魂／饥饿产物与诱饵 GUID；加载重启检测或向 SG 发 loading 的 springtrap，再 postpass 接诱饵。不是保存被移除目标全体状态。[存读](../../../raw/current/scripts/components/trap.lua#L439) |

### 宿主触发、耐久与离屏协作

SGtrap 醒着先播 trap_pre，再 DoSpring；睡着可直接结算。原普通陷阱 onharvested 扣一次 finiteuses，耗尽会 Remove，所以追加收获回调不能继续假定 inst 有效。组件自带 timer／库存监听，移除组件不删除 timer。鸟笼陷阱还在宿主离屏路径生成鸟后 DoSpring，不能只改本组件字段就宣称统一所有捕获概率。[SG](../../../raw/current/scripts/stategraphs/SGtrap.lua#L18) [原消耗](../../../raw/current/scripts/prefabs/trap.lua#L15-L33) [组件清理](../../../raw/current/scripts/components/trap.lua#L61) [鸟类离屏](../../../raw/current/scripts/prefabs/birdtrap.lua#L31-L53)

## 典型接入

### 普通捕捉陷阱搜索范围改为二个世界单位

目标模组 `modmain.lua`；只改原 Prefab trap，保留其 targettag、SG、诱饵、次数和饥饿协议，不改 birdtrap。[安装](../../../raw/current/scripts/prefabs/trap.lua#L66-L87) [搜索读取](../../../raw/current/scripts/components/trap.lua#L204-L208)

```lua
AddPrefabPostInit("trap", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.trap ~= nil then
        inst.components.trap.range = 2
    end
end)
```

下次搜索读取新值，不额外 Set／Reset，以免清掉已有产物。范围不改变目标 trappable 字段、收获次数或捕获产物；默认睡眠检测周期变十倍，另有宿主离屏行为，不能据范围推断每秒收益。[周期](../../../raw/current/scripts/components/trap.lua#L159-L176)

参数加载工厂时重设，停用并重载恢复默认；已经捕捉／消费的目标不回滚。有限保存数据不等于完整保留目标血量、Brain 或自定义组件，复杂物种需核验 settrapdata／restoredatafromtrap。相邻机制见 [[reference/components/timer]]、[[reference/components/lootdropper]]、[[reference/components/finiteuses]]。

## 常见陷阱与限制

- **Reset 是无损撤防**：它会清 lootprefabs／lootdata 和饥饿记录；已有捕获应先按真实业务收获或转存，不清字段冒充放生活体。[方法](../../../raw/current/scripts/components/trap.lua#L126-L146)
- **有 targettag 就必捕获**：inventoryitem／lootdropper 的 trappable 才决定保留目标；否则走安全拆陷阱事件。onspring 也只是回调，不是验证授权。[资格](../../../raw/current/scripts/components/trap.lua#L184-L199)
- **onharvest 后还可自由使用 inst**：原回调可能扣最后一次并删实体；源码明确 WARNING。追加逻辑缓存必要数据并查有效性，勿重复发原战利品。[收获](../../../raw/current/scripts/components/trap.lua#L350-L376)
- **onharvest 自带玩家／CHECKTRAP true 就有产物**：玩家只在事件中，Harvest 也可能未处于 sprung 而什么都不做。[事件](../../../raw/current/scripts/components/trap.lua#L350-L358) [动作](../../../raw/current/scripts/actions.lua#L1686-L1690)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对本页 sources 变更文件 actions.lua、componentactions.lua（更新报告 R-01/R-04）：本次 hunk 集中在施法门控／拾取放宽／新增 CORRUPTNIGHTMARE，未触及 CHECKTRAP 候选与执行链，结论无修订；锚点重定位 2 处（CHECKTRAP.fn 随前部插入 +3 行）；未覆盖：运行验证。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖完整 trap、原普通／鸟类宿主、SGtrap、收获／诱饵动作、触发资格／产物／魂／饥饿／存读。未全面审校鸟类生成器、所有生物 SG／trappable 设置、魂分发、物种扩展存储或多人并发。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整范围例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；安全拆除、最后一次耐久、诱饵被吃／偷、离屏捕获、饥饿／腐败和保存重建待实测。
