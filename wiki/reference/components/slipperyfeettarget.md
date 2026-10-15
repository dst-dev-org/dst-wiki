---
title: "slipperyfeettarget"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "把地面实体标记为滑面，提供按世界坐标的覆盖判定和针对行走者的滑度累计倍率，供 slipperyfeet 定向查询。"
aliases: ["slipperyfeettarget", "自定义冰面", "滑面范围", "冰面打滑倍率", "企鹅冰面"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/slipperyfeettarget.lua", "components/slipperyfeet.lua", "prefabs/penguin_ice.lua", "prefabs/pond.lua", "prefabs/warg_mutated_fx.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# slipperyfeettarget

## 快速判断

- 职责与适用：挂在地面滑面实体上，为 [[reference/components/slipperyfeet]] 提供“该位置是否在滑面内”及“该行走者累计多快”的回调；组件不追踪玩家、不定时更新、不直接造成跌倒。[几何接口](../../../raw/current/scripts/components/slipperyfeettarget.lua#L19-L31) [倍率接口](../../../raw/current/scripts/components/slipperyfeettarget.lua#L38-L43) [消费方](../../../raw/current/scripts/components/slipperyfeet.lua#L104-L114)
- 优先使用：有合适 Physics 半径时用默认圆形判定；无 Physics、椭圆或阶段性滑面用 `SetIsSlipperyAtPoint(fn)`；按角色／速度调整用 `SetSlipperyRate(fn)`，传入的是函数而非数值。[默认判定](../../../raw/current/scripts/components/slipperyfeettarget.lua#L19-L31) [设置倍率](../../../raw/current/scripts/components/slipperyfeettarget.lua#L34-L40)
- 重要副作用：构造加 `slipperyfeettarget` 标签，移除时撤掉；角色只搜索半径 12 内带此标签的实体。多个滑面不叠加，首个几何命中的目标决定倍率，不能假定最高倍率胜出。[标签](../../../raw/current/scripts/components/slipperyfeettarget.lua#L7-L12) [查询](../../../raw/current/scripts/components/slipperyfeet.lua#L3-L4) [首个命中](../../../raw/current/scripts/components/slipperyfeet.lua#L104-L114) [倍率消费](../../../raw/current/scripts/components/slipperyfeet.lua#L219-L227)
- 修改边界：原版在服务端添加组件；永久滑面 `penguin_ice` 在 `SetPristine` 前预加标签，组件源码也建议如此；冻结池塘和冰化余烬则动态增删／添加，不把双端组件当要求。[推荐](../../../raw/current/scripts/components/slipperyfeettarget.lua#L7-L8) [企鹅工厂](../../../raw/current/scripts/prefabs/penguin_ice.lua#L158-L179) [池塘](../../../raw/current/scripts/prefabs/pond.lua#L162-L182)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `SetIsSlipperyAtPoint(fn)` | `fn(surface, x, y, z) -> truthy/false`；`nil` 恢复默认 | 设置世界坐标判定；有回调时直接返回其结果，不再执行默认圆形逻辑。[slipperyfeettarget.lua:15-21](../../../raw/current/scripts/components/slipperyfeettarget.lua#L15-L21) |
| `IsSlipperyAtPosition(x, y, z)` | 默认返回 boolean | 无回调则读 `Physics:GetRadius()` 与 Transform 世界位置，按 XZ 平面严格 `distance² < radius²`；无 Physics 返回 false，圆周边界不含在内。[slipperyfeettarget.lua:19-31](../../../raw/current/scripts/components/slipperyfeettarget.lua#L19-L31) |
| `SetSlipperyRate(fn)` | `fn(surface, walker) -> number`；`nil` 恢复 1 | 没有校验／钳制；消费者直接乘入每帧滑度，推荐返回有限非负倍率，不传常量或 nil 返回值。[设置与读取](../../../raw/current/scripts/components/slipperyfeettarget.lua#L34-L43) [数值消费](../../../raw/current/scripts/components/slipperyfeet.lua#L254) |
| `GetSlipperyRate(target)` | 默认 `1` | `target` 是当前角色实体，不是位置；倍率只控制累计快慢，不改变移动速度、滑面大小或 SG 状态。[slipperyfeettarget.lua:38-43](../../../raw/current/scripts/components/slipperyfeettarget.lua#L38-L43) [累计公式](../../../raw/current/scripts/components/slipperyfeet.lua#L254) |
| `OnRemoveFromEntity()` | 框架移除钩子 | 只删标签；组件没有自己的任务、存档或回调持久化，重建时由宿主重设回调。[slipperyfeettarget.lua:11-12](../../../raw/current/scripts/components/slipperyfeettarget.lua#L11-L12) [回调字段](../../../raw/current/scripts/components/slipperyfeettarget.lua#L15-L16) |

## 典型接入

### 原版模式：永久几何、冻结切换与短时残留

| 原版对象／注册入口 | 组装与范围 | 生效／撤销边界与证据 |
|---|---|---|
| `penguin_ice` | 双端预加标签，服务端添加组件，设置椭圆几何和恒定倍率 `2.75`；无需依赖 Physics 默认半径。 | `isfaded` 时几何返回 false；未淡出时按冰面 FX 的 visual BB 检查矩形外壳与椭圆，FX 失效有默认 BB 退路。对象本身 `persists=false`。[几何](../../../raw/current/scripts/prefabs/penguin_ice.lua#L113-L136) [倍率](../../../raw/current/scripts/prefabs/penguin_ice.lua#L139-L140) [构造与注册](../../../raw/current/scripts/prefabs/penguin_ice.lua#L177-L191) |
| `pond`／`pond_mos` 的雪量切换 | 共用工厂有 `MakePondPhysics(inst, 1.95)`，冻结后动态加组件，使用默认圆形判定。 | 雪量 `> .02` 冻结时加；解冻时移除。倍率按目标 motor speed 是否大于 `WILSON_RUN_SPEED` 返回 `50` 或 `5`，不是固定池塘倍率。[Physics](../../../raw/current/scripts/prefabs/pond.lua#L223) [速度分支](../../../raw/current/scripts/prefabs/pond.lua#L136-L142) [冻结／解冻](../../../raw/current/scripts/prefabs/pond.lua#L145-L182) [初始与后续雪量检查](../../../raw/current/scripts/prefabs/pond.lua#L296-L301) |
| `warg_mutated_ember_fx` 冰化余烬 | 服务端 `SetIced` 动态添加，设置自定义圆形半径 1 与 `TUNING.ICED_EMBER_SLIPPERY_RATE`。 | `fade > 18` 后判定返回 false；并不依靠立即移除组件结束滑面。该 FX 非持久化，宿主提供重启与移除流程。[几何／倍率](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L524-L544) [服务端方法与注册](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L574-L596) |

### 保留企鹅冰面形状，只降低打滑累计倍率

- 文件／加载：目标模组 `modmain.lua`，完整入口如下；未假定 `env → GLOBAL` 代理，只给全局世界对象加 `GLOBAL`。前提是原版 `penguin_ice` 及已有 `slipperyfeet` 宿主，不增加美术或额外 Prefab。
- 替换点：`1.5` 替换为目标倍率；这是有意覆盖原版 `2.75` 的回调，不是与原倍率相乘。保留原几何、淡出与对象生命周期，仅作用于启用本例后创建的冰面；原对象 `persists=false`，由原版重新生成时再执行 hook，不为冰面新增存档。[原版装配](../../../raw/current/scripts/prefabs/penguin_ice.lua#L158-L186) [setter](../../../raw/current/scripts/components/slipperyfeettarget.lua#L34-L40)

```lua
AddPrefabPostInit("penguin_ice", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.slipperyfeettarget:SetSlipperyRate(function(surface, walker)
        return 1.5
    end)
end)
```

- 生效／撤销：跑动进入有效企鹅冰面且没有防滑保护时使用新倍率；离开范围撤销实体来源，但已有滑度仍按角色组件衰减。停用模组并重新载入后回归原版工厂；不声称支持运行中热卸载。防滑与多滑面重叠行为见 [[reference/components/slipperyfeet#来源累计与防滑顺序]]。[来源更新](../../../raw/current/scripts/components/slipperyfeet.lua#L219-L230) [累计与衰减](../../../raw/current/scripts/components/slipperyfeet.lua#L245-L257)

## 常见陷阱与限制

- **扩大判定到半径 12 外：** 覆盖回调再大，搜索仍先按角色到滑面实体的距离 12 取候选；超出搜索距离的区域不会执行几何判定。大范围方案要另设计分布式目标／角色来源，不把改半径回调当全图滑面。[搜索常量](../../../raw/current/scripts/components/slipperyfeet.lua#L3-L4) [查询调用](../../../raw/current/scripts/components/slipperyfeet.lua#L104-L109)
- **只有标签、没有组件：** 搜索能找到但消费者检查组件存在才执行几何；只有组件而无 Physics／自定义几何则默认 false。双端预加标签用于静态构造优化，不代表客户端必须拥有此组件。[组件门禁](../../../raw/current/scripts/components/slipperyfeet.lua#L107-L110) [默认无 Physics](../../../raw/current/scripts/components/slipperyfeettarget.lua#L24-L31) [原版端界](../../../raw/current/scripts/prefabs/penguin_ice.lua#L168-L179)
- **把倍率 setter 当数值 setter：** `SetSlipperyRate(2)` 不会立即报参数错误，但随后 `GetSlipperyRate` 会尝试调用数字；回调返回 nil 也无法参与累计乘法。返回值须由模组保证。[setter／调用](../../../raw/current/scripts/components/slipperyfeettarget.lua#L34-L40) [消费者](../../../raw/current/scripts/components/slipperyfeet.lua#L254)
- **多个滑面按强度覆盖：** 组件没有优先级或合并逻辑；首个命中的实体控制倍率。不要据叠加预期设计平衡，重叠顺序需要游戏实测。[首个命中](../../../raw/current/scripts/components/slipperyfeet.lua#L104-L114) [单目标倍率](../../../raw/current/scripts/components/slipperyfeet.lua#L226-L227)
- **移除滑面就立即摔倒／立即清零：** 目标仅参与来源和倍率，真正阈值、衰减、SG 事件在角色组件；离开有效范围不等于清掉角色已累计的滑度。详见 [[reference/components/slipperyfeet#关键接口]]。[来源撤销](../../../raw/current/scripts/components/slipperyfeet.lua#L222-L227) [非累计衰减](../../../raw/current/scripts/components/slipperyfeet.lua#L256-L257)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-23，Steam buildid 25643519；完整核验组件，定向核验 `slipperyfeet` 搜索与倍率消费、企鹅冰面的双端标签／几何、池塘雪量增删和冰化余烬阶段判定。示例只覆盖企鹅冰面倍率替换，不改其他滑面。
- 结构与语法：本页 `validate_page.py` 字段、大小及链接检查通过；`validate_source_links.py --strict` 源码锚点检查通过；完整示例通过 Lua 5.1 `parser_probe --syntax-only`，临时文件已清理。
- 游戏运行：未运行；倍率变化的实际跌倒节奏、重叠冰面选择、网络表现与其他模组 hook 冲突未实测。语法检查不验证加载绑定或游戏效果。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/slipperyfeet]]；[[reference/components/nonslipgritsource]]；[[reference/components/nonslipgrituser]]。
