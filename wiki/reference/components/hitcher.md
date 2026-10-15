---
title: "hitcher"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "管理拴柱侧占用、锁定标签与双方回调，区分动作候选锁、关系建立和宿主销毁清理。"
aliases: ["装扮台锁定拴牛交互", "拴柱占用", "hitcher"]
tags: [component, building, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/hitcher.lua", "components/hitchable.lua", "componentactions.lua", "actions.lua", "prefabs/beefalo_groomer.lua", "prefabs/yotb_post.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hitcher

## 快速判断

- 职责与适用：拴柱侧保存 hitched 引用、可占用／锁定标志，配合生物侧 [[reference/components/hitchable]]；原装扮台与比赛柱各自安装并提供宿主回调。[定义](../../../raw/current/scripts/components/hitcher.lua#L17-L44) [装扮台](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L262-L270)
- 优先使用：GetHitched 查询、SetHitched 建双方关系、Unhitch 解除；Lock 只联动 hitcher_locked 标签，不是所有调用路径的硬拒绝。[方法](../../../raw/current/scripts/components/hitcher.lua#L29-L62)
- 重要副作用：SetHitched 会先改柱占用，再通知生物，最后 hitchedfn；Unhitch 先恢复可用，再通知生物、调 unhitchfn、清引用并向柱发 unhitched。顺序用于避免双方无限互调。[建立／释放](../../../raw/current/scripts/components/hitcher.lua#L33-L58)
- 修改边界：组件不自行验证距离／归属／锁定／替换旧牛；正常 HITCHUP／Brain／HITCH 动作负责前置流程。不能对一个已占柱直接再 SetHitched 作为安全换牛操作。[请求](../../../raw/current/scripts/actions.lua#L2712-L2747) [建立动作](../../../raw/current/scripts/actions.lua#L2760-L2765)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `canbehitched`／`locked` | 默认 true／false；前者真加 hitcher 标签、占用时去掉，后者控制 hitcher_locked 标签。不要把无 hitcher 标签简单理解为组件不存在。[属性](../../../raw/current/scripts/components/hitcher.lua#L1-L27) |
| `SetHitched(target)`／`GetHitched()` | 建立不检查 enabled／距离／锁，只在 target 有 hitchable 时回调；Get 可能为 nil／失效引用，不验证完整性。[实现](../../../raw/current/scripts/components/hitcher.lua#L29-L44) |
| `hitchedfn`／`unhitchfn` | 字段回调 fn(post,target/oldtarget)，没有 SetOnHitchedFn 之类方法；保留宿主原回调。装扮台用于更新 groomer 占用并向旧牛发 unhitch。[回调字段](../../../raw/current/scripts/components/hitcher.lua#L33-L58) [宿主](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L116-L128) |
| `Unhitch()` | 互调后回调再清引用／发 unhitched；不是任意空占用下都安全，装扮台 unhitchfn 直接用 oldtarget，因此调用前查 GetHitched。[方法](../../../raw/current/scripts/components/hitcher.lua#L46-L58) [宿主要求](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L123-L128) |
| `Lock(bool)` | 只赋 locked；候选筛选会据标签隐藏 HITCHUP／UNHITCH，SetHitched／Unhitch 不读它，低层动作执行也没有统一锁校验。不据它推断跨 RPC／脚本的硬限制。[锁](../../../raw/current/scripts/components/hitcher.lua#L60-L62) [候选](../../../raw/current/scripts/componentactions.lua#L431-L439) [动作](../../../raw/current/scripts/actions.lua#L2750-L2765) |

两个组件 OnSave 返回空数据、OnLoad 不重建关系或锁；装扮台自身保存也只处理 burnt。需要持久锁／关系时另设计完整宿主协议，不能仅存 GUID 后单边赋值。原装扮台点燃和删除会显式解拴，比赛柱删除也有对应处理；本组件没有自动销毁清理钩子。[空存读](../../../raw/current/scripts/components/hitcher.lua#L64-L71) [装扮台保存](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L102-L114) [点燃](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L70-L75) [移除](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L130-L133) [比赛柱](../../../raw/current/scripts/prefabs/yotb_post.lua#L145-L148)

## 典型接入

### 装扮台通过服务端事件切换拴牛交互锁

目标模组 `modmain.lua`，无新资源。调用方在目标台上发 `post:PushEvent("my_mod_hitch_lock", {locked=true})`，传 false 恢复；这里只交付事件接收端，不包含 RPC、调用者授权或完整禁用建筑功能。

```lua
AddPrefabPostInit("beefalo_groomer", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_hitch_lock", function(post, data)
        if GLOBAL.type(data) ~= "table"
            or GLOBAL.type(data.locked) ~= "boolean" then return end
        local hitcher = post.components.hitcher
        if hitcher ~= nil then
            hitcher:Lock(data.locked)
        end
    end)
end)
```

锁定主要影响上述拴／解拴动作候选，不释放当前牛，也不自动关闭 groomer 的换装功能或阻止其他脚本直接调用。若需求是完整禁用装扮台，应继续核验 groomer 消费者，不能以一个锁标签代替。设置不保存，重载默认解除，需持久政策时由调用方重新应用；停用重载不再注册自定义事件。

实例保留原 hitchedfn／unhitchfn、点燃／销毁清理与牛事件；不全局覆盖字段。主动释放的双向验证例见 [[reference/components/hitchable#为原牛提供服务端释放事件保留柱端回调]]。

## 常见陷阱与限制

- **Lock 会阻止任何设置／解除关系**：它只管锁标签，底层方法不读 locked；直接调用方须处理自己的前提。[锁与方法](../../../raw/current/scripts/components/hitcher.lua#L33-L62)
- **柱子删了组件会自动通知牛**：必须核验宿主销毁／点燃路径，组件没有对应钩子；勿抹掉原 onremove 和回调。[原移除](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L130-L133)
- **两个 canbehitched 任意写一个就完成释放**：互调顺序、引用和宿主回调都需要，手动改标志会留下半关系。[释放链](../../../raw/current/scripts/components/hitcher.lua#L46-L58) [生物端](../../../raw/current/scripts/components/hitchable.lua#L31-L39)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；漂移文件 actions.lua、componentactions.lua 对照 R-01/R-04 hunk 地图逐条核验：hitcher 候选（componentactions.lua#L431-L439）与 UNHITCH／HITCH 执行链逐行未变，本批动作变更（禁长按、nomagiccast、拾取放宽、CORRUPTNIGHTMARE）不涉拴柱域；锚点重定位 3 处、结论无修订；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整柱组件、双向建立／解除、回调和候选锁、空存读、装扮台保存／点燃／删除和比赛柱删除。未全面审校 groomer 换装、比赛流程、权限／RPC或关系持久化迁移。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整锁事件例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；候选锁／既有占用、解拴重入、换装交互、火烧／锤毁／删除、重载默认与多模组显式调用待实测。
