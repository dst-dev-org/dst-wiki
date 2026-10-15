---
title: "oceanfishingtackle"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "海钓渔具数据载体：给鱼漂／鱼饵挂投掷参数与诱饵数据，海钓杆读取后合成抛竿距离、弹道与诱饵效果；一次性渔具用后删除。"
aliases: ["oceanfishingtackle", "海钓渔具", "鱼漂鱼饵数据"]
tags: [component, item]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/oceanfishingtackle.lua", "components/oceanfishingrod.lua", "prefabs/oceanfishingrod.lua", "prefabs/trinkets.lua", "prefabs/seeds.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# oceanfishingtackle

## 快速判断

- 职责与适用：挂在可装入海钓杆的渔具物品（鱼漂 bobber／鱼饵 lure）上的数据组件；字段由宿主 prefab 写入，海钓杆读取后叠加投掷距离与装配弹道预制件。[数据消费](../../../raw/current/scripts/components/oceanfishingrod.lua#L139-L150)
- 优先使用：自定义鱼漂走 `SetCastingData(casting_data, projectile_prefab)`，自定义鱼饵走 `SetupLure(data)`；一次性鱼饵在 data 里带 `single_use = true`。[接口](../../../raw/current/scripts/components/oceanfishingtackle.lua#L19-L31)
- 重要副作用：组件无行为逻辑；`IsSingleUse()` 由海钓杆在收线时消费——命中即把该物品从杆容器删除。[消耗](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L128-L133)
- 修改边界：全部在服务端读取；候选动作要求杆已打开容器且带 `accepts_oceanfishingtackle` 标签，客户端右键菜单依赖 replica 容器状态。[候选](../../../raw/current/scripts/componentactions.lua#L2823-L2826)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetCastingData(data, projectile_prefab)` | `data: {dist_max, max_dist_offset, max_angle_offset}` | 鱼漂数据；rod 把 `casting_data` 逐项加到基值、记录弹道预制件。字段形状见组件内注释模板（仅注释，未初始化）。[实现](../../../raw/current/scripts/components/oceanfishingtackle.lua#L19-L22) [消费](../../../raw/current/scripts/components/oceanfishingrod.lua#L139-L147) |
| `SetupLure(data)` | `data: {lure_data, single_use?, fns?, ...}` | `self.lure_data = data.lure_data`，整个 data 存为 `lure_setup`（rod 还会读 `lure_setup.fns`）。[实现](../../../raw/current/scripts/components/oceanfishingtackle.lua#L24-L27) [fns 消费](../../../raw/current/scripts/components/oceanfishingrod.lua#L125-L127) |
| `IsSingleUse()` | `-> boolean` | `lure_setup ~= nil and lure_setup.single_use or false`；鱼漂类不设置。[实现](../../../raw/current/scripts/components/oceanfishingtackle.lua#L29-L31) |

## 典型接入

### 给自制物品接海钓杆（原版装配参照）

- 前提与选择：一次性鱼饵仿 seeds——`SetupLure` 带 `single_use = true` 与 tuning 里的 lure_data；可复用鱼漂仿 trinkets——`SetCastingData` 用 OCEANFISHING 数据表。[seeds](../../../raw/current/scripts/prefabs/seeds.lua#L162-L163) [trinkets](../../../raw/current/scripts/prefabs/trinkets.lua#L165-L171)
- 生效与撤销：装进杆容器后 rod 的 `_CacheTackleData` 即时合成参数；单次鱼饵在收线环节被删除，复用渔具持续保留。[缓存](../../../raw/current/scripts/components/oceanfishingrod.lua#L139-L150)

```lua
-- 服务端：一次性鱼饵（仿 seeds 装配）
inst:AddComponent("oceanfishingtackle")
inst.components.oceanfishingtackle:SetupLure({
    build = "oceanfishing_lure_mis",
    symbol = "hook_seeds",
    single_use = true,
    lure_data = TUNING.OCEANFISHING_LURE.SEED,
})
```

- 调整与验证：原版宿主覆盖 trinkets 鱼漂／鱼饵、seeds、veggies、twigs、spoiledfood 及专用 oceanfishingbobber／oceanfishinglure；lure_data 结构与效果函数在 oceanfishinglure 的数据表内维护，本组件不校验字段完整性。[专用鱼饵](../../../raw/current/scripts/prefabs/oceanfishinglure.lua#L90-L91)

## 常见陷阱与限制

- **以为组件自带默认投掷参数：** 构造函数无字段初始化（注释模板不算赋值）；没调 SetupLure／SetCastingData 的渔具对 rod 不可见数据，rod 侧逐处判空回退默认值。[空构造](../../../raw/current/scripts/components/oceanfishingtackle.lua#L4-L17)
- **single_use 期望作用于鱼漂：** `IsSingleUse` 只读 `lure_setup`，而 rod 的消耗检查对容器内所有渔具一视同仁——鱼漂若被塞了 single_use 的 setup 同样会被删；语义上应只给鱼饵配置。[消耗检查](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L128-L133)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（32 行），核对 rod 组件的参数合成与读取、rod prefab 的单次消耗、trinkets／seeds 装配、inventory 右键候选；grep 确认 veggies／twigs／spoiledfood／oceanfishingbobber／oceanfishinglure 装配行。未审阅 lure_data 各效果函数体。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行；抛竿参数未实测。
