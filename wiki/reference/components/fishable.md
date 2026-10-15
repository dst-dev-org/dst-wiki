---
title: fishable
created: 2026-10-21
updated: "2026-10-28"
type: component
summary: 为鱼塘／绿洲湖宿主提供鱼量计数、上钩实体化、放生与定时刷新的服务端组件
aliases: ["鱼塘鱼量", "pond 钓鱼", "上钩接口 HookFish", "鱼群刷新"]
tags: [component, lifecycle, loot]
dst_build: "steam-buildid:26230496"
sources: ["components/fishable.lua", "components/fishingrod.lua", "prefabs/pond.lua", "prefabs/oasislake.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# fishable

## 快速判断

- 职责与适用：维护一座水域的鱼量（`fishleft`／`maxfish`）、按鱼表抽出鱼种、在上钩时把鱼实体化并挂到宿主下；只管"鱼"这一侧，钓竿状态机见 [[reference/components/fishingrod]]。
- 优先使用：宿主已带组件时用 `AddFish`、`SetGetFishFn`、`SetRespawnTime` 改鱼种与节奏；钓竿侧只判 `target.components.fishable`，入口见下文接口表。
- 重要副作用：`AddComponent` 后宿主带 `fishable` 标签；`HookFish` 立即 `SpawnPrefab` 并 `AddChild`、扣 `fishleft`；`frozen` 属性直接增删该标签。
- 修改边界：只在服务端创建（`pond`／`oasislake` 都在 `TheWorld.ismastersim` 之后添加），客户端只有标签；组件不推事件，刷新靠 `DoTaskInTime`。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `fish`／`maxfish`／`fishleft` | table／number／number | 构造默认 `maxfish=10`、`fishleft=10`；`fish` 是"prefab 名→prefab 名"集合 [构造](../../../raw/current/scripts/components/fishable.lua#L9-L22) |
| `AddFish(prefab)` | 无返回 | 只写 `fish`，不立即生成实体 [AddFish](../../../raw/current/scripts/components/fishable.lua#L33-L35) |
| `SetGetFishFn(fn)` | `fn(inst, fisherman)` 返回 prefab 名 | 覆盖随机取鱼；`oasislake` 用 `TryLuckRoll` 决定 `wetpouch` 或 `pondfish` [GetFish](../../../raw/current/scripts/prefabs/oasislake.lua#L195-L196) |
| `SetRespawnTime(time)` | 秒 | 只存值，`RefreshFish` 时才 `DoTaskInTime` [SetRespawnTime](../../../raw/current/scripts/components/fishable.lua#L41-L43) |
| `HookFish(fisherman)` | 返回实体或 `nil` | `SpawnPrefab`＋`AddChild`＋`Hide`＋`persists=false`，关阴影与物理，给 `weighable` 设 owner；`fishleft>0` 时减 1 [HookFish](../../../raw/current/scripts/components/fishable.lua#L58-L80) |
| `ReleaseFish(fish)` | 无返回 | 仅命中 `hookedfish[fish]` 时 `Remove`，并把 `fishleft`＋1（不超过 `maxfish`）[ReleaseFish](../../../raw/current/scripts/components/fishable.lua#L82-L90) |
| `RemoveFish(fish)` | 返回 fish 或 `nil` | 解除父子关系并返回鱼；不恢复显示／物理／`persists`（注释点名交给钓竿），无刷新任务时补 `RefreshFish` [RemoveFish](../../../raw/current/scripts/components/fishable.lua#L92-L110) |
| `GetFishPercent()` | `fishleft/maxfish` | 钓竿用它把等待时间映射到 0..1 [WaitForFish](../../../raw/current/scripts/components/fishingrod.lua#L112-L124) |
| `frozen`（属性）／`Freeze()`／`Unfreeze()` | 布尔 | 赋值即触发 `onfrozen`：`true` 移除、`false` 添加 `fishable` 标签 [onfrozen](../../../raw/current/scripts/components/fishable.lua#L1-L7) |
| `IsFrozenOver()` | 布尔 | 只读 `frozen` 字段，不碰鱼量 [IsFrozenOver](../../../raw/current/scripts/components/fishable.lua#L113-L115) |
| `OnSave`／`OnLoad` | 未满时 `{ fish = fishleft }` | 满塘不写存档；读档后 `RefreshFish` [OnSave](../../../raw/current/scripts/components/fishable.lua#L135-L146) |

## 典型接入

### 在原版鱼塘追加鱼种、调刷新间隔

- 前提与选择：目标文件 `modmain.lua`；`AddPrefabPostInit` 命中已注册宿主 `pond`／`pond_mos`，此时组件已建好、`OnLoad` 尚未跑。组件只在服务端存在，函数内先判 `TheWorld.ismastersim`。
- 生效与撤销：`AddFish` 立即进入 `fish` 集合，下一次 `HookFish` 就可能抽到；`SetRespawnTime` 只影响之后调度的刷新，单位秒（原版 `TUNING.FISH_RESPAWN_TIME`＝`day_time/3`，即 100 s）。撤销＝去掉 postinit 或写回原值，已存进档的 `fishleft` 不变。
- 原版依据：[pond 添加组件与刷新时间](../../../raw/current/scripts/prefabs/pond.lua#L264-L265)、[pond_mos 追加 pondfish](../../../raw/current/scripts/prefabs/pond.lua#L328-L329)。

```lua
-- modmain.lua：向原版鱼塘追加鱼种并缩短刷新间隔
local function TweakPond(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local fishable = inst.components.fishable
    if fishable == nil then
        return
    end
    fishable:AddFish("my_mod_fish")                        -- 自定义鱼 prefab，需先注册
    fishable:SetRespawnTime(GLOBAL.TUNING.FISH_RESPAWN_TIME / 2)  -- 单位：秒
end

AddPrefabPostInit("pond_mos", TweakPond)
AddPrefabPostInit("pond", TweakPond)
```

- 调整与验证：`my_mod_fish` 是占位名，需先注册可 `SpawnPrefab` 的 prefab；组件只把它塞进 `fish` 表，能否被 `AddChild` 以及 `Physics`／`DynamicShadow` 是否可选由你的 prefab 决定。原版洞窟塘用 `pondeel`（[pond](../../../raw/current/scripts/prefabs/pond.lua#L480)），不要把它当作地表塘的新鱼种。本例未运行，只做语法检查。

### 原版宿主对照

| 宿主（注册名） | 组件用法 | 证据 |
|---|---|---|
| `pond`／`pond_mos`（蛙塘／蚊塘） | 默认 10 条，`AddFish("pondfish")`；冬季 `snowlevel>0.02` 时 `Freeze()`，回暖 `Unfreeze()` | [OnSnowLevel](../../../raw/current/scripts/prefabs/pond.lua#L145-L170) |
| `pond_cave`（硝石塘） | `AddFish("pondeel")`；硝石状态在 `SetAcidic_Cave`／`SetBackToNormal_Cave` 里切冻结 | [SetAcidic_Cave](../../../raw/current/scripts/prefabs/pond.lua#L398-L405) |
| `oasislake` | `maxfish=15`、90 s（`seg_time*3`）、`SetGetFishFn` 换 `wetpouch`；干涸时 `Freeze()` | [组件配置](../../../raw/current/scripts/prefabs/oasislake.lua#L246-L249) |

注册名与工厂见 [pond 的 prefab 表](../../../raw/current/scripts/prefabs/pond.lua#L508-L510) 与 [oasislake 返回](../../../raw/current/scripts/prefabs/oasislake.lua#L274)。

## 常见陷阱与限制

- `Freeze()` 不等于不能钓：只切标签、不动 `fishleft`、不拦 `HookFish`；玩家侧动作在 [componentactions](../../../raw/current/scripts/componentactions.lua#L1263-L1269) 按标签决定 `FISH`／`OCEAN_FISHING_POND`，但脚本直接调方法仍会生成鱼。
- 冻结状态不入存档：`OnSave` 只写 `fishleft`；`pond` 靠 `OnSnowLevel`、`oasislake` 靠 `OnIsWet` 在读档后重算。自定义宿主必须自己重新 `Freeze()`。
- `RemoveFish` 不还原实体：显示、阴影、`Physics:SetActive`、`persists` 由 [fishingrod:Reel](../../../raw/current/scripts/components/fishingrod.lua#L166-L185) 与 [Collect](../../../raw/current/scripts/components/fishingrod.lua#L187-L201) 补；自写消费者漏做会让鱼永久隐藏且不入存档。
- `maxfish` 改晚不回填：`oasislake` 把上限设成 15，但构造时 `fishleft` 已是 10，新世界上初始就是 10/15；刷新任务要等 `OnLoad` 或首次 `RemoveFish` 才启动，之后每次 +1 追平。
- 空塘不可钓：`fishleft==0` → `GetFishPercent()==0` → `WaitForFish` 不排 nibble 任务（[条件](../../../raw/current/scripts/components/fishingrod.lua#L114-L121)），玩家只会空等。
- 多鱼种抽取无序：`GetRandomKey(self.fish)` 遍历哈希键，均匀但与插入顺序无关；要控制概率就 `SetGetFishFn`。
- 无事件：`HookFish`／`ReleaseFish`／`RespawnFish` 都不 `PushEvent`；要反馈就听钓竿的 `fishingstrain`、`fishingcatch`、`fishingcollect`。

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 0 处——本页引用的 componentactions fishingrod 候选段（1263-1269）行号与内容未变（本版 componentactions 改动均在 474/657/882 等价重写与 1574+ 新增段），fishable.lua、fishingrod.lua、pond.lua、oasislake.lua 全部未变；FISH_RESPAWN_TIME 无行锚点且不在 tuning-hunk-map 5 改值键内。依据：tuning-hunk-map 键级审阅＋变更清单命中 componentactions.lua、tuning.lua。
- 源码审阅：2026-10-21，steam-buildid:25643519，逐行读 `components/fishable.lua`（148 行），并与 `fishingrod.lua`、`prefabs/pond.lua`、`prefabs/oasislake.lua`、`componentactions.lua` 的调用点核对；覆盖鱼量增减、冻结标签、刷新调度、存档字段、空塘行为，未覆盖多人并发与替换组件的模组。
- 游戏运行：未运行。
- 结构检查：validate_page.py 本页 0 error／0 warning（93 行，低于 180 行／14 KiB 上限）；validate_source_links.py --strict 0 error／0 warning，本页源码引用 23 处（唯一目标 23 个，覆盖 5 个源文件）。
