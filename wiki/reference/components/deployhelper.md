---
title: "deployhelper"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "附近实体响应本地放置预览并显示辅助效果，按配方／key 过滤，停止刷新后自动关闭。"
aliases: ["deployhelper","部署辅助","建筑范围提示","放置辅助圈"]
tags: [component, ui, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/deployhelper.lua","components/placer.lua","prefabs/table_winters_feast.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# deployhelper

## 快速判断

- 职责与适用：挂在需要响应附近放置的现有实体，通过本地组件登记表接收 TriggerDeployHelpers；不在这里创建建筑或判定部署成功。[deployhelper.lua:4](../../../raw/current/scripts/components/deployhelper.lua#L4
- 优先使用：本地可见宿主加组件、配过滤与 onenablehelper，复用 [[reference/components/placer]] 刷新；原版冬季盛宴桌在非独服共同分支初始化。[table_winters_feast.lua:468](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L468
- 重要副作用：StartHelper 设置 delay=2 并启动 wall-update；重复触发续期，onenablehelper 只在开始／结束调用，onstarthelper 每次调用。[deployhelper.lua:55](../../../raw/current/scripts/components/deployhelper.lua#L55
- 修改边界：这是本地表现，不提供服务器距离、权限或材料校验；原版圈的创建／父级／删除由宿主回调负责。[table_winters_feast.lua:423](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L423

## 关键接口

| 接口／字段 | 约定 | 条件、副作用与证据 |
|---|---|---|
| `AddRecipeFilter(name)`／`AddKeyFilter(key)` | 字符串 | 两种过滤是 OR；都未设置时匹配全部，手动清为空表则不是“无过滤”。[deployhelper.lua:4](../../../raw/current/scripts/components/deployhelper.lua#L4)、[deployhelper.lua:39](../../../raw/current/scripts/components/deployhelper.lua#L39 |
| `canenablehelper(inst)` | 可选 bool 回调 | 全局触发筛选时调用；直接 StartHelper 不执行它。[deployhelper.lua:11](../../../raw/current/scripts/components/deployhelper.lua#L11 |
| `StartHelper(recipename,placerinst)` | 名字／预览实体 | 醒着且未激活才启动，重复仅续期；休眠时仍可能调 onstarthelper，不应把该回调视为可靠 enabled 通知。[deployhelper.lua:55](../../../raw/current/scripts/components/deployhelper.lua#L55 |
| `onenablehelper(inst,enabled,recipename?,placerinst?)` | 开／关 | 关闭只传 inst,false，其余参数可缺；回调自行创建／销毁 FX，不要假设始终有 placerinst。[deployhelper.lua:61](../../../raw/current/scripts/components/deployhelper.lua#L61)、[deployhelper.lua:71](../../../raw/current/scripts/components/deployhelper.lua#L71 |
| `onstarthelper(inst,name,placerinst)` | 每次 StartHelper | 与 onenablehelper 不同，重复请求也回调；需要逐帧更新预览引用可在此设计，不会自动刷新已有 FX 保存的引用。[deployhelper.lua:66](../../../raw/current/scripts/components/deployhelper.lua#L66 |
| `StopHelper()` | 无显式返回 | 活跃时停更新并关闭，未活跃则无操作。[deployhelper.lua:71](../../../raw/current/scripts/components/deployhelper.lua#L71 |
| `OnEntityWake/Sleep` | 生命周期回调 | 醒时加入登记表，睡时移出并 Stop；实体／组件移除均复用睡眠清理。[deployhelper.lua:27](../../../raw/current/scripts/components/deployhelper.lua#L27 |

TriggerDeployHelpers(x,y,z,range,recipe,placerinst) 是该模块定义的全局函数，range 为距离，内部平方并严格比较；正常 placer 每次刷新以 64 单位触发。recipe 过滤取 recipe.name，key 取 placerinst.deployhelper_key，传给回调的 name 优先 recipe.name。delay=2 是两次 wall-update 的计数，不是两秒；停止持续刷新后自动关闭。[deployhelper.lua:4](../../../raw/current/scripts/components/deployhelper.lua#L4)、[deployhelper.lua:81](../../../raw/current/scripts/components/deployhelper.lua#L81)、[placer.lua:535](../../../raw/current/scripts/components/placer.lua#L535

## 典型接入

### 冬季盛宴桌的放置辅助仅白天显示

目标模组 `modmain.lua`；复用原版桌子的过滤、圈资源及开关回调，只包装可启用条件。共同端执行，独服原本没有此组件，判空即可；不改变桌子聚餐距离或实际建造条件。[table_winters_feast.lua:469](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L469

```lua
AddPrefabPostInit("table_winters_feast", function(inst)
    local helper = inst.components.deployhelper
    if helper == nil then return end
    local original = helper.canenablehelper
    helper.canenablehelper = function(entity)
        return GLOBAL.TheWorld.state.isday
            and (original == nil or original(entity))
    end
end)
```

白天仍需命中过滤、距离和宿主状态；夜晚不续期，已有辅助在之后 wall-update 关闭，不承诺切夜瞬间消失。原版 onenablehelper(false) 删除圈，睡眠／移除也清理。停用后新建／载入恢复原条件，不写存档，无新增资源／任务。[deployhelper.lua:81](../../../raw/current/scripts/components/deployhelper.lua#L81)、[table_winters_feast.lua:438](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L438

## 常见陷阱与限制

- **服务端加完就能让客户端见圈**：本表和回调没有网络同步；按原版非独服共同分支建立本地辅助。[table_winters_feast.lua:468](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L468
- **delay=2 是两秒**：OnWallUpdate 每次减一，不乘 dt；不要用它实现持续 buff 或精确游戏计时。[deployhelper.lua:81](../../../raw/current/scripts/components/deployhelper.lua#L81
- **recipe 与 key 必须同时匹配**：实际 OR；也不要凭过滤匹配认定回调 name 一定是 key。[deployhelper.lua:7](../../../raw/current/scripts/components/deployhelper.lua#L7
- **onenablehelper 每次收到最新 placer**：活跃中 StartHelper 只续期，需更新已有引用时另外处理；退出分支须允许 name／placer 为 nil。[deployhelper.lua:55](../../../raw/current/scripts/components/deployhelper.lua#L55
- **StartHelper 就创建通用圆圈**：组件只调回调，绘制／移除方案在宿主；不要用辅助圈替代 [[reference/components/deployable]] 的实际验证。[table_winters_feast.lua:423](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L423

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整组件、placer 触发位置／范围、冬季盛宴桌非独服挂载与圈开关／删除。未审校所有 helper 宿主、圆圈视觉更新算法、引擎 wall-update 调度或全部休眠时序。
- 结构检查：格式、来源、大小、链接／源码锚点及登记；完整示例经 Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；昼夜切换、放置取消、休眠、多人／独服、手柄和不同预览切换待测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/placer]]；[[reference/components/deployable]]。
