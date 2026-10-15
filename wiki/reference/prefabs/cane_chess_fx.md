---
title: "城堡手杖棋子拖尾（cane_chess_fx）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "城堡手杖的黑白棋子拖尾：十三个注册、阵营选色、持有者追踪、最短存续与动画退场回收。"
aliases: ["cane_chess_fx", "城堡手杖棋子特效", "手杖棋子拖尾", "黑白棋子特效", "棋子拖尾消失", "cane_chess", "cane_chess_fx_bishop_b", "cane_chess_fx_king_b", "cane_chess_fx_knight_b", "cane_chess_fx_pawn_b", "cane_chess_fx_queen_b", "cane_chess_fx_rook_b", "cane_chess_fx_bishop_w", "cane_chess_fx_king_w", "cane_chess_fx_knight_w", "cane_chess_fx_pawn_w", "cane_chess_fx_queen_w", "cane_chess_fx_rook_w", "城堡手杖棋子拖尾"]
tags: [prefab, fx, lifecycle, server-client, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["prefabs/cane_chess_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 城堡手杖棋子拖尾（cane_chess_fx）

## 快速判断

- 职责与适用：[[reference/prefabs/cane|步行手杖]] 城堡皮肤的棋子动画拖尾，有黑／白各六种形状，不是会战斗的生物。[形状表](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L10-L31)、[本地动画](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L33-L60)
- 优先使用：查“手杖棋子拖尾／黑白棋子特效”选本页；主体选点与生成频率见 [[reference/prefabs/cane#跟随-fx-与拖尾任务如何选择和停止]]。
- 重要副作用：主模拟可按阵营重选形状；追踪手杖／持有者时取消默认退场任务，满足条件可持续停留，退出仍受最短存在时间约束。[阵营](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L78-L98)、[追踪](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L108-L157)
- 修改边界：联网代理保存形状／翻转／退场事件，非专服创建本地动画；本工厂没有玩法组件或 SG／Brain，追踪读取的是外部宿主组件。[构造](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L159-L209)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cane_chess_fx` | MakeChessFX(name,nil,prefabs) → 公共 netvar／监听 → SetPristine → 主模拟接口、任务与随机值 | 通用入口，初始从 12 种组合随机；不是生成编号子实体的包装器。[工厂](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L159-L209)、[注册](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L212-L225) |
| `cane_chess_fx_{shape}_{colour}` | colour=b／w；shape=bishop／king／knight／pawn／queen／rook，共 12 个实际注册 | 固定初始 shape netvar，仍可由 SetOwnerAffinity 重选；注册循环覆盖两色全部形状。[形状与排序](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L10-L31)、[循环注册](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L214-L224) |

皮肤 `STRINGS.SKIN_NAMES.cane_chess` 为“城堡手杖”；本次未发现对应 FX 注册名的直接 NAMES 翻译条目；标题中文部分是已核验的效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L394145-L394147)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源：cane_chess | fx_prefab 第 2 项，由主模拟在合法地面落点生成，再按可用接口传阵营与 owner／cane | 卸装停止后续生成；现存棋子自身追踪退场 | [[reference/prefabs/cane#跟随-fx-与拖尾任务如何选择和停止]]；[定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2929-L2941)、[生成后调用](../../../raw/current/scripts/prefabskin.lua#L3134-L3159) |
| netvar／dirty／net_event | shape、flipped 决定首次本地动画；fadeout 事件切换 chess_pst | 只在无 vfx／无待执行创建任务时创建；已有 vfx 不随 shape dirty 重建 | 独立机制 Wiki 待建；[本地消费](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L62-L76)、[声明](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L169-L185) |
| 外部组件：equippable／inventoryitem／rider | 追踪读取手杖装备状态、直接 owner 及持有者骑乘状态；不是给 FX 挂这些组件 | 主模拟每 0.5 秒检查；owner 必须有效、可见且仍为手杖持有者 | [[reference/components/equippable#关键接口]]；[[reference/components/inventoryitem#关键接口]]；[[reference/components/rider#关键接口]]；[消费者](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L123-L157) |
| 外部技能／SetOwnerAffinity | 主体检查 lunar_favor／shadow_favor，分别调用 LUNAR／SHADOW；FX 选择白／黑色组合 | 主模拟生成后调用；技能标签不直接装到 FX | [[reference/components/skilltreeupdater#关键接口]]；[主体分派](../../../raw/current/scripts/prefabskin.lua#L3137-L3155)、[FX 消费](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L78-L98) |
| 本地动画／资源 | 无 Network 的实体，bank／build=cane_chess_fx，swap_chess 换具体棋子；pre→loop→pst | 父代理 onremove 时移除本地 vfx；动画可在退场结束时反向删除代理 | [本地实体](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L33-L76)、[资源声明](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L1-L3) |
| canetrail 标签 | 主体选点查询附近 shadowtrail／canetrail，避开重叠落点 | 仅已核验此消费者，不穷举标签用途 | [标签](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L166-L167)、[主体消费](../../../raw/current/scripts/prefabskin.lua#L3106-L3131) |

## 典型接入

主体拖尾任务生成代理并传入阵营、持有者／手杖；代理追踪失效后退场并 Remove，本地动画随代理移除清理。本族使用 AnimState，不创建 VFXEffect，无 SetMaxNumParticles／SetSortOffset／SetKillOnEntityDeath 调用，不套用粒子上限或排序参数。[主体调用](../../../raw/current/scripts/prefabskin.lua#L3134-L3159)、[本地清理](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L57-L60)、[追踪／退场](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L100-L157)

### 阵营选色与初次动画

SetOwnerAffinity("LUNAR") 从 w 六形状随机，SHADOW 从 b 六形状随机，其他值从全部 12 形状随机。它会重设 shape，不只是保存阵营字段；普通入口在生成时已随机一次，主体随后可再次选择。[选择逻辑](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L78-L98)、[初始化](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L193-L204)

OnAnimDirty 只在没有本地 vfx 且没有待创建任务时安排创建；已有 vfx 后 shape 改变不在此函数内重新 OverrideSymbol。因此复用应沿原版生成后立即传阵营的调用方式，不能承诺任意时刻调用都刷新已显示棋子。[创建门禁](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L62-L69)、[实际调用](../../../raw/current/scripts/prefabskin.lua#L3134-L3159)

### 追踪、最短存在与退场

默认 1.5 秒后请求 FadeOut；TrackOwnerAndCane 取消默认任务并启动每 0.5 秒追踪。手杖有效、已装备、直接 owner 相同且有效可见，并且距离满足条件时保留：步行半径为 2，骑乘为 3；源码顶部未使用的 DISAPPEAR_DIST 常量不能当统一阈值。[默认任务](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L193-L204)、[真实距离与追踪](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L123-L157)

任一追踪条件失败就取消追踪并请求 FadeOut；FadeOut 防重复，必要时等到创建满 1.5 秒。DoFadeOut 推送网络事件，本地播放 chess_pst，animover 删除代理；主模拟另外安排 1.5 秒后兜底 Remove。因此 1.5 秒是最短退场起点，不是所有场景的固定总寿命。[最短时间与事件](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L100-L121)、[动画回收](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L71-L76)、[本地清理关联](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L57-L60)

## 常见陷阱与限制

- **误区：棋子有 equippable／rider 组件**：这些是追踪时读取的手杖／持有者组件，本 FX 未挂载。外部调用 TrackOwnerAndCane 不能随意传无对应组件的实体。[读取条件](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L123-L157)
- **误区：卸装时立刻清空地面棋子**：主体停止生成，既有棋子下一次追踪发现条件失效后进入退场；还有最短存在时间和动画过程。[主体停止](../../../raw/current/scripts/prefabskin.lua#L3177-L3186)、[退场条件](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L108-L157)
- **误区：b／w 变体永远固定颜色与形状**：仅固定初始 shape，SetOwnerAffinity 仍会按传值重选。[固定初值](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L169-L175)、[重选](../../../raw/current/scripts/prefabs/cane_chess_fx.lua#L78-L98)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整工厂及 13 注册、皮肤定义／译名、主体阵营与追踪接入、netvar 消费及双向清理已核验；主体已补链，未递归审校宿主组件／技能系统全部流程。
- 游戏运行：未运行；未解包棋子动画、未验证晚加入／事件先后、运行时更改阵营的实际显示或骑乘表现。无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
