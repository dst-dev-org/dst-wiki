---
title: "岩石碎裂音效代理（rock_break_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "只播一声岩石碎裂声的一次性实体：联网代理壳在服务端存活 1 秒转发位置，非专用机器本地生成仅含声音的实体播放后立即移除，无任何视觉。"
aliases: ["rock_break_fx", "碎裂音效", "岩石碎裂声", "rock break"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/rock_break_fx.lua", "prefabs/marblepillar.lua", "prefabs/spiderhole.lua", "prefabs/alterguardian_phase3dead.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 岩石碎裂音效代理（rock_break_fx）

## 快速判断

- 职责与适用：石头/大理石/凝固岩体被破坏时的**纯音效 FX**（无 AnimState）：位置代理模式——联网壳只带 Transform＋Network＋FX 标签，用于跨端转发位置；真正的声音实体只在各非专用机器本地出现一瞬。[代理壳](../../../raw/current/scripts/prefabs/rock_break_fx.lua#L15-L40)
- 优先使用：查"岩体碎掉的声响"选本页；生成入口是各矿物/岩石 workable 的破坏回调，SpawnPrefab 后仅设位置。
- 重要副作用：非专用机器 `DoTaskInTime(0)` 延迟一帧建本地声音实体（保证代理已定位），播放 `dontstarve/wilson/rock_break` 后当帧 Remove；服务端代理 `DoTaskInTime(1)` 自删。[声音实体](../../../raw/current/scripts/prefabs/rock_break_fx.lua#L1-L13)
- 修改边界：专用服务器不生成声音实体（`not TheNet:IsDedicated()` 判定）；无动画、无玩法组件、无存档。[判定](../../../raw/current/scripts/prefabs/rock_break_fx.lua#L23-L28)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `rock_break_fx` | fn：CreateEntity → Transform＋Network → FX 标签 → 非专用 DoTaskInTime(0, PlayRockBreak) → SetPristine → 主模拟侧 persists=false＋DoTaskInTime(1, Remove)；PlayRockBreak：从 proxy.GUID 取位（SetFromProxy）→ 建 Transform＋SoundEmitter 本地实体 → PlaySound → Remove | 单一注册；代理壳与声音实体分工见快速判断；注释明示延迟一帧是为定位完成或即将被移除的情况。[工厂](../../../raw/current/scripts/prefabs/rock_break_fx.lua#L15-L40)、[注册](../../../raw/current/scripts/prefabs/rock_break_fx.lua#L42) |

注册名无 NAMES 条目；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 模式：位置代理 | 声音需要实体承载、而非联网实体无法跨端定位——联网壳复制位置，各机本地出声 | 专用服务器只有壳；玩家机（含主机）各自建声音实体 | [SetFromProxy](../../../raw/current/scripts/prefabs/rock_break_fx.lua#L9) |
| 生成者：marblepillar | 破坏回调 `SpawnPrefab("rock_break_fx").Transform:SetPosition(pos)` → DropLoot → Remove 自身 | 主模拟 workable 回调；生成后只设位置、不管理生命周期 | [onworked](../../../raw/current/scripts/prefabs/marblepillar.lua#L20-L24) |
| 生成者：spiderhole | 蜘蛛洞岩壳（stalagmite 状）破坏时同模式生成 | 与惊吓特效（spooked 系）同点触发但互不相干 | [rock_onworked](../../../raw/current/scripts/prefabs/spiderhole.lua#L57-L62) |
| 生成者：其他已核验命中 | alterguardian_phase3dead 三阶段守卫死亡躯体 dead_onwork 破坏时与 collapse_big 同点生成（[L204](../../../raw/current/scripts/prefabs/alterguardian_phase3dead.lua#L204)）等矿物/岩体破坏点复用同一调用式 | 各生成点均为"SpawnPrefab＋定位"一行式，无参数可调 | grep 全库命中 spiderhole/tree_rocks/cave_entrance/cavein_boulder/cave_vents/grotto 系/marbleshrub 等 |
| 资源 | 无动画资产声明；音效 `dontstarve/wilson/rock_break` | 声音在本地实体上播放 | [PlaySound](../../../raw/current/scripts/prefabs/rock_break_fx.lua#L10) |

## 典型接入

### 在破坏回调里复用一次性碎裂声

- 前提与选择：任何主模拟侧的"岩体破坏"场景；无需资源声明外的配置，一行调用即可。
- 生效与撤销：代理壳 1 秒后自删、声音实体当帧自删，生成方完全不用清理。
- 原版依据：marblepillar 与 spiderhole 的 onworked 即标准调用式（[marblepillar](../../../raw/current/scripts/prefabs/marblepillar.lua#L20-L24)）。

```lua
local function onworked(inst, worker, workleft)
    if workleft <= 0 then
        local pos = inst:GetPosition()
        SpawnPrefab("rock_break_fx").Transform:SetPosition(pos:Get())
        inst:Remove()
    end
end
```

- 调整与验证：想换/加音效就不要复用本 prefab（声音硬编码、实体当帧移除，脚本层无法干预），自建 SoundEmitter 实体更合适。

## 常见陷阱与限制

- **误区：名字带 fx 就有视觉**：工厂没有 AnimState（[L15-L40](../../../raw/current/scripts/prefabs/rock_break_fx.lua#L15-L40)），只有一声响；要碎裂动画另找 FX。
- **边界：代理壳不是可操作的 FX 实体**：声音实体只在玩家机本地存在一瞬，服务端逻辑拿不到它；需要对该特效做延迟/修改时本 prefab 不适用。
- **边界：同帧生成又同帧移除会失去发声**：声音实体靠 `DoTaskInTime(0)` 延迟任务建立，代理壳被移除会连带取消未执行任务——照原版生成点"只定位、不手动 Remove"即可（壳自己 1 秒后删，[L37](../../../raw/current/scripts/prefabs/rock_break_fx.lua#L37)）。
- **边界：非联网分工**：壳联网（位置跨端一致），声音实体非联网（各机各响一声）；双端听到的是各自本地实体，时序上有同步位置保证、无声音同步包。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；43 行工厂全文（代理壳＋PlayRockBreak 全程）、marblepillar/spiderhole/alterguardian_phase3dead 生成点、其余命中点为 grep 行级确认、NAMES 检索（无命中）已核验；声音实体当帧移除后引擎是否完整播出音频、各生成点实际运行表现未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-light-index|灯光、火焰与音效代理]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页 sources 所引文件原始字节本轮全部未变（哈希等价批量沿用，无语义修订），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-3）。
