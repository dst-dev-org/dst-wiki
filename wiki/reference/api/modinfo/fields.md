---
title: modinfo 原生字段清单
created: 2026-10-11
updated: "2026-10-28"
type: api
summary: 查 modinfo.lua 的原生字段、默认行为和示例，包括优先级、运行端、图标、依赖与版本兼容
aliases: [modinfo 字段, 可选配置, priority, 加载优先级, icon_atlas, forumthread, server_filter_tags, mod_dependencies, standalone, restart_required, forcemanifest, version_compatible, api_version_dst]
tags: [modding, server-client]
dst_build: "steam-buildid:26230496"
sources: [modindex.lua, mods.lua, widgets/redux/modstab.lua, gamemodes.lua]
reliability: verified
validation: source-reviewed
status: active
---

# modinfo 原生字段清单

## 快速判断

- 优先使用：按下面的字段表选配；完整起步文件见 [[cookbook/mod-starter#最小-demo]]，配置项结构与读取见 [[reference/api/modinfo/configuration]]。
- 重要副作用：`priority` 越大越早加载；`standalone = true` 会排除本次加载列表中的其他模组；依赖声明不负责拓扑排序。[排序:490](../../../../raw/current/scripts/mods.lua#L490)、[独占:314](../../../../raw/current/scripts/modindex.lua#L314)
- 修改边界：覆盖本快照可见 Lua 消费链识别的 **23 个作者声明字段**；表内示例是分别选用的赋值，不应把所有特殊开关同时启用。开放表和项目私有字段没有有限的“所有值”枚举。

## 关键接口

### 基本信息与兼容声明

| 字段／类型 | 赋值示例 | 缺省行为、作用与限制 |
|---|---|---|
| `name` / string | `name = "Mod Starter"` | 当前 API 的必填展示名；不改变文件夹标识。多语言与开发后缀见 [[reference/api/modinfo/configuration#本地开发名称与多语言]]。 |
| `description` / string | `description = "新增一个随身物品。"` | 必填简介；可用 `\n` 换行。 |
| `author` / string | `author = "YourName"` | 必填作者展示文字，不是账号认证。 |
| `version` / string | `version = "1.2.0"` | 必填模组版本；加载后去首尾空白并转小写，不是 DST build。相同版本可能沿用旧元信息。 |
| `api_version` / number | `api_version = 10` | 当前 `MOD_API_VERSION` 为 10；缺失／较低走旧 API 提示，较高标记失败；正常当前 API 分支检查必填字段。 |
| `api_version_dst` / number | `api_version_dst = 10` | 可省略；存在时替换本次 DST 加载的 `api_version`，供共用文件区分 API。单独 DST 模组通常只写 `api_version`。 |
| `dst_compatible` / boolean | `dst_compatible = true` | 省略被补为 true，但记录“未明确声明”并提示；false 声明不兼容 DST。应显式填写。 |
| `dont_starve_compatible` / boolean | `dont_starve_compatible = false` | 省略被补为 true；声明是否兼容单机本体，不会把 DST 实现转换成单机实现。 |
| `reign_of_giants_compatible` / boolean | `reign_of_giants_compatible = false` | 省略被补为 true；声明是否兼容单机巨人国，同样不提供兼容实现。 |

依据：[必填及兼容缺省:619–684](../../../../raw/current/scripts/modindex.lua#L619-L684)、[API 版本:6](../../../../raw/current/scripts/mods.lua#L6)、[版本归一化与缓存:534](../../../../raw/current/scripts/modindex.lua#L534)。

### 运行端与用户配置

| 字段／类型 | 赋值示例 | 缺省行为、作用与限制 |
|---|---|---|
| `client_only_mod` / boolean | `client_only_mod = false` | 省略按假值处理；true 分类为纯客户端模组，不能承担服务器机制。 |
| `all_clients_require_mod` / boolean | `all_clients_require_mod = true` | 省略按假值处理；true 要求加入的客户端具备该服务器模组，参与版本检查；与 `client_only_mod = true` 互斥。 |
| `configuration_options` / table | `configuration_options = {}` | 可省略／空表表示无配置；非空顺序数组及全部条目字段、示例见 [[reference/api/modinfo/configuration#配置项与分组标题]]、[[reference/api/modinfo/configuration#客户端字符串选项]]。 |

按实际功能选择一行组合，不靠标志补齐代码或资源：[分类:144](../../../../raw/current/scripts/modindex.lua#L144)、[互斥警告:684](../../../../raw/current/scripts/modindex.lua#L684)、[客户端要求:897](../../../../raw/current/scripts/mods.lua#L897)。

| 模组用途 | `client_only_mod` | `all_clients_require_mod` |
|---|---|---|
| 仅本地 UI／输入辅助 | true | false |
| 服务器机制，客户端无需模组代码／资源 | false | false |
| 新物品／角色等需要客户端代码或资源 | false | true |

### 展示、链接与服务器标签

| 字段／类型 | 赋值示例 | 缺省行为、作用与限制 |
|---|---|---|
| `icon_atlas` / string | `icon_atlas = "images/modicon.xml"` | 与 `icon` 成对选配；路径相对模组根，缺一或文件不存在会清空图标字段。 |
| `icon` / string | `icon = "modicon.tex"` | 模组列表图标的纹理名；文件检查相对 atlas 所在目录，需匹配真实 XML／TEX 与纹理元素。不是物品栏图标声明。 |
| `forumthread` / string | `forumthread = "topic/12345-example/"` | 可省略；非 RAIL 平台拼到 Klei 论坛 `index.php?` 后，不填写任意完整 URL。无值时工坊模组转工坊页；本地模组转通用论坛入口。示例帖号需替换。 |
| `server_filter_tags` / string[] | `server_filter_tags = {"随身物品", "自定义规则"}` | 可省略；收集已启用模组的服务器标签，供列表识别／筛选；不会自行开启对应玩法。 |

依据：[图标校验:542](../../../../raw/current/scripts/modindex.lua#L542)、[图标资源加载:1344](../../../../raw/current/scripts/widgets/redux/modstab.lua#L1344)、[链接选择:917](../../../../raw/current/scripts/mods.lua#L917)、[标签收集:184](../../../../raw/current/scripts/mods.lua#L184)。本页不生成美术资源；选用图标前需有有效的 XML／TEX。

### 加载、版本与特殊行为

| 字段／类型 | 赋值示例 | 缺省行为、作用与限制 |
|---|---|---|
| `priority` / number | `priority = 100` | 缺省／无效类型按 0；数值越大越早加载。可用负数延后；数字字符串被转换，但推荐直接 number。详见下文。 |
| `mod_dependencies` / table[] | `mod_dependencies = {{workshop = "workshop-1234567890"}}` | 缺省无依赖；每个条目是一组候选，多个条目表示多项依赖。仅非纯客户端模组处理；ID 必须替换，详细结构见下文。 |
| `version_compatible` / string | `version_compatible = "1.1.0"` | 缺省／非字符串取当前 `version`；允许参与区间兼容检查。示例需配 `version = "1.2.0"` 并实际验证兼容；比较采用字符串顺序。 |
| `forcemanifest` / boolean | `forcemanifest = false` | nil 时工坊模组开启 manifest、本地模组关闭；true 强制开启，false 关闭。控制 `LoadModManifest` 分支及资源路径绑定；manifest 格式／生成需另查，不要为凑字段随意改默认。 |
| `standalone` / boolean | `standalone = false` | 仅严格 true 触发独占加载：发现后只返回该模组，其他模组不进入本次列表。适用于有意独占的特殊工程，普通模组省略或 false。 |
| `restart_required` / boolean | `restart_required = true` | 缺省不要求重启；UI 切换启用状态时弹确认，确认后保存并退出游戏，需重新启动。原注释用于需重启的客户端模组；不是服务器自动重启指令。 |
| `game_modes` / table | `game_modes = {{name = "mod_starter_mode", label = "示例模式"}}` | 缺省不注册模式；每条可有 `description`、`settings`。完整示例、初始化缺省与当前世界设置边界见 [[reference/api/modinfo/game-modes]]。 |

依据：[排序与 manifest:490–537](../../../../raw/current/scripts/mods.lua#L490-L537)、[依赖与模式:565–591](../../../../raw/current/scripts/modindex.lua#L565-L591)、[版本检查:1261](../../../../raw/current/scripts/modindex.lua#L1261)、[独占:314](../../../../raw/current/scripts/modindex.lua#L314)、[重启确认:1030–1059](../../../../raw/current/scripts/widgets/redux/modstab.lua#L1030-L1059)、[退出:1014](../../../../raw/current/scripts/widgets/redux/modstab.lua#L1014)。

## 典型接入

### 设置加载优先级

放在 `modinfo.lua`，仅在确有顺序需求时添加；以下是供其他模组调用的库希望早于默认优先级模组执行的示例。

```lua
priority = 100
```

100 早于 0，0 早于 -100。同优先级用展示名（异常类型退回实际模组标识）的 `stringidsorter` 排序，不按订阅顺序；不要依赖相同优先级形成固定前后关系。`priority` 排的是本轮模组入口加载顺序，不保证所有后续回调／实体初始化都按此顺序。[比较器与执行循环:500–526](../../../../raw/current/scripts/mods.lua#L500-L526)

### 声明工坊依赖与本地候选

放在 `modinfo.lua`；替换真实依赖 ID。最小方案优先用工坊标识，不填展示名或裸数字。

```lua
mod_dependencies = {
    {workshop = "workshop-1234567890"},
}
```

确需本地替代时，将同一条目替换为如下候选表；它表示一项依赖的可选实现，不是同时依赖三份：

```lua
mod_dependencies = {
    {
        workshop = "workshop-1234567890",
        ["library_dev"] = false,
        ["Library Display Name"] = true,
    },
}
```

候选的字符串键是模组标识／展示名，false 表示实际目录标识，true 表示调用 `GetModActualName` 反查展示名；不是字符串数组。未先选中已启用工坊依赖时，构建阶段为工坊主体优先排工坊候选，本地主体优先排已存在的本地候选；多个本地候选经 `pairs` 收集，顺序不保证。工坊候选即使未安装也可进入 UI 获取依赖流程。[构建:498](../../../../raw/current/scripts/modindex.lua#L498)、[UI:1038](../../../../raw/current/scripts/widgets/redux/modstab.lua#L1038)

当前 `FindEnabledMod` 对普通候选使用 `ipairs`，不会遍历上例的字符串键；只有 `workshop` 键明确先检查已启用。不能承诺任意本地候选“已启用就优先”。未形成候选会禁用主体；依赖信息用于获取／启用依赖，执行顺序另由 `priority` 排序。[选择与装载:484–593](../../../../raw/current/scripts/modindex.lua#L484-L593)、[依赖展开:1145](../../../../raw/current/scripts/modindex.lua#L1145)

### 声明版本兼容范围

合并到 `modinfo.lua`；只有真实验证过这些版本可互通时才放宽。下面展示写法，不是任意 1.x 模组都兼容的保证。

```lua
version = "1.2.0"
version_compatible = "1.1.0"
```

双方使用各自版本与兼容下界检查；未提供服务器兼容值的分支要求精确相等。源码用 Lua 字符串 `>=`，没有 SemVer 解析，`"1.10"` 与 `"1.9"` 的顺序不能按数值版本推断。保守做法是省略 `version_compatible`，使其等于当前版本。[归一化:534](../../../../raw/current/scripts/modindex.lua#L534)、[检查:1261](../../../../raw/current/scripts/modindex.lua#L1261)

## 常见陷阱与限制

- **把生成值当可选字段**：`folder_name`、`locale`、`ChooseTranslationTable` 是环境注入值；`iconpath`、`failed`、`modinfo_message`、`*_compatibility_specified` 是内部派生状态，不手写；`dependencies` 是索引内部数据，作者写 `mod_dependencies`。配置保存字段另见 [[reference/api/modinfo/configuration#配置字段清单]]。[环境:599](../../../../raw/current/scripts/modindex.lua#L599)、[图标:544](../../../../raw/current/scripts/modindex.lua#L544)、[依赖状态:577](../../../../raw/current/scripts/modindex.lua#L577)
- **元信息改了却未刷新**：同归一化 `version` 会返回 `prev_info`；图标、依赖、模式等可能沿用旧值。发布元信息更新时同步提高版本，测试时确认重新加载。[缓存:534](../../../../raw/current/scripts/modindex.lua#L534)
- **独占模组互相竞争**：多个 `standalone = true` 时取当前列表先发现的一份，不能承诺由最高 `priority` 决定；独占过滤发生在入口排序之前。[过滤:314](../../../../raw/current/scripts/modindex.lua#L314)、[排序:490](../../../../raw/current/scripts/mods.lua#L490)
- **套用其他产品／私有扩展**：本快照未找到 `shipwrecked_compatible`、`hamlet_compatible`、`server_only_mod` 的原生消费者；不将它们写成 DST 有效开关。任意项目私有变量只有被该项目读取才生效，不能把长 modinfo 中的每个赋值都当原生 API。

## 验证记录

### 版本增量复核 26148036（2026-10-25）
处理范围：锚点重定位 18 处（mods.lua 全部 -39；modindex.lua +6（独占过滤 308→314 等）；modstab.lua 重启确认／图标段 +3）；R43 核验：本页 23 个作者字段不含 Release ID 声明，R-11 注册表改版（mods.lua:13-15 改遍历 versioning.lua MODIDS、集合新增 R43_SHADOWYDEPTHS）不影响任一字段结论。依据：全局枢纽 diff 审阅＋变更清单命中 mods.lua、modindex.lua、widgets/redux/modstab.lua（gamemodes.lua 未变）。

2026-10-11，Steam buildid 25643519：检索本快照 Lua 元信息读取点并审阅字段装载、运行端、图标、论坛链接、标签、依赖、缓存、版本、优先级、manifest 开关、独占与重启 UI；23 个作者字段的声明示例及进一步解释均有入口。代码块经 Full Moon 3.0.0／Lua 5.1 语法和原字节往返检查。**未运行 DST**；依赖获取／切换、图标显示、跨版本联机和重启尚未实测，manifest 引擎内部不可见，不声称所有引擎／第三方扩展已穷尽。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。