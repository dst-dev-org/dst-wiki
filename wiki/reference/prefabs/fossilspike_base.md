---
title: "化石骨刺地面基座层（fossilspike_base、fossilspike2_base）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "Stalker 两代化石尖刺攻击的地面基座动画实体：由尖刺本体在起刺/砸地回调里生成并引擎父挂，自身零玩法组件、无自删，生命期随宿主；本体 fossilspike/fossilspike2 为带 combat 的混合结构，页内辨析不计 FX 覆盖。"
aliases: ["fossilspike_base", "fossilspike2_base", "fossilspike", "fossilspike2", "化石骨刺", "化石笼子", "骨刺地面基座", "Stalker 尖刺 FX"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fossil_spike.lua", "prefabs/fossil_spike2.lua", "prefabs/stalker.lua", "stategraphs/SGstalker.lua", "brains/stalkerbrain.lua", "simutil.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 化石骨刺地面基座层（fossilspike_base、fossilspike2_base）

## 快速判断

- 职责与适用：本页收拢两个同构工厂文件的 **2 个地面基座注册**：`fossilspike_base`（一代骨刺，`fossil_spike.lua`）与 `fossilspike2_base`（二代骨刺，`fossil_spike2.lua`）。两者都是联网动画实体（Transform／AnimState／Network），铺在背景层播"起刺前的地面裂纹／砸地冲击圈"，由尖刺本体在起刺回调里 `SpawnPrefab` 并引擎父挂。[一代注册](../../../raw/current/scripts/prefabs/fossil_spike.lua#L260)、[二代注册](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L306)
- 本体辨析：`fossilspike`（官方名"化石笼子"）与 `fossilspike2`（官方名"化石骨刺"）是带 combat／物理的**混合结构攻击实体**，不在本页 FX 覆盖内；基座只是它们的视觉下层。宿主是 Boss"复活的骨架"（Stalker，官方名见[验证记录](#验证记录)；含中庭 atriumstalker 形态），战斗规则不展开。
- 重要副作用：两个 base 都 **没有自删钩子**——`persists=false` 不入存档，但只要宿主不消失就永远定格在最后一帧；唯一回收路径是随宿主实体移除。可见伤害全部在宿主的 `DoDamage`（摧毁可工作物、拾取采集物、击杀／攻击单位），base 自身零玩法组件。
- 修改边界：只覆盖 2 个注册的工厂、生成点、父挂与清理；宿主尖刺的攻击数值、骨刺 Boss 的 Brain 决策与 SG 全貌、`erode_ash` 消散烟（通用 FX，另有归属）不展开。
- 名称：宿主官方名 `STRINGS.NAMES.FOSSILSPIKE`＝"化石笼子"（[po](../../../raw/current/scripts/languages/chinese_s.po#L342969-L342972)）、`STRINGS.NAMES.FOSSILSPIKE2`＝"化石骨刺"（[po](../../../raw/current/scripts/languages/chinese_s.po#L342974-L342977)）；`FOSSILSPIKE_BASE`／`FOSSILSPIKE2_BASE` 无 `STRINGS.NAMES.*` 键，标题中的"地面基座层"是已核验的效果描述，**非官方译名**。

## 注册与构造

两文件同构：`assets` 各一个 ANIM（`anim/fossil_spike.zip`／`anim/fossil_spike2.zip`），`prefabs` 依赖表声明 `erode_ash` 与各自的 base；文件尾部一次性返回本体与 base 两个 `Prefab`（[一代返回](../../../raw/current/scripts/prefabs/fossil_spike.lua#L259-L260)、[二代返回](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L305-L306)）。

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `fossilspike_base` | `basefn`：Transform／AnimState／Network → bank／build `fossil_spike`、`base_pre1`、`LAYER_BACKGROUND`＋排序 3 → `FX`＋`NOCLICK` 标签 → Pristine → 客户端早退 → 主模拟 `persists=false`＋随机变体 | 变体在主模拟定：`inst.variation = math.random(3)`，大于 1 时补播 `base_pre<n>`；收尾段 `base_pst<n>` 由宿主 KillSpike 代播。[工厂](../../../raw/current/scripts/prefabs/fossil_spike.lua#L227-L257)、[变体](../../../raw/current/scripts/prefabs/fossil_spike.lua#L251-L254) |
| `fossilspike2_base` | `basefn`：同壳但 bank／build `fossil_spike2`、固定播 `base_impact`、同层同排序同标签 → Pristine → 客户端早退 → 主模拟 `persists=false` | **无变体、无收尾段**：二代基座只播一次 `base_impact`，宿主收尾时不触碰它的动画（对照一代）。[工厂](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L278-L303) |

### 本体辨析（混合结构，不计 FX 覆盖）

- `fossilspike`（[fn](../../../raw/current/scripts/prefabs/fossil_spike.lua#L180-L225)）：Transform／AnimState／SoundEmitter／**Physics**（mass 99999、SMALLOBSTACLES、胶囊 .2×2）／Network，标签 `notarget`／`groundspike`／`fossilspike`；主模拟 combat 默认伤害 10，`persists=false`，出生即 `StartSpike`。它砸地时摧毁周围可工作物、白嫖采集物，对无 locomotor 的目标直接 `health:Kill()`、其余走 `DoAttack`（[DoDamage](../../../raw/current/scripts/prefabs/fossil_spike.lua#L70-L113)）。
- `fossilspike2`（[fn](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L238-L276)）：同链但**无 Physics**、换 `DynamicShadow`，combat 伤害取 `TUNING.FOSSIL_SPIKE_DAMAGE`＝100（[tuning](../../../raw/current/scripts/tuning.lua#L3396)）；标签只有 `notarget`／`fossilspike`。生命周期短：`appear` 动画结束（OnImpact）时砸地，0.35 秒后 KillSpike（[OnImpact](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L131-L163)）。

## 关联机制导航

### 生成点与宿主调用

| 生成点 | 涉及注册 | 端、条件与调用 | 证据 |
|---|---|---|---|
| `fossil_spike.lua` `StartSpike` | `fossilspike_base` | 主模拟；尖刺本体起刺时生成并父挂，随后播本体 `fossil_pst`、放音效、`DoDamage`；全库唯一生成点 | [生成](../../../raw/current/scripts/prefabs/fossil_spike.lua#L157-L165) |
| `fossil_spike2.lua` `OnImpact` | `fossilspike2_base` | 主模拟；二代尖刺 `appear` 动画结束（animover）时生成并父挂，随后砸地伤害、排定 0.35 秒后 KillSpike；全库唯一生成点 | [生成](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L151-L162) |
| `stalker.lua` `SpawnSnare` | `fossilspike`（宿主） | 主模拟；一代 Boss 的圈地困人技能：环形布点、避让已有关键词实体后逐点 `SpawnPrefab("fossilspike")`，再 `RestartSpike(delay, duration, variation)`（duration＝`STALKER_SNARE_TIME` 6±0.5 秒，[tuning](../../../raw/current/scripts/tuning.lua#L3379-L3380)） | [生成](../../../raw/current/scripts/prefabs/stalker.lua#L568-L594) |
| `stalker.lua` `DoSpawnSpikes` | `fossilspike2`（宿主） | 主模拟；二代 Boss 的螺旋尖刺雨：`GenerateSpiralSpikes` 生成 30 个渐开线落点后按时间片生成，`RestartSpike(0, variation, level)` | [生成](../../../raw/current/scripts/prefabs/stalker.lua#L845-L847)、[螺旋落点](../../../raw/current/scripts/prefabs/stalker.lua#L852-L884) |

宿主调用链：Brain 决策节点产出 `fossilsnare`／`fossilspikes` 事件（[brain 派发](../../../raw/current/scripts/brains/stalkerbrain.lua#L534-L535)、[snare 推送](../../../raw/current/scripts/brains/stalkerbrain.lua#L566-L568)）→ `SGstalker` 事件处理器进 `snare`／`spikes` 状态（[事件](../../../raw/current/scripts/stategraphs/SGstalker.lua#L135-L145)）→ 状态时间线帧回调（[帧 25.5 SpawnSnares](../../../raw/current/scripts/stategraphs/SGstalker.lua#L879-L886)、[帧 6 SpawnSpikes](../../../raw/current/scripts/stategraphs/SGstalker.lua#L916-L916)）→ 上表两个 Spawn 函数。两代 Boss 都会先以 `SNAREOVERLAP_TAGS = { "fossilspike", "groundspike" }` 查重避让已有尖刺（[查重](../../../raw/current/scripts/prefabs/stalker.lua#L517-L520)）。

### 层、父挂与端模型

| 对象 | 本组注册如何使用 | 条件与边界 | 证据 |
|---|---|---|---|
| 背景层 | 两 base 都 `LAYER_BACKGROUND`＋`SetSortOrder(3)`，铺在宿主尖刺（`SetFinalOffset(1)`）脚下 | 纯动画层，无贴地旋转设置；一代变体 1 不重播（初始动画即 `base_pre1`） | [一代](../../../raw/current/scripts/prefabs/fossil_spike.lua#L236-L238)、[二代](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L287-L289) |
| 引擎父子 | 生成后立刻 `inst.basefx.entity:SetParent(inst.entity)` 挂到尖刺本体；无 Lua children 登记、无 Follower | 位置即宿主位置（本地 0,0,0）；随宿主移动 | [一代](../../../raw/current/scripts/prefabs/fossil_spike.lua#L157-L158)、[二代](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L151-L152) |
| 端模型 | base 是联网实体，变体与动画切换都在主模拟决定，随联网实体复制到各端；**没有本地子动画、没有非专服分支** | 专用服务器上同样生成 base（主模拟＝专服），只是无渲染 | [客户端早退](../../../raw/current/scripts/prefabs/fossil_spike.lua#L243-L247)、[二代早退](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L294-L298) |

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `fossilspike_base` | 宿主 KillSpike：若 base 存活则在 base 上播 `base_pst<n>`，宿主 1 秒后 OnKill（在宿主位置生成 `erode_ash`），再 0.5 秒后 OnKill2（关物理＋`ErodeAway(inst, 1)`）；`ErodeAway` 侵蚀 1 秒后 `Remove` 宿主，base 作为引擎父子子实体随层级回收 | base 无任何自身移除钩子；`base_pst` 未播完即随宿主消失。若 KillSpike 时 base 已失效或未生成，宿主直接 `Remove`，base 同样随层级回收（或已不存在）。[KillSpike](../../../raw/current/scripts/prefabs/fossil_spike.lua#L126-L148)、[ErodeAway](../../../raw/current/scripts/simutil.lua#L575-L597) |
| `fossilspike2_base` | 宿主 KillSpike：不触碰 base 动画，直接生成 `erode_ash` 并 0.5 秒后 `ErodeAway`（二代宿主无 Physics，无关物理步骤）；宿主 Remove 后 base 随层级回收 | 二代宿主寿命由 OnImpact 后 0.35 秒的 killtask 控制（[killtask](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L162)）；base 若已失效则宿主直接 `Remove`。[KillSpike](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L109-L129) |

## 典型接入

### 复刻"宿主生成＋引擎父挂"的地面基座层

- 前提与选择：攻击／出生本体自带生命周期管理时，把地面层做成无自删子实体最省事——本体消失即整层回收，不需要对齐各自的自删时刻。base 壳带 Network，适合本体本身联网复制的场景。
- 生效与撤销：主模拟 `SpawnPrefab` 后立刻 `entity:SetParent`；变体（如需）在主模拟生成端写随机并补播动画，不要在客户端补。收尾动画（若有）由宿主在 Kill 链里对 base 调 `AnimState:PlayAnimation`。
- 原版依据：[一代生成](../../../raw/current/scripts/prefabs/fossil_spike.lua#L157-L158)、[二代生成](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L151-L152)。

```lua
local function SpawnGroundBase(host, basename)
    local base = SpawnPrefab(basename)
    if base ~= nil then
        base.entity:SetParent(host.entity)
    end
end
```

- 调整与验证：替换点是注册名；层与排序在 base 工厂内写死（`LAYER_BACKGROUND`＋排序 3），调用方无法覆盖。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：base 是"消散灰烬烟"。** 灰烬烟是另一个注册 `erode_ash`，由宿主 OnKill 在宿主位置生成（与 base 位置相同所以易混）；base 只播地面裂纹／冲击圈动画，两代 base 都没有 SoundEmitter。[OnKill](../../../raw/current/scripts/prefabs/fossil_spike.lua#L121-L124)、[二代 OnKill](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L104-L107)
- **边界：base 不自删。** 单独 `SpawnPrefab("fossilspike_base")` 会得到一张常驻背景层动画（`persists=false` 不入存档，但读档/换场景前不会消失）；复用必须挂宿主管理。[工厂尾部](../../../raw/current/scripts/prefabs/fossil_spike.lua#L249-L256)
- **边界：两代收尾不对称。** 一代宿主收尾时给 base 补播 `base_pst<n>`（变体随 base 的随机 variation），二代固定 `base_impact`、宿主收尾不触碰 base 动画；对照复刻时不要把一代行为套到二代。[一代收尾](../../../raw/current/scripts/prefabs/fossil_spike.lua#L138-L140)、[二代收尾](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L115-L124)
- **边界：伤害在宿主不在 base。** 一代对无 locomotor 目标绕过 combat 直接 `health:Kill`，二代 combat 伤害 100；base 是 `FX`＋`NOCLICK` 的纯动画层，零玩法组件。[一代 DoDamage 分支](../../../raw/current/scripts/prefabs/fossil_spike.lua#L94-L101)、[二代调参](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L264-L266)
- **边界：`groundspike` 标签只在一代本体上。** 二代标签只有 `notarget`＋`fossilspike`；Boss 布点查重用 `fossilspike`＋`groundspike` 两个标签，两代都会被命中，但依赖 `groundspike` 标签做检测的系统只对一代生效。[一代标签](../../../raw/current/scripts/prefabs/fossil_spike.lua#L203-L205)、[二代标签](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L255-L256)、[查重标签](../../../raw/current/scripts/prefabs/stalker.lua#L517-L520)
- **边界：`fossilspikes` 是事件名不是 prefab 名。** SGstalker／stalkerbrain 里的 `"fossilspikes"` 字符串是 Boss 技能事件（对应 `fossilspike2` 的螺旋雨），与 `fossilspike`／`fossilspike2` 的 prefab 注册是两回事；全库没有 `SpawnPrefab("fossilspike_base")` 之外的动态拼名生成。[brain 派发](../../../raw/current/scripts/brains/stalkerbrain.lua#L534-L535)、[反查记录](#验证记录)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/fossil_spike.lua`、`prefabs/fossil_spike2.lua`（本体／base 两套工厂、StartSpike／RestartSpike／KillSpike／OnImpact 全链），按需读取 `prefabs/stalker.lua` 的 SpawnSnare／SpawnSnares／DoSpawnSpikes／GenerateSpiralSpikes／SNAREOVERLAP_TAGS，`stategraphs/SGstalker.lua` 的 snare／spikes 状态与事件，`brains/stalkerbrain.lua` 的能力决策，`simutil.lua` 的 `ErodeAway`，`tuning.lua` 的 `STALKER_SNARE_*` 与 `FOSSIL_SPIKE_DAMAGE`。全 `raw/current/scripts` 反查：`grep -rn "fossilspike_base" --include=*.lua` 命中 3 行（依赖表 L9、生成 L157、注册 L260，均在 `fossil_spike.lua`）；`fossilspike2_base` 命中 3 行（L9／L151／L306，均在 `fossil_spike2.lua`）——两 base 均无外部生成点、无动态拼名；`fossilspike` 另命中 stalker.lua 依赖表（L39、L59）与生成（L568）及 SGstalker／brain 的事件名；`fossilspike2` 另命中 stalker.lua 依赖表（L60）与生成（L845）。`fossilsnare`／`fossilspikes`（brain L146、SGstalker L141/L535）已辨析为事件名。
- 名称核验：`languages/chinese_s.po` 定向检索——`FOSSILSPIKE`＝"化石笼子"、`FOSSILSPIKE2`＝"化石骨刺"（锚点见"快速判断"）、宿主 `STRINGS.NAMES.STALKER`＝"复活的骨架"（[L349995-L349997](../../../raw/current/scripts/languages/chinese_s.po#L349995-L349997)）；`FOSSILSPIKE_BASE`／`FOSSILSPIKE2_BASE` 无 `STRINGS.NAMES.*` 键，标题使用效果描述。
- 分档说明：`size_class: standard`——2 个同构注册、单一宿主链（一代困人／二代尖刺雨两条生成路径但共用同一结构），无多阶段生命周期；本体辨析限于快速判断一节。实测行数与字节见结构检查。
- 未覆盖：`erode_ash`（通用消散烟）、`blinkfocus_marker`（SpawnSnare 附带生成的控制器逃生标记）、Stalker 的 Brain 全决策树与其余技能、二代 Boss `atriumstalker` 形态差异、动画包内容与观感。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/fossilized_break_fx|化石碎裂与化石化特效]]（同属 Stalker／化石主题的一次性 FX；对照本地代理＋联网壳结构）
- [[reference/prefabs/groundpoundring_fx|冲击环、爆点与碎块表现特效]]（同族砸地环形背景层）
- 返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]
