# Paper2HTML 网页版一键流程（ChatGPT Agent mode 等）

> 使用方法（给使用者，不是给模型的指令）：把下面整段 prompt 贴给网页版模型的 agent 模式，替换最上方【输入区】的占位符即可。前提：该模型的 GitHub 连接器已授权访问私有仓库 `cnfjlhj/cnfjlhj-blog`，且有浏览网页 + 代码沙箱能力。

---

【输入区】
- 论文：{{arXiv 链接或论文标题}}
- 发布路径：{{topic}}/{{series}}/{{slug}}（其中 topic 必须是博客已有的课题目录，例如 `self-evolving-agent`；若 series 是新的，请填该系列的中文展示名）
- 新系列的中文展示名（仅在 series 不存在时需要）：{{series-title}}，展示顺序 order 选一个该课题下未被占用的整十数，例如 90
- 页面标题 research-title：{{paper title}}

---

你是一位顶级的 AI researcher、全栈开发者兼信息设计师。把指定论文转化为一篇可发布的单文件中文 HTML 精读页，并完成从验证到上线的全部步骤。质量标准以公开的 paper2html skill 为准。

## 第一步：获取 skill 规范（先做，全部用浏览工具或沙箱 curl 拉取）

从公开仓库逐个读取以下文件到你工作区（raw 地址，可 curl；若沙箱无法联网，就用浏览工具读取内容后写入沙箱同名文件）：

1. `https://raw.githubusercontent.com/cnfjlhj/ai-collab-playbook/main/skills/full/paper2html/SKILL.md` —— 主工作流
2. `https://raw.githubusercontent.com/cnfjlhj/ai-collab-playbook/main/skills/full/paper2html/assets/cheatsheet-template.html` —— 页面模板（含七个 Other Discussion Roles 的原始 prompt 与 `data-discussion-role` 标记）
3. `https://raw.githubusercontent.com/cnfjlhj/ai-collab-playbook/main/skills/full/paper2html/prompts/gemini-initial-html.md` —— 内容与写作规范
4. `https://raw.githubusercontent.com/cnfjlhj/ai-collab-playbook/main/skills/full/paper2html/references/other-discussion-roles.md` —— 七角色渲染契约
5. 以下脚本下载到沙箱以本地运行：
   - `https://raw.githubusercontent.com/cnfjlhj/ai-collab-playbook/main/skills/full/paper2html/scripts/tex_inventory.py`
   - `https://raw.githubusercontent.com/cnfjlhj/ai-collab-playbook/main/skills/full/paper2html/scripts/optimize_paper_images.py`
   - `https://raw.githubusercontent.com/cnfjlhj/ai-collab-playbook/main/skills/full/paper2html/scripts/validate_paper_html.sh`

## 第二步：冻结来源边界

从【输入区】的论文出发，收集可公开访问的材料：arXiv abstract、arXiv 源码包（`https://arxiv.org/e-print/<id>`，gzip + tar 解包）、PDF、OpenReview、项目主页、GitHub repo、appendix、dataset/model cards。写一份 `notes/source-boundary.md`：阅读来源与公开证据分开列出。若论文不可公开获取，停止并报告，不做 HTML。

## 第三步：grounding anchor（反“墙头草”）

写 `notes/grounding-anchor.md`，三段：
1. 人类已知边界——**必须至少部分来自论文之外的外部检索**（用你的搜索工具），不得仅由论文 related work 改写；每个问题标注“已解决 / 部分解决 / 未解决”并附公开链接。
2. 真实增量——站在审稿人立场，相对①的边界一句话说明本文到底推进了什么；增量平平就如实说。
3. 写作锚点——主体各节哪些承担增量论证、哪些是复述性事实。

anchor 是内部笔记，不进公开 HTML。写作全程以②为组织主线；写《我的评论》和七角色前自问：“我是在复述论文逻辑，还是在独立评估它是否成立？”

## 第四步：材料包与 figure 盘点

- 解包 arXiv 源码（若有），运行 `tex_inventory.py`（参数见其 `--help`）；该脚本对某些 TeX 布局会漏报，务必人工扫描 `\includegraphics` 与 figure 环境核对。
- 写 `notes/material-pack.md`：元数据、outline、claims、记号、公式、算法、实验细节（datasets、splits、baselines、metrics、prompts、超参、appendix 设置）。
- 写 `notes/figure-table-map.md`，六列表格：`item | source | original caption | what it shows | included | waiver`。每个 figure 默认收录；放不进主叙事的进“完整图谱 / Figure Gallery”；不收录必须写 waiver 理由。`included` 列填 HTML 中将使用的**纯文件名**（如 `framework-overview.pdf`）。

## 第五步：图片优化

把论文 figure 用 optimize_paper_images.py（需 Pillow）转成宽 1600px 的 WebP，并生成 data URI 注入单文件 HTML；沙箱缺依赖时用 Pillow 手工等效转换，或以带编号与 caption 摘要的占位卡片替代并在完成后报告。`<figure data-figure="basename.ext">` 标记必须与 figure-table-map 完全一致。

## 第六步：生成 index.html

以模板为骨架、gemini-initial-html.md 为内容规范生成完整页面。硬性要求：
- 章节顺序：`先讲一个故事`、`故事与技术如何对应`、`先给结论`、`研究动机`、`数学表示及建模`、`算法流程/方法`、`实验设计`、`实验结果`、`我的评论`、`One More Thing`、`Other Discussion Roles / 其他讨论角色`、`Reference / Evidence`。
- 七个角色各出现一次、顺序不变、英文原始 prompt 逐字保留、每个角色后写实质中文见解；`data-discussion-role` 标记不得改名。
- KaTeX：行内 `\(...\)` 或 `$...$`，块级 `$$...$$` 或 `\[...\]`；不要手写 `class="katex...` 渲染标记。
- 证据边界：不确定的显式标注，不编造实验、数字、引用。
- `<head>` 必须含下列 meta（title 与 `<title>` 一致；series/topic title 必须与目标博客 `_meta.json` 完全一致）：

```html
<meta name="research-source" content="论文公开链接" />
<meta name="research-title" content="{{paper title}}" />
<meta name="research-series-title" content="{{series-title}}" />
<meta name="research-topic-title" content="{{topic 的中文展示名}}" />
<meta name="research-description" content="一句话中文摘要" />
<meta name="research-date" content="YYYY-MM-DD" />
<meta name="research-tags" content="逗号分隔小写标签" />
```

- KaTeX 引用博客自托管路径 `/vendor/katex/0.16.9/`（katex.min.css、katex.min.js、contrib/auto-render.min.js），与站内现有页面一致。
- 除公式与专名外用中文；风格克制，是深度精读页不是营销页。

## 第七步：验证

在工作区跑 `bash validate_paper_html.sh index.html --public`（figure-table-map.md 必须位于 `notes/` 且与 index.html 同根目录）。该脚本静态检查占位符、本地路径泄露、七角色完整性与 figure 盘点一致性；若沙箱没有 `google-chrome`，浏览器渲染检查会自动跳过——在最终报告注明“浏览器校验未在沙箱执行”。静态检查任何一条不过，修复后重跑，直到通过。低成本补充自检：沙箱有 Node 时可执行 DOM 解析确认无重复 `data-figure`。

## 第八步：部署到博客（私有仓库 cnfjlhj/cnfjlhj-blog，分支 main）

目标仓库的部署是 GitHub Actions 自动化的：push 到 main 会触发「Deploy to Cloudflare Pages」工作流，依次执行依赖安装、Prettier format:check、ESLint、Astro build，最后用 wrangler 把构建产物上传到 Cloudflare Pages 项目 cnfjlhj-blog（push 自动部署；上传动作发生在 GitHub Actions 内，不是 Cloudflare Git 集成）。`public/research/` 参与 format:check，推送前必须先在沙箱完成 Prettier 格式化（步骤 3）。

通过 GitHub 连接器做且仅做以下操作：
1. 新建目录 `public/research/{{topic}}/{{series}}/{{slug}}/`，只放一个 `index.html`（notes、材料包、PDF、图片源文件一律不上传）。
2. `research-series-title` / `research-topic-title` 必须与博客现有 `_meta.json` 中的 `title` 完全一致。目标 series 已存在时不要创建新的 `_meta.json`；series 不存在时，才在 `<series>/` 目录写入：

```json
{
  "title": "{{series-title}}",
  "order": 90
}
```

3. 推送前在沙箱完成 Prettier 格式化：下载 `https://raw.githubusercontent.com/cnfjlhj/cnfjlhj-blog/main/.prettierrc.mjs`；沙箱能安装 npm 包时装 `prettier@^3.7.4`、`prettier-plugin-astro`、`prettier-plugin-tailwindcss` 并运行 `npx prettier --config .prettierrc.mjs --write index.html`；无法装包时用等效精简配置（printWidth 80、tabWidth 2、semi、双引号、trailingComma es5、parser html）代替——本页为纯 HTML 且无 Tailwind class，插件不影响输出。格式化后重跑第七步的 validate，确认 `data-discussion-role`、`data-figure` 与 KaTeX 公式未被破坏。
4. 不要修改 `src/`、配置文件或任何与本页面无关的文件。
5. Commit message：`research: add {{slug}} ({{topic}}/{{series}})`。
6. Push 到 `main`。

## 第九步：核验与交付

1. 等待 Actions 工作流（Deploy to Cloudflare Pages）完成；失败则读取日志定位并修复（只允许再产生一个修复 commit）。最常见失败点：Prettier format:check（未按第八步步骤 3 格式化）与 Astro build（`_meta.json` JSON 语法错误或 research meta 缺失）。
2. 部署成功后访问 `https://cnfjlhj-blog.pages.dev/research/{{topic}}/{{series}}/{{slug}}/` 确认返回 200 且包含 KaTeX 渲染和七个角色区块。
3. 向用户交付：页面 URL、commit 链接、Actions 运行链接、figure 盘点结论（收录数 / waiver 数）、grounding anchor 的增量结论、以及沙箱内验证与线上验证各自完成/未完成的清单。
4. 未经用户明确要求，不联系论文作者，不修改任何其他页面的内容。

## 禁止事项

- 不上传 notes/、material-pack、源码包或任何内部工作文件。
- 不在公开 HTML 暴露本地路径（`/home/`、`file://`、`main.tex`、`sections/`、`appendices/`）。
- 不为凑数捏造引用、实验数字或社会影响证据。
- 不跳过 grounding anchor 直接写作；不把搜索摘要或二手转述冒充论文原文。
