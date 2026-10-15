---
title: "domesticatable"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "管理驯化度、服从度、倾向与衰减，区分满进度锁存、正式驯化和牛宿主的骑乘／群体协作。"
aliases: ["刷毛驯化收益翻倍", "牛驯化度", "服从度衰减", "domesticatable"]
tags: [component, creature, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/domesticatable.lua", "prefabs/beefalo.lua", "components/rideable.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# domesticatable

## 快速判断

- 职责与适用：主模拟生物侧管理 domestication、obedience、倾向及任务；原牛先安装 hunger，再配驯化触发函数，不是任意实体只挂组件即可运行。[安装](../../../raw/current/scripts/prefabs/beefalo.lua#L1209-L1216)
- 优先使用：DeltaDomestication／DeltaObedience 保留限幅、事件和任务；满驯化进度先锁存，TryBecomeDomesticated 才按锁存正式完成。[增量](../../../raw/current/scripts/components/domesticatable.lua#L159-L183)
- 重要副作用：达到 1 不直接完成驯化；SetDomesticated 只改标志／标签并 Validate，BecomeDomesticated 才发 domesticated 事件。直接赋值或只设标志会漏宿主协作。[锁存与完成](../../../raw/current/scripts/components/domesticatable.lua#L71-L92) [标志](../../../raw/current/scripts/components/domesticatable.lua#L199-L205)
- 修改边界：暂停驯化衰减不暂停服从衰减，也不阻止触发条件成立时增长；牛宿主会按盐碱地／睡眠重新设置暂停开关。[周期](../../../raw/current/scripts/components/domesticatable.lua#L136-L156) [宿主](../../../raw/current/scripts/prefabs/beefalo.lua#L922-L924)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `DeltaDomestication(delta, doer)` | 夹在 0～1；正增量可按 doer 或当前骑手技能再乘倍率，变化发 domesticationdelta，再检查状态／任务。返回不代表数值变化量。[内部结算](../../../raw/current/scripts/components/domesticatable.lua#L110-L134) [公开入口](../../../raw/current/scripts/components/domesticatable.lua#L165-L170) |
| `DeltaObedience(delta)`／`SetMinObedience(min)` | 服从度按 minobedience／maxobedience 限幅；变化发 obediencedelta。设下限可立即补到下限并启动任务，不是只配置数字；宿主倾向切换会重设下限。[服从](../../../raw/current/scripts/components/domesticatable.lua#L101-L108) [下限](../../../raw/current/scripts/components/domesticatable.lua#L208-L214) [宿主](../../../raw/current/scripts/prefabs/beefalo.lua#L593-L603) |
| `CheckForChanges()`／`TryBecomeDomesticated()` | 未驯化且进度>=1 时锁存，<0.95 清锁；饥饿为零且驯化度为零可发 goneferal、清倾向并解除驯化。Try 只在锁存真时完成。[检查](../../../raw/current/scripts/components/domesticatable.lua#L71-L92) [尝试](../../../raw/current/scripts/components/domesticatable.lua#L180-L184) |
| `DeltaTendency(name, delta)` | 累加倾向值，不自动选出品种／调用变化检查；牛的 SetTendency 才消费倾向并设置伤害、速度和服从下限。[累加](../../../raw/current/scripts/components/domesticatable.lua#L172-L174) [宿主选型](../../../raw/current/scripts/prefabs/beefalo.lua#L565-L603) |
| `SetDomesticationTrigger(fn)` | fn(inst) 决定周期增长分支；需实际配置，周期直接调用它。Validate 还直接使用 hunger，不把这些要求误写成纯可选关系。[配置](../../../raw/current/scripts/components/domesticatable.lua#L48-L69) [周期](../../../raw/current/scripts/components/domesticatable.lua#L136-L156) |
| `PauseDomesticationDecay(pause)` | 单个布尔开关，无来源堆栈；暂停倾向／驯化损失，服从仍衰减。触发条件成立时仍增长，未触发时平移 lastdomesticationgain。[实现](../../../raw/current/scripts/components/domesticatable.lua#L136-L177) |

任务每 10 秒运行、首次延迟 0；自然增长调用局部 DoDeltaDomestication，绕过同名公开方法包装。Validate 在服从已到下限、饥饿和驯化都为零时取消任务；CancelTask 不代表持久禁用，之后入口可再启动。[更新](../../../raw/current/scripts/components/domesticatable.lua#L136-L156) [调度](../../../raw/current/scripts/components/domesticatable.lua#L186-L197)

保存含进度、倾向、锁存、驯化标志、服从／下限和距上次增长时间；读档恢复并重启检查，暂停布尔值不单独保存。骑具记录嵌入本组件数据，由 OnLoad 调 rideable:OnLoadDomesticatable 保证先后顺序，不能另复制一份独立鞍存档。[保存／恢复](../../../raw/current/scripts/components/domesticatable.lua#L216-L248) [骑具恢复](../../../raw/current/scripts/components/rideable.lua#L217-L233)

## 典型接入

### 刷毛等公开接口产生的正向驯化收益翻倍

目标模组 `modmain.lua`，无需资源；只包装原 beefalo 的公开增量入口，保留 doer、技能倍率、事件与限幅。原刷毛满足 numprizes>0 才调用该入口；内部每 10 秒自然增长不经过包装。[刷毛](../../../raw/current/scripts/prefabs/beefalo.lua#L482-L490) [内部路径](../../../raw/current/scripts/components/domesticatable.lua#L136-L156)

```lua
AddPrefabPostInit("beefalo", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local component = inst.components.domesticatable
    if component == nil then return end
    local original = component.DeltaDomestication
    component.DeltaDomestication = function(self, delta, doer)
        return original(self, delta > 0 and delta * 2 or delta, doer)
    end
end)
```

`2` 为公开正增量倍率，其他模组经该方法给的正收益也受影响，负面惩罚保留；不是所有驯化速度翻倍。存档恢复直接赋进度不经过该包装，重载不重复放大旧进度；停用恢复后续默认结算，已有收益不回滚。[恢复](../../../raw/current/scripts/components/domesticatable.lua#L232-L248)

达到上限仍保留原完成时机：正常未饱喂食调用 TryBecomeDomesticated；正式状态事件及牛后续处理关联群体启停、倾向与外观，不能只加一行 SetDomesticated(true) 声称完整驯化。[喂食](../../../raw/current/scripts/prefabs/beefalo.lua#L746-L758) [宿主处理](../../../raw/current/scripts/prefabs/beefalo.lua#L530-L562) 群体机制见 [[reference/components/herdmember]]。

## 常见陷阱与限制

- **进度、服从、已驯化是同一个值**：三个状态分别管理；进度满还可能等待锁存消费，服从会衰减并影响骑乘资格，不能互相替代。[锁存](../../../raw/current/scripts/components/domesticatable.lua#L71-L92)
- **只暂停一次就永久不掉驯化／服从**：这是单布尔值且宿主可重写，服从仍衰减；新增长、饥饿及技能消费者也独立存在。[周期](../../../raw/current/scripts/components/domesticatable.lua#L136-L156)
- **移除组件就自动完成回野生和卸骑具**：卸载只取消任务、去标签；没有发送全部宿主状态事件，也不代办骑手／鞍清理。[卸载](../../../raw/current/scripts/components/domesticatable.lua#L42-L46)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖增量／技能、锁存／完成、衰减／任务、倾向、保存／骑具顺序及刷毛／喂食／群体宿主。未全面审校技能树、骑乘、全部倾向资源或存档迁移。
- 结构检查：本批格式、来源、链接／行号、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；刷毛门槛、技能叠加、锁存完成、饥饿／衰减、存读／骑具与其他模组包装待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。