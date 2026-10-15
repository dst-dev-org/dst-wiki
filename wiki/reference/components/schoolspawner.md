---
title: "schoolspawner"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "按玩家附近海域、鱼数和阻挡标记尝试自然鱼群，并提供鱼书等直接生成鱼群与按季节地皮选鱼接口。"
aliases: ["schoolspawner", "海鱼群生成", "自然刷海鱼", "鱼书直接生鱼", "鱼群生成阻挡"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/schoolspawner.lua", "prefabs/forest.lua", "prefabs/world.lua", "prefabs/oceanfishdef.lua", "prefabs/fishschoolspawnblocker.lua", "prefabs/books.lua", "prefabs/chum_aoe.lua", "components/oceantrawler.lua", "tuning.lua", "componentutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# schoolspawner

## 快速判断

- 职责与适用：森林世界主模拟的海鱼群生成服务，兼有玩家自然调度、季节地皮选鱼、直接生成鱼群和伴随海洋生物判定。构造断言服务端，森林 `master_postinit` 安装；世界客户端在此前返回。[schoolspawner.lua:4-22](../../../raw/current/scripts/components/schoolspawner.lua#L4-L22) [forest.lua:639-646](../../../raw/current/scripts/prefabs/forest.lua#L639-L646) [world.lua:568-605](../../../raw/current/scripts/prefabs/world.lua#L568-L605)
- 优先使用：自然密度资格用 `ShouldSpawnANewSchoolForPlayer`；找自然候选点用 `GetSpawnPoint`；鱼书等指定效果用 `SpawnSchool`，并处理 nil／0 返回。三者职责独立，不应把直接生成当成自然资格检查。[schoolspawner.lua:95-105](../../../raw/current/scripts/components/schoolspawner.lua#L95-L105) [schoolspawner.lua:159-191](../../../raw/current/scripts/components/schoolspawner.lua#L159-L191) [schoolspawner.lua:206-260](../../../raw/current/scripts/components/schoolspawner.lua#L206-L260)
- 重要副作用：成功安排至少一条鱼时创建临时阻挡实体并向世界发 `schoolspawned`；中浪／汹涌海域还可能延后生成一角鲸、鲨鱼。返回数包含延后任务，不保证所有鱼已经出现或仍然存活。[schoolspawner.lua:193-260](../../../raw/current/scripts/components/schoolspawner.lua#L193-L260)
- 修改边界：自然任务没有季节、昼夜或睡眠暂停开关；季节只影响鱼种表。组件空存读钩子不恢复玩家任务剩余时间；初始化已有玩家循环另有变量使用缺口，见限制。[schoolspawner.lua:107-151](../../../raw/current/scripts/components/schoolspawner.lua#L107-L151) [schoolspawner.lua:275-282](../../../raw/current/scripts/components/schoolspawner.lua#L275-L282)

## 关键接口

| 接口／字段 | 签名或返回 | 条件、作用与证据 |
|---|---|---|
| `ShouldSpawnANewSchoolForPlayer(player)` | 返回布尔值 | 只判断玩家附近海域比例、阻挡幸运判定与现存可钓海鱼数量概率；不找具体落点或生成鱼。[schoolspawner.lua:157-175](../../../raw/current/scripts/components/schoolspawner.lua#L157-L175) |
| `GetSpawnPoint(pt)` | Vector3 位置或 nil | 依次尝试半径5～9的8方向、10～14的12方向、15～19的16方向扇形；候选须能按当前季节地皮选鱼且为海洋。[schoolspawner.lua:177-191](../../../raw/current/scripts/components/schoolspawner.lua#L177-L191) |
| `SpawnSchool(spawnpoint,target,override_spawn_offset?)` | 无鱼种返回 nil；否则返回安排数 count | `target` 传给特殊海洋生物的幸运掷点；可省略时按基础概率。自定义 offset 是相对中心偏移，不是世界坐标；只替换鱼群中心偏移搜索，不取消逐鱼游泳落点测试。[schoolspawner.lua:206-260](../../../raw/current/scripts/components/schoolspawner.lua#L206-L260) [TryLuckRoll componentutil.lua:2237-2251](../../../raw/current/scripts/componentutil.lua#L2237-L2251) |
| `GetFishPrefabAtPoint(spawnpoint)` | 鱼 prefab 名或 nil | 每次重新按季节与地皮加权选鱼，不生成、不占用配额，也不保证下次调用选同一种。[schoolspawner.lua:120-126](../../../raw/current/scripts/components/schoolspawner.lua#L120-L126) [schoolspawner.lua:262-269](../../../raw/current/scripts/components/schoolspawner.lua#L262-L269) |
| `OnSave()`／`OnLoad(data)` | 保存空表／空实现 | 不保存任务、玩家表、鱼群实体引用或特殊生物延迟；不能据此认为已经生成的实体也由此组件负责存档。[schoolspawner.lua:275-282](../../../raw/current/scripts/components/schoolspawner.lua#L275-L282) |

## 关键流程

### 自然尝试：排程、海域、阻挡与局部鱼数

玩家加入事件安排一次任务，离开事件取消该玩家任务；每次尝试后清任务槽并续排。每轮等待读取 `SCHOOL_SPAWN_DELAY`，默认15～60秒，无首次特殊短延迟。任务挂在玩家实体上；不是世界固定周期一次刷所有玩家。[schoolspawner.lua:95-118](../../../raw/current/scripts/components/schoolspawner.lua#L95-L118) [schoolspawner.lua:132-151](../../../raw/current/scripts/components/schoolspawner.lua#L132-L151) [tuning.lua:19-20](../../../raw/current/scripts/tuning.lua#L19-L20) [tuning.lua:5150-5155](../../../raw/current/scripts/tuning.lua#L5150-L5155)

以玩家为中心计算半径25范围的海洋地皮比例 `p`，必须严格大于0.1；再在半径30内数 `fishschoolspawnblocker`，以 `1 - 数量/3` 为基础概率进行 `LuckFormulas.SchoolSpawn` 判定。该公式只在正幸运时加 `luck*0.25`，所以“三个阻挡标记永远禁刷”不是对所有玩家都成立的结论。[schoolspawner.lua:157-175](../../../raw/current/scripts/components/schoolspawner.lua#L157-L175) [tuning.lua:5151-5155](../../../raw/current/scripts/tuning.lua#L5151-L5155) [公式 componentutil.lua:2078-2082](../../../raw/current/scripts/componentutil.lua#L2078-L2082) [绑定 componentutil.lua:2167](../../../raw/current/scripts/componentutil.lua#L2167 [幸运取值与判定 componentutil.lua:2203-2214](../../../raw/current/scripts/componentutil.lua#L2203-L2214)

通过阻挡判定后，在半径30内数同时有 `oceanfish`、`oceanfishable` 的实体。无鱼时随机数需小于 `p`；1～5条时需小于 `p/(鱼数+1)`；超过5条不进行自然生成。这个5是尝试前的局部密度门槛，不是每群大小或全世界硬容量，新生成整群可以让附近总鱼数超过5。[schoolspawner.lua:157-175](../../../raw/current/scripts/components/schoolspawner.lua#L157-L175) [tuning.lua:5151-5154](../../../raw/current/scripts/tuning.lua#L5151-L5154)

### 选鱼、鱼群中心与逐鱼生成

`PickSchool` 读取 `FISH_DATA.school[当前季节][落点地皮]` 加权选类型，再取鱼定义。定义表为其他季节复制秋季基础表并追加季节鱼；无对应表、空候选或无对应鱼定义就无结果。鱼种、`schoolmin/max` 和 `schoolrange` 来自鱼定义；例如 `oceanfish_small_1` 的群大小为6～10、散布范围2，不能把这一例当所有鱼固定大小。[schoolspawner.lua:120-126](../../../raw/current/scripts/components/schoolspawner.lua#L120-L126) [oceanfishdef.lua:65-77](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L65-L77) [oceanfishdef.lua:188-210](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L188-L210) [oceanfishdef.lua:1030-1084](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L1030-L1084)

SpawnSchool 重新选鱼，先在传入中心创建 `schoolherd_<鱼prefab>`。鱼群实际中心使用 override 偏移；未提供时依次尝试半径20、13、7的可游泳偏移，均失败回退零偏移。然后随机群大小与朝向，每条鱼再在 `schoolrange` 内找可游泳偏移：第一条立即生成，后续延迟0.1～1.1秒。[schoolspawner.lua:206-239](../../../raw/current/scripts/components/schoolspawner.lua#L206-L239)

实际 `DoSpawnFish` 先确认 herd 仍有效，再生成鱼、传送／旋转、启用 `herdmember`、设置 herd prefab，进入 `arrive` 并加入群体。count 在安排延后任务时就递增；herd 若在任务执行前失效，该鱼不会生成。count=0 时移除空 herd；count>0 时在原传入中心创建 blocker、向世界发 `{spawnpoint=spawnpoint}`。[schoolspawner.lua:193-260](../../../raw/current/scripts/components/schoolspawner.lua#L193-L260)

blocker 是非联网实体，带 `fishschoolspawnblocker` 标签和 timer，默认一游戏日后移除。它影响后续自然资格概率，不在 SpawnSchool 内阻止直接生成。[fishschoolspawnblocker.lua:2-26](../../../raw/current/scripts/prefabs/fishschoolspawnblocker.lua#L2-L26) [tuning.lua:5155](../../../raw/current/scripts/tuning.lua#L5155 [自然判定 schoolspawner.lua:163-166](../../../raw/current/scripts/components/schoolspawner.lua#L163-L166)

### 伴随一角鲸与鲨鱼

只有 count>0 且原传入中心地皮为 `OCEAN_SWELL`／`OCEAN_ROUGH` 才分别尝试两类生物。先以半径100内对应标签数量小于2为门槛，基础概率一角鲸0.1、鲨鱼0.075，再乘附近拖网返回的倍率并用 `SpecialSchoolSpawn` 幸运判定。查询遇到第一个有 oceantrawler 组件的实体就返回其倍率，不累乘所有拖网；已放下且满载时拖网返回专用倍率，其余为1。[schoolspawner.lua:27-93](../../../raw/current/scripts/components/schoolspawner.lua#L27-L93) [schoolspawner.lua:243-253](../../../raw/current/scripts/components/schoolspawner.lua#L243-L253) [tuning.lua:6088-6092](../../../raw/current/scripts/tuning.lua#L6088-L6092) [oceantrawler.lua:128-134](../../../raw/current/scripts/components/oceantrawler.lua#L128-L134)

一角鲸用10、鲨鱼用20的可游泳偏移，等待8～10秒后仅再检查落点没有船平台才生成；不在延迟结束时重查附近数量或概率。鲨鱼还先 `OnEnterOcean()` 再进 `eat_pre`，一角鲸进 `emerge`。该幸运公式与自然 blocker 判定不同，正幸运降低、负幸运提高伴随生物概率。[schoolspawner.lua:38-93](../../../raw/current/scripts/components/schoolspawner.lua#L38-L93) [公式 componentutil.lua:2084-2090](../../../raw/current/scripts/componentutil.lua#L2084-L2090) [绑定 componentutil.lua:2173](../../../raw/current/scripts/componentutil.lua#L2173

## 典型接入

### 原版鱼书与诱鱼剂：直接生成并自行决定重试

鱼书先确认世界有组件，在读者周围取候选中心并给小范围 override；直接调用 SpawnSchool，对 nil／0 结果换角度重试，所有轮次失败才返回无水提示。它没有先调用自然密度资格函数，所以自然鱼数／blocker 限制不能作为“鱼书一定失败”的判断。[books.lua:558-603](../../../raw/current/scripts/prefabs/books.lua#L558-L603)

诱鱼剂也直接调用 SpawnSchool，但自行要求附近可钓海鱼数小于默认5加额外10，传入半径8的中心偏移；失败时在自己的次数限制内重试。这是宿主独立门槛，不是 SpawnSchool 内置上限。模组复用时须在服务端保证世界组件存在、按实际效果控制重试与数量，并接受直接调用仍有 blocker、事件和伴随生物副作用；本页不另造跨世界 demo。[chum_aoe.lua:13-23](../../../raw/current/scripts/prefabs/chum_aoe.lua#L13-L23) [chum_aoe.lua:48-74](../../../raw/current/scripts/prefabs/chum_aoe.lua#L48-L74)

## 常见陷阱与限制

- **SpawnSchool 等于完整自然生成：** 它不调用海洋比例、局部鱼数、blocker 资格或 GetSpawnPoint；只按中心选鱼并执行自己的偏移／逐鱼流程，调用方负责额外门槛。[schoolspawner.lua:95-105](../../../raw/current/scripts/components/schoolspawner.lua#L95-L105) [schoolspawner.lua:206-260](../../../raw/current/scripts/components/schoolspawner.lua#L206-L260)
- **构造会正确补排所有已在线玩家：** 当前源码 `for i,v in ipairs(AllPlayers)` 内调用的是 `ScheduleSpawn(player)` 而非 `v`，组件内没有对应局部绑定；不能据此承诺已有玩家场景可用。若构造时表非空且全局 player 无值，调度将访问 nil；本页仅记录源码分支，未在游戏复现。正常加入事件使用具名 player 参数。[schoolspawner.lua:107-110](../../../raw/current/scripts/components/schoolspawner.lua#L107-L110) [schoolspawner.lua:132-151](../../../raw/current/scripts/components/schoolspawner.lua#L132-L151)
- **退出或移除组件会撤掉全部鱼群任务：** 玩家离开只取消该玩家的自然任务；逐鱼与特殊生物延后任务挂在世界上且没有记入 `_scheduledtasks`，组件也无 `OnRemoveFromEntity` 统一撤销钩子。已生成鱼群的生命周期应沿群体／鱼本体核验。[schoolspawner.lua:22](../../../raw/current/scripts/components/schoolspawner.lua#L22 [schoolspawner.lua:41-93](../../../raw/current/scripts/components/schoolspawner.lua#L41-L93) [schoolspawner.lua:113-138](../../../raw/current/scripts/components/schoolspawner.lua#L113-L138) [schoolspawner.lua:225-239](../../../raw/current/scripts/components/schoolspawner.lua#L225-L239)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件全文、森林／世界服务端安装路径、所用 tuning、鱼定义抽样与季节表构造、blocker 全文、鱼书／诱鱼剂完整生成回调、拖网倍率函数和对应幸运公式／掷点，覆盖自然资格、直接生成、数量返回、伴随生物与存读／退出边界。
- 游戏运行：未运行；未复现构造已有玩家的变量缺口，未测多人重叠、平台移动竞态、延后生成分布及鱼群持久化。未展开 herd／海鱼本体或拖网离屏捕获完整机制。
- 结构检查：`validate_page.py` 通过；两页合并执行 `validate_source_links.py --strict`，94 个行号引用、0错误／0警告。本页无新增 Lua 示例；机械检查不代替游戏测试。

相关：[[reference/components/herd]] · [[reference/components/herdmember]] · [[reference/components/luckuser]] · [[reference/components/oceanfishable]]
