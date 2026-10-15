---
title: "groomer"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "管理装扮设施的占用对象、换装用户与关闭清理；皮肤应用和弹窗协议由宿主提供。"
aliases: ["装扮台换装距离", "关闭牛换装", "groomer"]
tags: [component, building, ui, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/groomer.lua", "prefabs/beefalo_groomer.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# groomer

## 快速判断

- 职责与适用：设施保存 occupant 和 changers，驱动玩家换装状态并委托宿主应用皮肤；原牛装扮台安装 groomer、皮肤与拴柱回调。[构造](../../../raw/current/scripts/components/groomer.lua#L19-L61) [宿主配置](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L239-L248)
- 优先使用：CanBeginChanging 检查准入，BeginChanging 开始；EndChanging／EndAllChanging 清理。SetOccupant 只设置对象，不能替代 [[reference/components/hitcher]] 的拴牛关系。[准入](../../../raw/current/scripts/components/groomer.lua#L122-L139) [占用](../../../raw/current/scripts/components/groomer.lua#L64-L66)
- 重要副作用：首个用户启动组件更新；关闭会注销用户事件并退出 inwardrobe 状态；离开 range 或失去视线会自动结束。[开启](../../../raw/current/scripts/components/groomer.lua#L141-L171) [关闭](../../../raw/current/scripts/components/groomer.lua#L174-L205) [距离](../../../raw/current/scripts/components/groomer.lua#L249-L264)
- 修改边界：Enable(false) 只影响准入，不主动关闭当前用户；BeginChanging 本身不调用 CanBeginChanging。服务端调用需遵守完整前置协议。[开关](../../../raw/current/scripts/components/groomer.lua#L77-L79) [检查](../../../raw/current/scripts/components/groomer.lua#L122-L159)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetOccupant(target)`／`GetOccupant()` | 保存引用；occupantisself 为真时 Get 返回设施本身。没有自动绑定／保存对象的协议。[设置](../../../raw/current/scripts/components/groomer.lua#L64-L66) [获取](../../../raw/current/scripts/components/groomer.lua#L118-L120) |
| `SetCanUseAction(bool)`／`SetCanBeDressed(bool)` | 分别联动 groomer／dressable 标签，不构成完整的可用性检查。[属性](../../../raw/current/scripts/components/groomer.lua#L1-L17) [方法](../../../raw/current/scripts/components/groomer.lua#L69-L79) |
| `SetRange(distance)` | 默认 3；仅在 OnUpdate 对已有用户检查实体距离与视线。不能据此宣称动作发起距离或远程访问已放宽。[赋值](../../../raw/current/scripts/components/groomer.lua#L110-L112) [消费者](../../../raw/current/scripts/components/groomer.lua#L249-L264) |
| `CanBeginChanging(doer)` | 要有可用玩家 SG；拒绝 busy（opengift 除外）、关闭设施等。shareable 分支提前成功；canbeshared 控制另一条多人检查，不混用这两个字段。[实现](../../../raw/current/scripts/components/groomer.lua#L122-L139) |
| `SetCanBeShared(bool)` | 调整 canbeshared 并增删 ignite 监听；不是 shareable 的 setter。点燃处理会结束当前用户。[设置](../../../raw/current/scripts/components/groomer.lua#L81-L108) |
| `ActivateChanging(doer, skins)` | 要有 skins、玩家处于 openwardrobe、有 occupant，且通过可选 canactivatechangingfn；随后通过 dressupwardrobe／目标 skin_change 委托 ApplyTargetSkins。[转换](../../../raw/current/scripts/components/groomer.lua#L208-L244) |
| 宿主回调 | canbeginchangingfn／beginchangingfn／canactivatechangingfn／applytargetskinsfn 接收设施、occupant、doer，最后两者还带 skins；onclosepopupfn 将弹窗数据转为 skins。原牛换装检查毛量／皮肤差异并应用各部位。[宿主实现](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L142-L189) |

EndAllChanging 先复制用户表再逐一关闭；移除组件会关闭用户、清 ignite 监听和两个标签。原设施将拴牛／解拴回调接到 SetOccupant，unhitched 还用于关闭用户；不要仅写 occupant 绕过双方协议。[清理](../../../raw/current/scripts/components/groomer.lua#L198-L205) [移除](../../../raw/current/scripts/components/groomer.lua#L268-L275) [宿主桥接](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L116-L128)

## 典型接入

### 原牛装扮台允许用户在五单位内维持换装

目标模组 `modmain.lua`，无新资源；只改原设施已有用户的距离关闭阈值，保留视线、准入、拴牛、皮肤和弹窗协议。

```lua
AddPrefabPostInit("beefalo_groomer", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local groomer = inst.components.groomer
    if groomer ~= nil then
        groomer:SetRange(5)
    end
end)
```

配置在设施构造后生效，已有用户下一次 OnUpdate 也使用新值；不改变动作选取距离、不突破视线，也不提供新皮肤资源／界面。组件没有本字段的存读逻辑，重载由本 PostInit 重设；停用并重载恢复原构造值。需要立即禁用时须同时 Enable(false) 与 EndAllChanging，不能仅关准入后遗留已有用户。[更新消费](../../../raw/current/scripts/components/groomer.lua#L249-L264) [准入与结束](../../../raw/current/scripts/components/groomer.lua#L122-L139)

## 常见陷阱与限制

- **只 AddComponent 就能为任意实体换皮肤**：原宿主还提供 occupant、弹窗解析、皮肤差异与应用回调，以及玩家／目标状态支持；组件不生成皮肤或界面资源。[配置](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L239-L248) [应用](../../../raw/current/scripts/components/groomer.lua#L208-L244)
- **BeginChanging 会自动拒绝不合格用户**：它直接执行 beginchangingfn 并操作 SG；外部脚本先遵循 CanBeginChanging，不能向无 SG 对象调用。[入口](../../../raw/current/scripts/components/groomer.lua#L141-L159)
- **SetOccupant(nil) 等同完整解拴**：这只是引用赋值；原设施通过 hitcher 的解除事件和回调协作。[占用桥接](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L116-L128)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、准入／转换／关闭、距离／视线、回调签名与原装扮台皮肤和占用桥接。未全面审校弹窗网络、客户端预测、皮肤授权及全部状态图内部。
- 结构检查：本批格式、来源、链接／行号、大小与登记通过；完整范围例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；开始／取消换装、五单位边界／遮挡、拴牛解除、点燃／删除与多用户待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。