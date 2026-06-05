# offer-helper · 上岸 Offer 助手

> 装进 Claude 里的免费求职教练——经历库只写一次，每个 JD 秒级定制简历，
> 再用反向追问式压力面试把简历"焊牢"。

一个中文求职 [Agent Skill](https://www.anthropic.com/news/skills)，适合
**秋招 / 春招 / 实习 / 跳槽** 的大学生和年轻职场人。兼容 Claude Code、
Claude.ai、Cursor、Codex 等支持 Agent Skills 标准的运行时。

## 它能做什么

| 模块 | 做什么 | 和"直接问 ChatGPT"的区别 |
|------|--------|--------------------------|
| 📇 经历库 | 摄入你的**简历 / GitHub 仓库 / 其他文档**，整合成一份经历库，之后可不断追加 | 不用每次重新粘贴材料，简历和面试共享同一份可溯源素材 |
| 📄 定制简历 | 粘贴 JD → 分析关键词 → STAR 量化改写 → 出 A4 简历 + 匹配度报告 | **严格防虚构**：只重组真实经历，缺数据宁可留白 |
| 🎤 大厂面试 | 你指定哪份是简历，面试官**只针对这份简历的具体内容**做大厂式追问 | 不是背面经，而是把简历里撑不住追问的地方提前补牢 |

闭环：**摄入材料建库 → 简历 → 面试 → 漏洞反哺经历库**，经历库越用越厚。

> **两种运行环境，能力有差异**（很重要）：
> - **Claude Code / Cursor 等（有文件系统）**：经历库与简历会**真正存到你电脑上**
>   `~/offer-helper-data/`，跨对话自动记住，可一直追加；能自动 `git clone` 读你的
>   GitHub 仓库。这是体验最完整的方式。
> - **Claude.ai 网页版（无文件系统）**：用起来一样，但**经历库存不到你电脑**——
>   助手会把经历库文本发给你，请自己存进备忘录/文档，下次开新对话贴回来继续；
>   读 GitHub 仓库可能需要你把 README/技术栈粘给它。别担心，它会如实告诉你当前
>   环境能做什么。

## 30 秒上手

```
你：这是我的简历(上传 resume.pdf)，还有我的 GitHub：github.com/me/project，帮我建经历库
助手：（读取简历+仓库，抽取整合成经历库，只就缺失的量化数据问你几个问题，
      存到 ~/offer-helper-data/profile.md 并告诉你路径；网页版则把文本发你自存）

你：（粘贴一段 JD）帮我投这个岗位
助手：
  ① JD 关键词权重表（哪些是硬门槛/核心/加分）
  ② 简历 HTML（存为 resume-公司-日期.html；按「导出 PDF」三步变成可投递 PDF）
  ③ 匹配度报告：命中了哪几条、缺口在哪、怎么补

你：用我刚生成的这份简历模拟面试
助手：（确认基准简历后，扮演大厂面试官，对简历每段经历层层追问）
  → 结束后给：四维评分 + 简历漏洞清单 + 经历库回改建议
```

简历是 `.html`，**怎么变成能投的 PDF**：见 [`docs/export-guide.md`](docs/export-guide.md)
（浏览器打印 → 另存为 PDF，1 分钟，记得勾「背景图形」）。

📖 **工作原理示意**（含大厂面试"由浅及深"提问阶梯）：[`examples/demo.md`](examples/demo.md)
　｜　**发个 GitHub 仓库就能建经历库**：[`examples/github-intake-example.md`](examples/github-intake-example.md)

## 安装

**Claude.ai 网页 / 桌面版（小白推荐，无需命令行）：**
1. 在本仓库页面点绿色 **Code → Download ZIP**，解压得到 `offer-helper/` 文件夹。
2. 把 `offer-helper/` 这个文件夹**重新压成 zip**（确保 zip 解开后第一层就是
   `SKILL.md`，不要多套一层目录）。
3. 在 Claude.ai → **Settings → Capabilities → Skills** 上传该 zip。
   （若找不到 Skills 入口，说明你的账号类型暂不支持上传自定义 skill，请改用
   下面的 Claude Code 方式。）
4. 装好后在对话里说一句"**帮我准备求职**"，助手会主动开始建经历库 = 成功。

**Claude Code：**
```bash
git clone git@github.com:dominciyue/resume_skill.git
cp -r resume_skill/offer-helper ~/.claude/skills/offer-helper
```
之后在对话里说"帮我准备求职"即可触发。

> ⚠️ **更新 skill 前注意**：你的经历库存在 `~/offer-helper-data/`（独立于 skill
> 目录），所以重装/更新 `~/.claude/skills/offer-helper` **不会动到你的经历库**，
> 放心覆盖。

**Cursor / Codex 等：** 将 `offer-helper/` 放入对应工具的 skills 目录
（遵循 Agent Skills 标准，根目录含 `SKILL.md` 即可）。

## 隐私

- **你的经历库只存在本机** `~/offer-helper-data/`（网页版则在你自己保管的文本里），
  不上传、不外发。
- 本 skill 是**纯文本规则**：全部是 Markdown 指令 + 1 个 HTML 简历模板，
  无任何网络请求代码，可自行审计（`grep -ri "http\|fetch\|curl" offer-helper/`）。

## 关于"防虚构"

简历造假会在背调和压力面试中穿帮，所以本工具**拒绝帮你编造**数字、
头衔、没用过的技术或不存在的项目——这是 feature 不是 bug。缺数据时它会
引导你回忆真实细节，或如实用定性表述，并在匹配度报告里标出缺口和补齐
建议。压力面试模块就是用来提前演示"哪些表述会被问穿"的。

## Roadmap

- [ ] 分行业面经题库（互联网 / 快消 / 金融 / 国企）
- [ ] 英文简历与英文面试模式
- [ ] 考公搭子 skill（复用经历库结构：职位匹配 + 申论批改 + 备考计划）

## 贡献

欢迎 PR，尤其是**各行业的 JD 分析样本**和**面试追问策略**。
经历库模板、改写规则、面试流程都是纯 Markdown，改起来很轻。

## License

MIT
