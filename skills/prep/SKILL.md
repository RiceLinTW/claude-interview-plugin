---
name: prep
description: Generate a personalized interview script for a candidate, based on their Resume.Review.md and the 面試流程腳本.md template. Red-flag probes and tech-claim verification questions are clearly marked.
user-invocable: true
---

# Interview Prep

Generate a candidate-specific interview script before the interview.

## Step 1 — Resolve candidate

If a name argument was provided, use it. Otherwise ask.

## Step 2 — Read sources

1. `{recruitment_root}/Candidate/{name}/Resume.Review.md` — level, red flags, strengths, tech stack, experience timeline
2. `{recruitment_root}/面試流程腳本.md` — base interview framework and phase structure

## Step 3 — Generate personalized script

Use the template at `${CLAUDE_PLUGIN_ROOT}/skills/prep/Interview.Prep.template.md`.

**Phase 1 (Company intro):** Copy standard opening from 面試流程腳本.md. No personalization needed.

**Phase 2 (Self-intro):** Add 2–3 candidate-specific listening cues based on resume items
(technologies, projects, or tenures to note when the candidate mentions them).

**Phase 3 (Experience Q&A):**

For each 🚩 red flag from Resume.Review.md, generate a probe question. Mark each with 🚩:
- Short tenure (< 2 yr): "Your role at {company} lasted {N} months — can you walk me through why you left?"
- Unnamed/suspicious company: "I notice this role doesn't list the company name — can you tell me more about the company?"
- Claimed tech with no shipped evidence: "Your resume lists {tech} — can you describe a specific feature you shipped using it?"

For each Strength, generate 1–2 depth verification questions tailored to the specific tech or project.

**Always include an agentic-coding probe** in Phase 3 for EVERY candidate (regardless of whether the resume mentions it). The team uses a **Claude Team plan**, so confirming at least **basic agentic-coding capability** is a standing requirement — treat it as an evaluation dimension, not just a "keeps up with new tech" bonus.

Open the probe broadly, then drill into the workflow:
> 「你開發時有在用 AI 工具協助嗎？」→ follow-up（若有）：「可以聊聊你怎麼把它融進開發流程嗎？工作流大概怎麼設計？」

Keep the opener open — do NOT name specific tools. Then assess for **basic agentic competence**:
- Does the candidate actually use AI coding tools / agentic workflows (not just "heard of it")?
- How is the workflow designed — which steps it's used in, how they feed context, how they **verify / review the output**?
- Awareness of **limits and risks** (hallucinated APIs, over-trust, when NOT to use it).
- A candidate who designs a deliberate workflow + verifies output + knows the limits = passes this dimension. "Never used it" or blind trust = a gap to note.

Origin: `面試流程腳本.md` Phase 3 Senior 方向 (AI 工具協作). This is now a baseline expectation for all levels, not Senior-only.

Select 1–2 projects from the resume for "walk me through" deep-dives:
- Choose projects with the most complexity or the most years of tenure
- Ask about architecture decisions, challenges, team size, personal contribution

**Phase 4 (Candidate questions):** Use standard closing from 面試流程腳本.md.

## Step 4 — Write output

Write to `{recruitment_root}/Candidate/{name}/Interview.Prep.md`
