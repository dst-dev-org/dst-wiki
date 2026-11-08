---
title: "远古档案机关声画实体（archive_ambient_sfx 等 5 注册）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "远古档案设施与星象探测仪的 5 个声画注册：档案环境音发射器（世界生成 50% 铺设、电源事件驱动循环声）、华丽基座的双层贴图地面（大底座＋小踏板）、警戒光球环绕飞行音（Lua 子实体每帧绕圈）、星象探测仪地面标记（持久实体，1 天 timer 到期自删）。"
aliases: ["archive_ambient_sfx", "archive_switch_pad", "archive_switch_base", "archive_security_pulse_sfx", "archive_resonator_base", "远古档案环境音", "华丽基座地面", "档案电源贴图", "警戒光球环绕音", "星象探测仪标记", "archive 电源声效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/archive_props.lua", "prefabs/archive_resonator.lua", "components/archivemanager.lua", "components/timer.lua", "components/grottowarmanager.lua", "prefabs/archive_lockbox.lua", "prefabs/atrium_gate_pulsesfx.lua", "prefabs/cave.lua", "prefabs/vault_teleporter.lua", "map/maze_layouts.lua", "map/layouts.lua", "map/static_layouts/rooms/archive_end/archive_end.lua", "prefabs/archive_chandelier.lua", "entityscript.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 远古档案机关声画实体（archive_ambient_sfx 等 5 注册）

## 快速判断

- 职责与适用：远古档案设施（洞穴遗迹区）与星象探测仪的 **5 个机关声画注册**，共性是"零玩法组件、被机关/世界驱动"：`archive_ambient_sfx`（档案环境音发射器，世界生成按 50% 概率铺设，监听世界电源事件播循环环境声）、`archive_switch_base`／`archive_switch_pad`（华丽基座机关的**双层贴图地面**：大底座 SortOrder 1＋小踏板 SortOrder 2）、`archive_security_pulse_sfx`（警戒光球 `archive_security_pulse` 的**环绕飞行音**子实体，每帧绕圈）、`archive_resonator_base`（星象探测仪瞄准时的**地面标记**，带 1 天到期 timer）。注册行：[archive_security_pulse_sfx](../../../raw/current/scripts/prefabs/archive_props.lua#L1192)、[archive_switch_pad](../../../raw/current/scripts/prefabs/archive_props.lua#L1195)、[archive_switch_base](../../../raw/current/scripts/prefabs/archive_props.lua#L1196)、[archive_ambient_sfx](../../../raw/current/scripts/prefabs/archive_props.lua#L1198)、[archive_resonator_base](../../../raw/current/scripts/prefabs/archive_resonator.lua#L560)
- 重要副作用：5 个注册自身都无伤害、无玩法结算；发声与光效全部由**世界电源事件**（`arhivepoweron`／`arhivepoweroff`，由 archivemanager 推送，[L38](../../../raw/current/scripts/components/archivemanager.lua#L38-L42)）或宿主机关驱动。`archive_ambient_sfx` **不带 `FX` 标签**（只有 `NOBLOCK`＋`NOCLICK`）且**不关 persists**——随世界存档常驻；`archive_resonator_base` 同样是持久实体，靠 timer 组件 480 秒（一个整天周期）到期自删。
- 修改边界：只覆盖 5 注册的工厂、生成点、父子与清理；机关本体（`archive_switch` 华丽基座、`archive_security_pulse` 警戒光球、`archive_resonator` 探测仪、`archive_security_desk` 哨所）只在辨析链条时展开必要部分，玩法细节不入本页。
- 名称：`ARCHIVE_SWITCH_BASE` 有官方名"华丽基座"（与本体 `ARCHIVE_SWITCH` 同名，[po](../../../raw/current/scripts/languages/chinese_s.po#L338774-L338777)）；本体 `ARCHIVE_RESONATOR`＝"星象探测仪"（[po](../../../raw/current/scripts/languages/chinese_s.po#L338749-L338752)）仅作链条辨析用。覆盖的 5 注册中 `ARCHIVE_AMBIENT_SFX`、`ARCHIVE_SWITCH_PAD`、`ARCHIVE_SECURITY_PULSE_SFX`、`ARCHIVE_RESONATOR_BASE` 在简中 `chinese_s.po` 无 `STRINGS.NAMES` 键，标题中文为已核验效果描述，非官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `archive_ambient_sfx` | Transform／Network／SoundEmitter（**无 AnimState**）→ 标签 `NOBLOCK`＋`NOCLICK` → `SetPristine` → 客户端早退 → 主模拟监听 TheWorld 电源事件：`arhivepoweron` 播 `grotto/common/archive_on/1..4` 随机一条（loop 槽），`arhivepoweroff` 杀 loop；**未置 persists=false**。 | 档案区氛围声载体：只在主模拟注册监听（[工厂](../../../raw/current/scripts/prefabs/archive_props.lua#L1144-L1168)），客户端靠网络声效；世界生成铺设后随存档常驻，无任何自删。 |
| `archive_switch_pad` | Transform／AnimState／SoundEmitter／Network → bank/build `archive_switch_ground_small`、`idle`、OnGround、`LAYER_BACKGROUND`、SortOrder 2 → `SetPristine` → 主模拟 `persists=false`。 | 小踏板贴片：SoundEmitter 挂了但工厂内无任何 PlaySound；无显式移除路径——本会话常驻、不存档，读档后由本体再铺新块（[工厂](../../../raw/current/scripts/prefabs/archive_props.lua#L974-L998)、[铺设](../../../raw/current/scripts/prefabs/archive_props.lua#L960-L964)）。 |
| `archive_switch_base` | Transform／AnimState／SoundEmitter／Network → bank/build `archive_switch_ground`、`idle`、OnGround、`LAYER_BACKGROUND`、SortOrder 1；非专服装配 pointofinterest（SetHeight 220，在 SetPristine **之前**，各端本地存在）→ `SetPristine` → 主模拟：0 秒后找 10 格内 `archive_switch` 并转向它（angle−90）；监听 TheWorld 电源事件切换 `activate`→`activate_loop`＋`archive_switch/LP` 循环声或 `deactivate`→`idle`＋杀声。 | 大底座贴片：与本体 `archive_switch` 的 scrapbook 代理（[本体 L929](../../../raw/current/scripts/prefabs/archive_props.lua#L929)）；激活/断电动画与循环声都在这个贴片上播（[工厂](../../../raw/current/scripts/prefabs/archive_props.lua#L1001-L1058)）。 |
| `archive_security_pulse_sfx` | Transform／SoundEmitter／Network（AnimState 整段被注释，[L658-L665](../../../raw/current/scripts/prefabs/archive_props.lua#L658-L665)）→ `SetPristine` → 主模拟 `persists=false`、updatelooper 每帧 `OnUpdatePulseSFX`、播 `archive_security_desk/leave_LP`（loop 槽）。 | 环绕飞行音：作为警戒光球 Lua 子实体生成，初始偏移 (4,0,0)，每帧按"父级指向当前点的角度＋每 2 秒一圈"重算局部坐标，绕父级半径 4（SFXRANGE）旋转（[环绕实现](../../../raw/current/scripts/prefabs/archive_props.lua#L637-L648)）。 |
| `archive_resonator_base` | Transform／AnimState／Network／Light → Light（半径 0.5、强度 0.5、falloff 0.7、色 237/237/209，常开）→ bank/build `archive_resonator`、`idle_marker` 循环、OnGround、`LAYER_BACKGROUND`、SortOrder 3、泛光 `shaders/anim.ksh` → 标签 `FX`＋`NOCLICK` → `SetPristine` → 主模拟：OnSave 记 rotation／OnLoad 还原；timer 组件起 `expire` 计时 `TUNING.TOTAL_DAY_TIME`（480 秒），到期 Remove。 | 探测仪地面标记：**持久实体**而非播完即删；光常开＋泛光，是探测仪瞄准目标时落在目标后方的"落点灯"（[工厂](../../../raw/current/scripts/prefabs/archive_resonator.lua#L475-L515)、[到期](../../../raw/current/scripts/prefabs/archive_resonator.lua#L456-L460)、[timer 值](../../../raw/current/scripts/tuning.lua#L19-L20)）。 |

同文件未被本页收为覆盖注册的还有 `archive_moon_statue`、`archive_rune_statue`、`archive_security_desk`、`archive_security_pulse`、`archive_security_waypoint`、`archive_switch`、`archive_portal`、`rubble1/2`（[注册区](../../../raw/current/scripts/prefabs/archive_props.lua#L1188-L1200)）与 `archive_resonator`／`archive_resonator_item`／placer（[注册区](../../../raw/current/scripts/prefabs/archive_resonator.lua#L557-L559)）；清单外连带注册见[常见陷阱与限制](#常见陷阱与限制)末条。

## 关联机制导航

### 电源事件链：谁在听 arhivepoweron/off

| 监听方 | 收到 `arhivepoweron` | 收到 `arhivepoweroff` | 证据 |
|---|---|---|---|
| `archive_switch_base` | 播 `activate`→`activate_loop`＋循环声 LP | 播 `deactivate`→`idle`＋杀声 | [L1045-L1055](../../../raw/current/scripts/prefabs/archive_props.lua#L1045-L1055) |
| `archive_ambient_sfx` | 随机播 4 条档案开启环境声之一（loop） | 杀 loop | [L1160-L1165](../../../raw/current/scripts/prefabs/archive_props.lua#L1160-L1165) |
| `archive_chandelier`（吊灯，Wiki 见下方互链） | 吊灯 `updatelight()` | 同左 | [L407-L408](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L407-L408) |

吊灯页互链：[[reference/prefabs/archive_chandelier|远古吊灯与石像鬼石化特效]]——同表监听方还包括 vault_teleporter（本仓暂无该 Wiki 页，[L599-L600](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L599-L600)）。

事件源：`archive_switch` 凑齐 3 颗欧泊后 `archivemanager:SwitchPowerOn(true)`（[checkforgems](../../../raw/current/scripts/prefabs/archive_props.lua#L792-L815)），archivemanager 把 `arhivepoweron`/`arhivepoweroff` 推给 TheWorld（[L38-L42](../../../raw/current/scripts/components/archivemanager.lua#L38-L42)）；事件名拼写 `arhive`（少一个 c）为源码原样，监听两端一致才成立。

### 世界生成铺设：ambient 与 switch_base 的进场

`archive_ambient_sfx` 的生成点是**洞穴世界生成的区域函数**：档案各房型（hallway／keyroom／end／start）的静态布局把 `archive_areas` 挂为 areas（[maze_layouts.lua L180-L184](../../../raw/current/scripts/map/maze_layouts.lua#L180-L184)），其中 `archive_sound_area` 以 50% 概率返回该注册（[L21-L23](../../../raw/current/scripts/map/maze_layouts.lua#L21-L23)）；宝库前厅 `Vault_Lobby` 的布局里也有同名区域函数（[layouts.lua L1213-L1216](../../../raw/current/scripts/map/layouts.lua#L1213-L1216)）。`archive_security_pulse_sfx` 所在的 cave 实体依赖表亦登记了它（[cave.lua L106](../../../raw/current/scripts/prefabs/cave.lua#L106)）。`archive_switch_base` 则由 `archive_end` 房型布局**直接作为实体条目**铺进世界（[archive_end.lua L205-L213](../../../raw/current/scripts/map/static_layouts/rooms/archive_end/archive_end.lua#L205-L213)），进场后自行转向同布局内的 `archive_switch`。

### 探测仪 beam 链：base 什么时候被放

星象探测仪（本体，联网）部署后 5 秒扫描目标：瞄中 astral marker 则钻探；瞄中水下月石遗迹／月石／月坛王冠等则进入 beam 态——1 秒后在探测仪朝目标方向、距自身 3 单位处 `SpawnPrefab("archive_resonator_base")`，转向垂直（angle+90），播 `beam_marker` 后 `PushAnimation("idle_marker", true)`，本体同步播 beam 音（[生成](../../../raw/current/scripts/prefabs/archive_resonator.lua#L200-L208)）。base 生成后即进入 480 秒倒计时（[L510-L512](../../../raw/current/scripts/prefabs/archive_resonator.lua#L510-L512)），无论探测仪后续拆解还是耗尽都**不受影响**，走自己的 timer 到期或被玩家清场时随引擎删除。

### 警戒光球子实体链

`archive_security_desk`（哨所）经 childspawner 放出 `archive_security_pulse`（[childname](../../../raw/current/scripts/prefabs/archive_props.lua#L370)）；光球工厂里 `SpawnChild("archive_security_pulse_sfx")` 建 Lua 子实体并登记 `sfx_prefab`（[L624](../../../raw/current/scripts/prefabs/archive_props.lua#L624)），0 秒任务把音源摆到 (4,0,0)（[SetSfxPosition](../../../raw/current/scripts/prefabs/archive_props.lua#L543-L547)）。SFX 只服务端更新（updatelooper 在主模拟段装配），环绕声随网络声效传播到各端。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `archive_ambient_sfx` | 无自删、随世界存档常驻；世界生成只铺一次 | 若要移除需外部主动 Remove，本仓脚本层无此调用（全名反查见[验证记录](#验证记录)） |
| `archive_switch_pad` | 无显式移除；persists=false → 本会话常驻，读档不还原，由本体再铺新块 | 同一存档反复读写不会堆叠旧块（不存档），但会话内 debug 重复生成本体时旧块不清理 |
| `archive_switch_base` | 世界生成实体：随布局持久存在，无自删 | 电源动画是状态循环不是退场；布局删除它才会消失 |
| `archive_security_pulse_sfx` | 环绕更新里父字段为空即自删；父级光球 `Remove` 时经 `SpawnChild`→`AddChild` 登记的 Lua `children` 表删除（[entityscript Remove](../../../raw/current/scripts/entityscript.lua#L1779-L1796)），不是原生 `SetParent` 级联（该级联未核验） | [OnUpdatePulseSFX](../../../raw/current/scripts/prefabs/archive_props.lua#L638-L639)、[子实体生成](../../../raw/current/scripts/prefabs/archive_props.lua#L624) |
| `archive_resonator_base` | timer `expire` 480 秒到期 Remove；旋转存档还原，timer 存档续走（timer 组件 OnSave/OnLoad 保留 timeleft，[timer.lua](../../../raw/current/scripts/components/timer.lua#L122-L136)） | 不挂 animover——即使动画被打断也走满 480 秒；被 hammer/移除探测仪**不**连带删除它 |

## 典型接入

### 做一个"电源/状态事件驱动的世界级声效实体"

- 前提与选择：世界生成或机关布置阶段铺一个 Transform＋SoundEmitter＋Network 的轻实体，主模拟监听 TheWorld 全局事件切换 loop 声；适合区域氛围、设施待机声。ambient 用 `NOBLOCK`＋`NOCLICK` 让它对玩家完全透明。
- 生效与撤销：`arhivepoweron` 时 `PlaySound(..., "loop")`，`arhivepoweroff` 时 `KillSound("loop")`——loop 槽是唯一需要配对管理的状态（[ambient 工厂](../../../raw/current/scripts/prefabs/archive_props.lua#L1160-L1165)）。

```lua
-- 服务端：随世界常驻的区域声效实体（对应 archive_ambient_sfx 的模式）
inst:ListenForEvent("arhivepoweron", function()
    inst.SoundEmitter:PlaySound("grotto/common/archive_on/"..math.random(1,4), "loop")
end, TheWorld)
inst:ListenForEvent("arhivepoweroff", function()
    inst.SoundEmitter:KillSound("loop")
end, TheWorld)
```

- 调整与验证：替换点是事件名与声音路径；注意监听须在主模拟段注册、事件源用 TheWorld；要持久声效就别关 persists（ambient 的做法），要会话级贴片就 `persists=false`（pad 的做法）。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：`archive_ambient_sfx` 不带 `FX` 标签。** 工厂只有 `NOBLOCK`＋`NOCLICK`（[L1151-L1152](../../../raw/current/scripts/prefabs/archive_props.lua#L1151-L1152)）；按 `"FX"` 标签检索档案氛围声会落空。它也不 `persists=false`——是随存档常驻的世界实体。
- **边界：`archive_switch_pad` 没有退场路径。** 工厂无自删、无监听；它靠"不存档"在换图/重启后自然消失，会话内永远在场。想复用为临时贴片必须自己配移除逻辑（[工厂](../../../raw/current/scripts/prefabs/archive_props.lua#L974-L998)）。
- **边界：华丽基座官方名是底座与本体共享的。** `ARCHIVE_SWITCH_BASE` 与 `ARCHIVE_SWITCH` 在 po 中同为"华丽基座"（[L338769-L338777](../../../raw/current/scripts/languages/chinese_s.po#L338769-L338777)）；图鉴页（scrapbook）也以 base 为代理展示本体（[L929](../../../raw/current/scripts/prefabs/archive_props.lua#L929)）。检索名称时两者都命中属预期。
- **边界：`archive_resonator_base` 不是播完即删的 FX。** 它持久存档（旋转＋timer 都存）、480 秒前不会被任何档案机关连带清理；拆探测仪留下的"孤儿标记"会继续亮满一整天。需要短命标记时应自配 timer 或 animover（[到期链](../../../raw/current/scripts/prefabs/archive_resonator.lua#L507-L512)）。
- **边界：`archive_security_pulse_sfx` 的环绕只在服务端算。** 客户端副本不含 updatelooper 更新，靠网络声效收听；AnimState 调试段整段注释，实体无任何可见动画。
- **误区：电源事件叫 `archivepoweron`。** 源码事件名是 `arhivepoweron`/`arhivepoweroff`（拼写少一个 c，推送方与全部监听方一致，[推送](../../../raw/current/scripts/components/archivemanager.lua#L38-L42)）；按正确拼写监听永远收不到。
- **边界：清单外连带注册。** `archive_switch` 的通电播音助手 `spawnsounderobj` 生成的是 `archive_dispencer_sfx`（注册在 [archive_lockbox.lua L423](../../../raw/current/scripts/prefabs/archive_lockbox.lua#L423)，10 秒后自删，[生成](../../../raw/current/scripts/prefabs/archive_props.lua#L714-L719)）；其 `prefabs_switch` 依赖表里的 `grotto_war_sfx` 注册在本页姊妹篇的 makesfx 工厂（[atrium_gate_pulsesfx.lua L74](../../../raw/current/scripts/prefabs/atrium_gate_pulsesfx.lua#L74)），由 grottowarmanager 在战线铺设时生成（[L162](../../../raw/current/scripts/components/grottowarmanager.lua#L162)、[L266](../../../raw/current/scripts/components/grottowarmanager.lua#L266)）——这两个注册不属本页原定清单，按发现并入记录，不展开工厂。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/archive_props.lua`、`prefabs/archive_resonator.lua`、`components/timer.lua`，按需读取 archivemanager 推送两行、`prefabs/archive_lockbox.lua` 的 dispencer_sfx 工厂段、`components/grottowarmanager.lua` 战线生成段、`prefabs/atrium_gate_pulsesfx.lua` 注册行、`prefabs/cave.lua` 依赖表段、`map/maze_layouts.lua` 区域函数与房型挂接段、`map/layouts.lua` Vault_Lobby 段、`map/static_layouts/rooms/archive_end/archive_end.lua` 实体条目段、`tuning.lua` 时间常量与 ARCHIVE_SECURITY/RESONATOR 段。对 5 个注册名以完整名在 `raw/current/scripts` 全域反查（`grep -rn "注册名" --include=*.lua`）：`archive_security_pulse_sfx` 命中 3 处（依赖表 L33、SpawnChild 生成 L624、注册 L1192）——生成点唯一；`archive_switch_pad` 命中 3 处（依赖 L45、生成 L962、注册 L1195）——生成点唯一；`archive_switch_base` 命中布局条目（archive_end.lua:206）＋注册＋scrapbook 代理/debug 映射，生成点唯一（世界生成）；`archive_ambient_sfx` 命中 cave.lua:106 依赖、maze_layouts.lua:22 与 layouts.lua:1215 两个区域函数、注册——生成点均为世界生成区域函数（50% 概率）；`archive_resonator_base` 命中注册与 archive_resonator.lua:201 唯一生成点。
- 名称核验：`languages/chinese_s.po` 定向检索——`ARCHIVE_SWITCH`/`ARCHIVE_SWITCH_BASE`＝"华丽基座"（锚点见快速判断）、`ARCHIVE_RESONATOR`/`ARCHIVE_RESONATOR_ITEM`＝"星象探测仪"；`ARCHIVE_AMBIENT_SFX`、`ARCHIVE_SWITCH_PAD`、`ARCHIVE_SECURITY_PULSE_SFX`、`ARCHIVE_RESONATOR_BASE` 的 NAMES 键全文 0 命中，标题使用效果描述。
- 分档说明：`standard`（字段省略）——5 注册均为单层生成/清理，无多变体状态机；链路多为单向监听，standard 预算内可完整承载。
- 未覆盖：机关本体玩法（欧泊交易、警戒光球 AI 与 brain、探测仪钻探产物）、`archive_dispencer_sfx`/`grotto_war_sfx` 工厂细节（仅记注册位置与生成点）、动画包内容与观感、archive 音效资源的实际响度曲线；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录。
- 第 16 轮补正（2026-11-07）：警戒光球音的清理由“引擎父挂级联”改为 `SpawnChild`→`AddChild` 的 Lua `children` 删除，补 `entityscript.lua`；正文引用的 `archive_chandelier.lua`、`tuning.lua` 补入 sources。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/archive_chandelier|远古吊灯与石像鬼石化特效]]（同为档案电源事件监听方；checkforgems 逐盏调用吊灯 updatelight）
- [[reference/prefabs/atrium_gate_activatedfx|远古大门充能光层与警示声]]（`grotto_war_sfx` 注册在其 makesfx 工厂，由地穴战线铺设生成）
- 返回 [[reference/prefabs/fx-light-index|灯光、火焰与音效代理]]
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-06）。
