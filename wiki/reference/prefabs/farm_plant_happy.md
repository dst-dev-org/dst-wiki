---
title: "作物表情与活动彩纸（farm_plant_happy 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "作物压力变化与杂草照料表情、枕头战出界彩纸及牛年彩纸通用表条目；区分生成条件与仅依赖声明。"
aliases: ["farm_plant_happy", "farm_plant_unhappy", "yotb_confetti", "pillowfight_confetti_fx", "作物表情", "杂草照料表情", "枕头战出界彩纸", "牛年彩纸"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/farm_plants.lua", "prefabs/weed_plants.lua", "components/farmplantstress.lua", "prefabs/yotr_fightring.lua", "prefabs/yotb_stage.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 作物表情与活动彩纸（farm_plant_happy 等）

## 快速判断

- 四名均为通用表动画表现；作物的 happiness 压力改变、杂草照料和枕头战出界各有不同生成链。`yotb_confetti` 在已检索范围内只有表项及舞台依赖声明，尚未找到生成点。代理与本地动画、回收协议见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 压力状态与出界淘汰由生成者处理，特效不承担这些玩法；枕头战彩纸条目另带音效。端、位置和清理限制见 [[reference/mechanisms/generic-fx-factory#常见陷阱与限制]]。
- 标题及别名中的中文是依动画名称和使用语境核验的效果描述，非官方译名；在 `languages/chinese_s.po` 定向查找四个注册名的 `STRINGS.NAMES` 键均未命中。

## 注册与构造

四项由 `fx.lua` 数据表经通用加载器注册；下表未列的 `sound` 和 `fn` 均无该字段。`FinalOffset1` 设置 1，`FinalOffsetNegative1` 设置 -1。[helper](../../../raw/current/scripts/fx.lua#L1-L3)、[负偏移](../../../raw/current/scripts/fx.lua#L13-L15)

| 注册名 | bank／build | anim、sound、fn | 条目证据 |
|---|---|---|---|
| `farm_plant_happy` | `farm_plant_happiness`／同左 | `happy`；无 sound；`FinalOffset1` | [条目](../../../raw/current/scripts/fx.lua#L2314-L2318) |
| `farm_plant_unhappy` | `farm_plant_happiness`／同左 | `unhappy`；无 sound；`FinalOffset1` | [条目](../../../raw/current/scripts/fx.lua#L2321-L2325) |
| `yotb_confetti` | `beefalo_fx`／同左 | `transform`；无 sound、fn | [条目](../../../raw/current/scripts/fx.lua#L2328-L2331) |
| `pillowfight_confetti_fx` | `pillowfight_confetti`／同左 | `out`；`summerevent/cannon/fire1`；`FinalOffsetNegative1` | [条目](../../../raw/current/scripts/fx.lua#L3113-L3118) |

## 关联机制导航

| 生成者／关系 | 条件、选名与落点 | 证据 |
|---|---|---|
| 作物 `farmplantstress` | `happiness` 类别的值变化时调用回调；`stressed` 为真选 unhappy，否则选 happy；延时 0.5～1 秒在作物当前位置落点，无父挂。照料将其设为 false，点燃设为 true。组件与回调在主模拟侧安装。 | [变化判断](../../../raw/current/scripts/components/farmplantstress.lua#L37-L45)、[选名与生成](../../../raw/current/scripts/prefabs/farm_plants.lua#L108-L116)、[点燃](../../../raw/current/scripts/prefabs/farm_plants.lua#L93-L94)、[端分界](../../../raw/current/scripts/prefabs/farm_plants.lua#L830-L834)、[绑定](../../../raw/current/scripts/prefabs/farm_plants.lua#L854-L868) |
| 杂草 `farmplanttendable` | 照料回调延时 0.5～1 秒在杂草当前位置生成 happy，无父挂；回调在主模拟侧安装。 | [生成](../../../raw/current/scripts/prefabs/weed_plants.lua#L3-L9)、[端分界](../../../raw/current/scripts/prefabs/weed_plants.lua#L530-L534)、[绑定](../../../raw/current/scripts/prefabs/weed_plants.lua#L554-L555) |
| 枕头战场地 | 比赛开始后周期检查，合法参赛者超出场地半径时标记出界、推送事件，并在参赛者 x/z、y=0 位置生成 `pillowfight_confetti_fx`，无父挂；场地服务端逻辑。 | [检查与生成](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L118-L137)、[周期启动](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L350-L355)、[端分界](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L629-L631) |
| 牛年舞台依赖 | `yotb_stage` 的 `prefabs` 数组声明 `yotb_confetti`；此声明不证明生成。[声明](../../../raw/current/scripts/prefabs/yotb_stage.lua#L11-L20) | 未找到实际生成点，见验证记录。 |

## 常见陷阱与限制

- 作物表情反映的是 `happiness` 这一压力类别的变化，不是所有压力类别的总结果；重复设置相同值不会经 `SetStressed` 调用变化回调。[组件](../../../raw/current/scripts/components/farmplantstress.lua#L37-L45)
- `yotb_confetti` 与舞台另外声明的 `confetti_fx` 是不同注册名，不能把后者生成当作前者的证据。[舞台依赖](../../../raw/current/scripts/prefabs/yotb_stage.lua#L11-L20)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394；逐项核验四条 bank、build、anim、sound、fn 和上列调用链。对 `raw/current/scripts` 全树用四个完整注册名及 `farm_plant_`、`yotb`／`confetti`、`pillowfight` 等前缀及拼接模式反查；动态 `farm_plant_` 组装指向植物定义等，未发现额外的上述 FX 生成链。`yotb_confetti` 未找到生成点不证明不存在，也不判为死代码。动画实际外观和时长未运行核验。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-material-index|采集、制作、碎裂与生成烟]]。
