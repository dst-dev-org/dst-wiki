---
title: "建筑倒塌特效（collapse_big）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "可摧毁建筑／设施破坏时的标准倒塌表现：联网代理壳＋各端本地动画实体（collapse bank）＋烟声与按材质破碎声，SetMaterial 选声；同文件注册 collapse_small（小倒塌），脚本树内 200+ 处 SpawnPrefab。"
aliases: ["collapse_big", "collapse_small", "structure_collapse_fx", "建筑倒塌", "倒塌特效", "倒塌烟尘"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/structure_collapse_fx.lua", "prefabs/pigtorch.lua", "prefabs/coldfire.lua", "prefabs/altar_prototyper.lua", "prefabs/trophyscale_oversizedveggies.lua", "fx.lua", "prefablist.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 建筑倒塌特效（collapse_big）

## 快速判断

- 职责与适用：可摧毁建筑／设施破坏时的**标准倒塌表现**：联网代理壳＋各端本地动画实体（bank `collapse`／build `structure_collapse_fx`）＋烟声与按材质的破碎声；两个注册名——`collapse_big`（anim `collapse_large` 大倒塌）与 `collapse_small`（anim `collapse_small` 小倒塌）。脚本树内 SpawnPrefab 约 240 处（grep 计数：small 204／big 37）。[工厂](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L60-L92)、[注册](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L94-L95)
- 优先使用：查"建筑被锤毁／消失时的烟尘倒塌"选本页；"DropLoot＋SpawnPrefab＋SetMaterial"三件套见典型接入。
- 重要副作用：`SetMaterial(材质名)` 选破碎声（wood/metal/rock/stone/straw/pot，rock 用专用 rock_break），另叠加 destroy_smoke；`"none"` 完全无声；未设置只有烟声。[材质表](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L6-L22)、[播放](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L44-L51)
- 修改边界：动画实体只在非专服生成（专服仅代理壳，1 秒后自清）；实体 persists=false 不入存档。[dedicated 跳过](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L70-L75)、[移除](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L87-L88)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `collapse_big` | makefn("collapse_large")：Transform＋Network＋NOCLICK/FX → net_tinybyte material（"collapse_fx.material"）→ 非专服延迟 1 帧 playfx → SetPristine → 主模拟暴露 SetMaterial＋persists=false＋1 秒自清 | playfx 建非联网实体：SetFromProxy 定位、播 anim＋FinalOffset 3、按 material 播声、animover 自删。[工厂](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L60-L92)、[playfx](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L24-L54) |
| `collapse_small` | 同工厂，anim `collapse_small` | 小型设施／原木堆等用。[注册](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L94-L95) |

两注册名均无 NAMES 条目；标题中文为已核验效果描述，非官方译名。注意 prefablist 登记的 "structure_collapse_fx"（[L1245](../../../raw/current/scripts/prefablist.lua#L1245)）与文件名都不是实际注册名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 代理-本地实体模式 | 网络化壳只携带 material netvar；可见动画与声音是 playfx 延迟 1 帧创建的**非联网**实体（SetCanSleep(false)），专服不建 | 客户端与非专服主机各自建本地动画实体，专服不建；注释明言 dedicated 无需本地 fx | [makefn](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L60-L92)、[playfx](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L24-L54) |
| 材质系统 | SetMaterial(字符串) 经 `MATERIALS` 反查表写数字索引；播放端查回名字选声：rock→`rock_break`，其余→`destroy_<material>`，统一叠加 `destroy_smoke`；索引 0（未设置／未知名）→只有烟声；`"none"`→完全无声 | 6 种材质＋none；白名单外的字符串落回索引 0 | [表](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L6-L16)、[声表](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L18-L22)、[SetMaterial](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L56-L58) |
| 消费者模式 | onhammered 典型三件套：DropLoot→SpawnPrefab→SetMaterial→Remove（pigtorch wood）；烧毁／自然消失可不设材质（coldfire）；大型设施用 collapse_big＋stone（altar_prototyper） | 材质按被毁物选；不设即只有烟声 | [pigtorch](../../../raw/current/scripts/prefabs/pigtorch.lua#L29-L35)、[coldfire](../../../raw/current/scripts/prefabs/coldfire.lua#L15-L20)、[altar 大](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L480-L482)、[inline 用法](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L415)、[wood 例](../../../raw/current/scripts/prefabs/trophyscale_oversizedveggies.lua#L247-L250) |
| 同名资源复用 | fx.lua 通用表现表的 `waterplant_destroy` 条目直接用同 bank/build/anim（collapse_small），不经本 prefab 生成 | 属 fx.lua MakeFx 家族，行为独立 | [fx.lua 条目](../../../raw/current/scripts/fx.lua#L2261-L2267) |

## 典型接入

### 复用"建筑破坏倒塌"

- 前提与选择：主模拟 SpawnPrefab 后摆位即可；需要材质破碎声时再调 SetMaterial。
- 生效与撤销：动画实体延迟 1 帧出现（等定位稳定），代理壳 1 秒后自清；动画实体 animover 自删。
- 原版依据：pigtorch 的 onhammered 即最小完整用法（[实现](../../../raw/current/scripts/prefabs/pigtorch.lua#L29-L35)）。

```lua
local function onhammered(inst, worker)
    inst.components.lootdropper:DropLoot()
    local fx = SpawnPrefab("collapse_small")
    fx.Transform:SetPosition(inst.Transform:GetWorldPosition())
    fx:SetMaterial("wood")  -- 不设则只有烟声；"none" 完全无声
    inst:Remove()
end
```

- 调整与验证：材质白名单见 MATERIAL_NAMES（6 种＋none，[定义](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L6-L15)）；大型设施用 collapse_big。

## 常见陷阱与限制

- **边界：未 SetMaterial 不是没声**：material 索引 0 时仍播 destroy_smoke，只是没有材质破碎声；要完全静音用 `"none"`（连烟声也不播）。[播放分支](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L44-L51)
- **误区：SetMaterial 传数字**：接口收字符串材质名，反查失败即落回 0（只有烟声）；不要传索引。[SetMaterial](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L56-L58)
- **边界："structure_collapse_fx" 不是注册名**：文件名与 prefablist 登记名都是家族统称，`SpawnPrefab("structure_collapse_fx")` 会失败；实际注册名只有 collapse_big／collapse_small（[注册行](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L94-L95)）。
- **边界：专服无动画**：dedicated 不建 playfx 实体（注释明言的设计），不是缺资源；表现由各客户端自行生成。[dedicated 跳过](../../../raw/current/scripts/prefabs/structure_collapse_fx.lua#L70-L75)

## 验证记录

- 源码审阅：2026-10-30，build 26230496；96 行工厂全文（两工厂、playfx、材质系统、端分界）、pigtorch／coldfire／altar_prototyper／trophyscale 消费点、fx.lua 同资源条目、prefablist 登记名、NAMES 检索（无命中）、SpawnPrefab 计数（grep）已核验；anim zip 内容、240 处消费者仅代表性抽读、晚加入时序未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）；本批依据见[[updates/build-26335394/index#wiki-沿用记录]]（批次-1）。

返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]、[[reference/prefabs/index|Prefab 索引]]。
