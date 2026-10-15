---
title: "skinner_beefalo"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "保存牛五部位皮肤、发出服装变更并委托宿主刷新；授权、换装状态和恢复不是组件单独完成。"
aliases: ["清除牛装扮", "牛皮肤部位", "skinner_beefalo"]
tags: [component, creature, callback, event, network-sync, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/skinner_beefalo.lua", "prefabs/beefalo.lua", "prefabs/beefalo_groomer.lua", "prefabs/beef_bell.lua", "skinsutils.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# skinner_beefalo

## 快速判断

- 职责与适用：牛侧保存 beef_body／beef_horn／beef_head／beef_feet／beef_tail 五类皮肤名，初始空字符串；不是玩家 skinner、装备槽或任意贴图加载器。[构造](../../../raw/current/scripts/components/skinner_beefalo.lua#L1-L6)
- 优先使用：GetClothing 取副本，SetClothing 设置有效已登记部位，ClearAllClothing 清除并刷新；需要原牛正常脱装流程时调用宿主 UnSkin，界面协作见 [[reference/components/groomer]]。[接口](../../../raw/current/scripts/components/skinner_beefalo.lua#L89-L105) [原牛入口](../../../raw/current/scripts/prefabs/beefalo.lua#L406-L428)
- 重要副作用：SetClothing／ClearAllClothing 发 onclothingchanged 并刷新本牛 ApplyBuildOverrides；牛监听事件更新对应 net_string。直接改 clothing 表会绕过这些步骤。[设置](../../../raw/current/scripts/components/skinner_beefalo.lua#L89-L96) [清除](../../../raw/current/scripts/components/skinner_beefalo.lua#L134-L146) [监听](../../../raw/current/scripts/prefabs/beefalo.lua#L987-L991)
- 修改边界：有效名称检查只判断皮肤表登记且非默认，不验证用户拥有权。ApplyTargetSkins 要玩家并调用引擎 AssignItemSkins，不能把 SetClothing 当皮肤授权或新美术资源注册。[有效性](../../../raw/current/scripts/skinsutils.lua#L1492-L1494) [应用](../../../raw/current/scripts/components/skinner_beefalo.lua#L148-L161)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetClothing(name)` | 仅有效名称才据 BEEFALO_CLOTHING[name].type 写入，发事件并刷新；传空字符串不实现“清除此部位”。[设置](../../../raw/current/scripts/components/skinner_beefalo.lua#L89-L96) |
| `GetClothing()` | 返回五字段新表；修改返回表不改变组件本体，正式应用用接口。[副本](../../../raw/current/scripts/components/skinner_beefalo.lua#L98-L105) |
| `IsClothingDifferent(newclothes)` | 缺项按空字符串比较，双向检查键；用于完整目标套装差异，不是局部补丁比较。原装扮台用它判断是否开始换装。[比较](../../../raw/current/scripts/components/skinner_beefalo.lua#L108-L122) [消费](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L157-L162) |
| `ClearClothing(type)` | 指定键置空并发事件，**没有**调用 ApplyBuildOverrides；调用者处理刷新及合法键，勿传自造类型，牛监听会访问 inst.skins[type]。[方法](../../../raw/current/scripts/components/skinner_beefalo.lua#L143-L146) [消费者](../../../raw/current/scripts/prefabs/beefalo.lua#L987-L991) |
| `ClearAllClothing()` | 遍历当前表逐部位置空／发事件，最后刷新一次；没有运行原牛 UnSkin 的状态门禁与动画。[实现](../../../raw/current/scripts/components/skinner_beefalo.lua#L134-L141) |
| `ApplyTargetSkins(skins,player)` | assert 玩家，使用 userid 调 AssignItemSkins，先清再逐部位 SetClothing。不是安全的无玩家／任意名称入口；装扮台还有自身应用回调，不能假设必经本方法。[组件](../../../raw/current/scripts/components/skinner_beefalo.lua#L148-L161) [设施回调](../../../raw/current/scripts/prefabs/beefalo_groomer.lua#L164-L175) |
| `OnSave()`／`reloadclothing(clothing)` | 前者保存当前表；后者直接接管传入表、逐键发事件再刷新。组件没有 OnLoad，已核验牛铃保存服装和实体记录，恢复时显式调 reloadclothing；不推广为全部宿主自动恢复。[组件](../../../raw/current/scripts/components/skinner_beefalo.lua#L163-L180) [牛铃路径](../../../raw/current/scripts/prefabs/beef_bell.lua#L246-L269) |

动画辅助 SetBeefaloSkinsOnAnim 先清本牛符号覆盖、恢复隐藏符号，再按尾／身／脚／头／角顺序应用登记配置；骑手使用带牛 GUID 的 OverrideItemSkinSymbol 分支。牛宿主 ApplyBuildOverrides 桥接基础 build、衣服与倾向，不要自己重复一份符号表。脸部辅助另有 BRANCH==dev 和指定 build 门禁，不作为当前任意脸皮肤能力。[动画函数](../../../raw/current/scripts/components/skinner_beefalo.lua#L47-L87) [脸部门禁](../../../raw/current/scripts/components/skinner_beefalo.lua#L18-L27) [宿主桥接](../../../raw/current/scripts/prefabs/beefalo.lua#L259-L288)

## 典型接入

### 为原牛提供遵循原状态限制的服务端脱装事件

目标模组 `modmain.lua`，无新资源；调用方在服务端对目标牛发 `beef:PushEvent("my_mod_clear_beefalo_clothing")`。这里只交付接收端，不包含 UI、RPC、发起者身份／权限或远程管理。

```lua
AddPrefabPostInit("beefalo", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_clear_beefalo_clothing", function(beef)
        if beef.components.skinner_beefalo ~= nil
            and beef.UnSkin ~= nil then
            beef:UnSkin()
        end
    end)
end)
```

原 UnSkin 在 nointerrupt 状态不处理；睡眠时直接清除并按需出效果，清醒且有衣服时进入 skin_change 再清。事件发出不保证立即脱装或返回成功，不自行循环重试／硬切状态。保留原服装网络字段和事件监听；本例只影响实际收到事件的原牛，不禁用以后重新装扮。停用重载移除接收端，不恢复已清皮肤。[原行为](../../../raw/current/scripts/prefabs/beefalo.lua#L406-L428) [构造与监听](../../../raw/current/scripts/prefabs/beefalo.lua#L1268-L1269)

## 常见陷阱与限制

- **ClearClothing 会像 ClearAllClothing 一样刷新全部显示**：前者只置空和发事件；需要完整脱装优先用宿主入口，局部修改另核验显示／骑手协作。[两个方法](../../../raw/current/scripts/components/skinner_beefalo.lua#L134-L146)
- **HideAllClothing 是持久脱装**：它只清传入 anim_state 的覆盖，不改 clothing、事件或存档；后续宿主刷新可重新应用。[隐藏](../../../raw/current/scripts/components/skinner_beefalo.lua#L124-L132)
- **reloadclothing 可接收任意外部字典**：它直接替换表并发送所有键，牛监听对键有前提；仅用于可信、兼容的恢复数据，不作通用用户输入接口。[恢复](../../../raw/current/scripts/components/skinner_beefalo.lua#L167-L180) [键消费](../../../raw/current/scripts/prefabs/beefalo.lua#L987-L991)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 1 处（skinsutils#L1472-L1474→L1492-L1494，区间文本比对一致）；结论修订：无（有效性判定两版一致；skinsutils 其余改动在 819 插块与 2029 单行，不在引用区间）。
- 依据：变更清单命中 skinsutils.lua。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、五部位／差异／事件／刷新／辅助函数、有效性定义、牛构造与 UnSkin、装扮台回调和牛铃恢复。未全面审校一般世界存档的皮肤特殊恢复、引擎所有权校验、全部皮肤数据／骑手渲染与客户端时序。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整事件例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；清醒／睡眠／nointerrupt、客户端及骑手显示、再装扮、牛铃存读、事件授权与多模组交互待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。