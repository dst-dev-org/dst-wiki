---
title: "childspawner"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "管理巢内储备、在外子体、补员与回巢；含洞穴／地表洞口蝙蝠的昼夜出巢接线。"
aliases: ["蜂箱多两只蜜蜂", "巢穴生成生物", "子体回巢", "洞穴黄昏刷新蝙蝠", "蝙蝠出巢", "洞口蝙蝠", "batcave", "cave_entrance", "childspawner"]
tags: [component, creature, building, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/childspawner.lua", "components/homeseeker.lua", "prefabs/beebox.lua", "prefabs/beehive.lua", "prefabs/batcave.lua", "prefabs/cave_entrance.lua", "worldsettingsutil.lua", "actions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# childspawner

## 快速判断

- 职责与适用：巢穴侧管理普通／应急两套巢内数量、在外实体及补员；巢内主要是数量，不是隐藏着的完整生物。蜂箱在主模拟安装。[计数](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L52-L99) [宿主](../../../raw/archive/build-26148036/scripts/prefabs/beebox.lua#L274-L298)
- 优先使用：增加容量见下文案例；生成用 `SpawnChild`，接收入巢用 `GoHome`；生物侧家园引用／行动请求见 [[reference/components/homeseeker]]，组件不负责 Brain 的回家时机。
- 重要副作用：`SetMaxChildren` 同时按在外数量重算巢内储备；`GoHome` 接收已登记子体后移除实体、加回数量，不能理解为保留其全部状态。[设容量](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L315-L321) [接收](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L668-L715)
- 修改边界：普通出巢／补员分开控制；蜂箱／野蜂巢还接外部计时、季节与宿主门禁，不能只改一个字段就宣称固定出蜂频率或全年工作。[计时桥接](../../../raw/archive/build-26148036/scripts/worldsettingsutil.lua#L87-L121) [野巢季节](../../../raw/archive/build-26148036/scripts/prefabs/beehive.lua#L99-L109)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `childname`／`childreninside`／`childrenoutside` | prefab 候选／数量／实体表 | 构造时 inside=1、maxchildren=0，需宿主配置。`NumChildren()` 为普通 inside+outside；应急池另计。[初值](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L52-L99) [总数](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L169-L183) |
| `SetMaxChildren(max)` | 无显式返回 | inside=max−有效在外数量，下限 0；不删除在外子体，不只是改上限。`AddChildrenInside(count)` 只限制 inside 不超过 max，不能用来安全接管任意外来生物。[容量](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L305-L329) [添加](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L795-L808) |
| `SetSpawnPeriod(period, variance)`／`SetRegenPeriod(period, variance)` | 秒；variance 默认 period×0.1 | 重设组件下一次间隔；`GetTimeToNextSpawn/Regen` 在 period±variance 内随机。外部计时模式见下文，不能据此认定已重设外部剩余时间。[随机](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L113-L118) [设置](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L153-L163) |
| `StartSpawning/StopSpawning()`／`StartRegen/StopRegen()` | 无参数 | 出巢和缺员补充独立；StartSpawning 将下一次出巢设为 0，StopSpawning 不召回在外生物，也不停止补员。[补员开关](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L134-L151) [出巢开关](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L262-L275) |
| `SpawnChild(target, prefab, radius)` | 实体或 nil | 要有储备、宿主有效／未死、醒着或允许离屏、canspawnfn 通过；target 无敌可拒绝。成功后才扣 inside、接管子体；找不到落点或生成失败不扣数。传 prefab 仍可能被稀有／外来种类选择覆盖。[生成](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L517-L576) [入账](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L615-L633) [门禁](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L717-L737) |
| `TakeOwnership(child)` | 已存在实体 | 记 knownlocations 的 home（若有），添加／设置 homeseeker，监听死亡／捕获／移除／detachchild 并登记在外；不扣 inside，也不从旧巢转移账目。普通 SpawnChild 已调用，不能再重复接管。[监听](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L38-L49) [接管](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L457-L481) |
| `SetSpawnedFn(fn)`／`SetOnTakeOwnershipFn(fn)` | fn(home, child) | onspawned 在扣数／接管之前；ontakeownership 在设家与监听之后、登记在外之前，读计数必须考虑时序。setter 替换原回调，扩展时保留原逻辑。[设置](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L281-L287) [生成回调](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L566-L573) [接管顺序](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L457-L475) |
| `GoHome(child)`／`SetGoHomeFn(fn)` | 成功 true；验证拒绝 false；不归属则 nil | 只接收本巢登记的普通／应急子体；gohomevalidatefn 收 home。先向巢发 childgoinghome、向子体发 goinghome，再回调、移除在外记录和实体、恢复 inside。[实现](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L668-L715) |
| `ReleaseAllChildren(target, prefab, radius)` | 已放出的实体数组；可提前 nil | 按普通／应急各自门禁连续尝试，失败 3 次中止；不是强制绕过离屏、死亡或库存条件。应急还需 canemergencyspawn、池容量与附近玩家配额。[应急配额](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L652-L665) [门禁与死亡处理](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L729-L755) [释放](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L758-L792) |

### 计时、保存与清理

内置更新任务间隔为 5～10 秒，`LongUpdate` 也只调用一次 OnUpdate，不是按离屏时间循环补齐所有周期。离屏不能出巢时可排 queued_spawn，唤醒后再尝试；手动 SpawnChild 不要求 spawning=true。[更新与补员](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L185-L224) [调度](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L230-L259) [唤醒](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L579-L596) [追赶](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L819-L821)

蜂箱和野巢通过 `WorldSettings_ChildSpawner_*` 启用 `useexternaltimer`，注册出巢／补员计时。Start/Stop 经回调恢复／暂停，StartSpawning 还设外部剩余为 0；SetSpawnPeriod/SetRegenPeriod 本身只写组件字段。修改频率须另核验外部计时器，不能用内置调度说明替代。[桥接](../../../raw/archive/build-26148036/scripts/worldsettingsutil.lua#L34-L121) [蜂箱](../../../raw/archive/build-26148036/scripts/prefabs/beebox.lua#L285-L298)

OnSave 保存 inside、在外 GUID 引用、运行开关和倒计时；max 仅在 save_max_children 开启时保存。OnLoad 恢复数据，LoadPostPass 根据已重建实体重新接管并启动／停止更新，不重新生成全部外出子体。`data.spawning or self.spawning` 等表达式不保证保存的 false 能覆盖构造默认 true。[保存](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L332-L377) [加载](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L407-L455) [引用重连](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L483-L503)

在外子体死亡／捕获／移除／detachchild 会去监听、减在外计数并尝试更新；普通池才调用 onchildkilledfn。组件移除只清监听和任务，不批量杀掉子体或清掉其 home；野巢死亡就会移除组件，回家接收方可能因此消失。[失去子体](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L739-L755) [清理](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L121-L132) [野巢死亡](../../../raw/archive/build-26148036/scripts/prefabs/beehive.lua#L77-L85)

## 典型接入

### 普通蜂箱容量增加 2，继续采用原季节配置

放目标模组 `modmain.lua`，复用原蜂箱，无新资源。只处理 `beebox`，保留原容量 setter 的计数／调度，每次原季节回调传入基础容量后加 2；不处理 `beebox_hermit`、野巢或应急池。原季节回调会重新设置容量，单次赋 maxchildren 会被覆盖。[蜂箱季节](../../../raw/archive/build-26148036/scripts/prefabs/beebox.lua#L223-L235) [安装与季节监听](../../../raw/archive/build-26148036/scripts/prefabs/beebox.lua#L274-L298) [Hook](../../../raw/archive/build-26148036/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("beebox", function(inst)
    if not GLOBAL.TheWorld.ismastersim or not TUNING.BEEBOX_ENABLED then
        return
    end
    local spawner = inst.components.childspawner
    if spawner == nil then return end

    local original = spawner.SetMaxChildren
    spawner.SetMaxChildren = function(self, base)
        return original(self, base + 2)
    end
    spawner:SetMaxChildren(spawner.maxchildren)
end)
```

`2` 为额外普通子体容量。初始化这次调用也会重算 inside；存档随后按原 OnLoad／LoadPostPass 恢复 inside／outside，不承诺读档立刻满员。季节切换仍有原 setter 的补满副作用；不要再把已加成的 maxchildren 作为 base 重复传入。保留世界禁用、冬季／光照出巢条件、外部补员／出巢计时和蜂蜜生产；蜂蜜流程见 [[reference/components/harvestable]]。停用并重载恢复原配置，但不能保证立刻删除超额在外生物或把已有存量回滚到旧值。

原版模式按用途区分：蜂箱监听 childgoinghome 参与产蜜；野蜂巢受击可指定 killerbee 释放并有应急池，死亡移除生成器。回巢不保留原实体的生命／任意模组字段，特殊外来品种另有显式 opt-in 计数，不能据此称完整实体容器。[蜂箱事件](../../../raw/archive/build-26148036/scripts/prefabs/beebox.lua#L280-L282) [野巢受击](../../../raw/archive/build-26148036/scripts/prefabs/beehive.lua#L88-L109) [应急配置](../../../raw/archive/build-26148036/scripts/prefabs/beehive.lua#L186-L198) [外来种类](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L686-L697)

### 洞穴黄昏刷新蝙蝠由谁控制

控制普通巢穴储备与出巢的是 `childspawner`，昼夜开关由具体宿主监听世界状态；不是 [[reference/components/batspawner|未完整导出的 batspawner]]。先区分洞内巢穴与地表洞口：

| 原版宿主 | 时段开关与接入证据 |
|---|---|
| 洞内 `batcave` | 监听 `iscaveday`：洞穴白天 StopSpawning，非白天 StartSpawning。服务端配置 bat、容量、出巢／补员周期及世界设置计时，初始化巢内储备为0。[阶段开关](../../../raw/archive/build-26148036/scripts/prefabs/batcave.lua#L68-L74)、[巢穴配置](../../../raw/archive/build-26148036/scripts/prefabs/batcave.lua#L105-L120)、[状态监听](../../../raw/archive/build-26148036/scripts/prefabs/batcave.lua#L128-L129) |
| 地表打开的 `cave_entrance` 洞口 | 监听地表 `isday`：白天补员、停止出巢并请求在外蝙蝠归巢；非白天停止补员、开启出巢。另有 `worldmigrator:IsActive() or IsFull()` 的出巢门禁。[阶段开关](../../../raw/archive/build-26148036/scripts/prefabs/cave_entrance.lua#L36-L54)、[迁移器门禁](../../../raw/archive/build-26148036/scripts/prefabs/cave_entrance.lua#L87-L89)、[服务端配置](../../../raw/archive/build-26148036/scripts/prefabs/cave_entrance.lua#L210-L220)、[地表状态监听](../../../raw/archive/build-26148036/scripts/prefabs/cave_entrance.lua#L235-L236) |

“黄昏刷新”是从白天变为非白天后开启出巢流程，不是无条件立即生成固定数量；仍受储备、世界开关、宿主门禁、离屏和位置条件约束。洞内巢满且玩家靠近还有直接 `SpawnChild(player)` 分支，不能把所有出蝙蝠都归因于阶段回调。[靠近释放](../../../raw/archive/build-26148036/scripts/prefabs/batcave.lua#L22-L37)、[组件出巢门禁](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L717-L737)

洞穴光源 `cavelight` 暂存并放回原有蝙蝠是另一条接线，见 [[reference/components/hideout#洞穴光源暂存夜间撤离的蝙蝠|hideout 原版案例]]；不能据“蝙蝠收放”把它和巢穴补员混为一谈。

## 常见陷阱与限制

- **把回家当重新 SpawnPrefab**：正确接收是去外出记录、Remove 子体、加储备；手动额外生成或再次 TakeOwnership 会造成重复实体／账目。家园引用与登记是两回事。[生成入账](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L615-L633) [接收](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L668-L715)
- **只给生物 SetHome 就往任意建筑回家**：childspawner 拒绝未登记者；通用 GOHOME 对没有 spawner／childspawner／hideout 的目标会发事件并移除生物，不能用作安全校验。[动作](../../../raw/archive/build-26148036/scripts/actions.lua#L2946-L2961)
- **停出巢等于巢不再补员／清空**：两个开关独立，StopSpawning 不影响在外实体；重复设容量可能立即补储备，不能作为纯 UI 配置操作。[开关](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L134-L163) [容量](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L315-L321)
- **把头注释 delay／newchilddelay 当现行 API**：实际构造和计时接口是 spawnperiod／regenperiod、两个开关及可选外部计时；按实现配置，不按旧注释推断 nil 的禁用语义。[旧注释](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L1-L6) [实际初值](../../../raw/archive/build-26148036/scripts/components/childspawner.lua#L69-L99)

## 验证记录

- 局部增补：2026-10-27，Steam buildid 26148036；核验 batcave 与 cave_entrance 的服务端组件配置、iscaveday／isday 开关、地表迁移器门禁及洞内靠近释放；通用计数、门禁和计时复用本页既有依据。未运行游戏，未据此审校全部巢穴或迁移器内部。
- 版本增量复核：2026-10-25，Steam buildid 26148036；漂移文件 actions.lua、prefabs/beehive.lua 对照 R-01/R-04 hunk 地图逐条核验：GOHOME 动作（spawner→childspawner→hideout 分发）逐行未变，beehive 仅移除 lifedrainable 标签（:169，与出巢／补员／应急池无关）；锚点重定位 2 处、结论无修订；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖普通／应急计数、容量与补员、生成／接管／回家时序、必要存读／移除、蜂箱季节／世界计时与野巢代表分工。未完整审校外来种类概率、所有巢穴、worldsettingstimer 内部或完整生物 Brain／SG。
- 结构检查：格式、来源、链接／锚点、大小及唯一登记通过；完整 modmain 示例通过已有 Full Moon Lua 5.1 syntax-only。位置／语法检查不证明游戏行为。
- 游戏运行：未运行；新建／读档、换季、世界禁用、满员／超额、归巢、捕获／死亡、离屏／唤醒及多模组包装顺序待实测。
