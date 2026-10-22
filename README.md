# Agent Skills

**给任意 Agent 用的可安装 Skill 合集。**  
拷一个目录 → 开口就能用。不绑定某家客户端，不要求专属宿主工具。

<p align="center">
  <a href="#skills">Skills</a> ·
  <a href="#快速开始">快速开始</a> ·
  <a href="#为什么做这个仓库">为什么做</a> ·
  <a href="#english">English</a>
</p>

---

## 为什么做这个仓库

很多 Skill 要么锁死在某一个产品里，要么只是一堆零散提示词。  
这里想做成另一回事：

- **一个目录 = 一个 Skill**：丢进 Agent 的 skills / 技能目录就能用  
- **对话即用**：能写文件就自动落盘；不能写就在聊天里导出完整文件  
- **契约清晰**：带 schema、黄金样例与编译规则，行为可回归  
- **可持续扩展**：新 Skill 放进 `skills/<name>/`，不影响已有目录  

如果你在做 Agent、在团队里沉淀流程，或每周都在重复解释同一套步骤——这些 Skill 就是为了把「再说一遍」变成「装上就能触发」。

---

## 快速开始

```bash
git clone https://github.com/cheng273214/agent-skills.git
```

只装你需要的那一个，例如流程 Skill：

1. 把 `skills/skill-flow` **整个文件夹**复制到你的 Agent 的 skills / 技能目录  
2. 在对话里说：

> 请按照流程化创建 skill，我的需求是：……

无需额外 MCP，无需私有 SDK。

---

## Skills

| Skill | 一句话 | 安装 |
| --- | --- | --- |
| **[skill-flow](skills/skill-flow/)** | 把重复工作收成结构化 Skill：采访 → 节点 → 闸门 → 编译 `SKILL.md` → 落盘 / 导出 | 复制 `skills/skill-flow` |

后续开源的 Skill 会继续出现在这张表里。

---

## skill-flow 亮点

**痛点：**「帮我做个 Skill」常常变成散文、触发说不清，或模型随手编一份无法复用的 `SKILL.md`。

**你得到的是：**

1. 结构化采访（目标、触发说法、禁止、输入、输出、边界）  
2. 固定十种流程节点，连线规则明确  
3. 发布闸门（阻断 / 警告），草稿也能先存  
4. 确定性编译：同一份草稿 → 同一份 `SKILL.md`（仓库内含黄金样例）

| 环境 | 行为 |
| --- | --- |
| Agent 能写文件 | 生成 `<slug>/`，写入 `SKILL.md` + `skill-flow.json` |
| 仅对话 | 给出可复制的完整文件内容 |

细节见 [`skills/skill-flow/`](skills/skill-flow/)。

---

## 适合谁

- 希望 Skill **跨客户端可移植** 的 Agent 开发者  
- 想把团队流程沉淀成可安装能力的人  
- 不想每周重复口述同一套工作流的重度用户  

---

## 仓库结构

```text
agent-skills/
  README.md
  LICENSE
  skills/
    skill-flow/     ← 安装这个目录即可使用流程 Skill
      SKILL.md
      schema/
      examples/
      reference/
    <next-skill>/   ← 你的下一个开源 Skill
```

每个 Skill 自包含：你可以只拿走一个文件夹，不必永远依赖整个 monorepo。

---

## 贡献

欢迎 Issue / PR，尤其是：

- 新增 `skills/<name>/`（需有清晰的 `SKILL.md`）  
- 更强的示例与回归样例  
- 用户可见文案的翻译  

请保持 Skill **去品牌、去单一客户端绑定**，让任何人都能安装。

---

## License

[MIT](LICENSE) — 自由使用；注明来源更佳。

---

<a id="english"></a>

## English

**Open collection of installable Agent Skills.**  
Copy one folder into your Agent’s skills directory and start chatting. No proprietary host tools.

### Featured: skill-flow

Turn recurring work into a real Skill: interview → flow nodes → publish gates → compile `SKILL.md` → write files when possible, otherwise export in chat.

```bash
git clone https://github.com/cheng273214/agent-skills.git
# copy skills/skill-flow into your Agent skills directory
```

More skills will ship under `skills/<name>/`. MIT licensed.

