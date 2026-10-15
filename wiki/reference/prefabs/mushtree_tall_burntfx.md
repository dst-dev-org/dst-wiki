---
title: "蘑菇树焦壳与绽放焦壳（mushtree_tall_burntfx 等 7 注册）"
created: "2026-11-05"
updated: "2026-11-07"
type: prefab
summary: "mushtree.lua 三种季节蘑菇树与 mushtree_moon.lua 月亮蘑菇树的 7 个动态拼名焦壳 FX：树烧尽回调按本体注册名拼接 _burntfx／_bloom_burntfx，原 build 播 chop_burnt 塌落动画后自删；工厂共用、无玩法组件，读档焦树走同一回调。"
aliases: ["mushtree_tall_burntfx", "mushtree_medium_burntfx", "mushtree_small_burntfx", "mushtree_tall_bloom_burntfx", "mushtree_medium_bloom_burntfx", "mushtree_small_bloom_burntfx", "mushtree_moon_burntfx", "mushtree_burntfx", "蘑菇树焦壳", "菌树烧尽", "焦蘑菇树", "bloom_burntfx", "蘑菇树燃烧余烬"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/mushtree.lua", "prefabs/mushtree_moon.lua", "prefabs/mushtree_webbed.lua", "components/burnable.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 蘑菇树焦壳与绽放焦壳（mushtree_tall_burntfx 等 7 注册）

## 快速判断

- 职责与适用：7 个注册都是**蘑菇树烧尽的塌落余烬表现**，全部由两个宿主文件的 `treeset` 工厂按本体注册名动态拼名生成：`mushtree.lua` 为蓝／红／绿三种季节蘑菇树各注册 `_burntfx`（常态 build 焦壳）与 `_bloom_burntfx`（绽放 build 焦壳）共 6 个，`mushtree_moon.lua` 为月亮蘑菇树注册 `mushtree_moon_burntfx` 1 个（无绽放态）。注册处：6 注册 [mushtree.lua#L861-L862](../../../raw/current/scripts/prefabs/mushtree.lua#L859-L863)、[treeset 三次调用](../../../raw/current/scripts/prefabs/mushtree.lua#L865-L867)；月树 [mushtree_moon.lua#L362](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L339-L363)
- 优先使用：查"蘑菇树烧了以后那个塌落动画"或 `_burntfx`／`_bloom_burntfx` 后缀注册选本页；蛛网蓝蘑菇树的焦壳是独立注册 `mushtree_tall_webbed_burntfx`，见 [[reference/prefabs/mushtree_tall_webbed_burntfx|生物残骸与植物／菌类次级表现]]；酸雨覆盖期的挂树烟是 `acidsmoke_endless`，见 [[reference/prefabs/acidraindrop|酸雨落点烟与蘑菇树酸烟]]。
- 重要副作用：工厂是**联网壳自播动画**——AnimState／SoundEmitter 都在壳上，主模拟生成后随网络复制到各端；`persists=false`，`animover` 自删＋动画长度＋1 帧兜底。源码注释明确说明"不用 fx 代理"：这是本体的动画死亡状态，为避免塌落期间本体仍可交互的边界 bug，烧尽回调**先**生成 FX **再**立即 Remove 本体（[注释](../../../raw/current/scripts/prefabs/mushtree.lua#L371-L376)、[月树同注](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L99-L104)）。
- 修改边界：只覆盖 7 注册的工厂、动态拼名、生成点与清理；蘑菇树的季节绽放切换、酸雨玩法、孢子生成与再生玩法只在生成链需要处点到。 stump 烧尽（`stump_burnt`）只掉 ash 不生成任何焦壳 FX（[mushtree.lua#L21-L24](../../../raw/current/scripts/prefabs/mushtree.lua#L21-L24)、[月树](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L10-L13)）。
- 名称：宿主官方名 `MUSHTREE_TALL`＝"蓝蘑菇树"、`MUSHTREE_MEDIUM`＝"红蘑菇树"、`MUSHTREE_SMALL`＝"绿蘑菇树"、`MUSHTREE_MOON`＝"月亮蘑菇树"（锚点见[验证记录](#验证记录)）；7 个 FX 注册名均无 `STRINGS.NAMES.*` 键，标题"焦壳"等为已核验效果描述，**非官方译名**。

## 注册与构造

### 动态拼名的完整展开

两文件的 `treeset` 每次调用注册 4 个（月树 3 个）Prefab，本页覆盖其中的 FX 后缀项；`name` 与本体同名：

| 树（data） | `_burntfx`（常态焦壳） | `_bloom_burntfx`（绽放焦壳） | build／bank 来源 |
|---|---|---|---|
| `mushtree_tall`（蓝，冬） | `mushtree_tall_burntfx` | `mushtree_tall_bloom_burntfx` | bank `mushroom_tree`；build `mushroom_tree_tall`／`mushroom_tree_tall_bloom` |
| `mushtree_medium`（红，夏） | `mushtree_medium_burntfx` | `mushtree_medium_bloom_burntfx` | bank `mushroom_tree_med`；build `mushroom_tree_med`／`mushroom_tree_med_bloom` |
| `mushtree_small`（绿，春） | `mushtree_small_burntfx` | `mushtree_small_bloom_burntfx` | bank `mushroom_tree_small`；build `mushroom_tree_small`／`mushroom_tree_small_bloom` |
| `mushtree_moon`（月） | `mushtree_moon_burntfx` | —（无绽放机制） | bank `mushroom_tree`；build `mutatedmushroom_tree_build` |

[tree_data 三组](../../../raw/current/scripts/prefabs/mushtree.lua#L309-L351)、[月树 data](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L74-L92)、[拼名生成](../../../raw/current/scripts/prefabs/mushtree.lua#L16-L17)、[月树拼名](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L6-L6)

### 共享工厂 makeburntfx

两文件各有一份同构的 `makeburntfx`（常规树带 `bloom` 参数切 build，月树无此参数）：Transform＋AnimState＋SoundEmitter＋Network → build 取 `bloom and data.bloom_build or data.build`（月树固定 `data.build`）、bank 取宿主同款、播 `chop_burnt` 一次、播 `dontstarve/forest/treeCrumble` → `FX` 标签 → `SetPristine` → 客户端早退（壳即全部表现）→ 主模拟 `persists=false`＋`animover` Remove＋**动画长度＋1 帧兜底 Remove**（注释：防离屏动画睡眠不触发 animover）（[常规树工厂](../../../raw/current/scripts/prefabs/mushtree.lua#L377-L406)、[月树工厂](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L105-L135)）。

与本体工厂的关系：焦壳注册的 `assets` 只带对应 build 的动画包（常态壳带常态 build、绽放壳带 bloom build，[mushtree.lua#L861-L862](../../../raw/current/scripts/prefabs/mushtree.lua#L859-L863)）；工厂不复制本体的生长阶段缩放（0.9／1.0／1.1，[growth_stages](../../../raw/current/scripts/prefabs/mushtree.lua#L260-L307)）——矮／中／高阶段的树烧尽后焦壳一律按 scale 1 播放。

## 关联机制导航

### 生成点与宿主调用

全 `raw/current/scripts` 反查（`grep -rn "mushtree_tall_burntfx" --include=*.lua` 等 7 名逐一）：除两个宿主文件与 `prefablist.lua` 文件清单外零命中——**全部生成点都在宿主文件的 `tree_burnt` 回调内**，动态拼名是唯一入口。

| 注册组 | 生成语句（file:line） | 触发条件 |
|---|---|---|
| 6 个常规树焦壳 | `SpawnPrefab(inst.prefab..(treestate==BLOOMING and "_bloom_burntfx" or "_burntfx"))`：[tree_burnt](../../../raw/current/scripts/prefabs/mushtree.lua#L11-L19) | ①本体 burnable 烧尽回调（`MakeLargeBurnable` 后 `SetOnBurntFn(tree_burnt)`，[接线](../../../raw/current/scripts/prefabs/mushtree.lua#L739-L742)）；②读档 `loaddata.burnt` 为真（OnSave 取保存时 `IsBurning()`，[onsave](../../../raw/current/scripts/prefabs/mushtree.lua#L353-L356)、[onload](../../../raw/current/scripts/prefabs/mushtree.lua#L635-L643)）。生成前先掉 ash＋50% 炭，随后本体立即 Remove；拼名按**生成瞬间的 treestate** 选常态或绽放壳 |
| `mushtree_moon_burntfx` | `SpawnPrefab(inst.prefab.."_burntfx")`：[tree_burnt](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L1-L8) | 同构：本体烧尽（[接线](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L293-L296)）或读档 `burnt`（同样来自保存时 `IsBurning()`，[onsave](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L94-L97)、[onload](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L232-L240)）；月树无 treestate 分支 |

- 拼名输入只认 `inst.prefab`：本体注册名即 `mushtree_tall／medium／small／moon`（`mushtree_tall_stump` 等 stump 变体烧尽走 `stump_burnt`，不生成焦壳）；蛛网树（`mushtree_tall_webbed`，另一文件）烧尽生成的是自己的 `mushtree_tall_webbed_burntfx`，与本页 7 注册无关（[webbed 焦壳注册](../../../raw/current/scripts/prefabs/mushtree_webbed.lua#L339-L340)，机制见 [[reference/prefabs/mushtree_tall_webbed_burntfx|其页]]）。
- 端模型：主模拟生成、联网壳动画随网络复制；客户端不建任何本地实体（`SetPristine` 后早退）。与蛛网焦壳"SoundEmitter 在壳上"一致，塌落声 `treeCrumble` 随壳复制。

### 清理路径

7 注册同一套：主模拟 `animover` Remove＋`GetCurrentAnimationLength()+FRAMES` 兜底 Remove＋`persists=false`（不入存档）；壳无父挂、位置一次性（宿主回调 `SetPosition` 本体坐标后本体已删）。本体侧无字段持有焦壳，无显式 Kill 协议（[常规树](../../../raw/current/scripts/prefabs/mushtree.lua#L399-L402)、[月树](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L128-L131)）。

## 典型接入

### 手动播放一次焦树塌落

- 前提与选择：7 注册位置定位＋自删，直接 `SpawnPrefab` 即用；build 须与想要的树形一致（常规壳 6 个按树种与绽放分 build，月壳固定 `mutatedmushroom_tree_build`）。掉落（ash／charcoal）与本体移除由调用方自理。
- 生效与撤销：主模拟生成即可；壳按动画长度播放，`animover` 或动画长度＋1 帧兜底自删，无需管理。原版依据：[tree_burnt 完整链](../../../raw/current/scripts/prefabs/mushtree.lua#L11-L19)。

```lua
-- 服务端：在 (x,0,z) 播一次蓝蘑菇树常态焦壳
local fx = SpawnPrefab("mushtree_tall_burntfx")
if fx ~= nil then
    fx.Transform:SetPosition(x, 0, z)
end
```

- 调整与验证：替换点为注册名（7 选 1）；想要绽放期焦壳改用对应 `_bloom_burntfx`。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：存在名为 `mushtree_burntfx` 的注册。** 裸基名不注册——7 个注册名全部带树种前缀，任何 `mushtree_*_burntfx` 都要精确到树种（拼名由 `inst.prefab` 提供）；`mushtree_burntfx` 仅作为检索别名收录。[注册清单](../../../raw/current/scripts/prefabs/mushtree.lua#L859-L863)
- **边界：绽放焦壳按烧尽瞬间的 treestate 拼名。** 季节切换有 14 帧延迟动画＋随机 0-3 段延迟（[startchange](../../../raw/current/scripts/prefabs/mushtree.lua#L473-L485)）——燃烧开始时是绽放态、烧尽回调执行时已切回常态的树会生成常态焦壳；treestate 在 `swapbuild` 才真正翻转（[swapbuild](../../../raw/current/scripts/prefabs/mushtree.lua#L451-L463)）。
- **边界：读档焦壳来自保存时仍在燃烧的树。** `onsave` 的 `data.burnt` 取的是 `burnable:IsBurning()`（[L353-L356](../../../raw/current/scripts/prefabs/mushtree.lua#L353-L356)），不是“已烧尽但尚未清理”的标志；读档 `onload` 据此直接走 `tree_burnt` 收尾（掉 ash／炭并生成焦壳）。已正常烧尽的本体早已 `Remove`，通常不会以该字段等待读档。stump 读档走 `stump_burnt`，无焦壳。[onload 分支](../../../raw/current/scripts/prefabs/mushtree.lua#L635-L643)、[月树同构](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L94-L97)
- **边界：焦壳不复制本体缩放与旋转。** 工厂不调用 `SetScale`／`SetRotation`——本体生长阶段的 0.9-1.1 缩放与本体随机色（`SetMultColour` 只在主模拟本体上）都不继承。[缩放对照](../../../raw/current/scripts/prefabs/mushtree.lua#L260-L307)
- **边界：月树壳工厂的第三参数是死参。** `mushtree_moon.lua` 的 `makeburntfx(name, data)` 只有形参，`treeset` 调用处传的 `false` 被忽略（[调用](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L362-L362)）——从常规树文件复制的痕迹，不影响行为。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/mushtree.lua`（treeset／makeburntfx／tree_burnt／onload／growable 缩放段）与 `prefabs/mushtree_moon.lua` 全文；按需读取 `components/burnable.lua` 的 `SetOnBurntFn` 接线与 `onburnt` 消费点（确认回调由燃烧完成触发）、`prefabs/mushtree_webbed.lua` 的注册行（区分归属）。
- 穷举复核：7 个注册名逐一全域 `grep -rn "注册名" --include=*.lua` 反查——除 `prefabs/mushtree.lua`／`prefabs/mushtree_moon.lua`（声明＋拼名生成＋注册）与 `prefablist.lua` 文件清单外零命中；动态拼名输入经核对仅有 `inst.prefab`（`mushtree_tall/medium/small/moon` 与各自 `_stump` 变体），stump 变体烧尽走 `stump_burnt` 不生成 FX。另核 `mushtree_tall_webbed_burntfx`（批 I 页覆盖）注册于 mushtree_webbed.lua，不与本页 7 注册混叠。
- 名称核验：`languages/chinese_s.po` 定向检索——`MUSHTREE_TALL`＝"蓝蘑菇树"、`MUSHTREE_MEDIUM`＝"红蘑菇树"、`MUSHTREE_SMALL`＝"绿蘑菇树"、`MUSHTREE_MOON`＝"月亮蘑菇树"（[po 四键连续段 L346034-L346052](../../../raw/current/scripts/languages/chinese_s.po#L346034-L346052)）；`MUSHTREE_TALL_BURNTFX`／`MUSHTREE_MOON_BURNTFX` 等 7 个 FX 名均无 `STRINGS.NAMES.*` 键。
- 分档说明：`size_class: standard`——7 注册共享一个工厂、一种清理模式，动态拼名用一张表即可完整展开；生成点单一（`tree_burnt` 两条触发），无端差异（纯联网壳），未选 complex 因不存在多状态机或需整体保留的协作链。
- 未覆盖：蘑菇树季节绽放／酸雨／孢子／再生的完整玩法链；`acidsmoke_endless` 挂树烟（归 acidraindrop 页）；燃烧中焰层（burnable FXLevel 5 的 fire FX，归 fire 族页）；蛛网树焦壳（归 mushtree_tall_webbed_burntfx 页）；`small_puff` 与 stump 腐烂链；动画包内容与观感。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

- 第 15 轮补正（2026-11-07）：补 `prefabs/mushtree_webbed.lua` 来源、读档焦壳对应保存时仍在燃烧的树、删除 1～2 秒估计；同步维护日期。

## 相关链接

- [[reference/prefabs/mushtree_tall_webbed_burntfx|生物残骸与植物／菌类次级表现]]（蛛网蓝蘑菇树焦壳的主要页；同"chop_burnt＋双兜底清理"工厂模式）
- [[reference/prefabs/acidraindrop|酸雨落点烟与蘑菇树酸烟]]（同一批宿主树的酸雨覆盖表现）
- [[reference/prefabs/fire|火焰、火光与烟雾特效族]]（燃烧期间的焰层与 burnable FXLevel）
- 返回 [[reference/prefabs/fx-material-index|采集、制作、碎裂与生成烟]]（树）
