# 编译与闸门（附录）

本文件是 `SKILL.md` 的算法附录。有写文件权限时，请按本节规则自己生成 `SKILL.md` 与 `skill-flow.json`；回归样本见同级 `examples/weekly-ops-report.*`。

## 1. 节点类型

| type | 中文 | 配齐条件 | 连线 |
| --- | --- | --- | --- |
| `start` | 开始 | `trigger` 或 `desc` | `nextIds`。全图保留一个 |
| `collect` | 收集 | `params` 或 `desc` | `nextIds`。默认 `waitUser: true` |
| `call` | 调用 | 已绑 MCP、其他 Skill 或脚本 | `nextIds`。一节点一目标 |
| `process` | 处理 | `action` 或 `desc` | `nextIds` |
| `produce` | 生成 | `template` 或 `desc` | `nextIds` |
| `decide` | 判断 | `cond` | `yesId` / `noId`（忽略 `nextIds`） |
| `join` | 汇聚 | `merge` 或 `desc` | `nextIds`。上游至少两路 |
| `each` | 逐条 | `listFrom`，以及 `eachDo` 或 `desc` | `nextIds`。无子画布 |
| `check` | 校验 | `checks` | `nextIds` |
| `end` | 结束 | `success` 或 `desc` | 无出边 |

未知 `type` 丢弃。`id` 空则用 `n{序号}`（从 1）。

调用绑定：

- `callKind` 为 `script`，或虽未写但有 `scriptRelativePath` → 脚本调用，必须有相对路径。
- 否则 MCP 类：`mcp` 或 `skillDep` 至少一个非空。
- 有 `skillDep` 且无 `mcp` → 编译「其他 Skill：{skillDep}」。
- 有 `mcp` 且有 `tool` → `工具：{mcp} / {tool}`；无 tool → `MCP：{mcp}`。

## 2. 闸门

`publishReady` 当且仅当 `blocking` 为空。警告不阻止发布标记，但要展示。草稿允许保存。

**阻断**

- 名称空白 → `未填写 Skill 名称`
- 描述空白 → `未填写做什么（描述）`
- `whenToUse` 空白且开始节点无 `trigger` → `未填写触发场景（何时启用）`
- 无有名称输出 → `至少定义一个输出产物`
- 无 `end` → `缺少结束节点`
- 脚本调用无路径 → `调用节点「{name}」未上传本地脚本`
- MCP 类调用无 mcp 且无 skillDep → `调用节点「{name}」未绑定 MCP 或其他 Skill`
- 判断无 `cond` → `判断节点「{name}」未写条件`

**警告**

- 契约与开始节点都无禁止 → `未填写禁止事项`
- 无有名称用户输入 → `未填写用户输入。数据若全部来自流程里的工具调用，可以不填`
- 无校验节点 → `建议至少有一个校验节点`
- 无调用节点 → `若需读数、读文件或跑脚本，请添加「调用」并选择 MCP、其他 Skill 或本地脚本`
- 逐条缺 listFrom 或（eachDo/desc）→ `逐条节点「{name}」未写列表来源或每项做什么`
- 非判断/校验的 `nextIds`>1 且下游走不到 join → `「{name}」并行了 {n} 路，建议用「汇聚」合并后再处理`
- join 入边少于 2 → `汇聚「{name}」上游不足两路，无法合并并行结果`

## 3. 编译 `SKILL.md`

遍历：从 `start` 深度优先。判断先 `yesId` 再 `noId`。环只出现一次。走不到的节点按原数组顺序补末尾。步骤跳过 `start`/`end`。

frontmatter：

- `name` = 锁定后的 slug（名称转 slug：去首尾空白→小写→空白变 `-`→去掉除字母数字、`_`、`-`、CJK 以外的字符；空则 `custom-skill`）
- `description` = `{description}。当用户提到「{whenToUse}」时使用。`；`whenToUse` 空则退回开始 `trigger`；都空则 `{description}。`，正文「何时启用」写 `（缺触发场景）`；description 空则用 `（请填写这个 Skill 做什么）`

禁止：契约 `forbid` → 开始节点 `forbid` → `（请填写禁止事项）`。

开始 `scope` 非空时，在「何时启用」与「禁止」之间插入「适用边界」。

输入行：`- {name}（{载体}，必填|选填）{样例子句}`  
载体：`文件`/`file`→「文件」；`文本`→「文本」；其他原样；空→`string`。`required !== true` 为选填。无有名称输入 → `- （无）`。

输出行：`- {name}（{载体}）：{quality}{样例子句}`  
空载体→`text`。无有名称输出 → `- （无）`。

样例子句：

- 文本且有 `example` → ` 样例文本：{example}`（前有空格）
- 文件且有相对路径 → ` 样例文件：{path}（相对本 Skill 目录）。说明：{fileNote 或 未填写}。执行时请读取该样例，对照结构、口径和版式完成。`
- 文件仅有 fileName → 路径说明改为 `{fileName}（保存后放在本 Skill 目录 samples/）`
- 文件未上传 → `样例文件：未上传`

步骤一行：`{序号}. 【{中文类型}】{节点名称}：{片段用「；」连接}`

片段顺序：`desc`、`收集：{params}`、`追问：{ask}`、`action`、自由度（低/中/高）、`模板：{template}`、调用绑定、可写声明（`access=write`）、`若 {cond}`、汇聚等待（默认「等待全部完成」；`joinMode=any`→「任一完成即可继续」）与 `合并：{merge}`、逐条 `对「{listFrom}」逐条处理：{eachDo 或 desc}`（`eachMode=parallel` 再加「条目之间可以并行」）、多于一条 nextIds 时 `并行下一步：{下游名称}`、校验清单原文、`等待用户输入后再继续`（collect 默认）、`失败：{fail}`。无片段 → `按节点说明执行`。

失败处理：结束节点 `failOut`；空则 `遇无权限、空结果或校验失败：终止并说明原因，不编造数据。`

`success` 非空时，在「失败处理」后空一行再写「完成标准」。

骨架：

```markdown
---
name: {slug}
description: {compiled description}
---

# {name}

## 何时启用
{whenToUse}

## 禁止
{forbid}

## 输入
{inputs}

## 步骤
{steps}

## 输出
{outputs}

## 失败处理
{failOut}

## 完成标准
{success}
```

（有适用边界时插在「何时启用」与「禁止」之间。无完成标准则省略该节及其前空行。）

**回归**：`examples/weekly-ops-report.draft.json` 编译结果必须与 `examples/weekly-ops-report.SKILL.md` 逐字一致。

## 4. 从包生成时的规范化

- 脚本路径不在清单 → 清空路径与 MCP 字段，节点名追加「（脚本待确认）」（已含「待确认」不重复）
- 有 `skillDep` 的调用保留，清空脚本字段
- `mcp` 空且非脚本 → 清工具名，节点名追加「（待选工具）」
- 有效节点为 0 → 报错，不要落盘

## 5. 落盘约定

目录 `<slug>/`：

```text
<slug>/
  SKILL.md           # 编译结果（运行时读）
  skill-flow.json    # 仅 { "nodes": [ ... ] }，无绝对路径
  manifest.json      # 可选：name/description/whenToUse/forbid/inputs/outputs/version/nodes 等契约原文
```

脚本后缀白名单：`.py .js .mjs .cjs .ts .sh .bash .ps1 .rb .php`。相对路径形如 `scripts/check.py`。
