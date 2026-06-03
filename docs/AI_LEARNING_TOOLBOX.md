# AI Learning Toolbox (Cross-Domain)

A reusable, running reference of AI tools, platforms, techniques, and pedagogical-AI
patterns that apply to teaching **any** subject — not just aviation. Mined while
researching the PPCPilot flight school, but everything here is domain-agnostic and
meant to be reused on future projects. Keep adding to it.

**Last updated:** 2026-06-03
**Scope:** Latest (2025–2026) only. When an entry ages out, replace it rather than
stacking legacy notes.

---

## How to use this doc

Three buckets:

1. **Patterns** — reusable designs (RAG tutor, knowledge tracing, adaptive item
   selection). These are *what to build*.
2. **Techniques / algorithms** — named methods with evidence (FSRS, IRT, active
   recall, prompt caching). These are *how to make it work and keep it cheap*.
3. **Platforms / products** — things you can buy, fork, or copy ideas from
   (LearnLM, Khanmigo, MagicSchool, NotebookLM, H5P). These are *prior art*.

For each: what it is, where it shines, watch-outs, link.

---

## 1. Pedagogical-AI patterns

### Socratic / guided tutoring (don't give the answer)
- **What:** The tutor LLM is instructed to ask questions, give graduated hints, and
  make the learner construct the answer — "pedagogical alignment" — rather than
  solving it for them. This is the single most-cited design choice in 2025 tutoring
  research and the core of Khanmigo.
- **Where it shines:** Concept mastery, problem-solving, anything where doing the
  reasoning *is* the learning. Offer a toggle: some learners want Socratic, some want
  a direct concise answer — let them choose (research finding).
- **Watch-out:** Rigid/linear questioning misaligned with the learner's readiness
  causes cognitive overload and *hurts* learning. Good tutors perceive learner state
  first, then adapt (the GGuideEval "Perception → Orchestration → Elicitation" frame).
  Generic models default to bypassing the learning process with direct solutions.
- **Links:** [Co-designing LLM tutors — 5 takeaways (Learning Agency)](https://the-learning-agency.com/the-cutting-ed/article/co-designing-llm-tutors-for-student-success-five-key-takeaways-and-a-roadmap-for-developers/) · [GuideEval: Discerning Minds or Generic Tutors? (arXiv 2508.06583)](https://arxiv.org/html/2508.06583v1) · [Aligning LLMs with pedagogy via RL (arXiv 2505.15607)](https://arxiv.org/pdf/2505.15607)

### RAG-grounded tutor (cite your own corpus; don't hallucinate)
- **What:** Retrieval-Augmented Generation — retrieve passages from *your* trusted
  content, put them in the prompt, instruct the model to answer only from them and
  cite the source. Turns a general LLM into a domain expert that quotes your material.
- **Where it shines:** Any field with a canonical body of truth (regs, manuals,
  textbooks, SOPs, policy) where a wrong answer is costly. A 2025 medical study found
  NotebookLM's RAG was significantly more accurate than handing the *same* reference
  text to a general LLM, and reliably traced claims back to source.
- **Watch-out:** RAG reduces but doesn't eliminate hallucination — pair it with an
  output guardrail and a "I don't have a source for that" fallback. Chunking and
  retrieval quality dominate the result.
- **Links:** [NotebookLM RAG for tutoring (arXiv 2504.09720)](https://arxiv.org/html/2504.09720v2) · [LLM guardrails for RAG (IBM)](https://developer.ibm.com/tutorials/awb-how-to-implement-llm-guardrails-for-rag-applications/) · [Reduce hallucination with RAG (CustomGPT)](https://customgpt.ai/ai-guardrails-how-to-prevent-llm-hallucinations/)

### Knowledge tracing (model what the learner knows over time)
- **What:** A per-learner, per-skill estimate of mastery that updates with every
  answer. Lets you pick the next item, decide "mastered vs needs review," and explain
  *why*.
- **Spectrum:**
  - **BKT (Bayesian Knowledge Tracing):** simple, interpretable, per-skill probability.
    Needs skill-tagged items. Great default for small platforms.
  - **DKT / deep KT (LSTM/attention/GNN):** more accurate, captures skill dependencies,
    but needs lots of data and is harder to interpret.
  - **LLM-based KT (2025):** emerging — extract knowledge components from content,
    trace mastery from tutor-student *dialogue* (not just MCQ logs). Promising but the
    literature warns LLMs are **not** a general-purpose adaptive-education engine on
    their own; pair with an explicit, theory-grounded knowledge model.
- **Watch-out:** A common 2025 misconception is "the LLM can just track mastery." It
  can't reliably — keep an explicit state model (even BKT) as the source of truth.
- **Links:** [Systematic review: KT + LLMs (arXiv 2412.09248)](https://arxiv.org/pdf/2412.09248) · [Dialogue-KT with LLMs, LAK 2025 (code)](https://github.com/umass-ml4ed/dialogue-kt) · [Practical eval of deep KT for platforms (EDM 2025)](https://educationaldatamining.org/EDM2025/proceedings/2025.EDM.industry-papers.46/index.html)

### Adaptive item selection (ask the right question next)
- **What:** Pick the next question to maximize information about the learner —
  classically **IRT/CAT** (Computerized Adaptive Testing): each item has
  difficulty/discrimination params, you serve items near the learner's current ability
  estimate. Modern variants detect *misconceptions* while estimating ability.
- **Where it shines:** Placement, diagnostics, efficient testing (fewer items, same
  precision). Combine with KT for a continuous study loop.
- **Watch-out:** Real IRT needs calibrated item params (response data). Cold-start with
  a heuristic (difficulty tags + recency + error-rate) and graduate to IRT once you
  have volume.
- **Links:** [CAT to detect misconceptions + estimate ability (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC7711247/) · [Option tracing in KT (JLA)](https://learning-analytics.info/index.php/JLA/article/view/8557)

### LLM misconception diagnosis (read *why* they got it wrong)
- **What:** Feed the learner's wrong answers (which distractor they chose, free-text
  rationale, or dialogue) to an LLM and ask it to name the underlying misconception,
  then target remediation at *that*, not the topic broadly.
- **Where it shines:** Turning "you got 60% on weather" into "you confuse stability
  with instability in the lapse rate — here's the fix." Distractor-level analysis is
  the cheap win: write a one-line "why someone picks this wrong answer" per distractor.
- **Watch-out:** Validate against known misconception taxonomies where they exist;
  don't let the model invent a misconception that isn't there.

### Generate-then-verify (item generation with a review gate)
- **What:** LLM generates practice items / rubrics; a second pass (LLM verifier +
  human spot-check) validates correctness, alignment, and quality before items go live.
  The 2025 consensus: never ship AI-generated assessment items un-reviewed.
- **Where it shines:** Scaling a question bank from a content corpus. Auto-evaluators
  like MIRROR score generated questions on relevance/novelty/complexity/grammaticality.
- **Watch-out:** Hallucinated "facts" and mis-keyed answers. Always tie generated items
  back to a cited source passage; queue for human approval.
- **Links:** [LLM agents for verifiable QG + grading (Springer)](https://link.springer.com/chapter/10.1007/978-3-031-99264-3_22) · [MIRROR: auto-eval of question generation (arXiv 2410.12893)](https://arxiv.org/pdf/2410.12893) · [QG + assessment survey (arXiv 2410.09576)](https://arxiv.org/pdf/2410.09576)

### LLM grading of free responses (rubric-bound, human-in-the-loop)
- **What:** Grade short-answer/essay responses with an LLM **bound to an explicit
  rubric**, then route low-confidence or out-of-band scores to a human. This
  "LLM-as-filter" hybrid is the 2025 best practice for non-MCQ grading.
- **Where it shines:** Scaling formative feedback; consistency across many learners.
- **Watch-out:** Rubric quality is everything — graders that just "answer-match" are
  unreliable. Calibrate a tolerance band; humans review outside it.
- **Links:** [LLM grader with human-level guideline optimization (EDM 2025)](https://educationaldatamining.org/EDM2025/proceedings/2025.EDM.long-papers.80/index.html) · [Automated grading with human-in-the-loop (arXiv 2504.05239)](https://arxiv.org/pdf/2504.05239) · [Scoring + feedback with LLMs (arXiv 2405.00602)](https://arxiv.org/pdf/2405.00602)

### Generative branching scenarios (vs scripted)
- **What:** Instead of a hand-authored decision tree, an LLM improvises a realistic
  scenario, reacts to the learner's choices in context, and remembers state across the
  session. Scripted branching is rigid; generative is open-ended and adaptive.
- **Where it shines:** Decision-making, judgment, "difficult conversation," and
  consequence-based practice. Industry reports cite 80–90% completion and 70–80%
  30-day retention for AI roleplay vs ~half that for conventional courses (vendor data,
  treat as directional).
- **Watch-out:** Open-ended generation can drift off-pedagogy or off-fact — constrain
  with a rubric/world-model and a grounded knowledge source; log transcripts for review.
- **Links:** [AI scenario roleplay (Jenova)](https://www.jenova.ai/en/resources/ai-scenario-roleplay) · [AI conversation simulator (Jenova)](https://www.jenova.ai/en/resources/ai-conversation-simulator) · [AI training platforms 2026 (Virti)](https://www.virti.com/insights/news/7-best-ai-training-platform/)

---

## 2. Techniques & algorithms (evidence-backed)

### FSRS (Free Spaced Repetition Scheduler) — use over SM-2
- **What:** ML-trained spaced-repetition scheduler (2023, trained on ~700M reviews)
  that predicts your forgetting curve per card. The modern replacement for SM-2 (1987).
- **Evidence:** ~20–30% fewer reviews for the same retention vs SM-2; more accurate
  recall prediction than SM-2 for 99.5% of users *even with default params*. FSRS-5
  hit ~89.6% success in a 2025 multi-algorithm benchmark.
- **Where it shines:** Any flashcard/active-recall system. Open-source implementations
  exist in many languages (`py-fsrs`, `ts-fsrs`, etc.); the Anki default since 2024.
- **Watch-out:** Per-user parameter optimization helps but isn't required to beat SM-2.
- **Links:** [FSRS vs SM-2 (Neurako)](https://www.neurako.com/blog/fsrs-vs-sm2-spaced-repetition-algorithms-compared) · [FSRS vs SM-2 deep guide (MemoForge)](https://memoforge.app/blog/fsrs-vs-sm2-anki-algorithm-guide-2025/)

### Core learning-science levers (cheap, high-ROI, model-agnostic)
These predate AI but AI makes them easy to deliver at scale:
- **Active recall** — retrieval beats re-reading (30–50% better on assessments).
- **Spaced repetition** — expanding intervals; 50–200% retention gain vs massed.
- **Interleaving** — mix topics; harder short-term, better transfer.
- **Microlearning** — 8–12 min single-objective chunks; +20–30% retention, better
  completion for time-constrained adults.
- **Elaborative interrogation** — "why/how" prompts; LLMs generate these for free.
- **Multimedia (Mayer):** dual-coding, segmenting, pre-training.

### Prompt caching (the cost lever for any LLM-tutor)
- **What:** Cache the static prefix of a prompt (system prompt, retrieved corpus,
  persona, few-shot). Cache **reads cost ~10%** of normal input price.
- **Numbers (Claude, 2026):** cache read = 0.1× input on every model; 5-min cache write
  = 1.25× input, 1-hour write = 2× input. Pays off after **one** read (5-min) or **two**
  reads (1-hour). Combine with the Batch API (50% off) for up to ~95% total savings.
- **2026 gotcha:** Default TTL moved from 60 min → 5 min; long-lived sessions that
  assumed a 1-hour cache saw 30–60% cost increases. Set TTL deliberately.
- **Links:** [Prompt caching (Claude docs)](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) · [Cache pricing explained (TokenMix)](https://tokenmix.ai/blog/claude-api-cache-pricing) · [5-min TTL change (DEV)](https://dev.to/whoffagents/claude-prompt-caching-in-2026-the-5-minute-ttl-change-thats-costing-you-money-4363)

### Model routing (right model for the task)
- **Pattern:** Route cheap/high-volume turns (grading an MCQ explanation, generating a
  hint) to a small model; reserve the large model for hard reasoning (misconception
  diagnosis, scenario generation, free-response grading). 5–25× savings.
- **Claude lineup (June 2026), $/MTok in→out:** Haiku 4.5 `$1 → $5`; Sonnet 4.6
  `$3 → $15`; Opus 4.8 `$5 → $25`. Opus/Sonnet support 1M-token context flat-rate.
- **Links:** [Claude API pricing 2026 (CloudZero)](https://www.cloudzero.com/blog/claude-api-pricing/) · [Pricing (Claude docs)](https://platform.claude.com/docs/en/about-claude/pricing)

---

## 3. Platforms & products (prior art to copy or integrate)

### LearnLM (Google, in Gemini 2.5) — pedagogically tuned model
- **What:** Gemini fine-tuned for learning via "pedagogical instruction following" —
  you specify per-turn teaching behavior in the system prompt rather than hard-coding
  one theory. Folded into Gemini 2.5 at I/O 2025.
- **Why it matters:** In head-to-head pedagogy evals, Gemini 2.5 Pro (LearnLM) was
  preferred ~71% vs Claude 3.7 Sonnet, ~82% vs GPT-4o, ~74% vs o3 — evidence that
  *pedagogical tuning/prompting* beats raw capability for teaching. The lesson is
  portable: even on a non-Gemini stack, **invest in pedagogical system prompts**.
- **Links:** [LearnLM in Gemini 2.5 (Google blog)](https://blog.google/outreach-initiatives/education/google-gemini-learnlm-update/) · [LearnLM technical report (arXiv 2412.16429)](https://arxiv.org/pdf/2412.16429) · [Evaluating Gemini in a learning arena (arXiv 2505.24477)](https://arxiv.org/pdf/2505.24477)

### Khanmigo (Khan Academy) — the reference Socratic tutor
- **What:** GPT-4-based Socratic tutor; never gives the answer, asks at nearly every
  turn. 68k → 700k users in a year, 380+ districts. The canonical product to study for
  tutor UX and guardrails.
- **Link:** [Khanmigo growth story](https://aiforcause.org/stories/khanmigo-ai-tutor)

### MagicSchool — AI *teacher* assistant (authoring side)
- **What:** Suite of generators for teachers (lesson plans, spiral review, assessments
  aligned to standards). Good model for the **content-creation** half of a platform,
  distinct from the learner-facing tutor.
- **Link:** [AI tools for teachers overview](https://www.tomdaccord.com/ai-tools-for-math-teachers)

### NotebookLM (Google) — RAG-over-your-docs, productized
- **What:** Upload sources, ask grounded questions, get cited answers; also generates
  audio overviews. Strong reference implementation of "RAG tutor over a fixed corpus."
- **Link:** [NotebookLM as RAG tutor (arXiv 2504.09720)](https://arxiv.org/html/2504.09720v2)

### H5P — open-source interactive content
- **What:** Free library of interactive content types — branching scenarios, image
  hotspots, drag-and-drop, interactive video, fill-in-blank. Embeddable in any LMS or
  custom site. The pragmatic way to add interactivity without building widgets from
  scratch.
- **Link:** [h5p.org](https://h5p.org)

### AI roleplay/simulation platforms (judgment & conversation practice)
- **What:** Vendors (Virti, Mindtickle, CGS Cicero, Jenova) sell generative roleplay
  for sales/support/clinical training. Useful as **idea sources** for scenario UX;
  most are buy-not-build, but the patterns (persona, memory, after-action debrief) port
  cleanly to a custom build.
- **Links:** [Best AI roleplay simulators 2026 (Mindtickle)](https://www.mindtickle.com/blog/best-ai-roleplay-simulator-tools-for-contact-center-training/) · [Virti](https://www.virti.com/insights/news/7-best-ai-training-platform/)

---

## 4. Decision shortcuts (reusable rules of thumb)

- **Building a tutor?** Start RAG-grounded over your own corpus + a Socratic system
  prompt + an "only answer from sources / defer to an expert for sign-offs" guardrail.
  Pedagogical *prompting* gets you most of LearnLM's edge on any model.
- **Building assessment intelligence?** Explicit knowledge model (BKT to start) +
  distractor-level misconception notes + LLM diagnosis layered on top. Don't trust the
  LLM alone to track mastery.
- **Generating content?** Always generate-then-verify with a human gate, and tie every
  item to a cited source.
- **Spaced repetition?** Use FSRS, not SM-2.
- **Cost?** Prompt-cache the static prefix, route cheap turns to the small model,
  batch what isn't interactive. Set cache TTL deliberately.
- **Interactivity fast?** H5P for widgets; generative LLM for open-ended scenarios.

---

## Source index
- [Co-designing LLM tutors (Learning Agency)](https://the-learning-agency.com/the-cutting-ed/article/co-designing-llm-tutors-for-student-success-five-key-takeaways-and-a-roadmap-for-developers/)
- [GuideEval / Socratic LLM eval (arXiv 2508.06583)](https://arxiv.org/html/2508.06583v1)
- [Aligning LLMs with pedagogy via RL (arXiv 2505.15607)](https://arxiv.org/pdf/2505.15607)
- [NotebookLM RAG for active learning (arXiv 2504.09720)](https://arxiv.org/html/2504.09720v2)
- [LLM guardrails for RAG (IBM)](https://developer.ibm.com/tutorials/awb-how-to-implement-llm-guardrails-for-rag-applications/)
- [KT + LLM systematic review (arXiv 2412.09248)](https://arxiv.org/pdf/2412.09248)
- [Dialogue-KT (GitHub, LAK 2025)](https://github.com/umass-ml4ed/dialogue-kt)
- [Deep KT for platforms (EDM 2025)](https://educationaldatamining.org/EDM2025/proceedings/2025.EDM.industry-papers.46/index.html)
- [CAT misconception detection (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC7711247/)
- [Verifiable QG + grading (Springer)](https://link.springer.com/chapter/10.1007/978-3-031-99264-3_22)
- [LLM grader, guideline optimization (EDM 2025)](https://educationaldatamining.org/EDM2025/proceedings/2025.EDM.long-papers.80/index.html)
- [Automated grading human-in-the-loop (arXiv 2504.05239)](https://arxiv.org/pdf/2504.05239)
- [LearnLM in Gemini 2.5 (Google)](https://blog.google/outreach-initiatives/education/google-gemini-learnlm-update/)
- [LearnLM report (arXiv 2412.16429)](https://arxiv.org/pdf/2412.16429)
- [Khanmigo growth (AI for Cause)](https://aiforcause.org/stories/khanmigo-ai-tutor)
- [FSRS vs SM-2 (Neurako)](https://www.neurako.com/blog/fsrs-vs-sm2-spaced-repetition-algorithms-compared)
- [Prompt caching (Claude docs)](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- [Claude API pricing 2026 (CloudZero)](https://www.cloudzero.com/blog/claude-api-pricing/)
- [Bedrock KB cost / Aurora pgvector migration](https://ercanermis.com/cutting-amazon-bedrock-knowledge-base-costs-by-90-migrating-from-opensearch-serverless-to-aurora-serverless-v2-with-pgvector/)
- [AI roleplay platforms 2026 (Mindtickle)](https://www.mindtickle.com/blog/best-ai-roleplay-simulator-tools-for-contact-center-training/)
