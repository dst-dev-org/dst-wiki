---
title: "皮弗娄牛驯化变身特效（beefalo_transform_fx）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "单个通用表注册在皮弗娄牛 become_domesticated 状态播放变身表现：完成驯化与已驯化后野性化共用该状态；状态内 8 帧生成于牛体，骑乘状态下状态退出时再在骑手侧补一枚；条目声效为幽灵 haunt 且源码带 TODO 注释。"
aliases: ["beefalo_transform_fx", "皮弗娄牛变身特效", "驯化变身特效", "牛变身特效"]
tags: [prefab, fx]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "stategraphs/SGBeefalo.lua", "prefabs/beefalo.lua", "components/domesticatable.lua", "components/rider.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 皮弗娄牛驯化变身特效（beefalo_transform_fx）

## 快速判断

- 职责与适用：`fx.lua` 通用表单注册 `beefalo_transform_fx`，只服务皮弗娄牛 `become_domesticated` 状态的变身表现——完成驯化，以及已驯化后饥饿且驯化归零、仍判已驯化时进入的同一状态；与调谐／皮弗娄牛其他流程无关。工厂协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L1132-L1139)
- 优先使用：查"牛驯化变身表现"到本页；驯化数值与流程读 [[reference/components/domesticatable|domesticatable]]。
- 重要副作用：经 `SpawnChild` 生成于牛体（子实体随牛）；条目自带 `SetFinalOffset(1)` 与幽灵 haunt 声效，变身吼声 `hairgrow_pop` 由 SG 另播。[状态](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L722-L765)
- 修改边界：生成点在服务端 SG 状态内；骑乘时牛体已被 `AddChild` 给骑手并 `RemoveFromScene`，条目本地表现的实际可见性随挂接位置变化，动画素材内容未核验。[骑乘挂接](../../../raw/current/scripts/components/rider.lua#L149-L152)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `beefalo_transform_fx` | 通用表条目，fn=`FinalOffset1`（本地表现 `SetFinalOffset(1)`） | bank／build=`beefalo_fx`，anim=`transform`，sound=`dontstarve/ghost/ghost_haunt`，声效上一行有源码注释 `--#TODO: this one`。[条目](../../../raw/current/scripts/fx.lua#L1132-L1139) |

注册名无简中 `.po` 显示名称键（2026-10-31 定向检索无命中）；标题中"皮弗娄牛"为宿主官方名 [po:339265-339267](../../../raw/current/scripts/languages/chinese_s.po#L339265-L339267)，"驯化变身"为已核验效果描述，非官方特效译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 代理＋一帧本地表现，动画结束自删；`SpawnChild` 即代理父挂牛体 | 服务端 SG 生成，各非专服本地呈现 | [[reference/mechanisms/generic-fx-factory]] |
| 生成者：SGBeefalo `become_domesticated` | 状态 onenter 播 `transform` 动画与 `hairgrow_pop` 声；8 帧时刻 `inst:SpawnChild("beefalo_transform_fx")`；animover 转出，onexit 若 `GetParent() ~= nil`（骑乘中）再 `parent:SpawnChild` 一枚 | 服务端；onexit 补枚挂在骑手（玩家）侧 | [状态](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L722-L765)、[8 帧生成](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L734-L736)、[onexit 补枚](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L750-L755) |
| 触发窗口：`domesticationPending` | [[reference/components/domesticatable|domesticatable]] 推 `domesticated`／`goneferal` 事件置位 `domesticationPending`；SG 在 `idle` onenter 或 `actual_alert` ontimeout 检到置位即进变身状态 | 服务端；`goneferal` 在清除驯化前推送，仍判已驯化时置位 | [置位](../../../raw/current/scripts/prefabs/beefalo.lua#L530-L532)、[野性化分支](../../../raw/current/scripts/prefabs/beefalo.lua#L542-L547)、[idle 入口](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L130-L131)、[actual_alert 入口](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L591-L595) |
| 资源 | bank／build=`beefalo_fx`（加载器按 build 声明 `anim/beefalo_fx.zip`） | `yotb_confetti` 复用同 bank 同 anim（无 fn 无声），与驯化无关 | [复用条目](../../../raw/current/scripts/fx.lua#L2328-L2332) |

## 典型接入

### 自定义牛类生物复用变身表现

- 前提与选择：模组自定义生物想在状态关键帧沿用该表现时，在服务端 SG 状态内 `SpawnChild`，与原版同款写法。
- 生效与撤销：子实体随宿主移动，动画结束自删，无需清理。
- 原版依据：[SGBeefalo 8 帧生成](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L734-L736)。

```lua
-- 服务端 SG 状态 timeline 内（写法同原版 become_domesticated）：
local my_state = State{
    name = "my_transform",
    tags = {"busy"},
    timeline = {
        TimeEvent(8 * FRAMES, function(inst)
            inst:SpawnChild("beefalo_transform_fx")
        end),
    },
}
```

- 调整与验证：换注册名即换表现；该注册自带幽灵 haunt 声效，若宿主另有变身声需评估重叠。

## 常见陷阱与限制

- **边界：骑乘状态下变身可能出两枚**：8 帧枚挂在牛体；状态退出时若牛体已父接骑手（骑乘挂接）再在骑手侧补一枚，两者是不同实例而非重播。[生成点](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L734-L755)
- **误区：条目声效是牛叫声**：声效为幽灵 haunt（`dontstarve/ghost/ghost_haunt`），且源码注释 `--#TODO: this one` 表明待定；牛侧吼声 `hairgrow_pop` 由状态 onenter 另播。[条目](../../../raw/current/scripts/fx.lua#L1132-L1139)
- **边界：同 bank 同 anim 不等于同用途**：`yotb_confetti`（牛年活动纸屑）复用 `beefalo_fx` 的 `transform` 动画；按资源反查生成者时会多出这一无 fn 无声条目。[复用条目](../../../raw/current/scripts/fx.lua#L2328-L2332)
- **边界：无 SG 以外生成点**：`raw/current/scripts` 全域 `grep -rn`（含 `beefalo_transform` 前缀复核）仅命中 fx.lua 注册与 SGBeefalo 两处生成，幼牛 `grow_up` 等其他状态不走该注册。[检索即证据](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L722-L765)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。条目 bank／build／anim／sound／fn 与 TODO 注释逐项核对；`beefalo_transform_fx` 全域反查仅 SGBeefalo 两处生成；触发窗口读到 `domesticated`／`goneferal` 事件置位与 SG 两处入口，野性化事件在 `SetDomesticated(false)` 前推送，故仍能判已驯化并进同一状态；骑乘挂接核对 rider `AddChild`；`BEEFALO` 显示名称键与特效名无键在简中 `.po` 定向核验。未覆盖：动画素材内容与时长、驯化数值流程（[[reference/components/domesticatable|domesticatable]] 页范围）、`RemoveFromScene` 下子树渲染表现。
- 游戏运行：未运行；特效视觉未实测。

返回 [[reference/prefabs/fx-transition-index|变身、召唤、传送与退场]]、[[reference/prefabs/index|Prefab 索引]]。
