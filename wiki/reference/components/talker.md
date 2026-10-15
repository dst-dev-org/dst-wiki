---
title: talker
created: 2026-10-21
updated: "2026-10-28"
type: component
summary: 复用玩家与 NPC 说话能力，选择 Say 或 Chatter，处理消息同步、过滤、动画和取消
aliases: [头顶文字, 角色说话, NPC 台词, Say, MakeChatter, 禁止说话]
tags: [component, callback, lifecycle, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/talker.lua, networking.lua, prefabs/player_common.lua, prefabs/pigman.lua, behaviours/chattynode.lua, stategraphs/SGwilson.lua, widgets/followtext.lua, entityscript.lua, modutil.lua, tuning.lua]
reliability: verified
validation: source-reviewed
status: active
---

# talker

## 快速判断

- 职责与适用：按行播放台词、生成跟随文字、触发说话事件。玩家／猪人优先复用已有组件，避免覆盖说话回调；构造依据见“典型接入”和下文 NPC 模式。
- 优先使用：服务端临时文本用 `Say`；NPC 按各客户端 `STRINGS` 解析台词用已初始化的 `Chatter`。两条路径见“关键接口”。
- 重要副作用：获准的 `Say` 会取消旧台词，不排队；取消也可触发 `donetalking`。`noanim` 由宿主消费者处理，不屏蔽整个 SG 或声音。[玩家 SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2155-L2180)。
- 修改边界：专服没有本地 HUD 文字；远端接收要求宿主已有 `components.talker`。原玩家在共同构造阶段加组件，在后面才 `SetPristine` 并分端返回。[玩家构造](../../../raw/current/scripts/prefabs/player_common.lua#L2487-L2488)、[端分界](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2614)。

## 关键接口

### Say 的输入、门禁与消息路径

完整签名为 `Say(script, time, noanim, force, nobroadcast, colour, text_filter_context, original_author_netid, onfinishedlinesfn, sgparam)`，无成功返回协议。常规接入只需前两个或三个参数。[实现](../../../raw/current/scripts/components/talker.lua#L267-L295)。

| 参数／状态 | 本版本实际约定 |
|---|---|
| `script` | 字符串转成单行；或按 `ipairs` 消费的行表，每行为 `{ message=文本, noanim=布尔值, duration=秒 }`。表模式的 `time/noanim` 不会覆盖各行；不需手动调用全局 `Line`。[Line](../../../raw/current/scripts/components/talker.lua#L5-L9)、[转换](../../../raw/current/scripts/components/talker.lua#L291-L294) |
| `time` / `lineduration` | 单行时间；每行优先 `duration`，其次组件 `lineduration`，再默认 2.5 秒，上限 8 秒。实现只取 `min`，没有下限保护；调用者应提供正数。[消费](../../../raw/current/scripts/components/talker.lua#L178-L179)、[常量](../../../raw/current/scripts/tuning.lua#L90-L91) |
| `force` | 绕过组件自身的忽略、死亡／睡眠等分支，但仍受 `TheWorld.speechdisabled` 拒绝；不保证状态机一定播放动画。服务端死亡例外是有 `revivablecorpse`；客户端查标签和 health replica，未重复查 sleeper。[门禁](../../../raw/current/scripts/components/talker.lua#L269-L289) |
| `nobroadcast` | 真则不调用 `TheNet:Talker`；网络接收函数正是用真值防止重播。普通服务端接入保持默认，不在每个客户端重复发起同一句话。[广播](../../../raw/current/scripts/components/talker.lua#L193-L199)、[Networking_Talk](../../../raw/current/scripts/networking.lua#L139-L143) |
| `colour` | 传给本地文字的颜色数组，经 `unpack` 使用；组件 `colour` 则读取 Vector3 的 `x/y/z` 并补 alpha=1。两者形状不同，且每次 Say 的颜色并未作为 `TheNet:Talker` 参数传出。[颜色](../../../raw/current/scripts/components/talker.lua#L167-L175)、[网络参数](../../../raw/current/scripts/components/talker.lua#L193-L199) |
| `IgnoreAll(source)` / `StopIgnoringAll(source)` | 按来源键引用管理；省略键时用组件自身。最后一个来源解除才去掉 `ignoretalking`。不主动取消已开始的线程；需要立刻停说另调用 `ShutUp`。[实现](../../../raw/current/scripts/components/talker.lua#L134-L156) |
| `IsTalking()` / `ShutUp()` | 前者只查当前 `task` 是否非空；后者取消本地线程／文字，并在服务器清 chatter 网络状态和清除任务。普通 Say 的停止没有单独的 `TheNet` 停止广播路径。[停止](../../../raw/current/scripts/components/talker.lua#L298-L314) |

一次普通台词的流程是：门禁 → 服务端 `ontalk(inst, script)` → 取消旧任务 → 新线程逐行广播原始 `line.message` → 本端加工／过滤并显示 → `ontalkfn(inst, {noanim, message})` → 宿主 `ontalk` 事件（`noanim/duration/sgparam`）→ 睡眠行时长 → 自然结束清文字、调用 `donetalkingfn`、推 `donetalking`。`onfinishedlinesfn(inst)` 只在最后一行等待结束后调用，取消路径不调用它。[门禁与入口回调](../../../raw/current/scripts/components/talker.lua#L267-L295)、[逐行与自然结束](../../../raw/current/scripts/components/talker.lua#L178-L246)、[取消](../../../raw/current/scripts/components/talker.lua#L249-L264)。

本地显示依次受 `mod_str_fn`、角色特殊处理、Wonkey 观察者乱码及 `speech_override_fn` 影响，再按上下文和原作者标识做词语过滤。缺省／UNKNOWN 在本地显示时归 GAME；玩家聊天接收明确传 CHAT 与 netid，接用户输入应保留对应来源。[加工与过滤](../../../raw/current/scripts/components/talker.lua#L180-L211)、[聊天接收](../../../raw/current/scripts/networking.lua#L62)。

### NPC 的 Chatter 与显示设置

`MakeChatter()` 建立五个网络变量并在客户端监听 dirty；猪人在 `SetPristine` 前完成它。`Chatter(strtbl, strid, time, forcetext, echotochatpriority)` 只在已初始化的主模拟端执行。`strtbl` 为 `STRINGS` 下点分路径，id 为 0 取最终值，非零取索引；解析失败会 `ShutUp`，自定义表须在所需客户端注册。[解析与 dirty](../../../raw/current/scripts/components/talker.lua#L26-L78)、[网络初始化／发送](../../../raw/current/scripts/components/talker.lua#L81-L123)、[猪人宿主](../../../raw/current/scripts/prefabs/pigman.lua#L624-L635)。

Chatter 强制使 id dirty，支持重复同一句；`forcetext=true` 同时作为 `Say` 的 `noanim/force`。发送后 1 秒任务只是 `strtbl:set_local("")`，不是台词时长，也不直接终止文字。时间和 id 使用 tinybyte/smallbyte 网络类型，勿当任意浮点时长或无限索引；本页不补造这些类型的编码范围。[dirty 消费](../../../raw/current/scripts/components/talker.lua#L53-L57)、[发送与清除](../../../raw/current/scripts/components/talker.lua#L86-L122)。

`echotochatpriority` 可用 `true` 映射 LOW、`false/nil` 映射 NOCHAT 或显式优先级；大于 0 时，有 HUD 且本地玩家在相机观察范围内才加入聊天记录。原版 `ChattyNode` 的字符串键分支选随机 id 后调用 `Chatter`，并不是 `Say` 的别名。[聊天记录条件](../../../raw/current/scripts/components/talker.lua#L59-L71)、[优先级转换](../../../raw/current/scripts/components/talker.lua#L114-L117)、[消费者](../../../raw/current/scripts/behaviours/chattynode.lua#L74-L87)。

本地显示字段有 `font/fontsize/colour/symbol/offset`；动态偏移用 `SetOffsetFn(fn)`，传入宿主。`disablefollowtext` 仅阻止创建文字。默认偏移 `(0,-400,0)` 经符号位置投影，不能当世界坐标；玩家已有动态偏移，优先保留。[配置](../../../raw/current/scripts/components/talker.lua#L160-L175)、[投影](../../../raw/current/scripts/widgets/followtext.lua#L49-L63)。

## 典型接入

### 玩家实体初始化后尝试说一句提示，保留已有说话机制

以下为已有服务端模组的完整 `modmain.lua` 代码；无额外资源，沿用原版玩家 talker，不重设 `ontalk`、字体、偏移或 SG。模组元信息与名称环境见 [[cookbook/mod-starter#全局名称与加载环境]]。注册入口按玩家标签筛选；服务器延迟一秒尝试一次，已有台词时跳过，不重试。[注册](../../../raw/current/scripts/modutil.lua#L586-L589)、[原宿主](../../../raw/current/scripts/prefabs/player_common.lua#L2487-L2488)。

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst:DoTaskInTime(1, function(player)
        local talker = player.components.talker
        if talker ~= nil and not talker:IsTalking() then
            talker:Say("准备出发。", 2.5, true)
        end
    end)
end)
```

替换文本和正数秒数即可。`true` 是第三参数 `noanim`，保留默认门禁与广播；不用 `force` 硬穿过原版忽略条件。触发点是**玩家实体初始化**，包括加载、重新生成等，不是“账号只欢迎一次”；若需要真正入服／业务事件，应把调用移到已核验的业务入口。该任务由实体登记，实体移除会取消待执行任务；文字自然结束时自行清理。[任务登记](../../../raw/current/scripts/entityscript.lua#L1547-L1554)、[实体清理](../../../raw/current/scripts/entityscript.lua#L1739-L1750)、[文字结束](../../../raw/current/scripts/components/talker.lua#L236-L246)。

退出并禁用模组再重载即撤销这条提示，原版 talker 保留；不提供热卸载。专服＋远端客户端、初始化时已有台词、全局禁言和死亡／睡眠应分别验证；本例未做游戏运行，延迟一秒也不是客户端 HUD 已就绪的保证。

## 常见陷阱与限制

- **同名回调职责不同**：`ontalk` 在服务端获准 Say 时执行，`ontalkfn` 在本地逐行线程执行，`ontalk` 事件由 SG 等消费者处理。猪人的 `ontalk` 会播放哼声；直接覆盖会丢原功能。[猪人回调](../../../raw/current/scripts/prefabs/pigman.lua#L44-L46)、[安装](../../../raw/current/scripts/prefabs/pigman.lua#L646-L647)、[组件调用点](../../../raw/current/scripts/components/talker.lua#L214-L218)。
- **新宿主只加服务器组件**：`Networking_Talk` 找不到客户端 talker 就跳过；新增宿主应参考上述共同构造模式。[接收条件](../../../raw/current/scripts/networking.lua#L139-L143)。
- **本地中止不是全网中止承诺**：普通 Say 不把 `force/noanim/colour/sgparam` 全部打包广播，远端由 `Networking_Talk` 用默认值再说；不能推导出服务端强制标志或动画参数会同步。[广播实参](../../../raw/current/scripts/components/talker.lua#L193-L199)、[接收实参](../../../raw/current/scripts/networking.lua#L139-L143)。
- **清理路径有区别**：移除组件时退订 dirty、移除忽略标签并 `ShutUp`；实体移除用 `CancelSay`，实体通用清理另取消 pending tasks。自定义外部监听／任务仍由其创建者负责，不能把说话组件当通用任务管理器。[组件清理](../../../raw/current/scripts/components/talker.lua#L126-L131)、[实体回调](../../../raw/current/scripts/components/talker.lua#L314)、[通用清理](../../../raw/current/scripts/entityscript.lua#L1741-L1750)。

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 30 处（talker.lua chatter→聊天记录新增 silent 距离标记使 +1 偏移、SGwilson＋49、player_common−12、pigman＋2、chattynode＋6、entityscript＋3；聊天收录条件与台词链结论不变）；结论修订：无，仅锚点迁移。
- 依据：全局枢纽 hunk 地图（R-08）＋变更清单命中 components/talker.lua、stategraphs/SGwilson.lua、prefabs/player_common.lua、prefabs/pigman.lua、behaviours/chattynode.lua、networking.lua、entityscript.lua。

- 2026-10-21，`steam-buildid:25643519`：审阅组件全文、玩家／猪人共同构造和端边界、普通网络接收、ChattyNode 消费者、玩家 SG 的说话事件、FollowText 投影及实体任务清理。引擎 `TheNet:Talker` 的内部路由与精确可见距离未作实现核验，不据此宣称全服广播。
- 完整 `modmain.lua` 示例通过 Lua 5.1 语法检查；本页元数据、大小、链接及源码行锚点通过指定页检查。
- 未运行游戏；多语言、输入过滤表现、客户端视觉／动画及入服时序未做运行验证。没有将单一示例的语法通过升级为组件运行通过。

## 相关链接

- [[reference/components/storyteller]]：对故事道具播剧本的讲述端。
