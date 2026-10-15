---
title: "WAGpunk 装备蒸汽（wagpunksteam_hat_up 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "WAGpunk 帽与护甲目标追踪阶段变化的四种通用表蒸汽、音效和延迟父挂条件。"
aliases: ["wagpunksteam_hat_up", "wagpunksteam_hat_down", "wagpunksteam_armor_up", "wagpunksteam_armor_down", "WAGpunk装备蒸汽", "沃格朋克蒸汽"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/hats.lua", "prefabs/armor_wagpunk.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# WAGpunk 装备蒸汽（wagpunksteam_hat_up 等）

## 快速判断

- 四名是 WAGpunk 帽、护甲的升级与重置表现：蒸汽及其条目声效由通用表工厂处理，伤害倍率、移动速度与目标追踪仍在装备侧。通用网络代理与本地动画、清理见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 中文为依据注册动画和生成语境核验的效果描述，非官方译名；`languages/chinese_s.po` 定向检索四注册名的显示名键无命中。

## 注册与构造

四项 bank／build 均为 `wagpunk_fx`；各自 fn 对本地实体设 four-faced 和 `SetFinalOffset(3)`。均由通用表工厂注册。

| 注册名 | anim | sound | 条目 |
|---|---|---|---|
| `wagpunksteam_hat_up` | `hat_powerup` | `rifts3/wagpunk_armor/upgrade` | [条目](../../../raw/current/scripts/fx.lua#L3189-L3197) |
| `wagpunksteam_hat_down` | `hat_powerdown` | `rifts3/wagpunk_armor/downgrade` | [条目](../../../raw/current/scripts/fx.lua#L3200-L3208) |
| `wagpunksteam_armor_up` | `armor_powerup` | `rifts3/wagpunk_armor/upgrade` | [条目](../../../raw/current/scripts/fx.lua#L3211-L3219) |
| `wagpunksteam_armor_down` | `armor_powerdown` | `rifts3/wagpunk_armor/downgrade` | [条目](../../../raw/current/scripts/fx.lua#L3222-L3230) |

## 关联机制导航

| 生成者 | 触发、生成与清理 | 证据 |
|---|---|---|
| WAGpunk 帽 | 跟踪新目标与追踪时间跨阶段时调用 up；重置时持有者存在则调用 down。装备侧设置伤害倍率，不由蒸汽执行。 | [跟踪新目标](../../../raw/current/scripts/prefabs/hats.lua#L4314-L4319)、[时间阶段](../../../raw/current/scripts/prefabs/hats.lua#L4385-L4436)、[重置](../../../raw/current/scripts/prefabs/hats.lua#L4264-L4279) |
| WAGpunk 护甲 | 跟踪新目标及时间跨阶段时调用 up；重置时有持有者且装备移速倍率大于 1 才调用 down，随后归一。 | [跟踪](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L101-L109)、[时间阶段](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L161-L208)、[重置](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L62-L77) |
| 共享生成方式 | 装备取消自身先前未执行的蒸汽任务，在有效持有者上安排随机 0～0.3 秒延迟；执行时持有者仍有效且未死亡、未冰冻才 `AddChild(SpawnPrefab(...))`，即代理父挂持有者。帽与甲各自有同形 helper；本地表现随父对象和动画结束的协议见 [[reference/mechanisms/generic-fx-factory#常见陷阱与限制]]。 | [帽 helper](../../../raw/current/scripts/prefabs/hats.lua#L4241-L4257)、[帽主模拟绑定](../../../raw/current/scripts/prefabs/hats.lua#L4629-L4645)、[甲 helper](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L39-L55)、[甲主模拟分界](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L360-L362) |

## 常见陷阱与限制

- 帽和护甲的 down 触发条件不同，不能根据动画或 sound 名推断阶段变化。装备的其他持续外观或环境循环音并非本族蒸汽条目。
- 延迟任务可能被新的蒸汽任务取消，或在执行时因持有者无效、死亡、冰冻而跳过；阶段变化不保证每次都有可见蒸汽。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394；逐项核验 bank／build／anim／sound／fn。`raw/current/scripts` 全树检索四名以及 `wagpunksteam`、相关 helper 和拼名线索，命中帽与甲的生成链及其依赖声明；依赖声明不等于生成。未核验动画实际外观、时长或多人运行表现。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]。
