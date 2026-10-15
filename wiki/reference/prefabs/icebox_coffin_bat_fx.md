---
title: "冰冷坟墓开箱蝙蝠特效（icebox_coffin_bat_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "冰冷坟墓皮肤的开箱蝙蝠：宿主节流生成联网代理，客户端播一次性蝙蝠动画与渐弱扇翅声，两级各自回收。"
aliases: ["icebox_coffin_bat_fx", "冰冷坟墓蝙蝠", "棺材冰箱蝙蝠", "开箱蝙蝠特效", "icebox_coffin"]
tags: [prefab, fx, lifecycle, server-client, network-sync]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/icebox_coffin_bat_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "components/container.lua", "easing.lua", "netvars.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 冰冷坟墓开箱蝙蝠特效（icebox_coffin_bat_fx）

## 快速判断

- 职责与适用：[[reference/prefabs/icebox|冰箱]] 万圣夜皮肤“冰冷坟墓”的开箱效果：一次性蝙蝠动画加扇翅声。宿主 onopen 经 1.3 秒节流生成联网代理，主模拟 1 秒后移除；非专服客户端据 variation 另建本地动画实体，动画播完自删。[代理工厂](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L44-L70)、[本地动画](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L16-L42)
- 优先使用：查“棺材冰箱蝙蝠／开箱蝙蝠”选本页；皮肤本体与容器行为查 [[reference/prefabs/icebox#皮肤差异与开箱特效]]；同冰箱的持续霜气族（[[reference/prefabs/icebox_crystal_fx]] 等）是另一条 frost_fx 通道。
- 重要副作用：蝙蝠动画与声音都在本地实体上，代理只承载网络变量与位置；扇翅音量随强度按 outQuad 曲线衰减。[声音](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L9-L14)
- 修改边界：专用服务器不生成本地动画与声音（IsDedicated 门禁）；本地实体无 Network、带 FX／NOCLICK 标签，无组件、SG 或玩法作用。[端门禁](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L54-L57)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `icebox_coffin_bat_fx` | fn → 代理：Transform／Network＋FX 标签＋variation netvar → 非专服一帧后 PlayBatFX → SetPristine → 主模拟随机 variation、1 秒自删、persists=false | 单一注册；本地实体在 PlayBatFX 内另建：bank icebox_coffin、build bat_tree_fx、动画 bat1～3、SetFinalOffset(1)、FX／NOCLICK、animover 自删。[代理](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L44-L70)、[本地](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L16-L42)、[注册](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L72) |

皮肤键 `STRINGS.SKIN_NAMES.icebox_coffin` 为“冰冷坟墓”（官方译名）；未发现 FX 注册名的直接 NAMES 翻译条目，标题中“开箱蝙蝠特效”是已核验效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L398124-L398127)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源：icebox_coffin 皮肤 | fx_prefab 第 2 项映射 open_fx（第 1 项为空串→无 frost_fx），宿主只监听 onopen | 主模拟；placer 分支只设皮肤外观、不接 FX | [[reference/prefabs/icebox#皮肤差异与开箱特效]]；[皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L6955-L6966)、[初始化](../../../raw/current/scripts/prefabskin.lua#L3704-L3731) |
| 宿主节流 | icebox_opened 用宿主 _open_fx_time 做 1.3 秒节流，SpawnPrefab 后把代理 SetPosition 到冰箱位置 | 连续开关 1.3 秒内不重复出蝙蝠；节流不属于本工厂 | [节流](../../../raw/current/scripts/prefabskin.lua#L3678-L3695)、[onopen 推送](../../../raw/current/scripts/components/container.lua#L656) |
| 同步：variation | 主模拟 `variation:set(math.random(3))`；客户端 PlayBatFX 检查大于 0 后播放 `"bat"..variation` | net_tinybyte（3 位无符号）；本地一帧后检查，值未同步到则该次不播 | [netvar](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L52)、[赋值](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L65)、[消费](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L16-L33)、[类型](../../../raw/current/scripts/netvars.lua#L7) |
| 扇翅声 | DoFlutterSound 播 dontstarve/creatures/bat/flap，音量为 outQuad(intensity, 0, 1, 1)，每 5～6 帧强度递减 0.12，不大于 0.12 停止 | 本地实体 SoundEmitter；outQuad 为二次缓出（Penner） | [声音](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L9-L14)、[outQuad](../../../raw/current/scripts/easing.lua#L49-L52) |
| 回收 | 代理固定 DoTaskInTime(1, Remove)；本地实体监听 animover 自删；两实体独立，互不带走 | 无 SG／Brain，无其他退出条件 | [代理自删](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L66)、[本地自删](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L40) |
| 资源 | anim/bat_tree_fx.zip（蝙蝠动画与 build）＋DYNAMIC_ANIM icebox_coffin.zip | 声明与使用已核验，包内容未解包 | [资源声明](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L1-L7) |

## 典型接入

### 复用一次性开箱效果

原版链路：皮肤 fx_prefab 第 2 项 → 宿主 onopen 节流 SpawnPrefab → 代理随机 variation → 客户端一帧后建本地动画。复用时保留三处分工：节流在宿主、随机参数在主模拟代理、表现与声音在本地实体；回收按“代理定时＋本地 animover”双通道。[生成](../../../raw/current/scripts/prefabskin.lua#L3678-L3695)、[代理初始化](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L44-L67)

## 常见陷阱与限制

- **误区：蝙蝠是可交互生物**：本地实体只有 FX／NOCLICK 标签，无 health／combat／SG／Brain；代理同样不可点击，只承载网络变量与位置。[本地实体](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L16-L42)
- **误区：1.3 秒节流属于 FX 工厂**：节流状态 _open_fx_time 存在冰箱宿主上；直接 SpawnPrefab 本注册不经过节流。[宿主](../../../raw/current/scripts/prefabskin.lua#L3680-L3685)
- **误区：代理移除会带走蝙蝠**：本地实体不是代理子实体（无 SetParent），代理 1 秒自删后本地动画仍按自身 animover 回收；两者只是创建时位置同步。[代理自删](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L66)、[本地自删](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L40)
- **边界：专用服务器静默**：IsDedicated 为真时不安排 PlayBatFX，服务端只有代理与 variation；动画与声音全部发生在各客户端本地。[门禁](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L54-L57)
- **边界：variation 为 0 不播放**：PlayBatFX 有大于 0 的门禁；主模拟必设 1～3，客户端在赋值同步前检查则该次静默（时序未实测）。[门禁](../../../raw/current/scripts/prefabs/icebox_coffin_bat_fx.lua#L17)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整 72 行工厂、皮肤定义与映射、宿主 icebox_init_fn／opened 节流、container 开事件推送、easing.outQuad、netvars 类型、皮肤译名检索已核验；全库检索确认该注册名无皮肤系统外消费者。未覆盖：动画／声音资源内容、音量听感、晚加入与多次开关时序实测。
- 游戏运行：未运行。
- 结构检查：validate_page 与 validate_source_links --strict 通过（实际结果见本批交付报告）。

返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
