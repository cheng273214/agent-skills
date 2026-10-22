---
name: skill-flow
description: "一键流程化创建与维护 Skill。用户要新建 Skill、流程化创建skill、把重复工作收成步骤、画布式编排、从已有 Skill 包生成流程、编译/导出 SKILL.md、修改触发描述时使用。即使用户只说「做个 Skill」「写成技能」「以后都这样」「flow skill」「skill canvas」，也先用本 Skill。无需专属宿主工具；有写文件权限则落盘，否则在对话中给出完整文件。"
version: "2.0.0"
---

# 流程化创建 Skill（一键版）

你是任意 Agent 上「用流程契约创建/修改 Skill」的统一入口。安装本目录即可使用，**不依赖任何专属宿主工具**。

你的工作：问清需求 → 收成契约与节点 → 跑闸门 → **按本包规则编译 `SKILL.md`** → 有写文件能力则落盘，否则在对话里给出可复制全文。不要替用户去做那件业务本身。

详细编译表、闸门原文与回归约定见同目录 `reference/compile-and-gates.md`（可读则遵循；读不到则以本节摘要为准）。契约字段见 `schema/skill-flow-draft.schema.json`。黄金样例：`examples/weekly-ops-report.*`。

## 何时不要做成 Skill

一次就能做完、没有下次还会遇到的事，直接做，不要做成 Skill。稳定但很短的偏好优先记成记忆。重复出现的步骤、输入输出清楚、希望下次一句话就触发的，才做成流程 Skill。

## 模式选择

1. **新建 / 修改（对话）**：用户描述需求或要改已有流程 → 走「采访 → 节点 → 闸门 → 编译 → 持久化」。
2. **从包生成**：用户给出已有 Skill 目录或 `SKILL.md` 全文，且没有可用的 `skill-flow.json`（不存在或 `nodes` 为空）→ 走「从包生成」。已有非空画布时禁止覆盖，改为在其上修改。

## 先问清楚（新建）

缺了就追问，不要编造。对话里已有的步骤、工具和纠正先提取再补缺口。

- **目标**：这个 Skill 让 Agent 做什么（`description` / `name`）
- **触发说法**：用户真实会怎么说（写入 `whenToUse`；要具体，例如「经营周报、周报数据、本周汇总」而不是只写「处理周报」）
- **禁止事项**（`forbid`）
- **输入**：只写用户运行前必须提供的内容，可以没有。工具返回的列表/记录/字段是流程内部数据，不要写成输入
- **输出**：至少一个有名称的输出；有固定结构则写入生成节点 `template`
- **边界**：空结果、无权限、参数不齐时怎么停（结束节点 `failOut` / `success`）

## 节点规则

只使用十种类型：`start` `collect` `call` `process` `produce` `decide` `join` `each` `check` `end`。

- 保留唯一 `start`
- 调用只绑一个目标：已知 MCP/Tool、其他 Skill 名、或包内已有脚本相对路径。不要臆造不存在的工具或脚本
- 多工具并行：上游 `nextIds` 多路，再 `join`（上游至少两路）
- 逐条只写 `listFrom` + `eachDo`（或 `desc`），无子画布
- 连线：判断用 `yesId`/`noId`；其余用 `nextIds`；不要把节点 id 写进 `success`/`failOut`/`fail`/`yesTo`/`noTo` 等说明字段
- 纵向排布：默认 `x=280`，下一节点 `y = 上一.y + 132`；卡片约 228×86
- 不做误导、伪装能力或帮助未授权访问的 Skill

## 闸门（保存前必跑）

在对话中列出 `blocking` 与 `warnings`。`publishReady = (blocking 为空)`。用户要存草稿时仍可持久化，但必须说明还缺什么。

阻断摘要：无名称；无描述；无 whenToUse 且开始无 trigger；无有名称输出；无 end；脚本调用无路径；MCP 类调用无 mcp 且无 skillDep；判断无 cond。

警告摘要：无禁止；无用户输入；无校验；无调用；逐条缺来源/动作；并行未汇聚；join 上游不足两路。

完整条文见 `reference/compile-and-gates.md` §2。

## 编译 `SKILL.md`（你必须自己做）

本版**没有**外部编译器。你按下列规则生成全文，且对 `examples/weekly-ops-report.draft.json` 的编译结果必须与 `examples/weekly-ops-report.SKILL.md` 一致。

1. 从 `start` DFS；判断先 yes 后 no；步骤跳过 start/end。
2. frontmatter `name` = slug；`description` = `{description}。当用户提到「{whenToUse}」时使用。`（缺触发则仅 `{description}。`）。
3. 章节顺序：何时启用 →（可选适用边界）→ 禁止 → 输入 → 步骤 → 输出 → 失败处理 →（可选完成标准）。
4. 步骤行：`{n}. 【{中文类型}】{名称}：{片段用；连接}`。片段顺序与调用绑定文案见附录 §3。
5. 失败处理默认：`遇无权限、空结果或校验失败：终止并说明原因，不编造数据。`
6. 有 `success` 时，「完成标准」前留一空行。

完整表格、样例子句、slug 算法见 `reference/compile-and-gates.md` §3。

## 持久化

### 文件模式（优先）

当环境提供写文件/创建目录能力时：

1. 确定 slug（首次由名称生成并锁定）。
2. 创建目录 `<slug>/`（相对用户指定的 skills 目录，或当前工作区约定位置；**禁止本机绝对路径写进 JSON**）。
3. 写入：
   - `SKILL.md` — 编译结果
   - `skill-flow.json` — 仅 `{"nodes":[...]}`
   - 可选 `manifest.json` — 契约原文：`slug` `name` `description` `whenToUse` `forbid` `inputs` `outputs` `version`（从 `1.0.0`）`nodes` 等
4. 回报：路径、`publishReady`、阻断与警告；再给 2～3 句真实触发说法请用户确认是否覆盖场景。

### 对话模式（无写文件时）

依次给出可复制的 fenced 文件（标明建议文件名），至少：

1. `SKILL.md`
2. `skill-flow.json`
3. 可选 `manifest.json`

说明用户应把它们放进同一 Skill 目录。不要假装已经写入磁盘。

### 修改已有

若用户指出已有目录：先读其中 `skill-flow.json` / `manifest.json` / `SKILL.md`，在原契约上改，保留 slug；再编译并按上面两种模式之一写回或导出。

## 从包生成（无画布时）

适用：有 `SKILL.md`（或用户粘贴全文）+ 文件清单，但没有非空 `skill-flow.json`。

1. 忠实映射步骤与禁区到十种节点；含糊处用 `collect`/`check`，勿脑补业务。
2. 清单内脚本且说明提到脚本 → `call` + `scriptRelativePath`；只提 MCP → `call` + mcp/tool；都看不清 → `process` +「待确认」，勿假造 Tool。
3. 规范化：清单外脚本 → 清空路径，名称加「（脚本待确认）」；mcp 空且非脚本 → 名称加「（待选工具）」。
4. 先用三行中文摘要（节点数 / 待确认调用数 / 请确认后再保存），再给出单一 JSON（含 `nodes`，或节点数组）。
5. **展示给用户确认后**才进入持久化。不要静默覆盖已有画布。

## 明确不要做的事

- 不要声称存在或去调用任何「流程专用」宿主 API；只用当前 Agent 已有的通用能力（读/写文件、对话）。
- 不要把工具返回数据写成用户输入。
- 不要输出无法映射到十种节点的私有图形语言。
- 品牌中立：不要引入特定客户端或厂商名称。
