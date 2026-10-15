---
title: "rideable"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "管理坐骑侧鞍、骑手、服从测试和被骑事件，区分上鞍资格、上骑动作与完整骑手状态转换。"
aliases: ["上牛需要七成服从", "上鞍资格", "坐骑骑手", "rideable"]
tags: [component, creature, equipment, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/rideable.lua", "components/rider.lua", "components/domesticatable.lua", "prefabs/beefalo.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# rideable

## 快速判断

- 职责与适用：坐骑侧保存 saddle／rider、上鞍开关、可骑标签和资格测试；原牛配置服从阈值与自定义骑手检查。[构造](../../../raw/current/scripts/components/rideable.lua#L52-L82) [宿主](../../../raw/current/scripts/prefabs/beefalo.lua#L1197-L1200)
- 优先使用：SetRequiredObedience／SetCustomRiderTest 配资格，SetSaddle 管鞍生命周期；上骑须走动作和骑手转换，不能只 SetRider。[资格](../../../raw/current/scripts/components/rideable.lua#L94-L111) [动作](../../../raw/current/scripts/actions.lua#L4141-L4164)
- 重要副作用：SetSaddle 移除旧鞍、发事件并抛出，再尝试接收新鞍；SetRider 启停每 6 秒的 beingridden 事件和攻击转发。[鞍](../../../raw/current/scripts/components/rideable.lua#L113-L163) [骑手](../../../raw/current/scripts/components/rideable.lua#L174-L190)
- 修改边界：上鞍与上骑阈值不同；TestObedience 不查全部动作条件，Buck 仅发事件、不保证已下骑。[牛上鞍门禁](../../../raw/current/scripts/prefabs/beefalo.lua#L661-L669) [甩落请求](../../../raw/current/scripts/components/rideable.lua#L201-L205)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetSaddleable(bool)`／`canride` | saddleable 控制能否接收新鞍；canride 随鞍接收／移除改变并联动 rideable 标签。改 saddleable 不自动卸旧鞍或设置 canride。[接收](../../../raw/current/scripts/components/rideable.lua#L136-L171) |
| `TestObedience()`／`TestRider(player)` | 未配置阈值或无 domesticatable 时，服从测试放行；自定义骑手测试独立。rider:Mount 要求两项均通过。构造虽有 allowed_riders 空表，本次入口没有读取它，不将其当已接通的白名单 API。[测试](../../../raw/current/scripts/components/rideable.lua#L94-L111) [骑手入口](../../../raw/current/scripts/components/rider.lua#L67-L77) |
| `SetSaddle(doer, new)` | 旧鞍 ReturnToScene、unequipped、FlingItem；doer=nil 时落地触发 discardedcb。新鞍符合 saddleable 才挂为子实体、隐藏、发 saddlechanged／equipped 并设置符号；失败会抛新鞍并可 SuggestTarget。需实际 saddler、宿主 AnimState／lootdropper 等协作，不是裸数据 setter。[实现](../../../raw/current/scripts/components/rideable.lua#L113-L163) |
| `SetRider(player)` | 维护坐骑侧引用、攻击监听、任务与 riderchanged(oldrider,newrider)，不代办人物动画／碰撞／父子／replica 全链路。[实现](../../../raw/current/scripts/components/rideable.lua#L174-L190) |
| `GetRider()`／`IsBeingRidden()`／`TimeSinceLastRide()` | 分别读引用、判断非 nil、返回秒数；被骑 tick 更新 lastridetime，离骑时也更新。[tick](../../../raw/current/scripts/components/rideable.lua#L17-L34) [查询](../../../raw/current/scripts/components/rideable.lua#L90-L92) [转换](../../../raw/current/scripts/components/rideable.lua#L174-L198) |
| `Buck(gentle)` | 向有 rider 组件的骑手发 bucked；实际 SG 决定响应。牛自己的甩落定时甚至会重排请求，不能把 Buck 返回当下骑成功。[方法](../../../raw/current/scripts/components/rideable.lua#L201-L205) [宿主重试](../../../raw/current/scripts/prefabs/beefalo.lua#L655-L658) |

保存由 domesticatable 嵌套调用 OnSaveDomesticatable／OnLoadDomesticatable，保存鞍记录和距上次骑乘秒数，再通过 SetSaddle 恢复。SetShouldSave 标志另给骑手侧保存策略使用，不等于 rideable 自有普通 OnSave，也不证明任意无驯化组件坐骑可自动保存鞍。[嵌套记录](../../../raw/current/scripts/components/rideable.lua#L207-L233) [调用方](../../../raw/current/scripts/components/domesticatable.lua#L216-L248) 驯化条件见 [[reference/components/domesticatable]]。

死亡监听会卸鞍；组件卸载只停止 tick、去部分标签，不负责完整人物下骑／鞍回收。不要移除组件来代替正常下骑和卸鞍流程。[死亡监听](../../../raw/current/scripts/components/rideable.lua#L69-L88)

## 典型接入

### 原牛的服从测试阈值提高至 0.7

目标模组 `modmain.lua`，无资源；只改原牛的 requiredobedience，保留自定义骑手测试和所有动作门禁。[宿主设置](../../../raw/current/scripts/prefabs/beefalo.lua#L1197-L1200) [测试消费](../../../raw/current/scripts/components/rider.lua#L67-L77)

```lua
AddPrefabPostInit("beefalo", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local rideable = inst.components.rideable
    if rideable ~= nil then
        rideable:SetRequiredObedience(0.7)
    end
end)
```

`0.7` 是服从比例，不是驯化度；不直接提高上鞍门槛，不主动甩落当前骑手。战斗／死亡／冰冻／被拴／占用等动作限制仍在，不能将 TestObedience=true 当可立即骑乘。[动作条件](../../../raw/current/scripts/actions.lua#L4141-L4164) [上鞍门槛](../../../raw/current/scripts/prefabs/beefalo.lua#L661-L669)

每次构造重新应用；阈值没有在本组件的鞍记录内保存，停用重载恢复原配置，当前骑手或存档鞍不由此回滚。原牛 PotentialRiderTest 对物品型 leader 的实际持有者另有检查，本例不替换该回调。[原测试](../../../raw/current/scripts/prefabs/beefalo.lua#L822-L830)

## 常见陷阱与限制

- **SetRider 即完成骑乘／Buck 即完成下骑**：这两个入口只处理链条的一部分，完整转换还需骑手与 SG；不要手工改引用制造半骑乘状态。[引用转换](../../../raw/current/scripts/components/rideable.lua#L174-L205)
- **SetSaddle 可以直接传背包里的实体当普通赋值**：SADDLE 动作先从 inventory 移除再交接，组件本身不替调用方解除原库存所有权。[动作交接](../../../raw/current/scripts/actions.lua#L4174-L4186)
- **SetSaddle(nil,nil) 与玩家卸鞍等价**：nil doer 走丢弃后的落地回调，可能影响鞍耐久；用实际调用上下文区分。[丢弃](../../../raw/current/scripts/components/rideable.lua#L44-L49) [移除旧鞍](../../../raw/current/scripts/components/rideable.lua#L113-L134)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 中漂移文件 actions.lua、rider.lua（更新报告 R-01、C-34）——MOUNT／SADDLE 动作与 rider.lua Mount 门禁段未变；rider.lua 本轮仅 Mount 伤害转交新增装备攻击者豁免（C-34），本页无该结论；锚点重定位 3 处；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖资格、鞍交接、骑手事件／tick、Buck、嵌套保存及原牛／MOUNT/SADDLE 消费者。未全面审校骑手 SG、所有特殊坐骑、皮肤或技能。
- 结构检查：本批格式、来源、链接／行号、大小和登记通过；完整阈值例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；服从变化、物品持有者限制、自定义骑手、鞍替换／丢弃、死亡／下骑、存读和多模组配置待实测。
