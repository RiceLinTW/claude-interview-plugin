---
name: quiz
description: Generate a 10-question bilingual (EN/ZH-TW) exam from the QuizDatabase, tailored to the candidate's level and resume review. Every question must trace back to a DB file entry.
user-invocable: true
---

# Quiz

Generate a 10-question exam for the candidate using the QuizDatabase.

## Step 1 — Resolve candidate name

If a name argument was provided, use it.
Otherwise, list `{recruitment_root}/Candidate/` directories and ask the user.

## Step 2 — Read Resume.Review.md

Read `{recruitment_root}/Candidate/{name}/Resume.Review.md`
Extract: **Assessed Level** and **Recommended Quiz Topics** (priority order).

## Step 3 — Select 10 questions

For each recommended topic (in priority order), open the relevant DB file in
`{recruitment_root}/QuizDatabase/` and select the most appropriate question(s) for the candidate's level.

**Mandatory rules:**
- Every question MUST exist in a DB file as a `## Q{N}: Title` entry
- If a question you want to use does NOT exist in any DB file, run `interview:quiz-db-update` to add it first, then use it
- Embed a source comment immediately before each question heading: `<!-- DB: {filename} Q{N} -->`
- Aim for ~5 conceptual + ~5 coding questions
- No duplicate questions compared to other exams the same candidate has received
- **Order questions easy → hard** (warm-up/syntax first, deepest architecture/design last)
- **Each question is single-language (EN *or* ZH-TW), NOT both.** Do not output the bilingual EN+ZH layout in the final Quiz.md. After picking the 10 questions, confirm the language of each question with the user one at a time before writing the file.

**Mandatory category balance (10 questions total):**
| Category | Min | Notes |
|----------|-----|-------|
| Swift | 2 | Core language knowledge |
| Architecture | 1 | Design patterns, MVVM, SOLID, DI, etc. |
| Git | 1 | At least one git workflow/command question |
| ObjC | 1 | At least one Objective-C question |
| SwiftUI | 1 | At least one SwiftUI question |
| Concurrency / Testing / Performance / SPM / Data | — | Fill remaining slots based on candidate profile |

If the Resume.Review.md recommended topics do not cover a mandatory category, select the most appropriate question from that DB file for the candidate's assessed level.

## Step 4 — Output Quiz.md

Use the template at `${CLAUDE_PLUGIN_ROOT}/skills/quiz/Quiz.template.md`.
Write to `{recruitment_root}/Candidate/Ongoing/{name}/Quiz.md`.

- The header metadata (姓名/日期/時限/題數) MUST stay inside an HTML comment `<!-- ... -->` so it does NOT render in Markdown preview / printed PDF.
- Each question body is the single chosen language only (no EN+ZH pairing). Keep the question's code block in the body when the DB entry's question has one.
- Keep the per-question `<div style="page-break-after: always;"></div>` separators — the PDF is produced via VSCode Markdown Preview Enhanced → print, which relies on them for one-question-per-page.

If a Quiz.md already exists, confirm with the user before overwriting.

## Step 5 — Remind to export PDF

The exam handed to the candidate is the PDF, not the .md. **Do NOT try to convert it yourself** (pandoc/LaTeX ignores the HTML `<div page-break>` and breaks one-question-per-page). Remind the user to export manually:

> Quiz.md is ready. To produce Quiz.pdf: open it in VSCode → Markdown Preview Enhanced → right-click the preview → "Open in Browser" (or Chrome) → Print → Save as PDF. The page-break divs give one question per page.
