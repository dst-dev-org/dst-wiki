---
title: "birdspawner"
created: "2026-10-24"
updated: "2026-10-28"
type: component
summary: "管理森林世界按玩家调度的飞鸟生成、落点与鸟种选择，并提供显式生成、时间修正和离屏追踪接口。"
aliases: ["birdspawner", "世界刷鸟", "改变自然鸟生成速度", "地皮鸟种", "鸟离屏消失", "SetBirdTypesForTile"]
tags: [component, creature, lifecycle, modding, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/birdspawner.lua", "components/birdattractor.lua", "prefabs/forest.lua", "prefabs/player_common.lua", "prefabs/hats.lua", "prefabs/birds.lua", "prefabs/books.lua", "prefabs/birdtrap.lua", "util/sourcemodifierlist.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# birdspawner

## 快速判断

- 职责与适用：森林世界的主模拟飞鸟管理器，构造断言 `TheWorld.ismastersim`，原版在森林 master_postinit 安装；围绕活跃玩家尝试生成，不是通用物品周期产物器或单体住户管理。[断言](../../../raw/current/scripts/components/birdspawner.lua#L8-L10) [世界安装](../../../raw/current/scripts/prefabs/forest.lua#L639-L644) [自然尝试](../../../raw/current/scripts/components/birdspawner.lua#L154-L165)
- 优先使用：玩家附近数量／间隔差值走 [[reference/components/birdattractor]]；世界延迟倍率用 `SetTimeScaleModifier(factor,key)`，地皮鸟种用 `SetBirdTypesForTile`。改值后需重排现有任务才调用 `ToggleUpdate()`。[公共接口](../../../raw/current/scripts/components/birdspawner.lua#L593-L603) [强制重排](../../../raw/current/scripts/components/birdspawner.lua#L195-L213)
- 重要副作用：直接 `SpawnBird(point)` 不经过自然数量、夜晚或落点筛查，并可能被附近诱饵／陷阱改落点；它会改传入点的坐标。`StartTracking` 还改实体持久化并在休眠后删除。[生成](../../../raw/current/scripts/components/birdspawner.lua#L519-L565) [追踪](../../../raw/current/scripts/components/birdspawner.lua#L568-L590)
- 修改边界：依赖世界 Map／GroundCreep、状态、timer 等宿主设施；不为自定义生物或洞穴盲加组件。若当前世界无该组件，原版鸟书也直接拒绝；组件没有网络副本或自身存档钩子。[环境引用](../../../raw/current/scripts/components/birdspawner.lua#L63-L91) [初始化依赖](../../../raw/current/scripts/components/birdspawner.lua#L407-L425) [鸟书判断](../../../raw/current/scripts/prefabs/books.lua#L279-L282)

## 关键接口

| 接口／配置 | 条件、作用与证据 |
|---|---|
| `ToggleUpdate()` | 无参数；实际调用内部 ToggleUpdate(true)，取消所有活跃玩家任务后以 initialspawn 规则重排。夜晚或基础上限<=0时停止；不是切换布尔开关或立即刷鸟。[入口](../../../raw/current/scripts/components/birdspawner.lua#L441-L442) [门禁与重排](../../../raw/current/scripts/components/birdspawner.lua#L191-L213) |
| `SetTimeScaleModifier(factor,key)`／`RemoveTimeScaleModifier(key)` | 按世界实体为来源、独有 key 注册乘法修正，初值1；倍率乘生成延迟，越小越快，不修改鸟数。仅改来源表，不重排已排任务；模组提供有限正数并成对撤销。[初值](../../../raw/current/scripts/components/birdspawner.lua#L90-L91) [使用](../../../raw/current/scripts/components/birdspawner.lua#L172-L180) [接口](../../../raw/current/scripts/components/birdspawner.lua#L597-L603) |
| `SetBirdTypesForTile(tile_id,bird_list)` | 直接替换此生成器的地皮→鸟名列表；不是给鸟加地图标签。空列表可令 PickBird 返回 nil，SpawnBird 直接返回；未配置地皮默认 crow。实际鸟种还会经活动／稻草人、变异与冬季分支。[选择](../../../raw/current/scripts/components/birdspawner.lua#L222-L249) [替换](../../../raw/current/scripts/components/birdspawner.lua#L593-L594) [无结果](../../../raw/current/scripts/components/birdspawner.lua#L519-L522) |
| `GetSpawnPoint(pt,is_corpse?)` | 返回位置或 nil；距离中心6～12，扇形尝试12次。允许水面；阻止屏障竞技场、4半径内 birdblocker；普通鸟还排除月亮风暴和地面黏液，尸体模式仅放宽后两项。[完整测试](../../../raw/current/scripts/components/birdspawner.lua#L461-L484) |
| `SpawnBird(spawnpoint,ignorebait?)` | 根据地皮选择并生成实体、设置朝向／下落高度，再处理诱饵重定位，返回鸟或 nil；不调用 GetSpawnPoint。ignorebait=true 同时跳过诱饵和陷阱分支，不会绕过选鸟规则。[实现](../../../raw/current/scripts/components/birdspawner.lua#L519-L565) |
| `SpawnBirdCorpse(spawnpoint)`／`SpawnCorpseForPlayer(player)` | 前者按选鸟结果设置鸟尸体 build，改点y=15再 Teleport；后者检查玩家64半径内 birdcorpse 上限并尝试尸体落点。没有普通鸟的玩家吸引修正，也不自动安排下一轮。[尸体尝试](../../../raw/current/scripts/components/birdspawner.lua#L125-L151) [具体生成](../../../raw/current/scripts/components/birdspawner.lua#L495-L515) [公开包装](../../../raw/current/scripts/components/birdspawner.lua#L605-L606) |
| `StartTracking(target)`／`StopTracking(target)` | 开始记录原 persists 是否为true、设 false、监听目标 entitysleep；休眠下一任务确认仍被追踪且仍睡才 Remove。停止恢复保存的布尔值、删记录并撤监听，不直接移除鸟。[延后删除](../../../raw/current/scripts/components/birdspawner.lua#L258-L261) [追踪接口](../../../raw/current/scripts/components/birdspawner.lua#L568-L590) |
| `GetPostHailEasingMult()` | 返回月雹后恢复进度，timer无对应阶段时1；原鸟书用它降低生成数量，不是鸟种或伤害倍率。[计算](../../../raw/current/scripts/components/birdspawner.lua#L103-L105) [公共接口](../../../raw/current/scripts/components/birdspawner.lua#L609-L610) [消费者](../../../raw/current/scripts/prefabs/books.lua#L318-L321) |
| `SetSpawnTimes`／`SetMaxBirds`／`SpawnModeNever/Light/Med/Heavy` | 这些方法当前都是 deprecated 空实现，调用不改变排程或上限。不要沿训练记忆生成旧 setter 示例。[保留空接口](../../../raw/current/scripts/components/birdspawner.lua#L433-L459) |

### 自然排程、数量与天气

构造缓存 `TUNING.BIRD_SPAWN_MAX` 和延迟上下限，当前基础是4、5～15秒；运行中改全局 TUNING 不会自动更新已捕获的局部值。调度先要求非夜晚且基础上限>0；每位玩家尝试前在其周围64半径统计所有 bird 标签实体，再与基础值＋玩家吸引差值比较。不是全世界最多4只，也不是 `_birds` 追踪表大小控制生成。[缓存](../../../raw/current/scripts/components/birdspawner.lua#L69-L73) [常量](../../../raw/current/scripts/tuning.lua#L1078-L1083) [计数](../../../raw/current/scripts/components/birdspawner.lua#L154-L165)

常规下轮延迟在修正后的 min～max 内取随机；初次／强制重排改为 0～(max-min)，然后乘全局时间倍率。有来源修正时由调用者保证上下限有效；不能用“马上重排”推断立刻补齐数量。雨天倍率当前0.25，月雹后恢复阶段另加随 timer 变化的倍率；自然任务在月雹状态改为鸟尸体分支。事件 timer 还有独立批量掉尸体，夜间停止自然排程不等于禁止所有事件生成。[排程](../../../raw/current/scripts/components/birdspawner.lua#L168-L180) [雨修正](../../../raw/current/scripts/components/birdspawner.lua#L322-L328) [恢复更新](../../../raw/current/scripts/components/birdspawner.lua#L108-L114) [事件批量生成](../../../raw/current/scripts/components/birdspawner.lua#L268-L310) [当前倍率](../../../raw/current/scripts/tuning.lua#L2355-L2356)

## 典型接入

### 修改森林自然刷鸟的等待速度

在目标模组已确认森林主模拟初始化后的入口取得现有组件，使用独有字符串 key 设置正的时间倍率；如果要立即替换现有等待，再调用一次 ToggleUpdate。停止该效果时 RemoveTimeScaleModifier(原key)，按相同需求决定是否重排。这是调用与生命周期指引，不是跨世界通用刷鸟 demo；倍率影响所有活跃玩家，且与天气／其他来源相乘。[接口](../../../raw/current/scripts/components/birdspawner.lua#L597-L603) [来源乘法](../../../raw/current/scripts/components/birdspawner.lua#L90-L91) [消费者与重排](../../../raw/current/scripts/components/birdspawner.lua#L172-L213)

若只改变特定装备持有者的附近上限／等待差值，优先采用 [[reference/components/birdattractor#典型接入]] 的羽毛帽增删来源路径，不通过修改世界基础值影响全部玩家。

### 显式召鸟与追踪是两份协议

- **鸟书**：每只先 GetSpawnPoint，再 SpawnBird(pos,true)，有效生成后加 magicalbird 并进入 delay_glide。直接生成不走自然密度判断；书另行控制数量、月雹分流及成功结果，本页不把这套前提当 SpawnBird 内置。[取点与生成](../../../raw/current/scripts/prefabs/books.lua#L323-L334)
- **鸟本体**：普通鸟服务端工厂主动向世界注册追踪，并在 onremove／enterlimbo／death 调 StopTrackingFn。不是每个 SpawnPrefab 出来的任意生物都自动登记；新鸟需核验自己的工厂协议。[鸟接线](../../../raw/current/scripts/prefabs/birds.lua#L390-L396)
- **捕鸟器屏外捷径**：CatchOffScreen 直接用陷阱位置 SpawnBird，再把鸟交给 DoSpring；它不调用 GetSpawnPoint，不应用普通自然调度。世界自然刷鸟被限制，不能据此认定陷阱不会捕到鸟。[原版路径](../../../raw/current/scripts/prefabs/birdtrap.lua#L31-L43)

### 诱饵、恐吓与选鸟的边界

正常 SpawnBird 可在候选点15半径内查可吃且有 bait 的地面物品或设好的鸟陷阱，再改落点；两分支都要求目标附近8半径没有 scarytoprey。这个恐吓检查针对**诱饵／陷阱重定位**，不在 GetSpawnPoint 的一般落点测试内，不能当区域禁刷鸟开关。具体诱饵、陷阱行为复用 [[reference/components/bait]]、[[reference/components/trap]]。[重定位条件](../../../raw/current/scripts/components/birdspawner.lua#L518-L559) [危险半径](../../../raw/current/scripts/components/birdspawner.lua#L252-L255)

鸟种先按地皮取候选，再处理嘉年华／稻草人、雹后变异和冬季 robin 替换；稻草人不是提高 maxbirds 的玩家来源。自定义地皮鸟表需保留所需 prefab 资源，非空错误名称也未在生成后防御 SpawnPrefab 失败。[选择顺序](../../../raw/current/scripts/components/birdspawner.lua#L222-L249) [生成后直接访问](../../../raw/current/scripts/components/birdspawner.lua#L519-L527)

## 常见陷阱与限制

- **用空 setter 或改 TUNING 即时调频：** 旧方法不工作，基础参数已缓存；使用当前公开的倍率或玩家来源接口，并区分改未来任务和重排现有任务。[缓存](../../../raw/current/scripts/components/birdspawner.lua#L71-L73) [空方法](../../../raw/current/scripts/components/birdspawner.lua#L433-L459)
- **把 SpawnBird 视为安全资格检查：** 它只选种并生成，不重查夜晚、附近数量、birdblocker、地图可通行或屏障。需要受控落点时先 GetSpawnPoint，再处理 nil 与真实生成结果。[落点接口](../../../raw/current/scripts/components/birdspawner.lua#L462-L484) [生成入口](../../../raw/current/scripts/components/birdspawner.lua#L519-L525)
- **复用传入 Vector3 却期望坐标不变：** 生成鸟可把 y 改为15，诱饵／陷阱还改 xz；尸体生成也改 y。需要保留原位置时自行使用独立位置值。[鸟坐标](../../../raw/current/scripts/components/birdspawner.lua#L530-L563) [尸体坐标](../../../raw/current/scripts/components/birdspawner.lua#L511-L513)
- **追踪就是永久保存生成结果：** StartTracking 明确设 persists=false，睡眠后删除；宠物／库存等用途需沿实际宿主 StopTracking 协议，不把调试鸟数当世界存档列表。[追踪](../../../raw/current/scripts/components/birdspawner.lua#L568-L590)
- **动态移组件自动取消所有任务：** 实现没有 OnRemoveFromEntity；自然任务挂在玩家上，世界还有监听和雹事件任务，不能只移除 world.components.birdspawner 就认为完整停用。原版构造从 AllPlayers 建活跃表，加入／离开事件与 OnPostInit 恢复排程；组件无独立存档，雹阶段借用世界 timer，不是全部状态自动保存。[任务归属](../../../raw/current/scripts/components/birdspawner.lua#L172-L188) [玩家变动](../../../raw/current/scripts/components/birdspawner.lua#L376-L396) [初始化](../../../raw/current/scripts/components/birdspawner.lua#L402-L425)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 3 处（tuning BIRD_SPAWN 常量段 +4、雨天／雹后倍率 +10、forest master_postinit -3）；无结论修订，仅锚点迁移——BIRD_SPAWN_MAX=4／BIRD_RAIN_FACTOR=0.25／BIRD_POST_HAIL_FACTOR=6 值未变（tuning-hunk-map 5 改值键不含），birdspawner.lua 及各消费 prefab 未变（forest 本版删 prefabs 表 3 行）。依据：tuning-hunk-map 键级审阅＋变更清单命中 tuning.lua、prefabs/forest.lua。
- 源码审阅：2026-10-24，Steam buildid 25643519；完整读取 birdspawner，重点核验森林安装、自然玩家排程与吸引差值、公开接口、地面／鸟种门禁、追踪、鸟书和捕鸟器分流。月雹仅核对调度分支、timer恢复与倍率入口，未审校尸体突变、客户端声音管理器或所有生物SG。
- 结构检查：本页格式、来源、本地链接与源码引用严格检查通过；本批两页共90处引用无错误／警告。无完整 Lua 示例，导航登记由主代理统一检查。
- 游戏运行：未运行；多人重叠数量、天气延迟、直接生成、离屏移除及模组间鸟表／倍率冲突未实测。

相关：[[reference/components/birdattractor]]、[[reference/components/spawner]]、[[reference/components/periodicspawner]]。

## 相关链接

- [[reference/components/lunarhailbirdsoundmanager]]：月雹鸟声效档位的世界网络同步。
