---
title: "spawner"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "管理一只可进屋隐藏再释放的子实体、失去后的补员计时及内外两类保存，区分数量型巢穴。"
aliases: ["猪人重生等待减半", "保持同一只住户", "猪人房出门", "单体生成器", "spawner"]
tags: [component, creature, building, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/spawner.lua", "prefabs/pighouse.lua", "prefabs/rabbithole.lua", "worldsettingsutil.lua", "components/worldsettingstimer.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# spawner

## 快速判断

- 职责与适用：管理一个 child，进屋时挂为宿主子实体并 RemoveFromScene，出屋时恢复原实体；适合猪人房等住户。数量型出入巢见 [[reference/components/childspawner]]，独立周期产物见 [[reference/components/periodicspawner]]。[接收](../../../raw/current/scripts/components/spawner.lua#L295-L322) [释放](../../../raw/current/scripts/components/spawner.lua#L230-L293)
- 优先使用：宿主配置用 Configure，补员重排用 SpawnWithDelay；`IsOccupied()` 只判 child.parent==宿主，不表示有住户在外。减少未来猪人补员等待见下文案例。[配置](../../../raw/current/scripts/components/spawner.lua#L102-L127) [在屋判定](../../../raw/current/scripts/components/spawner.lua#L222-L224)
- 重要副作用：ReleaseChild 会先取消计时，child 不存在时才新建，否则释放同一住户；GoHome 隐藏实体、停移动、灭火、移除 homeseeker，不是 Remove 生物。[新建／释放](../../../raw/current/scripts/components/spawner.lua#L230-L253) [接收](../../../raw/current/scripts/components/spawner.lua#L295-L322)
- 修改边界：主模拟宿主仍负责昼夜、排队、烧毁与出门回调；猪人房接外部世界计时，不能只改 nextspawntime 当作所有模式都生效。[宿主安装](../../../raw/current/scripts/prefabs/pighouse.lua#L403-L420) [计时桥接](../../../raw/current/scripts/worldsettingsutil.lua#L158-L208)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `Configure(childname, delay, startdelay)` | prefab／秒／首轮秒数 | 保存后续失去住户的 delay，并立即排 startdelay 或 0；不是纯配置，也不是每次进屋就等 delay。原猪人房随后 CancelSpawning，由宿主初始化补住户。[方法](../../../raw/current/scripts/components/spawner.lua#L102-L120) [宿主](../../../raw/current/scripts/prefabs/pighouse.lua#L414-L420) [初始化](../../../raw/current/scripts/prefabs/pighouse.lua#L298-L310) |
| `SpawnWithDelay(delay)`／`CancelSpawning()`／`IsSpawnPending()` | 秒／无参数／boolean | 内置模式替换任务和 nextspawntime；外部模式转交 starttimerfn／stoptimerfn／timertestfn。pending 表示待执行计时存在，外部禁用时也可有记录，不等于住户已在屋或已能出门。[实现](../../../raw/current/scripts/components/spawner.lua#L109-L150) [外部判定](../../../raw/current/scripts/components/worldsettingstimer.lua#L49-L51) |
| `TakeOwnership(child)` | 已存在实体 | 替换旧 child 的监听及 knownlocations.home，给新 child 设置 home 坐标、四类失去事件监听、homeseeker；不自动收进屋、不杀旧住户，也不是安全跨房转移协议。[实现](../../../raw/current/scripts/components/spawner.lua#L186-L209) |
| `GoHome(child)` | 成功 true；否则事件后无显式返回 | 只接受 self.child 且尚未在屋；AddChild、坐标归零、RemoveFromScene、停止 locomotor、灭火、移除 homeseeker，最后 onoccupied(home, child)。失败向 child 发 gohomefailed。[实现](../../../raw/current/scripts/components/spawner.lua#L295-L322) |
| `ReleaseChild()` | 实际释放 true；其他可 nil | 先取消计时。无 child 才 SpawnPrefab、接管／收进屋并调 onspawnedfn；已在屋则解除父子、ReturnToScene、找落点／定位并调 onvacate。落点搜索失败仍有几何偏移兜底，不保证目标地形安全。[实现](../../../raw/current/scripts/components/spawner.lua#L230-L293) |
| `SetQueueSpawning(queued, retryperiod)` | boolean／重试秒数 | ReleaseChild 遇 queue_spawn 且 retryperiod 非 nil 才延后重排，不真正出门；只传 true 不足以保证拦截。兔洞用此协调昼夜／春季。[组件](../../../raw/current/scripts/components/spawner.lua#L130-L138) [释放分支](../../../raw/current/scripts/components/spawner.lua#L246-L253) [兔洞](../../../raw/current/scripts/prefabs/rabbithole.lua#L31-L64) |
| `SetOnlySpawnOffscreen(offscreen)` | boolean | 启用 entitysleep 监听；定时到期可等离屏再释放。直接 ReleaseChild 内不检查这个开关，宿主可主动释放。[设置](../../../raw/current/scripts/components/spawner.lua#L90-L100) [计时／离屏](../../../raw/current/scripts/components/spawner.lua#L7-L19) |
| `SetOnSpawnedFn/SetOnKilledFn/SetOnOccupiedFn/SetOnVacateFn(fn)` | fn(home, child) | 分别是新建子体、失去事件、进屋、出屋；onspawned 不是每次出门。四类失去事件共用 OnChildKilled，不能把 onkilled 当作仅死亡或全生命周期恰一次通知。[设置](../../../raw/current/scripts/components/spawner.lua#L69-L83) [监听](../../../raw/current/scripts/components/spawner.lua#L197-L209) [失去处理](../../../raw/current/scripts/components/spawner.lua#L324-L332) |

### 真实实体与保存边界

进屋隐藏会停止 Brain／SG、关闭物理及部分显示，出屋恢复；不能据此承诺全部组件任务、数值都冻结。猪人房的 onvacate 还主动重置变身触发并恢复满血，所以“同一实体”不等于宿主绝不改其属性。[隐藏](../../../raw/current/scripts/entityscript.lua#L348-L375) [恢复](../../../raw/current/scripts/entityscript.lua#L378-L412) [猪人房回调](../../../raw/current/scripts/prefabs/pighouse.lua#L146-L173)

OnSave 对在屋子体写 GetSaveRecord；开启 savenonpersistingchildasoccupied 时，非持久子体也可走记录分支。其余在外且活着的子体用 GUID 引用；无住户时内置计时记剩余 startdelay，外部模式另记 finished。OnLoad 先取消默认计时，记录分支用 SpawnSaveRecord 生成并收回；在外引用于 LoadPostPass 重连接管。跨重载按自身保存契约恢复，不承诺任意临时字段或同 GUID。[保存／加载](../../../raw/current/scripts/components/spawner.lua#L152-L184) [重连](../../../raw/current/scripts/components/spawner.lua#L212-L224)

组件移除会去离屏监听、取消内置任务，对在外住户清四类监听；并不自动释放在屋住户。原猪人房锤毁／点燃先显式 ReleaseChild，复制自定义房屋必须核验宿主销毁路径。[清理](../../../raw/current/scripts/components/spawner.lua#L45-L60) [锤毁](../../../raw/current/scripts/prefabs/pighouse.lua#L175-L190) [点燃](../../../raw/current/scripts/prefabs/pighouse.lua#L266-L270)

## 典型接入

### 普通猪人房未来失去住户后的补员等待减半

放目标模组 `modmain.lua`，无需资源。只改 pighouse 的 spawner.delay；OnChildKilled 在当前住户死亡／捕获／detachchild／移除后用这个值排新任务，不再次 Configure、不释放现有住户，也不重排当前倒计时。[宿主配置](../../../raw/current/scripts/prefabs/pighouse.lua#L414-L420) [失去后排程](../../../raw/current/scripts/components/spawner.lua#L324-L332) [Hook](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("pighouse", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local spawner = inst.components.spawner
    if spawner ~= nil then
        spawner.delay = spawner.delay * 0.5
    end
end)
```

`0.5` 是后续补员等待倍率，单位仍为秒。外部桥接把本次 delay 传入 worldsettingstimer，StartTimer 使用传入时长，世界 enabled 仍控制是否运行；本例不重新 AddTimer 或改其 maxtime／保存比例。[转交](../../../raw/current/scripts/worldsettingsutil.lua#L158-L168) [注册](../../../raw/current/scripts/worldsettingsutil.lua#L193-L208) [实际计时](../../../raw/current/scripts/components/worldsettingstimer.lua#L62-L99)

已有存档的剩余计时按原恢复链延续，不自动减半；初始住户、昼夜／光照出门、回家及毁屋逻辑仍由原宿主负责。原等待缩短不保证一到时就能在所有环境下看见猪人。停用并重新初始化恢复后续默认 delay，已经写入存档的剩余时间不回滚。[初始化／出门](../../../raw/current/scripts/prefabs/pighouse.lua#L284-L310) [白天门任务](../../../raw/current/scripts/prefabs/pighouse.lua#L212-L239)

兔洞的关键差异是排队与春季模式，并非猪人房换个 childname 即等价；不把本例扩展为全部巢穴频率修改教程。[兔洞模式](../../../raw/current/scripts/prefabs/rabbithole.lua#L31-L94)

## 常见陷阱与限制

- **IsOccupied=false 表示没有住户，可以再造**：住户在外也返回 false；先看 child，避免复制同一个房屋的生物。[判定](../../../raw/current/scripts/components/spawner.lua#L222-L224) [释放](../../../raw/current/scripts/components/spawner.lua#L230-L253)
- **暂停计时等于禁止手动出门**：CancelSpawning 只取消任务；ReleaseChild 自身还会先取消计时并处理释放。queue_spawn 需要 retryperiod；离屏开关不约束直接 ReleaseChild。[实现](../../../raw/current/scripts/components/spawner.lua#L130-L150) [释放](../../../raw/current/scripts/components/spawner.lua#L230-L253)
- **在屋就是每次销毁再造，或可完全冻结任意住户状态**：存活期间保留实体并隐藏，重载通过保存记录恢复；宿主可重置数值，组件也不承诺暂停所有组件任务。[接收](../../../raw/current/scripts/components/spawner.lua#L295-L322) [保存](../../../raw/current/scripts/components/spawner.lua#L152-L184) [宿主重置](../../../raw/current/scripts/prefabs/pighouse.lua#L146-L173)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 0 处（唯一命中的 entityscript.lua 两处引用区间本版未变）；结论修订：无。
- 依据：全局枢纽 hunk 地图（R-08）＋变更清单命中 entityscript.lua。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖单体接管／进出屋、回调时序、排程／排队／离屏、内外保存与移除，以及猪人房／兔洞必要宿主和外部计时传参。worldsettingstimer 仅核验本例所需注册／计时／禁用分支，未全面审校存档比例与版本迁移。
- 结构检查：格式、来源、链接／锚点、大小及唯一登记通过；完整补员倍率示例通过已有 Full Moon Lua 5.1 syntax-only，不证明实际补员／出门效果。
- 游戏运行：未运行；失去住户、在屋／在外保存、世界禁用、夜间／春季、毁屋、离屏、排队、异常落点和其他模组改变 delay 待实测。
