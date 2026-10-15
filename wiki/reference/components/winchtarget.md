---
title: "winchtarget"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "给沉没在水下的实体登记打捞回调与起吊深度，让绞车、漩涡等消费者能取回其中的物品。"
aliases: ["沉没物打捞目标", "绞车打捞", "winchtarget"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/winchtarget.lua", "prefabs/winch.lua", "prefabs/oceanfishableflotsam.lua", "prefabs/underwater_salvageable.lua", "components/oceanwhirlportalphysics.lua", "components/wagpunk_arena_manager.lua", "prefabs/wagpunk_floor_kit.lua", "components/retrofitforestmap_anr.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# winchtarget

## 快速判断

- 职责与适用：打捞目标的登记组件：宿主带 `winchtarget` tag 供实体搜索，`Salvage()` 负责取回沉没物品并返回打捞结果；只管登记与回调，不做搜索、起吊动画或物理。原版宿主为 `oceanfishableflotsam` 与 `underwater_salvageable`，均在服务端分支 AddComponent。[宿主1](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L273-L275)、[宿主2](../../../raw/current/scripts/prefabs/underwater_salvageable.lua#L61-L65)
- 优先使用：想让沉没物可被打捞，AddComponent 后 `SetSalvageFn`（返回打捞出的物品实体），需要指定起吊深度时设 `depth`；消费者侧用 `winchtarget` tag 搜索后调 `Salvage()`。[回调入口](../../../raw/current/scripts/components/winchtarget.lua#L15-L17)
- 重要副作用：`Salvage()` 先清掉宿主内物品的 `submersible.force_no_repositioning` 再调回调；构造时加 tag、`OnRemoveFromEntity` 时移除 tag。[Salvage](../../../raw/current/scripts/components/winchtarget.lua#L19-L26)、[tag 维护](../../../raw/current/scripts/components/winchtarget.lua#L8-L13)
- 修改边界：纯服务端组件；无 OnSave／OnLoad，回调函数不入存档，读档后由宿主 prefab 重新设置；组件本身不搜索、不移动物品。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `depth` | number，默认 -1 | -1 表示由消费者用打捞点海深代替；具体数值语义由消费者解读，`oceanfishableflotsam` 设 2。[默认值](../../../raw/current/scripts/components/winchtarget.lua#L4)、[消费读取](../../../raw/current/scripts/prefabs/winch.lua#L172-L175) |
| `SetSalvageFn(fn)` | `fn(inst)` → 物品实体或 nil | 记录打捞回调，`Salvage()` 内调用；不入存档。[L15-L17](../../../raw/current/scripts/components/winchtarget.lua#L15-L17)、[调用点](../../../raw/current/scripts/components/winchtarget.lua#L25) |
| `Salvage()` | 回调结果或 nil | 先对 `GetSunkenObject()` 的 `submersible.force_no_repositioning` 置 false，再调回调；无回调返回 nil。[L19-L26](../../../raw/current/scripts/components/winchtarget.lua#L19-L26) |
| `GetSunkenObject()` | 实体或 nil | 返回宿主 `inventory` 第 1 格物品；宿主无 inventory 返回 nil。[L28-L34](../../../raw/current/scripts/components/winchtarget.lua#L28-L34) |

## 常见陷阱与限制

- 打捞结果完全由回调决定：组件只透传 `salvagefn` 返回值，不会自己丢出宿主物品。原版两种模式：`underwater_salvageable` 用 1 格 inventory 存沉没物（[inventory 设置](../../../raw/current/scripts/prefabs/underwater_salvageable.lua#L69-L71)），`oceanfishableflotsam` 的回调自行生成结果实体。
- 多个消费者按 tag 搜索后直接调 `Salvage()`：绞车 [winch.lua#L127](../../../raw/current/scripts/prefabs/winch.lua#L127)、[L136](../../../raw/current/scripts/prefabs/winch.lua#L136)、[L175](../../../raw/current/scripts/prefabs/winch.lua#L175)；漩涡 [oceanwhirlportalphysics.lua#L185](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L185)、[L197](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L197)；wagpunk 场地清理 [wagpunk_arena_manager.lua#L1946](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1946)、[L1956](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1956) 与 [wagpunk_floor_kit.lua#L56-L66](../../../raw/current/scripts/prefabs/wagpunk_floor_kit.lua#L56-L66)。自定义宿主复用该 tag 就会被这些系统一并打捞／清理。
- `retrofitforestmap_anr` 在旧存档适配时用 `GetSunkenObject()` 检查沉没物（[L523-L524](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L523-L524)）。

### 最小接入

```lua
AddPrefabPostInit("oceanfishableflotsam", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst.components.winchtarget:SetSalvageFn(function(host)
        local item = host.components.winchtarget:GetSunkenObject()
        return item
    end)
end)
```

- 调整与验证：回调返回的实体会被绞车交给玩家；`depth` 只影响起吊判定深度，不影响掉落物。本例沿用原版宿主，仅替换回调，未单独运行验证。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/winchtarget.lua（36 行），核对两个原版宿主的 AddComponent 与参数设置，及 winch、oceanwhirlportalphysics、wagpunk_arena_manager、wagpunk_floor_kit、retrofitforestmap_anr 五处消费者的调用方式。未覆盖起吊动画、音效与网路同步表现。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；61 行、5711 字节，simple 档内）；validate_source_links.py --strict 通过（21 处源码引用，0 error／0 warning）；lua 示例经 parser_probe.exe --syntax-only 解析通过。
