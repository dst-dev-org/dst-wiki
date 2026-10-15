---
title: "废墟落石碎屑与炼金打印舱皮肤 FX（ruins_cavein_obstacle_rubble_fx、researchlab2_pod_fx）"
created: "2026-11-04"
updated: "2026-11-07"
type: prefab
summary: "两个宿主专属的纯表现注册：遗迹落石障碍采完后在原地生成的碎屑（联网、以自身为声源播碎裂声、animover 自删），以及炼金引擎「炼金打印舱」皮肤挂在宿主下的待机装饰层（DECOR/NOCLICK、0 组件、由皮肤清除函数移除）。"
aliases: ["ruins_cavein_obstacle_rubble_fx", "researchlab2_pod_fx", "废墟落石碎屑", "落石碎屑特效", "炼金打印舱特效", "实验室吊舱皮肤特效", "遗迹落石碎屑"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/ruins_cavein_obstacle.lua", "prefabs/minotaur.lua", "prefabs/scienceprototyper.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "components/highlight.lua", "skin_assets.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 废墟落石碎屑与炼金打印舱皮肤 FX（ruins_cavein_obstacle_rubble_fx、researchlab2_pod_fx）

## 快速判断

- 职责与适用：两个由宿主拉起的纯表现注册，自身都不挂玩法组件。`ruins_cavein_obstacle_rubble_fx` 是遗迹落石障碍被开采完时在原地生成的碎石屑，联网实体、自带 SoundEmitter，碎裂声由宿主调它播放；`researchlab2_pod_fx` 是炼金引擎「炼金打印舱」皮肤的常驻装饰层，父挂宿主、0 组件，客户端只补高亮登记。[碎屑生成](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L24-L43)、[皮肤 FX 工厂](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L268-L295)
- 优先使用：要「落石／矿岩被敲碎」的碎屑表现，选 `ruins_cavein_obstacle_rubble_fx`，照「生成 → 抄宿主颜色 → 设位置 → 用 FX 的 SoundEmitter 播声」四步走；要「皮肤自带、跟随宿主动画的常驻装饰」，走 `researchlab2_pod_fx` 的皮肤 `fx_prefab` 通道，并与 [[reference/prefabs/rabbithouse_yule_glow_fx|rabbithouse_yule_glow_fx]] 对照（后者不做动画镜像）。[OnWork](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L26-L37)
- 重要副作用：碎屑不结算伤害与掉落，战利品由宿主 `lootdropper` 掉；皮肤 FX 不自删，换肤时由 `researchlab2_clear_fn` 统一 `Remove`。[掉落](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L39)、[清除](../../../raw/current/scripts/prefabskin.lua#L3658-L3673)
- 修改边界：两个工厂都不设动画时长、不做定时清理——碎屑只靠 `animover`，皮肤 FX 只靠 clear_fn。4 个 `newfx1`～`newfx4` 符号覆盖只在主模拟执行，客户端符号解析与父子移除未核验。[placer／客户端门禁](../../../raw/current/scripts/prefabskin.lua#L3624-L3633)、[四符号覆写](../../../raw/current/scripts/prefabskin.lua#L3637-L3645)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `ruins_cavein_obstacle_rubble_fx` | `rubblefn(bank, build, anim, icon, tag, multcolour)`：CreateEntity → Transform → AnimState → Network → SoundEmitter → `FX` 标签 → bank／build `rock_ruins`、anim `break_small` → `animover`→`inst.Remove` → SetPristine → 客户端返回 → 主模拟 `persists=false` | 单注册、无 assets；形参被硬编码值忽略。工厂不设颜色、不播声、不建定时器，颜色与声音由生成方补，删除只走 animover。[工厂](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L335-L358)、[注册](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L360-L361) |
| `researchlab2_pod_fx` | `fxfn`：Transform＋AnimState＋Network（无 SoundEmitter）→ bank `researchlab2_pod_fx`、build `researchlab2`、`idle`（未显式传 loop）→ `SetFinalOffset(1)` → bloom `shaders/anim.ksh` → `DECOR`＋`NOCLICK` → SetPristine → 客户端装 `OnEntityReplicated` 后返回 → 主模拟 `persists=false` | 0 组件、无 assets／prefabs 参数；未加 `FX`，源码注释说明 `FX` 会触发 mouseover。客户端回调只做高亮登记。[工厂](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L268-L295)、[注册](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L298-L304) |

名称：`ruins_cavein_obstacle` 命中 `STRINGS.NAMES.RUINS_CAVEIN_OBSTACLE`＝「块状废墟」，`researchlab2` 命中 `STRINGS.NAMES.RESEARCHLAB2`＝「炼金引擎」；皮肤 `researchlab2_pod` 命中 `STRINGS.SKIN_NAMES.researchlab2_pod`＝「炼金打印舱」（皮肤名，不是 NAMES）。两个 FX 注册名在简中 `STRINGS.NAMES` 段（338465–353135）都没有键，标题与别名中的中文是已核验效果描述，非官方译名。[块状废墟](../../../raw/current/scripts/languages/chinese_s.po#L348859-L348862)、[炼金引擎](../../../raw/current/scripts/languages/chinese_s.po#L348594-L348597)、[炼金打印舱](../../../raw/current/scripts/languages/chinese_s.po#L406514-L406517)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 宿主 Prefab：`ruins_cavein_obstacle` | `baserock_fn` 建可开采落石（MINE，工作次数取 `TUNING.ROCKS_MINE`），`fn` 再挂 `inst.fall` 与 `loot1` 掉落表；碎屑只是它被采完时的表现。 | 主模拟；`charge_barrier` 让 Boss 冲撞该障碍时被挡下／眩晕，不参与碎屑逻辑。 | [构造](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L247-L333)、[冲撞](../../../raw/current/scripts/prefabs/minotaur.lua#L258-L258) |
| 生成点：`ruins_cavein_obstacle:OnWork` | `workleft<=0` 时在宿主位置生成碎屑，把宿主 `MultColour` 抄给碎屑并 `SetPosition(pt:Get())`；随后宿主自己播 `break_big`／`break_small`，再用碎屑的 SoundEmitter 播 `ancientguardian_rework/environment/pillar_break`，接着掉战利品、`animover`→`ErodeAway`。 | 主模拟；全树唯一 `SpawnPrefab("ruins_cavein_obstacle_rubble_fx")` 就在这里。 | [L26-L41](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L26-L41) |
| 宿主来源：`minotaur:startobstacledrop` | `startobstacledrop` 需 `spawnlocation` 存在且 Boss 距出生点平方距离 <25²；外层随机 3～4 次尝试，每次最多采样 8 个候选点，除可通行还检查附近 `PILLAR_MUST_TAGS`（quake_on_charge）实体；找到有效点才排延迟生成，故不保证最终落下 3～4 个障碍。生成时先把宿主放到出生点、再调 `obstacle.fall(obstacle, 落点)`；碎屑要等玩家开采才出现。 | 主模拟；全树唯一 `SpawnPrefab("ruins_cavein_obstacle")`。 | [L344-L378](../../../raw/current/scripts/prefabs/minotaur.lua#L344-L378)、[调用](../../../raw/current/scripts/prefabs/minotaur.lua#L207-L210) |
| 关联 Prefab：`warningshadow` | 宿主下落时在落点生成并随高度缩放（`Lerp(1.5, 4.5, height/35)`），宿主移除时一并清；与碎屑没有生成关系，分工见 [[reference/prefabs/houndwarning_lvl1|houndwarning_lvl1]]。 | 主模拟生成、联网显示。 | [L219-L238](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L219-L238) |
| 关联 Prefab：`ground_chunks_breaking` | 宿主落到洞口时由 `_BreakDebris` 生成；碎屑不走这条分支，两者是不同宿主事件下的两种碎块表现，覆盖见 [[reference/prefabs/groundpoundring_fx|groundpoundring_fx]]。 | 主模拟；洞口分支。 | [L66-L71](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L66-L71)、[洞口判断](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L190-L191) |
| 皮肤来源：`researchlab2_pod` | 皮肤条目同时声明 `prefabs`（预载，L10092）与 `fx_prefab = { "researchlab2_pod_fx" }`（L10095）；`CreatePrefabSkin` 把 `fx_prefab` 存进 `SKIN_FX_PREFAB`，挂载由宿主 init_fn 消费。 | 皮肤生效即生成，无季节／事件门控；init_fn 的调用点在引擎侧，脚本内只有赋值。 | [皮肤](../../../raw/current/scripts/prefabs/skinprefabs.lua#L10087-L10098)、[映射](../../../raw/current/scripts/prefabskin.lua#L4094-L4095) |
| 宿主挂载：`researchlab2_init_fn` | placer 只设皮肤外观，客户端直接返回；主模拟 `SetSkin`＋两个 `OverrideSymbol` 后按 `SKIN_FX_PREFAB[build_name]` 生成，`SetParent(inst.entity)`，再对 4 个 `newfx1`～`newfx4` 做 `OverrideItemSkinSymbol`，把 FX 记入 `highlightchildren` 并接管 `_PlayAnimation`／`_PushAnimation`。 | 主模拟生成并父挂；父挂随复制保留，客户端能读到父实体。 | [L3624-L3657](../../../raw/current/scripts/prefabskin.lua#L3624-L3657) |
| 动画镜像：`researchlab2_playanimation`／`pushanimation` | 宿主每次换动画都同步给 `highlightchildren[1]`；`push` 另负责 proximity 循环的闪烁排程，`applyflash` 同时给宿主和 FX 设 `LightOverride`。 | 主模拟；FX 被动跟随，不自行决定动画。 | [L3605-L3622](../../../raw/current/scripts/prefabskin.lua#L3605-L3622)、[闪烁](../../../raw/current/scripts/prefabskin.lua#L3550-L3555) |
| 客户端复制：`OnFXReplicated` | 客户端收到 FX 后若父实体是 `researchlab2`，就补进父的 `highlightchildren`；`Highlight:ApplyColour` 会遍历该表给子实体加同色高亮。 | 仅客户端分支；`NOCLICK` 让 FX 不抢鼠标。 | [回调](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L257-L266)、[高亮消费](../../../raw/current/scripts/components/highlight.lua#L84-L92) |
| 清理：`researchlab2_clear_fn` | 换肤时先 `SetBuild`＋清两个符号覆盖，再 `Remove` 所有 `highlightchildren`（即该 FX），复位动画覆盖并清空表。 | 皮肤清除入口；FX 本身没有定时器。 | [L3658-L3673](../../../raw/current/scripts/prefabskin.lua#L3658-L3673) |
| 资源 | 碎屑用 bank／build `rock_ruins`，动画包 `anim/rock_ruins.zip` 由同文件的宿主注册声明（碎屑注册自身无 assets）；皮肤 FX 用 bank `researchlab2_pod_fx`、build `researchlab2`，皮肤外观来自动态包 `anim/dynamic/researchlab2_pod.zip`／`.dyn`。 | 声明与路径已核；包内容与动画时长未解包。 | [素材](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L1-L5)、[动态包](../../../raw/current/scripts/skin_assets.lua#L5114-L5115) |

## 典型接入

### 复用落石碎屑的生成与发声顺序

- 前提与选择：服务端执行，宿主已有世界坐标。碎屑工厂不接收外部参数、自身无 assets，但注册沿用同文件 `prefabs` 预载表（`rocks`／`flint`／`rock_break_fx`／碎屑自身，[L8-L14](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L8-L14)）；FX 工厂不主动生成这些依赖实体。适合任何「岩石／矿体被敲碎」的场合；战利品或伤害要在宿主侧另算，别指望这个 FX。
- 生效与撤销：服务端生成并 `persists=false`；联网实体，客户端由网络复制。工厂在 SetPristine 前就注册了 `animover`，两端共用这一条工厂内唯一的自删路径（客户端实际触发时机未运行核验），不要额外加定时器。
- 原版依据：`OnWork` 的顺序是「生成 → 抄颜色 → 设位置 → 宿主破动画 → 用 FX 发声 → 掉落」。[L26-L41](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L26-L41)

```lua
-- 服务端：宿主采完时生成碎屑、继承宿主明暗，并让碎裂声从碎屑位置发出
local function SpawnCaveinRubble(host)
    local pt = host:GetPosition()
    local fx = SpawnPrefab("ruins_cavein_obstacle_rubble_fx")
    fx.AnimState:SetMultColour(host.AnimState:GetMultColour()) -- 返回 r, g, b, a 四个值
    fx.Transform:SetPosition(pt:Get())
    fx.SoundEmitter:PlaySound("ancientguardian_rework/environment/pillar_break")
    return fx
end
```

- 调整与验证：颜色与声音都落在碎屑实体上，换宿主时要同步这两步；`animover` 依赖动画正常播完，脚本没有兜底定时，改动画或换 bank 时先确认事件仍会触发（动画内容未解包，未运行验证）。

## 常见陷阱与限制

- **误区：看到碎屑就以为掉落已结算**：`SpawnPrefab` 排在 `DropLoot` 之前，碎屑自身没有 `lootdropper`；只有 `workleft<=0` 才触发，采集中断不会生成。[顺序](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L28-L41)
- **边界：碎屑动画不跟宿主的 `break_big`／`break_small` 分支**：1／3 版落石播 `break_big`，碎屑固定播 `break_small`，两种大小的落石碎屑外观相同。[宿主分支](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L32-L36)、[碎屑动画](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L345-L347)
- **边界：颜色只在生成时抄一次**：工厂不设 `MultColour`，`SetMultColour` 由生成方在服务端调用；客户端副本是否同色（AnimState 颜色是否随网络复制）未运行核验。[生成方设色](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L29)
- **边界：碎屑只带 `FX` 标签**：没有 `NOCLICK`／`DECOR`／`NOBLOCK`，按标签筛选时与常带 `NOCLICK` 的皮肤层、提灯类 FX 不同。[标签](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L343)
- **边界：皮肤 FX 是数据挂载，不是独立生成点**：全树只有 `fx_prefab` 这一处来源，`prefabs` 表只是预载；宿主 fn 里没有 `SpawnPrefab("researchlab2_pod_fx")`。[皮肤条目](../../../raw/current/scripts/prefabs/skinprefabs.lua#L10087-L10098)
- **边界：皮肤 FX 不自删**：宿主被锤掉或被烧掉时，脚本里没有对子 FX 的显式 `Remove`，只靠引擎父子关系，是否需要手动清理未核验。[清除函数](../../../raw/current/scripts/prefabskin.lua#L3658-L3673)
- **边界：`highlightchildren[1]` 被直接索引**：`_PlayAnimation` 与 `applyflash` 都没有空值保护；皮肤激活期间若单独移除 FX，后续换动画或闪烁会访问失效对象。原版只在 `clear_fn` 里成批移除并立即置空，复用时照这个顺序。[镜像动画](../../../raw/current/scripts/prefabskin.lua#L3605-L3608)、[闪烁](../../../raw/current/scripts/prefabskin.lua#L3550-L3555)
- **分工**：`warningshadow`（落点阴影）与 `ground_chunks_breaking`（洞口碎块）分别由 [[reference/prefabs/houndwarning_lvl1|houndwarning_lvl1]] 与 [[reference/prefabs/groundpoundring_fx|groundpoundring_fx]] 维护；「皮肤常驻装饰」的另一种写法见 [[reference/prefabs/rabbithouse_yule_glow_fx|rabbithouse_yule_glow_fx]]。

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394（`raw/current/BUILD.txt`）。两个注册逐项核对工厂、端、生成点、父挂与清理；全 `raw/current/scripts` 反查 `ruins_cavein_obstacle_rubble_fx`（依赖表 L13、生成 L28、注册 L361 三处）与 `researchlab2_pod_fx`（bank L275、注册 L304、皮肤依赖 L10092、`fx_prefab` L10095 四处）；宿主 `ruins_cavein_obstacle` 唯一生成点在 `minotaur` L375。名称按 `languages/chinese_s.po` 的 `NAMES` 段（338465–353135）与 `SKIN_NAMES` 定向检索。未覆盖：动画包内容与时长、客户端皮肤符号解析、父子移除的实际行为、网络时序与游戏内观感。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。
- 结构检查：`validate_page.py` 通过（原记录 89 行／15,240 字节；本轮补正后仍在 standard 档内，最终数见交付记录）；`validate_source_links.py --strict` 通过（40 处行引用、8 个来源文件，0 error／0 warning）；Lua 示例经 `parser_probe.exe --syntax-only` 通过。
- 第 17 轮补正（2026-11-07）：四符号覆写锚点改到 `prefabskin.lua` 的 init_fn 与 placer／客户端门禁；碎屑改为“工厂无外部参数与 assets，但注册沿用同文件 `prefabs` 预载表”；`startobstacledrop` 补 spawnlocation／25² 距离门禁、3～4 次尝试与每次最多 8 个候选，不保证最终数量。

返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]。
