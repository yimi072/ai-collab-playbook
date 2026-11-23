---
name: paper2html
description: Use when turning an academic paper PDF/arXiv/OpenReview page/local LaTeX source into a single-file Chinese HTML deep-reading page, especially when the user wants a Cheat-Sheet style explanation, KaTeX formulas, figure/table evidence, validation, a publishable research page, or a web-agent publishing path.
---

# Paper2HTML

## Overview

Create one self-contained Chinese `index.html` that lets the reader understand the paper without reopening the PDF. Open with a compact, source-faithful story, then deliver a clear Cheat-Sheet page with evidence boundaries, KaTeX formulas, complete figure coverage, reviewer-level analysis, and no decorative complexity.

Every completed page ends its substantive analysis with all seven MIT **Other Discussion Roles**. Each role shows the original English prompt verbatim and a source-grounded Chinese response.

Use scripts first. Keep this skill as workflow and quality gates, not a long prompt.

## Scripted Path (local coding agent)

```bash
cd /path/to/paper2html

# 1) Create workspace, notes, and starter index.html
python scripts/bootstrap_paper2html.py \
  --title "Paper Title" \
  --slug paper-slug \
  --out /path/to/output/paper-slug-html-YYYYMMDD \
  --arxiv 2504.07952 \
  --github https://github.com/org/repo \
  --publish-to topic/series/paper-slug

# 2) Inventory TeX/source when available
python scripts/tex_inventory.py \
  /path/to/output/source/unpacked \
  --out /path/to/output/notes/tex-inventory.md \
  --json /path/to/output/notes/tex-inventory.json

# 3) Optimize figures and create data URIs for single-file HTML
python scripts/optimize_paper_images.py \
  paper_figures/*.pdf paper_figures/*.png \
  --out-dir /path/to/output/assets/optimized \
  --width 1600 \
  --data-uri-json /path/to/output/assets/data-uris.json

# 4) Optional: ask Gemini for a visual draft on a copy
scripts/gemini_frontend_pass.sh \
  /path/to/output/index.html \
  --reference /path/to/good-reference.html \
  --timeout 240

# Inspect the Gemini request without making a model call
scripts/gemini_frontend_pass.sh /path/to/output/index.html --dry-run

# 5) Validate the completed page
scripts/validate_paper_html.sh /path/to/output/index.html --public

# 6) Optional publication through a compatible research-page importer
PAPER2HTML_BLOG_ROOT=/path/to/blog \
PAPER2HTML_PUBLIC_BASE_URL=https://example.com/research \
scripts/publish_paper2html.sh \
  /path/to/output/index.html \
  --to topic/series/paper-slug \
  --title "Paper Title" \
  --description "One sentence summary" \
  --tags "paper-reading,agent" \
  --ship
```

For a web-hosted model (ChatGPT agent mode, etc.) that has browsing and a code sandbox but no local skill checkout, use `prompts/web-deploy.md` instead: it packages the fetching of this skill, page generation, validation, and direct GitHub publishing into one prompt.

## Workflow

1. **Freeze the source boundary**
   - Treat user input as a locator for one paper, then look for public adjacent artifacts: arXiv source/PDF, OpenReview, proceedings, project page, GitHub, appendix, dataset cards, and model cards.
   - Run `bootstrap_paper2html.py` when starting from a known title, arXiv id, or GitHub repository. Otherwise manually create the same `notes/source-boundary.md`.
   - Separate reading sources from public evidence sources. Public pages must not expose local paths, private filenames, review metadata, private prompts, or run logs.
   - If public publishing is requested and no public source exists, generate private HTML only and do not publish.

2. **Extract the material**
   - Prefer TeX/source packages over PDF-only reading. Run `tex_inventory.py` when source exists; verify its output, since TeX layouts can confuse automated counting. When in doubt, scan `includegraphics` and figure environments manually.
   - Build a material pack covering metadata, outline, claims, notation, equations, algorithms, experiments, appendix details, and public links.
   - Use PDF rendering for visual verification and figure/table placement. `pdftotext` is only an auxiliary index.
   - Record every discovered paper figure in `notes/figure-table-map.md`. For each item, capture its source/page, original caption, what the visual itself shows, and either the exact `data-figure` basename used in HTML or an explicit waiver reason.
   - **Figure coverage gate:** every figure discovered from TeX/source/PDF appears in the final HTML by default. Figures that do not fit the main narrative go in a `完整图谱 / Figure Gallery` appendix section. Omit a figure only with an explicit waiver stating why it is duplicate, unreadable, non-public, or otherwise inappropriate.
   - Add a machine-checkable marker for every included figure: `<figure data-figure="basename.ext">`. Validation compares figure-map entries against these markers. Do not expose local/source-directory paths in public HTML markers.
   - Extract reproducibility details: datasets, splits, models, baselines, metrics, prompts, tool/prover/solver versions, decoding, timeouts, retries, and appendix-only settings.

3. **Ground against the human knowledge boundary**
   - 目的：在写主体内容之前，先抵抗论文逻辑链对注意力的污染——不让模型沦为论文叙述的“墙头草”复述者。论文 A→B→C→D 全扣起来“看起来正确”，模型极易接受，于是所有总结都变成作者观点的复述。
   - **反循环条款（最关键）**：anchor 第①段“人类已知边界”必须至少部分来自**论文之外的外部检索**——用你可用的独立检索通道（web search、arXiv、学术索引）查证相关问题的解决程度，**明确禁止仅由论文 related work 一节改写而来**。否则就是“墙头草式 grounding”——形式上完成步骤，机制上污染原样发生。用“已解决 / 部分解决 / 未解决”三档标注。
   - **边界必须落到公开链接**：没有公开链接支撑的“已知边界”就是另一种幻觉污染。无法给出可信公开依据时，如实标注“属模型记忆、待核”，与 skill 既有 evidence-boundary 精神一致。
   - 再锚定本论文的**真实增量**：相对于此边界，站在审稿人/人类立场看它给人类加了什么新知识，而非复述 abstract 的 contribution 段。若增量对整体不成立（如优化的是问题的 1% 而非 99%），如实标注。
   - **适用范围**：grounding 默认对所有论文启用。对增量平平的论文，anchor②如实标注“增量有限/属同一坐标内的重新表述”，不强行拔高；对纯综述类，anchor②改为“综述的组织增量”；对极短论文，三段可压缩为一段。无论如何不跳过 anchor。
   - 产出 `notes/grounding-anchor.md`，含三段：
     1. 人类已知边界（相关问题与解决程度，**附公开链接**；标注“已解决/部分解决/未解决”）
     2. 真实增量（一句话，审稿人立场，**相对于①的边界**定义）
     3. 写作锚点（主体各节如何围绕增量组织——哪些节承担增量论证、哪些是纯复述性事实；短清单，不膨胀成第二份 outline）
   - **anchor 是 7 role 的共享上游**：它把 Archaeologist（历史边界视角）与 Scientific Peer Reviewer（增量判断）从流程末尾前置到写作之前。这两个 role 的 response 应直接引用 anchor ①②，避免重复检索。
   - The anchor is an internal working note and never enters the public HTML. Public links used from it must also appear in `Reference / Evidence`.

4. **Write the page**
   - Use `assets/cheatsheet-template.html` as the visual skeleton and organize the page around the grounding anchor's real-contribution statement, not the paper's narrative order alone.
   - Before writing each major section, recall the anchor's 真实增量. Before writing `我的评论` and the seven roles, ask once: “这里我是复述论文能自圆其说的逻辑，还是在用独立的人类立场评估其是否成立？” If you are paraphrasing the paper instead of arguing around the increment, return to the anchor and rewrite that section.
   - Use the fixed section order unless the paper requires a small adjustment: `先讲一个故事`, `故事与技术如何对应`, `先给结论`, `研究动机`, `数学表示及建模`, `算法流程/方法`, `实验设计`, `实验结果`, `我的评论`, `One More Thing`, `Other Discussion Roles / 其他讨论角色`, `Reference / Evidence`.
   - Open with a compact narrative that gives the reader a concrete protagonist or system, its goal, the obstacle or anomaly, why the obvious explanation is insufficient, and the paper's turning point. Prefer a sourced discovery, experiment, or historical episode; otherwise label the opening as a thought experiment or teaching analogy. Never invent author dialogue, private motivation, chronology, or results.
   - Follow the story with a short `故事与技术如何对应` bridge that maps the story's actors and conflict to the paper's actual concepts, method, and evidence, marking which details are analogy and which are source facts.
   - Keep the opening proportional—usually one screen or a few short paragraphs. It is an entry into the paper, not a replacement for conclusions, equations, experiments, caveats, or the evidence boundary.
   - Add a short version note near the top with reading basis and retrieval date, without local paths.
   - Bind each major claim to evidence and state boundaries when evidence does not support a stronger conclusion.
   - Use Chinese by default; keep paper terms, model names, datasets, commands, and identifiers in English.
   - Use KaTeX for all math. Inline math uses `\(...\)` or `$...$`; block math uses `$$...$$` or `\[...\]`. Write raw TeX delimiters and let KaTeX auto-render them; never hand-write rendered KaTeX markup in the source HTML.
   - Figure/table captions are evidence-local: captions may claim only what the displayed visual/table itself shows. Broader interpretation goes in a nearby callout.
   - `One More Thing` is for bounded research insight, not unsupported extrapolation.
   - **Discussion-role close:** read `references/other-discussion-roles.md` and execute its rendering contract. The `Other Discussion Roles / 其他讨论角色` section is mandatory, is the final substantive section, and may be followed only by `Reference / Evidence`.
   - Include all seven roles exactly once and in source order: `Scientific Peer Reviewer`, `Archaeologist`, `Academic Researcher`, `Industry Practitioner`, `Hacker`, `Private Investigator`, `Social Impact Assessor`.
   - For every role, display the complete original English prompt verbatim, then provide a substantial Chinese response that performs every requested responsibility. Do not abbreviate the prompt, collapse roles, rename or silently omit a role, or substitute a generic summary.
   - Use the exact template markers `data-discussion-role="scientific-peer-reviewer|archaeologist|academic-researcher|industry-practitioner|hacker|private-investigator|social-impact-assessor"`. Validation treats missing, duplicated, renamed, or prompt-drifted roles as failure.
   - Role responses are analysis, not theatre: bind claims to the focal paper or public evidence; mark unavailable evidence explicitly; never fabricate a newer citing paper, author history, implementation run, product fact, or impact claim.
   - Do not contact authors or any external person without explicit user approval, even though the preserved Private Investigator prompt mentions contact.

5. **Reference policy**
   - Public page: `Reference / Evidence` contains only public online links.
   - Do not include `/home/...`, `file://`, `main.tex`, `sections/...`, `appendices/...`, private PDF paths, hidden prompts, unpublished logs, or private run roots.
   - If no public source exists, say the page is private-only and do not publish.

6. **Use the optional Gemini visual pass carefully**
   - `gemini_frontend_pass.sh` writes `index.gemini-draft.html`; it must never edit the source HTML.
   - Gemini may improve hierarchy, spacing, typography, callouts, responsive behavior, and component composition. It is not a factual source.
   - The script attaches the source boundary, material pack, TeX inventory, figure map, and grounding anchor when present. It requires preservation of role markers, verbatim prompts, figure markers, and evidence boundaries.
   - The primary agent must review and selectively merge useful visual changes back into `index.html`.

7. **Validate**
   - Run `scripts/validate_paper_html.sh <index.html> --public` from this skill.
   - It checks placeholders, public-path leaks, all seven role markers and verbatim prompts, response substance, final-section order, figure inventory/waiver coverage, hand-written KaTeX markup in source, rendered KaTeX output (`katex-mathml`, no `katex-error`), desktop/mobile screenshots, and mobile horizontal overflow.
   - Without a local Chrome the browser render checks are skipped automatically; treat a skipped-browser run as partial verification, not a full pass, and finish the browser checks on a machine that has Chrome.
   - Deliver user-visible artifacts through the active collaboration channel when supported, and report the validation result.

8. **Publish**
   - Local scripted path: the optional publisher requires a target repository with a compatible `pnpm research:publish` command and `PAPER2HTML_BLOG_ROOT`. `--ship` validates, imports, formats, checks, commits, pushes, watches deployment, and verifies the public URL. Use it only when the user explicitly requested those history and remote changes; the target repository owns staging, commit, and deployment behavior.
   - Web-agent path: for web-hosted models with a GitHub connector but no local checkout, follow `prompts/web-deploy.md` — it embeds the target repository contract (paths, `_meta.json`, HTML research meta tags) so the agent can push a validated page directly and the site's deploy workflow publishes it.

9. **Completion**
   - After creating the HTML, present a completion summary: validation results (KaTeX, overflow, screenshots, warnings), grounding anchor status (written / revised mid-writing / skipped with reason), and the deliverable path.
   - Mention the primary created file as Markdown inline code (e.g. `index.html`) so collaboration clients can render it as a link.
   - Offer the user the relevant follow-ups: open the page in a browser, publish to the research site, ingest the paper into a reference manager, archive the workspace, or process the next paper in a series. The delivery is a mid-point review gate, not the end.
   - If the session may continue later and work remains, write a short handoff note instead of relying on memory.

## Template

- `assets/cheatsheet-template.html` is the fixed visual skeleton. Copy it into the output directory, then fill content and embed images.
- Keep the template restrained and reading-focused. Do not turn paper pages into marketing landing pages.
