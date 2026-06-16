# Interview Plugin for Claude Code

An iOS-engineer interview-prep pipeline packaged as a [Claude Code](https://claude.com/claude-code) plugin. It turns a candidate's résumé into a full interview kit — résumé review, a DB-backed written exam, a scoresheet, and a personalized interview script — and closes the loop by scoring the exam after the interview.

Skills are namespaced under `interview:` once the plugin is installed.

## Skills

| Skill | What it does |
|---|---|
| `interview:prepare-candidate` | **End-to-end orchestrator** — runs the whole pipeline and produces every document for a candidate in one go. Start here. |
| `interview:calendar` | Read upcoming interview events from macOS Calendar.app (AppleScript, read-only) to pull dates automatically. |
| `interview:review-resume` | Analyze a candidate's `Resume.pdf` (and any linked external CV) → structured `Resume.Review.md` with level assessment, tech-stack evidence rating, timeline, red flags, strengths, and recommended quiz topics. |
| `interview:quiz` | Generate a 10-question, single-language-per-question exam from your question database, balanced across mandatory categories → `Quiz.md`. |
| `interview:quiz-db-update` | Append / remove / renumber a bilingual (EN + ZH-TW) question in the question database. |
| `interview:quiz-scoresheet` | Categorized scoresheet with per-question reference answers and a blank score table → `Quiz.Scoresheet.md`. |
| `interview:prep` | Phased interview script with red-flag probes, tech-claim verification, project deep-dives, and an agentic-coding baseline probe → `Interview.Prep.md`. |
| `interview:score-quiz` | Post-interview: score the exam, compute pass/fail against the level threshold, and write `Record.md` + update the master `Records.md` index. |

### How they fit together

```
NEW CANDIDATE (Resume.pdf in {recruitment_root}/Candidate/Ongoing/{name}/)
        │
  interview:prepare-candidate  ──────────── orchestrates ↓
        ├─ interview:calendar        → interview date/time
        ├─ interview:review-resume   → Resume.Review.md   (level, red flags, tech)
        ├─ interview:quiz            → Quiz.md            (10 Q, DB-backed, easy→hard)
        │     └─ interview:quiz-db-update  (only if a needed Q is missing / needs edit)
        ├─ interview:quiz-scoresheet → Quiz.Scoresheet.md (DB answer keys + blank scores)
        └─ interview:prep            → Interview.Prep.md  (script + probes)
        → add row to Candidate/Records.md (Pending)

AFTER INTERVIEW
        └─ interview:score-quiz      → Record.md + Records.md
```

## Install

```
/plugin marketplace add RiceLinTW/claude-interview-plugin
/plugin install interview@interview-plugin
```

Then restart Claude Code (or `/plugin` → reload). The `interview:*` skills become invocable.

## Setup — required, read this first

The skills do **not** ship your interview content. They reference a few files and a directory layout that **you** provide in your own workspace. Configure the location once (e.g. in your project `CLAUDE.md`) and the skills follow it.

### 1. `{recruitment_root}`

Every path in the skills is written relative to a placeholder, `{recruitment_root}` — the folder where your recruitment material lives. The original author used `Interview/Recruitment/`; set it to whatever suits you and tell Claude in your `CLAUDE.md`, e.g.:

```md
## Interview plugin
- `{recruitment_root}` = `Interview/Recruitment` (relative to the repo root)
```

### 2. Candidate folder layout

```
{recruitment_root}/
  Candidate/
    Ongoing/{name}/        # active candidates: Resume.pdf goes here
      Resume.pdf
    Archive/{name}/        # moved here after the interview is filed (NOT a verdict)
    Records.md             # master index, one row per candidate
  QuizDatabase/            # your question bank (see below)
  面試流程腳本.md           # your base interview script / framework  (user-supplied)
  測驗評分標準.md           # your scoring criteria + pass thresholds   (user-supplied)
```

### 3. Question database (`QuizDatabase/`)

`interview:quiz` selects questions from per-topic Markdown files named
`ios_exam_<topic>_question_database.md` (topics: `swift`, `concurrency`,
`architecture`, `git`, `spm`, `swiftui`, `testing`, `uikit`, `objc`,
`performance`, `data`). Each question is a `## Q{N}: Title` heading with EN +
ZH-TW bodies and an **Answer** section. `interview:quiz-db-update` appends new
ones in this format. Start the files empty and let the DB grow as you interview.

### 4. The two reference docs

- **`面試流程腳本.md`** — your standing interview framework (phase structure, standard openings/closings). `interview:prep` weaves candidate-specific probes into it.
- **`測驗評分標準.md`** — scoring weights and the level pass thresholds (the author used Junior ≥ 60 / Mid ≥ 65 / Senior ≥ 70). `interview:score-quiz` and `interview:quiz-scoresheet` read these.

Both are intentionally **not** included — they're yours to write.

## Conventions baked into the skills

- **Single-language questions.** Each quiz question is EN *or* ZH-TW, not both. Header metadata sits in an HTML comment so it doesn't render in the printed PDF.
- **DB traceability.** Every quiz question carries a `<!-- DB: {file} Q{N} -->` comment so the scoresheet/scorer can find its reference answer.
- **Three separate layers.** (1) `Record.md` / scoresheet 結論 = *recommendation*; (2) `Records.md` 結果欄 = *final decision*, set by a human, batched after the whole round — the scorer leaves it `Pending`; (3) `Ongoing/`↔`Archive/` = *lifecycle stage*, not a verdict.
- **PDF export is manual.** The quiz uses HTML `page-break` divs for one-question-per-page; export via VSCode → Markdown Preview Enhanced → print. Don't auto-convert with pandoc — it breaks the page breaks.
- **Agentic-coding baseline probe** is included for every candidate.

## Note on named companies

`interview:review-resume` flags a couple of specific company names as résumé red flags. These reflect companies already covered in public Taiwanese news reporting (fraud / money-laundering cases). Adjust or remove them for your own context.

## License

MIT — see [LICENSE](LICENSE).
