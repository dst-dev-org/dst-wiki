---
title: "酸雨落点烟与蘑菇树酸烟（acidraindrop）"
created: "2026-11-02"
updated: "2026-11-07"
type: prefab
summary: "洞穴酸雨落点池化烟与蘑菇树酸滴循环烟的两个本地 Prefab：重播、显示门禁和回收。"
aliases: ["acidraindrop", "acidsmoke_endless", "酸雨落点烟", "蘑菇树酸烟"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/acidraindrop.lua", "prefabs/caveacidrain.lua", "components/caveweather.lua", "prefabs/mushtree.lua", "prefabs/mushtree_webbed.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 酸雨落点烟与蘑菇树酸烟（acidraindrop）

## 快速判断

- 两个注册都创建非联网、不可休眠且不存档的本地动画实体：`acidraindrop` 是可池化的酸雨落点烟，`acidsmoke_endless` 是蘑菇树符号上的重复烟。两者均不承担酸雨伤害或地形玩法。[工厂](../../../raw/current/scripts/prefabs/acidraindrop.lua#L34-L55)、[循环工厂](../../../raw/current/scripts/prefabs/acidraindrop.lua#L76-L100)
- 中文标题和别名是按源码用途核验的效果描述，非官方译名；简中 `languages/chinese_s.po` 按两个注册名的显示名键检索未命中。动画包内部外观与时长未核验。

## 注册与构造

| 注册名 | 初始化与差异 | 证据 |
|---|---|---|
| `acidraindrop` | Transform、AnimState、SoundEmitter；bank／build `nitre_smoke_fx`；`RestartFx` 以 0.1～0.5 的随机比例播放 `smoke_1`～`smoke_3`，并排任务播酸液滋声，音量为 0.5 加比例。动画结束时池有效则移出场景并回池，否则移除。 | [重播](../../../raw/current/scripts/prefabs/acidraindrop.lua#L20-L32)、[构造](../../../raw/current/scripts/prefabs/acidraindrop.lua#L34-L55)、[回池](../../../raw/current/scripts/prefabs/acidraindrop.lua#L11-L18) |
| `acidsmoke_endless` | Transform、AnimState；bank／build `nitre_smoke_dense_fx`，复用随机烟动画和缩放；没有 SoundEmitter，故重播不播放滋声。动画结束时若已请求隐藏则隐藏，否则延迟 3～5 秒再次重播。 | [重播](../../../raw/current/scripts/prefabs/acidraindrop.lua#L24-L32)、[隐藏与重播](../../../raw/current/scripts/prefabs/acidraindrop.lua#L58-L74)、[构造](../../../raw/current/scripts/prefabs/acidraindrop.lua#L76-L100) |

## 关联机制导航

| 生成者／关系 | 触发、选名和清理 | 证据 |
|---|---|---|
| 洞穴酸雨 | `caveacidrain` 的 `SpawnRaindropAtXZ` 通过 `GetPooledFx("acidraindrop", pool)` 复用或新建实体，落在 x/z、y=0；普通采样点有雨棚时跳过原点并缓存雨棚，否则须通过可通行与酸雨门禁。雨棚边缘另采样，须可通行、未被雨棚遮挡且允许酸雨。洞穴天气根据降水量调整喷溅速率。 | [池与落点](../../../raw/current/scripts/prefabs/caveacidrain.lua#L54-L79)、[采样](../../../raw/current/scripts/prefabs/caveacidrain.lua#L199-L230)、[天气速率](../../../raw/current/scripts/components/caveweather.lua#L700-L715) |
| 池的结束 | 酸雨实例移除时清理池中实体并使池失效；喷溅速率为零时也清空池中实体。仍在播放而未回池的实体在动画结束时看到池失效后移除。 | [移除](../../../raw/current/scripts/prefabs/caveacidrain.lua#L66-L84)、[停止喷溅](../../../raw/current/scripts/prefabs/caveacidrain.lua#L250-L254)、[动画结束](../../../raw/current/scripts/prefabs/acidraindrop.lua#L11-L18) |
| 蘑菇树酸滴 | 普通蘑菇树和蛛网蘑菇树仅在非专用服各创建三个 `acidsmoke_endless`，给实体本身添加 Follower 并绑定酸滴符号；树移除时清理烟实体。同步的酸滴数量驱动显示／排队隐藏，普通树还根据开花状态切换跟随符号。 | [普通树生成](../../../raw/current/scripts/prefabs/mushtree.lua#L33-L43)、[安装](../../../raw/current/scripts/prefabs/mushtree.lua#L717-L726)、[显示与换符号](../../../raw/current/scripts/prefabs/mushtree.lua#L498-L523)、[蛛网树生成](../../../raw/current/scripts/prefabs/mushtree_webbed.lua#L28-L38)、[安装](../../../raw/current/scripts/prefabs/mushtree_webbed.lua#L290-L297) |

## 常见陷阱与限制

- `DoCustomHide` 只设置待隐藏标记；`animover` 会消费标记并隐藏，`DoCustomShow` 也会先显示再直接执行同一处理，若标记仍在便会立即隐藏。隐藏不会取消已排的重播任务；重复 Show 也可能继续排任务，这些操作都不等同销毁。[方法](../../../raw/current/scripts/prefabs/acidraindrop.lua#L58-L74)
- 酸雨发射器的 `SetMaxLifetime` 是粒子配置，不是本页两个 Prefab 的实体移除时间；落点烟的回池和蘑菇树烟的隐藏／移除各走上述路径。[粒子配置](../../../raw/current/scripts/prefabs/caveacidrain.lua#L103-L111)

## 验证记录

- 源码审阅：2026-11-02，Steam buildid 26335394；检查两个工厂、池管理和树上的消费者；在 `raw/current/scripts` 全树检索两个完整注册名、`acidraindrop`／`acidsmoke` 与相关重播方法，区分依赖声明、动态池生成和真实调用。动画资源内部及模组动态调用未核验。
- 游戏运行：未运行；保持 `source-reviewed`。

返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]。
