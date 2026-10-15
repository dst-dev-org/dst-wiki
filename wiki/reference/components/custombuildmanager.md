---
title: "custombuildmanager"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "维护分组到build的选择，并按组内同名symbol批量覆盖AnimState；说明显式刷新、清组不清覆盖和配置存读边界。"
aliases: ["custombuildmanager", "分组替换动画符号", "group到build覆盖", "清组不清符号"]
tags: [component, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/custombuildmanager.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# custombuildmanager

## 快速判断

- 职责与适用：在同一宿主上保存`groups[group]={symbol,...}`与`current[group]=build`，刷新时调用`AnimState:OverrideSymbol(symbol,build,symbol)`；目标symbol与来源symbol同名，不负责加载资源或选择Prefab皮肤。[custombuildmanager.lua:1-21](../../../raw/current/scripts/components/custombuildmanager.lua#L1-L21)
- 优先使用：已有明确分组和对应build资源、需要按组批量重写符号时核对下列接口；当前快照未找到组件之外的原版Lua安装／方法调用，不据此补造原版运行接线。[custombuildmanager.lua:23-40](../../../raw/current/scripts/components/custombuildmanager.lua#L23-L40)
- 重要副作用：清除current某组仅停止以后为该组写覆盖，不清理之前的AnimState覆盖；ClearOverrideSymbol在源码中被注释掉。[custombuildmanager.lua:10-18](../../../raw/current/scripts/components/custombuildmanager.lua#L10-L18) [custombuildmanager.lua:31-40](../../../raw/current/scripts/components/custombuildmanager.lua#L31-L40)
- 修改边界：实现没有端断言、网络变量、更新循环或删除恢复钩子；接入端和撤销应由宿主决定，不能当成已核验的玩家换装或同步系统。[custombuildmanager.lua:1-58](../../../raw/current/scripts/components/custombuildmanager.lua#L1-L58)

## 关键接口

| 接口／字段 | 签名或约束 | 条件、作用与证据 |
|---|---|---|
| `SetGroups(data)` | 组键→symbol数组的表 | 直接保存传入表引用，不复制、不验证、不刷新，也不清current旧键或先前覆盖。[custombuildmanager.lua:23-25](../../../raw/current/scripts/components/custombuildmanager.lua#L23-L25) |
| `SetCanSwapSymbol(fn)` | 可选`fn(inst)` | 保存回调，不刷新；刷新时对每个有current build的symbol分别调用，不传group／symbol，返回真值才覆盖。不是每组只调用一次。[custombuildmanager.lua:7-21](../../../raw/current/scripts/components/custombuildmanager.lua#L7-L21) [custombuildmanager.lua:27-29](../../../raw/current/scripts/components/custombuildmanager.lua#L27-L29) |
| `ChangeGroup(group,build)` | 已登记组键；build为覆盖来源 | 已知组且build为真值时写current，nil／false时删该键；未知组不改current，但无论是否改值都刷新所有组。[custombuildmanager.lua:31-40](../../../raw/current/scripts/components/custombuildmanager.lua#L31-L40) |
| `refreshart()` | 无参数；需要宿主AnimState | 对groups用pairs、组内数组用ipairs遍历；读取current并通过可选回调后写同名symbol覆盖。没有build或回调拒绝时不清旧覆盖。[custombuildmanager.lua:7-21](../../../raw/current/scripts/components/custombuildmanager.lua#L7-L21) |

## 典型接入

当前快照定向检索`custombuildmanager`及SetGroups／SetCanSwapSymbol／ChangeGroup，仅组件自身命中；debugcommands也没有实际引用。本页提供接口接线顺序说明，不提供缺少宿主依据的可运行demo。

若已有自定义宿主决定采用此组件，需先准备AnimState与实际build资源，注册完整分组表，再设置可选允许回调和组选择；SetGroups／SetCanSwapSymbol后需要显式refreshart或由ChangeGroup触发刷新。回调只收到inst，若需要按symbol判断，应另行设计接入，不能假设组件会传当前symbol。[custombuildmanager.lua:7-40](../../../raw/current/scripts/components/custombuildmanager.lua#L7-L40)

撤销某组或更换分组定义前，宿主应按自身覆盖归属明确恢复相关symbol；本组件既不保存原覆盖，也不调用ClearOverrideSymbol，直接Clear也可能影响其他来源，不能把ChangeGroup(group,nil)当恢复旧外观的协议。[custombuildmanager.lua:10-18](../../../raw/current/scripts/components/custombuildmanager.lua#L10-L18) [custombuildmanager.lua:23-40](../../../raw/current/scripts/components/custombuildmanager.lua#L23-L40)

## 常见陷阱与限制

- **一个symbol出现在多个组有固定优先级：** 外层用pairs，无排序或优先级规则；多个有build的组可能反复覆盖同一symbol，不应依赖最后胜出的组。[custombuildmanager.lua:7-20](../../../raw/current/scripts/components/custombuildmanager.lua#L7-L20)
- **回调改为false就会撤销旧外观：** 它仅拦截本次OverrideSymbol，既不自动刷新，也不清除既有覆盖；回调还可能在同次刷新按symbol多次执行，副作用需自行控制。[custombuildmanager.lua:7-21](../../../raw/current/scripts/components/custombuildmanager.lua#L7-L21) [custombuildmanager.lua:27-29](../../../raw/current/scripts/components/custombuildmanager.lua#L27-L29)
- **存档会保存分组与回调：** OnSave仅返回`{current=self.current}`，不保存groups或允许函数；OnLoad接收data.current后立即刷新。宿主需重建分组／回调，否则保存的build选择不会自动定义哪些symbol应被覆盖。[custombuildmanager.lua:42-56](../../../raw/current/scripts/components/custombuildmanager.lua#L42-L56)
- **移除组件就恢复所有覆盖：** 组件没有OnRemoveFromEntity或原值备份，不能依赖删除自动恢复AnimState状态。[custombuildmanager.lua:1-58](../../../raw/current/scripts/components/custombuildmanager.lua#L1-L58)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件全文；定向检索组件标识及方法调用，并检查debugcommands对应标识，未定位原版Lua接入点。覆盖分组遍历、逐symbol回调、显式刷新、清组残留与存读范围。
- 游戏运行：未运行；未测资源不存在、跨组同名覆盖、宿主端与同步、读档接线或外部模组消费者。未定位Lua调用不代表引擎／其他版本一定没有使用。
- 结构检查：`validate_page.py`通过；两页合并执行`validate_source_links.py --strict`，41个行号引用、0错误／0警告。本页无新增Lua示例；机械检查不代替游戏测试。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。