---
title: "lunarhailmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月雹风暴期间按玩家调度月玻璃碎块坠落：按地表拓扑节点选掉落物、警告阴影、落地伤害与遮挡豁免；当前版本已弃用且无安装点。"
aliases: ["lunarhailmanager", "月雹", "lunar hail", "月玻璃碎块", "lunarhail debris"]
tags: [component, lifecycle, loot]
dst_build: "steam-buildid:26230496"
sources: ["components/lunarhailmanager.lua", "prefabs/forest.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lunarhailmanager

## 快速判断

- 职责与适用：世界级组件，月雹（`islunarhailing` 世界状态）期间为每个玩家按间隔在其附近调度碎块（moonglass／moonglass_charged）坠落并结算落地效果。**当前版本没有任何原版安装点**：森林 master_postinit 中的安装行已被注释并注明 deprecated，本页接口按组件现存代码记录。[弃用注释](../../../raw/current/scripts/prefabs/forest.lua#L711
- 优先使用：替换掉落表用 `SetDebris(data)`，按拓扑节点标签扩展用 `SetTagDebris(tag, data)`；这是组件仅有的两个公开方法。[公开接口](../../../raw/current/scripts/components/lunarhailmanager.lua#L410-L420)
- 重要副作用：碎块生成时 `persists=false` 且拾取被临时禁用，落地后按 `LUNARHAIL_DEBRIS_KEEP_CHANCE` 保留并恢复拾取；坠落对玩家结算 `LUNARHAIL_DEBRIS_DAMAGE`（30）combat 伤害，另会压 stressed 农作物、弹开未持久化的月雹碎块掉落物。[落地分支](../../../raw/current/scripts/components/lunarhailmanager.lua#L158-L257)
- 修改边界：全部调度与结算函数仅在 mastersim 定义（客户端为 nil）；风暴本身的启停与雹量在 weather 侧，本组件只消费 `islunarhailing` 与 `lunarhaillevel` 状态。[主模拟分支](../../../raw/current/scripts/components/lunarhailmanager.lua#L48-L51) [头注释](../../../raw/current/scripts/components/lunarhailmanager.lua#L1-L3)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetDebris(data)` | table（master 限定） | 整体替换默认掉落表 `{ {weight=2.5, loot={"moonglass"}}, {weight=1, loot={"moonglass_charged"}} }`；权重随机后取 loot 中随机一项。[替换](../../../raw/current/scripts/components/lunarhailmanager.lua#L410-L414) [默认表](../../../raw/current/scripts/components/lunarhailmanager.lua#L31-L35) |
| `SetTagDebris(tag, data)` | table（master 限定） | 按世界拓扑节点的 tags 数组匹配扩展掉落表（可设空表表示该区域不掉默认物）；无命中标签时回落默认表。[标签掉落](../../../raw/current/scripts/components/lunarhailmanager.lua#L53-L100) |
| `ToggleLunarHail`（内部，WatchWorldState） | bool | 监听 `islunarhailing` 世界状态：开启时为所有活跃玩家排程，关闭时取消全部任务；玩家加入/离开由 `ms_playerjoined`／`ms_playerleft` 维护。[启停](../../../raw/current/scripts/components/lunarhailmanager.lua#L388-L404) [监听](../../../raw/current/scripts/components/lunarhailmanager.lua#L427-L432) |
| 排程间隔（内部） | Lerp 计算 | 下次掉落延迟 = Lerp(`RATE_MIN`, `RATE_MAX`, 1-sin(p·π)) × 附近玩家数，p 为 `lunarhaillevel/100`；即雹量峰值间隔最短、玩家越多间隔越长。[间隔](../../../raw/current/scripts/components/lunarhailmanager.lua#L302-L311) [常量](../../../raw/current/scripts/tuning.lua#L7888-L7894) |
| 落点与遮挡（内部） | — | 玩家位置 60 半径内随机扇形取点；点上有 shadecanopy／shadecanopysmall（树冠）或雨穹顶则跳过本次掉落。[取点](../../../raw/current/scripts/components/lunarhailmanager.lua#L322-L344) [遮挡豁免](../../../raw/current/scripts/components/lunarhailmanager.lua#L313-L320) |
| `SpawnDebris(spawn_point, override_prefab?)`（内部） | — | 按落点拓扑节点选掉落表，35 高度投放，随机缩放 0.8～1.0、50% 旋转，配 warningshadow 落点预警影子并挂帧级落地检测任务。[投放](../../../raw/current/scripts/components/lunarhailmanager.lua#L265-L300) |
| 落地结算（内部） | — | 只对玩家结算伤害（ sheltered 或装备 `lunarhailprotection` 豁免）；对 farmplantstress 设 happiness 压力；对非持久化的 lunarhaildebris 掉落物 Launch 弹开；之后按概率保留或弹跳后碎裂为 mining fx。[命中条件](../../../raw/current/scripts/components/lunarhailmanager.lua#L136-L156) [落地分支](../../../raw/current/scripts/components/lunarhailmanager.lua#L158-L257) |
| 存档 | 无 OnSave/OnLoad | 组件不持久化；碎块落地前不进存档（persists=false），落地保留后才恢复 persists。[碎块持久化](../../../raw/current/scripts/components/lunarhailmanager.lua#L131-L134) [无存读](../../../raw/current/scripts/components/lunarhailmanager.lua#L410-L433) |

## 典型接入

### 重新启用月雹碎块或替换掉落物（模组侧）

- 前提与选择：组件代码完整但无安装点；想启用需自行 `inst:AddComponent("lunarhailmanager")` 到 forest（或自定义世界）master_postinit——组件自会监听世界状态与玩家事件。月雹风暴本身由 weather/月雹世界状态驱动，启用组件不会制造风暴。[弃用位置](../../../raw/current/scripts/prefabs/forest.lua#L711
- 生效与撤销：islunarhailing 为真时自动开始排程；用 `SetDebris`/`SetTagDebris` 在组件安装后立即配置掉落表。移除组件不会取消已排的玩家级任务（排程任务挂在玩家实体上，离开世界时清理）。[排程归属](../../../raw/current/scripts/components/lunarhailmanager.lua#L346-L353)
- 原版依据：默认掉落 moonglass/moonglass_charged 与伤害、保留概率等常量都取自 tuning 的 LUNARHAIL 段；掉落间隔算法注释声明与 weather 的 CalculateLunarHailRate 同型。[常量](../../../raw/current/scripts/tuning.lua#L7888-L7894)

```lua
-- 服务器端：给森林世界重新挂载本组件并自定义掉落（组件当前无原版安装点）
AddPrefabPostInit("forest", function(inst)
    if not TheWorld.ismastersim or inst.components.lunarhailmanager ~= nil then
        return
    end
    inst:AddComponent("lunarhailmanager")
    inst.components.lunarhailmanager:SetDebris({
        { weight = 2.5, loot = { "moonglass" } },
        { weight = 1,   loot = { "moonglass_charged" } },
    })
end)
```

- 调整与验证：loot 里的 prefab 必须真实存在（SpawnPrefab 失败则该次掉落静默跳过）；投放在客户端无对应逻辑，纯 master 行为。[生成防御](../../../raw/current/scripts/components/lunarhailmanager.lua#L269-L298)

## 常见陷阱与限制

- **组件已弃用、原版无消费者：** 唯一相关行是 forest.lua 的注释掉安装行；其"月雹掉落物"体验在当前版本由其它路径承担。判断月雹行为时不要引用本组件为现行实现。[弃用注释](../../../raw/current/scripts/prefabs/forest.lua#L711
- **伤害只打玩家：** `DebrisCanHitTarget` 显式排除非玩家（FIXME 注释注明是临时的生物保护）；别指望碎块砸死 boss 或生物。[命中条件](../../../raw/current/scripts/components/lunarhailmanager.lua#L136-L143)
- **掉落表按拓扑节点 tags 匹配是"并集"：** 多个命中标签的表会被拼接后统一按权重抽取，不是按标签轮流；想要区域专属掉落需保证标签独占或权衡权重。[拼接](../../../raw/current/scripts/components/lunarhailmanager.lua#L62-L69)
- **保留碎块的存活期只有 5 秒：** 落地保留分支挂 `LUNARHAIL_DEBRIS_LIFETIME(5s)` 的到期碎裂任务，被拾取（enterlimbo）会取消任务并把 persists 置真；把保留碎块当持久地形会失望。[到期](../../../raw/current/scripts/components/lunarhailmanager.lua#L203-L229) [拾取恢复](../../../raw/current/scripts/components/lunarhailmanager.lua#L210-L216)
- **排程任务挂在玩家实体上：** 玩家离开世界由 OnPlayerLeft 取消；强制移除玩家实体（非正常 left 事件）可能留下已失效任务，重启用依赖下一次 islunarhailing 切换。[玩家维护](../../../raw/current/scripts/components/lunarhailmanager.lua#L366-L386)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 lunarhailmanager.lua，核验掉落表与标签扩展、排程间隔公式、落点与遮挡豁免、投放与落地结算全分支、无存读；核对 forest 弃用注释与 LUNARHAIL tuning 段。weather 侧的月雹风暴实现不在本页范围。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；重挂载组件与自定义掉落未实测。

相关：[[reference/components/lunarhailbuildup]]、[[reference/components/lunarhailbirdsoundmanager]]。

## 相关链接

- [[reference/components/lunarhailbuildup]]：月雹积累值侧组件。
- [[reference/components/lunarhailbirdsoundmanager]]：月雹鸟声效档位同步。
