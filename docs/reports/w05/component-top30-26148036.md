# 原版组件静态接入量 Top 30 试用

日期：2026-10-27。输入为知识根 `raw/current/scripts/`，`raw/current/BUILD.txt` 为 `26148036`。这是导航试用的工程记录，日常查询不加载；榜单展示维护在 [组件总索引](../../../wiki/reference/components/index.md#原版接入量-top-30)。

本报告保留 Top 30 首版时点的九主题／17 索引验收记录；后续主题细化至 16 个、全库索引至 24 篇，见 [主题细化](component-topic-refinement.md)，榜单统计不变。

## 统计口径与结果

- 扫描全部 4,084 个 `.lua` 文件，包括 prefabs、components 和公共 helper；不扫描历史快照或输入模组。
- 按 Lua 词法子集跳过行／长块注释和字符串内容，只识别 `:AddComponent("小写组件名")` 或单引号的完整直接调用，允许 token 之间有空白或注释。方法定义不计；匹配名称必须存在对应组件源码。
- 排名按不同调用文件数降序，同文件重复调用只计一次，同数按组件名升序。第 30 名 fueled 与第 31 名 pickable 都为 52 文件，依上述排序截取 30 项。
- 共识别 8,839 处字面量直接调用，涉及 773 个组件；两个动态参数调用位于 entityscript.lua 的 1961、1978 行，不归因给具体组件。easing.lua、nis/xbnyn.lua 非 UTF-8，以 Latin-1 保留字节扫描，没有跳过文件。
- 前三名为 inspectable 1,016／inventoryitem 522／lootdropper 455 文件；Top 30 中 29 篇 Wiki 已存在且 active，updatelooper 第 15 名（120 文件）尚无 Wiki，以现存源码入口展示，不用第 31 名补位。
- 这是源码静态分布，不表示运行触发次数、独立 Prefab 数、实体数量、查询频率或重要性。不会展开 MakeSmallBurnable 等公共 helper 的调用者或工厂变体，因此 burnable 等间接接入会被低估；不可据此宣布它们不常用。

## 导航试用与维护边界

根导航新增榜单直达链接，原人工精选的开发快捷入口继续保留。组件总索引用有序列表展示统计快捷入口，九个主题表缩成范围摘要；叶级索引、正文地址和旧章节均未移动，快捷链接不承担第二份叶级登记。

仅明确维护榜单或游戏版本维护涉及本统计时重新运行、审阅并更新日期／build；普通查询和写页不重新扫描源码。复现代码只读，既不调用游戏，也不重写索引、缓存或知识基线；未将它增加为工具箱能力或 Skill 的默认读取。

## 只读复现

PowerShell 中把 `<K>` 替换为知识根绝对路径；仅需已有 Python 标准库。输出保留计数和每个上榜组件前三个调用位置，用于核对榜单；不输出游戏源码正文。

```powershell
@'
import re,json,collections,pathlib,sys
K=pathlib.Path(sys.argv[1]).resolve()
root=K/"raw/current/scripts"
token=re.compile(r"--\[(?P<ceq>=*)\[.*?\](?P=ceq)\]|--[^\r\n]*|\[(?P<seq>=*)\[.*?\](?P=seq)\]|\"(?:\\.|[^\"\\])*\"|'(?:\\.|[^'\\])*'|[A-Za-z_][A-Za-z_0-9]*|[^\s]",re.S)
files=sorted(root.rglob("*.lua"))
hits=collections.defaultdict(set)
calls=collections.Counter()
locations=collections.defaultdict(list)
dynamic=[]
non_utf8=[]
for p in files:
    data=p.read_bytes()
    try:
        text=data.decode("utf-8-sig")
    except UnicodeDecodeError:
        non_utf8.append(p.relative_to(root).as_posix())
        text=data.decode("latin-1")
    ts=[m for m in token.finditer(text) if not m.group().startswith("--")]
    for i,m in enumerate(ts):
        if m.group()!="AddComponent" or i<1 or ts[i-1].group()!=":" or i+3>=len(ts) or ts[i+1].group()!="(":
            continue
        if i>=3 and ts[i-3].group()=="function":
            continue
        literal=re.fullmatch(r"""["']([a-z0-9_]+)["']""",ts[i+2].group())
        if not literal or ts[i+3].group()!=")":
            dynamic.append((p.relative_to(root).as_posix(),text.count("\n",0,m.start())+1))
            continue
        name=literal.group(1)
        if not (root/"components"/(name+".lua")).is_file():
            raise ValueError((name,p))
        rel=p.relative_to(root).as_posix()
        hits[name].add(rel)
        calls[name]+=1
        locations[name].append((rel,text.count("\n",0,m.start())+1))
ranked=sorted(hits,key=lambda x:(-len(hits[x]),x))
print(json.dumps({
    "lua_files": len(files), "non_utf8": non_utf8,
    "components": len(hits), "literal_calls": sum(calls.values()),
    "dynamic_calls": dynamic,
    "top30": [{"rank": i, "component": name, "files": len(hits[name]),
               "examples": locations[name][:3]}
              for i, name in enumerate(ranked[:30], 1)]
}, ensure_ascii=False, indent=2))
'@ | python -X utf8 -B - "<K>"
```

## 本轮验收

- 从本报告代码块实际复现统计，30 项顺序／文件数逐项与索引相同；关联检查确认 29 篇现存 Wiki 均为 active，唯一缺页为 updatelooper。
- 两张改动索引格式、大小、本地链接与标题锚点检查通过；全库导航 419 篇知识页、17 篇索引，遗漏／重复登记／父入口问题为零，原根导航快捷入口仍为 22 条。
- 组件总索引 66 行／5,529 UTF-8 字节；根导航 46 行／2,582 字节，均在 Skill 上限内。
- 榜单中唯一新增的源码链接为无行号的 updatelooper.lua，严格检查 1 处、零错误／警告；此结果仅证明文件可读。
- 查询工具直读榜单、主题导航和 inspectable 的快速判断章节均完整、无省略／截断，直接读取当前 Markdown、未使用缓存。已知名称继续可绕过榜单直读，主题浏览仍为原九路。
- 本轮未改游戏知识正文、来源或核验版本，不刷新知识页基线；未运行游戏或新增示例／临时文件，不声称已测得查询 token 收益。
