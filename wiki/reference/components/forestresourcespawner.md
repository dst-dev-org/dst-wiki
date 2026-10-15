---
title: "forestresourcespawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级基础资源再生器：在世界出生点登记的点位附近，按可再生集合随机补充燧石、树枝、草与浆果丛，受资源再生世界设置开关。"
aliases: ["基础资源再生", "资源补充器", "forestresourcespawner"]
tags: [component, mechanism, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/forestresourcespawner.lua", "prefabs/forest.lua", "prefabs/cave.lua", "worldsettings_overrides.lua", "prefabs/spawnpoint_multiplayer.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# forestresourcespawner

## 快速判断

- 职责与适用：挂在 TheWorld（地表 forest.lua:665 与洞穴 cave.lua:537 都挂）的常驻再生器，仅 mastersim（components/forestresourcespawner.lua:7）。只在"出生点"登记点位附近补充四组可再生资源（lines 25-43）。
- 优先使用：开关走世界设置——`resource_renewal` 设为 `always` 时经 `ms_enableresourcerenewal` 事件启动（worldsettings_overrides.lua:3819-3824），默认档映射为 none 即关闭。
- 重要副作用：周期任务永久自续（lines 104-119）；在无人 240 距离内才执行（lines 19、111），生成前查可种植、部署间距与道路（lines 90-94）。
- 修改边界：注册点被移除自动注销（lines 138-143）；组件无存读钩子，重启后依赖事件重新开启（构造尾部 `if _enabled` 恒为 false，lines 172-174）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `RENEWABLES`（内部常量） | 四组 `{spawns, matches}` | 燧石；树苗×2+twiggytree；草；浆果丛×2+juicy。matches 含衍生 prefab（twigs、cutgrass、depleted_grass、grassgekko、berrybush2 等），区域内有任一匹配即跳过该组（lines 25-51、75-79） |
| `ms_registerspawnpoint` 事件 | `(spawnpt 实体)` | 登记再生锚点；原版由 spawnpoint_multiplayer prefab 构造时推送（prefabs/spawnpoint_multiplayer.lua:13；handler lines 145-152） |
| `ms_enableresourcerenewal` 事件 | `(bool)` | 启停周期任务（handler lines 154-163、监听 line 170） |
| 周期 | `SEG_TIME * (1 + 随机)` | 每次执行后重排下一轮（lines 69-71、118） |
| 选点算法 | `easing.inQuint` 随机 + 轮转 | 偏向表头点位并轮换顺序，保证均衡（lines 105-108） |
| 单次生成量 | 每组最多 3 个 | `math.random(max)` 决定本轮尝试数（lines 84、114） |
| 生成范围 | `RENEW_RADIUS = 60` | 锚点周围随机圆内（lines 20、86-88） |

## 典型接入

### 为模组世界开启并扩展再生（服务端）

- 前提与选择：组件随 forest／cave 世界自动存在，模组通常只需推事件或登记锚点；想用自己的点位再生，可生成会自推 `ms_registerspawnpoint` 的 `spawnpoint_multiplayer`，或直接对组件宿主推事件。
- 生效与撤销：`ms_enableresourcerenewal(true)` 后任务开始；传 false 取消任务。重启世界后需重新推事件（无存档）。
- 原版依据：worldsettings_overrides.lua:3819-3824 是唯一开启方；洞穴共用同一套地表资源表（cave.lua:537 注释自认"would be nice"有待改进）。

```lua
AddPrefabPostInit("forest", function(inst)
    if not inst.ismastersim then
        return
    end
    inst:PushEvent("ms_enableresourcerenewal", true)
end)

-- 登记自定义再生锚点（在任意 master 端逻辑中）：
local marker = SpawnPrefab("spawnpoint_multiplayer")
marker.Transform:SetPosition(100, 0, 100) -- 构造时已自推 ms_registerspawnpoint
```

- 调整与验证：锚点必须在世界实体上监听（组件挂在 TheWorld）；`resource_renewal` 世界设置与手动事件都会改变同一 `_enabled` 状态（lines 154-163）。

## 常见陷阱与限制

- **只认登记锚点：** 没有任何 `ms_registerspawnpoint` 点位时，任务空转（`_spawnpts` 为空，lines 105-110），不会全图再生。
- **不区分地表／洞穴：** 洞穴也会刷燧石／草等同一套表（cave.lua:537），依赖生成点过滤避免穿模，但生物群系适配上与地表一致。
- **开启状态不存档：** `_enabled` 无 OnSave；依赖世界设置在每次载入时重推（worldsettings_overrides 的执行时机），模组手动开启需自处理重载。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。通读 components/forestresourcespawner.lua 全文（180 行）；核验 forest.lua:665、cave.lua:537、spawnpoint_multiplayer.lua:13、worldsettings_overrides.lua:3819-3824。未运行游戏。
- 结构检查：validate_page.py / validate_source_links.py 通过（见批级自验）。

相关：[[reference/components/plantregrowth]]（另一套基于实体自身的植物再生系统）
