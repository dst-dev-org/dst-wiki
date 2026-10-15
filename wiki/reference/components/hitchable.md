---
title: "hitchable"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "管理被拴生物侧引用、hitched 标签与战斗解拴，配合拴柱和 Brain 完成拴住流程。"
aliases: ["服务端释放拴牛", "被拴生物", "牛解拴", "hitchable"]
tags: [component, creature, ai, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/hitchable.lua", "components/hitcher.lua", "prefabs/beefalo.lua", "prefabs/beefalo_groomer.lua", "brains/beefalobrain.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hitchable

## 快速判断

- 职责与适用：被拴生物侧保存柱子引用与 canbehitched，联动 hitched 标签，并在 newcombattarget 时解拴。原牛安装此组件。[定义](../../../raw/current/scripts/components/hitchable.lua#L1-L29) [牛宿主](../../../raw/current/scripts/prefabs/beefalo.lua#L1280)
- 优先使用：查询 GetHitch；建立关系由柱子 [[reference/components/hitcher]]:SetHitched 发起，释放可用生物侧 Unhitch 并保留双方协作，不只清一个引用。
- 重要副作用：Unhitch 先去标签／战斗监听，再按对方状态回调柱端。生物组件本身不冻结移动；牛 Brain 在 hitched 且没有待到达柱时 StandStill。[释放](../../../raw/current/scripts/components/hitchable.lua#L31-L43) [Brain](../../../raw/current/scripts/brains/beefalobrain.lua#L211-L215)
- 修改边界：HITCHUP 是让牛前往目标的请求，不是立刻建立关系；普通 MOUNT 也会拒绝已拴或正在去柱的牛。[请求](../../../raw/current/scripts/actions.lua#L2712-L2747) [骑乘限制](../../../raw/current/scripts/actions.lua#L4147-L4156)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `canbehitched`／`GetHitch()` | 默认可拴／无柱；false 加 hitched 标签，true 去标签。Get 只读引用，不验证柱是否有效，不把标签当关系完整性证明。[定义](../../../raw/current/scripts/components/hitchable.lua#L1-L17) [查询](../../../raw/current/scripts/components/hitchable.lua#L41-L43) |
| `SetHitched(post)` | 播音效、监听 newcombattarget、canbehitched=false、记录 post；不校验距离、占用、锁定或已有关系，也不建立柱端引用。正常由 hitcher 调它。[方法](../../../raw/current/scripts/components/hitchable.lua#L23-L29) [正常调用方](../../../raw/current/scripts/components/hitcher.lua#L33-L44) |
| `Unhitch()` | 播音效、去监听、先设可拴，再通知仍处于占用状态的柱；最后清引用。内部假定柱／组件有效，无通用 IsValid 防护，不可用来修复任意损坏关系。[实现](../../../raw/current/scripts/components/hitchable.lua#L31-L39) |
| `OnSave/OnLoad` | 返回空表／不恢复字段；本组件不保存柱 GUID、布尔或监听，不能据此承诺读档重连。[实现](../../../raw/current/scripts/components/hitchable.lua#L45-L52) |

牛的 hitchto 事件记 hitchingspot、注册死亡／睡眠／点燃／移除清理；Brain 向目标提交 HITCH，动作再由柱侧建立关系。原柱回调向牛发 unhitch，牛才去掉这些监听，不能省略宿主回调。[牛事件](../../../raw/current/scripts/prefabs/beefalo.lua#L861-L880) [Brain 提交](../../../raw/current/scripts/brains/beefalobrain.lua#L165-L173) [动作](../../../raw/current/scripts/actions.lua#L2760-L2765) [装扮台回调](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L123-L133)

## 典型接入

### 为原牛提供服务端释放事件，保留柱端回调

目标模组 `modmain.lua`，无资源。仅接受有效双向关系且柱未锁定时的事件；调用方在选定牛上发 `beefalo:PushEvent("my_mod_unhitch")`。这里只交付接收端，不含按键、选牛或 RPC／权限流程。

```lua
AddPrefabPostInit("beefalo", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_unhitch", function(beefalo)
        local member = beefalo.components.hitchable
        local post = member ~= nil and member:GetHitch() or nil
        if post == nil or not post:IsValid() then return end
        local hitcher = post.components.hitcher
        if hitcher ~= nil and not hitcher.locked
            and hitcher:GetHitched() == beefalo then
            member:Unhitch()
        end
    end)
end)
```

锁定检查是此事件的调用策略，Unhitch 本身不检查 locked；所有收发事件名应一致。释放通过双方方法和原装扮台／柱子回调清理，不直接写 hitched=nil；失效引用和不匹配关系保留为调用方缺口，不硬调方法。停用重载后不再注册事件，原宿主关系与动作仍按原流程处理。

本例不处理尚在 hitchingspot 途中、还没有 GetHitch 的请求；不能把“未拴住”当已经取消去柱。组件也不代办群体／驯化、上骑或强制移动。[待到达状态](../../../raw/current/scripts/prefabs/beefalo.lua#L867-L880)

## 常见陷阱与限制

- **hitched 标签就是完整拴绳／移动冻结**：双向引用、柱回调与 Brain 都独立，单独增删标签可产生半状态。[组件](../../../raw/current/scripts/components/hitchable.lua#L1-L29) [Brain](../../../raw/current/scripts/brains/beefalobrain.lua#L211-L215)
- **Unhitch 对失效柱无条件安全／自动取消所有途中请求**：内部直接访问柱组件；本例只处理有效双向已拴关系，途中请求要另按宿主状态处理。[方法](../../../raw/current/scripts/components/hitchable.lua#L31-L39)
- **卸组件或读档会自动完整清理／重连**：没有专用卸载钩子和关系保存；实际销毁清理依赖宿主事件，不把空 OnLoad 当恢复协议。[空保存](../../../raw/current/scripts/components/hitchable.lua#L45-L52) [宿主清理](../../../raw/current/scripts/prefabs/beefalo.lua#L861-L880)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；漂移文件 actions.lua、prefabs/beefalo.lua 对照 R-01/R-04 hunk 地图逐条核验：HITCHUP／HITCH／MOUNT 拒拴分支逐行未变，beefalo 内容无差异（--ignore-cr-at-eol 下无 diff），actions.lua 本批变更不涉拴牛域；锚点重定位 3 处、结论无修订；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整生物侧组件、柱端互调、牛请求／清理、Brain StandStill／提交、动作限制与装扮台回调。未全面审校牛全部 SG、铃铛归属／权限、异步关系或损坏存档。
- 结构检查：本批格式、来源、链接／行号、大小与登记通过；完整释放事件例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；实际拴住／途中请求、锁定／解锁、战斗／睡眠／移除、回调重入与多模组关系变更待实测。
