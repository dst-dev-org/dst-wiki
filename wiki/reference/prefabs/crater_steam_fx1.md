---
title: "温泉蒸汽（crater_steam_fx 与 slow_steam_fx 族）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "月岛温泉的两组编号蒸汽动画：浴弹状态选 crater 四连发、平时选 slow 五连发，由温泉周期任务随机点名。"
aliases: ["crater_steam_fx1", "crater_steam_fx2", "crater_steam_fx3", "crater_steam_fx4", "slow_steam_fx1", "slow_steam_fx2", "slow_steam_fx3", "slow_steam_fx4", "slow_steam_fx5", "温泉蒸汽", "月岛温泉蒸汽", "crater steam", "slow steam", "温泉冒烟"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "prefabs/hotspring.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 温泉蒸汽（crater_steam_fx 与 slow_steam_fx 族）

## 快速判断

- 职责与适用：月岛温泉（hotspring）水面蒸汽的纯表现动画，共 9 个编号注册；无玩法组件，不产生热量或湿度。生成协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[注册循环](../../../raw/current/scripts/fx.lua#L4047-L4065)
- 优先使用：查"温泉冒烟／蒸汽动画"选本页；想借蒸汽表现可 `SpawnPrefab` 任意编号后自行落位，但温泉分支语义见下表。
- 重要副作用：无（仅动画与 FinalOffset 排序；两条循环均未配音效字段）。[条目](../../../raw/current/scripts/fx.lua#L4047-L4065)
- 修改边界：随温泉走网络代理＋本地动画两段回收；动画是否循环未核验。[代理 1 秒回收](../../../raw/current/scripts/prefabs/fx.lua#L156-L157)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `crater_steam_fx1`～`crater_steam_fx4` | 循环 `table.insert` 进通用表 → 加载器 `MakeFx`：代理＋一帧后本地实体，`fn=FinalOffset1`（FinalOffset 1） | bank／build=`crater_steam`，anim=`steam1`～`steam4` 逐号对应；其余字段与 slow 组完全一致。[循环](../../../raw/current/scripts/fx.lua#L4047-L4055) |
| `slow_steam_fx1`～`slow_steam_fx5` | 同上协议 | bank／build=`slow_steam`，anim=`steam1`～`steam5`；比 crater 组多 1 个编号。[循环](../../../raw/current/scripts/fx.lua#L4057-L4065) |

定向检索简中 .po 未发现这 9 个注册名的 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，无自定义 fn 之外的特殊字段 | 全部 9 名同协议 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L4047-L4065) |
| 生成者：hotspring | 周期任务 `push_special_idle` 按 `math.random` 点名：`bathbombable.is_bathbombed`（浴弹状态）→ crater 1～4，否则 → slow 1～5；玻璃化（`_glassed`）分支只播本体重叠闪光动画，不生成蒸汽 | 服务端温泉实体驱动；周期 `TUNING.HOTSPRING_IDLE.BASE`，StartFx 随补水启动 | [分派](../../../raw/current/scripts/prefabs/hotspring.lua#L30-L66)、[补水启动](../../../raw/current/scripts/prefabs/hotspring.lua#L148-L167) |
| 生命周期：StopFx | 放空（RemoveGlass）取消 `_fx_task`；玻璃化（TurnToGlassed）不取消任务，但 `_glassed` 分支只播本体闪光、不再点名蒸汽 | 玻璃温泉挖开走 RemoveGlass→Refill 重新 StartFx | [停止](../../../raw/current/scripts/prefabs/hotspring.lua#L174-L185)、[玻璃化](../../../raw/current/scripts/prefabs/hotspring.lua#L205-L230) |
| 资源 | bank／build=`crater_steam`／`slow_steam`，各号 anim 由温泉侧随机选择 | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L4047-L4065) |

## 典型接入

### 借用温泉蒸汽表现

- 前提与选择：需要水面白气时直接 `SpawnPrefab("slow_steam_fx"..n)`；crater 组语义绑定温泉浴弹状态，单独借用时两组只是资源与编号差异。
- 生效与撤销：代理 1 秒回收、本地动画 animover 自删，无需手动清理；位置用 `Transform:SetPosition` 一次性落点，不跟随宿主。
- 原版依据：温泉周期点名 [hotspring](../../../raw/current/scripts/prefabs/hotspring.lua#L30-L66)。

```lua
local fx = SpawnPrefab("slow_steam_fx" .. math.random(5))
fx.Transform:SetPosition(x, y, z)
```

- 调整与验证：温泉以 `math.random(4)`／`math.random(5)` 点名，模组想固定形态可直接指定编号；两条循环均无音效字段，需要蒸汽声自行另配。

## 常见陷阱与限制

- **误区：crater 组是"陨石坑蒸汽"**：crater 指温泉的火山口造型资源，生成者只有温泉；浴弹状态（bathbombable）才选 crater，平时是 slow 组。[分派](../../../raw/current/scripts/prefabs/hotspring.lua#L45-L54)
- **边界：动画是否循环未核验**：条目只调用一次 `PlayAnimation`，未显式传循环参数，实际是否重复由动画数据决定，本页不作断言。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)
- **边界：玻璃化温泉不出蒸汽**：`_glassed` 分支只播本体 sparkle 动画；想给玻璃温泉加蒸汽需绕过该分支自建生成点。[玻璃分支](../../../raw/current/scripts/prefabs/hotspring.lua#L31-L43)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。两组循环注册、加载器协议、hotspring 生成分派与启停链、.po 名称键核验完毕；未覆盖：动画资源内容、温泉 bathbombable 组件全流程、晚加入时序。
- 游戏运行：未运行；蒸汽出现频率与视觉表现未实测。

返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
