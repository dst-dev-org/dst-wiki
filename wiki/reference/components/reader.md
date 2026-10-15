---
title: "reader"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "人物正式阅读／翻阅分派、精神罚值倍率与阅读后回调；角色标签、物品效果和 SG 各有职责。"
aliases: ["麦斯威尔阅读精神倍率减半", "人物读书能力", "翻阅与正式阅读", "reader"]
tags: [component, callback, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/reader.lua", "components/book.lua", "actions.lua", "componentactions.lua", "prefabs/waxwell.lua", "prefabs/wurt.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# reader

## 快速判断

- 职责与适用：人物侧 Read(book) 选择 book:OnRead 或 OnPeruse，效果和次数消费由 [[reference/components/book]] 负责；不是给任意物品提供阅读 UI。[分派](../../../raw/current/scripts/components/reader.lua#L46-L60)
- 优先使用：SetSanityPenaltyMultiplier(mult) 修改正式阅读精神结算；SetAspiringBookworm(true) 改走翻阅；SetOnReadFn 仅用于正式阅读成功后的额外回调。[接口](../../../raw/current/scripts/components/reader.lua#L25-L60)
- 重要副作用：翻阅直接返回 OnPeruse，不执行 reader.onread；正式阅读先完成书端效果／消费／精神，再调用 onread(player,book)，回调返回值不改 success。[顺序](../../../raw/current/scripts/components/reader.lua#L46-L60) [书端](../../../raw/current/scripts/components/book.lua#L61-L80)
- 修改边界：构造添加 reader 标签，翻阅属性维护 aspiring_bookworm 标签；客户端 SG 看标签，服务端看组件。只加标签不提供服务端能力，只改一端也不能保证预测匹配。[属性](../../../raw/current/scripts/components/reader.lua#L1-L23) [服务端路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1034-L1039) [客户端路由](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L363-L368)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `Read(book)` | 假定 book 为有 components 的实体；无 book 组件则返回 nil。自身不验证持有、距离、燃烧、存活或冷却，不能当权限入口。[方法](../../../raw/current/scripts/components/reader.lua#L46-L60) [候选](../../../raw/current/scripts/componentactions.lua#L228-L232) |
| `SetAspiringBookworm(bool)`／`IsAspiringBookworm()` | 真值选翻阅、同步对应标签；不是“强化正式读书”的被动加成。原 Wurt 在共同初始化预放标签，服务端设置该模式。[方法](../../../raw/current/scripts/components/reader.lua#L25-L31) [共同端](../../../raw/current/scripts/prefabs/wurt.lua#L428-L432) [服务端](../../../raw/current/scripts/prefabs/wurt.lua#L633-L642) |
| `SetSanityPenaltyMultiplier(mult)`／`GetSanityPenaltyMultiplier()` | 未设置回落 1，0 在 Lua 中是真值，因此可保留 0；书端将其乘 read_sanity，翻阅不读此倍率。不是 modifiers 集合。[字段](../../../raw/current/scripts/components/reader.lua#L33-L39) [正式／翻阅](../../../raw/current/scripts/components/book.lua#L45-L59) |
| `SetOnReadFn(fn)` | 只存一个回调；正式阅读成功后 fn(player,book)。原麦斯威尔保留疯狂状态下的暗影生物逻辑；覆盖 setter 会替换它。[回调](../../../raw/current/scripts/components/reader.lua#L42-L60) [原效果](../../../raw/current/scripts/prefabs/waxwell.lua#L159-L171) |
| `OnRemoveFromEntity()` | 删除 reader 与 aspiring_bookworm 标签，无本组件存读；重建和默认参数来自人物初始化。[清理](../../../raw/current/scripts/components/reader.lua#L20-L23) |

## 典型接入

### 麦斯威尔保留原读书逻辑，将正式阅读倍率再乘半

目标模组 `modmain.lua`，不新增资源或读书资格。原 waxwell 共同端已有 reader 标签，服务端挂组件并配置角色倍率／回调；只改这个现成实例的数值。[原安装](../../../raw/current/scripts/prefabs/waxwell.lua#L269-L287)

```lua
AddPrefabPostInit("waxwell", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local reader = inst.components.reader
    if reader ~= nil then
        reader:SetSanityPenaltyMultiplier(reader:GetSanityPenaltyMultiplier() * 0.5)
    end
end)
```

0.5 乘当前倍率，不是把角色最终倍率直接设为 0.5；会与书本 read_sanity 的修改共同参与结算。保留原 OnReadFn，不改变失败、翻阅或 SG。值不由组件存档，构造时由原角色和本例重建；停用并重载恢复原角色设置。其他模组后续 setter 可覆盖结果，不称为来源可撤销的独立倍率系统。[结算入口](../../../raw/current/scripts/components/book.lua#L55-L59) [原配置](../../../raw/current/scripts/prefabs/waxwell.lua#L285-L287)

## 常见陷阱与限制

- **翻阅也会触发 onread 奖励**：翻阅分支直接返回，且 peruse_sanity 不乘 reader 倍率；需要翻阅效果应在书的对应回调处理。[分支](../../../raw/current/scripts/components/reader.lua#L46-L60) [精神](../../../raw/current/scripts/components/book.lua#L45-L59)
- **onread 返回 false 能阻止扣书**：书端流程已经执行，人物后回调返回值被忽略；拒绝条件放在实际执行前或书效果回调中，不能事后当事务回滚。[顺序](../../../raw/current/scripts/components/reader.lua#L51-L57)
- **给新角色只加 reader 标签就够了**：READ 服务端还检查组件，客户端／服务端 SG 又分别用标签／组件判断翻阅；新角色还需完整资源、动画与能力接入。本例不交付全角色读书补丁。[动作](../../../raw/current/scripts/actions.lua#L1328-L1339) [路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1034-L1039)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件（actions.lua=R-01、componentactions.lua=R-04、SGwilson.lua=SB-17、SGwilson_client.lua 无专条目、waxwell.lua=P-133、wurt.lua 无专条目）逐条核验，锚点重定位 4 处、结论修订 0 处——READ／书本候选／两端路由原文未变；P-133 改动（shadow_item dapperness、master_postinit 增 stalkerinspectable）不在本页阅读相关锚点域，wurt.lua 内容零变化（仅行尾）；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整 reader、书端结算、READ／地面候选、麦斯威尔原设置与后回调、Wurt 双端翻阅配置和双方 READ 状态路由。未完整审校全部读书 SG／预测、角色重选、特殊书或 RPC 权限。
- 结构检查：本批格式、来源、源码引用／行号、大小与登记通过；完整角色倍率例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；角色创建／重载、正式阅读／失败／翻阅、疯狂后回调及其他模组覆盖待实测。
