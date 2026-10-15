---
title: "cursable"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "玩家侧猴子诅咒计数、强制收纳、解除与死亡转移；与诅咒物品追踪及变身工具协作。"
aliases: ["服务端净化一枚猴子代币", "玩家猴子诅咒", "诅咒物品强制收纳", "cursable"]
tags: [component, item, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/cursable.lua", "components/curseditem.lua", "curse_monkey_util.lua", "prefabs/cursed_monkey_token.lua", "prefabs/player_common.lua", "stategraphs/SGwilson.lua", "stategraphs/SGmonkeyqueen.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# cursable

## 快速判断

- 职责与适用：玩家记录 curses 并处理诅咒物品收纳／解除；MONKEY 效果由 curse_monkey_util 触发，原玩家公共工厂已挂组件。不是任意 debuff 的通用计数器。[定义](../../../raw/current/scripts/components/cursable.lua#L3-L23) [宿主](../../../raw/current/scripts/prefabs/player_common.lua#L2848-L2853)
- 优先使用：物品施咒链参看 [[reference/components/curseditem]]；解除用 RemoveCurse，连同代币消费／效果更新处理，不只写 curses.MONKEY=0。原猴王解除也走该入口。[解除](../../../raw/current/scripts/components/cursable.lua#L26-L53) [原调用](../../../raw/current/scripts/stategraphs/SGmonkeyqueen.lua#L159-L164)
- 重要副作用：ForceOntoOwner 可挤掉主栏物品、移出原主人并 GiveItem；RemoveCurse(dropitems=true) 还会生成地面代币。解除物品和改变外观／变身并非无副作用赋值。[强制收纳](../../../raw/current/scripts/components/cursable.lua#L85-L120) [落物](../../../raw/current/scripts/components/cursable.lua#L26-L53)
- 修改边界：服务端玩家宿主，依赖 inventory、skinner、SG／talker 等实际协作。ApplyCurse 不自检资格，也不去重；其看似可选的 item 参数实际在末尾被直接使用，不能传 nil。[施加](../../../raw/current/scripts/components/cursable.lua#L8-L23) [效果工具](../../../raw/current/scripts/curse_monkey_util.lua#L81)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `ApplyCurse(item,curse?)` | 实际应传带 curseditem 的物品；从物品读 curse、绑定 cursed_target，按整堆数量增加计数，MONKEY 调 docurse，最后加 applied_curse。给了物品时传入 curse 会被覆盖，无显式成功结果。[方法](../../../raw/current/scripts/components/cursable.lua#L8-L23) |
| `RemoveCurse(curse,num,dropitems?)` | MONKEY 按 monkey_token 标签逐件查找／消费，dropitems 真时先生成新地面同名物；随后计数按**请求 num**减到最小零，再 uncurse。未找到足量物品也减请求数，调用者要核对实际库存／计数。[方法](../../../raw/current/scripts/components/cursable.lua#L26-L53) |
| `IsCursable(item)` | ghost 标签拒绝返回 nil；spawnprotectionbuff 拒绝 false；否则主栏未满、同名堆叠有空或可挤掉直接持有且非 nosteal／非 activeitem 时可 true。不是不丢东西的收纳保证。[门禁](../../../raw/current/scripts/components/cursable.lua#L57-L81) |
| `ForceOntoOwner(item)` | 先查宿主有效／非死亡；满栏先找同名未满堆，再尝试 DropItem 腾位，移出物品旧主人、清 prevcontainer／prevslot 并 GiveItem。不复查完整 IsCursable 门禁。[方法](../../../raw/current/scripts/components/cursable.lua#L85-L120) |
| `Died()` | 对库存 curseditem 按堆数 RemoveCurse(type,num,true)，不是自己注册 death 监听；SGwilson 的特定死亡分支调用，不能据此假定所有复活／尸体路径都相同。[方法](../../../raw/current/scripts/components/cursable.lua#L124-L129) [宿主](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3900-L3905) |
| `curses`／`OnSave()`／`OnLoad()` | 计数表初始空；保存字段／加载恢复逻辑都被注释。实际拾取链会施咒，不能把空保存视作永远无诅咒，也不能据此证明所有换人／重载流程已覆盖。[存读](../../../raw/current/scripts/components/cursable.lua#L133-L154) [拾取](../../../raw/current/scripts/components/curseditem.lua#L166-L178) |

### 变身与恢复不是同步布尔切换

MONKEY 的 docurse 可加局部猴化外观或排周期 trytransform；busy／dead／nomorph 等状态会延迟，达到阈值不等于已经变身。uncurse 刷新外观／事件，已是 wonkey 且计数归零时也通过重试任务恢复；本页不提供完整换人／皮肤保存方案。[重试条件](../../../raw/current/scripts/curse_monkey_util.lua#L3-L23) [解除工具](../../../raw/current/scripts/curse_monkey_util.lua#L33) [施咒工具](../../../raw/current/scripts/curse_monkey_util.lua#L81)

## 典型接入

### 玩家收到服务端净化事件时消费一枚代币并减一层猴子诅咒

目标模组 `modmain.lua`，复用原玩家组件／效果／资源；这是服务端已有奖励或净化逻辑的入口，不新增按钮、RPC 或免费获取奖励的客户端通道。先确认计数与原类型代币均存在，再调用原 RemoveCurse，不直接删实体／改皮肤。[原解除](../../../raw/current/scripts/components/cursable.lua#L26-L53) [原代币](../../../raw/current/scripts/prefabs/cursed_monkey_token.lua#L91-L104)

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_cleanse_one_monkey", function(player)
        local curse = player.components.cursable
        local inventory = player.components.inventory
        if curse == nil or inventory == nil or player:HasTag("playerghost")
            or player.components.health == nil or player.components.health:IsDead()
            or (curse.curses.MONKEY or 0) < 1 then return end
        local token = inventory:FindItem(function(item)
            return item:HasTag("monkey_token") and item.components.curseditem ~= nil
                and item.components.curseditem.curse == "MONKEY"
        end)
        if token ~= nil then curse:RemoveCurse("MONKEY", 1, false) end
    end)
end)
```

服务端对真实玩家发 `player:PushEvent("my_mod_cleanse_one_monkey")`；每次消费一件并减一计数，false 表示不生成可再次追人的地面代币，不是禁止效果刷新。对有新增 monkey_token 物品的模组，须保证原 RemoveCurse 标签查找与该类型约定一致，不把此例当所有自定义诅咒兼容层。[消费与计数](../../../raw/current/scripts/components/cursable.lua#L26-L53)

不新增存档字段或接管变身任务；原工具仍决定何时变回。停用并重载只取消该额外入口，不返还已经消费的代币、不回滚净化。若实际库存与计数已不一致，应先定向排查拾取／堆叠／保存链，不继续盲减计数。

## 常见陷阱与限制

- **ApplyCurse(nil,"MONKEY")／重复调用同一物品**：末尾直接 item:AddTag，nil 会失败；方法不查 applied_curse，重复会重复累计。原 Given 才承担首次判定。[实现](../../../raw/current/scripts/components/cursable.lua#L8-L23) [原门禁](../../../raw/current/scripts/components/curseditem.lua#L166-L178)
- **RemoveCurse 精确按实际删掉数量减计数**：它无论找到多少都按请求减，并触发效果；先核对数量，不用它作为容错扣账接口。[方法](../../../raw/current/scripts/components/cursable.lua#L26-L53)
- **删代币／清 curses 字段就完整恢复人物**：外观、标签、变身重试与物品归属各有协作；原 RemoveCurse／uncurse 是实际入口，不把单字段当整链。[解除](../../../raw/current/scripts/components/cursable.lua#L26-L53) [工具](../../../raw/current/scripts/curse_monkey_util.lua#L33)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 2 处（player_common cursable 挂载段 -14、SGwilson 死亡分支 ChangeFromMonkey +65）；无结论修订，仅锚点迁移——cursable.lua、curseditem.lua、curse_monkey_util.lua、cursed_monkey_token.lua、SGmonkeyqueen.lua 全部未变；player_common 本版删除仅涉虫王消化音效（P-83），SGwilson 本版改动域（施法／击退／受困／传送）不涉死亡分支内容。依据：player-hubs-hunk-map 41 断点偏移表＋变更清单命中 prefabs/player_common.lua、stategraphs/SGwilson.lua。
- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖完整 cursable／物品拾取绑定、猴化工具、原代币宿主、玩家挂载、SGwilson 死亡调用与猴王解除入口。未完整审校变身 SG／角色重选保存、全部库存差异、堆叠并发或自定义诅咒。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整单枚净化例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；单枚／整堆、满库存／手持、局部猴化／已变身、死亡／换人／重载与其他模组计数差异待实测。
