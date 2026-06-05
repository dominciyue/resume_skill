# offer-helper（上岸 Offer 助手）Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 构建一个中文求职 Agent Skill：经历库一次填写，每个 JD 秒级定制简历（STAR 量化改写 + 防虚构），并提供反向追问式压力模拟面试。

**Architecture:** 纯文档型 skill（Markdown 指令 + HTML 简历模板），无外部 API。`SKILL.md` 做入口路由（建库 / 简历 / 面试三条流程），细则下沉到 `resume/`、`interview/` 子文件按需加载；`experience/profile.md` 是用户本地经历库，为两个模块共享的数据中枢。

**Tech Stack:** Agent Skills 开放标准（SKILL.md + YAML frontmatter）、HTML/CSS（A4 打印简历模板）、Git。

**Repo layout:**

```
/home/ygwang/skill/
├─ offer-helper/                 # skill 本体（用户安装时复制此目录）
│  ├─ SKILL.md
│  ├─ resume/{jd-analysis,star-rules,anti-fabrication}.md
│  ├─ interview/{pressure,feedback}.md
│  ├─ templates/resume.html
│  └─ experience/profile.template.md
├─ tests/
│  ├─ fixtures/                  # 3 套「经历库 × JD」验收数据
│  └─ acceptance.md              # 端到端验收场景与判定标准
├─ docs/superpowers/{specs,plans}/
└─ README.md                     # 中文，安装/用法/隐私说明
```

**质量红线（来自调研）：** SKILL.md < 300 行；每个子文件单一职责；所有简历内容必须可溯源到经历库。

---

### Task 1: Skill 骨架与 SKILL.md 入口路由

**Files:**
- Create: `offer-helper/SKILL.md`
- Create: `offer-helper/resume/.gitkeep`、`offer-helper/interview/.gitkeep`、`offer-helper/templates/.gitkeep`、`offer-helper/experience/.gitkeep`

- [ ] **Step 1: 创建目录结构**

```bash
mkdir -p offer-helper/{resume,interview,templates,experience}
touch offer-helper/{resume,interview,templates,experience}/.gitkeep
```

- [ ] **Step 2: 写 SKILL.md**

```markdown
---
name: offer-helper
description: 中文求职助手——建立个人经历库，针对任意 JD 秒级定制 STAR 量化简历（严格防虚构），并提供反向追问式压力模拟面试。当用户提到求职、简历、面试、秋招、春招、实习、跳槽、JD、岗位描述时使用。
---

# offer-helper：上岸 Offer 助手

帮用户完成「经历库 → 定制简历 → 压力面试 → 反哺简历」的求职闭环。

## 硬性原则（任何流程都必须遵守）

1. **防虚构红线**：简历与面试回答建议中的每一条内容，必须能溯源到经历库
   原文。规则见 `resume/anti-fabrication.md`，简历输出前必须按其自检。
2. **数据本地**：经历库保存在用户本机 `experience/profile.md`，不上传、
   不外发。用户首次使用时主动告知这一点。
3. **中文输出**：简历、点评、报告默认简体中文；用户要求英文时再切换。

## 流程路由

按用户意图进入对应流程。**任何流程开始前先检查 `experience/profile.md`
是否存在**：不存在 → 先走流程 A 建库。

### A. 建立经历库（首次使用，约 15 分钟）
触发：用户说"帮我准备求职 / 建经历库"，或其他流程发现没有经历库。
做法：读取 `experience/profile.template.md`，按其中「访谈指南」逐段提问
（一次只问一段经历，挖到量化结果为止），最终生成 `experience/profile.md`。

### B. 定制简历
触发：用户粘贴 JD，或说"帮我改简历 / 投这个岗位"。
做法：
1. 按 `resume/jd-analysis.md` 分析 JD，产出关键词权重表。
2. 从经历库挑选最匹配的经历，按 `resume/star-rules.md` 改写。
3. 按 `resume/anti-fabrication.md` 自检每一条 bullet 的溯源。
4. 用 `templates/resume.html` 生成简历文件（A4，浏览器打印即 PDF），
   并输出「匹配度报告」：命中了 JD 哪些要求、缺口在哪、建议如何补。

### C. 压力模拟面试
触发：用户说"模拟面试 / 帮我练面试"。
做法：基于当前简历版本 + 目标 JD，按 `interview/pressure.md` 执行
反向追问式面试；结束后按 `interview/feedback.md` 输出逐题点评、
简历漏洞清单和经历库回改建议。

## 边界情况

- JD 过短或含糊（< 50 字或没有具体要求）→ 不硬猜，让用户补充岗位
  信息或公司名。
- 用户经历与 JD 严重不匹配（核心要求一条都对不上）→ 如实说明差距和
  风险，给出补齐建议，不粉饰。
- 用户要求"美化一下数据 / 编个项目" → 拒绝并解释：背调与压力面会
  穿帮；改为用真实经历中可挖掘的亮点替代。
```

- [ ] **Step 3: 验证结构与行数**

Run: `wc -l offer-helper/SKILL.md && ls offer-helper/`
Expected: 行数 < 300；五个子目录/文件就位。

- [ ] **Step 4: Commit**

```bash
git add offer-helper/ && git commit -m "feat: offer-helper skill 骨架与入口路由"
```

---

### Task 2: 经历库模板与访谈指南

**Files:**
- Create: `offer-helper/experience/profile.template.md`

- [ ] **Step 1: 写模板**

```markdown
# 个人经历库（profile.md 由此模板生成，仅保存在本机）

## 访谈指南（给 Claude：建库时按此逐段提问，一次只问一段）

对每段经历，追问到拿到以下全部字段为止；用户答不出量化结果时，
帮他从「规模、频次、对比、排名」四个角度回忆，仍没有就标注
`量化结果: 暂缺`，**绝不代替用户编一个数**。

- 背景：什么团队/课程/比赛？要解决什么问题？
- 我的动作：本人具体做了什么（区分「我做的」和「团队做的」）？
- 量化结果：数字、百分比、排名、规模（数据口径是什么？）
- 最难的点：卡在哪？怎么解决的？
- 可证伪细节：被面试官追问时能展开讲的技术/业务细节

## 基本信息
- 姓名 / 电话 / 邮箱 / 城市：
- 求职意向（岗位方向，可多个）：

## 教育经历
- 学校 / 专业 / 学历 / 时间：
- GPA 或排名（口径）：
- 相关课程（与求职方向相关的 3-5 门）：

## 实习与工作经历（每段一节，可复制）
### [公司] - [岗位]（[起止时间]）
- 背景：
- 我的动作：
- 量化结果：
- 最难的点：
- 可证伪细节：

## 项目经历（每段一节，结构同上）
### [项目名]（[角色]，[起止时间]）
- 背景：
- 我的动作：
- 量化结果：
- 最难的点：
- 可证伪细节：

## 技能
- 硬技能（语言/工具/框架，注明熟练度和实际用过的场景）：
- 证书 / 语言能力（如四六级分数）：

## 获奖与其他
- 奖项（级别、排名、获奖比例）：
- 社团 / 志愿 / 自媒体等（有量化结果才写）：
```

- [ ] **Step 2: 验证模板自洽**

检查：访谈指南字段（背景/动作/量化/最难/可证伪）与每段经历小节字段一一对应；无 TBD。

- [ ] **Step 3: Commit**

```bash
git add offer-helper/experience/ && git commit -m "feat: 经历库模板与访谈指南"
```

---

### Task 3: JD 分析方法（resume/jd-analysis.md）

**Files:**
- Create: `offer-helper/resume/jd-analysis.md`

- [ ] **Step 1: 写 JD 分析规则**

```markdown
# JD 分析方法

输入一份 JD，输出「关键词权重表」和「匹配策略」，供简历改写使用。

## 第一步：拆解 JD 为四类要求

| 类别 | 说明 | 例子 |
|------|------|------|
| 硬性门槛 | 不满足直接被筛掉 | 学历、专业、毕业年份、工作年限、证书 |
| 核心技能 | JD 中重复出现 / 放在职责第一条的技能 | Java、SQL、用户增长、内容策划 |
| 加分项 | "优先/加分/熟悉……者优先" | 开源经历、英语口语、相关实习 |
| 软素质 | 沟通、抗压、跨部门协作等 | 不单独写进简历，融入经历描述 |

## 第二步：输出关键词权重表

按下面格式输出给用户看（也是后续选材依据）：

| 关键词 | 类别 | 权重 | 经历库中的对应证据 |
|--------|------|------|--------------------|
| （从 JD 原文提取，保留原词） | 硬门槛/核心/加分 | 高/中/低 | 引用 profile.md 的哪一段；没有则写「缺口」 |

权重判定：出现在岗位职责前两条或重复出现 = 高；
硬性门槛恒为高；加分项 = 中；只出现一次的泛泛要求 = 低。

## 第三步：给出匹配策略

- 高权重且有证据 → 进简历，放对应模块最上方
- 高权重但缺口 → 写进「匹配度报告」，给补齐建议（如：突击一个小项目、
  补一句真实相关的课程/自学经历），**不允许在简历中虚构**
- 低权重 → 不为它挤占简历版面

## 注意

- 关键词使用 JD 原词（机筛按原词匹配），不要同义改写
- JD < 50 字或只有岗位名 → 停下来让用户补充，不硬猜
```

- [ ] **Step 2: Commit**

```bash
git add offer-helper/resume/jd-analysis.md && git commit -m "feat: JD 分析方法"
```

---

### Task 4: STAR 改写规则（resume/star-rules.md）

**Files:**
- Create: `offer-helper/resume/star-rules.md`

- [ ] **Step 1: 写改写规则（含正反例）**

```markdown
# STAR 量化改写规则

把经历库中的原始素材改写成简历 bullet。每条 bullet 的信息只能来自
经历库（见 anti-fabrication.md）。

## 结构

每条 bullet = 动词开头 + 做了什么(T/A) + 量化结果(R)。
背景(S)只在必要时用半句话带出，不展开。

## 改写规则

1. **动词开头**：主导 / 搭建 / 优化 / 推动 / 落地……禁止"参与了""负责了"
   开头（如果经历库里写的就是打杂，用"协助"如实表述，但要写清协助的
   具体动作）。
2. **量化优先**：优先用经历库里的数字。经历库标注「量化结果: 暂缺」时，
   退而求其次用定性结果（"上线后被 X 团队采用"），**不得自造数字**。
3. **每条 ≤ 2 行**：超过就拆分或删细节。
4. **关键词对齐**：把 JD 高权重关键词自然融入（前提：经历库证据支持）。
5. **区分个人与团队**：经历库写明是团队成果的，表述为"作为 X 角色，
   负责其中的 Y"。

## 正反例

❌ 参与了学校社团公众号的运营工作，提升了影响力。
✅ 运营校园公众号（独立选题+撰写），半年发布 32 篇，单篇最高阅读
   8,400，粉丝从 1,200 增长到 4,700。

❌ 负责后端开发，使用了 Java 和 MySQL，学到了很多。
✅ 用 Java/Spring Boot 重构订单查询接口，引入 Redis 缓存，P99 延迟
   从 1.2s 降到 180ms（压测口径：500 并发）。

❌ （经历库无数字时）优化了报表流程，效率提升 50%。← 编造，禁止
✅ （经历库无数字时）把人工汇总的周报表改为脚本自动生成，组内 4 名
   运营从手工操作中解放。

## 简历整体编排

- 应届生顺序：教育 → 实习 → 项目 → 技能/获奖；社招：工作 → 项目 → 教育
- 与 JD 最匹配的经历放所属模块第一条
- 单页为目标；超出时先砍低权重 bullet，不缩字号到 10.5pt 以下
```

- [ ] **Step 2: Commit**

```bash
git add offer-helper/resume/star-rules.md && git commit -m "feat: STAR 量化改写规则"
```

---

### Task 5: 防虚构红线（resume/anti-fabrication.md）

**Files:**
- Create: `offer-helper/resume/anti-fabrication.md`

- [ ] **Step 1: 写红线规则与自检清单**

```markdown
# 防虚构红线

简历造假在背调和压力面试中必然穿帮。本文件是简历输出前的强制自检。

## 红线（违反任何一条 = 不允许输出）

1. 不编造数字、百分比、排名（包括"合理估计"——估也得用户自己估）
2. 不编造头衔、角色（组员不能写组长）
3. 不编造未使用过的技术栈/工具（用户"听说过"≠"用过"）
4. 不虚构不存在的项目、实习、获奖
5. 不把团队成果写成个人独立完成

## 输出前自检（逐条 bullet 执行）

对简历中每一条 bullet 问：
- [ ] 这条内容能在 `experience/profile.md` 中找到原文依据吗？
- [ ] 数字与经历库一致吗（含口径）？
- [ ] 动词的力度与用户真实角色相符吗？

任何一条不通过 → 改写或删除该 bullet，不询问用户是否"通融"。

## 用户要求美化/编造时的回应模板

> 这条我不能帮你编：压力面试环节我会演示它怎么穿帮。
> 但你的真实经历里有 [X]，我们可以把它挖深：[具体建议]。

## 数据缺口的正确处理

- 引导用户回忆（规模/频次/对比/排名四角度）
- 仍缺 → 用定性表述（见 star-rules.md 规则 2）
- 在「匹配度报告」中如实列出缺口与补齐建议
```

- [ ] **Step 2: Commit**

```bash
git add offer-helper/resume/anti-fabrication.md && git commit -m "feat: 防虚构红线与自检清单"
```

---

### Task 6: 中文 A4 简历模板（templates/resume.html）

**Files:**
- Create: `offer-helper/templates/resume.html`

- [ ] **Step 1: 写 HTML 模板**

单文件、内联 CSS、A4 打印优化。生成简历时复制此模板替换占位内容。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>{{姓名}} - {{求职岗位}}</title>
<style>
  @page { size: A4; margin: 14mm 16mm; }
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    font-family: "PingFang SC", "Microsoft YaHei", "Noto Sans CJK SC", sans-serif;
    font-size: 10.5pt; line-height: 1.5; color: #222;
    max-width: 210mm; margin: 0 auto; padding: 14mm 16mm;
  }
  header { display: flex; justify-content: space-between;
           align-items: baseline; border-bottom: 2px solid #2b4a6f;
           padding-bottom: 6px; margin-bottom: 10px; }
  h1 { font-size: 17pt; color: #2b4a6f; }
  .contact { font-size: 9.5pt; color: #555; text-align: right; }
  h2 { font-size: 11.5pt; color: #2b4a6f; border-left: 3px solid #2b4a6f;
       padding-left: 6px; margin: 12px 0 6px; }
  .entry { margin-bottom: 8px; }
  .entry-head { display: flex; justify-content: space-between; }
  .entry-head b { font-size: 10.5pt; }
  .entry-head span { color: #666; font-size: 9.5pt; }
  ul { padding-left: 1.2em; margin-top: 2px; }
  li { margin-bottom: 2px; }
  @media print { body { padding: 0; } }
</style>
</head>
<body>
<header>
  <h1>{{姓名}}</h1>
  <div class="contact">{{电话}} · {{邮箱}} · {{城市}}<br>求职意向：{{岗位}}</div>
</header>

<h2>教育经历</h2>
<div class="entry">
  <div class="entry-head"><b>{{学校}} · {{专业}}（{{学历}}）</b><span>{{时间}}</span></div>
  <ul><li>{{GPA/排名/相关课程}}</li></ul>
</div>

<h2>实习经历</h2>
<div class="entry">
  <div class="entry-head"><b>{{公司}} · {{岗位}}</b><span>{{时间}}</span></div>
  <ul>
    <li>{{STAR bullet 1}}</li>
    <li>{{STAR bullet 2}}</li>
  </ul>
</div>

<h2>项目经历</h2>
<div class="entry">
  <div class="entry-head"><b>{{项目名}}（{{角色}}）</b><span>{{时间}}</span></div>
  <ul>
    <li>{{STAR bullet 1}}</li>
  </ul>
</div>

<h2>技能与获奖</h2>
<ul>
  <li>{{硬技能（注明实际使用场景）}}</li>
  <li>{{证书/语言/奖项}}</li>
</ul>
</body>
</html>
```

- [ ] **Step 2: 验证 A4 不破版**

Run: 用任一无头浏览器或本机浏览器打开打印预览检查；无浏览器环境时检查 CSS：`grep -c "@page" offer-helper/templates/resume.html`
Expected: `@page size A4` 存在；单页排版（占位内容下不超 1 页）。

- [ ] **Step 3: Commit**

```bash
git add offer-helper/templates/ && git commit -m "feat: 中文 A4 简历 HTML 模板"
```

---

### Task 7: 压力面试流程（interview/pressure.md）

**Files:**
- Create: `offer-helper/interview/pressure.md`

- [ ] **Step 1: 写压力面流程**

```markdown
# 反向追问式压力模拟面试

目标不是让用户背面经，而是攻击简历中的模糊表述，把它"焊牢"。

## 前置

- 需要：当前简历版本 + 目标 JD。缺 JD 时让用户提供或选择通用模式。
- 开场告知用户规则：我会扮演追问型面试官，过程中不点评不安慰，
  全部结束后统一反馈；随时可以说"暂停"退出角色。

## 面试官人设

资深业务面试官：礼貌但不放过任何含糊。不夸奖、不引导答案、
不在中途解释意图。每轮只问一个问题，等用户回答后再追。

## 追问策略（对简历每段重点经历执行）

按以下五连击逐层下钻，每层根据用户回答决定是否继续：

1. **角色还原**："这个项目里你具体负责哪部分？团队几个人？"
2. **口径拷问**："简历写 P99 从 1.2s 降到 180ms——这个数怎么测的？
   谁测的？"（凡有数字必问口径）
3. **最难时刻**："过程中最大的坑是什么？当时怎么解决的？"
4. **备选方案**："为什么用方案 A 不用 B？如果重做一次你会改什么？"
5. **迁移验证**："我们的场景是 [JD 中的业务]，你这段经验怎么迁移？"

## 升压规则

- 回答含糊（"我们当时就是……大概……"）→ 立刻追问细节，不放行
- 回答与简历矛盾 → 指出矛盾："你简历上写的是 X，刚才说的是 Y？"
- 回答流畅且有细节 → 换下一段经历，不浪费轮次

## 记录（面试过程中静默维护，反馈阶段使用）

每个问题记录：问题、用户回答要点、漏洞类型
（含糊 / 口径不清 / 与简历矛盾 / 无法迁移 / 表现良好）。

## 收尾

5-8 段追问链或用户说"结束"后，退出角色，
转入 `interview/feedback.md` 输出反馈。
```

- [ ] **Step 2: Commit**

```bash
git add offer-helper/interview/pressure.md && git commit -m "feat: 反向追问式压力面试流程"
```

---

### Task 8: 面试反馈格式（interview/feedback.md）

**Files:**
- Create: `offer-helper/interview/feedback.md`

- [ ] **Step 1: 写反馈输出格式**

```markdown
# 面试反馈输出格式

压力面结束后，按以下结构输出（全部基于面试中的真实记录，不泛泛而谈）。

## 1. 总评（四维各 1-5 分 + 一句话依据）

| 维度 | 分 | 依据（引用面试中的具体回答） |
|------|----|------------------------------|
| 结构性（答案有没有框架） | | |
| 具体性（能否落到细节） | | |
| 量化可信度（数字口径是否站得住） | | |
| 迁移能力（经验能否对接 JD 场景） | | |

## 2. 逐题点评

每题：原问题 → 你的回答要点 → 问题在哪 → 更好的回答思路
（思路只能基于用户自己说过的真实素材组织，不替用户编内容）。

## 3. 简历漏洞清单（核心交付物）

| 简历原文 | 暴露的问题 | 修改建议 |
|----------|------------|----------|
| 引用 bullet 原文 | 如：口径说不清 / 角色被放大 | 具体改法或建议删除 |

## 4. 经历库回改建议

列出本次面试中用户口头补充的、但经历库里没有的真实细节
（新数字、新案例、踩坑故事），建议追加进 `experience/profile.md`
对应小节——这是经历库越用越厚的机制。

最后问用户：要不要现在就按漏洞清单更新简历和经历库？
```

- [ ] **Step 2: Commit**

```bash
git add offer-helper/interview/feedback.md && git commit -m "feat: 面试反馈输出格式"
```

---

### Task 9: 验收 fixtures（3 套经历 × JD）

**Files:**
- Create: `tests/fixtures/profile-cs-newgrad.md`（应届后端，经历真实但部分量化缺失——用于测防虚构）
- Create: `tests/fixtures/jd-backend.md`
- Create: `tests/fixtures/profile-marketing.md`（应届非技术，社团/公众号经历）
- Create: `tests/fixtures/jd-operations.md`
- Create: `tests/fixtures/profile-jobhopper.md`（2 年经验跳槽，经历与 JD 有明显缺口——用于测「如实说明差距」）
- Create: `tests/fixtures/jd-pm.md`

- [ ] **Step 1: 写 fixture 1 —— 应届后端（关键：两处「量化结果: 暂缺」）**

`tests/fixtures/profile-cs-newgrad.md`：按 profile.template.md 结构填写：
华中某 211 软件工程本科 2026 届，GPA 前 30%；一段 4 个月中型互联网公司
后端实习（订单系统，写过 Redis 缓存改造，**量化结果: 暂缺**）；一个课程
项目（教务选课系统，3 人团队成员，**量化结果: 暂缺**）；技能 Java/Spring
Boot/MySQL/Redis（实习中实际使用）、Python（课程作业水平）；六级 489。

`tests/fixtures/jd-backend.md`：某电商公司 Java 后端校招 JD：本科以上、
Java/Spring/MySQL、高并发经验优先、熟悉 Redis 加分、良好沟通。

- [ ] **Step 2: 写 fixture 2 —— 应届运营**

`tests/fixtures/profile-marketing.md`：双非市场营销本科 2026 届；校园
公众号主笔（32 篇/半年，最高阅读 8,400，粉丝 1,200→4,700）；一段连锁
茶饮品牌门店运营实习（私域社群 3 个、月均 GMV 数据有口径）；会用
剪映/Canva；四级 530。

`tests/fixtures/jd-operations.md`：消费品公司新媒体运营校招 JD：内容
策划、小红书/公众号运营经验、数据敏感、加分项视频剪辑。

- [ ] **Step 3: 写 fixture 3 —— 社招跳槽（带硬缺口）**

`tests/fixtures/profile-jobhopper.md`：2 年传统软件公司测试工程师，想
转产品经理；有需求评审参与经历、无独立产品设计经历（**与 JD 的"1 年
以上产品经验"硬门槛冲突**）。

`tests/fixtures/jd-pm.md`：互联网公司产品经理 JD：1 年以上产品经验
（硬门槛）、需求分析、原型设计（Axure/Figma）、数据驱动。

- [ ] **Step 4: Commit**

```bash
git add tests/fixtures/ && git commit -m "test: 三套经历×JD 验收 fixtures"
```

---

### Task 10: 验收场景文档 + 端到端验收

**Files:**
- Create: `tests/acceptance.md`

- [ ] **Step 1: 写验收标准**

```markdown
# offer-helper 端到端验收

每个场景：以 fixture 的 profile 作为 experience/profile.md、对应 JD 作为
用户输入，完整走一遍流程 B（简历）与流程 C（面试），按下表判定。

## 场景 1：应届后端 × 电商 Java JD（测防虚构）
- [ ] 关键词权重表使用 JD 原词，Redis 标为加分项
- [ ] 两处「量化结果: 暂缺」的经历：bullet 用定性表述，**未出现任何
      编造数字**（核心断言）
- [ ] 匹配度报告列出"高并发经验"缺口并给出真实可行的补齐建议
- [ ] 简历 HTML 单页、A4 打印不破版

## 场景 2：应届运营 × 新媒体 JD（测正向改写质量）
- [ ] 公众号数据（32 篇/8,400/1,200→4,700）全部保留且与 fixture 一致
- [ ] bullet 全部动词开头、≤2 行
- [ ] 压力面对"GMV 数据口径"发起追问（五连击中的口径拷问生效）

## 场景 3：测试转产品 × PM JD（测如实告知差距）
- [ ] 明确指出"1 年以上产品经验"硬门槛不满足，不粉饰
- [ ] 不把测试经历改写成产品经历（角色红线）
- [ ] 给出真实迁移点（需求评审参与经历）+ 差距补齐路径

## 通用断言（三场景都查）
- [ ] 简历每条 bullet 可在 fixture 原文中找到依据
- [ ] 面试反馈含四维评分、漏洞清单、经历库回改建议三部分
- [ ] 主动告知用户数据保存在本地
```

- [ ] **Step 2: 执行验收**

对每个场景，dispatch 一个子代理：加载 offer-helper skill 文件 + 对应
fixtures，模拟完整用户对话，对照 acceptance.md 逐项打勾并报告失败项。
（若子代理不可用——当前 API 偶发 529——则在主会话逐场景执行。）

Expected: 三场景全部断言通过；任何失败项 → 修复对应规则文件后重跑该场景。

- [ ] **Step 3: Commit**

```bash
git add tests/acceptance.md && git commit -m "test: 端到端验收场景与执行结果"
```

---

### Task 11: README 与发布准备

**Files:**
- Create: `README.md`

- [ ] **Step 1: 写中文 README**

内容必须包含：
1. 一句话定位 + 适用人群（秋招/春招/实习/跳槽）
2. 30 秒上手示例（对话截图风格的 markdown 演示：粘贴 JD → 得到简历）
3. 安装方法三种：Claude Code（`cp -r offer-helper ~/.claude/skills/`）、
   Claude.ai（打包上传）、Cursor/Codex（按 Agent Skills 标准目录放置）
4. 隐私说明：经历库仅存本机，skill 无任何网络上传逻辑（可审计：纯
   Markdown + 1 个 HTML 模板）
5. 防虚构声明：本工具拒绝编造经历——这是 feature 不是 bug
6. Roadmap：行业面经题库 → 英文简历 → 考公搭子
7. License: MIT；欢迎 PR（尤其各行业 JD 分析样本）

- [ ] **Step 2: 终检**

Run: `wc -l offer-helper/SKILL.md && grep -rn "TODO\|TBD\|占位" offer-helper/ README.md || echo CLEAN`
Expected: SKILL.md < 300 行；输出 CLEAN。

- [ ] **Step 3: Commit**

```bash
git add README.md && git commit -m "docs: 中文 README 与发布说明"
```

---

## Self-Review 结果

1. **Spec 覆盖**：设计文档 §2 双模块（Task 3-8）、§3 结构（Task 1-2）、§4 三流程（Task 1 路由 + 2/3-6/7-8）、§5 错误处理（SKILL.md 边界情况 + anti-fabrication + acceptance 场景 3）与测试（Task 9-10 即设计中"3-5 套组合"的落实，取 3 套）、§6 发布（Task 11；提交收录各 skill 合集在发布后人工执行，不属于代码任务）——无缺口。
2. **占位符扫描**：模板中的 `{{...}}` 是简历模板的运行时占位符（设计如此），非计划占位符；其余无 TBD/TODO。
3. **一致性**：文件路径在各任务间一致（`experience/profile.md`、`resume/star-rules.md` 等引用名与创建名逐一核对无误）；fixture 文件名在 Task 9/10 间一致。
