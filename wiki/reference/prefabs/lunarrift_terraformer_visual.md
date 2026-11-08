---
title: "裂隙地皮改造视觉层与河灯影层（lunarrift_terraformer_visual、floatinglantern_shadow）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "两个宿主附层注册：月裂传送门的地皮改造管理器本身无 AnimState，全靠 lunarrift_terraformer_visual 按 23 秒阶段表演出瓦片改造进度；天灯（河灯）的 floatinglantern_shadow 是引擎父挂的地面影子，动画由本体 PlaySyncedAnimation 成对驱动。"
aliases: ["lunarrift_terraformer_visual", "floatinglantern_shadow", "rift_terraformer", "月裂地皮改造视觉", "裂隙地皮改造者", "河灯影层", "天灯影子", "天灯"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/rift_terraformer.lua", "prefabs/lunarrift_portal.lua", "prefabs/floatinglantern.lua", "recipes.lua", "simutil.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 裂隙地皮改造视觉层与河灯影层（lunarrift_terraformer_visual、floatinglantern_shadow）

## 快速判断

- 职责与适用：两个"宿主附层"注册，分属两条不相干的宿主链：`lunarrift_terraformer_visual` 是隐形管理器 `rift_terraformer` 的**表现层**——本体只有 Transform／Network（无 AnimState），逐瓦片的改造进度全部由这一层在背景层演出来；`floatinglantern_shadow` 是玩法实体 `floatinglantern`（官方名"天灯"）的**地面影子层**，引擎父挂、动画成对同步。[visual 注册](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L455)、[shadow 注册](../../../raw/current/scripts/prefabs/floatinglantern.lua#L497)
- 本体辨析：`rift_terraformer`（[工厂](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L231-L267)）持有 `_terraform_tasks` 任务表、调度地皮变换，本身不可见；`floatinglantern`（[工厂](../../../raw/current/scripts/prefabs/floatinglantern.lua#L378-L455)）是可拾取、可燃、随风飞走的玩法实体，两者玩法均不在本页展开。
- 重要副作用：visual 由 master 侧 timer 组件驱动 `appear→glow→finish` 阶段并排队约 25 条爬行音效；收到 `earlyexit` 时随机 0–1 秒后自删（若还没出现过则先放爆炸）。shadow 自身无任何清理钩子：燃尽收尾时由宿主显式 `ErodeAway`，宿主直接 `Remove` 时是否随引擎父子回收未核验。
- 修改边界：只覆盖 2 个注册的工厂、生成点、阶段驱动／跟随与清理；瓦片改造的 `SetTile`／`undertile` 数据流、裂隙传送门的阶段玩法、天灯的燃料与风系统不展开。`lunarrift_terraformer_explosion` 与 `lunarrift_portal_shadow` 已由 [[reference/prefabs/impact|impact（命中与落点伤害表现及传送门投影 FX 族）]] 覆盖，本页只在交叉点互链。
- 名称：宿主 `STRINGS.NAMES.FLOATINGLANTERN`＝"天灯"（[po](../../../raw/current/scripts/languages/chinese_s.po#L342894-L342897)）；`LUNARRIFT_TERRAFORMER`／`LUNARRIFT_TERRAFORMER_VISUAL`／`FLOATINGLANTERN_SHADOW` 无 `STRINGS.NAMES.*` 键，标题中"裂隙地皮改造视觉层""河灯影层"是已核验的效果描述，**非官方译名**。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lunarrift_terraformer_visual` | `terraformvisualsfn`：Transform／AnimState／SoundEmitter／Network → bank／build `groundcrystal_growth`、`OnGround` 贴地、`LAYER_BACKGROUND`、缩放 1.5 → 标签 `birdblocker`／`FX`／`ignorewalkableplatforms`／`NOBLOCK` → Pristine → 客户端早退 → 主模拟：timer 组件、`_hidden=true`＋创建即 `Hide()`、`_facing_direction="_horizontal"`、监听 `earlyexit` 与 `timerdone`、`persists=false` | 朝向可由宿主 `SetFacing` 归一到 `_horizontal`／`_diagonal` 两档（角度取模四象限）；`SetAppearTime` 把出现时刻与瓦片任务对齐。[工厂](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L406-L452)、[SetFacing](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L362-L374) |
| `floatinglantern_shadow` | `fn_shadow`：Transform／AnimState／Network → bank／build `floatinglantern`、`idle_ground_shadow` 循环、`LAYER_BACKGROUND`＋排序 3 → 标签 `FX` → Pristine → 客户端：`OnEntityReplicated` 登记进父实体 `highlightchildren` → 主模拟：`persists=false` | 影子自己不会动：位置完全随引擎父子，动画全部由本体代播；非专服端复制后自行补挂高亮子链。[工厂](../../../raw/current/scripts/prefabs/floatinglantern.lua#L467-L494)、[客户端登记](../../../raw/current/scripts/prefabs/floatinglantern.lua#L459-L465) |

### visual 的阶段驱动（master timer 链）

`FINISH_DELAY = 23.0`（[常量](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L324)）是"生长动画时长"。宿主排定瓦片任务时把任务剩余时间交给 `SetAppearTime`：

| 阶段 | timer 键 | 动作 | 证据 |
|---|---|---|---|
| 对时 | `do_appear` | `SetAppearTime(time)`：time＜23 → 立即显示并循环 `grow_idle<朝向>`、`do_glow`＝time；否则 `do_appear`＝time−23 | [SetAppearTime](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L343-L359) |
| 生长 | `do_appear` 触发 | 播 `grow<朝向>`＋循环 `grow_idle<朝向>`，`do_glow` 重置为 23 秒；按朝向排队 25／26 条 `floor_creep` 音效时间点 | [appear](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L325-L341)、[音效表](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L295-L322) |
| 发光 | `do_glow` 触发 | 播 `glow<朝向>`＋`glow_idle<朝向>`，`do_finish`＝1 秒 | [glow](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L282-L293) |
| 收尾 | `do_finish` 触发 | 生成 `lunarrift_terraformer_explosion` 覆盖（工厂与伤害见 impact 页，链接见页尾与"快速判断"），播 `glow_dispel<朝向>`，1 秒后 `Remove` | [finish](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L270-L280) |
| 提前退出 | `earlyexit` 事件 → `do_earlyexit` | 随机 0–1 秒延时后：若 `_hidden`（从未显示过）则先显示并生成爆炸覆盖；随后 `Remove` | [helper＋earlyexit](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L377-L389) |

`timerdone` 里 `do_appear`／`do_glow`／`do_finish` 都要求**没有挂起的 `do_earlyexit`**——提前退出排定后其余阶段不再执行（[timerdone](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L392-L404)）。

### shadow 的成对动画同步

本体把两个同步方法挂到自己身上（[装配](../../../raw/current/scripts/prefabs/floatinglantern.lua#L419-L420)）：`PlaySyncedAnimation`／`PushSyncedAnimation` 播本体动画的同时查 `SHADOW_ANIMS` 表（`idle_ground`／`inflate`／`fall_pst`／`place`，[表](../../../raw/current/scripts/prefabs/floatinglantern.lua#L16-L22)）——命中则 `shadow:Show()` 并播 `<名>_shadow`，未命中（如飞行 `idle_air`、放气 `deflate`、下落 `fall_pre`／`fall_loop`）则 `shadow:Hide()`。影子生命周期内的所有形态切换都经这两个方法，本体没有任何一处直接操作影子的 AnimState 之外的接口。

## 关联机制导航

### 生成点与宿主调用

| 生成点 | 涉及注册 | 端、条件与调用 | 证据 |
|---|---|---|---|
| `rift_terraformer.lua` `terraformer_addtask` | `lunarrift_terraformer_visual` | 主模拟；`not is_revert`（正向改造）时生成，定位到瓦片中心、`SetAppearTime(time)`、可选 `SetFacing`；全库唯一生成点。同瓦片已有任务时先推 `earlyexit` 给旧 visual | [生成](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L116-L124)、[覆盖旧任务](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L99-L108) |
| `lunarrift_portal.lua` `make_terraformer_proxy` | `rift_terraformer`（宿主） | 主模拟；裂隙传送门按阶段调用 `do_portal_tiles` 时惰性生成代理（`SpawnPrefab` 后监听其 onremove）；全库唯一生成点 | [生成](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L47-L56)、[瓦片任务](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L75-L94) |
| `floatinglantern.lua` `fn` 主模拟段 | `floatinglantern_shadow` | 主模拟；河灯创建即生成影子并父挂＋登记 `highlightchildren`；全库唯一生成点。本体进入世界走制作配方（`recipes.lua`：幸运金块×4，`PERDOFFERING_ONE`），脚本层无 `SpawnPrefab("floatinglantern")` | [生成＋父挂](../../../raw/current/scripts/prefabs/floatinglantern.lua#L415-L417)、[配方](../../../raw/current/scripts/recipes.lua#L1098) |

### 层、跟随与端模型

| 对象 | 本组注册如何使用 | 条件与边界 | 证据 |
|---|---|---|---|
| visual 贴地层 | `OnGround`＋`LAYER_BACKGROUND`＋缩放 1.5， bank／build `groundcrystal_growth` | 动画在主模拟播放、随联网实体复制；无客户端本地子实体；音效由主模拟 `DoTaskInTime` 排队（随联网实体广播） | [工厂动画段](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L414-L419)、[音效排队](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L316-L322) |
| shadow 引擎父子 | `inst.shadow.entity:SetParent(inst.entity)`：本地坐标 0,0,0，位置随本体；无 Follower、无独立位移 | 本体拾取入库存（INLIMBO）时 Lua 侧不触碰影子，其场景可见性由引擎父子层级决定，本页未逐路径运行核验 | [父挂](../../../raw/current/scripts/prefabs/floatinglantern.lua#L415-L417)、[OnPickup](../../../raw/current/scripts/prefabs/floatinglantern.lua#L284-L292) |
| shadow 端模型 | 联网实体随本体复制；客户端 `OnEntityReplicated` 后把自己插进父实体 `highlightchildren`（本体在主模拟生成时已插过一次） | 专用服务器上影子只剩壳；高亮传播链两端各登记一次 | [客户端登记](../../../raw/current/scripts/prefabs/floatinglantern.lua#L459-L465) |
| visual 存档 | visual `persists=false` 不入存档；宿主 `rift_terraformer` OnSave 只记任务表（瓦片、朝向、剩余时间），读档重新 `AddTerraformTask` 会**再生成新的 visual** | 宿主还有 `OnLongUpdate` 快进与 timer `remove` 自毁（[timerdone](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L224-L228)） | [OnSave／OnLoad](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L173-L200) |

### 清理路径（两注册分述）

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `lunarrift_terraformer_visual` | 三条路径：① 走完阶段表——`do_finish` 放爆炸覆盖后 1 秒自删；② `earlyexit`——宿主覆盖同瓦片任务或 `forcefinishterraforming` 时被推事件，随机 0–1 秒后自删（未显示过则先补爆炸）；③ 宿主直清——`rift_terraformer.OnParentRemoved` 遍历任务表对每个 visual 直接 `Remove` | ①② 都会生成 `lunarrift_terraformer_explosion`（②仅限未显示过时）；③ 无爆炸。`_RevertTile`（地皮还原）也直接生成爆炸覆盖，但 revert 任务**不生成 visual**。[finish](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L270-L280)、[earlyexit](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L377-L385)、[宿主直清](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L145-L156)、[revert 爆炸](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L92-L94) |
| `floatinglantern_shadow` | ① 燃尽坠落收尾：本体落地播完 `fall_pst` 后 4 秒 `DoErodeAway`——对本体与影子**各自**调用 `ErodeAway`（同时侵蚀消散）；② 宿主直接 Remove（如飞行途中睡着即删）时影子是否随引擎父子回收未核验 | 影子自身无 animover／定时兜底；坠落收尾里本体还有 3 秒后加 `NOCLICK`、4 秒侵蚀的缓冲段。[DoErodeAway](../../../raw/current/scripts/prefabs/floatinglantern.lua#L191-L196)、[坠落收尾](../../../raw/current/scripts/prefabs/floatinglantern.lua#L198-L204)、[睡删](../../../raw/current/scripts/prefabs/floatinglantern.lua#L226-L231) |

## 典型接入

### 复刻"管理器隐形＋视觉层对时"的进度演出

- 前提与选择：需要按服务器侧任务时刻表演出进度（地皮改造、建造、事件倒计时）时，把可见层与任务持有者拆成两个实体：管理器持有任务表并可存档，visual 只吃 `SetAppearTime` 对时。`earlyexit` 事件是任务被覆盖／强制完成时的取消通道。
- 生效与撤销：主模拟生成 visual 并对时；任务取消时推 `earlyexit` 而不是直接 Remove（保留"没出现过就补爆炸"的演出语义）；强制完成时对 visual 推 `earlyexit` 并立刻执行任务回调。
- 原版依据：[addtask](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L96-L136)、[forcefinish](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L159-L170)。

```lua
local function ScheduleVisualizedTask(manager, tx, ty, time)
    local visual = SpawnPrefab("lunarrift_terraformer_visual")
    visual.Transform:SetPosition(tx, 0, ty)
    visual:SetAppearTime(time)
    manager._tasks[tx .. " " .. ty] = { visual = visual, time = time }
end
```

- 调整与验证：替换点是任务表结构与完成回调；`SetFacing` 需要传二维朝向数组，角度会被归到水平／斜向两档。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：`rift_terraformer` 是可见装置。** 它连 AnimState 都没有，标签带 `FX`／`NOBLOCK`，纯任务管理器；瓦片上能看到的一切（生长水晶光膜、爬行音效、爆炸覆盖）都来自 `lunarrift_terraformer_visual` 或 `lunarrift_terraformer_explosion`。[宿主工厂](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L231-L246)
- **误区：`OnParentRemoved` 是引擎钩子。** 它只是宿主起的方法名——唯一调用点是 `lunarrift_portal.on_portal_removed` 里的**显式调用**（随后 portal 会重建代理并排还原任务）；引擎不会在父实体移除时自动调它。[显式调用](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L376-L381)
- **边界：visual 的 23 秒是动画时长不是任务时长。** 任务剩余时间超过 23 秒时 visual 先隐藏等待（`do_appear`＝剩余−23），剩余不足 23 秒时直接从 `grow_idle` 循环进入收尾倒计时；把 FINISH_DELAY 当"总时长"会算错对时。[SetAppearTime](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L343-L359)
- **边界：earlyexit 挂起会吞掉后续阶段。** `do_earlyexit` 存在期间 `do_appear`／`do_glow`／`do_finish` 一律不执行；对同一个 visual 既推 earlyexit 又等 finish 会得到 earlyexit 的结果（随机延时自删＋可能补爆炸）。[timerdone 门控](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L392-L404)
- **边界：shadow 不跟随飞行。** `idle_air`／`fall_pre`／`fall_loop` 不在 `SHADOW_ANIMS` 表里，飞行全程影子隐藏，落地 `fall_pst` 才重新显示；给本体单独 `AnimState:PlayAnimation` 会绕过成对同步导致影期错位。[SHADOW_ANIMS](../../../raw/current/scripts/prefabs/floatinglantern.lua#L16-L22)、[同步方法](../../../raw/current/scripts/prefabs/floatinglantern.lua#L53-L75)
- **边界：shadow 无自身清理。** 除燃尽收尾的显式 `ErodeAway(inst.shadow)` 外，影子没有自身清理；宿主直接 `Remove` 时是否随引擎父子回收未核验。本体 `OnPickup` 不处理影子，入库存期间影子的场景状态也由引擎父子层级决定（未运行核验）。[DoErodeAway](../../../raw/current/scripts/prefabs/floatinglantern.lua#L191-L196)、[OnPickup](../../../raw/current/scripts/prefabs/floatinglantern.lua#L284-L292)
- **边界：本体河灯没有脚本生成点。** 只能通过制作配方进入世界（作祟／风把点燃的灯吹上天）；把它当"世界里随手能捞的实体"检索会扑空。[配方](../../../raw/current/scripts/recipes.lua#L1098)、[反查记录](#验证记录)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/rift_terraformer.lua`（两工厂、addtask／forcefinish／OnParentRemoved／OnSave／OnLoad／OnLongUpdate 全链、阶段函数与音效时间表）与 `prefabs/floatinglantern.lua`（本体工厂、影子工厂、成对动画方法、飞行／坠落／燃尽全链）；按需读取 `prefabs/lunarrift_portal.lua` 的 `make_terraformer_proxy`／`do_portal_tiles`／`on_portal_removed`／`portal_forcefinishterraforming` 与 timer `remove` 排定，`recipes.lua` 天灯配方行，`simutil.lua` 的 `ErodeAway`，`tuning.lua` 的 `FLOATINGLANTERN_*`。全 `raw/current/scripts` 反查：`lunarrift_terraformer_visual` 命中 3 处（依赖表 L8、生成 L118、注册 L455，均在 `rift_terraformer.lua`）——唯一生成点、无动态拼名；`rift_terraformer` 命中 `lunarrift_portal.lua` 依赖表（L23）与生成（L49）、`prefablist.lua`、存档升级与森林翻新代码（不生成实体）；`floatinglantern_shadow` 命中 3 处（依赖表 L13、生成 L415、注册 L497，均在 `floatinglantern.lua`）——唯一生成点；`floatinglantern` 除自身外命中 `recipes.lua`（配方）、`recipes_filter.lua`、`prefablist.lua`，无 `SpawnPrefab`。
- 与既有页的交叉：`lunarrift_terraformer_explosion`（本文件 L92／L275／L381 生成）与 `lunarrift_portal_shadow`（`lunarrift_portal.lua` 内）已由 [[reference/prefabs/impact|impact 页]] 覆盖，本页只在阶段收尾与 revert 处互链，未重复覆盖其工厂与伤害。天灯燃烧子 FX `floatinglanternfire` 已由 [[reference/prefabs/fire|火焰、火光与烟雾特效族]] 覆盖，本页不展开。
- 名称核验：`languages/chinese_s.po` 定向检索——`FLOATINGLANTERN`＝"天灯"（锚点见"快速判断"）；`LUNARRIFT_TERRAFORMER`／`LUNARRIFT_TERRAFORMER_VISUAL`／`FLOATINGLANTERN_SHADOW` 无 `STRINGS.NAMES.*` 键，标题使用效果描述。
- 分档说明：`size_class: standard`——2 个注册分属两条独立宿主链，各有一条阶段／跟随驱动与三条／两条清理路径；无多状态机、无多变体族。实测行数与字节见结构检查。
- 未覆盖：瓦片改造的 `SetTile`／`undertile`／`WORLD_TILES.RIFT_MOON` 数据流与 `TERRAFORM_TILE_REMOVE_*` 清场规则；裂隙传送门的阶段推进与 grazer 生成；天灯燃料／风／`camerafade` 细节与 `floatinglanternfire` 燃烧层（见 fire 页）；`globalmapicon` 地图图标；动画包内容与观感。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/impact|命中与落点伤害表现及传送门投影 FX 族]]（含 `lunarrift_terraformer_explosion` 与 `lunarrift_portal_shadow`）
- [[reference/prefabs/fire|火焰、火光与烟雾特效族]]（含天灯燃烧层 `floatinglanternfire`）
- 返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
