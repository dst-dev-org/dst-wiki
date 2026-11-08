---
title: "纸灯笼花瓣飘落特效（lantern_flower_fx_held）"
created: "2026-10-29"
updated: "2026-11-08"
type: prefab
summary: "纸灯笼皮肤提灯的飘落花瓣特效组：手持时由发射器调度 42 个本地花瓣循环飘落，落地时常驻 pre→loop 动画，KillFX 走 pst 退场。"
aliases: ["lantern_flower_fx_held", "lantern_flower_fx_ground", "纸灯笼特效", "纸灯笼花瓣", "提灯花瓣特效"]
tags: [prefab, fx, lifecycle, server-client]
size_class: standard
dst_build: "steam-buildid:26474873"
sources: ["prefabs/lantern_flower_fx.lua", "prefabs/lantern_insect_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "prefabs/mininglantern.lua", "components/equippable.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 纸灯笼花瓣飘落特效（lantern_flower_fx_held）

## 快速判断

- 职责与适用：纸灯笼（官方皮肤译名，见下）提灯的飘落花瓣特效组。一个工厂文件注册 `lantern_flower_fx_held` 与 `lantern_flower_fx_ground` 两个 Prefab：提灯点亮且被装备时生成 held，点亮且落地时生成 ground（见"注册与构造"与皮肤协议）。held 发射器本体无 AnimState，可见物是客户端本地创建的 42 个花瓣实体；ground 是联网动画实体。
- 优先使用：查"纸灯笼／花瓣飘落特效"选本页；本页同时是该同族（gothic／insect／winter）机制细节的主页，族内其他页只写自身差异并回链本页。提灯照明不来自本工厂：照明是独立 `lanternlight` 实体（`entity:AddLight()`）。[照明实体](../../../raw/current/scripts/prefabs/mininglantern.lua#L206-L230)
- 重要副作用：花瓣只在有图形端的机器上由发射器用 DoTaskInTime 拉起并按 animover 自我重播；专服上 held 只剩联网空壳。地面实体靠 animover 在 petal_loop 上自循环，KillFX 只是标记 killed，等播 pst 后才 Remove。
- 修改边界：本工厂无 AddLight／Light 组件、无 SoundEmitter；花瓣实体非联网、不入存档（persists=false）。动画循环参数与端分界见下表；换肤清理协议在 [lantern_clear_fn](../../../raw/current/scripts/prefabskin.lua#L3501-L3515)。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lantern_flower_fx_held` | heldfn → Transform／Network＋FX 标签 → 非专服调度 6 股×7 变体花瓣（本地实体，非 SpawnPrefab）→ SetPristine → 主模拟 persists=false | 发射器本体无 AnimState；不安装 KillFX；花瓣 bank lantern_flower_fx、build lantern、符号 petal、动画 petalfall1..7。[发射器](../../../raw/current/scripts/prefabs/lantern_flower_fx.lua#L71-L104)、[花瓣构造](../../../raw/current/scripts/prefabs/lantern_flower_fx.lua#L35-L58) |
| `lantern_flower_fx_ground` | groundfn → Transform／AnimState／Network＋FX 标签 → petal_pre → SetPristine → 主模拟 POPULATING 直入 petal_loop 随机帧、监听 animover、安装 KillFX | 联网动画实体；persists=false；动画 pre／loop／pst 未显式传循环参数，loop 靠 animover 重播。[构造](../../../raw/current/scripts/prefabs/lantern_flower_fx.lua#L119-L151)、[注册](../../../raw/current/scripts/prefabs/lantern_flower_fx.lua#L153-L154) |

皮肤键 `STRINGS.SKIN_NAMES.lantern_flower` 为"纸灯笼"（官方译名）；未发现 FX 注册名的直接 NAMES 翻译条目，标题"花瓣飘落特效"是按 petalfall 动画名写的已核验效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L398294-L398297)。alt 皮肤 `lantern_flower_alt`（build_name_override 同为 lantern_flower）复用同一组 FX。[alt 定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L7343-L7353)

### 与同族的同构关系

对 lantern_flower／gothic／insect／winter 四个工厂逐对 diff（去除皮肤名／bank／符号／动画前缀后）结论：四文件结构近同构——相同的 KillFX、IsMovingStep（step 0 与 3 恒显、其余步随 ismoving 显隐）、animover 重播协议、heldfn 6 股×7 变体调度（股距 86/6 帧，帧偏移 1/7/13/30/41/58/67）、CheckMoving、groundfn pre→loop→pst 与 POPULATING 随机帧，仅下列参数不同：

| 工厂文件 | bank | 花瓣符号覆盖 | 花瓣动画前缀 | 注册名 |
|---|---|---|---|---|
| lantern_flower_fx.lua | `lantern_flower_fx` | `petal` | `petalfall` | …_fx_held／…_fx_ground |
| lantern_gothic_fx.lua | `lantern_gothic_fx` | `glowflake` | `glowfall` | 同上结构 |
| lantern_insect_fx.lua | `lantern_insect_fx` | `glowflake` | `petalfall` | 同上结构 |
| lantern_winter_fx.lua | `lantern_winter_fx` | `snowflake` | `snowfall` | 同上结构 |

注意 insect 混搭：符号用 glowflake（同 gothic）、动画前缀用 petalfall（同 flower）。[insect 花瓣](../../../raw/current/scripts/prefabs/lantern_insect_fx.lua#L46-L52)。各页：[[reference/prefabs/lantern_gothic_fx_held]]／[[reference/prefabs/lantern_insect_fx_held]]／[[reference/prefabs/lantern_winter_fx_held]]。crystal 与 tesla 是另外两种更简单的工厂，不在此同构组：[[reference/prefabs/lantern_crystal_fx_ground]]／[[reference/prefabs/lantern_tesla_fx_held]]。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 来源皮肤：lantern_flower | fx_prefab 表第 1／2 项经 SKIN_FX_PREFAB 保存，lantern_init_fn 读出为 _heldfx／_groundfx | 仅主模拟（init_fn 有 ismastersim 分界）；提灯无皮肤时不接 FX | [皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L7331-L7341)、[映射保存](../../../raw/current/scripts/prefabskin.lua#L4094-L4096)、[init_fn](../../../raw/current/scripts/prefabskin.lua#L3475-L3499) |
| 宿主事件：lantern_on／lantern_off | 提灯 turnon／turnoff 推送事件，宿主监听后生成／Kill 本组 FX；unequipped 与 onremove 也触发 lantern_off | 仅主模拟；点亮前提（fueled 非空）在 turnon 内 | [开](../../../raw/current/scripts/prefabs/mininglantern.lua#L64-L89)、[关](../../../raw/current/scripts/prefabs/mininglantern.lua#L91-L110)、[监听](../../../raw/current/scripts/prefabskin.lua#L3489-L3492)、[unequipped 推送](../../../raw/current/scripts/components/equippable.lua#L122) |
| held 装载协议 | SpawnPrefab(held) → SetParent(owner)＋Follower:FollowSymbol(owner, "swap_object", 67, -7, 0) | Follower 由宿主添加；本工厂不建 Follower | [装载](../../../raw/current/scripts/prefabskin.lua#L3437-L3450) |
| ground 落地协议 | SpawnPrefab(ground) → SetParent(提灯本体)；有 KillFX 时再监听提灯 enterlimbo 记录 _lastpos | 仅 ground 装此监听；held 不装 | [落地](../../../raw/current/scripts/prefabskin.lua#L3456-L3470)、[enterlimbo](../../../raw/current/scripts/prefabskin.lua#L3390-L3409) |
| 移动门控 | CheckMoving 每帧比较发射器父级（持有者）位置得 ismoving；step 1/2/4/5 的花瓣仅移动时 Show | 仅非专服；step 0/3 恒显 | [CheckMoving](../../../raw/current/scripts/prefabs/lantern_flower_fx.lua#L60-L69)、[调度](../../../raw/current/scripts/prefabs/lantern_flower_fx.lua#L80-L93) |
| 照明分工 | 本组 FX 无 Light；照明由 turnon 生成的 lanternlight 实体承担（强度／半径随燃料插值） | lanternlight 独立联网实体，非本工厂产物 | [照明实体](../../../raw/current/scripts/prefabs/mininglantern.lua#L206-L230)、[燃料插值](../../../raw/current/scripts/prefabs/mininglantern.lua#L34-L41) |
| 资源 | anim/dynamic/lantern_flower.zip＋PKGREF lantern_flower.dyn；bank lantern_flower_fx、build lantern | 声明与使用已核验，包内容未解包 | [资源声明](../../../raw/current/scripts/prefabs/lantern_flower_fx.lua#L1-L5) |

## 典型接入

### 复用花瓣流或调整密度

- 前提与选择：模组皮肤要仿"边走边飘落"效果时，照抄 heldfn 协议：发射器为联网 FX 实体，花瓣用普通 CreateEntity 本地函数创建（不经 SpawnPrefab、无 Network），持有者移动状态由 CheckMoving 写入发射器字段。
- 生效与撤销：花瓣随 animover 重播把自身 SetPosition 到发射器世界坐标（非父子跟随）；发射器被 Remove 后，花瓣下次 animover 检测到 petalemitter 失效即 Remove。[重播与跟随](../../../raw/current/scripts/prefabs/lantern_flower_fx.lua#L19-L33)
- 原版依据：宿主侧装载／清理见上表；换肤走 lantern_clear_fn 统一 lantern_off。

```lua
-- 股间隔与变体偏移（源码常量，单位帧）
-- 自有工厂内：inst 为发射器，CreatePetal 为本地花瓣构造函数。
-- 仅在非专服分支执行这份调度；这里只示范每股的第一个变体。
for i = 0, 5 do
    local delay = i * 86 / 6 * FRAMES
    inst:DoTaskInTime(delay + FRAMES, CreatePetal, 1, i)
end
-- 其余变体按 7/13/30/41/58/67 帧偏移调度。
```

- 调整与验证：改股数／偏移只动 heldfn 调度表；花瓣动画名 petalfall1..7 与 bank 内动画段一一对应，bank 内容未解包，新增变体需自备动画。

## 常见陷阱与限制

- **误区：花瓣是联网实体**：CreatePetal 创建的是无 Network 组件的本地实体（源码注释 Non-networked entity），专服上完全不存在；只有发射器本体联网。想同步花瓣状态必须自建 netvar 协议。[花瓣构造](../../../raw/current/scripts/prefabs/lantern_flower_fx.lua#L35-L58)
- **误区：KillFX 立即删除地面实体**：ground 的 KillFX 在存活大于 0 帧时只置 `killed = true`，由 animover 处理器切 petal_pst、播完才 Remove；刚生成就 Kill 才直接 Remove。held 变体没装 KillFX，宿主 lantern_off 直接 Remove。[KillFX](../../../raw/current/scripts/prefabs/lantern_flower_fx.lua#L7-L13)、[宿主分派](../../../raw/current/scripts/prefabskin.lua#L3411-L3429)
- **边界：宿主符号覆盖对 held 跳过**：lantern_on 覆盖皮肤符号前检查 `AnimState ~= nil`，held 发射器没有 AnimState，实际符号覆盖由每个花瓣各自 OverrideItemSkinSymbol 完成。[宿主检查](../../../raw/current/scripts/prefabskin.lua#L3441-L3445)、[花瓣覆盖](../../../raw/current/scripts/prefabs/lantern_flower_fx.lua#L48)
- **边界：拾起位置还原依赖 enterlimbo**：拾起点亮提灯时变换先被清、lantern_off 后到，ground 实体靠 enterlimbo 记录的 _lastpos（含父级旋转换算）落回拾起点再播 pst；绕过宿主协议直接操作 FX 不会得到该还原。[enterlimbo](../../../raw/current/scripts/prefabskin.lua#L3390-L3409)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；完整 155 行工厂、flower／flower_alt 皮肤定义与 fx_prefab 映射、SKIN_FX_PREFAB 保存、lantern_init_fn／on／off／enterlimbo／clear_fn、mininglantern turnon／turnoff／lanternlight／ondropped、equippable 的 unequipped 推送点、皮肤译名检索已核验；gothic／insect／winter 三文件逐对 diff 同构已核验。未覆盖：动画包内容未解包、最终视觉、花瓣重播时序实测、引擎侧 skin init_fn 调用点。
- 游戏运行：未运行。
- 结构检查：validate_page 与 validate_source_links --strict 通过（实际结果见本批交付报告）。

返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-03）。
