---
title: "喷气孔地面特效（cave_vent_ground_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "洞穴喷气孔（Fumarole）配套的地面红晕动画实体：联网、可存档（animnum 与朝向持久化），但当前版本原版代码没有运行时生成点。"
aliases: ["cave_vent_ground_fx", "喷气孔地面特效", "喷气孔红晕", "cave vent ground fx"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/cave_vent_ground_fx.lua", "prefabs/cave_vents.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 喷气孔地面特效（cave_vent_ground_fx）

## 快速判断

- 职责与适用：喷气孔（宿主 `cave_vent_rock`）洞口的地面红晕动画实体：联网、地面背景层、bloom＋红色乘色（alpha=1）；主模拟随机朝向与 `animnum` 变体，走 savedrotation／OnSave 持久化。[工厂](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L71-L115)
- 优先使用：查"喷气孔地面红晕／裂口特效"选本页；宿主的喷雾玩法（含瘴气）在 [[reference/components/miasmamanager]] 与宿主状态动画里，不经本实体。
- 重要副作用：**当前版本原版代码无运行时生成点**：全库唯一 `SpawnPrefab("cave_vent_ground_fx")` 在本文件的 `checkspawn`，而调用它的那行被注释（[L111](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L111)）；宿主 `cave_vent_rock`（工厂文件 cave_vents.lua）只在 prefabs 声明里列了它，从未生成（[声明](../../../raw/current/scripts/prefabs/cave_vents.lua#L8-L17)）。
- 修改边界：工厂 `persists` 未关闭（对应行被注释并带问号），实体默认入存档；`checkspawn` 的三连复制逻辑是死代码，仅作布置意图参考。[存档](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L60-L68)、[死代码](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L26-L58)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cave_vent_ground_fx` | fn：Transform＋AnimState（bank/build `cave_vent_ground`、播 `idle1`、OnGround 背景层 SortOrder 3、bloom、MultColour(1,0.5,0.5,1)）→ FX／NOCLICK 标签 → SetPristine → 主模拟：随机朝向、随机 animnum(1-2)（非 1 重播 `idle{animnum}`）、savedrotation 组件、OnSave/OnLoad 存 animnum | 单一注册；客户端共享动画与标签，朝向与变体由主模拟定。[工厂](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L71-L115)、[存读](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L60-L68) |

宿主岩石的官方译名为"喷气孔"（NAMES.CAVE_VENT_ROCK，[译名](../../../raw/current/scripts/languages/chinese_s.po#L340174-L340177)）；本 FX 注册名无 NAMES 条目，标题"喷气孔地面特效"是已核验效果描述（借宿主官方名），非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 布置意图（死代码）：checkspawn | 按自身朝向取 8 距离外一点，地面校验失败即放弃；否则按 POSITIONS[animnum] 的 3 个偏移再生成 3 个自身并继承角度修正 | 仅被注释掉的 `DoTaskInTime(3, ...)` 调用过；POSITIONS 两组各 3 点 | [checkspawn](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L26-L58)、[POSITIONS](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L13-L24) |
| 宿主：cave_vent_rock | prefabs 依赖声明；喷雾时播 `full_geyser` 等宿主动画、走 miasmamanager 造雾，均不经本实体 | MIASMA 型气孔加 `miasma_venter` 标签、按 timer 周期造瘴气 | [依赖声明](../../../raw/current/scripts/prefabs/cave_vents.lua#L8-L17)、[喷雾](../../../raw/current/scripts/prefabs/cave_vents.lua#L179-L189)、[[reference/components/miasmamanager]] |
| 持久化 | savedrotation 组件存朝向；OnSave/OnLoad 存 animnum | 主模拟；工厂 persists 默认开启（未显式关闭） | [OnSave/OnLoad](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L60-L68)、[savedrotation](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L104) |
| 资源 | anim/cave_vent_ground.zip（bank/build 同名），动画含 idle/open/close/end 系列（文件头注释） | 声明与使用已核验，包内容未解包 | [资源声明](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L1-L11) |

## 典型接入

### 让自定义洞口复用地面红晕

- 前提与选择：主模拟生成即可（联网实体，客户端自动同步）；想还原原版布置意图就按 checkspawn 的 POSITIONS 偏移一次放 3 个并设同一 anglemod。
- 生效与撤销：实体自身永不自移除（无移除任务、无 animover 监听），宿主需要自己负责清理；persists 默认开启，会随存档保存。
- 原版依据：当前版本没有原版生成者可参照，运行行为只能按工厂代码与死代码布置意图推断。[工厂](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L71-L115)、[checkspawn](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L26-L58)

```lua
local fx = SpawnPrefab("cave_vent_ground_fx")
fx.Transform:SetPosition(x, 0, z)
fx.Transform:SetRotation(rot)       -- 主模拟随机朝向是原版做法
fx.anglemod = 0                     -- checkspawn 继承字段（模组复用时自备）
```

- 调整与验证：`animnum` 决定 idle 变体与（死代码中的）偏移组；加载后仅恢复 animnum 不恢复重播（OnLoad 只写字段），变体动画要自己重播。验证入口见工厂注册与存读（[工厂](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L97-L107)）。

## 常见陷阱与限制

- **误区：以为喷气孔会自动带这个地面特效**：宿主 cave_vent_rock 的喷雾表现全部走自身动画与 miasma_cloud，prefabs 声明里的 `cave_vent_ground_fx` 只是依赖登记；本实体在当前版本无任何生成者。[宿主喷雾](../../../raw/current/scripts/prefabs/cave_vents.lua#L164-L171)、[声明](../../../raw/current/scripts/prefabs/cave_vents.lua#L8-L17)
- **边界：实体不自清理且默认入存档**：无移除任务、persists 未关闭；模组生成后要自管生命周期，忘了 Remove 会永久留在存档里。[工厂](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L109-L112)
- **边界：checkspawn 是死代码**：其中的地面校验、8 距离落点与 3 点偏移都未在运行时执行过；不要按"原版会这样铺 3 个"写结论，只能当布置意图参考。[死代码](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L26-L58)、[注释调用](../../../raw/current/scripts/prefabs/cave_vent_ground_fx.lua#L111)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；117 行工厂全文、cave_vents.lua 宿主全文、全库 `cave_vent_ground`／`checkspawn` 引用检索已核验（仅 prefablist 登记、宿主声明与死代码自引用）；动画包内容、若启用 checkspawn 的实际表现、晚加入时序未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
