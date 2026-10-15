---
title: "scenariorunner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "在实体上挂载 scenarios/ 目录脚本并按 OnCreate/OnLoad/OnDestroy 约定执行，用于宝箱陷阱等一次性场景逻辑。"
aliases: ["场景脚本执行器", "剧情脚本", "scenariorunner"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/scenariorunner.lua", "prefabs/chest_mimic.lua", "gamelogic.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# scenariorunner

## 快速判断

- 职责与适用：把 `scenarios/<name>.lua` 的脚本绑到宿主上运行；OnCreate 只跑一次并随存档记忆，OnLoad 每次世界载入都跑，适合陷阱宝箱、惊喜礼物的首开逻辑。
- 优先使用：宝箱类宿主用 SetScript + Run，如 mimic 化宝箱（[chest_mimic](../../../raw/current/scripts/prefabs/chest_mimic.lua#L162-L164)）。
- 重要副作用：OnLoad 缺失的脚本在 Run 时被 `ClearScenario` 直接移除组件（[Run](../../../raw/current/scripts/components/scenariorunner.lua#L41-L45)）；ClearScenario/Reset 会调脚本的 `OnDestroy`。
- 修改边界：脚本文件必须能被 `require("scenarios/<name>")` 解析；存档只记 `scriptname` 与 `hasrunonce`，脚本内自建状态需脚本自己保存。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetScript(name)` | `name: string` | require `scenarios/<name>` 并断言导出 `OnCreate` 或 `OnLoad`；已有脚本时打印警告并覆盖。[实现](../../../raw/current/scripts/components/scenariorunner.lua#L26-L33) |
| `Run()` | 无参数 | 首次运行 `OnCreate(inst, self)` 并置 hasrunonce；随后每次运行 `OnLoad(inst, self)`，脚本无 OnLoad 时 ClearScenario。[实现](../../../raw/current/scripts/components/scenariorunner.lua#L35-L46) |
| `ClearScenario()` | 无参数 | `RemoveComponent("scenariorunner")` 并调 `OnDestroy(inst)`。[实现](../../../raw/current/scripts/components/scenariorunner.lua#L48-L53) |
| `Reset()` | 无参数 | 调 OnDestroy 后清 script/scriptname/hasrunonce，但不移除组件，可重新 SetScript。[实现](../../../raw/current/scripts/components/scenariorunner.lua#L55-L62) |
| `OnSave`／`OnLoad(data)` | 存读钩子 | 保存 `scriptname` 与 `hasrunonce`；载入后重挂脚本并恢复“已运行过”标记。[实现](../../../raw/current/scripts/components/scenariorunner.lua#L8-L24) |

## 典型接入

### 给容器挂一次性开箱陷阱

- 前提与选择：在宿主 master_postinit（或需要触发的时机）安装组件；脚本放模组 `scenarios/<name>.lua`，导出约定回调。
- 生效与撤销：Run 触发 OnCreate→OnLoad；陷阱触发后脚本内调 `scenariorunner:ClearScenario()` 自行摘除（原版 trap 惯例，见 [chestfunctions](../../../raw/current/scripts/scenarios/chestfunctions.lua#L58)）。
- 原版依据：宝箱装脚本（[treasurechest](../../../raw/current/scripts/prefabs/treasurechest.lua#L461-L463)）；世界载入时 gamelogic 统一对全部实体 Run（[gamelogic](../../../raw/current/scripts/gamelogic.lua#L633-L634)）。

```lua
-- scenarios/my_trap.lua
local function OnCreate(inst, scenariorunner)
    inst.mytrap_armed = true
end

local function OnLoad(inst, scenariorunner)
    if inst.mytrap_armed and inst.components.container ~= nil then
        inst:ListenForEvent("itemget", function()
            scenariorunner:ClearScenario()
        end)
    end
end

return { OnCreate = OnCreate, OnLoad = OnLoad }
```

```lua
-- 宿主 prefab 内（仿 wintersurprisespawner.lua:158-160）
local sr = inst:AddComponent("scenariorunner")
sr:SetScript("my_trap")
sr:Run()
```

- 调整与验证：脚本导出表可用 `OnCreate`／`OnLoad`／`OnDestroy` 三个键；OnCreate 只在 hasrunonce 为假时执行。

## 常见陷阱与限制

- 玩家存档重载时 mainfunctions 会按 `saved.data.scenariorunner` 重装脚本再 SetScript（[mainfunctions](../../../raw/current/scripts/mainfunctions.lua#L495-L500)），因此脚本文件必须常驻，改名或删除会让旧存档 require 失败。
- 首个示例只演示导出结构与"触发后摘除"惯例：itemget 触发条件请按需替换（如特定物品、开箱事件），脚本状态 `inst.mytrap_armed` 不入存档，跨读档的持久数据需自行用宿主 OnSave 承载。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 chest_mimic／treasurechest／gamelogic／mainfunctions 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
