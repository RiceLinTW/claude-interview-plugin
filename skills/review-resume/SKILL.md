---
name: review-resume
description: Analyze a candidate's Resume.pdf and produce a structured Resume.Review.md with level assessment, tech stack analysis, experience timeline, red flags, strengths, and recommended quiz topics.
user-invocable: true
---

# Review Resume

Read and analyze the candidate's resume. Produce a structured review file.

## Step 1 — Resolve candidate name

If a name argument was provided, use it.
Otherwise, list directories under `{recruitment_root}/Candidate/Ongoing/` and ask the user to pick one. Active candidates live in `Ongoing/{name}/`; finished ones in `Archive/{name}/`.

## Step 2 — Read the resume (and any external CV)

Read `{recruitment_root}/Candidate/Ongoing/{name}/Resume.pdf`.

**If the resume points to an external CV** (e.g. a Cake.me link, "請忽略 104 內容，以下方連結為主", a personal site, or a GitHub), **fetch it** — the 104/email PDF is often a thin skeleton and the real content (job descriptions, tech stack, projects) lives on the linked CV. Base the review on the fuller source.

## Step 2.5 — Verify each company against official registration (Taiwan)

For **every company** in the candidate's work history, query the g0v company-registration API (source: 經濟部商業司 company registry + 財政部 business-tax registry):

```
https://company.g0v.ronny.tw/api/search?q=<公司名>
```

Fetch via Bash `curl -s -G ... --data-urlencode "q=<公司名>"` and save the JSON to the scratchpad (don't dump the whole payload into context); read the saved file. Notes:
- Try the exact registered name; if `found` is 0, try variants (e.g. 網絡↔網路, drop 股份/有限公司). Multiple hits (`found` > 1) → disambiguate by 所在地 / 核准設立日期.
- Extract per company: **公司狀況**（核准設立 / 解散 / 撤銷）、**資本總額**、**代表人**、**董監事名單**（esp. **所代表法人** — the representing legal entity reveals holding structure）、**所營事業資料**（business items）、**核准設立日期**、**統一編號**.

Cross-check against the resume and flag:
- **登記業務 vs 履歷描述落差** — e.g. business items are all generic 「資訊軟體服務業」 but the resume says 體育遊戲/娛樂城/博弈. A clean registration masking gambling/gray-industry work is a known pattern.
- **關聯公司 / 控股結構** — a 代表法人 sharing address + representative + capital with the company points to a one-person related-company cluster (common in gray industry). Follow the 代表法人 with another API query when it looks material.
- **時間軸矛盾** — 核准設立日期 later than the candidate's claimed start/tenure; or a very new / tiny-capital company described as a large 「集團」.
- **公司狀況** — 解散/撤銷 explains missing web presence (not itself a red flag if the company was operating during the candidate's tenure).

Record findings as a 「公司登記查證」 table in Resume.Review.md (統編 / 狀況 / 資本額 / 代表人 / 設立日 / 結論), and feed any matches into Red Flags below.

**Limits:** only Taiwan-registered entities (offshore/China parents won't appear); data refreshes monthly (經濟部) / daily (財政部), not real-time. Treat registry facts as harder evidence than web search, but note gaps rather than over-claiming.

## Step 3 — Analyze

Fill in the template at `${CLAUDE_PLUGIN_ROOT}/skills/review-resume/Resume.Review.template.md`:

**Level Assessment** — assign Junior / Mid / Senior with 3–5 bullet reasons based on:
- **Years of actual shipped iOS experience — count from the tech used + job content of each role, NOT the 職稱/職務類別 column.** 104's role-category years are often wrong (e.g. lists "iOS工程師 2~3年" when the actual hands-on iOS work across roles is 6+ years, or inflates with non-iOS roles). Re-derive iOS years yourself from what each job actually did.
- Company size / product quality
- Complexity of projects described
- Note **breadth-vs-depth shape**: a horizontal generalist (many stacks, shallow each) vs a deep specialist reads differently even at the same year count.

**Tech Stack** — for each technology in the resume:
- Rate evidence as: Shipped / Mentioned / Claimed only
- Flag technologies with many years claimed but no shipped project evidence

**Experience Timeline** — chronological table of all roles:
- Flag tenures < 2 years with ⚠️
- Flag unnamed companies, especially any matching the Skyline/誠帷/遊戲公司 pattern from `{recruitment_root}/面試流程腳本.md` (left ~2025/10) with 🚨

**Red Flags** — patterns that warrant scrutiny during interview (include any 公司登記查證 findings from Step 2.5: registration-vs-resume mismatch, related-company clusters, timeline contradictions)

**Strengths** — what to probe positively

**Recommended Quiz Topics** — ordered priority list:
- topic name + DB file from `{recruitment_root}/QuizDatabase/` + recommended level (Junior/Mid/Senior)
- Put the highest-relevance topics first (based on resume evidence and level assessment)

## Step 4 — Write output

Write to `{recruitment_root}/Candidate/Ongoing/{name}/Resume.Review.md`
using the template structure exactly.
