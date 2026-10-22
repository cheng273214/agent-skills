# skill-flow

把一段说不清的流程、SOP 或工作流，收成一个结构化、可安装的 Agent Skill。

你用对话把做法讲完。skill-flow 先补齐触发说法、禁止、输入、输出和失败时怎么停，再收成固定节点，写成符合 [草稿 schema](skills/skill-flow/schema/skill-flow-draft.schema.json) 的契约。然后按编译规则生成 `SKILL.md`，并跑发布闸门。宿主能写文件，就落盘；不能写，就在对话里导出完整文件。同一份草稿按规则得到同一份 `SKILL.md`，仓库里的黄金样例可以逐字对照。

它不替你做那件业务。它做的是把「以后都这样」变成一个下次能触发的技能。

## 适合谁，什么时候用

同一套步骤会反复出现，材料却还是口述、旧 SOP、工单备注或一份散文式的 `SKILL.md`。例如周报、审批、发布检查、客服升级、数据修正。

适合交给 skill-flow：

- 流程有禁止项、输入输出和失败边界，漏掉就会做错
- 换一场对话，仍要用同一套触发说法执行
- 已经有 `SKILL.md`，但没有流程契约，想先收成节点再改

一次就能做完的事，直接做。很短、很稳定的偏好，记成记忆即可。

## 相对提示词或随手文档

一段提示词能把事情说一遍。skill-flow 每次交出同一类产物，并且能指出还不能发布的原因。

| | 提示词或随手文档 | skill-flow |
| --- | --- | --- |
| 下次怎么触发 | 重新粘贴、重新解释 | `whenToUse` 写进 frontmatter。触发说法用用户真实会说的话，例如「经营周报、周报数据、本周汇总」 |
| 步骤会不会漂 | 每次重新组织 | 只用十种节点：`start` `collect` `call` `process` `produce` `decide` `join` `each` `check` `end`。同一份草稿编译出同一份 `SKILL.md` |
| 怎样算做完 | 读起来通顺 | `publishReady` 仅当阻断为空。缺名称、缺描述、缺触发、没有有名称的输出、没有结束节点、脚本调用没有路径、调用未绑定、判断没写条件，都会阻断 |
| 信息不够 | 模型把缺口补上 | 缺了就追问，不编造。草稿可以先存，同时列出阻断和警告 |
| 宿主不能写文件 | 内容停在聊天里 | 按顺序给出可复制的 `SKILL.md`、`skill-flow.json`，以及可选的 `manifest.json`，标明文件名，并说明尚未写入磁盘 |
| 已有 Skill | 手工改散文 | 没有非空 `skill-flow.json` 时，从包生成节点，确认后才保存。已有画布则在其上修改，不覆盖 |

闸门原文、编译表和 slug 规则在 [`skills/skill-flow/reference/compile-and-gates.md`](skills/skill-flow/reference/compile-and-gates.md)。

## 快速开始

技能目录是 [`skills/skill-flow`](skills/skill-flow/)。安装整个目录：编译规则、schema 和黄金样例都在 `SKILL.md` 旁边。

```bash
npx skills add cheng273214/agent-skills --skill skill-flow
```

命令会让你选择要装到哪个 Agent。加 `-g` 则对所有项目可用，否则装进当前项目。

装好后可以直接说：

> 请按照流程化创建 skill，我的需求是：每周按区域汇总经营数据并写出周报。缺的先问我，不要编。编译完告诉我 publishReady、阻断和警告。这里如果写不了文件，把完整文件导出。

> 这份 SKILL.md 还没有 skill-flow.json。从包生成流程，先给我三行摘要，我确认后再保存。

第一条对应仓库里的黄金样例。[`examples/weekly-ops-report.draft.json`](skills/skill-flow/examples/weekly-ops-report.draft.json) 的编译结果与 [`examples/weekly-ops-report.SKILL.md`](skills/skill-flow/examples/weekly-ops-report.SKILL.md) 逐字一致。一轮做完，技能是这样的：

- **何时启用**：经营周报、周报数据、本周汇总，或要按区域出周报
- **禁止**：不得编造工具未返回的数字；无权限或空结果时停止
- **输入**：区域（文本，必填），样例「华东」
- **步骤**：收集区域 → 调用「经营数据 / 查询周报」→ 按区域汇总销售额与异常 → 按模板写周报 → 校验每个数字都能回溯到工具返回
- **输出**：周报（文本），含区域、本周要点和异常项
- **失败处理**：缺权限或空结果时说明原因并停止
- **完成标准**：向用户交付周报正文

能写文件时，黄金样例锁定的 slug 是 `weekly-ops-report`（给人看的名称是「经营周报」），目录是：

```text
weekly-ops-report/
  SKILL.md            # 编译结果
  skill-flow.json     # 只有 nodes
  manifest.json       # 可选，契约原文
```

回报里会有路径、`publishReady`、阻断、警告，以及 2～3 句真实触发说法，用来确认是否覆盖实际说法。

只说「做个周报」、没有有名称的输出时，闸门阻断 `至少定义一个输出产物`，`publishReady` 为假。这份草稿仍可保存，回复里会写明还缺什么。

## 本仓库

目前这里的技能是 skill-flow。以后若增加别的技能，会放在 `skills/` 下。

## 许可证

[MIT](LICENSE)

---

<a id="english"></a>

## English

skill-flow turns a messy process, SOP, or workflow into a structured Agent Skill you can install.

It interviews you for real trigger phrases, forbids, inputs, outputs, and what to do on failure, then maps the work onto a fixed set of nodes and a draft that matches the [schema](skills/skill-flow/schema/skill-flow-draft.schema.json). Compile rules then generate `SKILL.md`, and publish gates run on that draft. When the host can write files, it writes them. When it cannot, it exports the full files in chat. The same draft, under those rules, yields the same `SKILL.md`. This repo includes a golden example you can diff against.

It does not perform the business task. It turns “do it this way from now on” into a skill the next chat can trigger.

### Who it's for

Use it when the same procedure will run again, and the source is still a spoken explanation, an old SOP, ticket notes, or a prose `SKILL.md`. Weekly reports, approvals, release checks, support escalation, and data fixes are typical.

Use it when:

- A missing forbid, input, output, or failure rule changes the outcome
- A new chat should fire from the same trigger phrases
- A `SKILL.md` already exists and you want an editable flow before changing it

A task that finishes in one turn can be done directly. A short, stable preference belongs in memory.

### What stays fixed

| | Prompt or ad-hoc doc | skill-flow |
| --- | --- | --- |
| Next trigger | Paste or re-explain | `whenToUse` is compiled into frontmatter, using phrases people actually say |
| Drift | Reorganized every chat | Ten node types only. The same draft compiles to the same `SKILL.md` |
| Done | The prose reads well | `publishReady` is true only when blocking findings are empty: missing name, description, trigger, named output, or end node; a script call without a path; an unbound call; a decision with no condition |
| Gaps | The model fills them in | It asks. It does not invent. A draft can still be saved, with blocking items and warnings listed |
| Host cannot write files | The text stays in chat | Copyable `SKILL.md`, `skill-flow.json`, and optional `manifest.json`, each with a filename, plus an explicit note that nothing was written |
| Existing skill | Hand-edit the prose | With no non-empty `skill-flow.json`, it builds nodes from the package and waits for confirmation. An existing canvas is edited, not replaced |

Gate text, the compile table, and the slug rule: [`skills/skill-flow/reference/compile-and-gates.md`](skills/skill-flow/reference/compile-and-gates.md).

### Quick start

The skill is [`skills/skill-flow`](skills/skill-flow/). Install the whole directory. The compile rules, schema, and golden example sit next to `SKILL.md`.

```bash
npx skills add cheng273214/agent-skills --skill skill-flow
```

The command asks which agent to install into. Add `-g` to install for every project; omit it to install into the current project.

Then say:

> 请按照流程化创建 skill，我的需求是：每周按区域汇总经营数据并写出周报。缺的先问我，不要编。编译完告诉我 publishReady、阻断和警告。这里如果写不了文件，把完整文件导出。

> 这份 SKILL.md 还没有 skill-flow.json。从包生成流程，先给我三行摘要，我确认后再保存。

The first utterance is the repo’s golden example. Compiling [`examples/weekly-ops-report.draft.json`](skills/skill-flow/examples/weekly-ops-report.draft.json) matches [`examples/weekly-ops-report.SKILL.md`](skills/skill-flow/examples/weekly-ops-report.SKILL.md) byte for byte. A finished round looks like this:

- **When to use**: 经营周报、周报数据、本周汇总，或要按区域出周报
- **Forbid**: do not invent numbers the tool did not return; stop on missing permission or an empty result
- **Input**: 区域 (text, required), example「华东」
- **Steps**: collect the region → call「经营数据 / 查询周报」→ summarize sales and exceptions by region → write the report from the template → check that every number traces back to the tool
- **Output**: 周报 (text), with region, weekly points, and exceptions
- **On failure**: stop and say why; do not invent numbers
- **Done when**: the report body has been delivered

When files can be written, the golden example locks the slug to `weekly-ops-report` (display name「经营周报」). The directory is:

```text
weekly-ops-report/
  SKILL.md            # compiled skill
  skill-flow.json     # nodes only
  manifest.json       # optional contract
```

The reply includes the path, `publishReady`, blocking items, warnings, and two or three real trigger phrases to confirm.

“做个周报” with no named output is blocking: `至少定义一个输出产物`. `publishReady` stays false. The draft can still be saved, and the reply says what is missing.

### This repo

skill-flow is the skill in this repository. Any later skills will live under `skills/`.

### License

[MIT](LICENSE)
