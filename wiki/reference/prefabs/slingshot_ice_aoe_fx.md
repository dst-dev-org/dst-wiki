---
title: "弹弓 AOE 命中圈（slingshot_ice_aoe_fx）"
created: "2026-10-31"
updated: "2026-11-01"
type: prefab
summary: "四个通用表元素 AOE 注册（ice/slow/shadow/lunar，tint＋贴地背景层）在本快照内未找到原版生成调用；实际工作路径是专属工厂 slingshot_aoe_fx 单 prefab 五色着色（含无独立注册的 horror），由弹弓魔法增幅弹在命中时落 (x,0,z) 并 SetColorType。"
aliases: ["slingshot_ice_aoe_fx", "slingshot_slow_aoe_fx", "slingshot_shadow_aoe_fx", "slingshot_lunar_aoe_fx", "slingshot_aoe_fx", "弹弓AOE", "命中圈特效", "弹弓魔法增幅"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/slingshotaoefx.lua", "prefabs/slingshotammo.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 弹弓 AOE 命中圈（slingshot_ice_aoe_fx）

## 快速判断

- 职责与适用：弹弓魔法增幅弹命中时地面展开的 AOE 指示圈。四个通用表元素注册已正常注册（三名带 tint）但快照内未找到原版生成调用；当前表现由专属工厂 `slingshot_aoe_fx` 单实体着色完成。通用工厂协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[元素条目](../../../raw/current/scripts/fx.lua#L3842-L3887)
- 优先使用：查"弹弓 AOE／命中圈特效"到本页；常规命中表现读 [[reference/prefabs/slingshotammo_hitfx_rock|弹弓弹药命中表现]]。
- 重要副作用：hub 本体播 `target_fx_pst` 贴地动画并携带本地渐隐圆盘，颜色经 net_tinybyte 同步。[hub 工厂](../../../raw/current/scripts/prefabs/slingshotaoefx.lua#L143-L186)
- 修改边界：五处消费全部在 `inst.magicamplified` 门禁内——未增幅的弹药不出圈；hub 与蓄力前后层属同文件兄弟注册，不在通用表。[门禁](../../../raw/current/scripts/prefabs/slingshotammo.lua#L299-L308)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `slingshot_ice_aoe_fx` | 通用表条目带 fn：OnGround 朝向＋LAYER_BACKGROUND＋SortOrder 3 | bank/build=`slingshotammo`，anim=`slingshot_aoe`，tint=蓝灰(163,185,203)。**快照内无生成点**。[条目](../../../raw/current/scripts/fx.lua#L3842-L3851) |
| `slingshot_slow_aoe_fx` | 同上协议 | tint=紫(73,28,85)。**无生成点**。[条目](../../../raw/current/scripts/fx.lua#L3854-L3863) |
| `slingshot_shadow_aoe_fx` | 同上协议 | tint=黑(0,0,0)。**无生成点**。[条目](../../../raw/current/scripts/fx.lua#L3866-L3875) |
| `slingshot_lunar_aoe_fx` | 同上协议，无 tint | 白色本体。**无生成点**。[条目](../../../raw/current/scripts/fx.lua#L3878-L3887) |
| `slingshot_aoe_fx`（专属工厂） | slingshotaoefx.lua `aoefn`：本体 target_fx_pst 贴地、`SetColorType` 五色着色、本地圆盘 target_fx_ring 渐隐、animover 自删 | 非通用表；horror 有色无独立元素注册。[工厂](../../../raw/current/scripts/prefabs/slingshotaoefx.lua#L143-L186) |
| `slingshot_powerup_fx`／`_mounted_fx`（同文件兄弟） | MakePowerup：蓄力前后两层（FinalOffset 2／-2、半透明、骑乘六面向） | 非通用表。[工厂](../../../raw/current/scripts/prefabs/slingshotaoefx.lua#L258-L260) |

四名通用表注册在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 四名元素注册为通用表条目（当前无消费者）；hub 为独立网络实体工厂 | 元素注册与 hub 是两套并存的路径 | [[reference/mechanisms/generic-fx-factory]]、[hub 返回](../../../raw/current/scripts/prefabs/slingshotaoefx.lua#L258-L260) |
| 颜色系统 | `SetColorType(colorname)` 按色表 SetMultColour 并同步 net 颜色；lunar 加泛光＋LightOverride 0.5、horror LightOverride 1、圆盘 horror 覆写为黑色 | 双端：客户端听 colordirty 刷新圆盘 | [色表与圆盘覆写](../../../raw/current/scripts/prefabs/slingshotaoefx.lua#L29-L41)、[SetColorType](../../../raw/current/scripts/prefabs/slingshotaoefx.lua#L116-L136) |
| 生成者：魔法增幅命中 | 弹弓五类弹药（thulecite→shadow、冰→ice、slow→slow、horrorfuel→horror、purebrilliance→lunar）在 `magicamplified` 分支：AOE 结算后落 `slingshot_aoe_fx` 于 `(x, 0, z)` 并 SetColorType | 服务端生成；y 压平为 0 | [五处](../../../raw/current/scripts/prefabs/slingshotammo.lua#L180-L182)、[ice](../../../raw/current/scripts/prefabs/slingshotammo.lua#L305-L307)、[slow](../../../raw/current/scripts/prefabs/slingshotammo.lua#L383-L385)、[horror](../../../raw/current/scripts/prefabs/slingshotammo.lua#L538-L540)、[lunar](../../../raw/current/scripts/prefabs/slingshotammo.lua#L865-L867) |
| 依赖声明 | 部分弹药 prefabs 依赖列表含 `slingshot_aoe_fx`（freeze／purebrilliance／horrorfuel 等条目旁）——声明不证明生成路径，实际生成在 onhit 的 SpawnPrefab | 服务端装配期声明 | [声明](../../../raw/current/scripts/prefabs/slingshotammo.lua#L1410-L1410) |
| 资源 | bank/build=`slingshotammo`：元素注册用 `slingshot_aoe`，hub 用 `target_fx_pst`＋`target_fx_ring` | 动画时长未核验 | [条目](../../../raw/current/scripts/fx.lua#L3842-L3851)、[hub](../../../raw/current/scripts/prefabs/slingshotaoefx.lua#L155-L160) |

## 典型接入

### 复用 AOE 命中圈

- 前提与选择：模组自定义 AOE 想复用原版指示圈，直接落 `slingshot_aoe_fx` 并调 SetColorType——这是原版在用的路径；四个元素注册同样可被 SpawnPrefab（三名自带 tint 与贴地层），但本快照内未找到原版生成调用。
- 生效与撤销：本体 animover 自删，圆盘随本体回收；颜色立即生效。
- 原版依据：[slingshotammo 生成写法](../../../raw/current/scripts/prefabs/slingshotammo.lua#L305-L307)。

```lua
local fx = SpawnPrefab("slingshot_aoe_fx")
fx.Transform:SetPosition(x, 0, z)      -- 贴地落点（y 压平）
fx:SetColorType("ice")                 -- ice/slow/shadow/horror/lunar 五选一
```

- 调整与验证：SetColorType 是 hub 实体方法（服务端），客户端自动经 net 颜色同步圆盘；元素注册可直接落（tint 由加载器应用），只是没有原版调用先例可参照。

## 常见陷阱与限制

- **边界：元素注册无原版生成调用**：`slingshot_ice／slow／shadow／lunar_aoe_fx` 四名都进入通用工厂正常注册（三名带 tint 与贴地层），SpawnPrefab 可用；但本快照内未找到原版生成调用（全 scripts 字面与 `_aoe_fx` 后缀检索仅命中 fx.lua 数据与 slingshotaoefx.lua／slingshotammo.lua 的 hub 引用）——原版表现全部走 hub 着色，元素注册更像未启用的备选路径。[检索结论](../../../raw/current/scripts/fx.lua#L3842-L3887)
- **边界：horror 有色无注册**：色表五行含 horror（红 174,37,32，圆盘覆写黑色），但通用表没有 `slingshot_horror_aoe_fx`——horrorfuel 弹药走 hub 着色而非独立注册。[色表与圆盘覆写](../../../raw/current/scripts/prefabs/slingshotaoefx.lua#L29-L41)
- **边界：门禁在弹药侧**：五处生成都在 `inst.magicamplified` 条件内；普通命中即使带元素伤害也不出圈。[门禁](../../../raw/current/scripts/prefabs/slingshotammo.lua#L299-L308)
- **边界：落点压平**：生成位置固定 `(x, 0, z)`——目标在船上或高处时圈仍贴海平面／地面高度。[落点](../../../raw/current/scripts/prefabs/slingshotammo.lua#L306-L306)

## 验证记录

- 源码审阅：2026-10-31，Steam buildid 26230496。四元素注册配置与 tint、hub 工厂（本体／圆盘／五色／net 同步）、slingshotammo 五处 magicamplified 消费与落点、命中列表引用、负向检索（元素注册全库无生成点）核验。未覆盖：动画资源内容、slingshot_powerup 两兄弟注册的蓄力接线、晚加入时序。
- 游戏运行：未运行；命中圈视觉未实测。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
