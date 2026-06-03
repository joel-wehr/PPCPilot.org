# PPCPilot Flight School — Strategy & Build Plan

**Status:** Strategy synthesis (2026-06-03)
**Synthesizes:** `FLIGHT_SCHOOL_CONTENT_AUDIT.md` (content gaps), `FLIGHT_SCHOOL_PEDAGOGY.md` (how to teach), `FLIGHT_SCHOOL_AI_TOOLS.md` (AI tooling) + the cross-domain `AI_LEARNING_TOOLBOX.md`.
**Audience:** owner decision + a build roadmap.

## TL;DR / recommendations
1. **Build the AI "Ask-the-CFI" tutor — YES.** RAG grounded in *our* knowledgebase, every regulatory claim cited to eCFR, Socratic-by-default with a "just explain it" toggle, and a hard "defer to a CFI for sign-offs/judgment" guardrail. Cheap on our existing Claude/Bedrock stack (prompt caching + model routing). It's the single biggest "interactive, not just testing" upgrade.
2. **Make it mastery-based, not a flat question pool.** Sequence the 10 ACS subjects into units; gate "current" on a fresh retrieval check (~80–90%); the existing recommendation engine picks the next unit from profile + measured weakness.
3. **Adaptive assessment = explicit-model-first.** Upgrade the scheduler SM-2 → **FSRS** (drop-in `py-fsrs`, ~20–30% fewer reviews) + per-subject **BKT mastery**; use the LLM only for **misconception diagnosis** of wrong answers (the LLM is not a reliable mastery tracker). Both feed `recommendations.py`.
4. **Fix content accuracy + fill safety gaps first** (some content is now wrong post-MOSAIC).

## Where we are (good foundation)
- Knowledgebase: **13 modules (~53k words**; the README's "119k" is stale — fix it), mirrored into the web app as Module→Chapter, **+ ~300 ACS-task-coded questions** with explanations + deep-links across 10 subjects. PPC-specific and solid on weather, systems, ADM, maintenance.
- App already has: ExamSession, an SM-2-lite spaced scheduler, a profile-driven `recommendations.py` (from the rich-profile work), and the go/no-go thresholds in `FLIGHT_ASSESSMENT_METHODOLOGY.md`.
- The 8-phase `flight-school-roadmap.md` is already science-aligned — this plan mostly reframes + connects it.

## Three pillars

### 1. Content (accuracy → gaps → structure)
- **Accuracy (urgent):** the certification content says sport pilots can't fly at night — **wrong post-MOSAIC** (eff. 2026-10-22; §61.329 permits night with training/endorsement/medical). Audit for other MOSAIC-stale claims. Add a Part 103-vs-Light-Sport-PPC decision tree + a MOSAIC explainer.
- **Safety gaps:** add ballistic reserve/BRS, canopy collapse/line-over/cravat/asymmetric, PIO, water ditching.
- **Missing teaching content:** Aeromedical/Human-Factors (CO, spatial-disorientation illusions) and Weight & Balance have question banks but **no lessons** — write them.
- **Thin:** airport ops/comms (CTAF phraseology, markings, runway-incursion), ground-reference/maneuver set.
- **Restructure:** monolithic 1-chapter modules → the audit's **21-module, new→current, ACS-aligned** structure; add glossary + diagrams. Sources: FAA-H-8083-29 + MOSAIC addendum, PHAK 25C, Sport-Pilot ACS, Part 103, MOSAIC final rule, InFO 26006.

### 2. Pedagogy (not "test and test")
- **Mastery-gated learning paths** over the ACS units (Bloom 2-sigma).
- Each lesson = **learn → worked example → faded practice → retrieve** (free-recall cards, not passive reading).
- **Spaced + interleaved** auto-review with visible currency decay (green→amber); stop mass-practice of one subject.
- **Scenario-based ADM** as a first-class item tier across all units ("given these conditions: go / no-go / wait — why"), reusing the real flight-assessment thresholds — this is the FAA's own SBT/ACS model.
- **Feedback for self-regulation:** immediate, explanatory (why + handbook deep-link), deliberate-practice structured, **fading** as mastery rises; a per-subject pass-readiness + **confidence-calibration** dashboard (surface the "confidently wrong" quadrant).
- **Accessibility (WCAG 2.2 AA / UDL) is a hard gate.**

### 3. AI tooling
- **"Ask the CFI" RAG tutor** (always-available panel): pgvector over our KB (~90% cheaper than OpenSearch), Claude, citations to our content + eCFR, Socratic default + explain toggle, CFI-deferral guardrail. (Competitor validation: Sporty's shipped ChatCFI/ChatFAR/ChatDPE in 2026.)
- **Adaptive engine:** FSRS scheduler + BKT per-subject mastery + distractor-level/LLM misconception diagnosis → turns wrong answers into "review section X" → feeds `recommendations.py` so "next 15 minutes" is *earned by performance*, not just profile.
- **AI question-gen + free-response grading + targeted feedback**, generated from the KB **with human review** before items go live.
- Use-vs-build + cost guardrails detailed in `FLIGHT_SCHOOL_AI_TOOLS.md`.

## Phased roadmap
- **Phase 0 (quick win):** swap SM-2 → FSRS (`py-fsrs`), small/low-risk.
- **Phase 1:** MVP **RAG "Ask the CFI"** tutor panel (pgvector + Claude + citations). Biggest interactivity win for least code.
- **Phase 2:** mastery-gating + currency decay over the 10 ACS units; wire BKT into `recommendations.py`.
- **Phase 3:** scenario-based ADM item tier (reuse go/no-go thresholds) + explanatory/fading feedback + confidence dashboard.
- **Phase 4:** content accuracy fixes + gap-fill (MOSAIC, safety, aeromedical/W&B lessons, 21-module restructure) — can run in parallel with a content agent.
- **Phase 5:** LLM misconception diagnosis + AI question-gen (reviewed) + richer interactive sims.

## Cross-domain
`AI_LEARNING_TOOLBOX.md` captures the general-purpose AI/edtech patterns + platforms (reusable for teaching any field) per the owner's ask — maintain it as future projects surface more.

## Open decisions for the owner
- Greenlight the **AI tutor** build (Phase 0+1) now, or wait for "part two"?
- Where the platform lives: keep evolving the **existing Django flightschool app** (recommended — it's an evolution, not a rebuild) vs. the planned separate Next.js/FastAPI `ppc-training-platform`.
- Content fixes: dispatch a content agent to fix the MOSAIC accuracy errors + write the missing lessons in parallel?

## References
- Detailed docs: `FLIGHT_SCHOOL_CONTENT_AUDIT.md`, `FLIGHT_SCHOOL_PEDAGOGY.md`, `FLIGHT_SCHOOL_AI_TOOLS.md`, `AI_LEARNING_TOOLBOX.md`.
- Supersedes the older Moodle recommendation in `ppc-knowledgebase/INTERACTIVE_COURSE_PLATFORM_ANALYSIS.md`.
