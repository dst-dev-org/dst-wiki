---
title: "步行手杖（cane）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "手杖／加速手杖的装备移速、持握与七种皮肤映射：雪花、花瓣、彩纸、紫烟、闪光、影子和棋子拖尾的生成清理及 FX 页面入口。"
aliases: ["cane", "步行手杖", "手杖", "加速手杖", "Walking Cane", "手杖皮肤特效", "远古手杖", "拐杖糖", "城堡手杖", "愚人手杖", "玫瑰红手杖", "腐败手杖", "山羊头手杖", "手杖雪花特效", "手杖花瓣特效", "手杖彩纸特效", "手杖紫烟特效", "手杖闪光粒子", "手杖影子残影", "手杖棋子拖尾"]
tags: [prefab, item, equipment, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["prefabs/cane.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "standardcomponents.lua", "tuning.lua", "entityscript.lua", "languages/chinese_s.po", "prefabs/cane_ancient_fx.lua", "prefabs/cane_candy_fx.lua", "prefabs/cane_chess_fx.lua", "prefabs/cane_harlequin_fx.lua", "prefabs/cane_rose_fx.lua", "prefabs/cane_sharp_fx.lua", "prefabs/cane_victorian_fx.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 步行手杖（cane）

## 快速判断

- 职责与适用：装备加速的可拾取手持武器；移速来自 equippable.walkspeedmult，伤害来自 weapon。皮肤 FX 由额外皮肤初始化和装备事件连接。[工厂配置](../../../raw/current/scripts/prefabs/cane.lua#L60-L73)、[皮肤连接](../../../raw/current/scripts/prefabskin.lua#L3188-L3208)
- 优先使用：改加速／精神属性查 [[reference/components/equippable#修改已有手杖装备加速与精神基础贡献]]；新建类似物品查 [[cookbook/equipment/hand]]；按皮肤找特效读下方“皮肤差异与特效映射”。
- 重要副作用：装备回调控制 swap_object 和手臂显示；皮肤可另外生成跟随实体或周期拖尾，清理入口与装备属性分开。[持握](../../../raw/current/scripts/prefabs/cane.lua#L7-L28)、[FX 生命周期](../../../raw/current/scripts/prefabskin.lua#L3163-L3186)
- 修改边界：公共端配置外观、weapon 标签和 floater，主模拟加组件；本页覆盖本版七种手杖皮肤映射与主体调度，不展开各 FX 的完整粒子实现。[端分界](../../../raw/current/scripts/prefabs/cane.lua#L30-L75)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cane` | fn → 公共物理／动画／漂浮 → SetPristine → 主模拟 weapon、inspectable、inventoryitem、equippable、作祟 | 仅注册主体；皮肤标识与特效注册名分别查下表。[工厂与注册](../../../raw/current/scripts/prefabs/cane.lua#L30-L78) |

名称键 `STRINGS.NAMES.CANE` 为“步行手杖”／Walking Cane；“手杖、加速手杖”按用户提供收录。[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L339505-L339507)、[默认名称映射](../../../raw/current/scripts/entityscript.lua#L840-L844)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| weapon | 伤害取 CANE_DAMAGE；不由名字或外观决定战斗能力 | 主模拟；当前值表达式为 wilson_attack×0.5 | [[reference/components/weapon#关键接口]]；[绑定](../../../raw/current/scripts/prefabs/cane.lua#L60-L61)、[配置键](../../../raw/current/scripts/tuning.lua#L373) |
| equippable | 穿脱回调维护持握；walkspeedmult=1.25 | 主模拟；只携带在主栏不等同装备 | [[reference/components/equippable#关键接口]]；[绑定](../../../raw/current/scripts/prefabs/cane.lua#L67-L71)、[倍率](../../../raw/current/scripts/tuning.lua#L414) |
| inspectable／inventoryitem | 检视和拾取；皮肤 init 可改图标 | 主模拟；主体无 finiteuses 配置 | [[reference/components/inspectable#关键接口]]；[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/cane.lua#L63-L65) |
| floater／hauntable | 漂浮 helper 在公共端添加 floater；主模拟调用 MakeHauntableLaunch | 漂浮与作祟各有入口，和皮肤 FX 分开 | [[reference/components/floater#关键接口]]；[[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/cane.lua#L47-L48)、[调用](../../../raw/current/scripts/prefabs/cane.lua#L73)、[作祟 helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)、[漂浮 helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| 动画／皮肤回调 | cane、swap_cane 动画包；普通持握 OverrideSymbol，皮肤用 OverrideItemSkinSymbol；额外 FX 来自 SKIN_FX_PREFAB | 皮肤定义的 fx_prefab 保存到映射；cane_init_fn 读取并监听装备事件 | [资源与持握](../../../raw/current/scripts/prefabs/cane.lua#L1-L28)、[映射写入](../../../raw/current/scripts/prefabskin.lua#L4094-L4096)、[读取](../../../raw/current/scripts/prefabskin.lua#L3196-L3208) |

### 皮肤差异与特效映射

皮肤名、皮肤 build 与 FX 注册名分开列。下列中文皮肤名来自 `STRINGS.SKIN_NAMES`；每项实际走 cane_init_fn。[名称条目](../../../raw/current/scripts/languages/chinese_s.po#L394135-L394167)、[皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2902-L2996)

| 皮肤（标识） | 额外 Prefab／模式 | 生成与清理／入口 |
|---|---|---|
| 远古手杖 `cane_ancient` | [[reference/prefabs/cane_ancient_fx|cane_ancient_fx]]，影子动画拖尾，含 3 个编号变体 | fx_prefab 第 2 项；主体周期生成，卸装停止新增，已生成对象自行退出。[定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2902-L2913)、[效果](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L23-L29) |
| 拐杖糖 `cane_candycane` | [[reference/prefabs/cane_candy_fx|cane_candy_fx]]，跟随的雪花粒子 | fx_prefab 第 1 项；装备时生成并跟随持有者，卸装移除。[定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2915-L2927)、[雪花](../../../raw/current/scripts/prefabs/cane_candy_fx.lua#L90-L102) |
| 城堡手杖 `cane_chess` | [[reference/prefabs/cane_chess_fx|cane_chess_fx]]，黑白棋子拖尾，含 12 个形状／颜色变体 | 第 2 项；生成后传阵营与 owner／手杖，既有棋子按自身追踪退场。[定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2929-L2941)、[协作](../../../raw/current/scripts/prefabskin.lua#L3134-L3159)、[棋子](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L10-L31) |
| 愚人手杖 `cane_harlequin` | [[reference/prefabs/cane_harlequin_fx|cane_harlequin_fx]]，五色彩纸粒子，跟随载体 | 第 1 项；使用共用跟随生命周期。[定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2943-L2955)、[彩纸资源](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L7-L17)、[五色](../../../raw/current/scripts/prefabs/cane_harlequin_fx.lua#L23-L61) |
| 玫瑰红手杖 `cane_rose` | [[reference/prefabs/cane_rose_fx|cane_rose_fx]]，红白黄花瓣粒子，跟随载体 | 第 1 项；使用共用跟随生命周期。[定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2957-L2968)、[花瓣资源](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L7-L17)、[三色](../../../raw/current/scripts/prefabs/cane_rose_fx.lua#L23-L45) |
| 腐败手杖 `cane_sharp` | [[reference/prefabs/cane_sharp_fx|cane_sharp_fx]]，紫色烟雾，跟随载体 | 第 1 项；使用共用跟随生命周期。[定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2970-L2982)、[烟雾资源](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L1-L12)、[颜色](../../../raw/current/scripts/prefabs/cane_sharp_fx.lua#L20-L41) |
| 山羊头手杖 `cane_victorian` | [[reference/prefabs/cane_victorian_fx|cane_victorian_fx]]，淡黄色闪光粒子，跟随载体 | 第 1 项；使用共用跟随生命周期。[定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2984-L2996)、[闪光资源](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L1-L12)、[颜色](../../../raw/current/scripts/prefabs/cane_victorian_fx.lua#L20-L32) |

以上七组 FX 已建页并互链；表内只留可见效果、关系与入口，变体、粒子参数和独立回收逻辑由对应 FX 页维护。效果描述有源码纹理／注释、包络或动画依据，未解包资源或运行验证最终观感。

## 典型接入

### 修改手杖加速时保留持握与皮肤

普通手杖的移速使用 equippable 数值字段，穿戴回调负责外观。改变 walkspeedmult 可沿用已有组件案例；替换 SetOnEquip／SetOnUnequip 时须保留原持握和皮肤事件，不必为了改倍率重做整套皮肤特效。[字段与回调](../../../raw/current/scripts/prefabs/cane.lua#L7-L28)、[配置](../../../raw/current/scripts/prefabs/cane.lua#L67-L71)

### 跟随 FX 与拖尾任务如何选择和停止

装备时，第 1 项生成单个 _vfx_fx_inst 并挂到持有者 swap_object；第 2 项启用每 6 帧执行、初次延迟 2 帧的任务。拖尾先取 GrandOwner，检查可见性，再找合法地面位置并排除附近 shadowtrail／canetrail；不是每次任务都一定生成。卸装分别 Remove 跟随 FX、Cancel 拖尾任务。[装备／卸装](../../../raw/current/scripts/prefabskin.lua#L3163-L3186)、[拖尾选点](../../../raw/current/scripts/prefabskin.lua#L3106-L3136)

拖尾生成后可按宿主技能设置 FX 阵营，也可调用 FX 自身的追踪接口；主体只负责传参，不在本页推导目标 FX 的全部行为。[生成后协作](../../../raw/current/scripts/prefabskin.lua#L3134-L3159)

## 常见陷阱与限制

- **误区：从 cane.lua 没看到 SpawnPrefab 就认定没有皮肤实体**：实际逻辑位于皮肤定义与 prefabskin.lua，须核验定义→映射→消费链。[映射](../../../raw/current/scripts/prefabskin.lua#L3196-L3208)
- **误区：清皮肤函数保证立即销毁所有特效**：本版 cane_clear_fn 只恢复外观并移除事件监听；实际 FX／任务清理在 cane_unequipped，移除监听不等于执行该回调。重皮肤时序及只配拖尾的主体移除表现未实测。[clear](../../../raw/current/scripts/prefabskin.lua#L3210-L3217)、[清理回调](../../../raw/current/scripts/prefabskin.lua#L3177-L3186)
- **误区：拖尾只在移动中生成**：moving 分支用于预估落点，函数未把移动设为统一生成门禁；仍受可见性、地面与附近 FX 数量约束。[选点](../../../raw/current/scripts/prefabskin.lua#L3107-L3136)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；沿用已核验的 cane.lua、helper／TUNING，复核七皮肤定义／名称、主体 init／clear、装备／卸装与拖尾生成；完整读取七个 FX 工厂，主体仅增效果摘要与链接，详细行为由七篇 FX 页维护。
- 游戏运行：未运行；未覆盖全部换肤调用时序、额外装备栏、晚加入或实际网络／视觉表现，资源仅核验声明与使用。无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
