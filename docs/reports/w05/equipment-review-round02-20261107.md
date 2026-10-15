# 装备 Wiki 第二轮检视

源码快照 `26335394`，知识仓基准 `97afb4e`。按[工具与野外实用索引](../../../wiki/reference/prefabs/tools-field-index.md)复审第一轮之外的 12 页，涵盖照明与火源、雨具与移动、生活工具、水上工具、实用与投掷。已独立检视 22/69 页，其中第一轮 10 页已收口，本轮 12 页待修复复核，另有 47 页待检视。

结论为需修复，共 6 项 P1、16 项 P2。逐页对照注册与使用链，涉及外观时区分 Lua 接线和引擎表现。本轮只写检视报告，机械检查留待装备批次收口，未运行游戏。以下 Wiki 行号对应检视时工作树。

## P1

### R01 暗影香炉 6 格范围有战斗侧消费者

`tools-misc.md:48–55,131` 把 `THURIBLE_AOE_RANGE` 列为死键，并将机制限于引诱。远古织影者的 `IsNearShadowLure` 实际以玩家为中心，用这 6 格搜索 `shadowlure`，参与群体目标更新、部分保留目标和缠绕选取。各分支有独立门禁，例如保留目标会考虑近期攻击者。补出战斗侧消费者，与 Brain 中 20 格寻找持炉者、面对落地香炉的路径分开，并同步快速判断和索引第 45 行。 [Wiki](../../../wiki/reference/prefabs/tools-misc.md#L48-L55) · [索引](../../../wiki/reference/prefabs/tools-field-index.md#L45) · [目标更新](../../../raw/current/scripts/prefabs/stalker.lua#L184-L207) · [保留目标](../../../raw/current/scripts/prefabs/stalker.lua#L306-L312) · [缠绕选取](../../../raw/current/scripts/prefabs/stalker.lua#L529-L541) · [跟随门禁](../../../raw/current/scripts/brains/stalkerbrain.lua#L252-L270)

### R02 小风扇站立制冷示例摘错监听者

`living-tools.md:100–110` 用 `owner:RemoveEventCallback("locomote", inst._onlocomote)`。原版由风扇 `inst` 监听持有者，解绑应使用 `inst:RemoveEventCallback("locomote", inst._onlocomote, owner)`。现例运行后原监听仍在，停止前进可照常停耗、停冷和停转，无法实现示例承诺。若示例要保留原移动态的隔热效果，站立开启时也应设置夏季隔热量，或者说明只改变制冷和耗油。 [Wiki](../../../wiki/reference/prefabs/living-tools.md#L100-L113) · [原注册与解绑](../../../raw/current/scripts/prefabs/minifan.lua#L31-L54) · [事件 API](../../../raw/current/scripts/entityscript.lua#L1249-L1256) · [移动门控](../../../raw/current/scripts/prefabs/minifan.lua#L149-L163)

### R03 小风扇初始热学方向写反

`living-tools.md:37,91,137` 称新实例初次装备时有“外放热窗口”，同时又说只在移动时制冷。工厂在挂 heater 后立即执行 `SetThermics(false, true)`，并给初始夏季隔热。首次装备时，即使还未开始耗油，这两项也可能已在装备栏生效。开始前进后再停下，或者卸装，才会走关闭或清零路径。区分构造、开始耗油和停耗三个状态。示例中的 55 是装备热源判定与温差贡献的参数，环境温度等项仍参与总温度计算。同步索引第 46 行。 [Wiki](../../../wiki/reference/prefabs/living-tools.md#L87-L94) · [陷阱](../../../wiki/reference/prefabs/living-tools.md#L137) · [索引](../../../wiki/reference/prefabs/tools-field-index.md#L46) · [初始化](../../../raw/current/scripts/prefabs/minifan.lua#L119-L134) · [停止与卸装](../../../raw/current/scripts/prefabs/minifan.lua#L39-L65) · [温度计算](../../../raw/current/scripts/components/temperature.lua#L367-L378)

### R04 传送伞找不到目的地仍会进入扣费回调

`umbrella.md:22,78,128` 声称失败不扣燃料。没有合格 pad 时 `RemoteTeleporter:Teleport` 也执行 `onteleportedfn(..., false, nil)`。传送伞的回调不检查 `success`，最多扣一天，余额不足则暂留 `0.00001`，在状态退出时由 `OnStopTeleport` 清空尾数。分别描述找到目的地、无目的地和在调用 `Teleport` 前被拒绝的路径。 [Wiki](../../../wiki/reference/prefabs/umbrella.md#L124-L129) · [动作门禁](../../../raw/current/scripts/actions.lua#L6032-L6043) · [无目标回调](../../../raw/current/scripts/components/remoteteleporter.lua#L123-L137) · [扣费与退出](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L285-L303) · [失败退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24843-L24852)

### R05 渔网 visualizer 由状态机推进

`fishing-tools.md:120` 把 `fishingnetvisualizer` 写成无 SG。工厂明确安装 `SGfishingnetvisualizer`。状态机调用移动、开网、收网和最终拾取接口，并在开网阶段生成水花 FX。保留组件负责捕获和回收的数据流，把阶段推进归回 SG。 [Wiki](../../../wiki/reference/prefabs/fishing-tools.md#L113-L120) · [工厂](../../../raw/current/scripts/prefabs/fishingnetvisualizer.lua#L31-L45) · [状态机](../../../raw/current/scripts/stategraphs/SGfishingnetvisualizer.lua#L24-L85)

### R06 玩家种壳炸漏船板有 PvP 门禁

`utility-and-throwables.md:23,58,69` 把落船未命中就漏水写成通用机制。玩家种壳仅在 `TheNet:GetPVPEnabled()` 分支传 `break_boats=true`。即使启用 PvP，也必须未打到有效目标，且落点满足船板距离阈值。PvE 分支仍结算 AOE，但不走玩家种壳的漏水分支。同步摘要、正文和陷阱。 [Wiki](../../../wiki/reference/prefabs/utility-and-throwables.md#L56-L69) · [漏水门禁](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L64-L71) · [PvP 与 PvE 分派](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L109-L126)

## P2

### R07 火炬缺皮肤 FX 映射时使用空表

`torch.md:76` 写皮肤没有映射时回退 `torchfire`。装备与投掷只在无皮肤时用默认 FX，有皮肤却缺键时走空表。区分燃烧状态和实际生成光效。 [Wiki](../../../wiki/reference/prefabs/torch.md#L75-L77) · [装备](../../../raw/current/scripts/prefabs/torch.lua#L145-L159) · [投掷](../../../raw/current/scripts/prefabs/torch.lua#L279-L297)

### R08 火炬攻击引燃的技能读取缺少组件门禁

`torch.md:90` 需要交代引燃表达式先掷点，失败后直接读取 `attacker.components.skilltreeupdater`。让没有该组件的自定义攻击者持火炬时，该分支存在 nil 调用风险。打火机有显式组件判空，两个入口不可当成同一模板。 [Wiki](../../../wiki/reference/prefabs/torch.md#L88-L90) · [火炬回调](../../../raw/current/scripts/prefabs/torch.lua#L205-L210) · [打火机回调](../../../raw/current/scripts/prefabs/lighter.lua#L231-L239)

### R09 打火机的燃料还会被烹饪直接扣除

`lighter.md:89` 的“燃料只随时间消耗”与后文烹饪冲突。攻击不额外扣油，燃烧计时与烹饪扣油是两个入口。原版门禁下专家厨师每次扣 1%，放开门禁后非专家分支扣 5%。索引第 33 行的“仅装备燃烧”宜明确限定为计时消耗。 [Wiki](../../../wiki/reference/prefabs/lighter.md#L87-L93) · [索引](../../../wiki/reference/prefabs/tools-field-index.md#L33) · [烹饪回调](../../../raw/current/scripts/prefabs/lighter.lua#L286-L299) · [烹饪门禁](../../../raw/current/scripts/components/cooker.lua#L12-L23)

### R10 提灯关着丢下也会重新点亮

`lantern.md:22,88,131` 把丢弃写成保持已点亮状态。落地回调无条件先 off 再 on，是否点亮由剩余燃料决定。另注明只有装备且有持有者时，耗尽才发送 `torchranout`。 [Wiki](../../../wiki/reference/prefabs/lantern.md#L84-L91) · [点亮门禁](../../../raw/current/scripts/prefabs/mininglantern.lua#L64-L89) · [落地回调](../../../raw/current/scripts/prefabs/mininglantern.lua#L121-L124) · [耗尽分支](../../../raw/current/scripts/prefabs/mininglantern.lua#L172-L184)

### R11 提灯本地事件与组件读档顺序写得过强

`lantern.md:68,92` 把 `lantern_on/off` 写成远端客户端也能直接监听，并断言按组件挂载顺序先恢复燃料再重放开关。实际是主模拟本地 `PushEvent`，远端可见的是联网光源和 FX。组件持久化数据由 `pairs(data)` 迭代，源码没有按挂载顺序的保证。写清各组件如何读档及 `turnon` 检查当时余量即可。 [Wiki 事件](../../../wiki/reference/prefabs/lantern.md#L66-L70) · [Wiki 读档](../../../wiki/reference/prefabs/lantern.md#L92) · [事件分发](../../../raw/current/scripts/entityscript.lua#L1312-L1345) · [读档循环](../../../raw/current/scripts/entityscript.lua#L1979-L2013) · [开关恢复](../../../raw/current/scripts/components/machine.lua#L50-L61) · [燃料恢复](../../../raw/current/scripts/components/fueled.lua#L133-L143)

### R12 女王火炬落地光圈取决于 thrower 记录

`queen-torch.md:97` 写成落地默认回 6，只有点了技能才覆盖。应区分新建光实体的 6、已有光实体复用、以及存在 `thrower` 时的 `5 × brightnessmod`。持有者有 skilltreeupdater 即可建立记录，未点光圈技能也能以倍率 1 覆写到 5。 [Wiki](../../../wiki/reference/prefabs/queen-torch.md#L94-L99) · [光圈函数](../../../raw/current/scripts/prefabs/queen_torch.lua#L45-L56) · [投掷](../../../raw/current/scripts/prefabs/queen_torch.lua#L135-L155) · [入袋与丢弃](../../../raw/current/scripts/prefabs/queen_torch.lua#L193-L210) · [初始半径](../../../raw/current/scripts/prefabs/queen_torch.lua#L370-L385)

### R13 蛇年假灯笼有猎踪生成链

`redlantern.md:56` 声称 spawner 的世界布点在数据侧且 scripts 只见声明。猎踪 `GetHuntedBeast` 按活动分支返回 `yots_worm_lantern_spawner`，完成猎踪后经 `SpawnPrefab(beastprefab)` 生成。补入这个动态生成点，并把森林的依赖声明与其区分开。 [Wiki](../../../wiki/reference/prefabs/redlantern.md#L55-L56) · [活动分诊](../../../raw/current/scripts/components/hunter.lua#L369-L378) · [动态生成](../../../raw/current/scripts/components/hunter.lua#L418-L431)

### R14 传送伞以角色附近阈值选台

`umbrella.md:78` 的“镜头外最近／镜头内最远”来自变量名。实际比较角色与 pad 的距离，阈值为角色物理半径加 `TELEPAD_DETECTION_RADIUS=2`，外侧最近优先，找不到再选内侧最远。搜索附近物品时还会排除不可拾取对象。 [Wiki](../../../wiki/reference/prefabs/umbrella.md#L74-L79) · [选台实现](../../../raw/current/scripts/components/remoteteleporter.lua#L90-L129) · [阈值](../../../raw/current/scripts/tuning.lua#L7239-L7241) · [物品筛选](../../../raw/current/scripts/components/remoteteleporter.lua#L41-L56)

### R15 步行手杖名称锚点仍指旧版本行号

`cane.md:31,45` 的两处 .po 引用在当前快照分别命中吹箭和伍迪服装。正文中文名与当前快照一致，更新为主体 `L340100–L340102` 和皮肤 `L394765–L394797`，默认名称映射锚点改为实际的 `GetBasicDisplayName`。 [Wiki 主体名](../../../wiki/reference/prefabs/cane.md#L31) · [Wiki 皮肤名](../../../wiki/reference/prefabs/cane.md#L45) · [主体名称](../../../raw/current/scripts/languages/chinese_s.po#L340100-L340102) · [皮肤名称](../../../raw/current/scripts/languages/chinese_s.po#L394765-L394797) · [显示名称路径](../../../raw/current/scripts/entityscript.lua#L687-L692)

### R16 手杖主体与 FX 专页仍共用效果检索词

`cane.md:7` 及工具索引第 44 行仍把七个具体效果写入主体别名，包括已在 `cane_candy_fx` 名下的“手杖雪花特效”。主体保留“手杖皮肤特效”、皮肤名和映射表，七个效果词归各 FX 专页。此项通过页面和索引文本复现。 [Wiki](../../../wiki/reference/prefabs/cane.md#L7) · [工具索引](../../../wiki/reference/prefabs/tools-field-index.md#L44) · [FX 专页](../../../wiki/reference/prefabs/cane_candy_fx.md#L7)

### R17 浇水的扣水入口需与动作耐久表区分

`living-tools.md:23,75` 的快速判断把工具消耗统一到 `OnUsedAsItem`。浇水壶工厂将 `onspreadprotectionfn` 接到 `onuse`，由该回调直接调用 `finiteuses:Use()`，这条链不依赖 `SetConsumption(POUR_WATER,...)`。把快速判断改成按工具分流，避免复用时重复扣水。 [Wiki](../../../wiki/reference/prefabs/living-tools.md#L19-L24) · [原厂直接扣水](../../../raw/current/scripts/prefabs/wateringcan.lua#L69-L72) · [回调挂载](../../../raw/current/scripts/prefabs/wateringcan.lua#L131-L149) · [动作耐久表](../../../raw/current/scripts/components/finiteuses.lua#L101-L116)

### R18 海钓抛竿与抛竿后的动作门禁不同

`fishing-tools.md:108` 把 `fishing_idle` 写成 CAST/REEL/STOP/CATCH 的统一条件。初次抛竿走 replica 有效、当前 target 为 nil、右键海面或虚拟海洋点且满足距离的路径。已有 target 时才由 `GetFishingAction` 读取 `fishing_idle`，随后按 target 状态选择动作。 [Wiki](../../../wiki/reference/prefabs/fishing-tools.md#L103-L110) · [抛竿候选](../../../raw/current/scripts/componentactions.lua#L2160-L2175) · [已下竿动作](../../../raw/current/scripts/componentactions.lua#L86-L100)

### R19 手持渔网应保留调试生成入口

工具索引第 52 行的“无生成点”超出页面检索范围。全 scripts 字面检索找到 `d_spawnallhandequipment_onstands` 把 `fishingnet` 纳入调试上架列表。未找到常规配方或生成链仍可保留，注明检索范围，区分调试上架、注释配方和世界依赖声明。 [Wiki](../../../wiki/reference/prefabs/fishing-tools.md#L148-L150) · [索引](../../../wiki/reference/prefabs/tools-field-index.md#L52) · [调试清单](../../../raw/current/scripts/debugcommands.lua#L1817-L1852) · [清单消费](../../../raw/current/scripts/debugcommands.lua#L1906) · [调试生成](../../../raw/current/scripts/debugcommands.lua#L14-L26)

### R20 瓶中信概率决定显影或便签

`utility-and-throwables.md:30,42` 把空瓶写成概率产物，并把坏档修正写成“复位”。便签分支立即 `turn_empty`，实际显影成功也会通过 `on_reveal_map_spot_pst` 调同一函数。裂珍珠相关修正是把缺失的 `gotcrackedpearl` 置 true，在标记已经为真时也可直接强制显影。 [Wiki](../../../wiki/reference/prefabs/utility-and-throwables.md#L26-L46) · [强制显影与便签](../../../raw/current/scripts/prefabs/messagebottle.lua#L86-L137) · [显影后事件](../../../raw/current/scripts/prefabs/messagebottle.lua#L174-L183) · [空瓶替换](../../../raw/current/scripts/prefabs/messagebottle.lua#L57-L68)

### R21 投掷火炬落地后可被外力熄灭

`torch.md:22` 及索引第 32 行把落地持续照明写到燃尽，遗漏投掷后可被灭火器、水球等熄灭。地面燃料未空时，`OnExtinguish` 会移除跟随 FX、复位动画并弹起本体。这个路径和燃料耗尽的 `ErodeAway` 应分开。 [Wiki](../../../wiki/reference/prefabs/torch.md#L22) · [索引](../../../wiki/reference/prefabs/tools-field-index.md#L32) · [外部灭火回调](../../../raw/current/scripts/prefabs/torch.lua#L369-L392) · [耗尽分支](../../../raw/current/scripts/prefabs/torch.lua#L254-L265)

### R22 探测杖的目标缓存和最远档需分开描述

`tools-misc.md:71` 将最远档写成 `cold≤400`，容易读成 400 外不再给档。循环在检查距离前就写入档位和 FX，有目标但超出 400 仍保留 cold，ping 按默认 8 秒，强度在 300 外钳为 0。候选表仅在首次查找时从 `Ents` 建立，后续在缓存里筛有效且不在 limbo 的对象。 [Wiki](../../../wiki/reference/prefabs/tools-misc.md#L67-L76) · [建缓存和筛选](../../../raw/current/scripts/prefabs/diviningrod.lua#L24-L48) · [档位与强度](../../../raw/current/scripts/prefabs/diviningrod.lua#L53-L92) · [阈值](../../../raw/current/scripts/tuning.lua#L2248-L2256)

## 逐页结果

| 页 | 待修项 |
| --- | --- |
| 火炬 `torch` | R07、R08、R21 |
| 薇洛的打火机 `lighter` | R09 |
| 晨星锤 `nightstick` | 未发现 P1/P2，核对装备点亮、燃料、电伤、回充和 FX 清理 |
| 提灯 `lantern` | R10、R11 |
| 女王火炬 `queen-torch` | R12 |
| 红灯笼 `redlantern` | R13 |
| 伞族 `umbrella` | R04、R14 |
| 步行手杖 `cane` | R15、R16 |
| 探具与仪式工具 `tools-misc` | R01、R22 |
| 生活工具 `living-tools` | R02、R03、R17 |
| 钓鱼与水上工具 `fishing-tools` | R05、R18、R19 |
| 实用与投掷装备 `utility-and-throwables` | R06、R20 |

## 处置结果（2026-11-07）

R01～R22 逐条对照当前快照源码复核，22 项全部成立并采纳；改 11 页与工具与野外实用叶索引，未运行游戏。

- R01：按实际分支改——洞穴 `stalker` 的 RetargetFn／KeepTargetFn 用 6 格光环排除玩家，`stalker_atrium` 的 FindSnareTargets 跳过光环内玩家；删除“THURIBLE_AOE_RANGE 无消费者”，同步快速判断、陷阱、验证记录与索引第 45 行。
- R02：示例解绑改为 `inst:RemoveEventCallback("locomote", inst._onlocomote, owner)`，并补 `SetInsulation`，说明监听方是 inst、事件源是 owner。
- R03：区分构造（`SetThermics(false,true)`＋夏季隔热 60）、开始耗油、停走／卸装归零三个状态；删“初始外放热窗口”错误命题，索引第 46 行同步。
- R04：传送伞三分支写清——`CanActivate` 拒绝不扣费；无合格目标仍走 `onteleportedfn(false,nil)` 扣最多 1 天，`remote_teleport_in` 退出时 `OnStopTeleport` 清尾数。
- R05：渔网 visualizer 行为改归 `SGfishingnetvisualizer`（casting→opening→retrieving→final_pickup），组件只保留数据流。
- R06：种壳漏水限定 PvP 分支（`break_boats`＋未命中有效目标＋落点距离阈值），摘要、正文、陷阱与索引同步。
- R07：火炬皮肤 FX 改为“有皮肤只取 `SKIN_FX_PREFAB`，缺键为空表”；无皮肤才回退 `torchfire`。
- R08：补 onattack 先掷点、失败后才读 `attacker.components.skilltreeupdater`，并注明打火机同类回调有显式判空，不能照搬。
- R09：打火机燃料区分计时消耗与烹饪扣油（expertchef 1%／放开门禁后非 expertchef 5%），摘要、正文、索引第 33 行同步。
- R10：提灯落地固定先 `turnoff` 再 `turnon`，余量非空即重亮；`torchranout` 限定装备且有持有者时。
- R11：`lantern_on/off` 改为主模拟本地 `PushEvent`；存档顺序改为 `pairs(data)` 无跨组件保证，不能假定 fueled 先恢复。
- R12：女王火炬区分新建光实体 6、装备刷新 5×修正、投掷复用三条路径；有 `skilltreeupdater` 即记录 thrower 并覆写 5。
- R13：YOTS spawner 补猎踪 `GetHuntedBeast` 动态生成点，并把它与世界布点、`forest.lua` 依赖声明分开；sources 补 `components/hunter.lua`。
- R14：传送伞选台改为按传送者距离阈值（物理半径＋2）取舍，变量名不再当镜头判定；补附近物品过滤。
- R15：手杖 .po 锚点改为主体 L340100-L340102、皮肤 L394764-L394797，默认名称路径改 `GetBasicDisplayName`。
- R16：七种效果别名从主体与索引第 44 行移除，保留各 FX 专页与“手杖皮肤特效”。
- R17：快速判断改为按工具分流，浇水壶 `onspreadprotectionfn=onuse` 直接 `finiteuses:Use(1)`，无 `SetConsumption` 表项。
- R18：海钓竿区分首次抛竿（target 为 nil＋海面点）与已下竿（`fishing_idle`＋`GetFishingAction`）两条门禁。
- R19：渔网“无生成点”限定常规玩法，补调试上架清单 `d_spawnallhandequipment_onstands`（第 1851 行）并声明 `debugcommands.lua`。
- R20：瓶中信改为两条路都变空瓶（便签分支立即 `turn_empty`、显影完成走 `on_reveal_map_spot_pst`）；`gotcrackedpearl` 改为置真并强制显影，不是“复位”。
- R21：火炬补落地燃烧态可被灭火器／水球／冰杖外部熄灭，与燃料耗尽 `ErodeAway` 分开；索引第 32 行同步。
- R22：探测杖补“首次从 `Ents` 建缓存、之后只筛有效且不在 limbo”，以及超出 400 仍为 cold、ping 用默认 8 秒、强度 300 外为 0。

校验：11 页 `validate_page.py` 全部通过（tools-misc 24515 B、redlantern 24523 B 最接近上限，压缩的是验证记录与重复表述，未删必要证据）；`validate_source_links.py --strict` 1,071 条引用 0 error 0 warning；`record` 11 页，2 页因新增 sources 刷新（redlantern、fishing-tools）；`validate_page --check-indexes` 导航通过；minifan 修改示例 `parser_probe --syntax-only` 通过。未运行游戏，未修改 raw，未提交或推送。

## 复检补正（2026-11-07，替代上文相应表述）

- R01：改用实际游戏名“复活的骨架（`stalker`）／远古织影者（`stalker_atrium`）”。复活的骨架选敌时排除 6 格光环内玩家；保留目标时仅当对方不是近期攻击者（6 秒记录）才放弃。缠绕筛选（`FindSnareTargets`）对复活的骨架与远古织影者都生效，不是织影者独有。
- R03：正文两处“体温高于 55 就降温”改为“按 (55−体温)×倍率提供降温贡献”，并注明最终体温仍由环境等项决定；索引改为“新实例首次装备即吸冷，卸装后再装备要等前进”。
- R12：装备态投掷走 `TOSS`→`DropItem`→强制卸装，`onunequip` 先移除原光源并灭火；`OnDropped` 的 `IgniteTossed` 再新建光（初始 6），`Launch`→`OnThrown` 覆写 5×修正。原装备光源不会保留，删除“复用原装备光源”的说法。
- R14：快速判断与目的地两处改为“最多扣 1 天”；选台明确为先取阈值外最近的合格台，外侧没有才取阈值内最远的台。
- R13：猎踪分支补“蛇年活动开启且蛇年神龛有效（`IsSnakeShrineActive`）”门禁；撤回“另有数据侧世界布点”的无证据表述，明确 `forest.lua` 仅是 prefab 依赖声明，猎踪是 scripts 内唯一可核验的生成路径。

校验：5 页＋工具叶索引 `validate_page.py` 通过（tools-misc 24548 B、redlantern 24546 B 最紧）；`validate_source_links.py --strict` 539 条引用 0 error 0 warning。未运行游戏，未修改 raw，未提交或推送。
