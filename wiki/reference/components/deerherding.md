---
title: "deerherding"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级鹿群放牧组件：维护鹿群中心与朝向、吃草/游荡节奏、受惊逃离与成员活跃判定，鹿大脑按它决定群体行为。"
aliases: ["deerherding", "鹿群放牧", "鹿群移动", "羊鹿群惊吓"]
tags: [component, creature, ai, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/deerherding.lua", "components/deerherdspawner.lua", "brains/deerbrain.lua", "prefabs/forest.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# deerherding

## 快速判断

- 职责与适用：装在森林世界实体上的鹿群“队形与节奏”组件，与 [[reference/components/deerherdspawner]]（生成/迁徙）配套：后者调 `Init` 注入羊鹿群位置，本组件每帧更新但内部节流 0.2 秒。不管理鹿的生成、存亡与迁徙。[安装](../../../raw/current/scripts/prefabs/forest.lua#L653) [Init](../../../raw/current/scripts/components/deerherding.lua#L40-L44) [节流](../../../raw/current/scripts/components/deerherding.lua#L212-L215)
- 优先使用：关闭惊吓用 `disable_spooked=true`；强制不离群用 `force_keep_herd_together=true`；自定义落点校验用 `SetValidAareaCheckFn(fn)`。三者都是构造注释里写明的模组开关。[开关注释](../../../raw/current/scripts/components/deerherding.lua#L33-L35) [落点校验](../../../raw/current/scripts/components/deerherding.lua#L46-L48)
- 重要副作用：移动群位置时会给每只活跃鹿记忆 `herdoffset` knownlocation 并 `brain:ForceUpdate()`；活跃状态直接写在 deerherdspawner 的成员表上。[队形记忆](../../../raw/current/scripts/components/deerherding.lua#L93-L96) [写活跃表](../../../raw/current/scripts/components/deerherding.lua#L149-L151)
- 修改边界：构造无 `ismastersim` 断言（依赖安装位置保证）；有 OnSave/OnLoad（群位置、 grazetimer、isgrazing）与 LoadPostPass（重绑世界 deerherdspawner）。无 OnRemoveFromEntity，停止更新无入口。[存读](../../../raw/current/scripts/components/deerherding.lua#L292-L312) [重绑](../../../raw/current/scripts/components/deerherding.lua#L314-L318)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `Init(startingpt,herdspawner)` | 设定群初始位置与所属 deerherdspawner；未 Init 前更新直接返回（Dormant）。deerherdspawner 每次成功召群都会重新调用。[实现](../../../raw/current/scripts/components/deerherding.lua#L40-L44) [未初始化返回](../../../raw/current/scripts/components/deerherding.lua#L186-L188) |
| `SetValidAareaCheckFn(fn)` | 包装为 `(inst,x,y,z)` 后作 FindWalkableOffset 的自定义校验，全部点失败则群不挪位。fn 返回真表示位置可用。[实现](../../../raw/current/scripts/components/deerherding.lua#L46-L48) [使用](../../../raw/current/scripts/components/deerherding.lua#L103-L106) |
| `CalcHerdCenterPoint(detailedinfo?)` | 统计距群位置 25 半径内的成员，返回平均位置/朝向/最远距离/成员表；0 只返回 nil。多处逻辑复用。[实现](../../../raw/current/scripts/components/deerherding.lua#L50-L78) |
| `UpdateDeerHerdingStatus()` | 每节流周期刷新成员活跃标记：无效、距群中心 >35、10 半径内 wall/structure 超 20（“城市化”）、或 20 半径（`SALTLICK_CHECK_DIST`）内有 saltlick，都判为不活跃。[实现](../../../raw/current/scripts/components/deerherding.lua#L130-L153) [盐舔判定](../../../raw/current/scripts/tuning.lua#L3265) |
| `CalcIsHerdSpooked()` | 任一活跃成员着火、有战斗目标或 hauntable panic 即受惊；`disable_spooked` 置真跳过。[实现](../../../raw/current/scripts/components/deerherding.lua#L155-L173) |
| `SetHerdAlertTarget(deer,target)`／`GetClosestHerdAlertTarget(deer)` | 鹿大脑在玩家靠近时登记警戒目标（[调用](../../../raw/current/scripts/brains/deerbrain.lua#L26-L27)），按 500 平方距离内取最近；配套 `HerdHasAlertTarget`/`IsAHerdAlertTarget`。[实现](../../../raw/current/scripts/components/deerherding.lua#L252-L290) |
| `IsGrazing()`／`IsActiveInHerd(deer)` | 吃草查询被大脑用于行为门（[消费](../../../raw/current/scripts/brains/deerbrain.lua#L79)）；活跃查询是群体节点条件（[消费](../../../raw/current/scripts/brains/deerbrain.lua#L130)）。 |

### 更新循环：吃草/游荡/受惊

- 非受惊时：有成员睡着立即进入吃草；`grazetimer` 归零后切换吃草/游荡，分别计时 `grazing_time`/`roaming_time`（各 20 秒，构造可覆盖）（[节奏](../../../raw/current/scripts/components/deerherding.lua#L195-L203)、[常量](../../../raw/current/scripts/components/deerherding.lua#L10-L11)）。
- 受惊沿平均玩家方位反方向走：20 半径（`DEER_HERD_MOVE_DIST*2`）内玩家取平均位置算逃离朝向，`keepheading=true` 锁定，按 2 倍距离挪群（[逃离](../../../raw/current/scripts/components/deerherding.lua#L221-L240)、[调参](../../../raw/current/scripts/tuning.lua#L686)）；刚受惊时结束吃草、计时减半。
- 游荡挪群按 `DEER_HERD_MOVE_DIST`(10)：中心偏移超过 radius*2.8 时不挪（成员太散）；挪前先按队形散布重记 `herdoffset`，找可行走点时先避墙再退而避水，最后强刷成员大脑（[挪群](../../../raw/current/scripts/components/deerherding.lua#L80-L119)）。

## 典型接入

### 调整羊鹿群行为开关

- 前提与选择：服务端、森林世界已安装。`disable_spooked`、`force_keep_herd_together`、`valid_area_check` 都是源码注释声明的定制点；前者关闭惊吓判定，后者让群只在你认可的区域内挪动。改动即时生效，不改生成与迁徙（那是 deerherdspawner 的职责）。
- 生效与撤销：布尔开关直接写回即可恢复默认（nil）；`force_keep_herd_together` 源码注释提醒“可能把鹿圈死在墙内”，慎用（[注释](../../../raw/current/scripts/components/deerherding.lua#L35)）。

```lua
-- 服务端：森林世界 master_postinit 之后
local herding = TheWorld.components.deerherding
if herding ~= nil then
    herding.disable_spooked = true            -- 不再因火焰/目标/惊吓判定群受惊
    herding.grazing_time = 10                  -- 吃草时长（默认 20）
    herding.roaming_time = 30                  -- 游荡时长（默认 20）
    herding:SetValidAareaCheckFn(function(inst, x, y, z)
        return TheWorld.Map:IsPassableAtPoint(x, 0, z)
    end)
end
```

- 调整与验证：时间字段单位秒，`grazetimer` 存档恢复（[OnSave](../../../raw/current/scripts/components/deerherding.lua#L300-L301)）；`SetValidAareaCheckFn` 的 fn 在每次挪群时最多被调用 8×2 次，保持轻量。

## 常见陷阱与限制

- **把它当每只鹿的组件：** 组件在世界上、单实例管理整群；个体差异靠各鹿大脑查询（herdlocation、IsGrazing、alerttargets）。给鹿实体加同名组件不会被任何消费方读取（消费方固定走 `TheWorld.components.deerherding`）。[消费方式](../../../raw/current/scripts/brains/deerbrain.lua#L57-L80)
- **SetHerdAlertTarget 里的 keepheading 分支疑似笔误：** 插入成功后才检查 `next(self.alerttargets) == nil`，此时恒为假，`keepheading=true` 实际不会执行；不要依赖该副作用。[分支](../../../raw/current/scripts/components/deerherding.lua#L252-L262)
- **活跃判定会被建筑与盐舔“赶散”：** “城市化”与盐舔成员直接判不活跃，群行为只对剩余成员生效；想限制该行为需覆盖 UpdateDeerHerdingStatus 或调整阈值常量（文件顶部局部量，构造后不可改）。[判定](../../../raw/current/scripts/components/deerherding.lua#L2-L8) [常量捕获](../../../raw/current/scripts/components/deerherding.lua#L125-L128)
- **中心点统计不含不活跃成员：** CalcHerdCenterPoint 遍历的是 spawner 全部成员，但只把 25 半径内的计入平均；被判定不活跃的鹿仍留在 spawner 表里，只是行为脱队。[统计口径](../../../raw/current/scripts/components/deerherding.lua#L57-L71)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 deerherding.lua 与 deerherdspawner.lua，核验 Init 链路、更新循环、惊吓/城市化/盐舔判定、警戒目标协议、存读与鹿大脑消费点；核对 TUNING 常量与森林安装。未运行游戏；deer 实体与迁徙事件处理归 deer 页面范围。

相关：[[reference/components/deerherdspawner]]、[[reference/components/klaussackspawner]]。
