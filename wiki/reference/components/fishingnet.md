---
title: fishingnet
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "渔网物品的薄逻辑入口：只按目标点生成渔网表现实体并移交投网流程，动作、耐久与捕获都在别处。"
aliases: ["渔网", "渔网组件", "投网", "撒网", "fishingnet"]
tags: [component, item, equipment]
dst_build: "steam-buildid:26230496"
sources: ["components/fishingnet.lua", "components/fishingnetvisualizer.lua", "prefabs/fishingnet.lua", "prefabs/forest.lua", "prefablist.lua", "recipes.lua", "scrapbook_prefabs.lua", "prefabs/player_common.lua", "debugcommands.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGfishingnetvisualizer.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# fishingnet

## 快速判断

- 职责与适用：挂在“渔网”物品上的极薄服务端组件，整个组件只有 `CastNet` 一个入口——按目标点生成 `fishingnetvisualizer`，把投网、捕获、撒落全部交给表现实体；自身不移动、不捕获、不扣耐久。[fishingnet.lua:1-12](../../../raw/current/scripts/components/fishingnet.lua#L1-L12)
- 优先使用：原版动作链已经接好，`componentactions` 的 `fishingnet` 处理器在海面点／海面目标上给出右键 `CAST_NET`，动作 fn 再调 `CastNet`；模组复用入口即可，不必重写投网。[componentactions.lua:2115-2119](../../../raw/current/scripts/componentactions.lua#L2115-L2119)、[actions.lua:4954-4970](../../../raw/current/scripts/actions.lua#L4954-L4970)
- 重要副作用：每次调用都 `SpawnPrefab("fishingnetvisualizer")`，并把新实体写进 `self.visualizer`（只保留最后一次，旧引用被覆盖）；不检查是否已有网在场。[fishingnet.lua:6-9](../../../raw/current/scripts/components/fishingnet.lua#L6-L9)
- 修改边界：服务端组件，宿主在 `TheWorld.ismastersim` 之后才 `AddComponent`；耐久由宿主 `finiteuses` 按 `ACTIONS.CAST_NET` 消费，不归组件；组件无 `OnSave`／`OnLoad`／`OnRemoveFromEntity`。[prefabs/fishingnet.lua:55-66](../../../raw/current/scripts/prefabs/fishingnet.lua#L55-L66)、[:59-63](../../../raw/current/scripts/prefabs/fishingnet.lua#L59-L63)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CastNet(pos_x, pos_z, doer)` | `(number, number, entity) → boolean`，恒返回 `true`，无失败分支 | 生成表现实体后立刻 `BeginCast(doer, pos_x, pos_z)`；不自检距离／海面，不做耐久扣除，也不切换 `doer` 的 SG。[fishingnet.lua:5-12](../../../raw/current/scripts/components/fishingnet.lua#L5-L12) |
| `pos_x` / `pos_z` | 世界坐标（世界单位）；y 恒被丢弃 | 由调用方保证是海面点；原版用动作点，无动作点时退化为 `act.target` 的世界位置。[actions.lua:4960-4967](../../../raw/current/scripts/actions.lua#L4960-L4967) |
| `self.visualizer` | entity／`nil`；首次调用前为 `nil` | 只记最近一次生成的表现实体，不参与清理，也不代表“网是否在使用中”。[fishingnet.lua:9](../../../raw/current/scripts/components/fishingnet.lua#L9 |

### 与 fishingnetvisualizer 的分工

| 维度 | `fishingnet` | `fishingnetvisualizer` |
|---|---|---|
| 角色 | 逻辑入口：物品 → 目标点 → 生成表现实体 | 表现与执行：运动、动画事件、捕获、撒落、自删 |
| 生命周期 | 随物品存在（耐久耗尽才 `inst.Remove`） | 每次投网新建一个，收网结束时 `inst:Remove()` |
| 是否联网 | 宿主是联网物品 prefab | 宿主 prefab 无 `AddNetwork()`，仅主机端实体 |
| 谁先结束 | 投出后不再参与 | `final_pickup` 状态内自删 |

## 典型接入

### 原版链路：装备渔网右键海面

- 前提与选择：物品需同时具备 `fishingnet`、`inventoryitem`、`equippable`、`finiteuses`；动作由 `componentactions` 的 `fishingnet` 处理器提供，条件为右键 + 目标点在海面（`IsOceanAtPoint` 或存在虚拟海实体）+ 与投掷者距离平方大于 4（即大于 2 世界单位）。[componentactions.lua:4-13](../../../raw/current/scripts/componentactions.lua#L4-L13)、[:2115-2119](../../../raw/current/scripts/componentactions.lua#L2115-L2119)、[:2331-2336](../../../raw/current/scripts/componentactions.lua#L2331-L2336)
- 生效与撤销：`CAST_NET` 声明 `rmb=true, distance=12`；投掷者由 `SGwilson` 的 `ActionHandler(ACTIONS.CAST_NET, "cast_net")` 进入 `cast_net`，6 帧时 `PerformBufferedAction` 才真正生成网。撤销侧由表现实体向投掷者推 `begin_retrieving`／`begin_final_pickup`，依次进入 `cast_net_retrieving`／`cast_net_release`，`cast_net_release_pst` 播完回 `idle`；网每投一次扣 1 点耐久，`TUNING.FISHING_NET_USES` 为 22，耗尽执行 `inst.Remove`。[actions.lua:564](../../../raw/current/scripts/actions.lua#L564)、[SGwilson.lua:1481](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1481)、[:13884-13900](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13884-L13900)、[:13930-13974](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13930-L13974)、[prefabs/fishingnet.lua:59-63](../../../raw/current/scripts/prefabs/fishingnet.lua#L59-L63)、[tuning.lua:4048](../../../raw/current/scripts/tuning.lua#L4048
- 原版依据：注册名 `fishingnet`（`Prefab("fishingnet", fn, assets, prefabs)`），森林 prefab 列表与生成的 `PREFABFILES` 都包含它，因此实体可加载、可按名生成；但 `recipes.lua` 的配方整行被注释，正常玩法拿不到，只能用调试生成，属未完成内容。[prefabs/fishingnet.lua:83](../../../raw/current/scripts/prefabs/fishingnet.lua#L83)、[prefabs/forest.lua:342](../../../raw/current/scripts/prefabs/forest.lua#L342)、[prefablist.lua:440-441](../../../raw/current/scripts/prefablist.lua#L440-L441)、[recipes.lua:671](../../../raw/current/scripts/recipes.lua#L671)、[debugcommands.lua:1851](../../../raw/current/scripts/debugcommands.lua#L1851

### 模组复用：在自定义动作或组件里调用 CastNet

- 前提与选择：`doer` 必须是玩家——`BeginCast` 直接改 `doer.AnimState` 的手臂显隐并 `FacePoint`，非玩家实体没有这些接口；并且 `doer` 的 SG 要能处理 `begin_retrieving`／`begin_final_pickup`，当前只有 `SGwilson` 定义了 `cast_net*` 四状态，其他 SG 下网仍会照常播完并撒落，但投掷者不会退出原动作。[components/fishingnetvisualizer.lua:31-34](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L31-L34)、[SGwilson.lua:13930-13941](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13930-L13941)
- 生效与撤销：组件只负责建表现实体；网的收尾与物品放回全在 visualizer，因此调用方要自己保证网能走到 `final_pickup`。绕过 `CAST_NET` 直接调用不会扣耐久，也不会触发 `SetOnFinished(inst.Remove)`。

```lua
-- 复用原版入口：先自己判定海面与距离，再交给组件
local invobject = act.invobject
if invobject ~= nil and invobject.components.fishingnet ~= nil then
    local pt = act:GetActionPoint()
    if pt ~= nil then
        invobject.components.fishingnet:CastNet(pt.x, pt.z, act.doer)
    end
end
```

- 调整与验证：目标点单位是世界坐标，距离阈值 2 来自 `CanCastFishingNetAtPoint` 内的 `min_throw_distance`；替换 `CastNet` 前先确认 `visualizer` 的清理路径（见该页“常见陷阱与限制”）。

## 常见陷阱与限制

- 重复投网不互斥：`CastNet` 不检查是否已有网在场，`self.visualizer` 只覆盖引用，前一个表现实体仍按自己的 SG 收尾。→ 需要互斥时调用前检查 `self.visualizer ~= nil and self.visualizer:IsValid()`。[fishingnet.lua:5-11](../../../raw/current/scripts/components/fishingnet.lua#L5-L11)
- 组件不扣耐久：消费写在宿主 prefab 的 `finiteuses:SetConsumption(ACTIONS.CAST_NET, 1)`；直接调 `CastNet` 既不掉耐久也不会在耗尽时删除物品。[prefabs/fishingnet.lua:59-63](../../../raw/current/scripts/prefabs/fishingnet.lua#L59-L63)
- 原版内容未完成：prefab 资产注释写 “fishingnet unused”，Scrapbook 登记与配方均被注释，`player_common.lua` 注明对应动画 “Unfinished, bad file. For unused fishingnet”；表现实体 prefab 还没有网络组件。→ 不要把渔网当可照搬的原版成品案例。[prefabs/fishingnet.lua:3](../../../raw/current/scripts/prefabs/fishingnet.lua#L3)、[scrapbook_prefabs.lua:1063](../../../raw/current/scripts/scrapbook_prefabs.lua#L1063)、[player_common.lua:2008](../../../raw/current/scripts/prefabs/player_common.lua#L2008)、[prefabs/fishingnetvisualizer.lua:11-35](../../../raw/current/scripts/prefabs/fishingnetvisualizer.lua#L11-L35)
- 坐标由调用方负责：组件不做海面与距离校验，校验只在 `componentactions`；控制台或自定义动作直接调用可把网投到陆地任意位置。[componentactions.lua:4-13](../../../raw/current/scripts/componentactions.lua#L4-L13)
- 与钓竿无关：渔网走独立动作与表现链，不使用 `fishable` 的鱼量／咬钩状态机，也不共享 `fishingrod` 的目标与上钩字段；两者只在“钓鱼”语义上并列，见 [[reference/components/fishingrod]]、[[reference/components/fishable]]。

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 11 处（componentactions／actions／SGwilson 的 CAST_NET 链、tuning.lua 耐久、prefablist／player_common）；结论修订：无，仅锚点迁移。依据：枢纽 diff 审阅＋变更清单命中 actions.lua、componentactions.lua、SGwilson.lua、tuning.lua、recipes.lua、prefablist.lua、prefabs/forest.lua、debugcommands.lua、scrapbook_prefabs.lua、prefabs/player_common.lua（命中区域逐行核对均未漂移）。
- 源码审阅：2026-10-21，`steam-buildid:25643519`。通读 `components/fishingnet.lua` 全 14 行，并核验 `prefabs/fishingnet.lua`（组件挂载、耐久、装备回调、注册名）、`prefabs/forest.lua`、`prefablist.lua`、`recipes.lua`、`scrapbook_prefabs.lua`、`player_common.lua`、`debugcommands.lua`、`actions.lua`（`CAST_NET` 定义与 fn）、`componentactions.lua`（`CanCastFishingNetAtPoint` 与三处 `fishingnet` 处理器）、`SGwilson.lua`（`cast_net*` 四状态）、`SGfishingnetvisualizer.lua`、`tuning.lua`。主源码 SHA256 `6ab9de4e8742091a923f09846fef300d2d42da47cc8a24a0331479f5679e0fdb`。
- 游戏运行：未运行；未覆盖多人客户端表现、存档往返与耐久耗尽时的移除时序。
- 结构检查：`validate_page.py`（`--format` 指向页面格式、`--source-root` 指向源码根）通过，0 error；`validate_source_links.py --strict` 通过，本文 32 处行锚点、28 个唯一目标、14 个源文件，0 error／0 warning。文中 Lua 片段为节选，单独写过临时文件做 `parser_probe --syntax-only`，解析通过（exit 0），临时文件已删除。
