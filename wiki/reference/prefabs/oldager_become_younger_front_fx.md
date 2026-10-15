---
title: "年龄转换特效（oldager_become_younger_front_fx）"
created: "2026-10-31"
updated: "2026-11-01"
type: prefab
summary: "六个通用表注册承载旺达年龄档切换表现：年轻段按前后两层（FinalOffset ±1）组合，变老单段；骑乘自动换 _mount 资源，父挂旺达持续跟随。触发在 healthdelta 按生命百分比重选年龄档，静默变身不出 FX；名称前缀 oldager 不表示组件直接生成。"
aliases: ["oldager_become_younger_front_fx", "oldager_become_younger_back_fx", "oldager_become_older_fx", "oldager_become_younger_front_fx_mount", "oldager_become_younger_back_fx_mount", "oldager_become_older_fx_mount", "年龄转换特效", "旺达年龄", "变老特效", "变年轻特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/wanda.lua", "components/oldager.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 年龄转换特效（oldager_become_younger_front_fx）

## 快速判断

- 职责与适用：六个通用表注册承载旺达（[[reference/components/oldager|oldager]] 宿主）三档年龄切换的时段沙漏表现——年轻段前后两半、变老整段，各带骑乘变体。工厂协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目组](../../../raw/current/scripts/fx.lua#L2484-L2531)
- 优先使用：查"年龄转换特效／变老／变年轻"到本页；年龄伤害换算与回春读 [[reference/components/oldager|oldager]]。
- 重要副作用：FX 父挂旺达持续跟随（无解挂 fn），前后层用 FinalOffset 分层；切换声（older／younger_transition）与台词由 wanda 另播。[父挂](../../../raw/current/scripts/prefabs/wanda.lua#L42-L49)
- 修改边界：become* 与 PlayAgingFx 均为 wanda.lua 局部函数，未导出；静默路径不生成任何表现。[静默判定](../../../raw/current/scripts/prefabs/wanda.lua#L173-L173)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `oldager_become_younger_front_fx` | 通用表条目带 fn=`FinalOffset1`（前层渲染偏移） | bank／build=`wanda_time_fx`，anim=`younger_top`，`nofaced`。[条目](../../../raw/current/scripts/fx.lua#L2484-L2490) |
| `oldager_become_younger_back_fx` | 条目带 fn=`FinalOffsetNegative1`（后层） | 同 bank，anim=`younger_bottom`。[条目](../../../raw/current/scripts/fx.lua#L2492-L2498) |
| `oldager_become_older_fx` | 条目带 fn=`FinalOffset1` | 同 bank，anim=`older`。[条目](../../../raw/current/scripts/fx.lua#L2500-L2506) |
| `oldager_become_younger_front_fx_mount`／`_back_fx_mount` | 同上协议，fn 同名 | bank／build 换 `wanda_time_fx_mount`。[条目](../../../raw/current/scripts/fx.lua#L2509-L2523) |
| `oldager_become_older_fx_mount` | 同上协议 | 同 mount 资源，anim=`older`。[条目](../../../raw/current/scripts/fx.lua#L2525-L2531) |

六名在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。旺达官方名见 [po:351100-351102](../../../raw/current/scripts/languages/chinese_s.po#L351100-L351102)。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 代理＋一帧本地表现 | 消费者父挂代理到旺达 | [[reference/mechanisms/generic-fx-factory]] |
| 生成入口：PlayAgingFx | 骑乘时给 fx 名追加 `_mount` 后缀，生成后 `SetParent(inst.entity)` 父挂旺达 | 服务端；跟随宿主移动 | [PlayAgingFx](../../../raw/current/scripts/prefabs/wanda.lua#L42-L49) |
| 触发链 | `healthdelta` → `onhealthchange` 先按当前档分派：old 档在生命升回 OLD 阈值上时动作——**非静默一律先转 normal**（即使已越过 YOUNG 阈值），仅静默且 ≥YOUNG 才直达 young；young 档对称（非静默跨过 OLD 阈值也先转 normal）；**normal 档与无档位记录共用**阈值直选分支：≤OLD→becomeold、≥YOUNG→becomeyoung、中间→becomenormal（normal→normal 不落 FX）。落 FX 按切换前档映射：→old 与 young→normal 落 `older_fx`，→young 与 old→normal 落 younger 前后两枚 | 服务端；阈值 `WANDA_AGE_THRESHOLD_OLD`／`_YOUNG` | [onhealthchange](../../../raw/current/scripts/prefabs/wanda.lua#L173-L197)、[接线](../../../raw/current/scripts/prefabs/wanda.lua#L231-L231) |
| 提前返回 | `nomorph` 状态、`playerghost` 或已死亡时 `onhealthchange` 直接 return——完全不切档，皮肤与 FX 都不发生 | 服务端；鬼魂／死亡不是静默切档，是整段跳过 | [提前返回](../../../raw/current/scripts/prefabs/wanda.lua#L167-L171) |
| 静默分支 | `silentmorph` 状态、实体不可见或 forcesilent 时 silent=true：仍切换年龄档与皮肤，但 FX／声／台词全跳过 | 服务端；forcesilent 的调用方未核验 | [silent 判定](../../../raw/current/scripts/prefabs/wanda.lua#L173-L173) |
| 年龄伤害来源 | oldager 组件把年龄伤害按 OLDAGE_HEALTH_SCALE 累计并折算为 `health:DoDelta(..., "oldager_component")`，从而触发 healthdelta | 服务端；组件本体见组件页 | [折算](../../../raw/current/scripts/components/oldager.lua#L33-L48)、[DoDelta](../../../raw/current/scripts/components/oldager.lua#L89-L89) |
| 资源 | 一套两资源：wanda_time_fx 与 wanda_time_fx_mount，各三 anim（younger_top／younger_bottom／older） | 动画时长未核验 | [条目组](../../../raw/current/scripts/fx.lua#L2484-L2531) |

## 典型接入

### 模组触发年龄档切换表现（实现参照）

- 前提与选择：`become*`／`PlayAgingFx` 是 wanda.lua 局部函数，模组不能直接调用；要走原版表现，改生命百分比（触发 healthdelta）或仿写下述 helper。
- 生效与撤销：FX 父挂旺达、animover 自删；骑乘自动用 mount 资源是 helper 内的后缀逻辑，仿写时必须保留。
- 原版依据：[PlayAgingFx](../../../raw/current/scripts/prefabs/wanda.lua#L42-L49)。

```lua
-- 实现参照（原版为 wanda.lua 局部函数，此处为等价仿写，非公开 API）：
local function PlayAgingFx(inst, fx_name)
    if inst.components.rider ~= nil and inst.components.rider:IsRiding() then
        fx_name = fx_name .. "_mount"
    end
    local fx = SpawnPrefab(fx_name)
    fx.entity:SetParent(inst.entity)
end
```

- 调整与验证：变年轻必须同时落 front 与 back 两枚才有完整前后层；改皮肤走 skinner 与本 FX 无关。

## 常见陷阱与限制

- **误区：oldager 组件直接生成 FX**：组件只把年龄伤害折算为生命变化（"oldager_component"），FX 由 wanda 的 healthdelta 档位切换播放；直接给非旺达实体挂 oldager 不会出本组表现。[折算](../../../raw/current/scripts/components/oldager.lua#L33-L48)
- **边界：提前返回与静默是两回事**：`nomorph`／鬼魂／死亡时 `onhealthchange` 整段跳过（连皮肤都不换）；`silentmorph`／不可见／forcesilent 才是静默切档——只换皮肤不落 FX／声／台词。[提前返回](../../../raw/current/scripts/prefabs/wanda.lua#L167-L171)
- **边界：非静默逐档过渡**：old 档恢复生命即使一次越过 YOUNG 阈值也先转 normal——落的是 **younger 前后两枚**（normal 化按切换前 old 档播 younger 对），不直达 young；young 档对称（先转 normal，落 `older_fx`）。跨两档的直达路径只在静默分支存在。[分派](../../../raw/current/scripts/prefabs/wanda.lua#L173-L197)
- **边界：年轻段是两枚组合**：front（younger_top，前层）＋back（younger_bottom，后层）成对生成；只落 front 会缺背层。older 为单枚前层。[配对调用](../../../raw/current/scripts/prefabs/wanda.lua#L118-L119)
- **边界：父挂是持续跟随**：与取位式水花不同，本组条目 fn 只设 FinalOffset、不解挂——FX 随旺达移动直至动画结束。[fn](../../../raw/current/scripts/fx.lua#L2484-L2490)
- **边界：骑乘变体靠名称后缀**：`_mount` 资源不是运行时切换，而是 helper 在骑乘时改拼注册名；绕过 helper 直接落非 mount 注册会得到站立资源。[helper](../../../raw/current/scripts/prefabs/wanda.lua#L42-L49)

## 验证记录

- 源码审阅：2026-10-31，Steam buildid 26230496。六注册配置（两资源三 anim 与 FinalOffset）、PlayAgingFx 后缀与父挂、onhealthchange 提前返回／静默判定／按当前档分派与四条转换路径、oldager 组件折算链、po 旺达名核验。未覆盖：动画资源内容、wanda_time_fx 资源构成、watch 道具侧的换档入口、forcesilent 调用方、晚加入时序。
- 游戏运行：未运行；年龄转换视觉未实测。

返回 [[reference/prefabs/fx-transition-index|变身、召唤、传送与退场]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
