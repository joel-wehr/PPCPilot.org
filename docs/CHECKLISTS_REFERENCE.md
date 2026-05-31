# Checklists Feature — Authoritative Reference

**Feature:** `checklists` Django app + React checklist editor / run / log UI
**Audience:** build agents and reviewers working on the PPCPilot checklists feature
**Primary source:** FAA-H-8083-29, *Powered Parachute Flying Handbook*, **Chapter 5 — Preflight and Ground Operations** (and Ch. 6–7 for takeoff/landing flow). Secondary: manufacturer POH conventions (Rotax/Six Chuter/Powrachute/Pegasus-style operating handbooks) and FAA-S-8081-32 Powered Parachute PTS for the *task elements* an examiner expects.

> The FAA is explicit that a PPC is flown to a **manufacturer-provided printed checklist for the specific make/model**, and that the handbook content is a *general guideline applicable to all powered parachutes*. Our app therefore ships sensible defaults but is built to be **fully customizable** so a pilot can mirror their own POH.

## Sources

- FAA-H-8083-29 Powered Parachute Flying Handbook (full PDF): https://www.faa.gov/sites/faa.gov/files/regulations_policies/handbooks_manuals/aviation/powered_parachute_handbook.pdf
- FAA-H-8083-29 Ch. 5 "Preflight and Ground Operations" (text mirror used for extraction): http://avstop.com/ac/power_parachute/chapter5_6.html (and pages 5_7 … 5_17)
- FAA-H-8083-29 MOSAIC Addendum (Oct 20, 2025): https://www.faa.gov/regulations_policies/handbooks_manuals/aviation/PPC_HB_Addendum_(MOSAIC).pdf
- FAA-S-8081-32A Private Pilot PTS for Weight-Shift Control / Powered Parachute: https://www.faa.gov/training_testing/testing/acs/private_wsc_pp_pts_32.pdf
- FAA Aviation Handbooks & Manuals index: https://www.faa.gov/regulations_policies/handbooks_manuals/aviation

## Why the standard checklist set is what it is

Chapter 5 defines the PPC ground flow. The **walk-around covers five main tasks**, quoted verbatim:

> "A powered parachute walk around will cover five main tasks: 1. Cart inspection 2. Powerplant inspection 3. Equipment check 4. Engine warm-up and check 5. Wing and suspension line inspection."

These map cleanly onto the standard PPC checklist set the community (and our mobile apps) already use. Below, each default template is grounded in a specific handbook section.

### 1 & 2. Pre-Flight (Detailed) and Pre-Flight (Quick)
Source: Ch. 5 "Visual Inspection," "Cart Inspection," "Fuel and Oil," "Powerplant Inspection," "Equipment check."

Handbook basis (quotes):
- *Visual inspection:* "The purpose of the routine preflight inspection is twofold: to determine the powered parachute is legally airworthy, and that it is in condition for safe flight … performed in accordance with a printed checklist provided by the powered parachute manufacturer."
- *Unloading from trailer:* "look for any damage that may have occurred during transit … tires with low air pressure, structural distortion, wear points, cart damage, and dripping fuel or oil leaks. All tie-downs, control locks, and chocks should be removed."
- *Cart:* "Check the front nosewheel for proper play, tire inflation, and secure axle bolt … inspect the cable lock … check seats, seat rails, and seat belt attachment points … battery and ignition switches need to be in the OFF position at the beginning … manipulate the engine throttle control … check that all cables are free of kinks, frays, abrasions or broken strands … Inspect the rear wheel and axle assembly."
- *Fuel & oil:* "confirm the fuel quantity indicated on the fuel gauge(s) by visually inspecting the level … Ensure the fuel caps have been securely replaced … the vents are free and open … check the fuel filter for contaminates … Check the oil reservoir to ensure the proper oil is used … the oil reservoir on a two-stroke must be checked for adequate venting."
- *Powerplant:* "Inspect the propeller for any signs of … cracking … nicks in the leading edge … Carburetor(s) must be checked to make sure they are secure; check the air filter … Check the rubber manifolds for cracks and check spark plugs … Check gear reduction boxes for leaking seals."

Two templates because the handbook treats the detailed walk-around as the airworthiness inspection, while pilots in practice carry an abbreviated front/left/center/right flow for repeat flights the same day.

### 3. Engine Start
Source: Ch. 5 "Engine Starting."
- "Do not start the engine with the back of the cart … pointed toward an open hangar door, parked automobiles, or a group of bystanders."
- "First look around, and then shout CLEAR PROP. Wait for a response."
- "When activating the starter, keep one hand on the throttle. The other hand should be on the ignition … A low RPM setting is recommended immediately following engine start … check the oil pressure … If it does not rise to the manufacturer's specified value … shut down immediately."
- "avoid continuous starter operation for periods longer than 30 seconds without a cool down period."

### 4. Warm-Up / Run-Up
Source: Ch. 5 "Engine Warm-Up." The handbook gives an explicit ordered step list, reproduced as our default items:

> "Generally, the engine start-up will follow these steps: Walk-around is complete. Safety check to include: front wheels properly braced, engine and propeller area clear of loose and foreign objects, area behind the cart is clear of debris, wing lines are away from the propeller. Prime the fuel system (as equipped). Activate strobe light if switch is independent of magneto switch. Shout CLEAR PROP and wait for CLEAR response from bystanders. Turn magnetos on. Engine gauge switch on. Check throttle at idle. Start engine."

Plus the run-up checks: "Once the engine has been brought up to normal operating temperatures, check that the engine will produce sufficient RPM … test the ignition switches if the engine has dual ignition systems installed. By turning one switch off and checking the RPM and then alternating … assure that both ignition switches are operational." And: "Do not fly the powered parachute if the temperature readings are not normal!"

### 5. Wing Layout
Source: Ch. 5 "Wing Inspection," "Line Twists," "Line-Overs."
- "Check the wind direction and manually point the cart directly into the wind."
- "It is critical that the bag not be twisted, rotated or turned when removing it … as doing so will twist and entangle the suspension lines."
- "Place the wing bag on the ground directly behind the airframe … Tilt the wing bag toward the cart to spill the folded wing out."
- "You will see an x in the lines; this x should be positioned directly behind the centerline of the prop."
- "Remove the protective sleeves that cover the suspension lines (line sleeves)."
- "While laying out the wing, check for tears in the fabric, torn or loose stitching, abrasions, and deterioration … check for line twists and line-overs." (Line-over: "If a line is over the top surface of the wing, the pilot risks serious injury or death if takeoff is attempted.")

### 6. Before Takeoff
Source: Ch. 5 "Engine Warm-Up" safety check + Ch. 6/7 takeoff flow + standard POH passenger-briefing convention.
Final pre-takeoff gate: wind re-check, passenger/helmet/intercom briefing, prop area clear, instruments in the green, controls free and correct, take off into wind. ("take off into wind" / kiting is the controlling factor per Ch. 5 "Preparing for Takeoff" → Ch. 7.)

### 7. In-Flight Practice  — **counter items**
Source: FAA-S-8081-32 PTS Areas of Operation (takeoffs & landings, performance maneuvers, ground reference, emergency operations) + Ch. 6 "Four Fundamentals." These are *tally* items (how many of each maneuver you practiced this session), which is why `item_type = COUNTER` rather than a checkbox. This is the one template that must keep counter behavior (per project gotcha: counters, not checkboxes).

### 8. In-Flight Checks (periodic monitoring) and Pre-Landing / Landing
Source: Ch. 5 "Engine Warm-Up" temperature-monitoring guidance carried into cruise; Ch. 7 landing flow. "release the flare on the wing; this is done to prevent the aircraft from becoming airborne again. Not releasing the flare on landing is a critical and common mistake."

### 9. Post Flight
Source: Ch. 5 "Clearing the Runway," "Parking," "Postflight," "Packing the Wing."
- "taxi to the edge of or even off of the runway before collapsing the wing."
- "select a location which will prevent the propeller blast of other airplanes from striking the powered parachute broadside … engine surfaces will be extremely hot."
- "pull the trailing edge of the wing forward toward the cart and roll the leading edge and cell openings under."
- "A flight is never complete until the engine is shut down and the aircraft is secure … accomplish a postflight inspection … the oil should be rechecked and fuel added if required … put the wing properly back in the bag to keep it out of the sun."
- "Packing the Wing back into the wing bag at the end of the flight is a necessary task … directly affects whether or not the wing is easy to unpack for the next flight."

## Data-model decisions that flow from the sources

| Decision | Source / rationale |
|---|---|
| Templates have ordered **sections** (e.g. "Cart," "Fuel & Oil," "Powerplant," "Wing Setup") | Handbook organizes preflight by physical area; sections group items in both editor and run views. |
| Per-item `item_type`: **Checkbox** vs **Counter** | Inspection/procedure items are checkboxes; In-Flight Practice maneuvers are tallies (PTS maneuver list). Toggle per item. |
| Optional per-item **`note`/help text** | Many handbook items carry "why" guidance (e.g. why the fuel vent matters); surfaced as item help. |
| `is_default` templates seeded; user can **duplicate, edit, reorder, deactivate** | FAA says fly your *make/model* checklist — defaults are a starting point, customization is required. |
| `required` flag per item (default true) | Lets a pilot mark advisory items optional so completion % is meaningful. |
| Run view records a **ChecklistLog** with per-item snapshots | Recordkeeping; logs survive template edits (description/section snapshotted). |
| Per-section / per-template **display config** persisted via `AppSetting` (`config.checklists.*`) | UX standard: per-section gear menu (show notes, compact mode, confirm-on-complete, default flight link). |

## Default template set shipped by the seed

`Pre-Flight (Detailed)` · `Pre-Flight (Quick)` · `Engine Start` · `Warm-Up / Run-Up` · `Wing Layout` · `Before Takeoff` · `In-Flight Checks` · `Pre-Landing` · `Landing` · `Post Flight` · `In-Flight Practice` (counters).

All item text is grounded in the quotes above. These mirror the mobile apps' 7-checklist concept while expanding to the full handbook ground flow.
