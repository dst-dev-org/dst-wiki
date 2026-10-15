---
title: "positionalwarp"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "记录移动位置采样，供旺达倒走表选取历史落点；回退点数不代表秒数。"
aliases: ["旺达回溯距离", "倒走表历史位置", "位置回退", "positionalwarp"]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/positionalwarp.lua", "prefabs/wanda.lua", "prefabs/pocketwatch.lua", "modutil.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# positionalwarp

## 快速判断

- 职责与适用：维护位置环形历史与可选标记；选点本身不移动人物，也不提供施法动作／冷却。[取点](../../../raw/current/scripts/components/positionalwarp.lua#L125-L144)、[倒走表消费](../../../raw/current/scripts/prefabs/pocketwatch.lua#L368-L381)
- 优先使用：`SetWarpBackDist(n)` 设回退采样点数；`GetHistoryPosition(false)` 预读，成功回退后由宿主消费历史。[接口](#关键接口)
- 重要副作用：传 `true` 会移动历史游标并更新标记，但不执行传送；不能在实际传送失败前无条件消费。[取点实现](../../../raw/current/scripts/components/positionalwarp.lua#L125-L144)
- 修改边界：默认每 0.1 秒检查，移动超过 2 单位才记录；无 SG／jumping 状态和禁传送位置不记点，不能用于精确“回到 N 秒前”。[构造](../../../raw/current/scripts/components/positionalwarp.lua#L12-L29)、[采样](../../../raw/current/scripts/components/positionalwarp.lua#L99-L123)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetWarpBackDist(num_cache_points)` | 写 `history_rollback_dist`，默认 1；不校验整数或边界，建议正整数。[实现](../../../raw/current/scripts/components/positionalwarp.lua#L71-L74) |
| `GetHistoryPosition(rewind)` | 空历史返回 nil，否则返回 x,y,z；最多回退指定点数，遇最旧点停止；true 消费游标。[实现](../../../raw/current/scripts/components/positionalwarp.lua#L125-L144) |
| `Reset()` | 清游标并把当前位置作为基点；当前位置禁传送则基点写 0,0,0，尚无可回退点。[实现](../../../raw/current/scripts/components/positionalwarp.lua#L146-L157) |
| `SetMarker(prefab)` | 默认建立三个复用标记；需可生成的 Prefab 及 ShowMarker／HideMarker，SetMarkerViewer 可选。不是任意 FX 名都可用。[建立](../../../raw/current/scripts/components/positionalwarp.lua#L52-L69)、[显示](../../../raw/current/scripts/components/positionalwarp.lua#L76-L97) |
| `EnableMarker(enable)` | 只控制历史落点标记可见性，不停止采样或禁止传送。[实现](../../../raw/current/scripts/components/positionalwarp.lua#L159-L162) |
| `OnSave()`／`OnLoad(data)` | 保存历史数组与游标，不保存回退点数或 marker 配置；迁移存在 `inst.migration` 时不恢复历史。[实现](../../../raw/current/scripts/components/positionalwarp.lua#L164-L187) |

## 生命周期

构造启动采样并安排 0 延迟 Reset；正常 OnLoad 恢复历史后取消初始 Reset。历史容量默认 60，游标相撞推进最旧端；修改容量不是只改一个数字就完成数组迁移。[初始化](../../../raw/current/scripts/components/positionalwarp.lua#L12-L29)、[环形推进](../../../raw/current/scripts/components/positionalwarp.lua#L113-L121)、[加载](../../../raw/current/scripts/components/positionalwarp.lua#L174-L187)

旺达服务端挂载组件并延迟创建原版标记。倒走表只预读落点并写 SG 数据；旺达 `OnWarpBack` 再消费或 Reset。移除组件取消周期任务、移除标记；外部移除标记会尝试重建，不应通过手动删 marker 来“关闭”。[宿主](../../../raw/current/scripts/prefabs/wanda.lua#L390-L401)、[消费](../../../raw/current/scripts/prefabs/wanda.lua#L339-L348)、[清理与重建](../../../raw/current/scripts/components/positionalwarp.lua#L33-L61)

## 典型接入

### 旺达各年龄的倒走表回退点数翻倍

目标模组 `modmain.lua`，包裹组件 setter，保留旺达不同年龄调用时的配置；仅作用旺达，沿用倒走表／SG／标记资源，不另造传送动作。[年龄调用](../../../raw/current/scripts/prefabs/wanda.lua#L86)、[原 setter](../../../raw/current/scripts/components/positionalwarp.lua#L71-L74)、[组件 Hook](../../../raw/current/scripts/modutil.lua#L567-L573)

```lua
AddComponentPostInit("positionalwarp", function(self)
    if not GLOBAL.TheWorld.ismastersim or self.inst.prefab ~= "wanda" then
        return
    end
    local old = self.SetWarpBackDist
    self.SetWarpBackDist = function(component, points)
        return old(component, points * 2)
    end
    self:SetWarpBackDist(self.history_rollback_dist)
end)
```

`2` 是点数倍率；实际可回退长度受已存轨迹与边界限制，不能称“距离严格翻倍”。组件 Hook 在构造完成后、人物后续配置前接入（[调用顺序](../../../raw/current/scripts/entityscript.lua#L655-L667)）；之后年龄再次调用仍走包装方法。重新构造实体时会重新设置，不把倍率写入历史存档。

## 常见陷阱与限制

- **把 false 取点当回退：** 不移动实体、不耗冷却、不消耗历史；实际施法仍依赖倒走表和 SG。[原版消费入口](../../../raw/current/scripts/prefabs/pocketwatch.lua#L368-L381)
- **把历史当路径安全保证：** 缓存时只检当前位置许可，不等于未来任意目标的可达／地图／平台安全验证；迁移也明确不恢复历史。[采样许可](../../../raw/current/scripts/components/positionalwarp.lua#L107-L111)、[加载条件](../../../raw/current/scripts/components/positionalwarp.lua#L174-L187)
- **仅挂组件就给其他角色倒走表：** 原版需要物品资格、SG、事件和资源协作；本页只覆盖旺达参数修改，不提供通用角色回溯骨架。

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 0 处（命中 entityscript.lua／prefabs/wanda.lua 的引用区间本版未变）；结论修订：无。R-09 会话落点保护改走 virtualroommanager（mainfunctions），本页采样／存档链不涉该路径。
- 依据：全局枢纽 hunk 地图（R-08/R-09）＋变更清单命中 entityscript.lua、prefabs/wanda.lua。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖采样／环形历史、取点消费、标记／保存清理及旺达和倒走表的必要协作；未全面审阅 SG 实际移动与任意地形安全。
- 结构与语法：本批执行格式、登记、链接／源码锚点检查及完整示例 Lua 5.1 语法检查；结果见本批交付报告。
- 游戏运行：未运行；年龄切换、连续倒走、读档／迁移、标记重建、多人和其他模组方法包装待实测。
