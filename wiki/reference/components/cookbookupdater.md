---
title: "cookbookupdater"
created: "2026-10-24"
updated: "2026-10-28"
type: component
summary: "维护玩家的游戏内料理记录，区分临时缓存、客户端个人 TheCookbook、学习事件与定向 RPC；不是项目 cookbook 维护工具。"
aliases: ["cookbookupdater", "料理指南记录", "烹饪配方记录", "食物已品尝标记", "TheCookbook"]
tags: [component, data, recipe, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/cookbookupdater.lua", "cookbookdata.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "components/stewer.lua", "networkclientrpc.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# cookbookupdater

## 快速判断

- 职责与适用：玩家两端的游戏内料理记录适配层，把配方材料组合或已品尝状态交给 CookbookData，必要时定向通知该玩家客户端；与本知识库的 cookbook 骨架没有关系。[构造](../../../raw/current/scripts/components/cookbookupdater.lua#L9-L13) [两类记录](../../../raw/current/scripts/components/cookbookupdater.lua#L16-L37)
- 优先使用：已有玩家组件用 `LearnRecipe(product, ingredients)` 记录料理组合，`LearnFoodStats(product)` 记录已品尝；制作配方解锁另见 [[reference/components/builder]]／[[reference/components/teacher]]，食物数值另见 [[reference/components/edible]]。[方法](../../../raw/current/scripts/components/cookbookupdater.lua#L16-L37) [已品尝字段](../../../raw/current/scripts/cookbookdata.lua#L147-L160)
- 重要副作用：本地玩家激活时将新建的临时 cookbook 换成全局 `TheCookbook`，并启用保存；专用服务器及主模拟中的远端玩家只在底层报告 updated 且有 userid 时发送对应客户端 RPC。[切换](../../../raw/current/scripts/components/cookbookupdater.lua#L1-L6) [发送条件](../../../raw/current/scripts/components/cookbookupdater.lua#L23-L25)
- 修改边界：原版在 `SetPristine` 和客户端返回之前添加，普通玩家已有组件。它不注册自己的 netvar、存档或通用 RPC，不应在其他实体上直接复制后期待个人资料持久化。[公共安装](../../../raw/current/scripts/prefabs/player_common.lua#L2516) [端界](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2614)

## 关键接口

| 接口／字段 | 参数与约定 | 作用、边界与证据 |
|---|---|---|
| `LearnRecipe(product, ingredients)` | 料理名、材料名数组 | 两项非 nil 才调用 AddRecipe；本方法不返回 updated，也不自行校验材料类型或长度。底层要求料理在 cookbook_recipes 中，归一熟食名称并排序后记组合。[入口](../../../raw/current/scripts/components/cookbookupdater.lua#L16-L27) [有效料理](../../../raw/current/scripts/cookbookdata.lua#L131-L136) [组合处理](../../../raw/current/scripts/cookbookdata.lua#L191-L228) |
| `LearnFoodStats(product)` | 料理名 | 底层首次设置 `has_eaten=true`；不是修改营养值、模拟吃食物或解锁制作配方。本方法不返回底层更新结果。[入口](../../../raw/current/scripts/components/cookbookupdater.lua#L30-L37) [实际字段](../../../raw/current/scripts/cookbookdata.lua#L147-L160) |
| `cookbook` | 新建 CookbookData，或本地 `TheCookbook` | 构造独立空数据；监听的是**玩家自身**的 playeractivated。仅非专服且 inst==ThePlayer 时切换，全局本地记录不与临时表自动合并。[构造与切换](../../../raw/current/scripts/components/cookbookupdater.lua#L1-L13) [宿主事件](../../../raw/current/scripts/prefabs/player_common.lua#L826-L827) |
| `cookbook.save_enabled` | 新建对象默认 nil；本地激活设 true | AddRecipe／LearnFoodStats 在 updated 且此值为真时才保存；保存到个人 persistent string `cookbook`，不是组件写入世界存档。[初值与保存](../../../raw/current/scripts/cookbookdata.lua#L6-L23) [保存门禁](../../../raw/current/scripts/cookbookdata.lua#L164-L168) [配方保存](../../../raw/current/scripts/cookbookdata.lua#L230-L234) |

每个料理最多保留六种材料组合；已有组合排在第 3 项以后时，重新学习会把它移到前面并返回 updated=true。因此网络通知条件不是“绝不重复的首次发现”，也不是客户端 ACK；临时缓存按组件实例维护，不能当全服统一食谱档案。[容量](../../../raw/current/scripts/cookbookdata.lua#L3) [重排与新增](../../../raw/current/scripts/cookbookdata.lua#L210-L227) [RPC门禁](../../../raw/current/scripts/components/cookbookupdater.lua#L23-L25)

## 典型接入

### 沿原版玩家事件接入料理记录

这是已有玩家和料理系统的接入指引，不是一份新食物或新厨锅 demo：

1. 玩家已监听 `learncookbookrecipe` 与 `learncookbookstats`；前者数据为 `{product=料理名, ingredients=材料名数组}`，后者数据直接是料理名字符串。事件回调从当前玩家取组件并转调，不在世界实体上推同名事件。[监听对象](../../../raw/current/scripts/prefabs/player_common.lua#L688-L690) [载荷读取](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L908-L919)
2. 厨锅收获时，只有收获者 userid 等于 chef_id，且料理已进入对应 cookbook 分类，才把实际材料组合发给该收获者。这个门禁属于 stewer 的调用方，直接 LearnRecipe 不会重验“是否亲手烹饪”。加工与消费继续见 [[reference/components/stewer#典型接入]]。[原版通知条件](../../../raw/current/scripts/components/stewer.lua#L285-L293)
3. 玩家 `oneat` 的回调仅对 `preparedfood` 取 `food_basename or prefab` 并记录已品尝；没有替任意物品增删 edible 或更改收益。[食用记录入口](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L922-L926)
4. 服务器发给特定 userid 后，客户端 RPC 处理器取 ThePlayer 的组件，再调用同一方法；配方处理器还要求材料非空。新模组应提供有效料理／材料，不把空数组或任意实体直接当作记录输入。[客户端处理](../../../raw/current/scripts/networkclientrpc.lua#L1425-L1437)

### 分清个人记录与世界存读

新玩家实例先持有临时 CookbookData，本地激活后才用 `TheCookbook`；组件无 OnSave／OnLoad，不能依靠世界存档保存其临时去重表。个人 CookbookData 的 Load／Save 负责自己的 persistent string，模组新增料理还需进入真实料理注册路径，单写一个任意 product 不会通过 IsValidEntry。[组件完整定义](../../../raw/current/scripts/components/cookbookupdater.lua#L1-L42) [个人存读](../../../raw/current/scripts/cookbookdata.lua#L18-L33) [有效性检查](../../../raw/current/scripts/cookbookdata.lua#L131-L136)

## 常见陷阱与限制

- **把 LearnFoodStats 当作修改属性：** 它只设置已品尝标记，食物三维仍归 edible／eater；无效料理会被底层静默拒绝。[底层实现](../../../raw/current/scripts/cookbookdata.lua#L147-L160)
- **所有客户端都直接保存服务器表：** 网络发送只传料理和展开的材料，由目标客户端更新其个人表；远端玩家组件不等于本机 TheCookbook。新实例缓存也不代表该客户端实际已知的全部记录。[分端条件](../../../raw/current/scripts/components/cookbookupdater.lua#L1-L6) [定向发送](../../../raw/current/scripts/components/cookbookupdater.lua#L23-L25)
- **激活前写临时表就会自动合并到个人资料：** 激活回调直接替换 cookbook 引用，没有合并或重放逻辑；需要本地持久记录时遵守现成玩家激活流程，不自行强开服务器临时对象的 save_enabled。[替换](../../../raw/current/scripts/components/cookbookupdater.lua#L1-L6)
- **把方法返回值当成功结果：** 更新器两方法不返回底层结果；重复、无效输入与真实更新不能通过其返回值区分。方法也不提供通用移除／遗忘或网络确认接口。[方法定义](../../../raw/current/scripts/components/cookbookupdater.lua#L16-L37)

## 验证记录

- 增量复查：2026-10-25，Steam buildid 26148036；仅锚点重定位（player_common −12/−14 段、player_common_extensions +4 段），组件与数据层无变化，P-83/P-84 变更区不涉本页语义，结论维持。
- 源码审阅：2026-10-24，Steam buildid 25643519；完整更新器、CookbookData 的有效性／学习／排序／容量与保存，玩家公共构造／激活／事件、厨锅通知条件、已品尝入口及客户端 RPC。未全面审阅前端料理界面、在线资料同步、料理注册和平台存储故障恢复。
- 结构检查：本页格式、来源、本地链接和源码锚点严格检查通过；无完整 Lua 示例，导航登记由本批统一检查。
- 游戏运行：未运行；本地／专服／远端切换、个人资料保存、重连重学与新增料理交互未实测。
