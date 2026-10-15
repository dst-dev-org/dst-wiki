---
title: "蟹王伴随表现特效族（crab_king_bubble1）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "fx.lua 通用表 7 注册：蟹王气泡三变体（施法喷泉与炮击海堆冒泡）、水面喷泉（阶段收尾与四种投射物落水共用）、镶嵌宝石闪光、施法结束地面圆环与已停用冰晶；喷泉的伤害与击飞归生成者结算，本族纯表现。"
aliases: ["crab_king_bubble1", "crab_king_bubble2", "crab_king_bubble3", "crab_king_waterspout", "crab_king_shine", "crab_king_icefx", "crabking_ring_fx", "蟹王气泡", "蟹王水柱", "水面喷泉特效", "镶嵌宝石闪光", "施法圆环"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/crabking.lua", "stategraphs/SGcrabking.lua", "prefabs/cannonballs.lua", "prefabs/mortarball.lua", "prefabs/trident.lua", "prefabs/winona_catapult_projectile.lua", "prefabs/deer_fx.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 蟹王伴随表现特效族（crab_king_bubble1）

## 快速判断

- 职责与适用：`fx.lua` 通用表 7 注册——蟹王战各阶段的伴随动画表现：`crab_king_bubble1/2/3` 气泡（施法喷泉与炮击阶段海堆冒泡，生成方随机拼名）、`crab_king_waterspout` 水柱（喷泉收尾、海堆销毁与船炮／迫击炮／三叉戟／投石机落水共用）、`crab_king_shine` 镶嵌宝石闪光、`crabking_ring_fx` 施法结束地面圆环、`crab_king_icefx` 冰晶（当前快照无活跃生成点）。全部纯表现无组件；协议见 [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂·关键流程]]。[条目组](../../../raw/current/scripts/fx.lua#L2156-L2204)
- 优先使用：查"蟹王战某表现何时出／能否借用气泡水柱"读本页；投射物落水复用与命中结算分工见 [[reference/prefabs/cannonball_used|船炮与迫击炮弹命中表现]]；本体战流程与 boss 实体源码入口 [crabking.lua](../../../raw/current/scripts/prefabs/crabking.lua)（Wiki 待建）。
- 重要副作用：仅 `crab_king_waterspout` 条目自带音效，由本地表现实体自播（专服无声）；无玩法作用——船体扣血、漏水、oceanfishable 击飞、海堆销毁全在生成者代码。[音效挂接](../../../raw/current/scripts/prefabs/fx.lua#L43-L50)
- 修改边界：服务端生成代理、各非专服一帧后建本地表现、animover 自删；气泡有海洋判定（不在海面不出；施法喷泉在船上只震屏，炮击海堆冒泡直接不出）；`crab_king_icefx` 无生成点，见陷阱。[回收](../../../raw/current/scripts/prefabs/fx.lua#L98-L102)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `crab_king_bubble1` | 通用表字面量条目 → 加载器打包代理＋一帧后本地实体（公共协议，落点式不父挂） | bank `Bubble_fx`，build `crab_king_bubble_fx`，anim `bubbles_1`，无声，fn=FinalOffset1（抬显示层）。[条目](../../../raw/current/scripts/fx.lua#L2156-L2161)、[fn](../../../raw/current/scripts/fx.lua#L1-L3) |
| `crab_king_bubble2` | 同上协议 | 同 bank/build，anim `bubbles_2`；与 1/3 仅动画不同，生成方随机三选一。[条目](../../../raw/current/scripts/fx.lua#L2163-L2168) |
| `crab_king_bubble3` | 同上协议 | 同 bank/build，anim `bubbles_3`。[条目](../../../raw/current/scripts/fx.lua#L2170-L2175) |
| `crab_king_waterspout` | 同上协议 | 同 bank/build，anim `waterspout`；**唯一带 sound**：`hookline_2/creatures/boss/crabking/waterspout`（本地实体按 sounddelay=0 播放）。[条目](../../../raw/current/scripts/fx.lua#L2177-L2183) |
| `crab_king_shine` | 同上协议 | bank/build `crab_king_shine`，anim `shine`，无声，fn=Bloom（泛光 shader＋抬层）。[条目](../../../raw/current/scripts/fx.lua#L2185-L2190)、[Bloom](../../../raw/current/scripts/fx.lua#L29-L31) |
| `crab_king_icefx` | 同上协议 | 借用无眼鹿资源 bank/build `deer_ice_flakes`，anim `idle`，fn=Bloom；当前无活跃生成点。[条目](../../../raw/current/scripts/fx.lua#L2192-L2197) |
| `crabking_ring_fx` | 同上协议 | bank/build `crabking_ring_fx`，anim `idle`，无声，fn=GroundOrientation（`OnGround` 朝向＋`LAYER_BACKGROUND` 贴地层）。[条目](../../../raw/current/scripts/fx.lua#L2199-L2204)、[fn](../../../raw/current/scripts/fx.lua#L24-L27) |

`languages/chinese_s.po` 定向检索无这 7 个注册名的显示名键（命中仅为 `STRINGS.NAMES.CRABKING*` 本体／炮塔／小怪与皮肤条目）；标题中文为已核验效果描述，非官方译名。七条目均未设 tint／transform／animqueue／父挂标记，代理按默认四朝向（[朝向分支](../../../raw/current/scripts/prefabs/fx.lua#L138-L146)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 7 条目同协议：服务端隐形代理＋各非专服一帧后本地实体，animover 自删、代理 1 秒回收 | 全组同协议 | [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂·关键流程]] |
| 气泡生成者 A：炮击阶段海堆冒泡 | 本体激活 → `SpawnCannons` 找 30 内海堆，每堆两条任务链：冒泡任务挂在**海堆实体**上（0.3~0.4s 后执行一次）、1~3s 后销毁海堆 | 服务端；`"crab_king_bubble"..math.random(1,3)` 随机拼名，**海堆**周围半径≤2 且 `IsOceanAtPoint` 才出；冒泡的"每轮自重排"是死链——重排把 `inst.BubbleSeaStack` 挂到海堆上调（该方法仅蟹王导出，海堆上为 nil，scheduler 对 nil 静默跳过），实际每堆只出初排一枚（Klei 代码缺陷，见验证记录） | [BubbleSeaStack](../../../raw/current/scripts/prefabs/crabking.lua#L1071-L1084)、[任务链](../../../raw/current/scripts/prefabs/crabking.lua#L1114-L1116)、[SpawnCannons](../../../raw/current/scripts/prefabs/crabking.lua#L1103-L1129) |
| 气泡生成者 B：施法喷泉冒泡 | SG `cast_blue_pre` 动画结束在本体周围半径 8 放 6 个 `crabking_geyserspawner`（隐形发生器，Wiki 待建），每个按施法时长把冒泡间隔从 0.2s 递减到 0.01s | 服务端；半径≤6，海面出泡、**船上只震屏不出泡**；间隔递减基准 [TUNING.CRABKING_WAVE_ATTACK_TIMEOUT_TIME=8 秒](../../../raw/current/scripts/tuning.lua#L5419-L5419) | [burble](../../../raw/current/scripts/prefabs/crabking.lua#L1757-L1776)、[放发生器](../../../raw/current/scripts/stategraphs/SGcrabking.lua#L456-L474)、[geyserfn](../../../raw/current/scripts/prefabs/crabking.lua#L1860-L1911) |
| waterspout 生成者 A：喷泉收尾 | 每个发生器 8 秒超时后 `endgeyser`：每轮 8 枚（[CRABKING_DEADLY_GEYSERS](../../../raw/current/scripts/tuning.lua#L5351-L5351)）在半径≤4.5 内、`IsOceanAtPoint` 才出柱；每枚出柱后把 2 内挂 `oceanfishable` 组件的实体转为投射物击飞（含可钓浮渣，不限鱼类） | 服务端；`endgeyser` 每 2s 自重排直到发生器被移除；击飞与扣船血在生成者，非 FX | [endgeyser](../../../raw/current/scripts/prefabs/crabking.lua#L1779-L1858)、[自重排](../../../raw/current/scripts/prefabs/crabking.lua#L1857-L1857) |
| waterspout 生成者 B：海堆销毁 | 炮击阶段每堆 1~3s 后 `DoDestroySeaStack`：**无海洋判定**直接在堆位出柱，随后 `workable:Destroy` 销毁海堆 | 服务端；销毁结算与柱并列 | [DoDestroySeaStack](../../../raw/current/scripts/prefabs/crabking.lua#L1086-L1100) |
| waterspout 生成者 C：投射物落水共用 | 船炮石弹与迫击炮弹 OnHit 真落海分支、三叉戟 `create_water_explosion` 四方向（海瓦片且无船，有船改漏水＋推力）、温娜投石机 `OnHit` 落水分支 `DoOceanFishing` | 服务端；落水判据与船甲板语义见 [[reference/prefabs/cannonball_used#关联机制导航|cannonball_used 页导航]] | [cannonballs](../../../raw/current/scripts/prefabs/cannonballs.lua#L127-L139)、[mortarball](../../../raw/current/scripts/prefabs/mortarball.lua#L145-L163)、[trident](../../../raw/current/scripts/prefabs/trident.lua#L180-L187)、[投石机分支](../../../raw/current/scripts/prefabs/winona_catapult_projectile.lua#L401-L403)、[DoOceanFishing 生成](../../../raw/current/scripts/prefabs/winona_catapult_projectile.lua#L237-L258) |
| shine 生成者：宝石闪光 | `ShineSocketInSlot` 给生成的代理加 `Follower` 并绑定 `gem..slot`，未调用 SetParent；入口四处：镶嵌时（加载路径不重放）、施法收尾紫／蓝、炮击阶段红／黄、施法循环每≥1.5s 蓝色节流重放 | 服务端生成代理；本地动画创建时从代理取变换，源码未证明它持续跟随符号 | [ShineSocketInSlot](../../../raw/current/scripts/prefabs/crabking.lua#L279-L283)、[本地动画](../../../raw/current/scripts/prefabs/fx.lua#L18-L37)、[按色错峰](../../../raw/current/scripts/prefabs/crabking.lua#L285-L298)、[镶嵌](../../../raw/current/scripts/prefabs/crabking.lua#L224-L231)、[收尾紫蓝](../../../raw/current/scripts/prefabs/crabking.lua#L971-L976)、[炮击红黄](../../../raw/current/scripts/prefabs/crabking.lua#L1127-L1128)、[施法节流](../../../raw/current/scripts/stategraphs/SGcrabking.lua#L42-L46) |
| ring_fx 生成者：施法结束 | `EndCastSpell` 在本体位置出环（先移除全部喷泉发生器，再排冰环／冰墙／小怪任务） | 服务端；SG `cast_pst` 进出法收尾时调用（[SGcrabking](../../../raw/current/scripts/stategraphs/SGcrabking.lua#L574-L583)） | [EndCastSpell](../../../raw/current/scripts/prefabs/crabking.lua#L944-L956) |
| 触发窗口与清理 | 喷泉发生器终止条件：施法收尾 `EndCastSpell` 移除全部，或本体被冰冻 `OnFreeze` 移除 25 内 `crabking_spellgenerator`；发生器 onremove 取消冒泡任务并停 LP 声 | 服务端 | [EndCastSpell 移除](../../../raw/current/scripts/prefabs/crabking.lua#L946-L952)、[OnFreeze](../../../raw/current/scripts/prefabs/crabking.lua#L1131-L1141)、[onremove](../../../raw/current/scripts/prefabs/crabking.lua#L1892-L1902) |
| 资源 | `crab_king_bubble_fx`（三气泡＋水柱共用）、`crab_king_shine`、`crabking_ring_fx` 各一 zip；`deer_ice_flakes` 与无眼鹿同名 FX 共用动画资源（[[reference/prefabs/deer_ice_circle|无眼鹿冰火法阵]]） | 音效仅 waterspout 条目自带；anim 内容与时长未核验 | [条目组](../../../raw/current/scripts/fx.lua#L2156-L2204)、[deer_fx 同名 FX](../../../raw/current/scripts/prefabs/deer_fx.lua#L515-L515) |

## 典型接入

### 借用气泡族做海面冒泡表现

- 前提与选择：服务端 `SpawnPrefab` 落点式生成；仿原版用随机拼名＋海洋判定，避免在陆地／船上出泡（船上表现可自选震屏替代，见 burble）。
- 生效与撤销：一次性动画，animover 自删，宿主无需清理；三个注册仅动画不同，按名选用或随机。
- 原版依据：[BubbleSeaStack](../../../raw/current/scripts/prefabs/crabking.lua#L1071-L1084)。

```lua
local prefab = "crab_king_bubble"..math.random(1, 3)
if TheWorld.Map:IsOceanAtPoint(x, 0, z) then
    SpawnPrefab(prefab).Transform:SetPosition(x, 0, z)
end
```

- 调整与验证：水柱落水复用与投射物结算分工已在 [[reference/prefabs/cannonball_used#典型接入|cannonball_used 页典型接入]] 维护一份，不在此重复；注册名需经依赖声明或世界预载保证可生成（[[reference/mechanisms/generic-fx-factory#典型接入|工厂页·新增通用特效]]）。

## 常见陷阱与限制

- **误区：waterspout 出柱即有伤害**：船体扣血（[CRABKING_GEYSER_BOATDAMAGE](../../../raw/current/scripts/prefabs/crabking.lua#L1840-L1841)）、漏水、oceanfishable 击飞（满足搜索条件且挂该组件的实体转投射物，含可钓浮渣，不按鱼类标签限定；玩家／无该组件的物品／其他怪不受）、海堆销毁全部写在生成者回调里；FX 无 combat／health，表现本身不结算玩法。但只删注册不等于安全：`endgeyser`、`DoDestroySeaStack` 与三叉戟水柱都在未判空时直接 `SpawnPrefab("crab_king_waterspout").Transform`，缺注册返回 nil 会中断后续击飞／销毁／扣耐久；要移除表现应同时删改或判空这些调用，不能只删注册而保留未判空调用。[击飞归属](../../../raw/current/scripts/prefabs/crabking.lua#L1802-L1819)、[海堆销毁](../../../raw/current/scripts/prefabs/crabking.lua#L1094-L1100)、[三叉戟](../../../raw/current/scripts/prefabs/trident.lua#L183-L197)
- **边界：crab_king_icefx 当前无活跃生成点**：唯一生成逻辑 `freezefx` 只被已停用的 `crabking_feeze` 实体使用——其 `Prefab` 注册整行被注释，`freezeprefabs` 依赖块亦注释；检索范围见验证记录。它与 [[reference/components/freezefx|freezefx 组件]]（通用冰冻碎裂表现）无关，同名仅为蟹王文件内局部函数。[注册注释](../../../raw/current/scripts/prefabs/crabking.lua#L2138-L2143)、[停用声明](../../../raw/current/scripts/prefabs/crabking.lua#L72-L76)
- **边界：气泡与 deadly 喷泉的落点判定不同**：冒泡与收尾水柱都要求 `IsOceanAtPoint`，非海面不出（burble 在船上只 `ShakeAllCameras`）；海堆销毁的水柱无判定，堆在哪柱在哪。[船分支](../../../raw/current/scripts/prefabs/crabking.lua#L1769-L1773)
- **边界：shine 代理与本地动画各自清理**：`inst.shinefx` 是代理引用，蟹王赋值后不再读取或移除；代理按工厂计时移除，本地动画按自身 animover 删除。`AddSocketDecoration` 加载路径跳过闪光，存档恢复后宝石只剩贴图替换。[生成处](../../../raw/current/scripts/prefabs/crabking.lua#L224-L231)、[[reference/mechanisms/generic-fx-factory#关键流程|工厂生命周期]]
- **边界：七条目表现形态分三类**：气泡／水柱只抬显示层（FinalOffset1），shine／icefx 加泛光 shader，ring_fx 贴地背景层——借用时按需选名，不要指望同一注册换形态。[fn 定义](../../../raw/current/scripts/fx.lua#L1-L31)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。7 条目逐字段比对（fx.lua L2156-2204）与三个 fn helper；生成点反查：7 注册名在本快照 `raw/current/scripts` 全域（根级＋prefabs／components／stategraphs）检索，命中仅注册表与上述消费者；动态拼名核对 `"crab_king_bubble"..math.random(1,3)` 两处，另审计 `"crab_king_`／`"crabking_` 前缀拼接（scrapbookdata 数据表与 crabking_claw 的 build 字符串，均非 SpawnPrefab）。触发链逐点核验：SG `activate`／cast 状态链、`SpawnCannons`／`EndIceStage` 两条炮击入口、`OnFreeze` 清场、`endgeyser` 自重排、shine 四入口；独立复核发现炮击冒泡的周期重排为死链（重排任务把仅蟹王导出的 `BubbleSeaStack` 挂到海堆上调，海堆上为 nil，scheduler 静默跳过——每堆实际只出初排一枚），已记入导航行。简中 .po 定向检索 7 名无显示名键。第 4 轮复核补正：`endgeyser` 击飞门禁是 `oceanfishable` 组件而非鱼类标签（含可钓浮渣）；`endgeyser`／`DoDestroySeaStack`／三叉戟在未判空时取 `SpawnPrefab(...).Transform`，只删注册会中断后续击飞／销毁／扣耐久。未覆盖：动画内容／时长／循环性（二进制资源）、TUNING.CRABKING_GEYSER_BOATDAMAGE 等战斗数值未逐项展开、专服与暂停表现未实测。
- 游戏运行：未运行。
- 结构检查：validate_page.py 与 validate_source_links.py --strict 通过（standard 档内）；Lua 示例经 parser_probe 语法检查。

相关：[[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]、[[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/prefabs/cannonball_used|船炮与迫击炮弹命中表现]]、[[reference/prefabs/deer_ice_circle|无眼鹿冰火法阵]]。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]、[[reference/prefabs/index|Prefab 索引]]。
