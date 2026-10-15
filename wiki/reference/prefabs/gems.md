---
title: "宝石族（紫宝石、蓝宝石、红宝石、橙宝石、黄宝石、绿宝石、彩虹宝石）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "prefabs/gem.lua 单工厂 buildgem 注册的七种可拾取宝石：同一实体模板（可堆叠 40、ELEMENTAL 可食、GEM 材料修复者、鼹鼠饵料），颜色差异只有动画、浮力参数与注册名；主要消费是配方材料、蟹王嵌彩与档案馆开关（opal）。"
aliases: ["purplegem", "bluegem", "redgem", "orangegem", "yellowgem", "greengem", "opalpreciousgem", "紫宝石", "蓝宝石", "红宝石", "橙宝石", "黄宝石", "绿宝石", "彩虹宝石", "宝石族", "Iridescent Gem"]
tags: [prefab, item]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/gem.lua", "recipes.lua", "componentactions.lua", "prefabs/crabking.lua", "prefabs/archive_props.lua", "prefabs/tumbleweed.lua", "prefabs/bishop.lua", "prefabs/dragonfly.lua", "prefabs/pigking.lua", "components/quaker.lua", "components/repairable.lua", "components/repairer.lua", "constants.lua", "tuning.lua", "prefabs/mole.lua", "brains/molebrain.lua", "prefabs/rocky.lua", "prefabs/rocky_boss.lua", "prefabs/dustmoth.lua", "prefabs/compostingbin.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 宝石族（紫宝石、蓝宝石、红宝石、橙宝石、黄宝石、绿宝石、彩虹宝石）

## 快速判断

- 职责与适用：`prefabs/gem.lua` 用单一工厂 `buildgem(colour, precious)` 注册 7 个矿物类材料物品：六色常规宝石 `purplegem`／`bluegem`／`redgem`／`orangegem`／`yellowgem`／`greengem` 与 `opalpreciousgem`（彩虹宝石）。七注册共用同一 fn：可堆叠 40、ELEMENTAL 食物类型（喂矿物食性生物）、GEM 材料修复者、鼹鼠饵料与交易接受门槛。[工厂与注册](../../../raw/current/scripts/prefabs/gem.lua#L17-L101)
- 优先使用：查宝石组件、获取与消费路径读本页；配方用量直达 [recipes.lua](../../../raw/current/scripts/recipes.lua)；给新物品借鉴"单工厂多色变体"组装时看注册表；蟹王嵌彩行为见 [crabking.lua](../../../raw/current/scripts/prefabs/crabking.lua#L1360-L1378)。
- 重要副作用：地上宝石带 `quakedebris` 标签，洞穴地震落石落点 2 距离内会将其砸碎移除（[quaker.lua L25](../../../raw/current/scripts/components/quaker.lua#L25)、[L228-L235](../../../raw/current/scripts/components/quaker.lua#L228-L235)）；`sparkletask` 闪光任务只在 master sim 建立。
- 修改边界：颜色差异全部来自工厂闭包参数，原版 fn 内无按颜色的组件分支；给个别宝石加组件用 `AddPrefabPostInit`。`repairer` 的 GEM 材料在原版没有 `repairable` 消费者（见陷阱），模组接入需自行配对。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `purplegem` `bluegem` `redgem` `orangegem` `yellowgem` `greengem` | `buildgem(colour)` → `Prefab(colour.."gem")`；共用闭包 fn：网络段（Transform／AnimState／SoundEmitter／Network、`MakeInventoryPhysics`、bank/build `gems`、播 `colour.."gem_idle"`、`molebait`／`quakedebris`／`gem` 标签、`pickupsound`、`inst.colour` 字段、`MakeInventoryFloatable("small", fp)`）→ `SetPristine` → 客户端返回 → master 段组件（见关联机制导航） | 仅 `colour` 参数不同：动画名前缀、`inst.colour`、浮力参数 [FLOATER_PROPERTIES](../../../raw/current/scripts/prefabs/gem.lua#L6-L15)（orange 0.10/0.82、yellow 0.10/0.85、green 0.05/0.75，其余 0.10/0.80）；组件零差异 |
| `opalpreciousgem` | `buildgem("opal", true)` → `Prefab("opalpreciousgem")`（[L92](../../../raw/current/scripts/prefabs/gem.lua#L92)），与六色共用同一 fn 代码 | `precious=true` 只参与注册名拼接，无额外组件或端分界；差异仅动画前缀 `opal` 与浮力参数 0.10/0.87；外部特殊接线（蟹王／档案馆）都在消费方按注册名判断，不在本工厂内 |

本工厂内部除浮力表和注册列表外没有第二处颜色判断；工厂外部按颜色／注册名的特判在消费方（蟹王嵌彩、档案馆开关，见下节），不在本工厂内。

## 获取与主要消费

### 配方与转化

- **合成获得**：`purplegem` = redgem＋bluegem（[recipes.lua L468](../../../raw/current/scripts/recipes.lua#L468)，`TECH.MAGIC_TWO`）；Wilson 炼金技能转化链 7 条：蓝↔红互转各 1 条、紫、橙（紫×3）、黄（橙×3）、绿（黄×3）、彩虹宝石（五色＋蓝共 6 宝石）[L1465-L1473](../../../raw/current/scripts/recipes.lua#L1465-L1473)。
- **作为材料**（配方数按快照 `recipes.lua` 计）：红／蓝／紫用量最大——法杖 [firestaff L492](../../../raw/current/scripts/recipes.lua#L492)、[icestaff L493](../../../raw/current/scripts/recipes.lua#L493)、护符 [L674-L676](../../../raw/current/scripts/recipes.lua#L674-L676)、远古四色护符／法杖 [L799-L804](../../../raw/current/scripts/recipes.lua#L799-L804)、夜灯 [L434](../../../raw/current/scripts/recipes.lua#L434)、盐盒 [L564](../../../raw/current/scripts/recipes.lua#L564)、弹弓弹药与框架 [L219-L244](../../../raw/current/scripts/recipes.lua#L219-L244)、哑铃 [L258-L262](../../../raw/current/scripts/recipes.lua#L258-L262)、WX-78 模块 [L766-L784](../../../raw/current/scripts/recipes.lua#L766-L784) 等。`opalpreciousgem` 仅 2 条配方：[book_moon L286](../../../raw/current/scripts/recipes.lua#L286) 与彩虹转化；`opalstaff` 无合成配方，拆解返还 1 颗彩虹宝石（[DeconstructRecipe L1637](../../../raw/current/scripts/recipes.lua#L1637)）。
- **掉落获得**：风滚草 6 色低概率（[tumbleweed.lua L157-L162](../../../raw/current/scripts/prefabs/tumbleweed.lua#L157-L162)，chance 0.01–0.04）；发条主教紫宝石必掉（[bishop.lua L41-L53](../../../raw/current/scripts/prefabs/bishop.lua#L41-L53)）；龙蝇 6 色大量掉落（[dragonfly.lua L62-L74](../../../raw/current/scripts/prefabs/dragonfly.lua#L62-L74)）；洞穴地震稀有碎屑红／蓝宝石（[quaker.lua L773-L774](../../../raw/current/scripts/components/quaker.lua#L773-L774)）。未检：遗迹箱子与场景脚本散布（`scenarios/`、`map/` 有命中，未逐条核验概率）。

### 特色消费：蟹王嵌彩与档案馆开关

- **蟹王嵌彩**：玩家对蟹王空巢槽递交宝石积攒 `gemcount`（[crabking.lua L1360-L1378](../../../raw/current/scripts/prefabs/crabking.lua#L1360-L1378)）；蓝宝石扩冰冻距离／加冰墙血量（[L212](../../../raw/current/scripts/prefabs/crabking.lua#L212)、[L910](../../../raw/current/scripts/prefabs/crabking.lua#L910)），紫宝石加冻抗与回血（[L756-L768](../../../raw/current/scripts/prefabs/crabking.lua#L756-L768)）；`opalpreciousgem` 计数时按 6 色＋opal 各加 1，且发光判定视作任意颜色（[L293](../../../raw/current/scripts/prefabs/crabking.lua#L293)）；战利品选择：已嵌珍珠（`gemcount.pearl > 0`）直接出珍珠款饰品；未嵌珍珠时需 opal≥3 且寄居蟹已交出珍珠（`hermit.pearlgiven`）才改珍珠款，其余出普通款（[GetWintersFeastOrnaments L1497-L1505](../../../raw/current/scripts/prefabs/crabking.lua#L1497-L1505)）。
- **档案馆开关 `archive_switch`**：trader 只接受 `opalpreciousgem`，其他宝石报 `WRONGGEM`（[archive_props.lua L683-L690](../../../raw/current/scripts/prefabs/archive_props.lua#L683-L690)）；提交后进入 pickable 供玩家取回（[L821-L842](../../../raw/current/scripts/prefabs/archive_props.lua#L821-L842)），存档恢复时 `spawnopal` 生成一颗并经 `trader:AcceptGift(nil, opal, 1)` 交给开关，恢复嵌宝状态（[OnLoadPostPassSwitch L886-L891](../../../raw/current/scripts/prefabs/archive_props.lua#L886-L891)）。
- **进食与饵料**：ELEMENTAL 食性消费者——鼹鼠 [mole.lua L229](../../../raw/current/scripts/prefabs/mole.lua#L229)、尘蛾 [dustmoth.lua L257](../../../raw/current/scripts/prefabs/dustmoth.lua#L257)、石虾及其头目 [rocky.lua L214-L215](../../../raw/current/scripts/prefabs/rocky.lua#L214-L215)、[rocky_boss.lua L496](../../../raw/current/scripts/prefabs/rocky_boss.lua#L496)；堆肥桶**拒收** ELEMENTAL（`DONT_ACCEPT_FOODTYPES` 清单 [compostingbin.lua L16-L21](../../../raw/current/scripts/prefabs/compostingbin.lua#L16-L21)，[L202](../../../raw/current/scripts/prefabs/compostingbin.lua#L202) 以 `not DONT_ACCEPT_FOODTYPES[foodtype]` 判定，宝石进不了堆肥桶）。`molebait` 标签使鼹鼠 AI 主动捡走地面宝石（[molebrain.lua L64](../../../raw/current/scripts/brains/molebrain.lua#L64)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | `foodtype = FOODTYPE.ELEMENTAL`，`hungervalue = 5` | 七注册同值；只喂矿物食性生物，玩家无法经进食获得收益未核验 | [gem.lua L67-L70](../../../raw/current/scripts/prefabs/gem.lua#L67-L70)；消费者见上文进食小节 |
| 组件：tradable | 无参挂载（`goldvalue` 保持默认 0） | GIVE 动作的组件入口按 `tradable` 组件注册——没有该组件的物品对交易者不出现 GIVE 选项；猪王侧再按 `goldvalue > 0` 过滤，宝石不被猪王接受 | [gem.lua L69](../../../raw/current/scripts/prefabs/gem.lua#L69)、[componentactions.lua L1860-L1870](../../../raw/current/scripts/componentactions.lua#L1860-L1870)、[pigking.lua L604-L605](../../../raw/current/scripts/prefabs/pigking.lua#L604-L605) |
| 组件：repairer | `repairmaterial = MATERIALS.GEM`（常量值 `"gem"`），`workrepairvalue = TUNING.REPAIR_GEMS_WORK = 1` | 七注册同值；原版无 `repairable` 结构配 GEM 材料（全域检索仅本文件命中），接入见典型接入 | [gem.lua L74-L76](../../../raw/current/scripts/prefabs/gem.lua#L74-L76)、[constants.lua L1940-L1958](../../../raw/current/scripts/constants.lua#L1940-L1958)、[tuning.lua L2106](../../../raw/current/scripts/tuning.lua#L2106)；[[reference/components/repairer]]、[[reference/components/repairable]] |
| 组件：stackable／inventoryitem／inspectable／bait | `maxsize = TUNING.STACK_SIZE_SMALLITEM`（40）；`bait` 无参 | 七注册同值；`inventoryitem` 默认配置 | [gem.lua L72-L83](../../../raw/current/scripts/prefabs/gem.lua#L72-L83)、[tuning.lua L83](../../../raw/current/scripts/tuning.lua#L83) |
| API／数据：`quakedebris` 标签 | 地震落石落点扫描 `SMASHABLE_TAGS` 命中后直接 `Remove` | 洞穴地震期间地面宝石会被后续落石砸碎 | [quaker.lua L25](../../../raw/current/scripts/components/quaker.lua#L25)、[L228-L235](../../../raw/current/scripts/components/quaker.lua#L228-L235) |
| API／数据：sparkle 任务 | `OnEntityWake` 起 4–5 秒随机延迟播 `colour.."gem_sparkle"` 再回 idle，唤醒才排程 | 仅 master sim（赋值在 mastersim 分支之后）；动画同步依赖实体 AnimState 复制 | [gem.lua L18-L33](../../../raw/current/scripts/prefabs/gem.lua#L18-L33)、[L87-L88](../../../raw/current/scripts/prefabs/gem.lua#L87-L88) |
| 关联 Prefab：`crabking` | 嵌彩消费方，按注册名判色 | 见获取与主要消费 | [[reference/prefabs/crab_king_bubble1]]；主逻辑 [crabking.lua](../../../raw/current/scripts/prefabs/crabking.lua#L1360-L1378) |
| 关联 Prefab：`archive_switch` | trader／pickable 消费 opal | 仅 opalpreciousgem | [archive_props.lua L1194](../../../raw/current/scripts/prefabs/archive_props.lua#L1194)；档案馆机制 [[reference/components/archivemanager]] |
| 资源：`anim/gems.zip` | 七色共用一个 bank/build `gems` | 工厂 assets 单一声明 | [gem.lua L1-L4](../../../raw/current/scripts/prefabs/gem.lua#L1-L4) |

## 典型接入

### 给原版结构接入宝石修复

- 前提与选择：宝石侧 `repairer`（`work_".."gem` 标签由 [repairer.lua L11-L12](../../../raw/current/scripts/components/repairer.lua#L11-L12) 挂上）已就位，缺的是结构侧 `repairable` 且材料同为 GEM；`repairable.repairmaterial ~= repairer.repairmaterial` 时 [Repair 直接判负](../../../raw/current/scripts/components/repairable.lua#L154-L156)。给目标结构 `AddPrefabPostInit` 补配对即可，不改动宝石。
- 生效与撤销：修复值走 workable 分支（`workrepairvalue=1` 加回工作进度，[repairable.lua L117-L123](../../../raw/current/scripts/components/repairable.lua#L117-L123)），每次修复消耗 1 颗宝石；撤销即移除 PostInit。

```lua
-- modmain.lua：让夜灯可被宝石修复（补回工作进度，每次消耗 1 颗）
AddPrefabPostInit("nightlight", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst:AddComponent("repairable")
    inst.components.repairable.repairmaterial = GLOBAL.MATERIALS.GEM
end)
```

- 调整与验证：玩家需先敲损结构（`workleft < maxwork` 才可修，[repairable.lua L117-L123](../../../raw/current/scripts/components/repairable.lua#L117-L123)）；示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：原版宝石可以修复某些建筑**：`repairer` 挂了 GEM 材料但快照内没有任何结构 `repairable.repairmaterial = MATERIALS.GEM`（全域检索 `MATERIALS.GEM` 仅 gem.lua L75 命中）；当前它只是给模组预留的接口，见典型接入补齐消费侧。
- **边界：地震会销毁地面宝石**：`quakedebris` 标签既是"地震产物"标识，也让落石落点 2 距离内的同类宝石被清除——洞穴里攒宝石要避免留在正在地震的区域。
- **边界：七注册零组件差异**：想给彩虹宝石加特殊组件不能改工厂参数（`precious` 只影响注册名），要用 `AddPrefabPostInit("opalpreciousgem", ...)`；工厂闭包也不提供颜色查询 API，颜色信息在 `inst.colour` 字段（[gem.lua L56](../../../raw/current/scripts/prefabs/gem.lua#L56)）。
- **边界：交易门槛而非交易价值**：`tradable` 组件不等于能换金子（`goldvalue = 0`）；它的作用是让 trader 系（档案馆开关、石虾等）接受递礼，实际收益由各 trader 的接受测试决定。

## 验证记录

- 源码审阅：2026-11-08，steam-buildid:26335394。`prefabs/gem.lua` 全文（工厂、浮力表、组件段、注册）；`recipes.lua` 宝石相关配方与转化按行核验；消费方 crabking（嵌彩计数／蓝紫效果／opal 阈值）、archive_props（开关 trader／pickable／存档补发）、quaker（碎屑掉落与落点砸碎）、tumbleweed／bishop／dragonfly（掉落表行）、mole／molebrain／rocky／rocky_boss／dustmoth（ELEMENTAL 进食消费）、compostingbin（DONT_ACCEPT_FOODTYPES 拒收清单）、repairable／repairer／trader（修复配对与交易门槛）、constants／tuning（常量）核验；`chinese_s.po` 七个 `NAMES.*` 键逐一读取。未核验：玩家能否直接进食宝石（未查玩家 eater 食物组）、宝石在 `scenarios/`／`map/` 布局中的散布概率、动画包内部内容。
- 游戏运行：未运行。
- 结构检查：validate_page 通过（0 issue；95 行、14914 字节，standard 档内）；validate_source_links 通过（60 处引用、0 error／0 warning）；Lua 示例经 parser_probe --syntax-only 通过（只查语法，未执行，临时文件已删）。
- 独立检视修订（2026-11-08）：堆肥桶 ELEMENTAL 表述由「收」改「拒收」（`DONT_ACCEPT_FOODTYPES`，prefabs/compostingbin.lua L16-L21、L202 判定，检视 P1 回源码确认）；Wilson 转化链计数 6→7 条（recipes.lua L1465-L1473，蓝↔红互转按两条计）。
- 外部复检修订（2026-11-08，R10，主代理回源码核实；珍珠款优先级经复检二次勘正）：「工厂外部无按颜色分支」收窄为本工厂内部（消费方特判见蟹王／档案馆）；蟹王战利品选择——已嵌珍珠（gemcount.pearl>0）直接珍珠款，未嵌珍珠才需 opal≥3 且寄居蟹已交出珍珠（hermit.pearlgiven）改珍珠款（crabking.lua L1497-L1505）；档案馆存档恢复补 spawnopal 经 trader:AcceptGift 恢复嵌宝状态（archive_props.lua L886-L891）。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
