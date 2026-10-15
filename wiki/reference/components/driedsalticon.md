---
title: "driedsalticon"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "干货盐粒覆盖图标的服务端开关与双端同步：用 net_bool 驱动物品栏图像叠加并联动 inventoryitem 换图。"
aliases: ["盐粒图标", "干货盐覆盖", "driedsalticon"]
tags: [component, item, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/driedsalticon.lua", "prefabs/spoiledfood.lua", "prefabs/veggies.lua", "prefabs/meatrack2.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# driedsalticon

## 快速判断

- 职责与适用：给可干制食物一个"表面有盐"的服务端开关，双端在物品栏图像上叠加 `<prefab>.tex` 覆盖图；原版装在 spoiledfood 与咸干蔬菜（[安装](../../../raw/current/scripts/prefabs/spoiledfood.lua#L164-L165)、[veggies](../../../raw/current/scripts/prefabs/veggies.lua#L794-L795)）。
- 优先使用：晾架宿主按槽位状态调 ShowSaltIcon/HideSaltIcon，如珍珠多格晾架（[调用](../../../raw/current/scripts/prefabs/meatrack2.lua#L392-L402)）。
- 重要副作用：`showicon` 是 `net_bool`（事件 `showicondirty`），服务端 set 后客户端经 dirty 事件走默认图像逻辑（[netvar](../../../raw/current/scripts/components/driedsalticon.lua#L33)）。
- 修改边界：仅改物品栏呈现，不影响耐久/属性；`collects` 标记供晾架判盐归属，与图标显示互相独立。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `ShowSaltIcon()`／`HideSaltIcon()` | 仅服务端生效 | set 前判重；随后本地直接调 OnShowIconDirty 即时生效（客户端等 dirty 事件）。[实现](../../../raw/current/scripts/components/driedsalticon.lua#L57-L73) |
| `SetCollectsOnDried(collects)` | `collects: boolean`，仅服务端 | 标记"干成后该槽应收盐"；晾架 itemget 时读它决定 AddSalt（[消费](../../../raw/current/scripts/prefabs/meatrack2.lua#L397-L399)）。[实现](../../../raw/current/scripts/components/driedsalticon.lua#L50-L55) |
| `OverrideShowIconFn(fn)`／`OverrideHideIconFn(fn)` | `fn(inst)` | 覆盖默认叠加逻辑，show/hide 各一个。[实现](../../../raw/current/scripts/components/driedsalticon.lua#L42-L48) |
| `showicon` | `net_bool`，名 `driedsalticon.showicon`，dirty 事件 `showicondirty` | 双端状态源；客户端构造时注册监听。[构造](../../../raw/current/scripts/components/driedsalticon.lua#L27-L40) |
| `collects` | `boolean`（服务端） | 默认未显式赋值即假；仅服务端字段，不进存档。 |

## 典型接入

### 干货上叠盐图标并联动收盐

- 前提与选择：物品的 inventory atlas 需含 `<prefab>.tex` 覆盖图，否则默认逻辑设置 `inv_image_bg` 找不到图；master 侧换图名 `salt_dried_overlay` 依赖 [[reference/components/inventoryitem]]。
- 生效与撤销：Show/Hide 即时改 `inv_image_bg` 并同步；物品进出背包由晾架侧监听 onputininventory/ondropped 清除（[清除](../../../raw/current/scripts/prefabs/meatrack2.lua#L389-L393)）。
- 原版依据： spoiledfood 关闭收盐、veggies 开启收盐（[spoiledfood](../../../raw/current/scripts/prefabs/spoiledfood.lua#L164-L165)、[veggies](../../../raw/current/scripts/prefabs/veggies.lua#L794-L795)）。

```lua
-- 食物 prefab 的 master_postinit 内（仿 spoiledfood）
inst:AddComponent("driedsalticon")
inst.components.driedsalticon:SetCollectsOnDried(true)

-- 晾架侧切换显示（服务端）
item.components.driedsalticon:ShowSaltIcon()
```

- 调整与验证：想换覆盖图或行为用 OverrideShowIconFn/OverrideHideIconFn；默认分支里客户端走 `PushEvent("imagechange")`、服务端走 `ChangeImageName`（[分支](../../../raw/current/scripts/components/driedsalticon.lua#L9-L13)）。

## 常见陷阱与限制

- `ismastersim` 在构造时固化（[实现](../../../raw/current/scripts/components/driedsalticon.lua#L29)）；Show/Hide/SetCollectsOnDried 在客户端调用一律静默返回，不要指望客户端主动改图标。
- 默认覆盖图文件名固定为 `<prefab>.tex`（[默认逻辑](../../../raw/current/scripts/components/driedsalticon.lua#L7-L8)）：prefab 名与贴图名不一致的物品必须走 OverrideShowIconFn，否则物品栏显示不出叠加。

## 相关链接

- [[reference/components/dryingracksaltcollector]]：多格晾架的收盐登记与图标联动。
- [[reference/components/inventoryitem]]：ChangeImageName 换图与 inv_image_bg 呈现。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 spoiledfood／veggies／meatrack2 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
