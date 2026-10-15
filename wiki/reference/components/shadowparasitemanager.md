---
title: "shadowparasitemanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴世界暗影裂隙的寄生体波次管理器：裂隙入池积累波次，按权重宿主生成寄生体小队，拦截凝胶blob玩家并孵化被寄生玩家替身。"
aliases: ["shadowparasitemanager", "寄生体", "shadow parasite", "暗影寄生波次", "parasite mask"]
tags: [component, creature, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/shadowparasitemanager.lua", "prefabs/cave.lua", "components/gelblobspawner.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shadowparasitemanager

## 快速判断

- 职责与适用：洞穴世界组件（断言 mastersim），围绕暗影裂隙（riftspawner 的 SHADOW 亲和裂隙）调度寄生体（shadowthrall_parasite）与被寄生宿主小队；被寄生宿主由凝胶 blob 生成拦截（OverrideBlobSpawn）等路径触发。[安装](../../../raw/current/scripts/prefabs/cave.lua#L527 [断言](../../../raw/current/scripts/components/shadowparasitemanager.lua#L64
- 优先使用：模组想改宿主池改 `_WEIGHTED_HOSTS_TABLE`／`_HOST_CAN_SPAWN_TEST`（构造时显式暴露给 mods）；波次触发走 `BeginParasiteWave`，定点放寄生体用 `SpawnFloater(pos)`。[模组暴露](../../../raw/current/scripts/components/shadowparasitemanager.lua#L77-L78) [放浮体](../../../raw/current/scripts/components/shadowparasitemanager.lua#L284-L301)
- 重要副作用：宿主生成即被塞入并装备 `shadow_thrall_parasitehat` 面具；波次结算后对目标玩家推 5～8 只宿主并 SuggestTarget；被寄生玩家的骷髅会孵化出继承 userid 与皮肤的 `player_hosted` 替身。[面具](../../../raw/current/scripts/components/shadowparasitemanager.lua#L134-L141) [玩家替身](../../../raw/current/scripts/components/shadowparasitemanager.lua#L358-L376)
- 修改边界：一切波次要求暗影裂隙存在（`GetShadowRift()`）；宿主池在构造时按世界设置裁剪（兔子屋/蜘蛛巢/石蛤密度开关）；num_waves 与目标 userid 进存档，寄生体/浮体列表不存。[裁剪](../../../raw/current/scripts/components/shadowparasitemanager.lua#L95-L101) [存读](../../../raw/current/scripts/components/shadowparasitemanager.lua#L405-L443)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `_WEIGHTED_HOSTS_TABLE` | {prefab=权重} | 宿主概率：bunnyman 0.40、rocky 0.20、spider 0.20、spider_dropper 0.12、spider_warrior 0.08；构造时按 `HOST_CAN_SPAWN_TEST` 的世界设置（RABBITHOUSE_ENABLED 等）剔除不可用宿主。[权重表](../../../raw/current/scripts/components/shadowparasitemanager.lua#L7-L22) [裁剪](../../../raw/current/scripts/components/shadowparasitemanager.lua#L95-L101) |
| `OverrideBlobSpawn(player)` | → bool | gelblobspawner 生成 blob 前调用：有暗影裂隙、宿主池非空、num_waves>0 且通过运气掷骰（基数 `SHADOWTHRALL_PARASITE_BLOBOVERRIDE=0.2`）时返回 true——blob 被替换为寄生流程，玩家记入目标集并触发波次。[拦截](../../../raw/current/scripts/components/shadowparasitemanager.lua#L338-L356) [blob 侧](../../../raw/current/scripts/components/gelblobspawner.lua#L121-L124) |
| `OnRiftAddedToPool()`（ms_riftaddedtopool） | 内部 | 非加载期（非 POPULATING）每加入一个裂隙 num_waves+1；这是波次"预算"的唯一来源。[入池计数](../../../raw/current/scripts/components/shadowparasitemanager.lua#L208-L214) |
| `BeginParasiteWave()` | 内部 | num_waves-1 后二选一：裂隙处 `PLAYER_REVEAL_RADIUS` 内有玩家→在裂隙位置分批（≤5s 随机延迟）生成 `SHADOWTHRALL_PARASITE_WAVE_MIN(6)+random(6)` 个浮体；无人→直接对全部目标玩家各生成小队。[波次](../../../raw/current/scripts/components/shadowparasitemanager.lua#L313-L334) [常量](../../../raw/current/scripts/tuning.lua#L8552-L8553) |
| `SpawnParasiteWaveForPlayer(player, joining?)` | 内部 | 5+random(3) 只宿主，绕玩家 `PLAYER_CAMERA_SEE_DISTANCE+0~12` 距离的可走点分布；joining=true 时延迟 3 秒 SuggestTarget（换线玩家回归场景），否则立即仇恨玩家。[小队](../../../raw/current/scripts/components/shadowparasitemanager.lua#L179-L206) |
| `SpawnHostCreature()`／`AddParasiteToHost(host)` | 内部 | 按权重抽宿主 SpawnPrefab、塞面具、淡入；AddParasiteToHost 返回宿主与面具（供 mods）。[宿主](../../../raw/current/scripts/components/shadowparasitemanager.lua#L147-L165) |
| `SpawnHostedPlayer(inst)`／`ReviveHosted(inst)` | 内部 | 宿主化玩家的骷髅复活：生成 player_hosted、复制皮肤与 userid、播 intro（SGwilson 消费）；ReviveHosted 由各宿主 SG 在醒来时调用（脱战、停跟随、出特效）。[替身](../../../raw/current/scripts/components/shadowparasitemanager.lua#L358-L376) [SG 消费](../../../raw/current/scripts/stategraphs/SGspider.lua#L144-L145) |
| `StartTrackingParasite(parasite)`／`StopTrackingParasite(parasite)` | — | 登记寄生体并启动 5 秒一次的群体吟唱任务（SHADOWTHRALL_PARASITE_CHANT 随机台词）；全部移除后停任务。[追踪](../../../raw/current/scripts/components/shadowparasitemanager.lua#L251-L272) |
| `OnPlayerJoined/Left(player)` | 内部 | 加入：若 userid 在目标名单（离开时仍被标记）立即对其生成小队；离开：目标态转 userid 记录。跨会话由存档 targetuserids 恢复。[进出](../../../raw/current/scripts/components/shadowparasitemanager.lua#L105-L130) |
| `OnSave()`／`OnLoad(data)` | num_waves + targetuserids | 波次预算与被标记玩家跨存档保留；活跃寄生体/浮体不持久化。[存读](../../../raw/current/scripts/components/shadowparasitemanager.lua#L405-L443) |

## 典型接入

### 自定义宿主或强制触发寄生波次

- 前提与选择：需在洞穴世界且暗影裂隙已存在；自定义宿主要能装备物品（inventory 组件）并接受面具。模组在裂隙入池前后修改 `_WEIGHTED_HOSTS_TABLE` 即可改变宿主分布（构造裁剪只跑一次，之后新增键不会被世界设置剔除）。[权重暴露](../../../raw/current/scripts/components/shadowparasitemanager.lua#L77-L78)
- 生效与撤销：`BeginParasiteWave` 消耗一个 num_waves；无波次预算时模组可手动 `self.num_waves = self.num_waves + 1` 再触发。撤销只能移除生成的实体，目标集/吟唱任务随追踪清理。[预算](../../../raw/current/scripts/components/shadowparasitemanager.lua#L323
- 原版依据：gelblobspawner 是唯一官方波次入口（玩家挖凝胶 blob 的运气事件）；裂隙入池计数由 riftspawner 的 `ms_riftaddedtopool` 世界事件驱动。[blob 入口](../../../raw/current/scripts/components/gelblobspawner.lua#L121-L124) [裂隙源](../../../raw/current/scripts/components/riftspawner.lua#L80-L87)

```lua
-- 服务器端：把自定义生物加入宿主池（无世界设置剔除路径）
AddPrefabPostInit("cave", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst:DoTaskInTime(0, function()
        local mgr = inst.components.shadowparasitemanager
        if mgr ~= nil then
            mgr._WEIGHTED_HOSTS_TABLE.myhost = 0.3
        end
    end)
end)
```

- 调整与验证：宿主必须真的存在（SpawnPrefab 失败静默跳过该只）；GetDebugString 显示波次/寄生体/浮体/目标计数。[调试串](../../../raw/current/scripts/components/shadowparasitemanager.lua#L460-L468)

## 常见陷阱与限制

- **构造期裁剪之后新增的宿主不会被剔除：** `ApplyWorldSettings` 只在构造跑一次；运行时往 `_WEIGHTED_HOSTS_TABLE` 加键会绕过世界设置开关，自行保证宿主可用性。[裁剪时机](../../../raw/current/scripts/components/shadowparasitemanager.lua#L95-L101)
- **波次预算只在裂隙"加入池"时增加：** 加载期（POPULATING）入池不计；裂隙被拆出池不返还预算。世界只有一条裂隙（MAXIMUM_RIFTS_COUNT=1）时，一整个裂隙生命周期只有一次预算。[加载豁免](../../../raw/current/scripts/components/shadowparasitemanager.lua#L209-L211)
- **浮体（floater）是中间态：** SpawnFloater 只在无玩家旁观裂隙时使用；浮体全部移除（OnFloaterRemoved 清空）才对目标玩家铺小队，中途新浮体不重置该等待。[浮体协议](../../../raw/current/scripts/components/shadowparasitemanager.lua#L276-L282)
- **目标玩家记录跨会话但玩家可能不再回来：** targetuserids 永久保留到该玩家再次加入并结算；模组清场需同时清 `_targetplayers` 与 `_targetuserids`。[存档字段](../../../raw/current/scripts/components/shadowparasitemanager.lua#L412-L424)
- **ReviveHosted 不清攻击者目标：** 源码 FIXME 注明周围敌人不会失去目标，宿主间可能互殴；依赖该行为的逻辑要自行处理。[FIXME](../../../raw/current/scripts/components/shadowparasitemanager.lua#L393

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 shadowparasitemanager.lua，核验宿主池与裁剪、blob 拦截、波次预算与两路生成、玩家替身/复活协议、吟唱任务与存读；核对 cave 安装、gelblobspawner 消费、SGwilson/各宿主 SG 消费与 tuning 常量。面具本体（shadow_thrall_parasitehat）的寄生计时未展开。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；波次节奏、宿主分布与替身孵化未实测。

相关：[[reference/components/riftspawner]]、[[reference/components/gelblobspawner]]。

## 相关链接

- [[reference/components/riftspawner]]：暗影裂隙的生成与入池事件源。
- [[reference/components/gelblobspawner]]：blob 生成拦截的调用方。
