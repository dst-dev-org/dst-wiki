---
title: "远古吊灯与石像鬼石化特效（archive_chandelier／gargoyle_*_fx）"
created: "2026-11-04"
updated: "2026-11-08"
type: prefab
summary: "档案馆与宝库三盏吊灯＋石像鬼石化五特效的工厂、灯光相位、端与清理；vault_crawler_chandelier 兼作 vault_crawler 生成点。"
aliases: ["archive_chandelier", "vault_chandelier", "vault_crawler_chandelier", "gargoyle_houndatk_fx", "gargoyle_hounddeath_fx", "gargoyle_werepigatk_fx", "gargoyle_werepigdeath_fx", "gargoyle_werepighowl_fx", "远古吊灯", "宝库吊灯", "蠕虫吊灯", "石像鬼石化特效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/archive_chandelier.lua", "prefabs/gargoyles.lua", "prefabs/vault_key_trial.lua", "prefabs/vaultroom_defs.lua", "prefabs/archive_props.lua", "prefabs/cave.lua", "prefabs/hound.lua", "prefabs/pigman.lua", "prefablist.lua", "components/archivemanager.lua", "components/playerprox.lua", "map/static_layouts/rooms/archive_end/archive_end.lua", "stategraphs/SGvault_crawler.lua", "languages/chinese_s.po"]
size_class: complex
reliability: verified
validation: source-reviewed
status: active
---

# 远古吊灯与石像鬼石化特效（archive_chandelier／gargoyle_*_fx）

## 快速判断

- 职责与适用：八个注册分两族。吊灯族三盏共用 `MakeChandelier`：档案馆吊灯（供电开关＋火苗＋环境音）、宝库吊灯（房间集玩家数开关、可读档变体）、宝库蠕虫吊灯（额外带爪音、`vault_crawler` 生成；出生抖屏在 crawler 的 spawn 状态）。石像鬼族五个 `_fx` 共用 `makefx`：石像月岩化定型瞬间的碎石／落尘表现，全族无玩法组件。[吊灯注册](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L541-L543)、[石像注册](../../../raw/current/scripts/prefabs/gargoyles.lua#L344-L348)、[出生抖屏](../../../raw/current/scripts/stategraphs/SGvault_crawler.lua#L230-L238)
- 优先使用：查"档案馆／宝库吊灯怎么亮""拉闸天降蠕虫""石像石化特效"直达本页；吊灯的摆放与机关接线见关联机制导航的 vault_key_trial、vaultroom_defs 两行。
- 重要副作用：灯光是真 `Light`（主模拟下发、客户端调光），不是纯贴图；拉闸时 `vault_crawler_chandelier` 真的生成生物并推事件；火苗子 FX、爪音、石像本地动画只在非专服存在。[灯光端分界](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L171-L188)、[生成点](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L486-L490)
- 修改边界：本页不覆盖石像本体玩法（workable／loot／named／Reanimate）与 `vault_chandelier_broken`／`_decor`；相位判断、生成与清理都在主模拟分支，客户端只读 `_lightphase`／`_lightlerp`／`clawsfx` 三个 netvar。
- 规模档：`complex`——两套工厂（`MakeChandelier` 三注册、`makefx` 循环五注册）、两条生成侧（静态布局／金库房间布局＋试炼机关）与一条玩法耦合（`vault_crawler` 生成），相位与端差异需整体保留。

## 注册与构造

### 吊灯三注册（同一 MakeChandelier 工厂）

`MakeChandelier(name, build, light_params, flamedata, sfxheight, common_postinit, master_postinit, assets, prefabs)` 的公共流程：Transform／AnimState／Light／Network → bloom＋bank/build 同名播 `idle` 循环 → `Light:EnableClientModulation(true)` → FX／NOCLICK／`archive_chandelier` 标签 → 以 OFF 相位初始化 `_currentlight`／`_startlight` 并 `pushparams` → netvar `_lightphase`（dirty 事件 `lightphasedirty`）与 `_lightlerp` → 非专服建 `sfxprop`（父挂、y=`sfxheight`） → 0 帧 `OnInit` → common_postinit → SetPristine → 客户端早退 → 主模拟随机帧、暴露 `updatelight` → master_postinit。[工厂](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L331-L397)、[灯光初值](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L351-L374)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `archive_chandelier` | 工厂参数：build `chandelier_archives`、`LIGHT_PARAMS`（仅 ON／OFF）、`FLAMEDATA`、sfx y=8、无 common_postinit、master `archive_master_postinit`；master 内挂 playerprox 并监听世界供电事件 | 唯一带火焰子 FX 的变体（flame1-4 跟随符号，`chandelier_fire` build）；`playerprox:SetDist(20, 23)` 的 near／far 都回调 `updatelight`（源码注释残留 `--15,17`）。[注册](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L541)、[后置](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L401-L409)、[火焰](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L212-L232) |
| `vault_chandelier` | 同工厂：build `chandelier_vault`、`LIGHT_PARAMS_VAULT`（ON／OFF／DROP 三相）、无 flamedata、sfx y=6、master `vault_master_postinit`；`vaultpowered=true` 走房间集玩家数 | `SetVariation` 播 `idle`／`idle_2` 并写 OnSave／OnLoad；监听 `ms_virtualroomset_playerschanged` 且只认 VAULT；DROP 相位在本变体无触发路径。[注册](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L542)、[后置](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L434-L448)、[变体](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L413-L432) |
| `vault_crawler_chandelier` | 同工厂：build `chandelier_vault2`、`LIGHT_PARAMS_VAULT`、无 flamedata、sfx y=6、common `crawler_common_postinit`（clawsfx netvar）、master `crawler_master_postinit`（vaultpowered、`DropCrawler`） | 三注册中唯一带 `prefabs` 依赖（`vault_crawler`）的变体；`updatelight` 被换成 `crawler_UpdateLight`，掉落中走 DROP 相位；掉落链在 fall 动画结束时生成蠕虫。[注册](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L543)、[依赖](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L17-L20)、[掉落链](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L468-L515) |

### 石像鬼石化五注册（makefx 循环）

`makefx(data)` 为每个数据项注册一个网络外壳，外壳只有 Transform＋Network＋FX 标签：非专服在 0 帧后 `PlayFX` 建本地实体（`Transform:SetFromProxy` 跟随外壳、bank `sculpture_fx`、build `petrified_tree_fx`、播 `data.name..data.anim`、播 `petrified/post` 音、animover 自删）；外壳 SetPristine 后客户端早退，主模拟只把 `persists=false` 并在 1 秒后 Remove。[工厂](../../../raw/current/scripts/prefabs/gargoyles.lua#L219-L279)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `gargoyle_houndatk_fx`／`gargoyle_hounddeath_fx`／`gargoyle_werepigatk_fx`／`gargoyle_werepigdeath_fx`／`gargoyle_werepighowl_fx` | 同一 `makefx` 循环注册（先注册本体再注册 fx）；五者仅数据表动画名不同：`houndatk`／`hounddeath`／`werepigatk`／`werepigdeath`／`werepighowl` | 外观与音效完全相同，数据表只喂 `name`／`anim`；本页只覆盖 `_fx`，同名不带 `_fx` 的本体走 `makegargoyle`。[循环注册](../../../raw/current/scripts/prefabs/gargoyles.lua#L344-L348)、[数据表](../../../raw/current/scripts/prefabs/gargoyles.lua#L281-L342)、[本地层](../../../raw/current/scripts/prefabs/gargoyles.lua#L226-L249) |

八个注册名均无 `STRINGS.NAMES` 条目（`chinese_s.po` 全文检索无匹配），标题"远古吊灯""石像鬼"为已核验效果描述或社区叫法，**非官方译名**。相邻键 `VAULT_CHANDELIER_BROKEN`＝"圣殿华灯"属另一文件 `vault_chandelier.lua` 的破损吊灯，不是本页 `vault_chandelier`；`GARGOYLE_HOUND`／`GARGOYLE_WEREPIG`＝"可疑的月岩"是石像本体经 `SetPrefabNameOverride` 取的名字，也不适用于 `_fx`。[名称键](../../../raw/current/scripts/languages/chinese_s.po#L351184-L351187)、[石像名键](../../../raw/current/scripts/languages/chinese_s.po#L343099-L343107)、[本体取名](../../../raw/current/scripts/prefabs/gargoyles.lua#L173)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`playerprox` | `archive_master_postinit` 挂载，`SetDist(20, 23)` 后 near／far 都指向 `updatelight`；回调第二参 player 未被使用 | 仅 `archive_chandelier`、仅主模拟；20 内算近、23 外才算远 | [挂载](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L401-L409)、[接口](../../../raw/current/scripts/components/playerprox.lua#L165-L180) |
| 世界组件：`archivemanager` | `_updatelight` 读 `GetPowerSetting()`；并监听 `arhivepoweron`／`arhivepoweroff`（原版拼写少一个 c，必须沿用） | 仅 archive 档、仅主模拟；组件缺失时按**通电**处理（`nil` 短路为 true，仅为避免崩溃，玩家靠近即亮） | [_updatelight](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L281-L290)、[监听](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L407-L408)、[[reference/components/archivemanager#关键接口]] |
| 虚拟房间：`GetPlayersInfoForVirtualRoomSetName(VIRTUALROOMSETS.VAULT)` | vault 两档以 VAULT 房间集在线玩家数判通电；同时监听 `ms_virtualroomset_playerschanged` 并按 `data.roomsetname` 过滤 | 仅主模拟；房间集未注册／空房间时恒 OFF（`players` 可为 nil） | [判定](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L281-L289)、[监听](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L441-L447)、[[reference/mechanisms/vault-virtualroom-migration#快速判断]] |
| 世界组件交互：`archive_props` 宝石开关 | 集齐 3 颗宝石 `SwitchPowerOn(true)` 后按 `archive_chandelier` 标签找 10 半径内吊灯逐个 `updatelight(ent)`；取走宝石断电 | 仅主模拟；吊灯标签 `archive_chandelier` 是该刷新链的检索键 | [标签](../../../raw/current/scripts/prefabs/archive_props.lua#L790-L791)、[刷新](../../../raw/current/scripts/prefabs/archive_props.lua#L803-L813)、[断电](../../../raw/current/scripts/prefabs/archive_props.lua#L850-L853)、[开关接口](../../../raw/current/scripts/components/archivemanager.lua#L34-L48) |
| 机关：`vault_key_trial` | 布局期生成 4 个 `vault_crawler_chandelier` 记为 `light1-4`；拉杆命中时 `light:DropCrawler()`；收到 `ms_vaultcrawler_dropped` 后改追踪 crawler 并设为试炼守卫 | 仅主模拟；`light.DropCrawler` 判空是原版同款写法（客户端没有该字段） | [依赖](../../../raw/current/scripts/prefabs/vault_key_trial.lua#L7-L15)、[生成](../../../raw/current/scripts/prefabs/vault_key_trial.lua#L290-L299)、[拉杆](../../../raw/current/scripts/prefabs/vault_key_trial.lua#L141-L159)、[事件转追踪](../../../raw/current/scripts/prefabs/vault_key_trial.lua#L465-L476) |
| 关联 Prefab：`vault_crawler` | 掉落链里 `SpawnPrefab` 到吊灯 x／z（y 固定 0）并 `sg:GoToState("spawn")`；`prefabs` 数组让它随吊灯加载 | 主模拟生成；`spawn` 状态带 `vault_crawler_dropping`／`temp_invincible`／`nofreeze` 等标签 | [生成](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L486-L490)、[状态标签](../../../raw/current/scripts/stategraphs/SGvault_crawler.lua#L231-L232)、[依赖](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L17-L20) |
| 生成点：档案馆房间静态布局 | 布局对象以 `type = "archive_chandelier"` 声明，由静态布局生成器落成实体 | 世界生成期；`cave.lua` 与 `prefablist.lua` 另有加载声明，声明不等于生成 | [布局条目](../../../raw/current/scripts/map/static_layouts/rooms/archive_end/archive_end.lua#L124-L127)、[洞穴依赖](../../../raw/current/scripts/prefabs/cave.lua#L94)、[模块注册](../../../raw/current/scripts/prefablist.lua#L42) |
| 生成点：金库房间布局 `vaultroom_defs` | puzzle1 出口灯、hall 的 `lightvar` 变体、lore1 等 `CreateRoomEntities` 里 `SpawnPrefab("vault_chandelier"):SetVariation(...)` | 房间生成期；变体调用只在主模拟暴露、Lua 字段无专用 netvar；AnimState 跨端表现未运行核验 | [puzzle1](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L222-L223)、[hall](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L378-L386)、[lore1](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L459-L465) |
| 触发链：石像月岩化 | hound／pigman 监听 `moonpetrify` 时随机 `SpawnPrefab` 本体并调 `Petrify()`；本体在 `OnPetrified` 里才 Spawn 对应 `_fx` 并换成月岩 build | 主模拟；本体不忙或休眠时才石化，`_fx` 只在石化完成那一刻出现 | [hound](../../../raw/current/scripts/prefabs/hound.lua#L713-L723)、[pigman](../../../raw/current/scripts/prefabs/pigman.lua#L986-L999)、[fx 生成](../../../raw/current/scripts/prefabs/gargoyles.lua#L68-L76) |
| 关联 Prefab：`gargoyle_*` 本体五注册 | 本体的 `prefabs` 数组声明 `_fx`；本体自带 workable／loot／named／Reanimate，属玩法，Wiki 待建 | 本体不属本页；改玩法不要改 `makefx` | [依赖声明](../../../raw/current/scripts/prefabs/gargoyles.lua#L27-L32)、[本体工厂](../../../raw/current/scripts/prefabs/gargoyles.lua#L192-L202) |
| 对比：[[reference/prefabs/petrified_trunk_break_fx]] | 树木石化是独立工厂，用 `petrified_tree_fx` 同名 bank／build；石像鬼 `_fx` 只借 `petrified_tree_fx` 当 build、bank 为 `sculpture_fx` | 两族生成点互不复用：树木走 evergreens 石化链 | [石像资源](../../../raw/current/scripts/prefabs/gargoyles.lua#L219-L224)、[树石化页](petrified_trunk_break_fx.md) |
| 资源与同步：chandelier 三 build、`chandelier_fire`、`sculpture_fx`、`petrified_tree_fx`；netvar `_lightphase`／`_lightlerp`／`clawsfx` | 火焰、环境音循环 `grotto/common/chandelier_LP`、爪音 `rifts7/vault_claw/open`／`withdraw`、石像 `petrified/post`；`_lightphase` 用 tinybyte 传相位，`_lightlerp` 只在客户端构造时读 | 专服无火苗／爪音／本地石像动画；资源声明不等于资源出现在快照 | [资源头](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L1-L15)、[sfxprop](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L147-L167)、[相位 netvar](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L361-L366)、[爪音](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L452-L466) |

## 典型接入

### 复用"接近＋供电"双条件开关灯（archive 档结构）

- 前提与选择：自家实体已 `AddLight()`，在 master_postinit 里挂 `playerprox` 并把同一个 `updatelight` 同时登记为 near／far 回调；供电读 `archivemanager`，组件缺失时 `nil` 短路为真、按通电处理。选这条是因为它把"玩家接近"与"档案馆是否通电"两个条件分成显式判空，不容易在独服／无档案世界崩。
- 生效与撤销：`SetDist(near, far)` 是迟滞区间；主模拟改 `Light:Enable` 才是真光，客户端只跟着网络状态渲染。卸载时随实体移除，无额外清理。
- 原版依据：[_updatelight](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L281-L302)、[挂载](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L401-L409)、[playerprox 接口](../../../raw/current/scripts/components/playerprox.lua#L165-L180)

```lua
-- 在自有实体的 master_postinit 内；回调第二参 player 与 archive 档一样不参与判断
local function updatelight(inst)
    local prox = inst.components.playerprox
    local manager = TheWorld.components.archivemanager
    local powered = (prox == nil or prox:IsPlayerClose())
        and (manager == nil or manager:GetPowerSetting())
    inst.Light:Enable(powered)
end

inst.updatelight = updatelight
inst:AddComponent("playerprox")
inst.components.playerprox:SetDist(20, 23)
inst.components.playerprox:SetOnPlayerNear(updatelight)
inst.components.playerprox:SetOnPlayerFar(updatelight)
```

- 调整与验证：`SetDist` 两值含义见 playerprox 的 near／far 判定；两个回调要在首次 tick（默认 `10 * FRAMES`）前登记完——若玩家已在范围内而 `onnear` 仍为 nil，`isclose` 会先置真但不触发回调，灯要等玩家离开再回来才亮（[默认周期](../../../raw/current/scripts/components/playerprox.lua#L127-L143)、[调度](../../../raw/current/scripts/components/playerprox.lua#L190-L196)）。原版另有 `archive_props` 的标签刷新链，纯自定义实体不需要。

### 由机关触发天降蠕虫（vault_key_trial 结构）

- 前提与选择：吊灯必须由主模拟 `SpawnPrefab` 并自行保管引用（原版用 entitytracker 记 `light1-4`）；触发时调 `DropCrawler`。选这条是因为它把"灯"和"生成点"拆成两个显式步骤，不会在每次刷新灯光时误生成。
- 生效与撤销：`DropCrawler` 只生效一次（`dropped` 拦截）；吊灯播 `fall`，animover 时在自身 x／z、y=0 生成 `vault_crawler` 并入 `spawn` 状态，随后播 `fall_pst`→`withdraw` 并自删，`persists=false` 不入存档。
- 原版依据：[掉落函数](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L506-L515)、[动画链](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L477-L504)、[机关调用](../../../raw/current/scripts/prefabs/vault_key_trial.lua#L141-L159)

```lua
local function DropCrawlerAt(x, z)
    local light = SpawnPrefab("vault_crawler_chandelier")
    light.Transform:SetPosition(x, 0, z)

    -- 拉闸：吊灯播放 fall，落地时生成 vault_crawler
    if light.DropCrawler ~= nil then
        light:DropCrawler()
    end
    return light
end
```

- 调整与验证：`DropCrawler` 只在主模拟 master_postinit 注入，客户端拿不到该字段，判空必须保留；掉落点用吊灯自身 x／z、y 固定 0，原版没有再按高度偏移。想复用"拉闸→生成"而不带蠕虫时，改用 `vault_chandelier`（无 `DropCrawler`、无掉落链）。

## 常见陷阱与限制

- **误区：`vault_crawler_chandelier` 只是灯**：它同时是 `vault_crawler` 的生成点——`DropCrawler` 播 `fall` 并在 animover 时真生成生物、推 `ms_vaultcrawler_dropped`；不拉闸时它才只是灯。[掉落链](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L477-L515)
- **边界：`vault_chandelier` 的 DROP 相位没有触发路径**：三相表由两档共用，但只有 crawler 档的 `crawler_UpdateLight` 会 `set(DROP)`；`vault_chandelier` 的 `updatelight` 只写 ON／OFF。[相位写入](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L468-L475)
- **死代码：`OnSpawnTask` 不是现役生成逻辑**：它引用 `components.hideout`，但本文件从未 `AddComponent("hideout")`，也没有调用点；别照它给吊灯补生成。[孤立函数](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L272-L279)
- **边界：变体调用只在主模拟暴露**：`SetVariation` 在 master_postinit 暴露、OnSave／OnLoad 也只在主模拟；本文件 netvar 只有 `_lightphase`／`_lightlerp`／`clawsfx`，没有 variation 专用同步通道。客户端没有 `SetVariation` 方法，也不持有 `variation` 字段；但该方法直接改联网实体的 AnimState，不能用“无专用 netvar”推出远端停在默认 `idle`，跨端实际画面未运行核验。[变体](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L413-L432)、[netvar 清单](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L361-L366)
- **边界：专服没有火苗、爪音和本地石化动画**：`sfxprop` 与火焰子 FX 由非专服创建，石像本地层也被 `TheNet:IsDedicated()` 挡住；专服只剩 1 秒空壳。[音效载体](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L368-L372)、[石像专服分支](../../../raw/current/scripts/prefabs/gargoyles.lua#L259-L264)、[兜底删除](../../../raw/current/scripts/prefabs/gargoyles.lua#L272-L273)
- **边界：石像 `_fx` 只在石化完成时出现**：`OnPetrified` 里 Spawn；`Reanimate`（复活）与 `crumble`（挖碎）都不生成该特效，只播本体动画。[石化点](../../../raw/current/scripts/prefabs/gargoyles.lua#L68-L76)、[复活分支](../../../raw/current/scripts/prefabs/gargoyles.lua#L133-L147)
- **误区：本体不是本页对象**：`gargoyle_houndatk` 等 5 个是带 workable／loot（werepig 系另有 named）的玩法本体，`_fx` 才是纯表现；改玩法别动 `makefx`。[本体工厂](../../../raw/current/scripts/prefabs/gargoyles.lua#L192-L217)
- **边界：石像外壳没有 AnimState**：网络外壳只有 Transform＋Network，画面全在本地实体；改外观要改 `PlayFX`，改外壳无效。[外壳](../../../raw/current/scripts/prefabs/gargoyles.lua#L251-L276)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394（实读 `raw/current/BUILD.txt`）。逐文件核验：`archive_chandelier.lua` 543 行全文（三注册与工厂、三套后置、相位／火焰／音效链、掉落链）、`gargoyles.lua` 350 行全文（`makefx`、五注册、数据表、Petrify 生成点与 Reanimate 分支）、`vault_key_trial.lua`（生成／追踪／拉杆）、`vaultroom_defs.lua`（puzzle1／hall／lore1 等布局的 `SpawnPrefab` 点）、`archive_props.lua`（宝石开关与吊灯刷新）、`archivemanager.lua` 与 `playerprox.lua` 接口、`archive_end.lua` 静态布局条目、`cave.lua`／`prefablist.lua` 加载声明、`hound.lua`／`pigman.lua` 月岩化触发链、`chinese_s.po` 名称检索。未覆盖：动画实际画面与帧数、变体在远端客户端的实际观感、空房间集下 `GetPlayersInfoForVirtualRoomSetName` 的返回值实测、灯光衰减的引擎侧表现。
- 游戏运行：未运行。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict`、`parser_probe.exe --syntax-only`（两段 Lua 示例）通过，结果见交付报告。

返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]、[[reference/prefabs/index|Prefab 索引]]；石像鬼等生物表现见 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-10）。
