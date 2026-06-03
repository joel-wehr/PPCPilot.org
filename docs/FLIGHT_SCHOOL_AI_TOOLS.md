# Flight School — AI Tools & Interactive Learning (Recommendations)

How to make the PPCPilot flight school an **interactive learning environment**, not
"test and test." Concrete, buildable recommendations on our actual stack, with the
latest (2025–2026) research behind each call.

**Last updated:** 2026-06-03
**Audience:** PPCPilot owner + build agents.
**Companion:** cross-domain patterns/techniques live in
[`AI_LEARNING_TOOLBOX.md`](./AI_LEARNING_TOOLBOX.md) — this doc is the PPC-specific
application of those.

---

## TL;DR — the two headline calls

1. **AI tutor: YES.** Build an always-on, RAG-grounded "CFI study buddy" that answers
   only from our knowledgebase + cited FAA reg text, uses Socratic hinting, and defers
   to a real CFI for anything that requires a sign-off or judgment call.
   *One-line why:* it's the single biggest "interactive, not just testing" upgrade, it's
   cheap on our existing Claude/Bedrock stack via prompt caching, and grounding +
   citation is exactly what keeps a *safety-critical aviation* tutor honest.

2. **Adaptive assessment approach:** keep an **explicit mastery model** (extend the
   existing `QuestionState`/spaced-repetition with per-subject **BKT-style mastery** and
   richer item metadata), use **distractor-level + LLM misconception diagnosis** to turn
   wrong answers into targeted next steps, and **feed that into the existing
   `recommendations.py` engine** so "what to study next" is driven by real performance,
   not just the profile. Do **not** rely on the LLM alone to track mastery — research is
   explicit that LLMs are not a reliable standalone adaptive-education engine.

Everything below is the detail and the phased roadmap.

---

## What we already have (the starting point)

From `ppc-pilot-web/backend/flightschool/`:
- **Content model:** `Module → Chapter` (markdown handbook, ~119k words imported via
  `import_knowledgebase`), plus `Subject` (ACS areas), `Question`/`Choice` (MCQ bank
  with `explanation`, `distractor_explanation`, `acs_task_code`, `reference_chapter`,
  `regulation_citation` → live eCFR fetch).
- **Practice:** study mode (`StudyQueueView` interleaves by subject; SM-2-lite via
  `QuestionState`) and exam mode (`_sample_exam_questions` weighted by
  `Subject.target_exam_questions`; 60Q / 70% / 2.5h).
- **Recommendations:** `recommendations.py` maps **pilot profile** (cert + status +
  goal) → modules + a practice exam. Advisory only, never gates content (owner decision).
- **Infra:** React/Vite + Django/DRF, AWS Lightsail, **Claude API access**, an existing
  **Bedrock content-generator**. eCFR integration already proves we can ground on
  authoritative reg text.

The gap the owner named: it's still fundamentally read-then-test. The four additions
that change that are a **tutor**, **performance-driven diagnosis**, **AI-assisted item
generation/feedback**, and **interactive scenarios**.

---

## 1. AI Tutor — recommendation: build it

### Verdict
**Yes, add an always-on AI tutor.** Current research and products converge on this being
the highest-leverage interactivity upgrade, and our stack already has every piece
(Claude API, a clean content corpus, eCFR grounding). Competitors validate the demand:
Sporty's 2026 course shipped **ChatCFI** (AI instructor), **ChatFAR** (reg search), and
**ChatDPE** (virtual checkride coach), plus an AI tool that analyzes FAA knowledge-test
results into a personalized study guide. We can do the PPC-native version — and the new
**MOSAIC Sport Pilot** rules make PPC/Sport content especially timely.
([Sporty's 2026 AI tools](https://flighttrainingcentral.com/2025/10/sportys-launches-2026-learn-to-fly-course-with-advanced-ai-tools-and-new-flight-maneuver-training/),
[AOPA review](https://www.aopa.org/news-and-media/all-news/2026/march/flight-training/product-review-sportys-learn-to-fly-2026-updates))

### Design (RAG-grounded, Claude-based, safety-railed)

**Retrieval / grounding.** Chunk our `Chapter.content_markdown` (by heading; we already
have `reference_section_anchor` for fine-grained anchors) into a vector index. Two viable
homes, both on AWS:
- **Aurora PostgreSQL + `pgvector`** (recommended). It's a first-class, AWS-documented
  Bedrock Knowledge Base vector store, and ~**$30–50/month** floor vs **~$700/month** for
  OpenSearch Serverless — a ~90% saving for a small KB.
  ([Bedrock KB → Aurora pgvector cost](https://ercanermis.com/cutting-amazon-bedrock-knowledge-base-costs-by-90-migrating-from-opensearch-serverless-to-aurora-serverless-v2-with-pgvector/),
  [AWS vector DB comparison](https://docs.aws.amazon.com/prescriptive-guidance/latest/choosing-an-aws-vector-database-for-rag-use-cases/vector-db-comparison.html))
- For an MVP we can even skip a managed Bedrock KB and do retrieval in Django (embed
  chunks, store in Postgres `pgvector`, top-k cosine) — keeps everything in the app we
  already run. NotebookLM-style RAG was shown **more accurate than feeding the same text
  raw** to a general LLM, and **traces claims to source** — exactly our requirement.
  ([NotebookLM RAG study](https://arxiv.org/html/2504.09720v2))

**Generation.** Claude via the existing API/Bedrock. Model routing for cost:
- **Haiku 4.5** (`$1/$5` per MTok) for hint-style turns and reg lookups.
- **Sonnet 4.6** (`$3/$15`) for normal tutoring/explanations.
- **Opus 4.8** (`$5/$25`) only for hard reasoning (misconception diagnosis, scenario
  authoring). ([Claude pricing 2026](https://www.cloudzero.com/blog/claude-api-pricing/))

**Cost-aware usage.**
- **Prompt-cache the static prefix** (system prompt + persona + retrieved chunks +
  few-shot). Cache reads cost ~10% of input; pays off after the first reuse. Set TTL
  deliberately — the 2026 default dropped to 5 min, which can raise costs 30–60% on
  long sessions if you assumed an hour.
  ([Prompt caching docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching),
  [TTL change](https://dev.to/whoffagents/claude-prompt-caching-in-2026-the-5-minute-ttl-change-thats-costing-you-money-4363))
- Cap per-user volume (Sporty's caps ChatCFI at 100 Q/month — a reasonable anti-abuse
  default).

**Persona + pedagogy.** A friendly **PPC CFI study buddy** ("Otto" or similar). The
research-backed behaviors:
- **Socratic by default** — ask a question / give a graduated hint before the answer;
  this is the core of every effective 2025 tutor (Khanmigo). But offer a **toggle** to
  "just explain it" — learners differ, and forcing Socratic when the learner isn't ready
  causes overload and *hurts* learning.
  ([Co-design takeaways](https://the-learning-agency.com/the-cutting-ed/article/co-designing-llm-tutors-for-student-success-five-key-takeaways-and-a-roadmap-for-developers/),
  [GuideEval](https://arxiv.org/html/2508.06583v1))
- **Pedagogical *prompting* matters more than model size.** LearnLM (pedagogically tuned
  Gemini) beat raw Claude/GPT/o3 on teaching quality 71–82% of the time — the lesson is
  that a strong pedagogical system prompt closes most of that gap on *our* model. Invest
  the effort in the system prompt: perceive the learner's state, adapt the strategy,
  elicit reflection. ([LearnLM in Gemini 2.5](https://blog.google/outreach-initiatives/education/google-gemini-learnlm-update/))

**Aviation accuracy / safety guardrail (non-negotiable for a flight school).**
- **Answer only from retrieved sources.** If retrieval returns nothing relevant, the
  tutor says "I don't have that in the handbook" and points to a CFI — it does **not**
  free-form a reg. RAG + an output guardrail is the standard hallucination control.
  ([IBM RAG guardrails](https://developer.ibm.com/tutorials/awb-how-to-implement-llm-guardrails-for-rag-applications/),
  [CustomGPT guardrails](https://customgpt.ai/ai-guardrails-how-to-prevent-llm-hallucinations/))
- **Cite every regulatory claim** to a Title 14 CFR section via the existing eCFR
  fetch — never paraphrase a reg without the citation and live text.
- **Defer to a CFI for anything requiring judgment or a sign-off** (endorsements, "am I
  ready to solo," go/no-go for a specific real flight, medical). The tutor teaches and
  quizzes; it does not authorize flight. State this in the persona and in the UI.
- **Bedrock Guardrails** (if we route via Bedrock) can add a topic/PII/grounding filter
  as defense-in-depth.
  ([Reducing hallucinations w/ Bedrock](https://aws.amazon.com/blogs/machine-learning/reducing-hallucinations-in-large-language-models-with-custom-intervention-using-amazon-bedrock-agents/))

**Where it lives in the UI.**
- A **persistent "Ask the CFI" panel** available across study/exam-review/handbook pages.
- **Context-aware entry points:** on a chapter, "Explain this section"; after a wrong
  answer in study mode, "Why is this the answer?" (pre-loaded with the question + the
  cited `explanation`/`distractor_explanation`); on a reg, "What does this mean for me?"
- Each answer renders **citations** (chapter + section anchor, or CFR section) as
  deep-links into content we already serve.

---

## 2. Adaptive assessment / diagnosis — "where am I, what next"

### Recommended approach (layered, explicit-model-first)

**Layer 1 — explicit mastery model (extend what exists).**
- Keep `QuestionState` for scheduling, but **upgrade SM-2 → FSRS**: ~20–30% fewer
  reviews for the same retention, more accurate forgetting prediction for 99.5% of users
  even on default params. Drop-in `py-fsrs`. This is a small, high-ROI change.
  ([FSRS vs SM-2](https://www.neurako.com/blog/fsrs-vs-sm2-spaced-repetition-algorithms-compared))
- Add a **per-(user, subject) mastery probability** updated BKT-style on each answer
  (the `SubjectViewSet` already computes a crude "mastered = streak≥2"; formalize it).
  BKT is interpretable, needs only our existing subject tags, and is the right default
  at our data scale. The literature is explicit: keep a theory-grounded knowledge model
  — don't let the LLM be the mastery tracker.
  ([KT + LLM review](https://arxiv.org/pdf/2412.09248))
- Enrich `Question` metadata for adaptivity: a **difficulty** field exists; add
  **discrimination** / observed p-value (correct-rate) so we can graduate from a
  difficulty heuristic to lightweight **IRT/CAT** item selection once we have response
  volume. CAT can estimate ability *and* flag misconceptions simultaneously.
  ([CAT + misconceptions](https://pmc.ncbi.nlm.nih.gov/articles/PMC7711247/))

**Layer 2 — LLM misconception diagnosis (the "why wrong").**
- **Cheap win first:** ensure every distractor has a one-line `distractor_explanation`
  ("someone who picks this confuses X with Y"). The field already exists — fill it
  (AI-assisted, human-reviewed; see §3).
- **Then:** after an exam or a study run, send the pattern of wrong answers (which
  distractors, on which `acs_task_code`s) to Claude (Opus for this) and ask it to name
  the **underlying misconception** and the **specific sections to review** — turning
  "60% on weather" into "you're mixing stable vs unstable lapse rates → reread
  03_weather §X." LLMs detect subtle misconceptions at a scale humans can't.
  ([LLM agents for education](https://aclanthology.org/2025.findings-emnlp.743.pdf))

**Layer 3 — feed the recommendation engine.**
- Today `recommendations.py` is **profile-only**. Extend `build_recommendations(profile)`
  to also take **performance signals**: weakest subjects (lowest BKT mastery), overdue
  FSRS cards, and the latest misconception diagnosis. The output payload already carries
  `recommended_modules` + `recommended_exam`; add a `focus_areas` block ("Review these 2
  sections; then a 15-question Weather drill") sourced from real performance.
- This keeps the owner's "advisory, never gates" rule while making the advice *earned*
  by the data, not just the stated goal.

**Net:** a closed loop — answer → update FSRS + BKT → diagnose misconceptions →
recommend next study + drill → re-test. That loop is the "interactive, not just testing"
core, and it's mostly an evolution of code we already have.

---

## 3. AI question/quiz generation + grading + feedback

**Generation (scale the bank from the corpus).**
- Pipeline: for a chapter, retrieve the passage → Claude drafts MCQs **with** the correct
  answer keyed to the passage, `explanation`, per-distractor `distractor_explanation`,
  `acs_task_code`, and `reference_chapter`/`reference_section_anchor`. This fills our
  existing schema directly.
- **Generate-then-verify with a human gate** — the 2025 consensus is never to ship
  AI-generated items un-reviewed. Add an `is_active=False` "review queue" + a verifier
  pass (second LLM scores relevance/correctness/alignment, MIRROR-style) + owner approval
  before `is_active=True`. Tie every item to its source passage so review is fast.
  ([Verifiable QG + grading](https://link.springer.com/chapter/10.1007/978-3-031-99264-3_22),
  [MIRROR auto-eval](https://arxiv.org/pdf/2410.12893))

**Free-response grading + feedback (new capability, big interactivity gain).**
- Add short-answer / "explain it" items (oral-exam style — directly relevant to a
  checkride). Grade with Claude **bound to an explicit rubric**, return targeted
  feedback, and route low-confidence scores to the owner/CFI (LLM-as-filter,
  human-in-the-loop — the 2025 best practice). Rubric quality is everything; answer-
  matching graders are unreliable.
  ([LLM grader + guidelines](https://educationaldatamining.org/EDM2025/proceedings/2025.EDM.long-papers.80/index.html),
  [Human-in-the-loop grading](https://arxiv.org/pdf/2504.05239),
  [Scoring + feedback](https://arxiv.org/pdf/2405.00602))

**Reuse the existing Bedrock content-generator** for the generation/verification jobs —
it's already in place; this is an extension, not a new system.

---

## 4. Interactive environments (the "not just testing" payoff)

Ranked by realism-on-our-stack:

**a) Scenario-based ADM practice (generative branching) — high value, very buildable.**
- Aviation training is fundamentally about **aeronautical decision-making**. Use Claude
  to run **generative branching scenarios** (weather go/no-go, deteriorating conditions,
  engine-out site selection, traffic-pattern conflicts) that react in context to the
  learner's choices and remember state across the session — far better than scripted
  trees. Ground the scenario "world" and the after-action debrief in our handbook +
  regs, with a rubric so it stays on-pedagogy and on-fact. Industry reports cite ~80–90%
  completion and 70–80% 30-day retention for AI roleplay (vendor data; directional).
  ([AI scenario roleplay](https://www.jenova.ai/en/resources/ai-scenario-roleplay),
  [conversation simulator](https://www.jenova.ai/en/resources/ai-conversation-simulator))
- Store each run as a transcript for owner/CFI review; score the decisions against the
  rubric and feed weak ADM areas back into Layer 3 recommendations.

**b) AI-powered spaced repetition — already half-built, finish it.**
- We have `QuestionState`. Move to **FSRS** (above), surface a daily "due" widget, and
  let the **tutor generate elaborative-interrogation prompts** ("why does aft CG reduce
  stability?") as recall items, not just MCQs. Active recall + spacing is the highest-ROI,
  most-evidenced lever we have.

**c) Conversational practice / "oral exam coach" — buildable now.**
- A focused tutor mode that role-plays a **DPE oral exam** for the PPC checkride: asks
  ACS-task questions, follows up Socratically, grades free-text answers (§3), debriefs.
  This is exactly Sporty's ChatDPE, PPC-native. Reuses the tutor + free-response grader.

**d) Voice — realistic but Phase 3.**
- Web Speech API (TTS/STT) for a **hands-free checklist / walk-around trainer** and
  voice oral-exam practice. Genuinely useful for a kinesthetic, around-the-aircraft task,
  but it's polish, not core — defer until the text loop is solid. Watch latency/accuracy.

**e) Interactive widgets (H5P) — buy-not-build for diagrams.**
- For aircraft-systems diagrams, image hotspots, and label-the-parts drills, embed
  **H5P** rather than hand-building widgets. Complements the AI features; cheap interactivity.

---

## 5. Build recommendations (use vs build) + phased roadmap

### Use / integrate vs build
| Capability | Decision | Why |
|---|---|---|
| Tutor LLM | **Use** Claude (have it) | No reason to self-host; route models for cost |
| Vector store | **Use** Postgres `pgvector` (have Postgres) | First-class Bedrock support; ~90% cheaper than OpenSearch |
| RAG orchestration | **Build** (thin) in Django, or Bedrock KB | MVP can be ~100 lines; KB if we want managed |
| Pedagogy / Socratic behavior | **Build** (prompts) | Pedagogical prompting ≈ most of LearnLM's edge |
| Spaced repetition | **Build** on FSRS lib | Drop-in `py-fsrs`; we own the data |
| Knowledge tracing | **Build** BKT (small) | Interpretable, fits our scale; LLM ≠ mastery tracker |
| Question generation | **Build** on existing Bedrock generator | Extends what exists; fills our schema |
| Interactive diagrams | **Use** H5P | Cheap interactivity, no widget engineering |
| Roleplay/scenario engine | **Build** thin on Claude | Vendors are buy-only; pattern ports easily |
| Voice | **Use** Web Speech API | Native, free; defer to Phase 3 |

### Cost & accuracy guardrails (carry into every phase)
- **Cost:** prompt-cache static prefixes; route Haiku/Sonnet/Opus by task; batch
  non-interactive jobs (generation, diagnosis); per-user usage caps. Expect a hobby-scale
  bill in the low tens of $/month plus ~$30–50 Aurora floor if used.
- **Accuracy/safety:** RAG-only answers + citations; eCFR for all reg claims; "defer to
  CFI" for sign-offs/judgment; human review gate on all generated items and on
  low-confidence free-response grades; log transcripts.

### Phased roadmap
**Phase 0 — groundwork (small).** Swap SM-2 → FSRS in `scheduler.py`/`QuestionState`;
add `discrimination`/observed-p-value to `Question`; backfill `distractor_explanation`
(AI-assisted, reviewed).

**Phase 1 — MVP tutor (the headline).** `pgvector` index over chapters; thin Django RAG
endpoint; Claude (Sonnet) with the Socratic + safety system prompt; citations + eCFR;
"Ask the CFI" panel + context entry points; prompt caching + usage cap. *Ships the
biggest interactivity win.*

**Phase 2 — adaptive loop.** Per-subject BKT mastery; post-exam LLM misconception
diagnosis (Opus, batched); extend `recommendations.py` with `focus_areas` from real
performance. Add the AI question-generation pipeline with the review queue.

**Phase 3 — richer interaction.** Generative ADM branching scenarios + DPE oral-exam
coach (free-response grading, rubric-bound, human-in-the-loop); H5P diagrams; optional
voice (Web Speech) for checklist/oral practice.

**Phase 4 — polish/scale.** IRT/CAT once item-response volume supports calibration;
analytics dashboards; consider managed Bedrock KB if retrieval scale warrants.

This is an **evolution of the existing `flightschool` app**, not a rebuild — it
supersedes the earlier Moodle recommendation in
`ppc-knowledgebase/INTERACTIVE_COURSE_PLATFORM_ANALYSIS.md` (we already built custom on
Django/React, which that doc listed as the high-end option).

---

## Sources
- [Sporty's 2026 AI tools (ChatCFI/ChatFAR/ChatDPE)](https://flighttrainingcentral.com/2025/10/sportys-launches-2026-learn-to-fly-course-with-advanced-ai-tools-and-new-flight-maneuver-training/) · [AOPA review](https://www.aopa.org/news-and-media/all-news/2026/march/flight-training/product-review-sportys-learn-to-fly-2026-updates)
- [Co-designing LLM tutors — 5 takeaways](https://the-learning-agency.com/the-cutting-ed/article/co-designing-llm-tutors-for-student-success-five-key-takeaways-and-a-roadmap-for-developers/)
- [GuideEval / Socratic LLM eval (arXiv 2508.06583)](https://arxiv.org/html/2508.06583v1)
- [LearnLM in Gemini 2.5 (Google)](https://blog.google/outreach-initiatives/education/google-gemini-learnlm-update/) · [LearnLM report (arXiv 2412.16429)](https://arxiv.org/pdf/2412.16429)
- [Khanmigo growth](https://aiforcause.org/stories/khanmigo-ai-tutor)
- [NotebookLM RAG for tutoring (arXiv 2504.09720)](https://arxiv.org/html/2504.09720v2)
- [RAG guardrails (IBM)](https://developer.ibm.com/tutorials/awb-how-to-implement-llm-guardrails-for-rag-applications/) · [Guardrails (CustomGPT)](https://customgpt.ai/ai-guardrails-how-to-prevent-llm-hallucinations/) · [Bedrock anti-hallucination](https://aws.amazon.com/blogs/machine-learning/reducing-hallucinations-in-large-language-models-with-custom-intervention-using-amazon-bedrock-agents/)
- [KT + LLM systematic review (arXiv 2412.09248)](https://arxiv.org/pdf/2412.09248) · [Dialogue-KT (LAK 2025)](https://github.com/umass-ml4ed/dialogue-kt) · [LLM agents for education (EMNLP 2025)](https://aclanthology.org/2025.findings-emnlp.743.pdf)
- [CAT misconception detection (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC7711247/)
- [Verifiable QG + grading (Springer)](https://link.springer.com/chapter/10.1007/978-3-031-99264-3_22) · [MIRROR (arXiv 2410.12893)](https://arxiv.org/pdf/2410.12893) · [LLM grader, guidelines (EDM 2025)](https://educationaldatamining.org/EDM2025/proceedings/2025.EDM.long-papers.80/index.html) · [Grading human-in-the-loop (arXiv 2504.05239)](https://arxiv.org/pdf/2504.05239) · [Scoring + feedback (arXiv 2405.00602)](https://arxiv.org/pdf/2405.00602)
- [AI scenario roleplay (Jenova)](https://www.jenova.ai/en/resources/ai-scenario-roleplay) · [Conversation simulator (Jenova)](https://www.jenova.ai/en/resources/ai-conversation-simulator) · [AI training platforms (Virti)](https://www.virti.com/insights/news/7-best-ai-training-platform/)
- [FSRS vs SM-2 (Neurako)](https://www.neurako.com/blog/fsrs-vs-sm2-spaced-repetition-algorithms-compared)
- [Prompt caching (Claude docs)](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) · [TTL change (DEV)](https://dev.to/whoffagents/claude-prompt-caching-in-2026-the-5-minute-ttl-change-thats-costing-you-money-4363) · [Claude pricing 2026 (CloudZero)](https://www.cloudzero.com/blog/claude-api-pricing/)
- [Bedrock KB → Aurora pgvector cost](https://ercanermis.com/cutting-amazon-bedrock-knowledge-base-costs-by-90-migrating-from-opensearch-serverless-to-aurora-serverless-v2-with-pgvector/) · [AWS vector DB comparison](https://docs.aws.amazon.com/prescriptive-guidance/latest/choosing-an-aws-vector-database-for-rag-use-cases/vector-db-comparison.html)
