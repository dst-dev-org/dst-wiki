---
title: "projectile"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "实体目标型投射物的追踪、命中与伤害归属，区分自身投掷和独立弹体。"
aliases: ["projectile", "追踪弹", "吹箭速度", "远程弹体", "投射物命中"]
tags: [component, item, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/projectile.lua", "components/weapon.lua", "components/combat.lua", "prefabs/blowdart.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# projectile

## 快速判断

- 职责与适用：朝目标实体移动，默认追踪，通过距离／轨迹判断命中，再调用攻击者 combat；落点抛物线选 [[reference/components/complexprojectile]]，两组件不应同挂。[projectile.lua:11](../../../raw/current/scripts/components/projectile.lua#L11)、[:43](../../../raw/current/scripts/components/projectile.lua#L43)
- 优先使用：工厂设置速度与命中／落空回调；普通远程武器用 [[reference/components/weapon#关键接口|SetProjectile]] 接发射链，直接 Throw 需提供有效 owner／target、Physics 和起点。[weapon.lua:123](../../../raw/current/scripts/components/weapon.lua#L123)、[projectile.lua:160](../../../raw/current/scripts/components/projectile.lua#L160
- 重要副作用：Hit 在 onhit 前执行 combat:DoAttack；命中回调再做普通攻击可能重复结算。Stop 不删除实体，Miss 也不自动停止物理速度，清理由 Prefab 回调负责。[projectile.lua:205](../../../raw/current/scripts/components/projectile.lua#L205)、[:236](../../../raw/current/scripts/components/projectile.lua#L236)
- 修改边界：服务端配置组件；新 Prefab 在 SetPristine 前声明 projectile 标签并提供物理／网络；已有吹箭 PostInit 可复用原版工厂。[blowdart.lua:48](../../../raw/current/scripts/prefabs/blowdart.lua#L48

## 关键接口

| 接口／字段 | 参数／默认 | 条件、作用与证据 |
|---|---|---|
| `SetSpeed(speed)` | number；初始 nil，须配置 | 已投掷时立即改 SetMotorVel；不是攻击伤害或射程。[projectile.lua:93](../../../raw/current/scripts/components/projectile.lua#L93 |
| `SetHoming(homing)` | 默认 true | true 更新目标坐标；false 不再自动转向，仍只检测指定目标，不自动击中沿途任意实体。[projectile.lua:144](../../../raw/current/scripts/components/projectile.lua#L144)、[:367](../../../raw/current/scripts/components/projectile.lua#L367)、[:462](../../../raw/current/scripts/components/projectile.lua#L462) |
| `SetHitDist(dist)`／`SetRange(range)` | 默认 1／nil | 判中距离加目标物理半径，使用三维距离；range 从 start 起算，超过才 Miss，不是 weapon.attackrange。[projectile.lua:104](../../../raw/current/scripts/components/projectile.lua#L104)、[:367](../../../raw/current/scripts/components/projectile.lua#L367) |
| `Throw(owner, target, attacker?)` | 无显式成功返回 | 保存 owner／target，启用更新／activeprojectile，向弹体／目标发 onthrown／hostileprojectile；第三参用于通知与发射偏移，不替换 self.owner。[projectile.lua:160](../../../raw/current/scripts/components/projectile.lua#L160 |
| `SetLaunchOffset(Vector3)`／`SetLaunchAngle(degrees)` | 偏移 x 朝向距离、y 高度、z 忽略 | 偏移仅 Throw 第三参非 nil 时应用；不传角度则朝目标。自身投掷链通常只传两参。[projectile.lua:148](../../../raw/current/scripts/components/projectile.lua#L148)、[combat.lua:1168](../../../raw/current/scripts/components/combat.lua#L1168 |
| `SetOnThrownFn(fn)` | `(inst, owner, target, attacker)` | 更新／标签／事件之后调用，不承担自动拆堆。[projectile.lua:181](../../../raw/current/scripts/components/projectile.lua#L181 |
| `SetOnPreHitFn(fn)`／`SetOnHitFn(fn)` | `(inst, attacker, target)` | Stop 与 Physics:Stop 后，依次 prehit→DoAttack→onhit，DoAttack 后弹体已失效则跳过 onhit；返回值不取消攻击，不假定 target 仍有效。[projectile.lua:236](../../../raw/current/scripts/components/projectile.lua#L236 |
| `SetOnMissFn(fn)`／`SetOnCaughtFn(fn)` | `(inst, attacker, target)`／`(inst, catcher)` | Miss 停组件但不 Physics:Stop；Catch 仅 cancatch 时执行并停止物理，默认不消费／入栏。[projectile.lua:194](../../../raw/current/scripts/components/projectile.lua#L194 |
| `SetCanCatch(bool)`／`IsThrown()` | 默认 false／target 非 nil | catchable 标签还依赖有 target；有 catcher 的目标才由 Throw 启动观察，不是自动让玩家接住。[projectile.lua:3](../../../raw/current/scripts/components/projectile.lua#L3)、[:189](../../../raw/current/scripts/components/projectile.lua#L189) |

### 伤害与发射分工

1. **物品自身飞出去**：combat 将带 projectile 的武器从物品栏 DropItem 后 Throw，例如 blowdart_pipe；owner 是人物，弹体自身 weapon 决定武器伤害。[combat.lua:1168](../../../raw/current/scripts/components/combat.lua#L1168)、[blowdart.lua:235](../../../raw/current/scripts/prefabs/blowdart.lua#L235
2. **武器发射另一 Prefab**：weapon:LaunchProjectile 生成弹体，Throw 的 owner 是发射武器、attacker 是人物。Hit 将 owner 还原到武器的直接持有者；通常用发射武器结算，只有弹体 `has_damage_set=true` 且自带 weapon 时保留弹体武器。[weapon.lua:129](../../../raw/current/scripts/components/weapon.lua#L129)、[projectile.lua:236](../../../raw/current/scripts/components/projectile.lua#L236
3. Hit 向 DoAttack 传当前弹体，避免再次发射；伤害、回调和扣次由 [[reference/components/combat]]／[[reference/components/weapon]] 结算，onhit 不是净伤害成功确认。[combat.lua:1168](../../../raw/current/scripts/components/combat.lua#L1168)、[:1212](../../../raw/current/scripts/components/combat.lua#L1212)

（26148036 核验：Hit 在 DoAttack 后新增弹体有效性早退——combat 结算中弹体被移除（如 Deflect、回调内 Remove）时跳过 onhit，依赖该回调做清理的包装需复查；新增偏转 API `SetOnDeflectFn`／`SetKeepOnDeflect`／`Deflect`；weapon 新增 `weapononprojectilelaunched` 事件且 `weapononattack` 事件附带 `projectile` 字段，本页发射链与归属结论不变。目标侧端点见 [[reference/components/deflectable]]。）

## 典型接入

### 普通吹箭飞行速度改为四十

追加到目标模组 `modmain.lua`；无需全局代理、新资源或注册。注册名 blowdart_pipe，原版速度 60，已有命中效果／删除和 Miss 删除；只改速度，不覆盖回调。[blowdart.lua:89](../../../raw/current/scripts/prefabs/blowdart.lua#L89)、[:31](../../../raw/current/scripts/prefabs/blowdart.lua#L31)、[:315](../../../raw/current/scripts/prefabs/blowdart.lua#L315)

```lua
AddPrefabPostInit("blowdart_pipe", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local projectile = inst.components.projectile
    if projectile ~= nil then
        projectile:SetSpeed(40)
    end
end)
```

40 是物理马达速度，不改基础伤害、攻击距离或消耗。保留原版投出后取消存档、命中／Miss 删除；关闭模组后新建／载入物品使用原版配置，已消耗吹箭不恢复。[blowdart.lua:229](../../../raw/current/scripts/prefabs/blowdart.lua#L229)、[:243](../../../raw/current/scripts/prefabs/blowdart.lua#L243)

## 常见陷阱与限制

- **不追踪就得到碰撞子弹**：非 homing 分支只检查指定 target 和卡住／超程；任意实体碰撞、穿透或群伤需另行实现。[projectile.lua:374](../../../raw/current/scripts/components/projectile.lua#L374
- **第三参固定伤害来源**：Hit 读 self.owner；独立弹体链再读武器 inventoryitem.owner。移交／移出武器后的归属不能只靠 Throw 第三参保证。[projectile.lua:160](../../../raw/current/scripts/components/projectile.lua#L160)、[:243](../../../raw/current/scripts/components/projectile.lua#L243)
- **Stop 等于落地回收**：它恢复 LIMITS 碰撞、清 owner／target、停更新；停止速度／删除／回收需明确处理。移除组件主要清标签和任务。[projectile.lua:217](../../../raw/current/scripts/components/projectile.lua#L217)、[:73](../../../raw/current/scripts/components/projectile.lua#L73)
- **离屏永远飞行**：有 target 时睡眠启动两秒后 Miss，唤醒取消任务；DelayVisibility 还有独立隐藏与发射者状态监听，不只是视觉开关。[projectile.lua:268](../../../raw/current/scripts/components/projectile.lua#L268)、[:505](../../../raw/current/scripts/components/projectile.lua#L505)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 29 处、结论修订 1 处（Hit 有效性早退与偏转 API，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖构造／配置、Throw／Hit／Miss／Catch／Stop、追踪／非追踪、离屏／隐藏，核对 weapon／combat 两种发射链和吹箭工厂。未审校 catcher、全部弹体／攻击状态或跨模组改写。
- 结构检查：发布时核对格式、来源、大小、链接／锚点与唯一登记；完整 modmain 示例只作 Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；速度、延迟可见、目标消失、武器移交、客户端表现和读档待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/complexprojectile]]；[[reference/components/weapon]]；[[reference/components/combat]]。
