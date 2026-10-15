---
title: "plantregrowth"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "按植物种类时钟、原始区域密度与落点条件在母株周边生成后代，区分本株成长和采后恢复。"
aliases: ["常青树周边再生加快", "树木自然繁殖", "植物周边生成树苗", "plantregrowth"]
tags: [component, lifecycle, callback, worldgen, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/plantregrowth.lua", "regrowthutil.lua", "prefabs/evergreens.lua", "prefabs/deciduoustrees.lua", "mainfunctions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# plantregrowth

## 快速判断

- 职责与适用：母株周边生态再生，通常另生成树苗；本株阶段成长用 [[reference/components/growable]]，采后恢复用 [[reference/components/pickable]]，批量产物累积用 [[reference/components/harvestable]]。[尝试生成](../../../raw/current/scripts/components/plantregrowth.lua#L208-L236) [常青树配置](../../../raw/current/scripts/prefabs/evergreens.lua#L907-L910)
- 优先使用：保留原版母株和世界数据；首次配置 SetRegrowthRate 才登记调度。已有实例调整基础等待时改 regrowthrate，按需 ResetGrowthTime，**不要再次调用 setter 当普通改值**。[登记](../../../raw/current/scripts/components/plantregrowth.lua#L76-L100) [接口](../../../raw/current/scripts/components/plantregrowth.lua#L151-L159)
- 重要副作用：到期先重置下一次时间，落点／密度失败也等下轮；母株区域缺原始密度时会注销调度。不是到期必生成，也不保证移到合适区域自动恢复。[流程](../../../raw/current/scripts/components/plantregrowth.lua#L208-L236) [区域数据](../../../raw/current/scripts/regrowthutil.lua#L9-L35)
- 修改边界：服务端母株、种类时间倍率及世界生成密度共同生效；仅 AddComponent 给新 Prefab 不构成完整繁殖方案。该组件的周边生成不负责被砍树原地复活。[种类时钟](../../../raw/current/scripts/components/plantregrowth.lua#L23-L72) [落点](../../../raw/current/scripts/components/plantregrowth.lua#L183-L201)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetRegrowthRate(rate)` | 正的基础等待量，供种类内部时钟使用 | 设置 regrowthrate 并 RegisterUpdate，必要时初始化到期值；没有已登记保护，重复调用会重复插入桶。[实现](../../../raw/current/scripts/components/plantregrowth.lua#L155-L159) [桶插入](../../../raw/current/scripts/components/plantregrowth.lua#L91-L100) |
| `regrowthrate`／`ResetGrowthTime()` | 新到期=当前种类时钟＋rate 的随机 ±20% | 改字段影响以后重置；显式 Reset 会丢弃本轮剩余量。要求先有对应种类内部时钟，不单独初始化注册。[实现](../../../raw/current/scripts/components/plantregrowth.lua#L151-L159) |
| `SetProduct(prefab)`／`SetSearchTag(tag)` | 后代名／密度计数的必需标签 | 未设置则都用母株 prefab；换后代不改变 GetFiveRadius 采用母株种类的规则，计数标签要覆盖希望限制的群体。[setter](../../../raw/current/scripts/components/plantregrowth.lua#L163-L168) [消费](../../../raw/current/scripts/components/plantregrowth.lua#L229-L236) |
| `SetSkipCanPlantCheck(bool)` | 仅跳过 CanPlantAtPoint | 不跳过 Prefab 过滤、道路、实体占位和建筑／墙体阻挡；不是强制生成开关。[实现](../../../raw/current/scripts/components/plantregrowth.lua#L171-L172) [门禁](../../../raw/current/scripts/components/plantregrowth.lua#L196-L200) |
| `TrySpawnNearby()` | 无显式成功返回 | 未到期早退；到期先重置，必要时缓存母株 fiveradius，再选点／查目标区域半径，数量<5 才 SpawnPrefab 并放置。生成失败无 nil 保护，product 必须有效。[实现](../../../raw/current/scripts/components/plantregrowth.lua#L208-L236) |
| `TimeMultipliers` | 类上公开的种类→函数表 | 模块用母株 prefab 查函数，每轮推进种类时钟；新增种类没有对应项会调用 nil，不能只给现有组件改 product 就当注册新母株。[表](../../../raw/current/scripts/components/plantregrowth.lua#L23-L67) [导出](../../../raw/current/scripts/components/plantregrowth.lua#L149) |

### 计时、密度与清理协作

模块共享种类时钟，增量是实际 dt×种类／季节倍率×TUNING.REGROWTH_TIME_MULTIPLIER；例如 evergreen 冬季为 0、夏季为 2 倍，而 deciduoustree 仅春季推进。世界任务每 31 秒只处理一个桶，每桶最多 50 个登记项；不等于每株每 31 秒检查，也不能把 rate 直接解释成现实必达秒数。[倍率](../../../raw/current/scripts/components/plantregrowth.lua#L23-L35) [轮询](../../../raw/current/scripts/components/plantregrowth.lua#L62-L100)

GetFiveRadius 先定位 topology 节点，再取 generated.densities[区域ID][母株prefab]，由原始密度算半径；它不是当前植物数量，也不是任意地图点均可用。候选点在母株半径的 1/2～1 倍间尝试 10 个角度，检查土壤、过滤、道路、3 单位内无实体、8 单位内无 structure／wall，再在候选点所属区域半径内按 searchtag 计数。[半径](../../../raw/current/scripts/regrowthutil.lua#L2-L35) [选点](../../../raw/current/scripts/components/plantregrowth.lua#L183-L203) [计数](../../../raw/current/scripts/components/plantregrowth.lua#L229-L236)

OnSave 只存相对种类时钟的剩余量，OnLoad 恢复到期值，配置及内部时钟由构造／首次登记先建立；不是保存真实时间或补算离线繁殖。移除组件和移除实体两个钩子都会注销，正常无重复登记时最后一个桶清空会停共享任务。[存读](../../../raw/current/scripts/components/plantregrowth.lua#L244-L254) [移除钩子](../../../raw/current/scripts/components/plantregrowth.lua#L175-L180) [注销](../../../raw/current/scripts/components/plantregrowth.lua#L103-L130)

## 典型接入

### 常青树周边再生的基础等待减半，保留原版条件

放目标模组 `modmain.lua`，不新增 Prefab／资源／配方。常青树工厂已登记 plantregrowth，产物为 pinecone_sapling；此处改字段并重置等待，避免第二次注册。evergreen_normal／tall／short 的工厂也将实体规范为 evergreen，实际生成会执行规范名 PostInit；不是只影响随机高度变体。[配置](../../../raw/current/scripts/prefabs/evergreens.lua#L59-L65) [规范名／标签](../../../raw/current/scripts/prefabs/evergreens.lua#L850-L856) [初次登记](../../../raw/current/scripts/prefabs/evergreens.lua#L907-L910) [Hook 调用](../../../raw/current/scripts/mainfunctions.lua#L388-L404) [注册](../../../raw/current/scripts/modutil.lua#L593-L598)

```lua
AddPrefabPostInit("evergreen", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local regrowth = inst.components.plantregrowth
    if regrowth == nil or regrowth.regrowthrate == nil then return end
    regrowth.regrowthrate = regrowth.regrowthrate * 0.5
    regrowth:ResetGrowthTime()
end)
```

`0.5` 缩短内部时钟的基础等待，随机范围仍为新值 ±20%；不改本株阶段速度、季节倍率、落点和密度。新实例本轮即重置；已有存档恢复在 PostInit 之后，会覆盖本轮到期值，后续 Reset 才用新 rate。停用后构造配置恢复原值，但已生成后代与已存剩余量不回滚。[存档恢复顺序](../../../raw/current/scripts/mainfunctions.lua#L485-L503) [组件恢复](../../../raw/current/scripts/components/plantregrowth.lua#L252-L254)

原版阔叶树则配置 acorn_sapling 和 deciduoustree 标签，种类时钟只在春季前进；换母株须同时核验其倍率、原始区域密度与计数标签，不能仅替换示例 Hook 名。即便基础等待减半，冬季停滞、多个桶轮询、空间被占或密度达标仍可阻止生成。[阔叶树配置](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1358-L1361) [春季条件](../../../raw/current/scripts/components/plantregrowth.lua#L33-L35)

## 常见陷阱与限制

- **给新植物挂组件并设 rate 就能复制原版繁殖**：还缺种类时钟倍率、世界原始密度、母株计数标签和有效后代。缺倍率可能报错，缺区域密度则注销；本页不提供自定义种类／世界生成密度注册教程。
- **反复 SetRegrowthRate 加速**：同一组件可能重复进入共享桶，正常一次注销不足以清重复项；已有对象改字段并按需 Reset，保留原初次登记流程。见接口与注销实现。
- **换 product 或开 skip 就绕过地图限制**：半径仍基于母株，skip 只绕过 CanPlantAtPoint；其余过滤和目标区域密度仍生效。生成失败还会消耗本轮等待，不能用成功返回值判断产出。
- **母株移动后自动重新评估区域**：fiveradius 缓存不随移动重算，缺密度时还会注销。此组件用于原位植被，移动植物需另设计清理／重建策略，不直接反复注册补救。[缓存与注销](../../../raw/current/scripts/components/plantregrowth.lua#L219-L225)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 3 处（mainfunctions／deciduoustrees 行号迁移，区间文本逐段比对一致）；结论修订：无，仅锚点迁移。
- 依据：全局枢纽 hunk 地图（entityscript／mods／mainfunctions）＋变更清单命中 mainfunctions.lua、prefabs/deciduoustrees.lua。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖种类时钟／桶调度、首次配置与重复登记风险、半径／选点／密度、生成与存读／移除、常青树和阔叶树必要工厂及示例。未审校全部树种、世界密度生成链或自定义母株迁移。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整 modmain 例通过已有 Full Moon Lua 5.1 syntax-only，不证明生态再生的游戏效果。
- 游戏运行：未运行；冬夏／春季、世界再生关闭、满密度／无落点、旧地图缺 generated 数据、存读计时及其他模组叠加待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。