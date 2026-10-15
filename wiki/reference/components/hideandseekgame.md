---
title: "hideandseekgame"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "捉迷藏小游戏管理组件：登记藏匿点并把附近玩家吸为找方，按找到数计奖励、藏匿点清空即结算，狸猫巢穴小游戏的核心裁判。"
aliases: ["捉迷藏游戏", "捉迷藏管理", "hideandseekgame"]
tags: [component, callback, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/hideandseekgame.lua", "prefabs/kitcoonden.lua", "components/hideandseeker.lua", "components/hideandseekhidingspot.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hideandseekgame

## 快速判断

- 职责与适用：挂在"游戏场地"实体（原版为狸猫巢穴，prefabs/kitcoonden.lua:408）上，管理藏匿点登记、找方（seeker）吸纳与结算（components/hideandseekgame.lua:3-48）。与 `hideandseekhider`（藏方狸猫）、`hideandseekhidingspot`（藏匿点）、`hideandseeker`（找方玩家）三个组件协作。
- 优先使用：宿主调 `RegisterHidingSpot` 开局、`AddSeeker` 强制加入；进度查询用 `IsActive`／`GetNumHiding`／`GetNumFound`（lines 56-58、138-148）。
- 重要副作用：首个藏匿点登记即启动 1 秒脉冲，把范围内未参与的存活可见玩家自动加为找方（lines 97-110、121-123）；找方被给 `hideandseeker` 组件（lines 83-95）。
- 修改边界：`Abort`／`UnregisterHidingSpot` 解除全部监听并结束脉冲（lines 60-65、126-136）；`OnRemoveEntity` 中止所有藏匿点（lines 50-54）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `hiding_range`／`hiding_range_toofar` | 默认 10／10 | 吸纳玩家与找方跑太远的判定半径；原版调大并配跑离公告（kitcoonden.lua:412-416；hideandseeker.lua:19-33） |
| `RegisterHidingSpot(spot)` | 藏匿点实体 | 重置找到计数、监听 `onremove`／`onhidingspotremoved`、启动脉冲（lines 112-124） |
| `onhidingspotremoved` 数据 | `{finder}` | 由 hideandseekhidingspot 组件推出（hideandseekhidingspot.lua:43）；finder 是玩家时计数 +1 并调 `OnHidingSpotFound`（lines 35-47） |
| `AddSeeker(seeker, started_game)` | 玩家实体 | 给玩家挂 `hideandseeker` 并回绑游戏；重复加入有判重（lines 83-95） |
| `IsActive()` | `-> bool` | 还有藏匿点未找到（lines 56-58） |
| `GetNumHiding`／`GetNumSeekers`／`GetNumFound` | `-> number` | 场上统计，结算奖励用（lines 138-148；kitcoonden.lua:226-255） |
| `Abort()` | 手动终止 | 逐个注销并中止藏匿点组件（lines 60-65） |
| `OnSave`／`LoadPostPass` | 存藏匿点 GUID 引用与计数 | 载入后重登记（lines 150-174） |
| 宿主回调 | `OnAddSeeker`／`OnHidingSpotFound`／`OnHideAndSeekOver`／`OnHideAndSeekPulse` | 全部由宿主赋值（kitcoonden.lua:409-411）；结算时清找方与计数（lines 67-81） |
| 公告字段 | `seeker_too_far_announce` 等字符串 | 由宿主填、`hideandseeker` 消费（kitcoonden.lua:414-416） |

## 典型接入

### 自建捉迷藏场地（服务端 master postinit）

- 前提与选择：藏匿点必须是带 `hideandseekhidingspot` 组件的实体，否则 `OnRemoveEntity` 的中止与 `onhidingspotremoved` 链路不成立（lines 50-54；hideandseekhidingspot.lua:43）。
- 生效与撤销：登记第一个藏匿点即开局；最后一个藏匿点被找到或移除时自动 `_HideAndSeekOver` 结算并停脉冲（lines 21-29、126-136）。
- 原版依据：kitcoonden.lua:408-416 完整示范回调与参数配置；开局的藏匿点筛选与分配见 kitcoonden.lua:154-190。

```lua
local function OnHidingSpotFound(inst, finder, hiding_spot)
    if finder.components.talker ~= nil then
        finder.components.talker:Say("Found one!")
    end
end

-- 场地实体 master postinit：
inst:AddComponent("hideandseekgame")
inst.components.hideandseekgame.hiding_range = 12
inst.components.hideandseekgame.OnHidingSpotFound = OnHidingSpotFound
inst.components.hideandseekgame.OnHideAndSeekOver = function(inst)
    inst:PushEvent("my_hideandseek_over")
end
-- 开局：inst.components.hideandseekgame:RegisterHidingSpot(spot_ent)
```

- 调整与验证：`hiding_range` 单位同世界距离；脉冲每秒全玩家扫一次（line 122），玩家数量大时注意开销。

## 常见陷阱与限制

- **藏匿点被非玩家移除也算推进：** `onhidingspotremoved` 无 finder 时静默注销（lines 44-46），藏匿点被烧毁等同样会推向结算，需要注意藏匿点的保护。
- **找方身份靠组件判重：** `AddSeeker` 以 `hideandseeker` 组件存在与否去重（lines 84-86）；玩家中途被移除该组件后会再次被脉冲吸回。
- **计数在登记时清零：** 每次 `RegisterHidingSpot` 都重置 `num_hiders_found`（line 113），中途加藏匿点会丢已找到数。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。通读 components/hideandseekgame.lua 全文（180 行）；核验 kitcoonden.lua:154-190、226-255、408-416，hideandseeker.lua:5-33，hideandseekhidingspot.lua:43。未运行游戏。
- 结构检查：validate_page.py / validate_source_links.py 通过（见批级自验）。

相关：[[reference/components/kitcoonden]]、[[reference/components/hideandseeker]]、[[reference/components/hideandseekhidingspot]]
