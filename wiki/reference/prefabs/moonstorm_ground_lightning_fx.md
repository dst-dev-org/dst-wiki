---
title: "月风暴地面闪电（moonstorm_ground_lightning_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "月风暴的地面落雷表现：紫蓝色地面动画加电流声，13 帧后沿自身旋转加 90 度方向的 8 距离外链式生成下一道（限可视地面、月风暴节点与玩家 30 距离内），动画播完自删。"
aliases: ["moonstorm_ground_lightning_fx", "月风暴闪电", "月风暴落雷", "地面闪电特效"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/moonstorm_ground_lightning_fx.lua", "components/moonstormlightningmanager.lua", "prefabs/forest.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 月风暴地面闪电（moonstorm_ground_lightning_fx）

## 快速判断

- 职责与适用：月风暴期间的**地面落雷表现实体**：非联网（无 AddNetwork），bank/build `moonstorm_groundlight`，50/50 播 strike/strike2，OnGround＋LAYER_BACKGROUND＋SortOrder 3，蓝紫色调 MultColour(0.5,0.5,1,1)＋bloom。[工厂](../../../raw/current/scripts/prefabs/moonstorm_ground_lightning_fx.lua#L66-L102)
- 优先使用：查"月风暴背景落雷"选本页；主生成者是 forest 世界的 moonstormlightningmanager 组件（仅非专用机器挂载），本 fx 落雷链自身还会续生。[组件](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L1-L5)、[挂载](../../../raw/current/scripts/prefabs/forest.lua#L626-L634)
- 重要副作用：**自链式生成**——DoTaskInTime(13 帧) 后在"自身旋转 +90°±20°"方向 8 距离处再生成一道（须可视地面＋目标点在月风暴节点＋距 ThePlayer 30 内）；生成即播电流声。[链式](../../../raw/current/scripts/prefabs/moonstorm_ground_lightning_fx.lua#L21-L64)
- 修改边界：纯表现层、无伤害组件；fn 的 pondtype 参数与 ENDPOINTS=5 常量均未被使用（遗留代码）。[注册](../../../raw/current/scripts/prefabs/moonstorm_ground_lightning_fx.lua#L104)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `moonstorm_ground_lightning_fx` | fn：CreateEntity → Transform＋AnimState＋SoundEmitter → bank/build＋随机动画 → OnGround/背景层/排序 3 → 蓝紫 MultColour → 随机朝向（0-360）→ bloom → fx/NOCLICK 标签 → 13 帧链式任务＋animover Remove → 生成即播 `moonstorm/common/moonstorm/electricity` → persists=false | 单一注册；无 SetPristine/mastersim 分支——非联网实体仅在生成机本地存在。[工厂](../../../raw/current/scripts/prefabs/moonstorm_ground_lightning_fx.lua#L66-L102) |

注册名无 NAMES 条目；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：moonstormlightningmanager（生成者） | SpawnLightning（即 spark.spawnfn 默认值）随机朝向落雷；OnUpdate 按相机距离在玩家周围可视半径（30-75）掷点，checkground 双重校验后生成 | forest 世界仅非专用挂载（[挂载](../../../raw/current/scripts/prefabs/forest.lua#L626-L634)）；各客户端各自生成自家落雷 | [[reference/components/moonstormlightningmanager]]；[SpawnLightning](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L1-L5)、[OnUpdate](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L50-L76) |
| 数据：moonstorms._moonstorm_nodes 净值 | 链式 checkspawn 与组件 checkground 都要求落点节点索引在活跃月风暴节点集合内 | 读 `TheWorld.net.components.moonstorms`；缺组件的世界该读取会出错 | [节点判定](../../../raw/current/scripts/prefabs/moonstorm_ground_lightning_fx.lua#L47-L57) |
| 链式推进：checkspawn | 新 fx 继承 anglemod 并累加旋转（`新旋转 = 旧旋转 + anglemod/DEGREES`），同一偏移使整条链沿弧线推进；玩家 30 距离外链条终止 | 仅生成它的本机（非联网；非专服主机也可生成）；新 fx 位置须 IsVisualGroundAtPoint | [checkspawn](../../../raw/current/scripts/prefabs/moonstorm_ground_lightning_fx.lua#L21-L64) |
| 定位参照：ThePlayer | checkspawn 以 ThePlayer 距离（30²）做门槛 | ThePlayer 为空时直接返回（不链生） | [距离门槛](../../../raw/current/scripts/prefabs/moonstorm_ground_lightning_fx.lua#L23-L25)、[L57](../../../raw/current/scripts/prefabs/moonstorm_ground_lightning_fx.lua#L57) |
| 资源 | anim/moonstorm_groundlight.zip；音效 `moonstorm/common/moonstorm/electricity` 生成即播 | 声明与使用已核验，包内容未解包 | [资产声明](../../../raw/current/scripts/prefabs/moonstorm_ground_lightning_fx.lua#L1-L4)、[音效](../../../raw/current/scripts/prefabs/moonstorm_ground_lightning_fx.lua#L97) |

## 典型接入

### 手动落一道地面闪电

- 前提与选择：非联网实体，本地 SpawnPrefab 即得；只在世界含月风暴节点数据的环境下链式条件才可能满足。
- 生效与撤销：动画播完（strike/strike2 单参播放、未显式传循环参数）animover 自删；不想要后续链雷就赶在其 13 帧任务前 Remove。
- 原版依据：组件 SpawnLightning 即"生成＋随机朝向＋定位"三步，无额外配置（[实现](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L1-L5)）。

```lua
local fx = SpawnPrefab("moonstorm_ground_lightning_fx")
fx.Transform:SetPosition(x, 0, z)
fx.Transform:SetRotation(math.random() * 360)
-- 注意：13 帧后会尝试链式续生下一道（须月风暴节点与玩家距离条件满足）
```

- 调整与验证：密度与半径调整走组件页的 `spark.per_sec`/`sparks_per_sec_mod` 注入点，不在本 prefab 上。

## 常见陷阱与限制

- **边界：手动生成也会自链**：链式任务是工厂自带行为（[L94](../../../raw/current/scripts/prefabs/moonstorm_ground_lightning_fx.lua#L94)＋[L58-L61](../../../raw/current/scripts/prefabs/moonstorm_ground_lightning_fx.lua#L58-L61)），把本 prefab 当"一次性落雷"复用会沿弧线续生出一串雷。
- **边界：非联网、各机各自生成**：专用服务器上管理器不挂载、无此 fx；客户端各自生成与发声（[挂载](../../../raw/current/scripts/prefabs/forest.lua#L626-L634)）。
- **边界：checkspawn 硬依赖月风暴网络组件**：ThePlayer 判空返回，但 `TheWorld.net.components.moonstorms` 不判空（[L48](../../../raw/current/scripts/prefabs/moonstorm_ground_lightning_fx.lua#L48)）；在无 moonstorms 组件的世界触发链式任务会报错——移植前先换掉或清空链式任务。
- **误区：strike2 是落雷二段**：只是 50/50 随机二选一的开场动画（[L75-L77](../../../raw/current/scripts/prefabs/moonstorm_ground_lightning_fx.lua#L75-L77)），与链式推进无关。
- **边界：纯表现无伤害**：本实体无 combat/health 等组件；落雷的伤害机制在别的实体，不要借本 prefab 做攻击。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；104 行工厂全文（含 checkspawn 链式全程）、moonstormlightningmanager 组件与 forest.lua 挂载分支、NAMES 检索（无命中）已核验；动画包内容、链式推进的实际路径形状、晚加入时序未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页 sources 所引文件原始字节本轮全部未变（哈希等价批量沿用，无语义修订），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-3）。
