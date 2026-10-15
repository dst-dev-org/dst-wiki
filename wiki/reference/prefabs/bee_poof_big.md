---
title: "蜂蜜飞溅、蜂群烟与病害蝇群烟（bee_poof 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "fx.lua 通用表相邻四注册：巨大蜂窝受击蜂蜜飞溅、蜂王护卫聚焦变身烟与图书召蜂烟、病害关联的蝇群腾起烟（当前无生成点缺口）。"
aliases: ["bee_poof_big", "bee_poof_small", "honey_splash", "disease_puff", "蜂群烟", "蜂群出巢烟", "蜂蜜飞溅", "蜂王护卫变身烟", "病害特效", "病害蝇群烟", "蝇群烟"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/beeguard.lua", "prefabs/books.lua", "prefabs/beequeenhive.lua", "stategraphs/SGbeequeen.lua", "prefabs/grass.lua", "components/diseaseable.lua", "prefabs/grassgekko.lua", "prefabs/honey_trail.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 蜂蜜飞溅、蜂群烟与病害蝇群烟（bee_poof 族）

## 快速判断

- 职责与适用：`fx.lua` 通用表中相邻的三个表现用途共 4 注册——`honey_splash` 蜂蜜受击飞溅（巨大蜂窝被锤击）、`bee_poof_big`／`bee_poof_small` 蜂群烟（蜂王护卫聚焦变阵／还原与图书召蜂）、`disease_puff` 蝇群腾起烟（注册名与相邻注释块指向病害用途，当前快照无生成点）。全部纯表现、无组件；代理与本地实体分工见 [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂·关键流程]]。[条目组](../../../raw/current/scripts/fx.lua#L1485-L1541)
- 优先使用：做"结构受击掉蜜／碎屑飞溅"参考 honey_splash 及其调用方；做"蜂类出巢／变阵烟"参考 bee_poof 两名的选名方向（变阵 big／还原 small）；接入世界病害表现应使用 `diseaseflies`（[[reference/components/diseaseable]] 的实际特效），不是 disease_puff。
- 重要副作用：无玩法作用。bee_poof 两名自带 `dontstarve/common/deathpoof` 声，big 的 fn 另播 `bee_queen/beeguard/puff`（音量 0.6）；honey_splash 无声（受击声在调用方）；disease_puff 自带 `flies_appear` 声。[bee_poof_big 条目](../../../raw/current/scripts/fx.lua#L1513-L1523)、[honey_splash 条目](../../../raw/current/scripts/fx.lua#L1534-L1541)
- 修改边界：三处生成点全部在服务端模拟（状态图时间线、workable 回调、图书 fn），落点式不父挂；表现按通用工厂协议各端分派，专服只有代理。disease_puff 无原版生成先例，借用动画时长未知。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `disease_puff` | 通用表条目 → 加载器代理＋一帧后本地实体 | bank/build=`flies`，anim=`flies_puff`，声 `dontstarve/common/flies_appear`；无缩放、无 fn，默认四向。当前快照全 scripts 根无生成点。[条目](../../../raw/current/scripts/fx.lua#L1485-L1490) |
| `bee_poof_big` | 同上协议 | bank/build=`bee_poof`，anim=`anim`，声 `deathpoof`；`transform` 三轴 1.4 缩放；fn=`SetFinalOffset(1)`＋播 `bee_queen/beeguard/puff`（音量 0.6）。[条目](../../../raw/current/scripts/fx.lua#L1513-L1523) |
| `bee_poof_small` | 同上协议 | 同 bank/build/anim 同缩放，带 `deathpoof` 声；fn=FinalOffset1（仅抬层），没有 big 的额外 puff 声。[条目](../../../raw/current/scripts/fx.lua#L1525-L1532) |
| `honey_splash` | 同上协议 | bank/build=`honey_splash`，anim=`anim`，`nofaced`（不设朝向组件）；1.4 缩放；fn=FinalOffset1；无声。[条目](../../../raw/current/scripts/fx.lua#L1534-L1541)、[FinalOffset1](../../../raw/current/scripts/fx.lua#L1-L3) |

- [L1491-L1511](../../../raw/current/scripts/fx.lua#L1491-L1511) 的 `disease_fx_small`／`disease_fx`／`disease_fx_tall` 三条目整块处于 Lua 块注释中，未注册、不可生成；同段仅 `disease_puff` 生效。
- 简中 `languages/chinese_s.po` 定向检索无这 4 个注册名的显示名键；标题中文为已核验效果描述，非官方译名。宿主名核对：BEEQUEEN=蜂王、BEEGUARD=嗡嗡蜜蜂（Grumble Bee）、BOOK_BEES=养蜂笔记（Apicultural Notes）。[BEEGUARD](../../../raw/current/scripts/languages/chinese_s.po#L339314-L339317)、[BOOK_BEES](../../../raw/current/scripts/languages/chinese_s.po#L339849-L339852)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 4 条目同协议：服务端代理＋各端一帧后本地动画实体，animover 自删、代理 1 秒回收；本页差异仅资源、缩放、朝向与 fn 声音 | 全组落点式（生成者仅 `SetPosition`，不父挂） | [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂·关键流程]] |
| `bee_poof_big`／`small` 生成点：蜂王护卫聚焦链 | 蜂王 SG `focustarget` 状态按小队给嗡嗡蜜蜂分目标并调其 `FocusTarget(v)`（[L646](../../../raw/current/scripts/stategraphs/SGbeequeen.lua#L646-L653)）；护卫获得目标且未处于疾驰形态时换 `bee_guard_puffy_build` 并落 `bee_poof_big`（[L363](../../../raw/current/scripts/prefabs/beeguard.lua#L363)）；`FocusTarget(nil)` 且处于疾驰形态时还原 build 并落 `bee_poof_small`（[L376](../../../raw/current/scripts/prefabs/beeguard.lua#L376)） | 服务端状态图回调；清焦在 `focustarget_pst`／`flyaway`／`death` 收尾（[L784](../../../raw/current/scripts/stategraphs/SGbeequeen.lua#L784-L784)、[L307](../../../raw/current/scripts/stategraphs/SGbeequeen.lua#L307-L307)、[L410](../../../raw/current/scripts/stategraphs/SGbeequeen.lua#L410-L410)）与失去指挥（[OnLostCommander](../../../raw/current/scripts/prefabs/beeguard.lua#L428-L437)）；`FocusTarget` 导出与判据见 [L348-L378](../../../raw/current/scripts/prefabs/beeguard.lua#L348-L378)、[L582](../../../raw/current/scripts/prefabs/beeguard.lua#L582-L582) | 指挥链分工见 [[reference/components/commander]] |
| `bee_poof_big` 生成点：图书召蜂 | 养蜂笔记（book_bees）fn 按数量逐只延迟生成嗡嗡蜜蜂，每只落点同时 SpawnPrefab `bee_poof_big`；超上限返回 TOOMANYBEES 不生成 | 服务端；落点式于各蜂位置 | [deps](../../../raw/current/scripts/prefabs/books.lua#L884-L886)、[fn L916](../../../raw/current/scripts/prefabs/books.lua#L916-L916)、[book_bees 注册](../../../raw/current/scripts/prefabs/books.lua#L877-L877) |
| `honey_splash` 生成点：蜂窝受击 | 蜂王巨型蜂窝（beequeenhivegrown）workable 的 `OnWorked` 每次受击：`hive_hit` 声＋`large_hit` 动画＋honey_splash 落于蜂巢位置；随后才掷掉落与蜂王出现运气 | 服务端 workable 回调；每次受击必出，与运气判定无关 | [deps](../../../raw/current/scripts/prefabs/beequeenhive.lua#L13-L19)、[OnWorked L186](../../../raw/current/scripts/prefabs/beequeenhive.lua#L173-L210) |
| `disease_puff` | 无生成点；仅 `grass` 的 prefabs 依赖声明保留 | 缺口见陷阱 | [声明](../../../raw/current/scripts/prefabs/grass.lua#L17-L26) |
| 病害系统实际特效 | diseaseable 发病后 `DoFX` 生成 `diseaseflies` 父挂宿主并循环，与 disease_puff 无关 | 服务端组件任务 | [[reference/components/diseaseable]]、[[reference/prefabs/diseaseflies|病害蝇群表现]]、[DoFX](../../../raw/current/scripts/components/diseaseable.lua#L117-L138) |
| 资源 | 加载器按 build 声明 `anim/bee_poof.zip`、`anim/honey_splash.zip`、`anim/flies.zip`；音效字符串为条目数据（deathpoof、bee_queen/beeguard/puff、flies_appear） | 资源实际存在未单独核验，属引擎数据 | [加载器资产声明](../../../raw/current/scripts/prefabs/fx.lua#L5-L16) |

## 典型接入

### 借用蜂群烟做自定义蜂类表现

- 前提与选择：服务端模拟回调内（生成自定义蜂类、蜂巢受击等）直接 SpawnPrefab；选名按状态方向而非视觉大小——变阵／出巢用 big（多一层护卫 puff 声），还原／散场用 small。
- 生效与撤销：落点式一次性，工厂延迟创建本地表现、animover 自删；FX 代理独立于生成者，生成者同帧移除本身不会取消表现。若 FX 代理在延迟任务前被移除，则不能保证本地表现创建。[工厂](../../../raw/current/scripts/prefabs/fx.lua#L121-L135)
- 原版依据：图书逐蜂落点 [books.lua L916](../../../raw/current/scripts/prefabs/books.lua#L916-L916)；护卫变阵 [beeguard.lua L363](../../../raw/current/scripts/prefabs/beeguard.lua#L363-L363)。

```lua
local function OnSpawnBee(inst, x, y, z)   -- 服务端回调内
    local bee = SpawnPrefab("beeguard")
    bee.Transform:SetPosition(x, y, z)
    SpawnPrefab("bee_poof_big").Transform:SetPosition(x, y, z)
end
```

- 调整与验证：位置为世界坐标（y 取 0 即地面）；small 仍带 `deathpoof` 声，需要静音时应自行提供无声表现；不要在客户端回调生成（工厂代理需服务端权威）。

## 常见陷阱与限制

- **误区：disease_puff 是病害传播的表现**：本快照病害的实际表现是 `diseaseflies`（diseaseable `DoFX` 父挂循环），且全 scripts 根没有任何原版实体 `AddComponent("diseaseable")`——病害系统处于休眠（grassgekko 资源注释注明保留给 mod）。disease_puff 自身无生成点。[DoFX](../../../raw/current/scripts/components/diseaseable.lua#L117-L138)、[资源注释](../../../raw/current/scripts/prefabs/grassgekko.lua#L6-L6)
- **边界：disease_puff 无生成点的检索范围**：全 scripts 根（含 prefabs／components／stategraphs／brains／map）literal 检索 `disease_puff` 仅命中 fx.lua 注册与 grass.lua 依赖声明；`SpawnPrefab` 动态拼名清扫（poof／splash／disease／honey 前缀拼接）无产生该名的路径。未检索到不等于其他快照或 DLC 不存在。
- **边界：bee_poof_big／small 的差异是声音不是外形**：同 bank/build/anim、同 1.4 缩放，仅 big 的 fn 多播一声；两名的选名只随变阵方向（变阵 big／还原 small），不按尺寸档。
- **边界：FocusTarget 重聚焦不重复出烟**：两分支都先判当前移速档，已处于目标形态时不再生成；烟只出现在实际换 build 的那一刻。[判据](../../../raw/current/scripts/prefabs/beeguard.lua#L348-L378)
- **对比：honey_splash 与 honey_trail**：honey_trail 是蜂王蜜迹（专属 prefab、带减速玩法；Wiki 待建，源码 [prefabs/honey_trail.lua](../../../raw/current/scripts/prefabs/honey_trail.lua#L132-L142)）；honey_splash 只是受击飞溅，无玩法。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。4 条目逐字核验（含 L1491-L1511 注释块确认未注册）、FinalOffset1 helper；三生成点全函数核验：beeguard `FocusTarget`／`OnLostCommander`／导出、SGbeequeen `focustarget`／`focustarget_pst`／`flyaway`／`death` 四状态调用点、books book_bees fn、beequeenhivegrown `OnWorked`；diseaseable 全文、grass.lua 依赖段；全 scripts 根 literal 检索 4 注册名与动态拼名清扫；chinese_s.po 定向核验 4 FX 名无显示名键并核对宿主 BEEQUEEN／BEEGUARD／BOOK_BEES 键。未核验：各动画时长与循环性、音效资源在游戏数据中的实际存在、SGbeequeen 各状态的上游进入条件全链、书自身读限。
- 游戏运行：未运行；烟团／飞溅视觉与声音未实测。
- 结构检查：2026-11-01 `validate_page.py`（本页 0 问题，81 行／11 KiB 内，standard 档；未登记叶级索引属预期，另有同状态既有页）与 `validate_source_links.py --strict`（0 错误 0 警告）通过；Lua 示例经 parser_probe `--syntax-only` 可解析。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/prefabs/index|Prefab 索引]]。
