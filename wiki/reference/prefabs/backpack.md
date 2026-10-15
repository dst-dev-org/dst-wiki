---
title: "背包（backpack）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "原版普通背包的身体槽装备、八格容器、穿脱开关、燃烧掉物与皮肤跟随特效接线入口。"
aliases: ["backpack", "背包", "普通背包", "Backpack", "背包格子", "背包放冰箱", "穿戴开包"]
tags: [prefab, item, equipment, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["prefabs/backpack.lua", "containers.lua", "components/container.lua", "components/container_replica.lua", "components/inventory.lua", "standardcomponents.lua", "entityscript.lua", "languages/chinese_s.po", "prefabskin.lua", "prefabs/skinprefabs.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 背包（backpack）

## 快速判断

- 职责与适用：可拾取、装备在 BODY 槽的八格普通背包；穿上打开，脱下关闭。适合借鉴“装备物本身也是容器”的组装。[装备与容器](../../../raw/current/scripts/prefabs/backpack.lua#L268-L278)、[八格布局](../../../raw/current/scripts/containers.lua#L23-L41)
- 优先使用：改接收条件查 [[reference/components/container#让原版背包只接收具有食物类型标签的物品]]；改可收纳资格查 [[reference/components/inventoryitem#关键接口]]；新增独立物品不能只复制 WidgetSetup 调用。
- 重要副作用：点燃后禁止打开，熄灭恢复，燃尽先掉出全部内容再生成 ash 并移除背包。[火情回调](../../../raw/current/scripts/prefabs/backpack.lua#L201-L222)
- 修改边界：普通 backpack 不等于所有背包；本页不覆盖 icepack、piggyback、krampus_sack 或额外装备栏模组。公共外观／漂浮与主模拟装备／容器分开构造。[端分界](../../../raw/current/scripts/prefabs/backpack.lua#L236-L278)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `backpack` | `fn` → 公共外观、地图图标、floater → SetPristine → 主模拟容器／装备／燃烧 | `backpack` 标签、BODY 槽、WidgetSetup("backpack")；cangoincontainer=false。[公共构造](../../../raw/current/scripts/prefabs/backpack.lua#L236-L269)、[服务端组装](../../../raw/current/scripts/prefabs/backpack.lua#L271-L294) |
| `backpack_swap_fx` | `swapfxfn` → Follower／Network／highlightchild → SetPristine → 主模拟 colouradder／bloomer | 非持久化皮肤跟随 FX；按皮肤条件生成，不是另一只背包。[FX 工厂与注册](../../../raw/current/scripts/prefabs/backpack.lua#L297-L329) |

名称键 `STRINGS.NAMES.BACKPACK` 对应“背包”／Backpack。[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L338380-L338382)；未在本工厂覆盖默认名称映射。[默认取名](../../../raw/current/scripts/entityscript.lua#L840-L844)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| inspectable／inventoryitem | 检视、拾取；禁止放进其他 container | 主模拟；不会阻止放入人物主物品栏 | [[reference/components/inspectable#关键接口]]；[[reference/components/inventoryitem#关键接口]]；[配置](../../../raw/current/scripts/prefabs/backpack.lua#L266-L269) |
| equippable | BODY 槽，绑定 onequip／onunequip／onequiptomodel | 主模拟；穿戴者与模型展示路径不同 | [[reference/components/equippable#关键接口]]；[绑定](../../../raw/current/scripts/prefabs/backpack.lua#L271-L275) |
| container | WidgetSetup("backpack")，两列四行、pack 类型、侧边栏、openlimit=1 | 主模拟组件；布局定义须客户端也可取到 | [[reference/components/container#新容器的客户端布局]]；[布局](../../../raw/current/scripts/containers.lua#L23-L41) |
| container Replica／container_classified | WidgetSetup 联动 Replica；Replica 主模拟创建 classified，同步槽物品和开关信息 | 实际组件调用及 Replica 构造已核验，不从同名文件推定 | [联动](../../../raw/current/scripts/components/container.lua#L86-L97)、[网络桥](../../../raw/current/scripts/components/container_replica.lua#L3-L45) |
| burnable／propagator／hauntable | helper 添加，随后替换三种燃烧回调；作祟调用 LaunchAndDropFirstItem | 主模拟；helper 名称不保证其注释掉的效果仍生效 | [[reference/components/burnable#生命周期]]；[[reference/components/propagator#关键接口]]；[[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/backpack.lua#L280-L286)、[burnable helper](../../../raw/current/scripts/standardcomponents.lua#L164-L173)、[propagator helper](../../../raw/current/scripts/standardcomponents.lua#L200-L212) |
| floater | helper 配小型漂浮、偏移和 swap_data | 公共端；漂浮表现不提供容器开关 | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/backpack.lua#L257-L258)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| colouradder／bloomer／highlightchild | 跟随皮肤时给主体动态加 colouradder／bloomer，换皮肤时移除；FX 挂 highlightchild | 主体只在 AddSkinIdleFollowFx 路径动态添加；FX 两端有 highlightchild，主模拟另有色彩组件 | [[reference/components/colouradder#关键接口]]；[[reference/components/bloomer#关键接口]]；[[reference/components/highlightchild#帽子背包-fx-随持有者高亮]]；[动态添加](../../../raw/current/scripts/prefabs/backpack.lua#L16-L31)、[换皮清理](../../../raw/current/scripts/prefabs/backpack.lua#L116-L139)、[FX 挂载](../../../raw/current/scripts/prefabs/backpack.lua#L312-L323) |
| ash／backpack_swap_fx／资源 | 燃尽显式生成 ash；皮肤分支显式生成跟随 FX；声明 backpack、swap_backpack、ui_backpack_2x4 动画包 | 声明、生成和资源使用分别核验；未验证压缩包内容 | [资源／依赖声明](../../../raw/current/scripts/prefabs/backpack.lua#L1-L12)、[生成 FX](../../../raw/current/scripts/prefabs/backpack.lua#L16-L22)、[生成灰烬](../../../raw/current/scripts/prefabs/backpack.lua#L201-L209) |

本文件未安装 SG／Brain，也未给普通背包挂 preserver；保鲜能力需按 [[reference/components/preserver#关键接口]] 另行配置。

### 皮肤差异与跟随实体

| 皮肤／条件 | 额外关系与生成／清理 | 入口与覆盖 |
|---|---|---|
| 共用 backpack_init_fn／clear | 先解除旧皮肤钩子，再安装新 fns，并调用 OnBackpackSkinChanged；主体按 usefollowsymbol 和装备状态切换 backpack_swap_fx | [初始化与恢复](../../../raw/current/scripts/prefabskin.lua#L144-L173)；FX 的注册、组件和清理均保留本页 |
| backpack_labrat | 雷击／电击、freeze、haunted 等事件切换颜色和 usefollowsymbol；按当前装备状态重建跟随 FX，穿脱／uninitialize 撤销对应监听；颜色通过皮肤钩子存读 | [事件与清理](../../../raw/current/scripts/prefabs/skinprefabs.lua#L1494-L1576)、[存读](../../../raw/current/scripts/prefabs/skinprefabs.lua#L1577-L1605) |

本表只展开已核验的附加逻辑路径，不按每个外观皮肤复制工厂。其余皮肤全集仍未审校；不据共用函数推定每种皮肤都使用跟随 FX。

## 典型接入

### 穿脱背包如何打开和关闭容器

onequip 设置角色 swap_body 外观，随后 container:Open(owner)；onunequip 清除外观并 Close(owner)；onequiptomodel 只关闭容器，不复用穿戴开包。因此“添加 equippable＋container”并不会自动生成这组回调。[穿戴](../../../raw/current/scripts/prefabs/backpack.lua#L154-L173)、[脱下与模型](../../../raw/current/scripts/prefabs/backpack.lua#L175-L199)

人物主物品栏的溢出范围由 GetOverflowContainer 返回 BODY 槽物品的可打开容器。背包在地面、其他槽位或 canbeopened=false 时，不能据此假定仍是同一溢出容器。[实际选择](../../../raw/current/scripts/components/inventory.lua#L1459-L1467)

### 修改格子和接收条件时保留两端布局

八格来自 containers.params.backpack 的 slotpos；服务端 WidgetSetup 同时委派 Replica，客户端 Replica 构造也读取 containers 配置。若新增布局键，须保证双方都能取得相同定义；格子、皮肤外观与装备回调是不同修改面。[布局生成](../../../raw/current/scripts/containers.lua#L23-L41)、[组件委派](../../../raw/current/scripts/components/container.lua#L86-L97)、[客户端读取](../../../raw/current/scripts/components/container_replica.lua#L47-L54)

改变普通背包接收物品条件可复用现有 [[reference/components/container#让原版背包只接收具有食物类型标签的物品]]，不需要重新实现全套皮肤跟随逻辑。

### 火情和皮肤扩展的生命周期

onignite／onextinguish 分别置 canbeopened=false／true，onburnt 掉出内容并 Close，再生成灰烬与移除主体。包装这些回调时须保留原清理，不只处理背包外观。[火情路径](../../../raw/current/scripts/prefabs/backpack.lua#L201-L222)

换皮肤先移除旧 idlefx／swapfx 并重置动态组件，再按 usefollowsymbol、skin_build 和装备状态选择重建；卸装跟随 FX 可能先移出场景，睡眠后再删除。不能将“当前不可见”视作“已经没有 FX 实例”。[卸装清理](../../../raw/current/scripts/prefabs/backpack.lua#L85-L139)

## 常见陷阱与限制

- **误区：普通背包能直接塞进冰箱**：cangoincontainer=false 是物品侧限制，冰箱自身过滤还会进一步限制。模组放宽收纳资格后，保鲜仍应按直接 owner 判断，见 [[reference/components/preserver#常见陷阱与限制]]。[普通背包限制](../../../raw/current/scripts/prefabs/backpack.lua#L268-L269)
- **误区：作祟一定掉出第一件内容物**：MakeHauntableLaunchAndDropFirstItem 的掉物分支在本版被注释，实际保留发射路径。[helper](../../../raw/current/scripts/standardcomponents.lua#L1486-L1525)
- **误区：OnSave／OnLoad 只保存皮肤就丢容器内容**：本文件实体回调确实仅委派皮肤钩子，不能据此代替组件存档检查；容器机制入口见 [[reference/components/container#关键接口]]。[实体回调](../../../raw/current/scripts/prefabs/backpack.lua#L224-L234)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整读取 backpack.lua，核验 helper 添加组件、八格布局、container／Replica 配置、人物溢出选择、燃烧和皮肤动态关系、两个注册名与中文名称。
- 皮肤补核：同日读取 backpack_init_fn／clear 与 backpack_labrat 的 fns、定义和色彩辅助函数，补齐事件来源、撤销及存读导航，未核验全部皮肤实现。
- 游戏运行：未运行；未验证皮肤全集、资源压缩包内容、假人展示、多人开关、存档恢复或其他装备栏模组。未审校每个组件内部行为；无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
