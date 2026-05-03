# Flight School Roadmap

A staged plan for evolving PPCPilot.org's Flight School from a working
practice-test platform into a comprehensive PPC training program.

**Status as of 2026-05-03**: Working practice test (300 questions), study
mode with spaced repetition, exam mode, handbook browser with deep-linking,
live eCFR integration for regulation citations.

**Audience**: powered-parachute student pilots, sport-pilot applicants, and
current PPC pilots maintaining proficiency.

**Guiding principles**:
- *Evidence-based learning*: testing effect, spaced repetition, immediate
  feedback, elaborative encoding. Every feature should have a clear
  learning-science rationale.
- *Handbook + regs as twin sources*: handbook teaches the *why*; eCFR
  proxy provides the authoritative *what*. Never duplicate regulatory text.
- *Real over abstract*: prefer interactive tools the user reaches for in
  actual flight planning over passive reading or trivia.
- *Ship in vertical slices*: each phase ends with something usable on its
  own, even if the next phase isn't started.

---

## Current state (baseline)

**Backend** (`ppc-pilot-web/backend/flightschool/`):
- 300 question bank across 10 ACS subject areas
- SM-2-lite spaced-repetition scheduler
- Per-user `ExamSession`, `ExamSessionQuestion`, `QuestionState`
- Handbook content from `ppc-knowledgebase` (11 modules / ~44k words)
- eCFR live proxy at `/api/v1/flightschool/regs/<citation>/`, 7-day cache

**Frontend** (`ppc-pilot-web/frontend/src/pages/flightschool/`):
- Dashboard, Study, Practice Exam, Exam History, Handbook
- HandbookSectionModal — opens chapter inline, scrolls to section anchor
- RegulationModal — fetches live eCFR text
- StudyNextPanel — aggregates missed-question chapters

**Coverage gaps**:
- Handbook missing: Part 61 sport-pilot certification, most of Part 91
  general operating rules. 47 of 300 questions show eCFR-only links.
- No interactive tools (chart reader, weather analyzer, performance calc).
- No instructor / multi-user features.

---

## Phase 1 — Close the content gaps

**Goal**: Every test-bank topic has at least one teaching-style handbook
chapter that explains it, not just a regulation citation.

**Two new modules** added to `ppc-knowledgebase/training_content/`:

1. `12_pilot_certification/01_sport_pilot_certification.md` (~6k words)
   - Sport pilot eligibility (age, language, medical alternatives)
   - Training requirements: ground, flight, solo, cross-country
   - Knowledge test, practical test, ACS overview
   - Privileges & limitations (altitude, speed, passengers, compensation,
     night, towing, business use)
   - Recency of experience (90-day passenger currency)
   - Flight review (24-month requirement, what counts)
   - Add-on endorsements: make/model, Class B/C/D airspace
   - Logbook requirements
   - Address-change reporting

2. `13_general_operating_rules/01_part_91_for_sport_pilots.md` (~4k words)
   - PIC responsibility & emergency authority (§ 91.3)
   - Preflight action (§ 91.103)
   - Right-of-way rules (§ 91.113) — includes diagrams
   - Speed limits (§ 91.117)
   - Minimum safe altitudes (§ 91.119)
   - Altimeter settings (§ 91.121)
   - Fuel requirements (§ 91.151)
   - VFR weather minimums (§ 91.155)
   - VFR cruising altitudes (§ 91.159)
   - Required documents — ARROW (§ 91.203, § 91.9)
   - Alcohol & medication (§ 91.17, § 61.53)
   - Careless or reckless operation (§ 91.13)

Each chapter:
- Cross-links to the eCFR section via `[§ 91.113](handbook-link)` style
- Includes "Common misconceptions" boxes
- Includes 3–5 worked scenarios per chapter
- Pedagogical: explains *why* the rule exists, *what behavior the FAA wants
  to prevent*, and *how a sport pilot in a PPC actually applies it*

**Wiring**:
- `import_knowledgebase` re-run to refresh the JSON snapshot
- Update `_section_anchors.py` for the 47 currently regulation-only questions
- Update question references (`pilot_qualifications.py`, `regulations.py`)
  so the handbook modal opens the new chapters

**Done when**: every active question has a `reference_chapter` set, deep-
linked to a relevant section heading. eCFR remains the secondary
authoritative source via the existing modal.

**Effort**: ~2 days of writing + half a day of wiring.

---

## Phase 2 — Interactive sectional-chart trainer

**Goal**: Make airspace and chart reading concrete. The Airspace ACS area
is 35 questions / ~12% of the bank, and it's where new pilots struggle most.

**Build**:
- New page `/flight-school/charts/airspace-trainer`
- Embed FAA VFR raster charts (free public-domain GeoTIFF tiles served
  via a tile server like FAA's own or a mirror)
- React-Leaflet base layer with the chart tiles
- Annotation overlay: clickable rings/lines/symbols that pop a small card
  explaining the feature
- Curriculum: a series of guided "missions" — each highlights one feature
  type (Class B/C/D boundaries, special-use airspace, MEFs, obstacle
  symbols, sectionals legend lookups)
- Quiz mode: zoom to a random region, ask "what class is this airspace?
  what's the floor / ceiling? what frequency for entry?"

**Why it matters**: Visual / spatial information about airspace doesn't
transfer well from text. Pilots who can read a chart fluently make better
go/no-go decisions; those who can't quietly avoid unfamiliar airports.

**Dependencies**:
- VFR chart tiles (FAA publishes; need to host or proxy)
- Existing React-Leaflet stack already in `CommunityMapPage.tsx`

**Done when**: a student can complete a 10-mission curriculum and a
20-question chart-quiz, both with feedback and SR scheduling.

**Effort**: ~5 days. The annotation data is the long pole — needs
geographic coordinates of features per chart region.

---

## Phase 3 — Weather decision practice

**Goal**: Train ADM (aeronautical decision making), not just weather
trivia. The user already has weather infrastructure on the dashboard
(Golden Hour Forecast, PPC Flight Assessment).

**Build**:
- New page `/flight-school/scenarios/weather`
- Pull live METAR/TAF/AIRMET for a user-chosen airport via existing
  weather APIs
- Generate a flight-scenario prompt: "It's 1430Z, you want to fly a
  60-NM XC from KXYZ to KABC, here are the current conditions. Go,
  no-go, or wait?"
- After the user commits, reveal an analysis: which cues were green,
  yellow, red; what a CFI would say; what hazards a less-experienced
  pilot might miss
- Reuse the SR scheduler — scenarios cycle through the user's queue

**Variants**:
- Real-time (live data, current conditions)
- Curated library (~30 hand-built scenarios with known-correct ADM)
- "What if" mode: user adjusts time of day or wind speed and sees how the
  go/no-go calculus shifts

**Why it matters**: Weather is the #1 factor in fatal GA accidents. Trivia
about METAR codes is necessary but not sufficient. Practice with real
ambiguous data builds the judgment that saves lives.

**Dependencies**:
- Existing weather API integration (`WeatherPage`, `PPCFlightAssessmentPage`)
- New `Scenario` Django model + curated content

**Done when**: 30 curated scenarios live, real-time mode works for any
US airport, and a per-user "ADM accuracy" stat is on the dashboard.

**Effort**: ~5 days, with ~2 of those for content (the 30 scenarios).

---

## Phase 4 — Performance calculator

**Goal**: Turn abstract performance questions into a tool the user
actually opens before every flight.

**Build**:
- New page `/flight-school/calculator`
- Inputs: airport elevation, OAT, altimeter setting, humidity (optional)
- Outputs: pressure altitude, density altitude, takeoff distance estimate,
  rate of climb estimate, service ceiling check
- Weight & balance section: pulls the user's saved aircraft from
  `ppc-pilot-web` Equipment models, lets them load fuel + pilot + passenger
- POH-curve entry: the user enters their aircraft's takeoff curve once
  (or imports a default for a Pegasus/Powrachute/etc.) and the calculator
  uses it for all subsequent computations
- Save snapshot to flight log: "today's calculations" stored with each
  flight entry

**Why it matters**: Density-altitude awareness is the single most missed
self-knowledge among GA pilots. A tool that sits in the pre-flight loop
internalizes the math better than any quiz.

**Dependencies**:
- Existing `Equipment` and `Flight` Django models in `core/`
- A small library of POH performance curves (manual entry to start)

**Done when**: the user can enter today's airport + weather, get density
altitude and takeoff distance, save that to a flight, and the practice
test's PERF questions deep-link to the relevant calculator inputs.

**Effort**: ~4 days.

---

## Phase 5 — Spaced repetition for handbook reading

**Goal**: Bridge "I read the chapter" and "I remember the chapter."

**Build**:
- Each handbook chapter section (H2/H3) becomes a reviewable "card"
- New `HandbookCard` model: `chapter_id`, `section_anchor`, `prompt`,
  `expected_recall_points` (3–5 bullets the reader should be able to
  generate from memory)
- Reuse existing SM-2-lite scheduler for the new cards alongside
  practice-test questions
- Study session interleaves: "Recall the four hazardous attitudes from
  ADM Section 3" → user attempts → reveal — same loop as MCQ but with
  free-recall self-grading

**Why it matters**: Reading is passive; retrieval is active. Free-recall
is the highest-impact study technique per the cognitive-science literature.

**Dependencies**:
- Phase 1 (more chapters to extract cards from)
- Existing scheduler (no changes)

**Done when**: every handbook chapter has 5–15 recall cards, integrated
into the study queue, surfaced on the dashboard.

**Effort**: ~3 days for the system + ongoing card authoring.

---

## Phase 6 — Cross-test mastery view

**Goal**: The user knows how ready they are for the real FAA test, and
exactly which subject is dragging them down.

**Build**:
- Per-subject mastery score (already partially computed in
  `SubjectViewSet.list`)
- Rolling exam-pass-confidence model: based on the last N exam-mode
  attempts and study-mode SR state, estimate probability of passing the
  real FAA test
- Dashboard widget: "Pass-readiness: 78% · weakest area: Right-of-way
  rules · suggested next study session: 20 mixed cards focused on
  AIRSPACE"
- Per-subject drill-down view: which questions, which sections, which
  ACS task codes are weak

**Why it matters**: Self-regulated learning depends on accurate
self-assessment. Surfacing weak areas converts vague worry into
targeted action.

**Dependencies**:
- Phases 1 and 5 for fuller signal density

**Effort**: ~3 days.

---

## Phase 7 — Mock practical-test prep

**Goal**: Practice the *talking* part of the checkride, not just the
written.

**Build**:
- Scenario walkthroughs aligned to the FAA Sport Pilot — PPC ACS
- Each scenario has a setup ("you're planning to fly to a Class D
  airport for fuel, here's the chart"), a series of CFI-style questions,
  and a rubric
- AI-graded feedback: integrate Claude API to play the DPE role, ask
  follow-up questions, identify gaps in the user's reasoning
- Recorded transcripts saved to the user's history for review

**Why it matters**: The actual checkride is conversational. Many students
pass the written and bomb the oral because they've never practiced it.

**Dependencies**:
- Anthropic API integration (new) — backend has the JWT/auth pattern
  to layer on
- ACS task definitions (Subject + acs_task_code already in the data
  model, just needs scenario content per task)

**Done when**: 20 scenarios available across the 10 subject areas,
AI-graded with consistent rubric, transcripts in history.

**Effort**: ~5 days infra + ~5 days content.

---

## Phase 8 — Instructor & community features

**Goal**: Move from solo-study to a flight-school tool.

**Build**:
- CFI account type with a roster view
- Assignment workflow: instructor picks topics or specific scenarios,
  assigns due dates, sees student progress
- Comment threads on missed questions — instructor can leave context
  ("see what I told you about right-of-way last week?")
- Aggregate flight-school dashboard: how all my students are doing
- Optional: shared progress badges, leaderboards within a school

**Why it matters**: Sport-pilot training is small-scale (one CFI / a few
students). Tools that make that workflow easier get adopted because
nothing else in the market addresses it well. Aligns with PPCPilot.org's
existing Flight Schools community directory.

**Dependencies**:
- Existing `community` Django app's flight-school directory
- New roles in the auth system (or a `Roster` model layered on top)

**Effort**: ~10 days (feature-rich phase; could ship a thinner v1 in 4).

---

## Cross-cutting concerns

**Telemetry**: instrument every learning interaction. Per-question time-
to-answer, confidence calibration, abandonment rate. Build a small
analytics dashboard in the Django admin so we can see what's working.

**Content quality loop**: when a question has high error rate (>60%
wrong) AND low confidence-correct calibration, flag it for human review.
The bank should be self-improving.

**Mobile parity**: most students study on phones. Audit each new feature
for mobile usability before declaring it done. (Existing pages are
responsive; new ones should be too.)

**Accessibility**: keyboard navigation for the test interface, screen-
reader-friendly labels on all controls.

**Deploy reliability**: the deploy.sh self-modification gotcha (script
gets `git pull`-updated mid-execution) burns time. Move to a two-step
deploy: `pull-and-prepare.sh` (idempotent) + `apply.sh` (executed in a
fresh shell).

---

## Suggested order

If we ship in this order, the platform stays useful at every step:

1. **Phase 1** (handbook gaps) — closes the most embarrassing gap fast
2. **Phase 4** (performance calculator) — visible utility, modest scope
3. **Phase 3** (weather decision practice) — high learning ROI, leverages
   existing weather infrastructure
4. **Phase 2** (sectional chart trainer) — bigger build, biggest visual
   impact
5. **Phase 5** (handbook SR cards) — needs Phase 1 content to be useful
6. **Phase 6** (mastery view) — needs phases 1, 3, 5 for signal density
7. **Phase 7** (mock practical) — separable, can interleave anywhere
   after Phase 1
8. **Phase 8** (instructor features) — last; valuable but only after the
   solo-student experience is excellent

Phases are independently shippable; the order above optimizes for
learning-ROI per week of build time.
