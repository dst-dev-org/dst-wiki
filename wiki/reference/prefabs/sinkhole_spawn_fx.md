---
title: 地陷尘爆与落石预警族（sinkhole_spawn_fx 等三族）
created: 2026-11-01
updated: 2026-11-01
type: prefab
summary: "蚁狮地陷袭击链上的 7 个纯表现 FX：地陷尘爆三名（陷坑塌陷/修复、巨兽落坑、翻地补尘）、地面预警小尘三名（表面端袭击倒计时）、洞穴落石碎屑（洞穴端同链警告）；玩法作用全部在生成者。"
aliases: ["sinkhole_spawn_fx_1", "sinkhole_spawn_fx_2", "sinkhole_spawn_fx_3", "sinkhole_warn_fx_1", "sinkhole_warn_fx_2", "sinkhole_warn_fx_3", "cavein_debris", "地陷尘爆", "落石预警", "洞穴落石碎屑", "蚁狮地陷特效", "塌陷尘土"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/antlion_sinkhole.lua", "prefabs/eyeofterror_sinkhole.lua", "stategraphs/SGeyeofterror.lua", "components/sinkholespawner.lua", "components/caveins.lua", "prefabs/cave.lua", "componentutil.lua", "components/terraformer.lua", "prefabs/pitchfork.lua", "prefabs/farm_plow.lua", "prefabs/fossilized_fx.lua", "prefabs/lavaarena_fossilizing.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 地陷尘爆与落石预警族（sinkhole_spawn_fx 等三族）

## 快速判断

- 职责与适用：`fx.lua` 通用表中蚁狮地陷袭击链相邻的 7 名纯表现 FX，分三个表现角色——`sinkhole_spawn_fx_1/2/3` 地面开裂尘爆（陷坑塌陷与修复、巨兽落坑、翻地补尘）、`sinkhole_warn_fx_1/2/3` 玩家位置预警小尘（仅表面端）、`cavein_debris` 洞穴端同链警告落石尘。spawn／warn 语义来自消费端调用窗口（warn 出现在袭击倒计时警告，spawn 出现在陷坑实体塌陷/修复），不是工厂协议字段；标题为已核验效果描述，非官方译名。公共协议（服务端代理→一帧后各端本地动画→animover 自删）见 [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂·关键流程]]。
- 优先使用：查"地陷/塌陷的地面尘土、蚁狮袭击前的小尘、洞穴警告碎石"读本页；袭击调度与洞穴执行归 [[reference/components/sinkholespawner]]、[[reference/components/caveins]]；新增或改动条目走 [[reference/mechanisms/generic-fx-factory#典型接入|通用 FX 工厂·典型接入]]。
- 重要副作用：warn 三名与 `cavein_debris` 的 fn 在各自本地实体加 SoundEmitter 播 `ground_break` 变参声（warn size=0.01、cavein size=0）；spawn 三名条目无声，地裂声由生成者在陷坑自身 SoundEmitter 播放。
- 修改边界：全部由主模拟 `SpawnPrefab` 生成后经网络代理分发，专服只有代理无本地表现；warn 仅在目标玩家与蚁狮同世界（表面）时本地生成，洞穴目标改由 caveins 在洞穴世界演示（见下表）；生成节奏由 TUNING.ANTLION_SINKHOLE 驱动。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `sinkhole_spawn_fx_1`／`_2`／`_3` | 通用工厂：bank/build `sinkhole_spawn_fx`，anim `idle1`/`idle2`/`idle3`；无 sound、无 fn、无 transform | 纯动画三变体；随机选名由各生成者自己做；[条目组](../../../raw/current/scripts/fx.lua#L1645-L1660) |
| `sinkhole_warn_fx_1`／`_2`／`_3` | 同上 bank/build/anim，加 `transform` 三轴 0.75 缩放与 fn | fn 在本地实体播 `ground_break` size=0.01（PlaySoundWithParams）；[条目组](../../../raw/current/scripts/fx.lua#L1663-L1684) |
| `cavein_debris` | bank/build `cavein_debris_fx`，anim `anim`，fn | fn 播 `ground_break` size=0；[条目](../../../raw/current/scripts/fx.lua#L1687-L1691) |

## 关联机制导航

| 类型／对象 | 本族如何被使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 消费者：`sinkholespawner`（表面警告） | `DoTargetWarning` 每次警告 tick 在目标玩家位置 ±3.5 随机偏移（y=0）落 `ceil((12−剩余警告数+1)/3.2)` 枚 warn fx（首轮 1 枚、随倒计时增多），伴随镜头摇晃；每 4 次让目标玩家喊话 | 仅主模拟；玩家与蚁狮同世界才本地生成——洞穴玩家位置为空，经跨分片事件由 caveins 表现 | [生成环](../../../raw/current/scripts/components/sinkholespawner.lua#L91-L103)；[[reference/components/sinkholespawner]]；节奏 [TUNING](../../../raw/current/scripts/tuning.lua#L3288-L3301) |
| 消费者：`caveins`（洞穴警告） | 收到跨分片 warn 后 `DoTargetWarning`：`CanPointHaveQuaker` 门禁＋镜头摇晃，`DoTargetDebris` 链式落 3~4 枚 `cavein_debris`（0.4~0.6s 间隔，spread 0.2 起每枚 +0.5 封顶 1；玩家 4 单位内直落玩家坐标） | 仅洞穴世界（cave.lua 安装、构造断言 mastersim）；攻击路径 `ms_miniquake` 只落 `cavein_boulder`，不出本 FX | [碎屑链](../../../raw/current/scripts/components/caveins.lua#L34-L66)；[[reference/components/caveins]]、[[reference/components/quaker]]；依赖声明 [cave.lua](../../../raw/current/scripts/prefabs/cave.lua#L76-L76) |
| 消费者：`antlion_sinkhole` 陷坑 | `SpawnFx`：中心 1 枚＋环 7 枚（半径 1.6 ±10% 抖动、y=0、scale 0.8 起 +0~0.2、偶数枚 x 轴镜像）；每级塌陷（3 级、1s 周期，`startcollapse` 事件驱动）与修复 timer 回调（scale 0.45，`IsAsleep` 不出）各一轮；`ground_break` size=(stage/3)² 在陷坑自身播放 | 服务端；塌陷还会摧毁/削弱可工作物并抛起物品，归陷坑页 | [SpawnFx](../../../raw/current/scripts/prefabs/antlion_sinkhole.lua#L36-L51)、[修复轮](../../../raw/current/scripts/prefabs/antlion_sinkhole.lua#L54-L70)、[塌陷轮](../../../raw/current/scripts/prefabs/antlion_sinkhole.lua#L106-L127)；实体页 Wiki 待建（源码入口 prefabs/antlion_sinkhole.lua） |
| 消费者：`eyeofterror_sinkhole`／`daywalker_sinkhole`／`bearger_sinkhole` | 同款 `SpawnFx`（scale=实体 scale 0.6/1/1，`ground_break` size=2）；`docollapse` 事件一轮、20s 修复 timer 一轮（scale 减半，`IsAsleep` 不出） | 服务端；docollapse 由 SGeyeofterror chomp 50 帧、SGdaywalker／SGbearger 落击推送 | [SpawnFx](../../../raw/current/scripts/prefabs/eyeofterror_sinkhole.lua#L25-L50)、[docollapse](../../../raw/current/scripts/prefabs/eyeofterror_sinkhole.lua#L91-L149)、[修复轮](../../../raw/current/scripts/prefabs/eyeofterror_sinkhole.lua#L53-L61)、[chomp 推送](../../../raw/current/scripts/stategraphs/SGeyeofterror.lua#L905-L910) |
| 消费者：SGeyeofterror chomp 25 帧 | 眼魔落地啃咬在落点直出中心 1 枚＋环 7 枚（scale 0.6±0.2），与陷坑生成同帧 | 仅 `IsVisualGroundAtPoint` 且不在船上；服务端 SG | [直出段](../../../raw/current/scripts/stategraphs/SGeyeofterror.lua#L875-L892) |
| 消费者：terraformer 翻地链 | `Terraform` 未设 `onterraformfn` 时走 `HandleDugGround`：原瓦片无对应草皮产物（GroundTiles.turf 映射缺失）则落 1 枚 spawn fx 于改地点 | 服务端；树内可达宿主仅 pitchfork；farm_plow 虽挂组件但设置了 `onterraformfn`（[farm_plow L232](../../../raw/current/scripts/prefabs/farm_plow.lua#L230-L232)），`Terraform` 永走该分支不进 `HandleDugGround`（[terraformer L28-L32](../../../raw/current/scripts/components/terraformer.lua#L28-L32)），不产出本 FX | [HandleDugGround](../../../raw/current/scripts/componentutil.lua#L31-L44)、[pitchfork](../../../raw/current/scripts/prefabs/pitchfork.lua#L79-L79) |
| 资源：anim `sinkhole_spawn_fx.zip`／`cavein_debris_fx.zip` | 两套动画资源（warn 与 spawn 共用前者）；条目 anim zip 由工厂按 build 自动声明（[prefabs/fx.lua](../../../raw/current/scripts/prefabs/fx.lua#L13-L15)），另两文件直接复用同 bank/build 播放：[fossilized_fx](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L63-L64)（碎裂代理）与 [lavaarena_fossilizing](../../../raw/current/scripts/prefabs/lavaarena_fossilizing.lua#L40-L41)（化石化），与本族注册无关 | antlion_sinkhole 等消费者声明的是 prefabs 名单非资产 | 依赖声明例 [antlion_sinkhole](../../../raw/current/scripts/prefabs/antlion_sinkhole.lua#L9-L17) |

## 典型接入

### 复用地陷尘爆的"中心＋环"图样

- 前提与选择：服务端代码（组件回调、SG timeline 等）内 `SpawnPrefab`；三名变体随机选名与镜像缩放是原版消费端逻辑，不在工厂协议里。
- 生效与撤销：每枚 fx 都是独立代理＋各端本地动画，animover 自删，生成者无需清理；地裂声按原版做法由调用方在自己实体上播，不靠 fx 条目。
- 原版依据：[antlion_sinkhole SpawnFx](../../../raw/current/scripts/prefabs/antlion_sinkhole.lua#L36-L51)；通用生成方式见 [[reference/mechanisms/generic-fx-factory#典型接入|通用 FX 工厂·典型接入]]。

```lua
local function SpawnGroundBreakFx(x, z, scale)
    SpawnPrefab("sinkhole_spawn_fx_" .. math.random(3)).Transform:SetPosition(x, 0, z)
    local theta = math.random() * TWOPI
    for i = 1, 7 do
        local dust = SpawnPrefab("sinkhole_spawn_fx_" .. math.random(3))
        dust.Transform:SetPosition(x + math.cos(theta) * 1.6, 0, z - math.sin(theta) * 1.6)
        local s = scale + math.random() * .2
        dust.Transform:SetScale(i % 2 == 0 and -s or s, s, s)  -- 偶数枚 x 轴镜像
        theta = theta + TWOPI / 7
    end
end
```

- 调整与验证：环半径/数量/缩放为原版消费端取值的简化复述（原版带 ±10% 位置抖动），坐标单位为世界单位；接入后需自验证各端表现，专服无动画为预期。

## 常见陷阱与限制

- **误区：`cavein_debris` 是洞穴地震的落石**：`quaker` 地震落下的实体是 `cavein_boulder`（`ms_miniquake` 消费入口 [quaker.lua](../../../raw/current/scripts/components/quaker.lua#L743-L743)）；`cavein_debris` 全 scripts 生成点仅 caveins 警告链一处（检索范围见验证记录，另含 cave.lua 仅依赖声明），它是蚁狮对洞穴玩家的警告表现。
- **边界：warn 与 spawn 视觉同源**：两族共用 `sinkhole_spawn_fx` bank/build 与同组 idle 动画，差异只有 0.75 缩放与自带声；不能按外观区分语义，语义在生成窗口。
- **边界：同一次眼魔 chomp 出两轮尘爆**：25 帧直出一段、50 帧 `docollapse` 再经 `DoCollapse` 一段，两轮都生效，不是互斥分支。
- **边界：修复轮在实体休眠时不出**：antlion_sinkhole 与 eyeofterror_sinkhole 的修复 timer 回调都带 `IsAsleep` 门禁；塌陷轮无此门禁。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。fx.lua 七条目逐项核对 bank/build/anim/transform/fn；生成点全量反查 `rg "sinkhole_spawn_fx|sinkhole_warn_fx|cavein_debris"` 于 raw/current/scripts 全域（根级、prefabs/、components/、stategraphs/、map/，含 `..math.random(3)` 动态拼名），命中生成点与仅依赖声明处均已列入上表，另核得 fossilized_fx／lavaarena_fossilizing 两文件复用同 bank/build（已记入资源行）；消费链读至触发窗口、落点（全落点式，无父挂）与端。`glass_fx`／`erode_ash`／`sleepbomb_burst` 另见 [[reference/prefabs/glass_fx]]。
- 语言文件：fx 条目无 `nameoverride`／`description` 字段，不可检视，`languages/chinese_s.po` 无对应显示名键；本页标题为非官方效果描述。
- 游戏运行：未运行；validation 维持 source-reviewed。
- 结构检查：validate_page.py（--format --check-indexes）与 validate_source_links.py --strict 已执行，结果见交付报告。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]。
