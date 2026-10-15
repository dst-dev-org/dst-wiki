---
title: "岩甲 Boss 剪击冲刺特效与枯萎变体（rocky_snip_fx、rocky_dash_fx、rocky_boss_shadow、worm_boss_dirt_ground_fx）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "Boss 战表现的 4 个登记：石虾 Boss 的 SG 生成攻击 FX（snip 剪击弧与 dash 起跳残影，均为宿主驱动的纯视觉、伤害留在宿主状态机）、其枯萎变体本体（完整 Boss 实体，本页只记客户端火层与 netvar 淡入淡出层）与洞穴蠕虫 Boss 的地面裂纹贴片（宿主单槽引用管理清理）。"
aliases: ["rocky_snip_fx", "rocky_dash_fx", "rocky_boss_shadow", "worm_boss_dirt_ground_fx", "剪击特效", "冲刺残影", "枯萎岩石大师", "地面裂纹", "Boss 战表现 FX"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/rocky_boss.lua", "stategraphs/SGrocky.lua", "stategraphs/commonstates.lua", "components/corruption.lua", "components/rockybossspawner.lua", "prefabs/worm_boss.lua", "prefabs/worm_boss_util.lua", "componentutil.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 岩甲 Boss 剪击冲刺特效与枯萎变体（rocky_snip_fx、rocky_dash_fx、rocky_boss_shadow、worm_boss_dirt_ground_fx）

## 快速判断

- 职责与适用：本页收拢两个 Boss 工厂文件里的 **4 个登记**，按载体分三类：`rocky_snip_fx`／`rocky_dash_fx` 是 `SGrocky` 在攻击帧上生成的纯视觉 FX（贴图弧线＋屏幕空间线条失真）；`rocky_boss_shadow` 是 `rocky_boss` 被暗影杖腐化后的**完整 Boss 变体**（生命／平面伤害／掉落俱全，不是 FX——本页只记它的客户端火焰跟随层与 netvar 淡入淡出层）；`worm_boss_dirt_ground_fx` 是洞穴蠕虫 Boss 潜地移动时铺在目标点的地面裂纹贴片（背景层循环动画＋生成音）。[snip 注册](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1166)、[dash 注册](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1167)、[shadow 注册](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1168)、[裂纹注册](../../../raw/current/scripts/prefabs/worm_boss.lua#L1678)
- 本体辨析：`rocky_boss`（岩石大师）与 `worm_boss`（巨大洞穴蠕虫）本体是 Boss 玩法实体，不在本页覆盖；`rocky_boss` 由 `rockybossspawner` 把普通石虾 `ReplacePrefab` 升级而来（[升级](../../../raw/current/scripts/components/rockybossspawner.lua#L70-L77)），战斗规则不展开。
- 重要副作用：`rocky_dash_fx` **没有自删路径**，靠宿主 `dash_attack` 状态 `onexit` 移除；两个攻击 FX 的伤害由宿主 SG 的 `DoAOEAttack` 结算，FX 实体自身零组件；裂纹贴片靠宿主的 `new_crack` 单槽引用清理，自身无移除钩子。四者均 `persists=false`（裂纹、snip、dash）或为持久 Boss 实体（shadow 本体）。
- 修改边界：只覆盖 4 个登记的工厂、生成点、父挂／跟随与清理；`rocky_boss` 的战斗状态机全貌、蠕虫 Boss 的分段移动算法、屏幕失真着色器实现与动画包内容不展开。
- 名称：宿主官方名 `STRINGS.NAMES.ROCKY_BOSS`＝"岩石大师"（[po](../../../raw/current/scripts/languages/chinese_s.po#L348684-L348687)）、`STRINGS.NAMES.ROCKY_BOSS_SHADOW`＝"枯萎岩石大师"（[po](../../../raw/current/scripts/languages/chinese_s.po#L348694-L348697)）、`STRINGS.NAMES.WORM_BOSS`＝"巨大洞穴蠕虫"（[po](../../../raw/current/scripts/languages/chinese_s.po#L352459-L352462)）；`ROCKY_SNIP_FX`／`ROCKY_DASH_FX`／`WORM_BOSS_DIRT_GROUND_FX` 无 `STRINGS.NAMES.*` 键，标题中的"剪击特效""冲刺残影""地面裂纹"是已核验的效果描述，**非官方译名**。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `rocky_snip_fx` | `snipfn`：Transform／AnimState／Network → `NOCLICK`＋`FX` → bank `rocky`、build `rocky_boss`、anim `fxheight`、缩放 1.4 → 非专用端挂 `updatelooper`＋`snip_PostUpdate` → Pristine → 主模拟 `persists=false`＋`animover` 自删 | 联网动画实体；客户端 PostUpdate 再造一个**非联网** `CreateLineFx` 子体（`snip_line_fx` 动画、贴地、按 build 分支加 Bloom 或 LightOverride）并推屏幕空间线条失真。[工厂](../../../raw/current/scripts/prefabs/rocky_boss.lua#L994-L1026)、[PostUpdate](../../../raw/current/scripts/prefabs/rocky_boss.lua#L941-L992)、[线条子体](../../../raw/current/scripts/prefabs/rocky_boss.lua#L889-L916) |
| `rocky_dash_fx` | `dashfn`：同壳（bank `rocky`、build `rocky_boss`、anim `fxheight`）→ 额外 `inst.target = net_entity(GUID, "rocky_dash_fx.target")` → 非专用端 `dash_PostUpdate` → 主模拟 `persists=false`（**无自删**） | 起跳点残影：PostUpdate 读 target（宿主）实时位置，在目标点再造 `CreateDashLine` 本地实体（`fxheight`＋子级 `dash_line_fx` 线条），0.4 秒内按剩余时间淡出并推失真。[工厂](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1128-L1161)、[PostUpdate](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1068-L1126)、[本地残影](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1030-L1055) |
| `rocky_boss_shadow` | `shadowfn = commonfn("rocky_boss_shadow_build", shadow_common_postinit, shadow_master_postinit)`：与 `rocky_boss` 共用 Boss 骨架（combat／health／physics），换 build、加 `shadowthrall`／`shadow_aligned`／`epic` 标签、`fade` netvar；非专用端造双肩火层 | **混合载体**：本体是有生命／平面伤害／掉落表的玩法 Boss；FX 侧只有两束 `CreateFlameLoop` 肩火（非联网、跟随肩符号）与淡入淡出层。[骨架](../../../raw/current/scripts/prefabs/rocky_boss.lua#L499-L530)、[common_postinit](../../../raw/current/scripts/prefabs/rocky_boss.lua#L819-L855)、[master_postinit](../../../raw/current/scripts/prefabs/rocky_boss.lua#L857-L885)、[注册](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1168) |
| `worm_boss_dirt_ground_fx` | `dirt_ground_fx_fn`：Transform／AnimState／SoundEmitter／Network → `FX`＋`NOCLICK` → bank／build `worm_boss`、`ground_rumble` 单播接 `ground_rumble_loop` 循环、贴地＋`LAYER_BACKGROUND`＋排序 3 → 主模拟播一次 `rifts4/worm_boss/ground_crack`、`persists=false` | 地面裂纹贴片；普通蠕虫与枯萎蠕虫共用同一普通 build 视觉；无自删，靠宿主引用移除。[工厂](../../../raw/current/scripts/prefabs/worm_boss.lua#L1420-L1450)、[注册](../../../raw/current/scripts/prefabs/worm_boss.lua#L1678) |

## 关联机制导航

### 生成点与宿主调用

| 生成点 | 涉及注册 | 端、条件与调用 | 证据 |
|---|---|---|---|
| `SGrocky` `snip_attack` 帧 15 | `rocky_snip_fx` | 主模拟；位于宿主朝向前方 `2.8×1.4` 处、按宿主朝向旋转；同帧 `DoAOEAttack` 结算伤害 | [帧事件](../../../raw/current/scripts/stategraphs/SGrocky.lua#L1033-L1064) |
| `SGrocky` `dash_attack` 帧 17 | `rocky_snip_fx` | 主模拟；冲刺收尾剪击，前方 `3.6×1.4` 处，同帧 AOE；宿主若为枯萎变体则先 `fx.AnimState:SetBuild("rocky_boss_shadow_build")` | [帧事件](../../../raw/current/scripts/stategraphs/SGrocky.lua#L1318-L1349) |
| `SGrocky` `dash_attack` 帧 12 | `rocky_dash_fx` | 主模拟；生成在冲刺起点 `tppos`、按宿主朝向旋转，`fx.target:set(inst)` 回指宿主，并重置 `dashcd` 计时 | [帧事件](../../../raw/current/scripts/stategraphs/SGrocky.lua#L1251-L1285) |
| 腐化变身 `stalker_corruption_pre` | `rocky_boss_shadow` | 主模拟；国王魔杖的 `corruption` 组件对可腐化目标推 `stalker_corruption_stun` → 通用状态 `animover` 时 `SpawnPrefab(inst.prefab.."_shadow")` 动态拼名生成、复制位置朝向，原体 `Remove`——**全库唯一生成方式是动态拼名** | [corruption 触发](../../../raw/current/scripts/components/corruption.lua#L97-L107)、[可腐化判定](../../../raw/current/scripts/componentutil.lua#L2473-L2477)、[通用状态生成](../../../raw/current/scripts/stategraphs/commonstates.lua#L3053-L3064)、[SGrocky 装配](../../../raw/current/scripts/stategraphs/SGrocky.lua#L1513-L1537) |
| `worm_boss_util` `Internal_CreateChunk` | `worm_boss_dirt_ground_fx` | 主模拟；蠕虫潜地规划新落点后延时生成：先铺裂纹，再 `CreateNewChunk` 出土丘 | [生成](../../../raw/current/scripts/prefabs/worm_boss_util.lua#L782-L792) |
| `worm_boss` `Dirt_OnAnimOver`（`dirt_emerge_loop_pre` 分支） | `worm_boss_dirt_ground_fx` | 主模拟；首个土丘播完 `dirt_emerge_loop_pre` 后 2 秒，在土丘位置再铺一枚裂纹，4 秒后钻头 | [分支](../../../raw/current/scripts/prefabs/worm_boss.lua#L1184-L1193)、[土丘状态选择](../../../raw/current/scripts/prefabs/worm_boss_util.lua#L101-L118) |

生成方式对照：`rocky_boss` 升级走 `ReplacePrefab`（保留 herd 成员资格），腐化走动态拼名 `SpawnPrefab`；两者都发生在主模拟。

### 层、跟随与同步

| 对象 | 本组登记如何使用 | 条件与边界 | 证据 |
|---|---|---|---|
| 屏幕空间线条失真 | `snip_PostUpdate`／`dash_PostUpdate` 用 `GetScreenPos` 把宿主与目标点的世界线段投到屏幕，`PushLineDistortion` 按 0.5s／0.4s 时长平方衰减强度；线段不出屏则不推 | 只在非专用端 updatelooper 里跑；成功注册失真后挂 `OnRemoveEntity` 保证清除，`ClearLineDistortion` 收尾 | [snip 失真](../../../raw/current/scripts/prefabs/rocky_boss.lua#L950-L991)、[dash 失真](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1084-L1124) |
| netvar `rocky_dash_fx.target` | 唯一自定义 netvar：宿主把自身 `set` 进去，客户端 PostUpdate 读 `inst.target:value()` 取冲刺终点 | 内部字段；target 为空时 PostUpdate 只等动画结束 | [netvar](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1145)、[读取](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1069-L1082) |
| 枯萎 Boss 双肩火层 | 非专用端两束 `CreateFlameLoop`（非联网、`FX` 标签、`flame_fx` 循环）`SetParent`＋`FollowSymbol(shoulder_L/R)`，随机错帧；登记进 `highlightchildren`，`colouraddersync` 的染色与淡入淡出都同步写到火层 | 专用服务器不存在火层；火层帧偏移为本地随机，各端观感略异 | [火层](../../../raw/current/scripts/prefabs/rocky_boss.lua#L832-L850)、[CreateFlameLoop](../../../raw/current/scripts/prefabs/rocky_boss.lua#L289-L308) |
| `rocky_boss_shadow.fade` 淡入淡出 | `net_smallbyte`（10–18 淡出、30–42 淡入两段编码），宿主导出 `StartFadeIn`／`StartFadeOut`／`ResetFade`／`IsFading`；客户端听 `fadedirty` 插值 `OverrideMultColour` 并同步到火层 | 当前唯一消费者是 `dash_attack` 连冲：进冲未淡完则补淡入、连冲前淡出、退出复位；淡入期间宿主获得无敌等状态标签 | [netvar](../../../raw/current/scripts/prefabs/rocky_boss.lua#L827)、[淡出](../../../raw/current/scripts/prefabs/rocky_boss.lua#L748-L761)、[淡入](../../../raw/current/scripts/prefabs/rocky_boss.lua#L763-L779)、[SG 消费](../../../raw/current/scripts/stategraphs/SGrocky.lua#L1120-L1126) |
| 裂纹贴片层 | `LAYER_BACKGROUND`＋排序 3 贴地，`ground_rumble_loop` 循环；音效只在主模拟 PlaySound（随联网实体广播） | 枯萎蠕虫的土丘换了 build（`worm_boss_shadow_build`），但裂纹工厂写死普通 `worm_boss` build，视觉不变 | [工厂](../../../raw/current/scripts/prefabs/worm_boss.lua#L1428-L1447)、[土丘 build](../../../raw/current/scripts/prefabs/worm_boss.lua#L1654-L1661) |

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `rocky_snip_fx` | 主模拟 `ListenForEvent("animover", inst.Remove)` 自删；客户端本地线条子体在 0.5s 后随 PostUpdate 收尾（失真清除、PostUpdate 注销） | 联网壳自删、本地子体自管，两条路径独立。[自删](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1022-L1023)、[收尾](../../../raw/current/scripts/prefabs/rocky_boss.lua#L987-L991) |
| `rocky_dash_fx` | **无自删**：宿主 `dash_attack` 的 `onexit` 里 `dashfx:Remove()`；本地残影实体与失真由 PostUpdate 0.4s 收尾或 `dash_OnRemoveEntity` 兜底 | 若宿主在状态外被移除，网络壳随宿主实体消亡前的 onexit 仍是唯一显式移除点。[onexit](../../../raw/current/scripts/stategraphs/SGrocky.lua#L1405-L1408)、[兜底](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1057-L1066) |
| `rocky_boss_shadow` | 持久 Boss 实体，正常走尸体／战利品流程；火层是子级本地实体随本体回收；淡入淡出在状态退出时 `ResetFade` 复位 | 本体可存档；火层与 fade 层不存档。[复位](../../../raw/current/scripts/stategraphs/SGrocky.lua#L1419-L1421) |
| `worm_boss_dirt_ground_fx` | 全部由宿主管理：`EmergeHead`（钻头）先 `new_crack:Remove()`；蠕虫 `OnDeath`／`OnRemoveEntity` 也移除引用 | `new_crack` 是蠕虫身上的**单槽引用**——新裂纹覆盖旧引用；死亡路径的 instant 建 chunk 不生成裂纹。[钻头移除](../../../raw/current/scripts/prefabs/worm_boss_util.lua#L537-L542)、[死亡移除](../../../raw/current/scripts/prefabs/worm_boss.lua#L238-L243)、[移除移除](../../../raw/current/scripts/prefabs/worm_boss.lua#L504-L519)、[instant 不铺](../../../raw/current/scripts/prefabs/worm_boss_util.lua#L929-L944) |

## 典型接入

### 复刻"SG 帧生成＋屏幕失真"的近战攻击 FX

- 前提与选择：攻击判定留在宿主 SG（`DoAOEAttack`），FX 只做两件事：联网动画壳（宿主 SG 定位、按需换 build）＋非专用端屏幕空间失真。适合大体型 Boss 的横扫／突进。
- 生效与撤销：壳按宿主朝向定位后无需再管（snip 自删）；dash 壳必须在宿主状态 `onexit` 显式 `Remove` 并把引用存进 `statemem`。失真要在 `OnRemoveEntity` 兜底清除，否则换场景后残留。[snip 生成点](../../../raw/current/scripts/stategraphs/SGrocky.lua#L1033-L1047)、[dash onexit](../../../raw/current/scripts/stategraphs/SGrocky.lua#L1405-L1408)
- 原版依据：[snipfn](../../../raw/current/scripts/prefabs/rocky_boss.lua#L994-L1026)、[dashfn](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1128-L1161)

```lua
local function spawn_snip_fx(host, dist)
    if not TheWorld.ismastersim then
        return
    end
    local fx = SpawnPrefab("rocky_snip_fx")
    local x, y, z = host.Transform:GetWorldPosition()
    local theta = host.Transform:GetRotation() * DEGREES
    fx.Transform:SetPosition(x + math.cos(theta) * dist, y, z - math.sin(theta) * dist)
    fx.Transform:SetRotation(host.Transform:GetRotation())
end
```

- 调整与验证：替换点是注册名与距离；枯萎变体换 build 需在 `SpawnPrefab` 之后、动画首帧前调 `SetBuild`。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：snip／dash FX 会造成伤害。** 两者零玩法组件；伤害与击中都由宿主 SG 的 `DoAOEAttack`／`FlashTargets` 在生成同帧结算，FX 只是后续 0.4–0.5s 的视觉余韵。[AOE](../../../raw/current/scripts/stategraphs/SGrocky.lua#L1063-L1064)、[dash AOE](../../../raw/current/scripts/stategraphs/SGrocky.lua#L1348-L1349)
- **边界：`rocky_dash_fx` 不自删。** 与 snip 不同，dash 工厂没有 `animover` 监听；唯一显式移除在宿主 `dash_attack` 的 `onexit`。直接 `SpawnPrefab` 而不接宿主状态机会留下永久残影壳。[对比](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1158-L1160)、[onexit](../../../raw/current/scripts/stategraphs/SGrocky.lua#L1405-L1408)
- **边界：失真与本地子体只在非专用端存在。** `updatelooper`＋PostUpdate 挂在 `not TheNet:IsDedicated()` 分支；专用服务器上两个 FX 只剩联网动画壳。[snip 分支](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1011-L1014)、[dash 分支](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1147-L1150)
- **边界：枯萎换 build 是生成后补写。** SG 先 `SpawnPrefab` 再按 `IsShadow`（`shadowthrall` 标签）`SetBuild`，工厂默认 build 恒为 `rocky_boss`；自定复刻若在客户端可见前漏掉这步，会闪一帧普通材质。[snip 补写](../../../raw/current/scripts/stategraphs/SGrocky.lua#L1043-L1045)、[dash 补写](../../../raw/current/scripts/stategraphs/SGrocky.lua#L1276-L1278)、[IsShadow](../../../raw/current/scripts/stategraphs/SGrocky.lua#L10-L12)
- **边界：`rocky_boss_shadow` 是玩法 Boss，不要当 FX 复用。** 它有 health／planardamage／掉落表／存档（OnSave 记 `endenraged`），血量 5000、平面伤害 20；本页只收它的火层与 fade 层两条表现链。[master_postinit](../../../raw/current/scripts/prefabs/rocky_boss.lua#L857-L883)、[掉落](../../../raw/current/scripts/prefabs/rocky_boss.lua#L47-L56)
- **边界：裂纹贴片的生命期挂在宿主单槽引用上。** 裂纹无自删、`persists=false`；读档时由管理器重建的 chunk 走 instant 路径不铺裂纹，蠕虫死亡／移除时只清当前引用。极端时序下引用被覆盖后的旧实体残留，本页未逐路径排除（未运行验证）。[工厂尾部](../../../raw/current/scripts/prefabs/worm_boss.lua#L1441-L1449)、[instant 路径](../../../raw/current/scripts/prefabs/worm_boss_util.lua#L941-L944)
- **边界：动态拼名是 `rocky_boss_shadow` 的唯一生成方式。** 全库没有字面 `"rocky_boss_shadow"` 的 `SpawnPrefab` 调用；依赖表、掉落表与 scrapbook 条目不生成实体。改名或加新变体时同步检查 `commonstates` 的 `inst.prefab.."_shadow"` 拼名。[动态拼名](../../../raw/current/scripts/stategraphs/commonstates.lua#L3053-L3064)、[反查记录](#验证记录)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/rocky_boss.lua`（四个工厂、火层／线条／残影子体工厂、淡入淡出函数族）与 `stategraphs/SGrocky.lua` 的 snip／dash 状态，按需读取 `commonstates.lua` 腐化状态族、`corruption.lua`、`rockybossspawner.lua`、`prefabs/worm_boss.lua`（dirt 工厂与 dirt 动画回调）、`prefabs/worm_boss_util.lua`（chunk 生成／钻头／尸体路径）。对 4 个完整注册名以 `grep -rn "注册名" --include=*.lua` 在 `raw/current/scripts` 全域反查：`rocky_snip_fx` 命中 4 处（注册、两份依赖表、SGrocky 两生成点）；`rocky_dash_fx` 命中 4 处（注册、两份依赖表、SGrocky 生成点、工厂内 netvar 字符串）；`rocky_boss_shadow` 命中 brain 注释、依赖表、掉落表、netvar 字符串、工厂注册与 scrapbook／皮肤数据——**无字面 SpawnPrefab**，生成点为 `commonstates.lua:3057` 的 `inst.prefab.."_shadow"` 动态拼名（`SGrocky` 经 `CommonStates.AddStalkerCorruptionStates` 装配，宿主注册名 `rocky_boss` 拼出目标名）；`worm_boss_dirt_ground_fx` 命中注册、两份依赖表与两个生成点（`worm_boss.lua:1189`、`worm_boss_util.lua:788`）。`TUNING.ROCKY_BOSS_SCALE=1.4`、`ROCKY_BOSS_DASH_RANGE=9`、`ROCKY_BOSS_DASH_CD=4`、`ROCKY_BOSS_SHADOW_HEALTH=5000`、`ROCKY_BOSS_SHADOW_PLANAR_DAMAGE=20` 已核对。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.ROCKY_BOSS`＝"岩石大师"、`STRINGS.NAMES.ROCKY_BOSS_SHADOW`＝"枯萎岩石大师"、`STRINGS.NAMES.WORM_BOSS`＝"巨大洞穴蠕虫"（锚点见"快速判断"）；`ROCKY_SNIP_FX`／`ROCKY_DASH_FX`／`WORM_BOSS_DIRT_GROUND_FX` 无 `STRINGS.NAMES.*` 键，标题使用效果描述。
- 分档说明：`size_class: standard`（4 注册分三类载体、两条 SG 生成链＋一条动态拼名链＋两条宿主管理清理，单页可承载），实测行数与字节见结构检查。
- 未覆盖：`rocky_boss` 战斗状态机其余状态与 `rocky_bossbrain` 决策树；蠕虫分段移动／进食／尖刺玩法与 `worm_boss_shadow` 变体链（其土丘换 build 但裂纹不变已记）；屏幕失真着色器与 `postprocesseffects.lua` 实现；`mirage` 残影池（非注册 prefab 的本地实体，未展开）；动画包内容与观感。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录（0 error 0 warning 后入库）。

## 相关链接

- [[reference/prefabs/warg_mutated_breath_fx|附身座狼喷焰与灼烬]]（同为 Boss SG 生成的攻击 FX 族；对照伤害归属写法）
- [[reference/prefabs/deerclops_icespike_fx|独眼巨鹿冰刺]]（同为池化地面攻击 FX）
- [[reference/prefabs/fossilized_break_fx|化石碎裂与化石化特效]]（同为本地代理＋联网壳的双层结构）
- 返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]
