---
title: "lighter"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "为点火工具接入 LIGHT 动作与目标 Ignite，区分点火尝试、成功燃烧及工具自身耗能。"
aliases: ["点火工具", "火把点火", "打火机点燃", "onlighterlight", "lighter"]
tags: [component, callback, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/lighter.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua", "prefabs/torch.lua", "prefabs/lighter.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lighter

## 快速判断

- 职责与适用：挂在**点火工具**上，把目标交给其 `burnable:Ignite`；不自行寻找目标、启动 AI 或创建火焰。主动定时生火另见 [[reference/components/firebug]]。[Light](../../../raw/current/scripts/components/lighter.lua#L18-L26)
- 优先使用：沿用 `LIGHT` 动作与目标 `canlight` 标签；附加工具消耗可包装 `SetOnLightFn`。燃烧时长与清理归 [[reference/components/burnable]]，传播归 [[reference/components/propagator]]，燃料归 [[reference/components/fueled]]。[动作收集](../../../raw/current/scripts/componentactions.lua#L1480-L1484)
- 重要副作用：调用目标 Ignite 时传入工具和 doer；合资格分支后调用工具回调，而 `onlighterlight` 在分支外发送给目标，无数据参数；两者均不能证明目标刚刚点燃。[Light](../../../raw/current/scripts/components/lighter.lua#L18-L26)
- 修改边界：原版火把、打火机在 pristine 前加 `lighter` 标签，服务端才加组件；组件构造也补标签，移除组件时去标签。新 Prefab 应保留这个端划分。[打火机工厂](../../../raw/current/scripts/prefabs/lighter.lua#L332-L354) [组件标签](../../../raw/current/scripts/components/lighter.lua#L1-L12)

## 关键接口

| 接口／字段 | 参数／返回 | 条件、作用与证据 |
|---|---|---|
| `Light(target, doer)` | 有效目标实体、执行者；无显式返回 | 目标有 burnable、非 INLIMBO，且不是“fueldepleted 而无 burnableignorefuel”才调用 `Ignite(nil, self.inst, doer)`；不检查 canlight。[实现](../../../raw/current/scripts/components/lighter.lua#L18-L26) |
| `SetOnLightFn(fn)`／`onlight` | 单槽 `(tool, target)`；默认 nil | 替换回调；不传 doer。在 Ignite 返回后执行，即使 Ignite 因已燃烧／fireimmune 跳过也可调用。包装时保留旧函数。[setter 与调用](../../../raw/current/scripts/components/lighter.lua#L14-L26)；Ignite 门禁见 [[reference/components/burnable#关键接口]]。 |
| `onlighterlight` | 发在 target；无 payload | 有效调用走到末尾就发送，包括无 burnable、耗尽或 INLIMBO 被拒的目标；不是成功事件。[发送位置](../../../raw/current/scripts/components/lighter.lua#L18-L26) |

原版动作链：`USEITEM` 收集时要求目标 `canlight` 与燃料／INLIMBO 门禁；`EQUIPPED` 还要求右键。`LIGHT.fn` 先给 doer 发 `onstartedfire { target=... }`，再调用 Light，只要 invobject 有 lighter 就返回 true；也不校验实际燃烧结果。SGwilson 的 LIGHT 进入 `catchonfire`，第 13 帧执行缓冲动作。[USEITEM](../../../raw/current/scripts/componentactions.lua#L1480-L1484) [EQUIPPED](../../../raw/current/scripts/componentactions.lua#L2383-L2387) [动作执行](../../../raw/current/scripts/actions.lua#L2689-L2697) [SG 注册](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1021) [状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12346-L12371)

（26148036 核验：上述两条 lighter 收集分支与 `LIGHT.fn` 均未变；同段新增 USEITEM.weapon 分支 [componentactions.lua:1976](../../../raw/current/scripts/componentactions.lua#L1976)——手持武器带 `extinguisher` 标签对阴燃／燃烧目标任意键、带 `rangedlighter` 标签对 `canlight` 目标仅右键时直接提供 ATTACK，点燃仍走攻击链 weapon 回调，不经本组件 Light／onlight；旧 `target.replica.combat == nil` 灭火回退已被官方注释标 "doesn't/didn't work?"（[componentactions.lua:1984](../../../raw/current/scripts/componentactions.lua#L1984)，存疑路径，勿当作有效点燃语义引用）。）

组件没有计时、启停或 OnSave／OnLoad；只保存于运行中实例的回调需由模组重建。移除组件由实体先停止更新、调用组件清理，再注销组件动作；目标后续燃烧不由 lighter 撤销。[组件全文](../../../raw/current/scripts/components/lighter.lua#L1-L28) [移除组件](../../../raw/current/scripts/entityscript.lua#L670-L684)

## 典型接入

### 给火把每次合资格点火调用增加燃料成本

完整 `modmain.lua`；复用原版 `torch` 工厂、资源、穿脱与耗尽清理，在服务端包装已有 onlight，每次进入 Light 的合资格分支额外扣 2 燃料单位。该费用按**点火尝试**收取；需要“首次成功点燃才扣费”的机制应另外核对目标前后燃烧状态。原版 torch 注册为同名 Prefab，并已配置 lighter 与 fueled。[火把组件](../../../raw/current/scripts/prefabs/torch.lua#L451-L475) [燃料与注册](../../../raw/current/scripts/prefabs/torch.lua#L500-L530) [PostInit 注册](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("torch", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local lighter = inst.components.lighter
    if lighter == nil then return end
    local old_onlight = lighter.onlight
    lighter:SetOnLightFn(function(tool, target)
        if old_onlight ~= nil then
            old_onlight(tool, target)
        end
        if tool:IsValid() and tool.components.fueled ~= nil then
            tool.components.fueled:DoDelta(-2)
        end
    end)
end)
```

保留原回调并防止旧回调删除工具后继续访问组件；不替换原燃料回调或自行删实体。DoDelta 归零可能走原版耗尽删除，详见 [[reference/components/fueled#关键接口]]。不会改变时间消耗率、燃烧传播或打火机本体；停用模组后新／重建实例恢复原初始化，已经扣掉的燃料不返还。没有新增任务或事件监听需要清理。

### 区分打火机的其他功能

原版 `lighter` Prefab 返回 `Prefab("lighter", fn, ...)`；装备点燃的是工具自身 burnable，卸下会 Extinguish；攻击点燃则由 weapon 回调直接 Ignite，绕过本组件的 Light／onlight。不要用一个 onlight 包装承诺覆盖所有点火及攻击行为。[注册](../../../raw/current/scripts/prefabs/lighter.lua#L399) [装备](../../../raw/current/scripts/prefabs/lighter.lua#L160-L174) [卸下与攻击](../../../raw/current/scripts/prefabs/lighter.lua#L194-L239)

## 常见陷阱与限制

- **把动作门禁当直接调用门禁**：Light 不查 canlight、工具是否装备或工具是否有剩余燃料，也不防 nil target；自定义直接调用者应先核验对象及业务资格，不把 LIGHT 返回 true 当燃烧成功。[Light](../../../raw/current/scripts/components/lighter.lua#L18-L26) [LIGHT.fn](../../../raw/current/scripts/actions.lua#L2689-L2697)
- **给工具装 lighter 就自动扣次数／燃料**：组件没有消耗逻辑；火把的时间消耗由 burnable／fueled 和穿脱协作完成，额外点火成本要像上例明确接入。[组件全文](../../../raw/current/scripts/components/lighter.lua#L1-L28)；燃料链复用 [[reference/components/fueled#典型接入]]。
- **只加服务端标签就完成新工具交互**：原版 pristine 前显式声明 lighter 标签，并由 AddComponent 注册动作；本页修改已有 torch，不替代新物品所需的网络、库存与资源构造。[原版标签／组件](../../../raw/current/scripts/prefabs/lighter.lua#L332-L354) [组件动作注册](../../../raw/current/scripts/entityscript.lua#L655-L665)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中的漂移文件 actions.lua（R-01）、componentactions.lua（R-04）、SGwilson.lua（SB-17）、prefabs/torch.lua（P-117）、entityscript.lua 逐 hunk 对照，锚点重定位 7 处；结论修订 1 处（"原版动作链"段落补记 USEITEM.weapon 新 ATTACK 分支的 extinguisher／rangedlighter 接入条件与旧灭火回退"doesn't/didn't work?"存疑标注，lighter 两条收集分支与 LIGHT.fn 本身未变）；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；完整审阅 lighter；核对动作收集／执行、SG 时点、火把与打火机注册及服务端构造、回调包装和组件移除。燃烧门禁与燃料消费复用同 build 的 burnable／fueled 页；未覆盖所有点火工具、客户端预测与技能分支。
- 结构检查：本页元数据、大小、本地链接与源码锚点经定向检查；完整 modmain 示例经 Lua 5.1 语法检查。
- 游戏运行：未运行；额外成本、耗尽时动作表现、存读档及其他模组回调兼容仍待实测。
