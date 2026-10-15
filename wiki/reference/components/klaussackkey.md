---
title: "klaussackkey"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "克劳斯式钥匙标记、真钥匙的世界恢复通知与宿主扩展键；不自行决定开锁结果或消费。"
aliases: ["普通鹿角解锁暴食公园门", "克劳斯真钥匙恢复", "鹿角钥匙", "klaussackkey"]
tags: [component, item, event, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/klaussackkey.lua", "components/klaussacklock.lua", "components/klaussackspawner.lua", "prefabs/deer_antler.lua", "prefabs/klaus_sack.lua", "prefabs/fence.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# klaussackkey

## 快速判断

- 职责与适用：为鹿角类物品加 klaussackkey 标记；SetTrueKey 标记特殊真钥匙并通知世界恢复关联。普通鹿角和真钥匙共用组件，宿主注册名不同。[组件](../../../raw/current/scripts/components/klaussackkey.lua#L1-L43) [注册](../../../raw/current/scripts/prefabs/deer_antler.lua#L52-L67)
- 优先使用：原赃物袋协议配 [[reference/components/klaussacklock]]；原鹿角工厂已处理 pristine 标签和真假分支，扩展原物品无需重挂组件。旧探测杖协议另看 [[reference/components/key]]。[工厂](../../../raw/current/scripts/prefabs/deer_antler.lua#L25-L53)
- 重要副作用：SetTrueKey(true) 首次转换时向 TheWorld 发 ms_restoreklaussackkey；POPULATING 时延迟 0 秒，否则立即发。世界已有袋子时可登记其钥匙引用，并非单纯设置一个无副作用布尔值。[切换](../../../raw/current/scripts/components/klaussackkey.lua#L18-L41) [消费者](../../../raw/current/scripts/components/klaussackspawner.lua#L166-L168)
- 修改边界：服务端行为；组件不验证钥匙能开什么，也不消费物品。truekey 由赃物袋回调消费；门锁的 keytype 是宿主扩展字段，不是此组件默认类型系统。[袋子](../../../raw/current/scripts/prefabs/klaus_sack.lua#L52-L76) [门](../../../raw/current/scripts/prefabs/fence.lua#L465-L482)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetTrueKey(value)` | 真值把 nil／false 转为 true 并通知；重复 true 不重复通知。false 将现有 true 清成 nil、取消待发任务，不向世界发送撤销通知。[实现](../../../raw/current/scripts/components/klaussackkey.lua#L23-L41) |
| `truekey` | 默认 nil（源码 false 行是注释）；由原工厂参数重设，没有本组件 OnSave／OnLoad。原真钥匙是 Prefab `klaussackkey`，不要只改普通鹿角字段就当完成可持久真钥匙。[默认](../../../raw/current/scripts/components/klaussackkey.lua#L1-L8) [工厂](../../../raw/current/scripts/prefabs/deer_antler.lua#L52-L67) |
| `task`／`OnRemoveFromEntity()` | 延迟恢复任务在执行前清引用；移除组件会取消任务并清标签，不做赃物袋引用撤销。[任务与清理](../../../raw/current/scripts/components/klaussackkey.lua#L10-L20) |
| `keytype`（宿主扩展） | 不是本组件初始化字段；暴食门回调比较它与目标 klaussackkeyid。与旧 key.keytype／LOCKTYPE 无关。[实际读取](../../../raw/current/scripts/prefabs/fence.lua#L465-L482) |

世界监听 ms_restoreklaussackkey 后，仅在已登记袋子有效且有 OnDropKey 时调用该函数；袋子已有 key 且此次 klaus 参数为 nil 时会保留旧关联。此事件不是通用钥匙注册表，也不会新建赃物袋。[世界监听](../../../raw/current/scripts/components/klaussackspawner.lua#L166-L196) [袋子登记](../../../raw/current/scripts/prefabs/klaus_sack.lua#L146-L155)

## 典型接入

### 普通一号鹿角兼作原暴食公园门钥匙

目标模组 `modmain.lua`，沿用鹿角与原门资源；仅在场景已有可用 `quagmire_park_gate` 时扩展其原回调。原门注册给定 gate_key，回调匹配后设置解锁网络值并开门；本例不启动暴食模式、不生成门，也不新增自定义锁。[匹配与效果](../../../raw/current/scripts/prefabs/fence.lua#L465-L482) [组件安装](../../../raw/current/scripts/prefabs/fence.lua#L732-L740) [注册](../../../raw/current/scripts/prefabs/fence.lua#L947-L948)

```lua
AddPrefabPostInit("deer_antler1", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.klaussackkey ~= nil then
        inst.components.klaussackkey.keytype = "gate_key"
    end
end)
```

原 deer_antler1 已在 pristine 前加候选标签，服务端已有组件；动作候选不检查 keytype，最终由目标回调比较，成功时 klaussacklock 消费一件。字段值是目标约定的字符串，不是新 LOCKTYPE。[原物品](../../../raw/current/scripts/prefabs/deer_antler.lua#L25-L53) [候选](../../../raw/current/scripts/componentactions.lua#L1470-L1476) [消费](../../../raw/current/scripts/components/klaussacklock.lua#L18-L35)

不调用 SetTrueKey，所以鹿角对原赃物袋仍走普通错误钥匙／召唤分支；该路径会消费鹿角，不能把“能开门”解释成“能直接开袋”。keytype 不由组件保存，加载时由此 PostInit 重设；停用并重载后该兼容消失，已解锁门的宿主保存状态不因此回滚。[袋子分支](../../../raw/current/scripts/prefabs/klaus_sack.lua#L52-L76) [错误钥匙结果](../../../raw/current/scripts/prefabs/klaus_sack.lua#L115-L125) [门保存恢复](../../../raw/current/scripts/prefabs/fence.lua#L498-L508)

## 常见陷阱与限制

- **SetTrueKey(false) 会通知袋子忘掉钥匙**：只取消本组件任务和清字段；实际世界消费者只有恢复登记路径。需要改变所有权／清理关联应另审实际宿主。[方法](../../../raw/current/scripts/components/klaussackkey.lua#L23-L41) [消费者](../../../raw/current/scripts/components/klaussackspawner.lua#L166-L168)
- **组件自动保存 truekey／keytype**：原工厂重建决定真假；本组件没有保存接口。给普通物品临时转真不能据此保证读档不变。[完整组件](../../../raw/current/scripts/components/klaussackkey.lua#L1-L45) [注册差异](../../../raw/current/scripts/prefabs/deer_antler.lua#L30-L36)
- **false 动作就不扣钥匙**：钥匙侧不控制消费，目标可返回 false,reason,true，详见 [[reference/components/klaussacklock#关键接口]]。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：componentactions.lua（R-04）对照 build-26148036 更新报告逐条核验，锚点重定位 0 处、结论修订 0 处：USEITEM.klaussackkey 候选（componentactions.lua:1470-1476）逐行无变化（R-04 变化集中于 spellcaster／corruption／nomagiccast／传送标签，不涉及本组件），其余 sources 文件均未变更；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整钥匙组件、鹿角真假注册、世界恢复监听／袋子关联和两种目标回调、动作标签与消费。未全面审校世界生成、暴食模式资源加载、袋子掉落表或 boss 行为。
- 结构检查：本批格式、来源、链接／锚点、大小和登记检查；完整 PostInit 示例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；暴食场景可用性、骑乘候选、开门／配对门、存读与多钥匙关联待实测。
