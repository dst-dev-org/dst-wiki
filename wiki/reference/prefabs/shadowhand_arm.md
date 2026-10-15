---
title: "暗影之手与恐惧之手的伸缩手臂（shadowhand_arm、shadowhand_arm_shrouded）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "暗影之手／恐惧之手贴地伸出时留在起点、朝移动手部拉伸的两只手臂 FX；普通与遮蔽两套 build，纯视觉、无自删，生成者负责拉伸目标与移除。"
aliases: ["shadowhand_arm", "shadowhand_arm_shrouded", "暗影之手手臂", "恐惧之手手臂", "影子手臂", "伸缩影臂"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/shadowhand.lua", "prefabs/charlie_hand.lua", "components/stretcher.lua", "components/highlightchild.lua", "components/shadowhandspawner.lua", "prefabs/lightsout_trial.lua", "prefabs/charlie_boss_trial.lua", "fx.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗影之手与恐惧之手的伸缩手臂（shadowhand_arm、shadowhand_arm_shrouded）

## 快速判断

- 职责与适用：`prefabs/shadowhand.lua` 的 `MakeShadowArm` 注册两只**纯视觉手臂**——`shadowhand_arm`（普通 build `shadow_creatures_ground`）与 `shadowhand_arm_shrouded`（遮蔽 build `shadow_creatures_ground_shrouded`）。手臂在生成点固定不动，靠 `stretcher` 组件朝移动中的手部拉伸，表现"手从地里伸出、影子还留在原处"。[工厂](../../../raw/current/scripts/prefabs/shadowhand.lua#L215-L260)、[注册](../../../raw/current/scripts/prefabs/shadowhand.lua#L346-L349)
- 优先使用：做"固定起点＋移动端点"的拉长视觉（触手、影子、绳索类贴地表现）时复用这套结构；直接借用 `shadowhand_arm` 时只需给它 `stretcher` 目标并自行清理。[典型接入](#给自定义宿主生成拉伸手臂)
- 重要副作用：手臂**没有伤害、没有碰撞、没有自删**；`NOCLICK`＋`FX` 标签，`persists=false`。手臂的 `LightWatcher` 触发 `enterlight` 时，普通／遮蔽手部会经各自监听消散；手臂没有自己的光照监听，而是在手部退场时被驱动播放 `arm_scare`，再由手部移除。[手臂工厂](../../../raw/current/scripts/prefabs/shadowhand.lua#L222-L252)、[手部监听](../../../raw/current/scripts/prefabs/shadowhand.lua#L180-L182)
- 修改边界：只覆盖两只手臂的工厂、两种 build、生成点、拉伸接线与由宿主负责的清理；`shadowhand`／`shadowhand_shrouded` 本体的追火、熄灭、理智光环与玩家邻近逻辑只作辨析；`shadowhand_fx` 是 `fx.lua` 的另一注册，不属本页。
- 名称：`STRINGS.NAMES.SHADOWHAND`＝"暗影之手"、`STRINGS.NAMES.SHADOWHAND_SHROUDED`＝"恐惧之手"（[po](../../../raw/current/scripts/languages/chinese_s.po#L349134-L349142)）；两只手臂注册名均无 `STRINGS.NAMES` 键，标题中的"伸缩手臂"是已核验效果描述。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shadowhand_arm` | `MakeShadowArm("shadowhand_arm", {build = "shadow_creatures_ground"})`：`NOCLICK`＋`FX` → Transform／AnimState／LightWatcher／Network → 光阈值 .2／.19 → bank `shadowcreatures`、build `shadow_creatures_ground`、`OnGround`、`LAYER_BACKGROUND`、排序 3、`arm_loop` 循环 → `highlightchild` → `SetPristine` → 非主模拟早退；主模拟加 `stretcher`（静止长度 4.75、宽度比 .35）、`persists=false`。 | 无 SoundEmitter、无 Physics、无 health／combat；`arm_loop` 在工厂里就播，生成者不需要再写动画。[注册](../../../raw/current/scripts/prefabs/shadowhand.lua#L346-L347)、[工厂](../../../raw/current/scripts/prefabs/shadowhand.lua#L215-L260) |
| `shadowhand_arm_shrouded` | `MakeShadowArm("shadowhand_arm_shrouded", {build = "shadow_creatures_ground_shrouded"})`：同上，仅 build 换成遮蔽贴图；`assets_shrouded` 同时带 `shadow_creatures_ground.zip`（动画）与 `shadow_creatures_ground_shrouded.zip`（build）。 | 只由 `shadowhand_shrouded` 的 `armprefab` 生成；`charlie_hand` 永远生成普通手臂，不生成这一只。[注册](../../../raw/current/scripts/prefabs/shadowhand.lua#L348-L349)、[遮蔽资产](../../../raw/current/scripts/prefabs/shadowhand.lua#L11-L20) |

工厂对两只手臂完全同构；差异只有 `build` 和生成者传入的 `armprefab`。[MakeShadowArm](../../../raw/current/scripts/prefabs/shadowhand.lua#L215-L260)

## 生成、拉伸与清理

### 两只宿主手部

| 宿主注册 | 生成点 | 说明 |
|---|---|---|
| `shadowhand`（暗影之手） | 低理智玩家附近有火源时由 `components/shadowhandspawner.lua:189` 生成；宝库熄灯试炼由 `prefabs/lightsout_trial.lua:248` 生成 | 追火、`playerprox` 近远切换、`sanityaura`；本体是玩法实体，不是 FX。[刷手](../../../raw/current/scripts/components/shadowhandspawner.lua#L176-L198)、[试炼](../../../raw/current/scripts/prefabs/lightsout_trial.lua#L242-L251) |
| `shadowhand_shrouded`（恐惧之手） | 仅 `prefabs/charlie_boss_trial.lua:347` | 遮蔽 build、更快移速、更大理智光环；本体同样不是 FX。[生成](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L337-L355)、[遮蔽差异](../../../raw/current/scripts/prefabs/shadowhand.lua#L338-L344) |

两只手部都在 `SetTargetFire` 里生成自己的手臂（[实现](../../../raw/current/scripts/prefabs/shadowhand.lua#L159-L201)）：

```lua
inst.arm = SpawnPrefab(inst.armprefab)
inst.arm.Transform:SetPosition(pos:Get())
inst.arm:FacePoint(fire:GetPosition())
inst.arm.components.stretcher:SetStretchTarget(inst)
```

- `pos` 是手部**当时的生成位置**；手臂从此固定，手部随后用 `locomotor` 向火堆爬，`stretcher` 每帧把手臂朝手部拉长。
- `armprefab` 在 `MakeShadowHand` 的数据里写死：普通手→`shadowhand_arm`，遮蔽手→`shadowhand_arm_shrouded`（[数据](../../../raw/current/scripts/prefabs/shadowhand.lua#L346-L349)）。
- 手部 `OnRemoveEntity` 会 `inst.arm:Remove()`，手臂不自己退场（[清理](../../../raw/current/scripts/prefabs/shadowhand.lua#L203-L209)）。

### 查理手部的第二生成点

`prefabs/charlie_hand.lua` 的 `SpawnShadowArm` 生成**普通** `shadowhand_arm`，放在施工点，朝远古大门方向，拉伸目标是查理手部本体，并把 `highlightchild` 的 owner 设为手部（[实现](../../../raw/current/scripts/prefabs/charlie_hand.lua#L118-L128)）：

```lua
local arm = SpawnPrefab("shadowhand_arm")
arm.Transform:SetPosition(pos.x, 0, pos.z)
arm:FacePoint(atrium_pos.x, 0, atrium_pos.z)
arm.components.stretcher:SetStretchTarget(inst)
arm.components.highlightchild:SetOwner(inst)
```

- 初始化和读档重建都走这个函数（[初始化](../../../raw/current/scripts/prefabs/charlie_hand.lua#L130-L142)、[读档](../../../raw/current/scripts/prefabs/charlie_hand.lua#L153-L179)）；`OnRemove` 负责删手臂（[清理](../../../raw/current/scripts/prefabs/charlie_hand.lua#L181-L185)）。
- 这是全库第二类生成点：唯一直接按字面名生成 `shadowhand_arm` 的地方；普通／遮蔽手部都只经 `inst.armprefab` 间接生成。

### 拉伸协议与清理边界

- `stretcher:SetStretchTarget` 会在主模拟立即 `OnUpdate(0)` 并按两实体三维距离设置 `AnimState:SetScale`；公式、休眠与失效行为见 [[reference/components/stretcher|stretcher 组件页]]，本页不重复。[组件](../../../raw/current/scripts/components/stretcher.lua#L16-L52)
- 手臂没有 `OnRemoveEntity`、没有 `animover` 自删、没有定时器；`persists=false` 只保证不入存档，**不保证实体自动消失**。目标失效时 `stretcher` 只是停更，手臂仍在原地。
- 因此清理责任完全在生成者：普通／遮蔽手部在 `OnRemoveEntity`，查理手部在 `OnRemove`；两者都必须显式 `arm:Remove()`。[普通手清理](../../../raw/current/scripts/prefabs/shadowhand.lua#L203-L209)、[查理手清理](../../../raw/current/scripts/prefabs/charlie_hand.lua#L181-L185)
- 手臂的 `LightWatcher` 本身只产生光照事件；普通／遮蔽手部都监听手臂的 `enterlight`，触发 `Dissipate`：手部播 `hand_scare`、手臂播 `arm_scare`，手部 `animover` 自删后再经 `OnRemoveEntity` 删除手臂。手臂没有自己的自删监听，退场由手部驱动；查理手部没有这条监听。[监听](../../../raw/current/scripts/prefabs/shadowhand.lua#L180-L182)、[消散](../../../raw/current/scripts/prefabs/shadowhand.lua#L22-L48)

## 关联机制导航

| 类型／对象 | 本注册如何使用 | 变体、端与条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：stretcher` | 朝移动手部拉伸手臂图像；静止长度 4.75、宽度比 .35 | 只在主模拟添加；目标失效停更，不删宿主 | [[reference/components/stretcher|stretcher 组件页]]、[配置](../../../raw/current/scripts/prefabs/shadowhand.lua#L248-L250) |
| `组件：highlightchild` | 让手臂参与 owner 的高亮链；查理手部把 owner 设为自身 | 普通手部不设 owner；专服不执行高亮 | [组件](../../../raw/current/scripts/components/highlightchild.lua#L24-L55)、[查理接线](../../../raw/current/scripts/prefabs/charlie_hand.lua#L125-L125) |
| `组件：LightWatcher` | 光阈值 .2／.19；`enterlight` 由手部监听并触发消散 | 普通／遮蔽手臂都挂；普通／遮蔽手部都监听，查理手部不监听 | [阈值](../../../raw/current/scripts/prefabs/shadowhand.lua#L230-L231)、[监听](../../../raw/current/scripts/prefabs/shadowhand.lua#L180-L182) |
| `Prefab：shadowhand`／`shadowhand_shrouded` | 手臂的生成者与清理者；手部本体是玩法实体 | 两只手部各有自己的 `armprefab` | [注册](../../../raw/current/scripts/prefabs/shadowhand.lua#L346-L349) |
| `Prefab：charlie_hand` | 施工演出中生成普通手臂，设置高亮 owner，读档重建 | 只生成 `shadowhand_arm`，不生成遮蔽手臂 | [生成](../../../raw/current/scripts/prefabs/charlie_hand.lua#L118-L128) |
| `注册：shadowhand_fx`（`fx.lua`） | **另一族**：通用表里的暗影手 `idle` FX，由 Wavey Jones 生成，已由既有页承载 | 同名后缀但无父子、拉伸或手臂结构关系 | [条目](../../../raw/current/scripts/fx.lua#L2239-L2242)、[[reference/prefabs/waterplant_destroy|暗影手／水流／水植特效]] |

## 典型接入

### 给自定义宿主生成拉伸手臂

- 前提与选择：宿主必须已存在且带 Transform；手臂只在主模拟有 `stretcher`，客户端只看到动画与缩放复制。需要"遮蔽"外观时换 `shadowhand_arm_shrouded`，但生成者要自己保证资产与场景一致。
- 生效与撤销：生成后立即 `SetStretchTarget(host)` 即可开始拉伸；手臂不会自删，宿主移除、场景切换或演出结束时必须显式 `Remove()`。
- 原版依据：[普通手部生成](../../../raw/current/scripts/prefabs/shadowhand.lua#L166-L173)、[查理手部生成](../../../raw/current/scripts/prefabs/charlie_hand.lua#L118-L128)。

```lua
local function SpawnStretchingArm(host, x, z)
    local arm = SpawnPrefab("shadowhand_arm") -- 遮蔽场景换 shadowhand_arm_shrouded
    arm.Transform:SetPosition(x, 0, z)
    arm.components.stretcher:SetStretchTarget(host)
    -- 可选：arm.components.highlightchild:SetOwner(host)
    return arm -- 宿主移除时调用 arm:Remove()
end
```

- 调整与验证：替换点是手臂注册名、起点与拉伸目标；拉伸比例在工厂里写死，要改需复制 `MakeShadowArm`。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：`persists=false` 不是自删。** 手臂没有生命周期代码，目标失效也不会 `Remove`；生成者忘了删就会留下一条停在原地的影子。[工厂](../../../raw/current/scripts/prefabs/shadowhand.lua#L252-L252)
- **边界：手臂不是手部的一部分。** Lua 层没有 `children` 登记、没有 `Follower`；手部靠 `OnRemoveEntity` 显式删除，不能指望引擎父子级联。[清理](../../../raw/current/scripts/prefabs/shadowhand.lua#L203-L209)
- **边界：两只手臂的 build 不能混用。** `shadowhand_arm_shrouded` 只配遮蔽手部；查理手部与普通手部都用普通 build。[注册](../../../raw/current/scripts/prefabs/shadowhand.lua#L346-L349)
- **边界：普通手部没给手臂设 `highlightchild` owner。** 组件存在但 owner 为 nil，只有查理手部显式 `SetOwner`；不要把"手臂会高亮"当通用行为。[查理接线](../../../raw/current/scripts/prefabs/charlie_hand.lua#L125-L125)
- **边界：照亮手臂会让普通／遮蔽手部消散。** 两者都监听手臂的 `enterlight` 并走 `Dissipate`；手臂不是自己监听，而是在手部退场时被驱动播放 `arm_scare` 并由手部移除。查理手部没有这条监听。这是玩法反制，不是手臂自己的清理。[监听](../../../raw/current/scripts/prefabs/shadowhand.lua#L180-L182)
- **误区：`shadowhand_fx` 是这只手臂的别名。** 它是 `fx.lua` 通用表的独立注册（bank／build 同名、`idle` 动画），生成者是 Wavey Jones，已由 [[reference/prefabs/waterplant_destroy|waterplant_destroy 页]]承载；与本页两只手臂无注册或生成关系。[条目](../../../raw/current/scripts/fx.lua#L2239-L2242)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/shadowhand.lua`（348 行）；按需读取 `prefabs/charlie_hand.lua`（生成 L118-L128、清理 L181-L185、读档重建 L153-L179）、`components/stretcher.lua` 全文、`components/highlightchild.lua`（L1-L55）、`components/shadowhandspawner.lua`（L176-L198）、`prefabs/lightsout_trial.lua`（L242-L251）、`prefabs/charlie_boss_trial.lua`（L337-L355）、`fx.lua`（`shadowhand_fx` 条目 L2239-L2242）。全域反查（`grep -rn "shadowhand_arm" --include=*.lua`）：仅 `shadowhand.lua` 的依赖／注册／`armprefab` 与 `charlie_hand.lua:119` 字面生成；`shadowhand_arm_shrouded` 只在 `shadowhand.lua` 注册与 `armprefab` 链出现，无独立生成语句。台账分诊结论（2 注册、生成 shadowhand.lua:170 动态＋charlie_hand.lua:119、NOCLICK+FX、stretcher）复核属实；补充实核：手臂无自删、普通手部不设 highlight owner、`charlie_hand` 只生成普通手臂、`shadowhand_fx` 另属 `fx.lua`。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.SHADOWHAND`＝"暗影之手"（msgid "Shadow Hand"，[L349134-L349137](../../../raw/current/scripts/languages/chinese_s.po#L349134-L349137)）、`STRINGS.NAMES.SHADOWHAND_SHROUDED`＝"恐惧之手"（msgid "Horror Hand"，[L349139-L349142](../../../raw/current/scripts/languages/chinese_s.po#L349139-L349142)）；两只手臂注册名无 `STRINGS.NAMES` 键。
- 分档说明：`size_class: standard`——两注册同构，但需承载两条生成链、拉伸／高亮／光照三套接线与"无自删"清理边界；无多状态生命周期，未升 complex。
- 未覆盖：手部本体的追火／熄灭／`playerprox` 行为、`charlie_hand` 的施工演出全流程、`stretcher` 的跨端缩放同步细节、`shadow_creatures_ground_shrouded` 动画包内容；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/components/stretcher|stretcher：朝目标拉伸动画图像]]（拉伸公式与组件边界）
- [[reference/prefabs/waterplant_destroy|暗影手／水流／水植特效]]（`shadowhand_fx` 通用表注册的既有承载页）
- 返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]（暗影之手／恐惧之手）
