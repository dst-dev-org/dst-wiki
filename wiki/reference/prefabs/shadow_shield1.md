---
title: "骨头盔甲格挡护盾（shadow_shield1）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "骨头盔甲 resistance 吸收伤害时的格挡护盾表现：六注册为三变体×水平镜像两向，PickShield 按 1.2 秒主盾冷却加概率选名，护盾父挂到骨甲持有者。"
aliases: ["shadow_shield1", "shadow_shield2", "shadow_shield3", "shadow_shield4", "shadow_shield5", "shadow_shield6", "格挡护盾", "骨甲护盾", "护盾动画", "骨头盔甲护盾"]
tags: [prefab, fx, network-sync]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "prefabs/armor_skeleton.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 骨头盔甲格挡护盾（shadow_shield1）

## 快速判断

- 职责与适用：骨头盔甲（注册名 `armorskeleton`，文件 armor_skeleton.lua，官方名 "Bone Armor"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L338209-L338212)）的 resistance 组件吸收伤害时的格挡护盾表现，六注册为三变体×镜像两向；纯表现，格挡结算在护甲本体。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[循环注册](../../../raw/current/scripts/fx.lua#L4067-L4079)
- 优先使用：查"骨甲格挡／护盾动画"选本页；护甲的免伤、燃料与冷却选 armor_skeleton 本体源码（[[reference/components/resistance|resistance 组件页]] 记协议）。
- 重要副作用：无；六注册统一配音效 `dontstarve/creatures/together/stalker/shield`、`FinalOffset 2`。[条目](../../../raw/current/scripts/fx.lua#L4067-L4079)
- 修改边界：护盾**父挂到骨甲持有者**（随其移动）；动画是否重复未核验。[挂接](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L57-L59)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shadow_shield1`～`shadow_shield3` | 双循环（j∈{0,3}×i∈1..3）`table.insert` 进通用表 → 加载器代理＋一帧后本地实体，fn=FinalOffset2 | bank／build=`stalker_shield`，anim=`idle1`～`idle3` 逐号对应；j=0 无 transform。[循环](../../../raw/current/scripts/fx.lua#L4067-L4079) |
| `shadow_shield4`～`shadow_shield6` | 同上协议（j=3 组） | 同 bank／anim，`transform = Vector3(-1, 1, 1)` 水平镜像——与 1～3 同动画不同朝向，不是不同动作。[循环](../../../raw/current/scripts/fx.lua#L4075-L4075) |

六注册在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表循环条目；差异仅 transform 镜像 | 全族同协议 | [[reference/mechanisms/generic-fx-factory]]；[循环](../../../raw/current/scripts/fx.lua#L4067-L4079) |
| 生成者：骨甲受击 | 骨头盔甲 resistance 吸收伤害（`OnResistDamage`）时按 `PickShield(inst)` 选名生成，`SetParent` 挂骨甲的 GrandOwner（充能抵抗要求护甲已装备，实际对象即穿戴者；`or inst` 仅为回退分支），随后重置 10 帧护盾任务、扣燃料、启动冷却，并**立即 `SetOnResistDamageFn(nil)` 关闭回调** | 服务端；10 帧窗口内的后续抵抗不重复生成 | [生成与关闭](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L57-L68) |
| 触发前提（回调与抵抗分层） | **生成通知回调**在工厂初始化即注册（[注册](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L230-L231)），触发后置空（[关闭](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L66-L66)），由护盾结束、重新充能、卸装三处恢复（[OnShieldOver](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L49-L55)、[OnChargedFn](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L84-L93)、[onunequip](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L149-L159)）；**是否实际吸收**由 resistance 条目（充能完成加入、护盾结束或卸装移除）与 `ShouldResistFn`（owner 有效且无 forcefield 标签，[绑定](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L230-L230)、[门禁](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L74-L82)）决定——吸收发生才生成护盾 | 服务端；回调存在不等于会抵抗 | [门禁](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L74-L82) |
| 选名算法 PickShield | 变体 3 是"主盾"：距上次主盾 ≥1.2 秒直接返回；冷却期内按 `dt/1.2` 概率抢主盾；否则按二级概率出变体 2 或 1；50% 概率加 3 翻转偏移（3→6） | 服务端；主盾记录时间在 armor 实体上 | [PickShield](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L29-L47)、[常量](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L8-L10) |
| 依赖声明 | armor_skeleton prefabs 循环声明全部六名 | 仅声明；生成只在 resistance 吸收时 | [声明](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L23-L27) |
| 资源 | bank／build=`stalker_shield` | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L4067-L4079) |

## 典型接入

### 复刻骨甲护盾选名

- 前提与选择：模组护甲想复用"主盾冷却＋概率"节奏时照抄 PickShield 结构（主盾时间门禁、两级概率、翻转偏移）；只想出表现可任选注册名父挂生成。
- 生效与撤销：护盾父挂持有者、animover 自删；表现不影响护甲结算。
- 原版依据：[PickShield](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L29-L47)。

```lua
local fx = SpawnPrefab("shadow_shield" .. tostring(math.random(6)))
fx.entity:SetParent(owner.entity)   -- 父挂持有者，随其移动
```

- 调整与验证：直接随机 1～6 会丢失主盾节奏（原版 3／6 号受 1.2 秒冷却门禁）；翻转组（4～6）只是水平镜像。

## 常见陷阱与限制

- **边界：六名是三动画×镜像，不是六种动作**：4～6 号与 1～3 号同 anim，仅 `transform=(-1,1,1)` 水平翻转；选名差异在 PickShield 的主盾门禁与翻转偏移。[循环](../../../raw/current/scripts/fx.lua#L4067-L4079)
- **误区：护盾平均随机**：主盾（3／6 号）受 1.2 秒冷却门禁，冷却期内按时间比例概率回收主盾资格；剩余概率再分给变体 2／1——不是均匀分布。[PickShield](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L29-L47)
- **边界：表现不扣燃料**：`fueled:DoDelta` 与冷却启动都在护甲本体的吸收回调里（[结算](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L65-L70)）；fx 只随吸收出现。
- **边界：父挂对象按正常触发描述**：原版正常触发时挂穿戴者（充能抵抗要求护甲已装备）；`or inst` 只是代码回退分支，不代表落地或背包中会触发。生成后不跟随护甲栏位变化重新挂接。[挂接](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L57-L68)
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。六注册循环配置（含镜像 transform）、加载器协议、armor_skeleton 的 PickShield 算法／生成挂接／常量、骨头盔甲官方名核验完毕；.po 六注册名键定向核验无结果。未覆盖：`stalker_shield` 动画资源内容、resistance 组件全流程（组件页范围）、晚加入时序。
- 游戏运行：未运行；护盾出现频率与镜像视觉未实测。

返回 [[reference/prefabs/fx-status-index|增益、受击、护盾与状态反馈]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
