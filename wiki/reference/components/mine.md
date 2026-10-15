---
title: "mine"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "地雷式陷阱的搜索、阵营过滤、重置与触发状态；伤害、消耗和表现由宿主回调负责。"
aliases: ["犬牙陷阱触发范围改为二", "地雷式触发", "犬牙陷阱重置", "mine"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/mine.lua", "prefabs/trap_teeth.lua", "componentactions.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# mine

## 快速判断

- 职责与适用：主动搜索附近目标、记录 inactive／issprung 并通知宿主；犬牙陷阱用 onexplode 打伤害并扣次数。本组件不内置范围爆炸，捕捉小动物另见 [[reference/components/trap]]。[搜索](../../../raw/current/scripts/components/mine.lua#L11) [宿主效果](../../../raw/current/scripts/prefabs/trap_teeth.lua#L37-L45)
- 优先使用：调半径用 SetRadius；布置／重新就绪用 Reset，停用用 Deactivate；恢复“已触发”显示用 Spring，不能当作再次攻击。[重置](../../../raw/current/scripts/components/mine.lua#L139-L156) [停用与显示](../../../raw/current/scripts/components/mine.lua#L181-L196)
- 重要副作用：Explode 先停搜索、记 target／触发状态，再调 onexplode；直接调用不检阵营、半径、inactive 或已触发状态，也无成功返回。[实现](../../../raw/current/scripts/components/mine.lua#L203-L211)
- 修改边界：服务端；SetReusable 只维护动作标签，不拦 Reset。玩家重置候选与动作执行分开，不能据按钮不可见推断服务端禁止重置。[标签](../../../raw/current/scripts/components/mine.lua#L122) [候选](../../../raw/current/scripts/componentactions.lua#L618-L620) [动作](../../../raw/current/scripts/actions.lua#L3088-L3091)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetRadius(radius)` | 默认 nil，不搜索；设置为世界距离，不改变伤害值或自动重启任务。[默认](../../../raw/current/scripts/components/mine.lua#L60) [搜索](../../../raw/current/scripts/components/mine.lua#L11) [设置](../../../raw/current/scripts/components/mine.lua#L94) |
| `SetAlignment(tag?)` | 默认 player；搜索排该标签，player 还额外排 possessedbody；nil 取消阵营排除，仍保留 flying／ghost／playerghost／spawnprotection 等公共排除。不是玩家 PVP 权限系统。[过滤](../../../raw/current/scripts/components/mine.lua#L6-L29) |
| `SetSearchTags(must,oneof)`／`SetSearchTestFn(fn)` | 默认要求 _combat 与 monster／character／animal 之一，并检查存活／CanBeAttacked；**自定义标签分支不自动保留默认测试**，应给 fn(candidate,mine) 补所需条件。[搜索分支](../../../raw/current/scripts/components/mine.lua#L1-L29) [设置](../../../raw/current/scripts/components/mine.lua#L130-L136) |
| `SetTestTimeFn(fn)` | fn(inst) 返回周期秒数，StartTesting 时取一次，默认 1～2 秒随机；首次延迟仍约 0.9～1 秒，离屏已有任务改十秒周期，醒来重新采样。[调度](../../../raw/current/scripts/components/mine.lua#L150-L177) |
| `Reset()`／`Deactivate()` | 前者清 target、启用并调 onreset 后搜索；后者停搜索、未触发／停用并调 ondeactivate，**不清 target**。仅 StopTesting 不改变标签／工作状态。[重置](../../../raw/current/scripts/components/mine.lua#L139-L147) [停用](../../../raw/current/scripts/components/mine.lua#L181-L187) |
| `Spring()`／`Explode(target)`／`GetTarget()` | Spring 只置触发状态并调 onsetsprung；Explode 保存目标并调 onexplode(inst,target)。两者不互相调用，也不自动扣耐久。[方法](../../../raw/current/scripts/components/mine.lua#L190-L211) |
| `SetOnExplodeFn`／`SetOnResetFn`／`SetOnSprungFn`／`SetOnDeactivateFn` | 保存宿主回调；前者带 target，其余 fn(inst)。回调返回不决定状态成功。[设置](../../../raw/current/scripts/components/mine.lua#L98-L112) [原配置](../../../raw/current/scripts/prefabs/trap_teeth.lua#L154-L165) |
| `OnSave()`／`OnLoad(data)` | 只保存 sprung 或 inactive；加载分派 Spring／Deactivate／Reset，不存目标和测试周期。已触发加载不执行伤害回调。[存读](../../../raw/current/scripts/components/mine.lua#L214-L228) |

组件监听 onputininventory／onpickup／teleported 后停用；移除组件停任务、解绑监听并清 mine 标签。原犬牙陷阱掉落停用、部署重置；拾取／放置与攻击不是同一个回调。[组件清理](../../../raw/current/scripts/components/mine.lua#L74-L91) [宿主](../../../raw/current/scripts/prefabs/trap_teeth.lua#L90-L97)

## 典型接入

### 普通犬牙陷阱搜索触发半径改为二

目标模组 `modmain.lua`，沿用原 trap_teeth 资源、阵营、伤害和次数回调；不改麦斯威尔变体，也不将搜索半径变成范围攻击。[安装](../../../raw/current/scripts/prefabs/trap_teeth.lua#L154-L165) [原攻击](../../../raw/current/scripts/prefabs/trap_teeth.lua#L37-L45)

```lua
AddPrefabPostInit("trap_teeth", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.mine ~= nil then
        inst.components.mine:SetRadius(2)
    end
end)
```

单位为世界距离；下次 MineTest 使用新半径，不强制立即检测，不手工重置已触发陷阱。默认过滤仍排玩家等对象；原伤害只打被选 target，finiteuses 耗尽仍由原回调移除组件／延迟销毁实体。[搜索](../../../raw/current/scripts/components/mine.lua#L11-L29) [耗尽](../../../raw/current/scripts/prefabs/trap_teeth.lua#L14-L21)

半径不由组件保存，每次重建由 PostInit 应用；停用并重载恢复原配置，已造成伤害或消耗不回滚。相关消耗机制见 [[reference/components/finiteuses]]，新增爆炸效果见 [[reference/components/explosive]]。

## 常见陷阱与限制

- **Explode 自动做爆炸／Spring 也会扣次数**：只有 onexplode 的宿主效果；Spring 走另一回调，存档恢复正用它还原显示。[实现](../../../raw/current/scripts/components/mine.lua#L190-L228)
- **自定义搜索标签仍保证目标有 combat**：分支切换后默认 _combat／存活测试可能不再应用；原犬牙回调直接访问 target.components.combat，自定义前必须补足前提。[搜索](../../../raw/current/scripts/components/mine.lua#L23-L29) [消费者](../../../raw/current/scripts/prefabs/trap_teeth.lua#L37-L45)
- **不可复用标签能阻止任意重置**：仅候选排除，RESETMINE 执行和 Reset 本身不重检；自定义控制入口要另设最终门禁。[候选](../../../raw/current/scripts/componentactions.lua#L618-L620) [执行](../../../raw/current/scripts/actions.lua#L3088-L3091)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 漂移文件 componentactions.lua、actions.lua 对照 build-26148036 更新报告（R-01、R-04，另见 C-03）逐条核验，锚点重定位 2 处、结论修订 0 处（RESETMINE 动作与 mine 候选收集未变；C-03 的 aoeweapon_base OnToss 改经全局 DeactivateInventoryItemBeforeLaunch 仍调 mine:Deactivate，本页未引用该路径）；未覆盖：运行验证。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖完整 mine、普通犬牙与麦斯威尔变体的回调／注册、重置动作／候选、库存／传送停用与睡醒调度。未全面审校蜂雷／海星等全部宿主、传送链、PVP 或目标并发删除。
- 结构检查：本批格式、来源、链接／源码锚点、大小与登记；完整半径例经已有 Full Moon Lua 5.1 syntax-only，结果见本轮报告。
- 游戏运行：未运行；半径边界、阵营／飞行／死亡、重置、入栏／传送、离屏及次数耗尽待实测。
