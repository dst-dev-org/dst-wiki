---
title: "dataanalyzer"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "维护 WX-78 对各扫描类型的可再生数据余额，区分目标扫描、数据结算、产物与表现层的职责。"
aliases: ["WX78扫描数据", "扫描生物重复收益", "生物数据恢复", "扫描数据余额", "dataanalyzer"]
tags: [component, data, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/dataanalyzer.lua", "wx78_moduledefs.lua", "prefabs/wx78.lua", "prefabs/wx78_scanner.lua", "stategraphs/SGwx78_scanner.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# dataanalyzer

## 快速判断

- 职责与适用：挂在 WX-78 玩家身上的**每扫描类型数据余额表**，供其扫描器读取、结算，并按时间恢复；不是每只生物各一份余额，也不是一般配方学习扫描器。学习目标配方的工具见 [[reference/components/recipescanner]]。[构造与读取](../../../raw/current/scripts/components/dataanalyzer.lua#L12-L43)、[宿主](../../../raw/current/scripts/prefabs/wx78.lua#L779-L780)
- 优先使用：`GetData(entity_or_id)` 查可领取整数；结算用定义解析后的 `scan_id` 调 `SpendData(scan_id)`；恢复周期用 `StartDataRegen(seconds)`，暂停用 `StopDataRegen`。[接口](../../../raw/current/scripts/components/dataanalyzer.lua#L18-L59)、[实际结算](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L735-L751)
- 重要副作用：GetData **会惰性初始化**合法扫描类型为满值；SpendData 一次取走现有整数余额、留下小数。组件不生成物品、不发余额事件、不设置 netvar 或 UI。[读取／消费](../../../raw/current/scripts/components/dataanalyzer.lua#L32-L59)、[产物由扫描器生成](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L735-L754)
- 修改边界：原版在玩家 master_postinit 添加；扫描定义来自 `wx78_moduledefs`，扫描资格、耗时、追踪、配方学习与表现仍在扫描器宿主和 SG。给其他人物单独加本组件不会获得整套扫描能力。[服务端入口](../../../raw/current/scripts/prefabs/wx78.lua#L734)、[人物注册](../../../raw/current/scripts/prefabs/wx78.lua#L843)、[扫描检查](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L414-L440)

## 关键接口

| 接口／字段 | 参数／返回 | 条件、作用与证据 |
|---|---|---|
| `GetData(ent_or_prefab)` | 实体或扫描 ID 字符串 → number | 通过定义函数解析类型；已有记录返回 floor，未记且有定义时记满值并返回 maxdata，无定义返回 0。首次 maxdata 返回未取整，原版消费会取整。[实现](../../../raw/current/scripts/components/dataanalyzer.lua#L32-L43) |
| `SpendData(prefab)` | **扫描 ID 字符串** → 已结算整数 | 直接以传入键查表；缺记录且定义存在时补满，再减去 floor(余额)。不接受“与 GetData 相同的实体解析”这一承诺，也不自动生成奖励。[实现](../../../raw/current/scripts/components/dataanalyzer.lua#L46-L59) |
| `StartDataRegen(dt)` | 周期秒数 | 先取消旧任务，再建立一个 DoPeriodicTask；每周期仅遍历已有记录，对有定义且未满的项增加 maxdata/16，并钳制上限。调用者提供有效正周期。[任务](../../../raw/current/scripts/components/dataanalyzer.lua#L18-L22)、[恢复量](../../../raw/current/scripts/components/dataanalyzer.lua#L3-L9) |
| `StopDataRegen()` | 无返回约定 | 取消并清空任务句柄；不清除 datahistory。[实现](../../../raw/current/scripts/components/dataanalyzer.lua#L25-L28) |
| `datahistory` | `scan_id → 数值余额` | 初始空表；可含小数，读／花费通常向下取整，不是历史扫描次数。存档原样保存与载入，没有数据迁移或类型校验。[初始化](../../../raw/current/scripts/components/dataanalyzer.lua#L12-L15)、[保存恢复](../../../raw/current/scripts/components/dataanalyzer.lua#L65-L76) |

### 扫描定义与 ID 归一

`GetCreatureScanDataDefinition(entity_or_id)` 返回 `(definition, scan_id)`：字符串直接当 ID；实体先按已登记特殊检查的顺序匹配，未命中才取 `entity.prefab`。特殊扫描有独立 ID，所以只传实体 prefab 可能绕过其特殊形态；应在选定目标后解析并保留返回 ID。[定义查询](../../../raw/current/scripts/wx78_moduledefs.lua#L38-L50)、[特殊登记](../../../raw/current/scripts/wx78_moduledefs.lua#L25-L34)

普通定义登记为 `{maxdata = maxdata or 1, module = module_name, recipename = recipename or nil}`，同 prefab 重登会覆盖；模块导出登记和查询函数。组件仅消费 `maxdata`，不自行实现 module／recipename 的用途。[普通定义](../../../raw/current/scripts/wx78_moduledefs.lua#L12-L17)、[导出](../../../raw/current/scripts/wx78_moduledefs.lua#L1686-L1693)

## 典型接入

### 调整 WX-78 重复扫描后的数据恢复速度

原版 `wx78` 的服务端人物初始化先添加组件，再以 `TUNING.SEG_TIME` 启动恢复。在已有 WX-78 的服务端 Prefab postinit 中，如需改变恢复速度，只需用所需秒数重新调用 StartDataRegen；它会替换任务而不叠加多个恢复器。该修改保留余额，不重置已领取数据，不改变扫描耗时。[原版接入](../../../raw/current/scripts/prefabs/wx78.lua#L779-L780)、[任务替换](../../../raw/current/scripts/components/dataanalyzer.lua#L18-L22)

恢复是离散步进：从 0 到某定义的 maxdata 需要 16 次恢复回调；小数累计到至少 1 才能被整数消费。组件不记离线经过时间，也不提供 LongUpdate；不能宣称“离线一天自动补满”。修改恢复周期需每次构造时重设，数据余额继续走原存档。[恢复步进](../../../raw/current/scripts/components/dataanalyzer.lua#L3-L9)、[取整](../../../raw/current/scripts/components/dataanalyzer.lua#L54-L57)、[完整保存接口](../../../raw/current/scripts/components/dataanalyzer.lua#L65-L79)

### 扫描完成与领取数据是两个阶段

1. 扫描器从周围实体筛目标，调用持有者 GetData；是否保留还结合扫描定义与可学习配方。扫描中的周期检查允许“无数据但仍有未知配方”等情况，不能仅以余额为 0 就判断所有扫描均失败。[候选筛选](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L491-L523)、[扫描中门禁](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L419-L431)
2. `OnSuccessfulScan` 取得并暂存 `_scanned_id` 和配方信息，忘记当前目标并停止扫描；**此时没有调用 SpendData**。返回后发 `scan_success`，SG 的成功动画第 21 帧才调用 SpawnData。[记录结果](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L573-L594)、[SG 结算时机](../../../raw/current/scripts/stategraphs/SGwx78_scanner.lua#L199-L221)
3. SpawnData 从当前 follower 的 leader 获取 dataanalyzer，以暂存 ID 结算；金额大于 0 才生成 `scandata`，其 stacksize 是结算量，最后清空 ID。余额不是原生物被扣的属性；多只同种目标共享该玩家此类型的记录。[领取与产物](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L735-L754)

扫描结果 ID 也由**扫描器自己的存档**保存、载入（兼容旧 `scanned_prefab`），不同于玩家 datahistory 存档；不要把结果丢失或扫描流程复原问题都归入本组件。[扫描器存档](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L791-L808)、[玩家余额存档](../../../raw/current/scripts/components/dataanalyzer.lua#L65-L76)

### 数据与表现同步的边界

本组件仅持有普通 Lua 表，不注册 replica、网络变量或余额变化事件。扫描器自己用 `_showringfx` 网络状态驱动扫描／成功／失败的光圈效果，雷达与信号增幅也各有 netvar；这些不是 datahistory 的网络副本。若自定义余额 HUD，需另外设计按需同步与显示，不能直接在客户端读服务端组件表。[完整组件](../../../raw/current/scripts/components/dataanalyzer.lua#L1-L79)、[扫描器网络变量](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L892-L912)、[状态发送](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L611-L619)、[表现消费](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L625-L640)

## 常见陷阱与限制

- **把查询当无副作用预览：** GetData 会填满新扫描类型；别用它统计“历史上是否已扫描”。首次有效定义即有 maxdata，不需要先增加次数。[读取初始化](../../../raw/current/scripts/components/dataanalyzer.lua#L32-L43)
- **向 SpendData 传实体或普通 prefab 代替特殊 ID：** 它的表索引用原入参，不像 GetData 使用返回 scan_id；原版先存扫描定义返回 ID，再拿该字符串消费。模组也应保持同一键，避免异常记录或错扣普通类型。[两方法差异](../../../raw/current/scripts/components/dataanalyzer.lua#L32-L59)、[宿主保存 ID](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L580-L584)
- **动态删组件等于停任务：** dataanalyzer 无 OnRemoveFromEntity，框架 RemoveComponent 不自动取消该组件创建的实体任务；先 StopDataRegen 再移组件。删除整个实体时框架才取消其 pending tasks。[组件全部定义](../../../raw/current/scripts/components/dataanalyzer.lua#L12-L79)、[组件移除](../../../raw/current/scripts/entityscript.lua#L670-L683)、[实体删除](../../../raw/current/scripts/entityscript.lua#L1741-L1750)
- **载入自动修复旧定义：** OnLoad 原样替换 datahistory；恢复只处理仍有定义且小于 maxdata 的记录，不清未知键，也不将高于新上限的旧余额主动压回。调整定义上限或重命名 ID 需另设显式迁移，不能假定读档自愈。[原样加载](../../../raw/current/scripts/components/dataanalyzer.lua#L71-L76)、[恢复判断](../../../raw/current/scripts/components/dataanalyzer.lua#L3-L9)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 2 处（wx78_moduledefs 导出段 -2、entityscript OnRemoveEntity 段 +3）；无结论修订，仅锚点迁移——dataanalyzer.lua、wx78_scanner.lua、SGwx78_scanner.lua 未变；wx78.lua 本版改动（master_postinit 内 2 行增删）净零，779-780／734／843 行内容未变。依据：全局枢纽 diff 审阅＋变更清单命中 wx78_moduledefs.lua、prefabs/wx78.lua、entityscript.lua。
- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整 dataanalyzer、扫描定义归一／导出、WX-78 挂载、扫描器目标筛选／成功记录／SG 结算／产物／结果存档与表现状态，及组件移除和实体任务清理；未全审技能、模块配方教学和扫描器完整 AI／所有定义。
- 结构检查：本页元数据、链接、大小及源码引用定向严格检查通过。未新增完整 Lua 示例，无示例语法检查项。
- 游戏运行：未运行；恢复周期、特殊形态切换、扫描返回后结算、离线重载、多人余额展示及自定义数据迁移仍待实测。

相关：[[reference/components/recipescanner]]；[[reference/components/follower]]；[[reference/components/entitytracker]]。
