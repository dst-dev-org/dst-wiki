---
title: "通用 FX 工厂（fx.lua 注册表）"
created: "2026-10-30"
updated: "2026-11-01"
type: mechanism
summary: "fx.lua 数据表经 prefabs/fx.lua 加载器批量注册动画特效的公共协议：网络代理与本地表现分工、父挂与一次性定位、音效与暂停时钟、fn／fntime 回调及三类回收。"
aliases: ["通用FX工厂", "fx.lua 通用工厂", "MakeFx", "fx_spawned", "通用特效注册", "通用 FX 工厂协议", "fx-table"]
tags: [mechanism, fx, network-sync]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "entityscript.lua", "update.lua", "scheduler.lua", "util.lua", "prefabs/oceanfishingbobber.lua", "components/playerspawner.lua", "prefabs/world.lua", "components/inspectable.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 通用 FX 工厂（fx.lua 注册表）

## 快速判断

- 职责与适用：`fx.lua` 是纯数据表，每项描述一个动画特效；`prefabs/fx.lua` 加载器 `require("fx")` 后逐项 `MakeFx`，把整表打包成一批 Prefab 返回。[加载器](../../../raw/current/scripts/prefabs/fx.lua#L165-L172)、[表收尾](../../../raw/current/scripts/fx.lua#L4203-L4203) 生成后服务端只存在一个隐形网络代理，各非专服模拟一帧后自行创建本地动画实体；本页是这份公共协议的唯一正文，具体特效族的成员与生成者见同族页。
- 优先使用：查"某通用特效如何生成／如何借用或新增动画特效"读本页；查具体特效（水花、烟团、树叶、蒸汽、生成烟雾等）选对应同族页：[[reference/prefabs/splash|splash 族]]、[[reference/prefabs/sand_puff|sand_puff 族]]、[[reference/prefabs/green_leaves|green_leaves 族]]、[[reference/prefabs/spawn_fx_small|spawn_fx 族]]、[[reference/prefabs/crater_steam_fx1|温泉蒸汽族]]；索引见 [[reference/prefabs/fx-table-index|通用表 FX 索引]]。
- 重要副作用：主模拟代理生成 1 秒后自删；本地实体挂 animover／animqueueover 自删；本地实体创建时 `TheWorld:PushEvent("fx_spawned", inst)`。[自删监听](../../../raw/current/scripts/prefabs/fx.lua#L98-L102)、[事件](../../../raw/current/scripts/prefabs/fx.lua#L116-L118)
- 修改边界：专用服务器不创建本地动画（只有代理）；本地实体无 Network、不同步、`persists=false`；`update_while_paused` 条目全部改走静态任务并 `AnimateWhilePaused(true)`。[非专服分支](../../../raw/current/scripts/prefabs/fx.lua#L127-L136)、[无 Network 实体](../../../raw/current/scripts/prefabs/fx.lua#L20-L35)

## 关键接口

数据表每项的字段都由 [startfx 与 fn 消费](../../../raw/current/scripts/prefabs/fx.lua#L18-L159)；未列字段不生效。

| 接口／配置项 | 签名或数据约束 | 消费者、条件、作用与证据 |
|---|---|---|
| `name` | 字符串，实际注册名 | `CreateEntity(t.name)` 与 `Prefab(t.name)`；主表为按序数组、尾部循环再 `table.insert`，名称唯一性由盘点核对，不由表结构保证。[注册](../../../raw/current/scripts/prefabs/fx.lua#L162-L162) |
| `bank`／`build`／`anim` | 字符串；`anim` 可为函数 | 本地实体 `SetBank`／`SetBuild`／`PlayAnimation(FunctionOrValue(anim))`；源码注释标明该求值是客户端侧函数。[动画](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)、[FunctionOrValue](../../../raw/current/scripts/util.lua#L1718-L1724) |
| `sound`／`sound2` | 事件名；`sounddelay`／`sounddelay2` 可选秒数 | 本地实体加 SoundEmitter，按延迟 `DoTaskInTime` 播放；`update_while_paused` 时改 `DoStaticTaskInTime`。[音效](../../../raw/current/scripts/prefabs/fx.lua#L43-L61) |
| `tint`／`tintalpha` | Vector3 ×RGB；alpha 缺省 1 | `SetMultColour`；只给 `tintalpha` 时按白色乘 alpha。[染色](../../../raw/current/scripts/prefabs/fx.lua#L69-L73) |
| `transform` | Vector3 | `AnimState:SetScale(t.transform:Get())`，按 x/y/z 分量分别设定三轴缩放。[缩放](../../../raw/current/scripts/prefabs/fx.lua#L75-L77) |
| `autorotate` | 布尔 | 有父级时一次性复制父级旋转，不随父级后续转向更新。[旋转](../../../raw/current/scripts/prefabs/fx.lua#L39-L41) |
| `twofaced`／`eightfaced`／`sixfaced`／`nofaced` | 布尔，互斥分支 | 代理的朝向组件；全不设时默认 `SetFourFaced`。[朝向](../../../raw/current/scripts/prefabs/fx.lua#L138-L146) |
| `update_while_paused` | 布尔 | 本地实体创建、音效、fn 延迟全部改走静态任务，动画 `AnimateWhilePaused(true)`；当前表中仅 `spawn_fx_medium_static`、`spawn_fx_ocean_static` 等少数条目使用。[静态分支](../../../raw/current/scripts/prefabs/fx.lua#L131-L135)、[例](../../../raw/current/scripts/fx.lua#L604-L611) |
| `animqueue` | 布尔 | 改听 `animqueueover` 自删，否则听 `animover`。[回收](../../../raw/current/scripts/prefabs/fx.lua#L98-L102) |
| `bloom` | 布尔 | 本地动画挂 `shaders/anim.ksh` 泛光。[泛光](../../../raw/current/scripts/prefabs/fx.lua#L94-L96) |
| `nameoverride`／`description` | 字符串／函数 | 加 `inspectable` 并改名；description 赋给 `descriptionfn`，检视时以 `(inst, viewer)` 直接调用，须为函数。这类条目不再加 FX 标签。[检视](../../../raw/current/scripts/prefabs/fx.lua#L79-L92)、[调用](../../../raw/current/scripts/components/inspectable.lua#L71-L72)、[标签条件](../../../raw/current/scripts/prefabs/fx.lua#L30-L32) |
| `build_is_skin`／`renderpath` | 布尔／资产参数 | 皮肤 build 走 DYNAMIC_ANIM 资产声明；renderpath 透传给 ANIM 资产。[资产](../../../raw/current/scripts/prefabs/fx.lua#L5-L16) |
| `fn`／`fntime` | `fn(inst, proxy)`；fntime 秒 | 回调收到本地实体与网络代理两个对象；fntime 延迟执行，`update_while_paused` 时走静态任务。[回调](../../../raw/current/scripts/prefabs/fx.lua#L104-L114) |

## 关键流程

| 入口／条件 | 当前步骤与结果 | 后续／中断／清理及证据 |
|---|---|---|
| `SpawnPrefab(注册名)`（主模拟） | `fn()` 建代理：Transform＋Network、朝向组件、FX 标签、`SetPristine`；纯客户端在此直接返回代理 | 代理自身无 AnimState，不显示任何内容。[代理构造](../../../raw/current/scripts/prefabs/fx.lua#L121-L153) |
| 非专用服务器收到代理 | 一帧后（`DoTaskInTime(0)`，暂停敏感条目用 `DoStaticTaskInTime(0)`）调用 `startfx` | 注释说明延迟是为了先落位、或生成当帧即被移除时不建表现。[延迟](../../../raw/current/scripts/prefabs/fx.lua#L127-L136) |
| `startfx(proxy)` | 建无 Network 本地实体；挂到**代理的父级**（代理无父则留在世界根）——父挂式下即宿主子实体、随宿主移动；`Transform:SetFromProxy(proxy.GUID)` 一次性复制代理位置旋转；播动画、染色、缩放、音效、fn | 父挂把代理的网络父子关系转成本地父子；`SetFromProxy` 只在创建时复制一次变换。[父挂](../../../raw/current/scripts/prefabs/fx.lua#L25-L28)、[取位](../../../raw/current/scripts/prefabs/fx.lua#L37-L37) |
| 本地动画结束 | `animover`（或 `animqueueover`）触发 `inst.Remove` | 动画是否循环、何时结束由动画数据决定，本协议不保证时长。[回收](../../../raw/current/scripts/prefabs/fx.lua#L98-L102) |
| 主模拟 1 秒 | 代理 `DoTaskInTime(1, inst.Remove)`，`persists=false` 不入存档 | 代理只是网络占位；1 秒回收不约束本地表现寿命。[代理回收](../../../raw/current/scripts/prefabs/fx.lua#L156-L157) |
| 服务器暂停 | `update_while_paused` 条目的任务在 staticScheduler 上，暂停期间 `StaticUpdate` 仍推进；普通任务在 Update 驱动的 scheduler 上 | 引擎在暂停时仍调用 StaticUpdate（注释注明此时只更新静态部分）。[DoStaticTaskInTime](../../../raw/current/scripts/entityscript.lua#L1525-L1535)、[静态调度](../../../raw/current/scripts/update.lua#L172-L185)、[调度器构造](../../../raw/current/scripts/scheduler.lua#L324-L325) |

## 典型接入

### 生成落点／父挂特效

- 前提与选择：任何已加载模拟中 `SpawnPrefab`；落点特效用 `SetPosition`，跟随宿主用 `proxy.entity:SetParent(宿主.entity)`（父挂信息由代理经网络带到客户端）。
- 生效与撤销：本地实体在代理落位一帧后出现；动画播完自删，宿主侧无需清理代理。
- 原版依据：落点例 [oceanfishingbobber](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L72-L74)；父挂例 [playerspawner](../../../raw/current/scripts/components/playerspawner.lua#L289-L291)。

```lua
local fx = SpawnPrefab("spawn_fx_small")
fx.Transform:SetPosition(x, y, z)          -- 落点式：本地表现贴代理位置
-- 或：fx.entity:SetParent(host.entity)     -- 父挂式：本地表现挂到代理的父级
```

### 新增一个通用动画特效

- 前提与选择：在 `fx.lua` 内 `return fx` 之前 `table.insert` 一项（或直接写进表字面量）即可，无需新 Prefab 文件；消费者需通过依赖声明或世界预载保证该名可生成（[world.lua 预载例](../../../raw/current/scripts/prefabs/world.lua#L199-L206)）。
- 生效与撤销：重启后由加载器自动展开为新注册名；撤销即删表项。
- 原版依据：[主表结构](../../../raw/current/scripts/fx.lua#L39-L47)、[循环注册例](../../../raw/current/scripts/fx.lua#L4047-L4055)

```lua
-- fx.lua 内、return fx 之前
table.insert(fx, {
    name = "my_fx",          -- 实际注册名
    bank = "sand_puff",
    build = "sand_puff",
    anim = "forage_out",
    sound = "dontstarve/common/deathpoof",
    fn = function(inst, proxy) inst.AnimState:SetFinalOffset(1) end,
})
```

- 调整与验证：`fn` 的 `inst` 是本地实体（各端各自创建），不要在其中写需要全服一致的状态；服务器权威逻辑应由生成者在自己的模拟里做。

## 常见陷阱与限制

- **误区：代理 1 秒回收等于特效 1 秒消失**：1 秒 `Remove` 只作用于隐形网络代理；可见寿命由本地实体的动画事件（或 fn 安装的自定义清理）决定。[代理回收](../../../raw/current/scripts/prefabs/fx.lua#L156-L157)、[动画回收](../../../raw/current/scripts/prefabs/fx.lua#L98-L102)
- **边界：跟随语义按挂接方式分两种**：落点式（生成者不父挂代理）的本地表现位置一次性；父挂式（生成者把代理 `SetParent` 到宿主）的本地表现与代理同父级、成为宿主子实体并随宿主移动。`SetFromProxy` 只负责创建时刻的变换复制；不跟随的仅是代理自身后续的独立位移。[父挂](../../../raw/current/scripts/prefabs/fx.lua#L25-L28)、[取位](../../../raw/current/scripts/prefabs/fx.lua#L37-L37)
- **边界：专用服务器完全没有本地表现**：`IsDedicated` 直接跳过 startfx，专服世界只有代理；任何"播放动画／声音"的推断都不适用于专服。[分支](../../../raw/current/scripts/prefabs/fx.lua#L127-L136)
- **边界：`fx_spawned` 每个模拟各推一次且当前无原版监听者**：事件携带的是该模拟的本地实体（不是代理）；26230496 快照 scripts 内未检索到消费者。需要拦截时应自建监听并注意多模拟语义。[推送](../../../raw/current/scripts/prefabs/fx.lua#L116-L118)
- **边界：`anim` 函数是客户端侧求值**：源码注释明确标注；各端可能得到不同动画名，不能当作服务器同步数据。[求值](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)
- **误区：本工厂管理玩法效果**：协议只建表现实体；伤害、照明、湿度等玩法作用一律由生成者或 fn 额外实现，不能从注册名推断。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。加载器 prefabs/fx.lua 全文、fx.lua 表结构与顶部 helper、G01～G05 全部 36 个条目及代表生成者／消费者核验；静态任务链读到 entityscript／scheduler／update 的调度驱动。未逐条核验其余约 468 个注册条目；各条目动画时长与循环性、晚加入时序未核验。
- 游戏运行：未运行；上表流程为源码审阅结论，未实测暂停边界与多客户端表现。

返回 [[reference/mechanisms/index|机制索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
