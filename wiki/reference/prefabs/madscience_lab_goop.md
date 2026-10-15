---
title: "实验台三层特效与粒子杂项（madscience_lab_goop、mist、nonslipgrit_buff_fx、kitcoon_hide_fx）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "四组六个无官方名称的外围 FX 对照：实验台 goop／fire／glow 三层、森林 Mist 节点雾、防滑砂 buff 粒子与小猫可躲藏动画。"
aliases: ["madscience_lab_goop", "madscience_lab_fire", "madscience_lab_glow", "mist", "nonslipgrit_buff_fx", "kitcoon_hide_fx", "疯狂科学家实验室特效", "森林雾粒子"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/madscience_lab.lua", "prefabs/mistparticle.lua", "prefabs/nonslipgrit.lua", "prefabs/kitcoon.lua", "gamelogic.lua", "components/emitter.lua", "emitters.lua", "components/hideandseekhider.lua", "components/debuffable.lua", "components/slipperyfeet.lua", "components/nonslipgrituser.lua", "components/nonslipgritsource.lua", "map/maptags.lua", "map/rooms/forest/graveyard.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 实验台三层特效与粒子杂项（madscience_lab_goop、mist、nonslipgrit_buff_fx、kitcoon_hide_fx）

## 快速判断

- 职责与适用：四组六个纯视觉注册（工厂都只加 `FX`、不挂玩法组件）：`madscience_lab_goop`／`_fire`／`_glow` 是疯狂科学实验室（`madscience_lab`）的三层附属表现，共用 bank／build、靠 `Hide` 符号分层；`mist` 是拓扑 `Mist` 节点的区域雾；`nonslipgrit_buff_fx` 是防滑砂 debuff 的持有者粒子；`kitcoon_hide_fx` 是小猫可躲藏瞬间的短动画。
- 优先使用：区域粒子读 [[reference/components/emitter]] 与本页 `mist`；"父挂跟随、按位移调量"的粒子看 `nonslipgrit_buff_fx`；"事件触发、短时自删"的贴图动画看 `kitcoon_hide_fx`；实验台本体与制作流程见 [[reference/components/madsciencelab]]。
- 重要副作用：生成点与端各异——goop／glow 由宿主在服务端生成并父挂，fire 随制作阶段重建并联动宿主 `Light`；`mist` 不联网、在非专用端（主机与客户端）各建一份、专服跳过；buff FX 随 debuff 附加生成、脱离删除；hide FX 在 1 秒或 `animover` 时自删。
- 修改边界：只覆盖四组六个注册的工厂、生成点、父挂与清理；不展开实验台本体与配方、防滑砂玩法、小猫可 AI；粒子视觉与音效未在游戏内复核。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `madscience_lab_goop` | `goop_fn`：Transform＋AnimState＋Network → 同 bank／build、bloom、`idle`、`Hide("lab")`／`Hide("glow")`、`LightOverride .5`、`FX` → 非主机 return → `persists = false` | 黏液层；不设 `SetFinalOffset`，动画由宿主推；工厂无自带 `Asset`。[工厂](../../../raw/current/scripts/prefabs/madscience_lab.lua#L352-L378)、[注册](../../../raw/current/scripts/prefabs/madscience_lab.lua#L438) |
| `madscience_lab_fire` | `fire_fn` → 同 bank／build、bloom、`FinalOffset 3`、`fire1`、`FX` → 非主机 return → `persists = false` | 火焰层；无 `Light`、无自删；宿主 `MakeFireFx` 父挂到 `fire_guide`，阶段回调推 `fire3_pre`／`fire3`。[工厂](../../../raw/current/scripts/prefabs/madscience_lab.lua#L326-L350)、[注册](../../../raw/current/scripts/prefabs/madscience_lab.lua#L439) |
| `madscience_lab_glow` | `glow_fn` → 同 bank／build、bloom、`idle`、`Hide("lab")`／`Hide("goop")`、`LightOverride .3`、`FinalOffset 2`、`FX` → 客户端挂 `OnEntityReplicated` 后 return → `persists = false` | 发光层；客户端仅当父级 `prefab == "madscience_lab"` 才写父级 `highlightchildren`。[工厂](../../../raw/current/scripts/prefabs/madscience_lab.lua#L387-L416)、[客户端钩子](../../../raw/current/scripts/prefabs/madscience_lab.lua#L380-L385)、[注册](../../../raw/current/scripts/prefabs/madscience_lab.lua#L440) |
| `mist` | `fn`：Transform（未 `AddNetwork`）、`FX`、客户端 `AddClientSleepable`、`persists = false` → envelope → `VFXEffect` 单发射器（寿命 31、上限 32、半径 25、排序 3）→ `emitter` 组件（地面高 0.4、每帧 1） | 区域雾；`area_emitter` 由消费者赋值后再 `Emit()`。[工厂](../../../raw/current/scripts/prefabs/mistparticle.lua#L41-L99)、[注册](../../../raw/current/scripts/prefabs/mistparticle.lua#L101) |
| `nonslipgrit_buff_fx` | `fn_buff_fx`：Transform＋SoundEmitter＋Network、`FX`、循环音、`persists = false` → `SetPristine` → 专服 return → 双发射器（`confetti`／`snow` 各 50 粒、寿命 1 秒）→ `EmitterManager` 按父级位移补发 | 生成于 `nonslipgrit_buff` 附着回调并父挂目标；专服无粒子。[工厂](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L304-L375)、[注册](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L381)、[生成](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L176-L189) |
| `kitcoon_hide_fx` | `hide_fx_fn`：Transform＋AnimState＋SoundEmitter＋Network、`SetSixFaced`、bank／build `kitcoon_fx`、`FinalOffset 1`、`FX` → 非主机 return → `persists = false`＋1 秒 `Remove`＋`animover` | 无父挂；位置／朝向由生成方复制；自身不播音。[工厂](../../../raw/current/scripts/prefabs/kitcoon.lua#L409-L439)、[注册](../../../raw/current/scripts/prefabs/kitcoon.lua#L452)、[生成](../../../raw/current/scripts/prefabs/kitcoon.lua#L120-L127) |

名称：六个注册名均无 `STRINGS.NAMES` 条目（按名检索 `.po`），标题是效果描述、非官方译名；`.po` 只有宿主名，如 `STRINGS.NAMES.MADSCIENCE_LAB`＝"疯狂科学家实验室"（[译名](../../../raw/current/scripts/languages/chinese_s.po#L344949-L344952)）。

分层：本体与 goop／glow 共用 `madscience_lab` bank／build，本体隐藏 `goop`／`glow`，goop 隐藏 `lab`／`glow`，glow 隐藏 `lab`／`goop`；本体／glow／fire 的 `FinalOffset` 为 1／2／3，goop 未设。[本体符号](../../../raw/current/scripts/prefabs/madscience_lab.lua#L270-L276) 宿主在 `PlayAnimation`／`PushAnimation` 同推三层，服务端生成 goop／glow 后 `SetParent`。[动画同步](../../../raw/current/scripts/prefabs/madscience_lab.lua#L83-L93)、[子实体生成](../../../raw/current/scripts/prefabs/madscience_lab.lua#L288-L295)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 宿主 prefab：`madscience_lab` | 服务端生成 goop／glow 并父挂、同推动画；按阶段重建 fire 并开关宿主 `Light` | 主模拟；客户端经网络拿到子实体 | [[reference/components/madsciencelab]]、[生成](../../../raw/current/scripts/prefabs/madscience_lab.lua#L288-L295)、[fire 挂接](../../../raw/current/scripts/prefabs/madscience_lab.lua#L129-L138) |
| 组件：`madsciencelab` | `OnStageStarted`／`OnScienceWasMade` 驱动 fire 与产物收尾 | 服务端 | [[reference/components/madsciencelab]]、[回调赋值](../../../raw/current/scripts/prefabs/madscience_lab.lua#L299-L302) |
| 组件：`emitter` | `mist` 的 VFX＋发射逻辑；`area_emitter`／`density_factor`／`Emit()` | 非专服；各端本地 | [[reference/components/emitter]]、[挂载](../../../raw/current/scripts/prefabs/mistparticle.lua#L92-L96)、[Emit 注册](../../../raw/current/scripts/components/emitter.lua#L74) |
| `EmitterManager`／`CreateAreaEmitter` | 两个粒子实体注册逐帧回调、`onremove` 注销；`mist` 用多边形质心采样 | 每帧；世界加载 | [AddEmitter／RemoveEmitter](../../../raw/current/scripts/emitters.lua#L16-L40)、[采样](../../../raw/current/scripts/emitters.lua#L169-L198) |
| 组件：`hideandseekhider` | 超时入点前调宿主 `OnHide`，kitcoon 据此生成 FX | 服务端 | [[reference/components/hideandseekhider]]、[调用点](../../../raw/current/scripts/components/hideandseekhider.lua#L19-L35) |
| 关联 prefab：`kitcoon` | `OnHide_HideAndSeek` 生成、复制位姿、覆盖 `hair`、播 emote；FX 在其依赖表 | 服务端 | [生成](../../../raw/current/scripts/prefabs/kitcoon.lua#L120-L127)、[依赖](../../../raw/current/scripts/prefabs/kitcoon.lua#L4-L6) |
| 组件：`debuffable`＋`nonslipgrit_buff` | 同名 buff 唯一实例：首次 `AddDebuff` 生成、重复走 `Extend`；附着建 FX、脱离删除 | 服务端；buff 为 `CLASSIFIED` 非联网，FX 联网 | [[reference/components/debuffable]]、[AddDebuff](../../../raw/current/scripts/components/debuffable.lua#L95-L108)、[附着／脱离](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L176-L197) |
| 物品侧链路 | fueled `OnDelta` 在跑动累计时 `AddDebuff`；`slipperyfeet` 跑动时把 `dt` 交给 `nonslipgrituser` → `nonslipgritsource` | 服务端 | [[reference/components/nonslipgrituser]]、[[reference/components/nonslipgritsource]]、[OnDelta](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L17-L20)、[跑动](../../../raw/current/scripts/components/slipperyfeet.lua#L245-L249)、[转交](../../../raw/current/scripts/components/nonslipgrituser.lua#L24-L31)、[源](../../../raw/current/scripts/components/nonslipgritsource.lua#L31-L34) |
| 数据：`Mist` 标签 | 房间声明 → `maptags` 转节点 TAG → `gamelogic` 只对含该 TAG 的节点生成雾 | 世界加载；专服跳过 | [转换](../../../raw/current/scripts/map/maptags.lua#L199)、[房间](../../../raw/current/scripts/map/rooms/forest/graveyard.lua#L5)、[生成循环](../../../raw/current/scripts/gamelogic.lua#L551-L577) |
| 资源 | `anim/madscience_lab.zip`（三层共用）、`ds_fog1.tex`＋`vfx_particle.ksh`、`confetti.tex`／`snow.tex`＋`winter2025.fsb`、`kitcoon_fx.zip` | 分别声明在宿主或工厂；声明不等于资源已验证 | [实验台](../../../raw/current/scripts/prefabs/madscience_lab.lua#L3-L6)、[雾](../../../raw/current/scripts/prefabs/mistparticle.lua#L1-L12)、[buff](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L162-L174)、[hide](../../../raw/current/scripts/prefabs/kitcoon.lua#L452) |

## 典型接入

### 按世界区域铺一片雾（复刻 gamelogic 的消费者写法）

- 前提：非专用服务器；`area_emitter` 返回**相对**坐标，`density_factor` 在 `Emit()` 时缩放粒子上限。[采样约定](../../../raw/current/scripts/emitters.lua#L195-L196)、[上限](../../../raw/current/scripts/components/emitter.lua#L23-L26)
- 生效与撤销：`Emit()` 只调一次，之后由 `EmitterManager` 每帧驱动；实体移除时自动注销；不联网，各端各跑一份。
- 原版依据：`gamelogic` 取 `node.cent` 定位、`CreateAreaEmitter(node.poly, node.cent)` 赋采样、按面积设密度。[生成循环](../../../raw/current/scripts/gamelogic.lua#L558-L575)

```lua
local function SpawnAreaMist(x, z, radius, density)
    local mist = SpawnPrefab("mist")
    mist.Transform:SetPosition(x, 0, z)
    mist.components.emitter.area_emitter = function()
        return UnitRand() * radius, UnitRand() * radius
    end
    mist.components.emitter.density_factor = density
    mist.components.emitter:Emit()
    return mist
end
```

- 调整与验证：`radius`／`density` 换成实际区域尺寸与面积估算；`Emit()` 前必须已赋采样函数。这是方形相对坐标采样；原版 `CreateAreaEmitter(node.poly, node.cent)` 是多边形采样，不能据本片段推断原版均匀分布。片段按原版调用顺序改写，仅做语法检查，未在游戏内运行。

### 复用"同 bank 分层"与"父挂跟随"

- 前提：三层须来自同一 build；`Hide` 互斥只适用于本体／goop／glow（fire 不参与），否则符号重叠；子层在服务端生成后 `SetParent`，宿主移除时的引擎级联未核验，自建宿主应在移除前显式 `Remove`。
- 生效与撤销：goop／glow 自身不驱动动画，由宿主统一推；fire 用 `AddFollower`＋`FollowSymbol` 贴 `fire_guide`，`RemoveFireFx` 删除并关灯。[fire 挂接与收尾](../../../raw/current/scripts/prefabs/madscience_lab.lua#L129-L148)
- 原版依据：三工厂均不声明 `Asset`，单独生成前须确保 `anim/madscience_lab.zip` 已加载。[资源声明](../../../raw/current/scripts/prefabs/madscience_lab.lua#L3-L6)、[注册](../../../raw/current/scripts/prefabs/madscience_lab.lua#L437-L440)

## 常见陷阱与限制

- **误区：三层是三份独立美术资源**：三者与结构共用 `madscience_lab` bank／build，只靠 `Hide` 互斥；漏掉任一层 `Hide` 会符号重叠；工厂也不带 `Asset`。[本体隐藏](../../../raw/current/scripts/prefabs/madscience_lab.lua#L270-L276)
- **边界：goop／glow 没有自删，也不自行跟随阶段换动画**：单独 `SpawnPrefab` 会停在 `idle` 并常驻；动画由宿主推，实体只 `SetParent` 跟随，宿主删除时的引擎级联未核验，自建宿主应在移除前显式 `Remove`。[子实体生成](../../../raw/current/scripts/prefabs/madscience_lab.lua#L288-L295)
- **边界：火焰光源在宿主 `Light`**：`MakeFireFx` 开启、`RemoveFireFx` 关闭；单独生成 fire 只有动画。[灯光耦合](../../../raw/current/scripts/prefabs/madscience_lab.lua#L129-L148)、[完成收尾](../../../raw/current/scripts/prefabs/madscience_lab.lua#L159-L165)
- **边界：glow 的 `highlightchildren` 有宿主名检查**：客户端钩子只接受 `parent.prefab == "madscience_lab"`；服务端也只登记 glow。[客户端钩子](../../../raw/current/scripts/prefabs/madscience_lab.lua#L380-L385)
- **陷阱：`mist` 的 `area_emitter` 默认是占位函数**：只打印 `no emitter`，不先赋值就 `Emit()` 不发射；返回值按相对坐标处理，`y` 取 `ground_height`（0.4）。[默认值](../../../raw/current/scripts/components/emitter.lua#L3)、[消费点](../../../raw/current/scripts/components/emitter.lua#L41)
- **边界：`mist` 不联网、专服不生成**：生成条件 `not TheNet:IsDedicated()`；密度按 `node.area`（缺失按 1）算 `ceil(node.area / 4) / 31`。[生成与密度](../../../raw/current/scripts/gamelogic.lua#L558-L575)
- **边界：buff FX 的生命周期是 debuff 实例**：`BUFF_DURATION = 0.25` 秒内同名 `AddDebuff` 只 `Extend`；到期／停止时 `OnDetached_buff` 删 FX，下次附着重建并重启循环音。[时长](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L160)、[附着／脱离](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L176-L197)、[复用](../../../raw/current/scripts/components/debuffable.lua#L95-L108)
- **边界：buff FX 回调要求先 `SetParent`**：无父级时不发射；按帧间位移在每路每秒 10～30 粒之间插值，两路合计 20～60 粒/秒，实际逐帧数量由累计余量与 `while > 0` 离散发射决定。[回调](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L354-L372)
- **边界：hide FX 无父挂、双重自删**：1 秒定时与 `animover` 先到者生效；位姿只在生成时复制，不跟随 kitcoon；实体自身不播音。[自删](../../../raw/current/scripts/prefabs/kitcoon.lua#L433-L436)、[生成](../../../raw/current/scripts/prefabs/kitcoon.lua#L120-L127)
- **源码观察：`node.area_emitter` 在本快照内无赋值点**：`gamelogic` 只读，实例字段 `area_emitter` 由 `emitter` 使用；按现有代码每次加载各生成一次，加载流程重复执行时该判断不会拦截（未找到赋值路径）。[读取](../../../raw/current/scripts/gamelogic.lua#L559)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394；通读四个 prefab 文件（`madscience_lab.lua` 448 行、`mistparticle.lua` 101 行、`nonslipgrit.lua` 381 行、`kitcoon.lua` 452 行），核对 `gamelogic.lua` 拓扑循环、`emitter.lua`、`emitters.lua`、`hideandseekhider.lua`、`debuffable.lua`、`maptags.lua`、`graveyard.lua`；全 `scripts`（4095 个 `.lua`）按注册名反查，生成点分别为 goop／fire／glow＝`madscience_lab.lua:288/130/291`，`mist`＝`gamelogic.lua:565`，buff FX＝`nonslipgrit.lua:185`，hide FX＝`kitcoon.lua:121`。未覆盖：游戏内视觉与音效、粒子数量、实验台结构与配方、防滑砂与小猫可玩法。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 通过；`validate_source_links.py --strict` 无错误／警告；Lua 片段经 `parser_probe.exe --syntax-only` 通过。

返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]。
