---
title: "shedder"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "按周期或手动生成指定脱落物，区分绝对出生高度、批量水平散射及熊獾宿主的冬眠和死亡停机。"
aliases: ["熊獾掉毛", "定时脱落物", "批量散落毛簇", "shedder"]
tags: [component, loot, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/shedder.lua", "prefabs/bearger.lua", "entityscript.lua", "scheduler.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# shedder

## 快速判断

- 职责与适用：周期或手动生成一种脱落物；构造只设字段，不自动开始。原版用于熊獾毛簇，不维护掉落表、库存或已生成物的集合。[shedder.lua:1](../../../raw/current/scripts/components/shedder.lua#L1 [shedder.lua:26](../../../raw/current/scripts/components/shedder.lua#L26
- 优先使用：定时生成配对 StartShedding／StopShedding；单次和批量分别用 DoSingleShed／DoMultiShed，先配置 shedItemPrefab。[shedder.lua:12](../../../raw/current/scripts/components/shedder.lua#L12 [shedder.lua:35](../../../raw/current/scripts/components/shedder.lua#L35
- 重要副作用：出生 y 直接取 shedHeight，**不加宿主 y**；批量生成仅对有活动 Physics 的产物施加水平速度。[shedder.lua:26](../../../raw/current/scripts/components/shedder.lua#L26 [shedder.lua:35](../../../raw/current/scripts/components/shedder.lua#L35
- 修改边界：原版在主模拟安装；季节、存活与冬眠条件由熊獾宿主处理，组件本身不检查。组件移除会 StopShedding，不回收已生成物。[bearger.lua:458](../../../raw/current/scripts/prefabs/bearger.lua#L458 [shedder.lua:47](../../../raw/current/scripts/components/shedder.lua#L47

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `shedItemPrefab`／`shedHeight` | prefab 名，默认 nil／世界 y，默认 6.5 | nil 名称不生成；高度是绝对值，宿主 x/z 各随机偏移约 ±0.5，不检查地形或落点。[shedder.lua:1](../../../raw/current/scripts/components/shedder.lua#L1 [shedder.lua:26](../../../raw/current/scripts/components/shedder.lua#L26 |
| `StartShedding(interval)` | 秒，省略时 60 | 取消已有 shedTask 后重排周期任务；初次也等待该间隔，不立即掉物。重复调用是重置等待，不是叠加多个任务。[shedder.lua:12](../../../raw/current/scripts/components/shedder.lua#L12 [entityscript.lua:1537](../../../raw/current/scripts/entityscript.lua#L1537 [scheduler.lua:295](../../../raw/current/scripts/scheduler.lua#L295 |
| `StopShedding()` | 无参数 | 取消任务并把 shedTask 清 nil；多次调用可重复。它只停周期，手动生成方法仍可调用。[shedder.lua:19](../../../raw/current/scripts/components/shedder.lua#L19 |
| `DoSingleShed()` | 返回生成实体或 nil | SpawnPrefab 成功后定位；没有 Physics 也能生成，不赋初始速度，不查周期是否开启。[shedder.lua:26](../../../raw/current/scripts/components/shedder.lua#L26 |
| `DoMultiShed(max, random)` | max 宜为正整数；无显式返回 | random 为真值时取 1～max，否则取 max 次尝试；不校验参数。逐次调用 DoSingleShed，仅活动 Physics 才设水平速度模长 4、竖直速度 0，不返回产物数组。[shedder.lua:35](../../../raw/current/scripts/components/shedder.lua#L35 |

## 典型接入

### 原版熊獾：定时毛簇与震地额外掉毛

bearger、mutatedbearger 共用 commonfn 的服务端组装：添加 shedder，设置 `shedItemPrefab="furtuft"`、shedHeight=6.5，并以 TUNING.BEARGER_SHED_INTERVAL 启动；本版该间隔为 45 秒。这里生成的是毛簇，不是其 lootdropper 掉落表中的 bearger_fur。[bearger.lua:487](../../../raw/current/scripts/prefabs/bearger.lua#L487 [bearger.lua:555](../../../raw/current/scripts/prefabs/bearger.lua#L555 [bearger.lua:796](../../../raw/current/scripts/prefabs/bearger.lua#L796 [bearger.lua:845](../../../raw/current/scripts/prefabs/bearger.lua#L845 [tuning.lua:2662](../../../raw/current/scripts/tuning.lua#L2662 [bearger.lua:76](../../../raw/current/scripts/prefabs/bearger.lua#L76

commonfn 将 groundpounder.groundpoundFn 设为 OnGroundPound；该回调按 TUNING.BEARGER_MULTISHED_CHANCE 判定，成功时 DoMultiShed(3,false)，即固定尝试生成 3 个毛簇，并非随机 0～3 个。这是宿主的额外触发，不依赖周期任务是否存在。[bearger.lua:261](../../../raw/current/scripts/prefabs/bearger.lua#L261 [bearger.lua:504](../../../raw/current/scripts/prefabs/bearger.lua#L504

普通熊獾的 sleeper 接 ShouldSleep／ShouldWake：无战斗目标、未受火伤且处于冬或春时，睡眠测试停止掉毛并进入冬眠；离开冬眠季节的唤醒测试重新 StartShedding。普通与变异熊獾各自 death 回调都显式 StopShedding。不能将普通熊獾的冬眠门禁推给所有安装 shedder 的实体。[bearger.lua:181](../../../raw/current/scripts/prefabs/bearger.lua#L181 [bearger.lua:282](../../../raw/current/scripts/prefabs/bearger.lua#L282 [bearger.lua:298](../../../raw/current/scripts/prefabs/bearger.lua#L298 [bearger.lua:583](../../../raw/current/scripts/prefabs/bearger.lua#L583 [bearger.lua:335](../../../raw/current/scripts/prefabs/bearger.lua#L335 [bearger.lua:402](../../../raw/current/scripts/prefabs/bearger.lua#L402

## 常见陷阱与限制

- **停止周期就禁止一切掉毛**：StopShedding 只取消任务；DoSingleShed／DoMultiShed 无该门禁，震地回调另有触发条件。若需彻底禁止，必须同时核对手动调用入口。[shedder.lua:19](../../../raw/current/scripts/components/shedder.lua#L19 [bearger.lua:261](../../../raw/current/scripts/prefabs/bearger.lua#L261
- **读档继续上次剩余时间**：组件无 OnSave／OnLoad／LongUpdate，未保存配置或周期剩余时间；熊獾构造重新配置并启动，宿主保存函数也未记录掉毛计时。是否再次停机由宿主后续行为决定。[shedder.lua:1](../../../raw/current/scripts/components/shedder.lua#L1 [bearger.lua:160](../../../raw/current/scripts/prefabs/bearger.lua#L160 [bearger.lua:487](../../../raw/current/scripts/prefabs/bearger.lua#L487
- **出生高度随宿主抬升、批量必然飞散**：高度是绝对世界 y；无 Physics 或 Physics 未激活时仍可生成，但不设置散射速度。掉落后的运动由产物配套负责。[shedder.lua:26](../../../raw/current/scripts/components/shedder.lua#L26 [shedder.lua:35](../../../raw/current/scripts/components/shedder.lua#L35

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、熊獾／变异熊獾共用配置与死亡停机、普通熊獾冬眠门禁、震地回调及宿主存读、周期任务首次延迟与 tuning 间隔。未展开毛簇物理实体或全部熊獾 SG／震地内部。
- 结构检查：validate_page 及源码引用 strict 检查通过；无新增完整 Lua 示例。机械检查不代替游戏运行。
- 游戏运行：未运行；绝对高度、物理散射、冬眠／唤醒与死亡、读档重排及手动触发待实测。

相关：掉落名单与死亡掉落见 [[reference/components/lootdropper]]。
