---
title: 模组起步与加载验证
created: 2026-10-11
updated: "2026-10-28"
type: cookbook
summary: 用 modinfo.lua 与 modmain.lua 建立最小 DST 模组，配置全局读取代理并检查加载
aliases: [新建模组, 从零开始一个模组, 模组框架, modinfo, modmain, GLOBAL, env, 全局代理]
tags: [cookbook, modding, global, server-client]
dst_build: "steam-buildid:26230496"
sources: [mods.lua, modindex.lua, modutil.lua, util.lua, main.lua, mainfunctions.lua]
reliability: verified
validation: source-reviewed
status: active
---

# 模组起步与加载验证

## 适用条件

从空目录建立一个 **DST 世界／服务器模组**，先确认游戏识别并加载入口。本 demo 提供基础元信息与全局读取代理，不需要美术资源、配置项或自定义 Prefab。

已有模组先沿用其入口和目录约定，再读取需要扩展的场景。本页采用 `env → GLOBAL` 的读取代理作为骨架约定；它不是游戏加载所有模组的硬性要求。[环境入口](../../raw/current/scripts/mods.lua#L258)

## 最小 demo

### 文件与替换点

在目标 DST 安装目录的 `mods/` 下建立独立本地目录，例如：

```text
mods/mod_starter/
├── modinfo.lua
└── modmain.lua
```

保存为 UTF-8 文本，确认扩展名是 `.lua`。`modinfo.lua` 与 `modmain.lua` 直接位于模组根目录；入口按这个位置加载。[modindex.lua:608](../../raw/current/scripts/modindex.lua#L608)、[mods.lua:555](../../raw/current/scripts/mods.lua#L555

可替换目录名、展示名称、描述、作者与版本；目录名和展示名称不必相同。这个阶段只创建这两个文件，功能页再提供各自必要的目录和资源。

### modinfo.lua

完整内容：

```lua
name = "Mod Starter"
description = "Minimal DST mod framework."
author = "Your Name"
version = "0.1.0"

api_version = 10
dst_compatible = true
dont_starve_compatible = false
reign_of_giants_compatible = false

client_only_mod = false
all_clients_require_mod = false
```

当前快照的 `MOD_API_VERSION` 为 10；它不是 DST build 号。`name`、`description`、`author`、`version`、`api_version` 为元信息检查项；没有配置项时可以省略 `configuration_options`。[mods.lua:6](../../raw/current/scripts/mods.lua#L6)、[modindex.lua:634](../../raw/current/scripts/modindex.lua#L634

这组运行端标志对应本 demo 尚无客户端依赖的世界模组骨架：

| 需求 | `client_only_mod` | `all_clients_require_mod` |
|---|---|---|
| 本 demo：由世界启用，无客户端代码／资源依赖 | `false` | `false` |
| 后续功能要求每位玩家加载相同模组代码／资源 | `false` | `true` |
| 独立客户端功能，由本地玩家启用 | `true` | `false` |

按实际功能选择；角色、自定义网络对象或 UI 的具体需求需在对应场景重新确认。两个标志不能同时为 `true`。[分类](../../raw/current/scripts/modindex.lua#L144)、[客户端需求记录](../../raw/current/scripts/mods.lua#L111)、[互斥检查](../../raw/current/scripts/modindex.lua#L684)

### modmain.lua

完整内容：

```lua
GLOBAL.setmetatable(env, {
    __index = function(t, k)
        return GLOBAL.rawget(GLOBAL, k)
    end
})
```

`env` 已有字段优先；缺失字段才从 `GLOBAL` 读取。代理仅处理读取：`MY_VALUE = 1` 写入模组环境，若确需写游戏全局应显式写 `GLOBAL.MY_VALUE`；修改 `TUNING` 等共享表的字段仍会影响共享对象。游戏提供 `GLOBAL = _G`、`env.env = env`，并在该环境执行入口。[mods.lua:289-298](../../raw/current/scripts/mods.lua#L289-L298)、[util.lua:788](../../raw/current/scripts/util.lua#L788

后续功能按所需时机接入；全局读取代理不会使世界或玩家提前可用。此骨架不附加日志输出或空回调。

### 全局名称与加载环境

上面的代理执行后，同一模组环境可直接使用 `TheWorld`、`TheSim`、`TheNet`、`STRINGS`、`ACTIONS`、`EQUIPSLOTS`、`Vector3`、`SpawnPrefab`、`CreateEntity` 等游戏全局，无需再逐个加 `GLOBAL.`；前提是当前阶段与运行端确有该对象。已有同名环境字段或局部变量仍优先，不批量替换项目现有写法。无代理时，未注入的游戏名称用 `GLOBAL.TheWorld` 等显式访问，或给出实际局部绑定。[mods.lua:289](../../raw/current/scripts/mods.lua#L289

| 名称／文件 | 查找与使用边界 |
|---|---|
| `TUNING`、`Class`、`require`、`math`、`table` 等 | 已直接注入模组环境，无代理也可使用；`TUNING` 指向共享表。[mods.lua:269-278](../../raw/current/scripts/mods.lua#L269-L278) |
| `AddPrefabPostInit`、`AddPlayerPostInit`、`GetModConfigData`、`modname`、`MODROOT`、`modimport` | 属于模组环境的注册入口／上下文，不应改成 `GLOBAL.AddPrefabPostInit` 等。[mods.lua:289](../../raw/current/scripts/mods.lua#L289)、[modutil.lua:329](../../raw/current/scripts/modutil.lua#L329)、[modutil.lua:586](../../raw/current/scripts/modutil.lua#L586 |
| `modimport` 导入文件 | 共享模组 `env`，在代理建立后导入即可使用它；不会继承调用文件的局部变量。[mods.lua:301](../../raw/current/scripts/mods.lua#L301 |
| 普通 `require` 模块、`scripts/prefabs/*.lua` | 常规加载器在游戏全局环境执行，不继承调用方的模组 `env`、代理或局部变量；其中通常直接用 `TheWorld`／`Prefab` 等，不能假定有 `GLOBAL`、`modname` 或模组注册函数。所需模组上下文显式传入，自定义 `setfenv`／加载器另查实际实现。[main.lua:129](../../raw/current/scripts/main.lua#L129)、[mainfunctions.lua:171](../../raw/current/scripts/mainfunctions.lua#L171 |
| `modinfo.lua`、`modworldgenmain.lua` | 前者是独立元信息环境，无运行时 `GLOBAL`／`env` 注入；同一加载轮先执行 `modworldgenmain` 再执行适用的 `modmain`，不能假定后者的代理已生效；世界生成可用 API／对象也有阶段限制。[modindex.lua:599](../../raw/current/scripts/modindex.lua#L599)、[mods.lua:540](../../raw/current/scripts/mods.lua#L540)、[modutil.lua:430](../../raw/current/scripts/modutil.lua#L430 |

全局代理只改变读取查找：不会同步普通 Lua 字段，也不会推迟访问。顶层 `local TheWorld = GLOBAL.TheWorld` 会捕获绑定当时的值；需要世界的逻辑应在合适回调内读取，不能因名称可见就假定世界已创建。

### 验证加载

1. 重新进入游戏，在创建／编辑世界的服务器模组选项中启用该模组，启动测试世界；本 demo 不是客户端模组。
2. 在承载该世界的服务器日志中定位该模组的 `Loading modmain.lua`，检查同次启动是否出现对应的 `Error loading mod!` 或 `Mod had no modmain.lua. Skipping.`。这些是游戏加载器自身的日志，无需给骨架添加测试代码。[mods.lua:553](../../raw/current/scripts/mods.lua#L553
3. 能进入世界且没有该模组的加载错误，是起步检查的预期结果。修改代码后重新加载世界再验证；`Loading` 只说明开始加载，不单独证明执行成功或玩法有效。托管／独立服务器及地表／洞穴分别查看对应进程／分片日志，不以客户端缺少加载行判失败。

入口加载失败会进入失败记录；缺少入口时会打印跳过信息。[mods.lua:550](../../raw/current/scripts/mods.lua#L550)。以上是待执行的游戏检查步骤，本页未执行实际游戏测试。

## 按需求扩展

- **添加配置项、分组标题或区分本地名称**：按需读 [[reference/api/modinfo/configuration#配置项与分组标题|配置与标题]]、[[reference/api/modinfo/configuration#读取配置值|读取配置]] 或 [[reference/api/modinfo/configuration#本地开发名称与多语言|名称与语言]]；起步文件不预填全部选项。
- **功能变多，需要分模块**：将入口注册代码放入实际需要的文件，再在 `modmain.lua` 按依赖顺序调用 `modimport("modmain/feature.lua")`。路径相对模组根，导入文件共享该模组环境；不要把此规则推到 `require` 或其他加载方式。[mods.lua:301](../../raw/current/scripts/mods.lua#L301
- **新增道具、装备、角色、建筑或配方**：从 [[cookbook/index|场景导航]] 选择已交付骨架，按该页创建必要文件；尚无骨架时定向查相关原版实现。只在加入对象后按需读 [[reference/components/finiteuses#典型接入|按次消耗]]、[[reference/components/equippable#典型接入|装备效果]] 等机制。
- **接入自定义动画、贴图、声音或详细 API**：在对应对象／接口页确认资源加载与机制；本页无需添加 `Assets`、`PrefabFiles` 或资源制作工具。

## 边界与验证

| 触发条件 | 边界／处理与依据 |
|---|---|
| 将全局代理放进 `modinfo.lua` | 元信息使用独立环境，没有本页 `modmain` 的 `GLOBAL`／`env` 注入；保持元信息为数据声明。[modindex.lua:599](../../raw/current/scripts/modindex.lua#L599 |
| 读得到全局名字，就在顶层访问世界／玩家 | 读取代理只改变名称查找，不保证对象已初始化；入口环境只注入当阶段已有字段，功能按对应初始化时机接入。[mods.lua:294-295](../../raw/current/scripts/mods.lua#L294-L295) |
| 把运行时入口直接搬进世界生成入口 | `modworldgenmain.lua` 与 `modmain.lua` 的阶段及可用 API 有差别；本 demo 不需要前者，缺文件会跳过。世界生成 API 要单独核验。[mods.lua:540](../../raw/current/scripts/mods.lua#L540)、[modutil.lua:430](../../raw/current/scripts/modutil.lua#L430 |
| 扩展成客户端依赖功能，仍沿用两个 `false` | 重新确定运行端与客户端安装要求，并分别验证托管、独立服务器和远端客户端；标志不自动同步任意 Lua 字段。[mods.lua:111](../../raw/current/scripts/mods.lua#L111)、[版本检查](../../raw/current/scripts/mods.lua#L897) |

### 版本增量复核 26148036（2026-10-25）
处理范围：锚点重定位 16 处（含收口检查修正：读取代理表格行的 mods.lua 旧 297 实为 CreateEnvironment 行，漏迁后指向空行，改指条件注入段 294-295）（mods.lua 全文件 -39：环境注入／modimport／加载日志／版本检查等；modindex.lua +6；main.lua 加载器 -5；mainfunctions +26）；R43 核验：本页未引用 Release ID／`CurrentRelease` 门控，R-11 的注册表改版（mods.lua:13-15 改遍历 versioning.lua MODIDS、新增 R43_SHADOWYDEPTHS）不影响本页 `api_version`/运行端结论。依据：全局枢纽 diff 审阅＋变更清单命中 mods.lua、modindex.lua、util.lua、main.lua、mainfunctions.lua（util 本版仅 +1 行、不影响 786）。

验证记录（2026-10-11，Steam buildid 25643519）：已审阅元信息检查、入口环境／执行、`modimport` 与加载器日志／失败分支。两份 demo 经 Full Moon 3.0.0 的 Lua 5.1 语法与原字节往返检查；页面结构、链接、大小及导航登记另行检查。**未启动 DST**，模组列表识别、世界启动与联机／分片加载仍待实际验证。

补充审阅（2026-10-16，同一 build）：核对注入名称、`modimport` 的 `setfenv`、普通 `require` 与 Prefab 加载器及元信息／世界生成阶段；补充全局查找和局部绑定边界。未新增运行验证。
