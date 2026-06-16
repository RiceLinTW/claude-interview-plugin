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

**Red Flags** — patterns that warrant scrutiny during interview

**Strengths** — what to probe positively

**Recommended Quiz Topics** — ordered priority list:
- topic name + DB file from `{recruitment_root}/QuizDatabase/` + recommended level (Junior/Mid/Senior)
- Put the highest-relevance topics first (based on resume evidence and level assessment)

## Step 4 — Write output

Write to `{recruitment_root}/Candidate/Ongoing/{name}/Resume.Review.md`
using the template structure exactly.
