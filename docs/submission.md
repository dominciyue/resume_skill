# 提交收录素材（投递到各 Agent Skills 合集）

把 offer-helper 提交到下列合集，是触达目标用户（求职的年轻人）的主渠道。
每个仓库的 list 格式略有不同，下面给好现成文案，按目标仓库选用。

## 目标合集（优先级从高到低）

1. **laolaoshiren/claude-code-skills-zh** —— 中文合集，最对口，受众重合度最高
2. **travisvn/awesome-claude-skills** —— 英文主力合集
3. **karanb192/awesome-claude-skills** —— 用 ⭐ 评级、强调 verified
4. **VoltAgent/awesome-agent-skills** —— 1000+ 跨运行时合集
5. **ComposioHQ/awesome-claude-skills**

---

## 一句话 list 条目（中文合集用）

```markdown
- [offer-helper（上岸 Offer 助手）](https://github.com/dominciyue/resume_skill) —
  中文求职 skill：摄入简历/GitHub 仓库/文档建立可持久化经历库，针对 JD 做
  STAR 量化简历（严格防虚构），并基于你的简历做大厂面试官式压力追问。
```

### 按目标仓库格式定制的条目（即贴即用）

**laolaoshiren/claude-code-skills-zh**（第三方技能表，分类放「🌏 中文专属」，
表格列 = 技能 / 说明）：
```markdown
| [offer-helper](https://github.com/dominciyue/resume_skill) | 中文求职助手：摄入简历/GitHub仓库/文档建经历库，按JD生成STAR量化简历（严格防虚构），并做大厂式由浅及深的深挖面试 |
```

**travisvn/awesome-claude-skills**（Community Skills → Individual Skills，
格式 `**[名称](链接)** - 描述`）：
```markdown
- **[offer-helper](https://github.com/dominciyue/resume_skill)** - Chinese job-hunting skill: ingests your resume / GitHub repos / docs into a persistent experience library, tailors STAR-quantified resumes to any JD (strict anti-fabrication), and runs big-tech-style progressive ("由浅及深") mock interviews grounded in your actual resume.
```

## 一句话 list 条目（英文合集用）

```markdown
- [offer-helper](https://github.com/dominciyue/resume_skill) — Chinese-language
  job-hunting skill. Ingests your resume / GitHub repos / docs into a persistent
  experience library, tailors STAR-quantified resumes to any JD (strict
  anti-fabrication), and runs big-tech-style pressure mock interviews grounded
  in your actual resume.
```

## 分类放置建议

多数合集按类别分节。offer-helper 应归入 **Career / Job Search / Writing /
Productivity** 一类；中文合集可放「生活/求职」或「文档写作」类目下。

---

## PR 标题

```
Add offer-helper — a Chinese job-hunting skill (resume tailoring + mock interview)
```

## PR 正文（英文）

```markdown
## What it is
offer-helper is a Chinese-language Agent Skill for job seekers (students &
early-career). It covers the full loop:

1. **Ingest** — pull your resume, GitHub repos, and other docs into one
   persistent, incrementally-updatable experience library.
2. **Tailor resume** — analyze a JD, rewrite matching experiences in quantified
   STAR form, output an A4 HTML resume + a match report.
3. **Mock interview** — a big-tech interviewer that probes your *actual* resume,
   surfacing claims that won't survive follow-up questions.

## Why it's different
- **Strict anti-fabrication**: every bullet must trace back to real source
  material; missing numbers stay blank rather than being invented. Verified by
  4 end-to-end acceptance scenarios in `tests/`.
- **Cross-runtime aware**: degrades honestly on Claude.ai web (no filesystem)
  vs. Claude Code / Cursor (persistent `~/offer-helper-data/`).
- Pure Markdown + one HTML template, no network code — auditable in minutes.

## Links
- Repo: https://github.com/dominciyue/resume_skill
- Walkthrough demo: examples/demo.md
- GitHub-repo intake example: examples/github-intake-example.md
```

## PR 正文（中文合集用）

```markdown
## 这是什么
offer-helper（上岸 Offer 助手）是一个面向求职者（学生 / 年轻职场人）的中文
求职 skill，覆盖完整闭环：

1. **摄入建库**：把简历 / GitHub 仓库 / 其他文档整合成一份可持久化、可增量
   更新的经历库。
2. **定制简历**：分析 JD，用 STAR 量化改写匹配经历，产出 A4 简历 + 匹配度报告。
3. **大厂面试**：面试官只针对你这份简历的具体内容层层追问，逼出撑不住的表述。

## 差异化
- **严格防虚构**：每条都要溯源到真实材料，缺数字宁可留白也不编造；含 4 套
  端到端验收场景。
- **跨运行时诚实降级**：区分有/无文件系统环境，网页版如实告知无法持久化。
- 纯 Markdown + 1 个 HTML 模板，无网络代码，可几分钟内审计。

## 链接
- 仓库：https://github.com/dominciyue/resume_skill
- 演示：examples/demo.md
```

---

## 提交前自查

- [ ] README 顶部有清晰的一句话定位与安装步骤 ✅
- [ ] 仓库可被 `cp -r offer-helper ~/.claude/skills/` 直接安装 ✅
- [ ] 有 LICENSE（MIT）✅
- [ ] 有可跑通的示例（examples/）✅
- [ ] 遵守目标合集的贡献规范（看其 CONTRIBUTING / PR 模板，按字母序或分类插入）
