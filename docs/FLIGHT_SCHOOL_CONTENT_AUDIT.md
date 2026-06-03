# Flight School Content Audit

**Purpose:** Assess whether the PPCPilot.org flight-school content covers everything a powered-parachute (PPC) pilot needs — new student through current pilot — measured against authoritative FAA sources. Identify gaps and recommend a module/topic structure and learning order.

**Scope of review (read-only):**
- Knowledgebase markdown: `ppc-flight-school/ppc-knowledgebase/training_content/` (13 module folders + `resources/`).
- Web flight-school seed: `ppc-pilot-web/backend/flightschool/seed/knowledgebase.json` (13 modules) and `seed/questions/*` (10 ACS subjects, ~300 questions).
- Web data model: `ppc-pilot-web/backend/flightschool/models.py` (Module → Chapter, Subject → Question/Choice, ExamSession, SM-2 spaced repetition).

**Date:** 2026-06-03. **Author:** kb-audit (research only; no content changed).

---

## 1. Inventory: What Exists Today

### 1a. Knowledgebase (markdown) — 13 modules, ~53,000 words

| # | Module folder | Chapter | Words | Authoritative analog |
|---|---|---|---|---|
| 01 | `01_foundations` | Principles of Flight for PPCs | 2,828 | PHAK Ch 4–5; PPC HB Ch 1 |
| 02 | `02_regulations` | Airspace Operations for PPC Pilots | 5,470 | PHAK Ch 15; PPC HB Ch 7 |
| 03 | `03_weather` | Weather and Meteorology | 3,789 | PHAK Ch 12–13 |
| 04 | `04_aircraft_systems` | PPC Aircraft Systems (cart→canopy) | 4,234 | PPC HB Ch 2–3; PHAK Ch 7 |
| 05 | `05_flight_operations` | Flight Operations (preflight→postflight) | 4,168 | PPC HB Ch 4–6, 10–11 |
| 06 | `06_emergency_procedures` | Emergency Procedures | 2,821 | PPC HB Ch 12 |
| 07 | `07_practical_skills` | Quick Reference Checklists | 2,299 | (derived) |
| 08 | `08_advanced_topics` | Aeronautical Decision Making & Risk | 6,282 | PHAK Ch 2 |
| 09 | `09_maintenance` | Maintenance & Inspection Guide | 5,806 | PPC HB Ch 2; mfr/Rotax |
| 10 | `10_cross_country` | Cross-Country Operations | 4,629 | PHAK Ch 16; PPC HB Ch 8–9 |
| 12 | `12_pilot_certification` | Sport Pilot Certification (PPC) | 4,351 | 14 CFR Part 61 Subpart J |
| 13 | `13_general_operating_rules` | General Operating Rules (Part 91) | 4,344 | 14 CFR Part 91 |
| — | `resources/` | 14 CFR Part 103 Summary | 2,065 | 14 CFR Part 103 |

Notes:
- There is **no module 11** (the numbering skips it). Modules 12 and 13 were added later (May 2026) and are the newest, most current material.
- The KB README still advertises "~119,000 words / 11 modules" (v2.0, dated Nov 2025). **The actual on-disk content is ~53k words across 13 modules** — the README's word count is roughly 2× the reality and is stale. (Not a content gap per se, but a documentation accuracy issue.)
- Each module is a **single monolithic chapter** (one long `.md` per folder). There is no sub-chapter / lesson granularity within a topic.
- A `Powrachute-Pegasus-POH.pdf` (the owner's actual aircraft) sits in the KB root but is **not ingested** into the structured content.

### 1b. Web seed — mirrors the KB, plus a real question bank

- `knowledgebase.json` (version 1) contains the **same 13 modules, 1 chapter each** — a 1:1 import of the markdown above. So the web "handbook" content and the KB are the same corpus.
- `seed/questions/` is a **separate, higher-value asset**: ~300 multiple-choice questions across **10 ACS-aligned subjects**, each question carrying an ACS task code (e.g. `IV.A.K1`), an explanation, distractor explanations, a deep-link to the reference chapter/section, and (where relevant) a live eCFR citation. Counts:

  | Subject (ACS area) | Questions | 60-Q exam target |
  |---|---|---|
  | Pilot Qualifications & Certification (I) | 20 | 4 |
  | Federal Aviation Regulations (II) | 40 | 8 |
  | Aeromedical Factors & ADM (III) | 25 | 5 |
  | Aerodynamics & PPC Principles (IV) | 30 | 6 |
  | Aircraft Systems (V) | 30 | 6 |
  | Weight & Balance and Performance (VI) | 25 | 5 |
  | Weather Theory & Services (VII) | 40 | 8 |
  | Airspace, Charts & Navigation (VIII) | 35 | 7 |
  | Comms, Airport Ops & Procedures (IX) | 25 | 5 |
  | Cross-Country & Emergency Procedures (X) | 30 | 6 |

- **The question bank's subject taxonomy is more complete than the handbook's module taxonomy.** The questions already have dedicated **Aeromedical** and **Weight & Balance/Performance** subjects, but the handbook has **no chapter** dedicated to either — those topics are only scattered as sub-sections inside other modules. This is the central structural gap (see §2).

### Overall verdict on what exists
The existing content is **genuinely good and authentically PPC-specific** — not warmed-over fixed-wing GA. Standout coverage: weather (incl. wind gradient, thermals, go/no-go), aircraft systems (Rotax 582, gearbox, ram-air wing, trim), ADM, maintenance, and the new MOSAIC-era certification module. The question bank is well-built and ACS-coded. The gaps below are mostly about **missing dedicated coverage of a few ACS areas, depth on PPC-specific emergencies, currency drift, and structural granularity** — not a broken foundation.

---

## 2. Gap Analysis (vs. authoritative sources)

Sources used as the yardstick: **FAA-H-8083-29 Powered Parachute Flying Handbook** (incl. the **Oct 2025 MOSAIC addendum**), **PHAK FAA-H-8083-25C** (incl. its MOSAIC addendum), the **Sport Pilot ACS / FAA-S-8081-31A** areas of operation, **14 CFR Part 103**, and **14 CFR Parts 61/91** (post-MOSAIC). Full citations in §4.

### Tier 1 — Safety-critical gaps (fix first)

1. **Ballistic Reserve / emergency parachute (BRS) — MISSING.** The emergency module covers engine-out, collapse, line break, spin, fire, etc., but never mentions a ballistic/hand-deploy reserve: when it is the right answer, minimum-altitude considerations, deployment technique, post-deployment body position, or repack/maintenance. Many two-place PPC carts carry a BRS; its absence is the single biggest emergency-content hole. *(PPC HB Ch 12.)*

2. **PPC-specific canopy malfunctions under-covered.** No treatment of **line-overs / cravats**, **asymmetric (side) collapse with directional recovery**, **canopy surge / dive on power application**, **pilot-induced oscillation (PIO) / parachutal stall / deep-stall**, or **riser-twist**. The existing "line break → spiral in" guidance is thin and arguably unsafe without qualification. These are the failure modes unique to a ram-air wing and deserve their own treatment. *(PPC HB Ch 1, 12.)*

3. **Water landing / ditching — MISSING.** Forced-landing section lists "water: risky" in one line. No ditching procedure, no flotation/PFD guidance, no "wing-over-pilot drowning" hazard discussion despite it being a known PPC fatality mode.

4. **Currency drift on night flying (MOSAIC).** The certification module (written against the new § 61.315 numbering) still flatly lists **"operate at night (sunset to sunrise)"** as a sport-pilot *prohibition*. Under the **MOSAIC final rule (effective Oct 22, 2025), new § 61.329** grants sport pilots a **night privilege** with night training, an instructor endorsement, and a **valid FAA medical** (the driver's-license pathway does not cover night). This statement is now incorrect and needs correction. *(See §4 — MOSAIC, InFO 26006.)*

### Tier 2 — Missing/thin ACS areas (the question bank tests these; the handbook doesn't teach them)

5. **Aeromedical & Human Factors — no dedicated module.** ADM (module 08) is excellent, but the *physiological* aeromedical content — hypoxia, hyperventilation, **CO poisoning (very real for a 2-stroke seated behind the engine)**, spatial disorientation and the visual/vestibular **illusions**, dehydration/heat stress, cold/hypothermia, **noise & hearing protection**, vision/scanning, alcohol/drugs/medication, fitness to fly — is not consolidated anywhere. The seed has 25 `AEROMED` questions with nowhere to "read about it." *(PHAK Ch 17; ACS Area III.)*

6. **Weight & Balance — no dedicated module.** W&B is mentioned in 10 files but never taught as a procedure: no datum/arm/moment worked example, **no PPC hang-point/CG concept**, no max-gross enforcement, no "how to weigh the cart and compute loaded W&B" walkthrough. Seed has 25 `PERFORMANCE` (W&B + performance) questions. *(PHAK Ch 10; ACS Area VI.)*

7. **Performance & density altitude — scattered, not consolidated.** Density altitude is well-explained inside Principles of Flight, but takeoff/landing distance estimation, the effect of wing loading on performance, and a "performance planning" workflow are not pulled together. *(PHAK Ch 11; PPC HB Ch 6.)*

8. **Airport operations & communications — thin and folded into Airspace.** Traffic-pattern entry/legs, **non-towered CTAF self-announce phraseology**, runway/taxiway markings, airport signs and lighting, light-gun signals, and **runway-incursion avoidance** are only lightly touched. Seed has 25 `AIRPORTS` questions. *(PHAK Ch 14; ACS Area IX.)*

9. **Ground reference & flight maneuvers — MISSING as a teachable unit.** The ACS/PTS require turns around a point, S-turns, rectangular course, and the basic maneuver set (slow flight, stall recognition, ground handling/kiting as a maneuver). Flight Operations covers normal takeoff/landing prose but not the **ACS maneuver list** a checkride candidate must perform. *(PPC HB Ch 5, 8; ACS Areas IV–VIII.)*

### Tier 3 — Depth, currency, and structure

10. **MOSAIC LSA reclassification not explained.** Under MOSAIC, PPCs fold into the broader light-sport framework (airworthiness changes effective **July 24, 2026**). The cert module uses MOSAIC numbering but doesn't explain *what MOSAIC is*, the Part 103 ultralight vs. light-sport PPC distinction in the new world, or the timeline — useful for a "current pilot" audience deciding which path applies to them.

11. **Part 103 vs. Sport-Pilot/LSA decision framing is implicit.** The content has all three pieces (Part 103 summary, sport-pilot cert, Part 91) but never gives the reader a single clear **"which regime am I operating under, and what does that mean for me"** decision tree (single-place ≤254 lb / 5 gal / daylight ultralight vs. two-place light-sport PPC requiring a certificate).

12. **Radio/communications procedures** beyond the emergency Mayday template are minimal — phonetic alphabet, frequency types (CTAF/UNICOM/FSS), position reporting, and requesting a Class D transition (which a sport pilot can do with the endorsement) are not taught.

13. **Towing, formation, and aerobatics prohibitions** are mentioned in passing but not as a "what you may not do and why" safety unit; PPC-relevant special ops (e.g., flour-drop / spot-landing fun-fly etiquette, banner concerns) are absent.

14. **Granularity / monolithic chapters.** Each module is one long markdown file. For an adaptive, spaced-repetition learning product, content benefits from **lesson-level sections** that can be individually tracked, recommended, and linked to specific questions. The `reference_section_anchor` field in the model already anticipates this but the content isn't chunked to exploit it.

15. **No glossary, acronym list, or figure/diagram assets.** Aerodynamics and systems especially would benefit from diagrams (the markdown is all prose). No consolidated glossary for the heavy terminology load.

### Things that are NOT gaps (already solid)
- Weather theory & services, including wind gradient/shear and go/no-go — strong, and complemented by the app's existing winds-aloft go/no-go assessment.
- Engine/systems (2-stroke + 4-stroke, Rotax 582 detail, gearbox, prop, ram-air wing, trim) — thorough and PPC-specific.
- ADM/risk (PAVE, IMSAFE, 3P, DECIDE, 5Ps, hazardous attitudes) — comprehensive.
- Maintenance/inspection and Part 103 regulatory summary — good.
- Airspace classes and VFR cloud-clearance/visibility — well covered.
- The ACS-coded question bank with explanations and reference deep-links — a real asset.

---

## 3. Recommended Module / Topic Structure (new → current pilot)

Goal: align the **handbook taxonomy with the ACS subject taxonomy** (so every tested area has teachable content), close the Tier-1/2 gaps, and order topics for a learner progressing from "never flown" to "current, proficient." Modules are ordered for learning; ACS area shown for traceability. Bold = **new/expanded vs. today**.

**Phase A — Orientation & Regulations (know before you go)**
1. **Welcome & How PPC Fits In** — what a PPC is, Part 103 ultralight vs. light-sport PPC **decision tree**, **what MOSAIC changed and its timeline**, the training journey. *(new framing; ACS I/II)*
2. Federal Regulations — Part 103 (single-place), Part 61 Subpart J sport pilot, Part 91 general ops. *(merge existing 12 + 13 + Part 103 resource; **correct the night-flight item**; ACS I/II)*
3. Pilot Certification & Currency — sport-pilot path, medical/driver's-license rule, flight review, 90-day currency, **night & controlled-airspace endorsements**. *(existing 12, currency-corrected; ACS I)*

**Phase B — Aircraft & Aerodynamics (understand the machine)**
4. Principles of Flight — four forces, ram-air wing, AOA/stall, pendulum stability, ground effect, density altitude. *(existing 01; ACS IV)*
5. Aircraft Systems — cart, 2-/4-stroke engines, fuel/ignition/cooling, gearbox, prop, electrical, instruments, **wing/lines/risers/trim**. *(existing 04; ACS V)*
6. **Weight & Balance** — datum/arm/moment, **PPC hang-point & CG**, max gross, a worked loaded-W&B example for a two-place cart. *(NEW dedicated module; ACS VI)*
7. **Performance** — density altitude planning, takeoff/landing distance, wing-loading effects, performance go/no-go. *(NEW, consolidates scattered content; ACS VI)*

**Phase C — Environment (decide whether to fly)**
8. Weather Theory — atmosphere, wind/gradient/shear, turbulence, fronts, thunderstorms, fog, stability. *(existing 03; ACS VII)*
9. Aviation Weather Services — METAR/TAF, **winds aloft (tie to the app's go/no-go tool)**, AIRMET/SIGMET, radar, briefings. *(split out of 03; ACS VII)*
10. **Aeromedical & Human Factors** — hypoxia, hyperventilation, **CO poisoning**, spatial disorientation & **illusions**, vision/scan, **noise/hearing**, dehydration/heat/cold, alcohol/drugs/meds, fitness/IMSAFE. *(NEW dedicated module; ACS III)*
11. Aeronautical Decision Making & Risk — PAVE, 3P, DECIDE, 5Ps, hazardous attitudes, personal minimums. *(existing 08; ACS III)*

**Phase D — Airspace, Airports & Navigation (where you fly)**
12. Airspace — classes, SUA, TFRs, VFR minimums, chart reading. *(existing 02; ACS VIII)*
13. **Airport Operations & Communications** — pattern entry/legs, **CTAF self-announce phraseology**, markings/signs/lighting, light-gun signals, **runway-incursion avoidance**, requesting Class D transition. *(NEW/expanded; ACS IX)*
14. Navigation & Cross-Country — pilotage, dead reckoning, GPS, fuel planning, route/chart work, en-route procedures. *(existing 10; ACS VIII/X)*

**Phase E — Flying the PPC (procedures & maneuvers)**
15. Ground & Preflight Operations — preflight inspection, **wing layout/kiting as a skill**, engine start, run-up, taxi. *(from existing 05 + 09 preflight; ACS Preflight)*
16. **Flight Maneuvers (ACS task set)** — normal/crosswind takeoff & climb, cruise, turns, **ground-reference maneuvers (turns about a point, S-turns, rectangular course)**, slow flight & stall recognition, approaches, flare/landing, go-around. *(expanded from 05 to match ACS/PTS; ACS IV–VIII)*
17. **Night Operations** — **NEW**, post-MOSAIC: night physiology, lighting/equipment, night training/endorsement requirements, night go/no-go. *(NEW; ACS — newly relevant)*

**Phase F — When Things Go Wrong (emergencies)**
18. Emergency Procedures — engine failure (takeoff/cruise), **canopy malfunctions: line-over/cravat, asymmetric collapse, surge, PIO/deep-stall, riser twist**, **ballistic reserve/BRS deployment**, **water landing/ditching**, fire, structural, forced landing, survival, NTSB reporting. *(expand existing 06 to close Tier-1 gaps; ACS Emergency Ops)*

**Phase G — Ongoing (stay a current, proficient pilot)**
19. Maintenance & Inspection — Part 103 maintenance philosophy, preflight/periodic inspection, wing care, Rotax procedures, recordkeeping. *(existing 09; ACS V)*
20. **Staying Current & Proficient** — flight review, currency tracking, recurrent training, personal-minimums review, **transition/cross-training to other categories under MOSAIC**. *(NEW capstone for the "current pilot" audience)*
21. Quick-Reference Checklists & Personal Minimums. *(existing 07, kept as a reference appendix)*

**Cross-cutting additions:** a **glossary/acronym index**, **diagrams** for aerodynamics/systems/airspace, and **lesson-level chunking** (sub-sections) so each lesson maps to specific question codes for adaptive review. Ingest the **Pegasus POH** as aircraft-specific reference tied to the owner's airframe.

---

## 4. Source Citations

- **FAA-H-8083-29, Powered Parachute Flying Handbook** — https://www.faa.gov/sites/faa.gov/files/regulations_policies/handbooks_manuals/aviation/powered_parachute_handbook.pdf (chapters: aerodynamics; components/systems; powerplants; preflight & ground ops; basic flight maneuvers; takeoffs & departure climbs; airspace; ground reference maneuvers; airport operations; approaches & landings; night, abnormal & emergency procedures).
- **FAA-H-8083-29 MOSAIC Addendum (Oct 20, 2025)** — https://www.faa.gov/regulations_policies/handbooks_manuals/aviation/PPC_HB_Addendum_(MOSAIC).pdf
- **PHAK FAA-H-8083-25C** — https://www.faa.gov/regulations_policies/handbooks_manuals/aviation/phak (Ch 2 ADM, Ch 10 Weight & Balance, Ch 11 Performance, Ch 12–13 Weather, Ch 14 Airport Ops, Ch 15 Airspace, Ch 16 Navigation, Ch 17 Aeromedical).
- **PHAK MOSAIC Addendum** — https://www.faa.gov/regulations_policies/handbooks_manuals/aviation/PHAK_Addendum_(MOSAIC).pdf
- **Sport Pilot ACS / PTS (WSC & Powered Parachute), FAA-S-8081-31A** — https://www.faa.gov/training_testing/testing/acs/sport_wsc_pp_pts_31.pdf (Areas of Operation: Preflight Preparation; Preflight Procedures; Airport Ops; Takeoffs/Landings/Go-Arounds; Performance Maneuver; Ground Reference Maneuvers; Navigation; Slow Flight & Stalls; Emergency Operations; Postflight).
- **FAA ACS landing page** — https://www.faa.gov/training_testing/testing/acs
- **14 CFR Part 103 — Ultralight Vehicles** — https://www.ecfr.gov/current/title-14/chapter-I/subchapter-F/part-103 (single occupant; <254 lb empty excl. safety devices; ≤5 U.S. gal fuel; ≤55 kt CAS full power level; power-off stall ≤24 kt CAS; sunrise–sunset, twilight only with 3-SM anticollision light).
- **MOSAIC Final Rule (FAA)** — https://www.faa.gov/newsroom/MOSAIC_Final_Rule_Issuance.pdf (sport-pilot privilege changes effective **Oct 22, 2025**; new-aircraft airworthiness certification effective **July 24, 2026**).
- **MOSAIC sport-pilot night privilege** — new **14 CFR § 61.329** (3 hr night training incl. a ≥25 NM night XC and 10 full-stop takeoffs/landings; valid FAA medical required — not driver's-license pathway). Background: AOPA MOSAIC FAQ https://www.aopa.org/news-and-media/all-news/2025/august/14/mosaic-explained-faq ; Pilot Institute https://pilotinstitute.com/mosaic-rule/
- **FAA InFO 26006** — sport-pilot CFI endorsement limitations after MOSAIC — https://www.afm.aero/faa-issues-info-26006-clarifying-sport-pilot-cfi-endorsement-limitations-following-mosaic-rule-implementation
- **USPPA / USUA Part 103 references** — https://usppa.org/far-part-103-ultralight-vehicles/ , https://www.usua.org/Rules/faa103.htm

---

## 5. Top-Priority Recommendations (one-screen summary)

1. **Add a Ballistic Reserve (BRS) + PPC canopy-malfunction unit** to Emergency Procedures (Tier 1 safety gap).
2. **Correct the night-flight prohibition** in the certification module — MOSAIC § 61.329 now permits sport-pilot night ops with training/endorsement/medical; add a **Night Operations** module.
3. **Create dedicated Aeromedical/Human Factors and Weight & Balance modules** — both are full ACS areas with question banks but no teaching content (esp. **CO poisoning** and **spatial-disorientation illusions** for PPC).
4. **Build out Airport Ops/Communications and the ACS flight-maneuver/ground-reference set** so checkride-relevant tasks are actually taught.
5. **Add a Part 103 vs. light-sport-PPC decision tree and a "what MOSAIC means" explainer** for both new and current-pilot audiences.
6. **Chunk monolithic modules into lessons** mapped to question codes (the model already supports `reference_section_anchor`), add a **glossary + diagrams**, and **fix the stale KB README word count (53k, not 119k)**.
