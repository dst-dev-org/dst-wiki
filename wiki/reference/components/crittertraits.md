---
title: "crittertraits"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "领养小宠物的四类性格分数、周期选主导与锁定；行为积分由宠物／主人事件及 SG 协作。"
aliases: ["小猫正向玩耍性格收益翻倍", "宠物主导性格", "crittertraits"]
tags: [component, creature, event, callback, lifecycle, data]
dst_build: "steam-buildid:26230496"
sources: ["components/crittertraits.lua", "prefabs/critters.lua", "stategraphs/SGcritter_common.lua", "actions.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# crittertraits

## 快速判断

- 职责与适用：原领养小宠物记录 COMBAT／WELLFED／PLAYFUL／CRAFTY 分数，以计时器衰减和选择主导性格；标签／实际表现仍由宿主和 SG 消费，不是任意随从通用人格系统。[构造](../../../raw/current/scripts/components/crittertraits.lua#L4-L21) [定义](../../../raw/current/scripts/tuning.lua#L525-L537) [组装](../../../raw/current/scripts/prefabs/critters.lua#L262-L266)
- 优先使用：IncTracker(name,multiplier) 改分数，RefreshDominantTrait 按分数选型，IsDominantTrait 查询；SetDominantTrait 只设字段／标签，不等于完整选型通知。[积分与衰减](../../../raw/current/scripts/components/crittertraits.lua#L147-L163) [主导](../../../raw/current/scripts/components/crittertraits.lua#L165-L212)
- 重要副作用：构造零延迟 StartTracking，读取当前 follower 主人并注册两端事件、启动计时；OnPet 进入 SG，积分由后续 critter_onpet 事件产生。[启动](../../../raw/current/scripts/components/crittertraits.lua#L116-L145) [抚摸](../../../raw/current/scripts/components/crittertraits.lua#L23-L32) [SG 事件](../../../raw/current/scripts/stategraphs/SGcritter_common.lua#L282-L310)
- 修改边界：需要 follower、timer、匹配 SG 和正确主人；StartTracking 不幂等、没有自动跟随主人变化重绑逻辑。宠物所有权／生成另见 [[reference/components/petleash]]，不要只挂本组件补齐整套行为。

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `IncTracker(name,multiplier)` | 名称转大写；已存在键才处理，缺 multiplier 默认 1；当前主导类另乘 1.1，乘对应 TUNING.inc 后累计，上限 40。负增量不在此截到 -6，衰减才有下限。[公式](../../../raw/current/scripts/components/crittertraits.lua#L147-L163) |
| `DecayTraits()` | 每类减 decay×DECAY_TICK_RATE，最低 -6；比例在模块加载时由 CRITTER_TRAIT_DECAY_DELAY／TOTAL_DAY_TIME 计算，不是每 tick 直接减完整 decay。[比例](../../../raw/current/scripts/components/crittertraits.lua#L4-L6) [衰减](../../../raw/current/scripts/components/crittertraits.lua#L159-L163) |
| `RefreshDominantTrait()` | locked 时返回；最高分>0 才选择，换到新主导时发 crittertraitchanged。并列按 pairs 遍历，无固定优先级；全不正数时清主导，但该分支不发变更事件。[选型](../../../raw/current/scripts/components/crittertraits.lua#L183-L212) |
| `SetDominantTrait(trait)`／`IsDominantTrait(trait)` | 大写名字，setter 删旧／加新 trait_标签；不检查名称表、分数和锁、不发 crittertraitchanged。不能拿它模拟正常性格成长全部副作用。[字段／标签](../../../raw/current/scripts/components/crittertraits.lua#L165-L180) |
| `OnPet(petter)`／`SetOnPetFn(fn)` | 先 SG emote_pet，再 fn(critter,petter)；不是直接加分。普通抚摸加 PLAYFUL 倍率 1，nuzzle 后窗口内加 5，并消耗窗口任务。[入口](../../../raw/current/scripts/components/crittertraits.lua#L23-L32) [积分窗口](../../../raw/current/scripts/components/crittertraits.lua#L72-L93) |
| 进食与锁定 | 已有主导且食品为 GOODIES 时锁定主导，之后仍会加分／衰减；食物喜好、料理／新鲜度及宠物 stale 状态影响 WELLFED 积分，不直接等于饥饿恢复量。[进食](../../../raw/current/scripts/components/crittertraits.lua#L34-L63) |
| 存读 | 保存主导、锁和各分数；恢复通过 setter 重建标签，兼容旧 AFFECTIONATE→PLAYFUL。该兼容分支假设旧数据有可相加的 PLAYFUL，不声称接收任意缺字段字典。[存读](../../../raw/current/scripts/components/crittertraits.lua#L214-L245) |

decay／dominant 为 timer 名：前者周期衰减，后者首次等待／后续主导等待／无主导重试使用不同参数；锁定只阻止选主导，不停止计时重排或积分。监听的主人是 StartTracking 当时取得的对象，转交宠物后的行为积分归属需要单独验证，不重复调用 StartTracking“补绑定”。[计时消费](../../../raw/current/scripts/components/crittertraits.lua#L105-L145)

## 典型接入

### 原领养小猫正向玩耍性格积分翻倍

目标模组 `modmain.lua`，无新资源；仅原注册 critter_kitten，包装公开 IncTracker 的 PLAYFUL 正向倍率，保留其他性格、负增量、分数上限、原主导偏置与动画计分时机。

```lua
AddPrefabPostInit("critter_kitten", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local traits = inst.components.crittertraits
    if traits == nil then return end
    local previous = traits.IncTracker
    traits.IncTracker = function(self, name, multiplier)
        local amount = multiplier == nil and 1 or multiplier
        if GLOBAL.string.upper(name) == "PLAYFUL" and amount > 0 then
            multiplier = amount * 2
        end
        return previous(self, name, multiplier)
    end
end)
```

单位是 TUNING.inc 之前的倍率，不是直接加两点；主导偏置和 40 上限可使最终分数增量不正好翻倍。不会立即选主导、强制解锁、提高抚摸频率或手工发第二次 critter_onpet，其他直接改 traitscore 的模组不经过包装。每次构造重装，分数／锁沿用存读；停用重载取消倍率，但不回滚已积累分数。[小猫注册](../../../raw/current/scripts/prefabs/critters.lua#L374) [积分公式](../../../raw/current/scripts/components/crittertraits.lua#L147-L157)

原实例 WELLFED 主导由进食回调影响 perishable 饥饿时长范围，不是通用 hunger 参数；具体食用与消耗复用 [[reference/components/eater]]，性格表现只按实际消费者判断。[原宿主消费](../../../raw/current/scripts/prefabs/critters.lua#L24-L40)

## 常见陷阱与限制

- **OnPet 后手工再加同份分数**：原 SG 在 animover 且 AnimDone 时发送 critter_onpet，重复补发／加分可能双计；中途打断不保证进入该分支。[动画事件](../../../raw/current/scripts/stategraphs/SGcritter_common.lua#L282-L310) [监听](../../../raw/current/scripts/components/crittertraits.lua#L119-L125)
- **locked 表示分数和状态完全冻结**：只在 RefreshDominantTrait 开头判断；IncTracker、DecayTraits 和直接 setter 不读锁。[积分](../../../raw/current/scripts/components/crittertraits.lua#L147-L176) [锁](../../../raw/current/scripts/components/crittertraits.lua#L183-L186)
- **任意实体装组件就能开性格系统**：延迟任务直接读 follower、owner、timer，OnPet 直接操作 SG；先完成宿主组装，不对无家野生实体生搬。[前提](../../../raw/current/scripts/components/crittertraits.lua#L116-L145)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；漂移文件 tuning.lua、actions.lua 对照 R-01 hunk 地图逐条核验：tuning 仅新增 Rifts8 数值段（本页引用的 CRITTER_TRAITS 常量未变、整体下移 1 行），actions.lua 无本页引用锚点且变更不涉小宠物域；锚点重定位 1 处、结论无修订；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、计分／衰减／选型／锁／存读、原小猫工厂和 SG 抚摸事件、宿主 WELLFED 消费、当前四类参数；PET 动作只作为已核验入口线索。未全面审校所有小宠物 Brain／SG、转交后监听重绑、平台统计和全部旧存档变体。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整正向倍率例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；普通／窗口抚摸、上限／并列选择、GOODIES 锁定、存读、转交和多模组覆盖待实测。
