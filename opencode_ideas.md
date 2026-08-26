# WebMCP Challenge — Medical / Healthcare Deep Dive

> 10-day challenge: Build a WebMCP-powered web app where humans + agents collaborate on same live page/session.
> Judging: WebMCP Leverage / Execution / Potential Impact / Creativity & Ambition — `instruction.md:41-49`
> Requirement: Live URL (ChatGPT in-app browser or Chrome `chrome://flags/#enable-webmcp-testing`) + `document.modelContext.registerTool()` `instruction.md:23`

**Why Healthcare wins WebMCP:**
- `developer.chrome.com/docs/ai/webmcp` — WebMCP executes *in page JS context* with user's auth/session. Backend MCP requires replicating PHI to server = HIPAA risk. WebMCP can be **local-first, zero-footprint** — data never leaves browser tab.
- `webmachinelearning.github.io/webmcp: Goals` — Medicine *requires* human-in-the-loop, confirmation dialogs for consequential actions. Perfect fit for `learn.chatgpt.com/docs/webmcp: Security and user controls` safety review before each tool.
- Healthcare is 80% terrible forms + human-first canvases (DICOM viewer, growth chart, pill box). Chrome docs: `Fill structured forms, Support human-first interfaces`

**Build rule:** Use synthetic data (Synthea FHIR, NIH ChestX-ray sample, OHIF dummy DICOM). Add `disclaimer: not a medical device` banner. Mark sensitive tools `annotations: { destructiveHint: true }` to force confirmation.

---

## CATEGORY A: PATIENT-OWNED, PRIVACY-FIRST (Strongest Impact Narrative)

### 1. CuraCanvas — Zero-Footprint DICOM Co-Viewer
**One-liner:** OHIF-style viewer where radiologist + agent annotate same scan.  
**Problem:** Rural clinic waits 3 days for second opinion. Existing viewers need PACS migration.  
**Why WebMCP vs Backend:** Agent sees same viewport, windowing, measurements. No DICOM upload to LLM server — tool reuses Cornerstone.js client logic in-browser.  
**Human+Agent impossible before:** Human: "Show me all nodules >4mm in this series and compare to last month's scan" → agent calls `measure_nodule` + `load_prior_comparison` → overlay appears → human confirms/rejects. Before: agent would screenshot and guess coordinates.  
**Tools (8):** `list_series()`, `set_windowing(preset)`, `measure_distance(points)`, `segment_nodule(seedPoint)`, `compare_prior(priorStudyId)`, `dictate_finding(structured)`, `export_annotated_pdf()`, `request_second_opinion(consensusHint:true)`  
**Sponsor:** Cloudflare R2 for encrypted DICOMweb cache, Vercel for viewer.  
**Demo (3min):** 60s upload 2 studies → 60s agent measures → 30s human corrects → 30s export.  
**Leverage: ★★★★★ | Impact: ★★★★★**

### 2. LabStory — Your Labs, Finally Readable
**Problem:** Patients get PDF labs, no trend. Cardiologist's PostVisit.ai (Anthropic winner Feb 2026, built in 7 days, 13k applicants) proved post-visit gap.  
**Why WebMCP:** Parses PDF in-browser (PDF.js + on-device LLM via WebLLM) → `extract_labs` → renders DuckDB-Wasm chart. Agent never sends PHI to cloud.  
**Flow:** Drop 5 years of labs → agent `extract_labs` → timeline → "Why is my A1c up?" → `correlate_meds(metformin adherence)` → suggests questions for doctor.  
**Tools (7):** `extract_labs(fileIds)`, `normalize_loinc()`, `chart_trend(test, years)`, `flag_outlier(referenceRange)`, `correlate_with_meds`, `generate_doctor_questions`, `redact_and_share(shareHint)`  
**Feasibility:** Low — Synthea synthetic labs, Chart.js. Highest judge empathy.

### 3. PillMap — Visual Polypharmacy Negotiator
**Problem:** 40% of elderly take 5+ meds; interaction checks are text lists, not visual.  
**Human+Agent:** Human drags pills onto weekly pillbox canvas → agent `check_interactions` → draws red arcs between conflicting pills *on canvas* → `suggest_timing_shift` → human approves → updates calendar.  
**Tools (7):** `add_medication(name,dose)`, `check_interactions()`, `check_duplicate_ingredient`, `suggest_schedule(chronotype)`, `simulate_adherence(missedDose)`, `export_for_pharmacist`, `set_reminder(time)`  
**Sponsor:** Netlify/Render for PWA offline.

### 4. WoundWatch — Longitudinal Photo Timeline
**Problem:** Diabetic foot, pressure ulcers — healing tracked in camera roll, no measurement.  
**Flow:** Human takes weekly wound photo with ruler → agent `estimate_area(pixels→mm)`, `track_healing_speed` → charts on timeline → `flag_stall` (alert to see clinician).  
**Tools (6):** `capture_photo(consent)`, `calibrate_scale(ruler)`, `segment_wound`, `measure_area`, `chart_healing`, `generate_report_for_clinic`  
**Privacy win:** All segmentation via ONNX in-browser (no photo leaves device).

### 5. AfterVisit — Informed Consent That You Understand
**Problem:** Consent forms 12 pages, patients sign blindly.  
**Flow:** Surgeon shares consent canvas (risks/benefits diagram) → patient + agent co-review at home → agent `explain_risk_in_plain_language(riskId, readingLevel:6)`, `quiz_understanding()` → `record_questions_for_surgeon` → signed only after 80% quiz.  
**Tools (6):** `explain_clause(clauseId, level)`, `visualize_risk(probability)`, `quiz_check`, `record_question`, `capture_signature(destructiveHint)`, `revoke_consent`

---

## CATEGORY B: CLINICIAN CO-PILOT (High Execution Score)

### 6. OncoBoard — Virtual Tumor Board Workspace
**Problem:** MDT meets 1hr/week, slides scattered.  
**Human+Agent:** Oncologist drops pathology + imaging + FHIR → agent `extract_stage(TNM)`, `check_guideline(NCCN)`, `suggest_trial` → places cards on shared board → human drags to prioritize → agent `draft_board_summary`.  
**Tools (9):** `ingest_pathology_report`, `extract_biomarkers`, `stage_cancer`, `check_guideline`, `search_trial(eligibility)`, `visualize_timeline`, `poll_board_members`, `draft_consensus`  
**Novel vs saturated:** Not a symptom checker — workflow board.

### 7. DoseCraft — Pediatric Double-Check
**Problem:** Pediatric dosing weight-based, 10x errors fatal.  
**Flow:** Human enters weight/age/allergy → agent `calculate_dose(drug, weight)` → shows math *on page* with formula visualization → requires human confirmation → second agent call `independent_recalculate` for double-check pattern.  
**Tools (5):** `calculate_dose`, `check_allergy`, `show_formula_steps`, `double_check(destructiveHint:true)`, `print_label`  
**Impact:** Safety + education, perfect for human-in-loop judging.

### 8. RehabLoop — Physio Motion Mirror
**Problem:** PostOp Physical Therapy AI winner (CAIDF hackathon 2022) used MediaPipe but server-side.  
**Why WebMCP:** Webcam pose detection (MediaPipe in-browser) → agent `assess_form(poseData)` → draws skeleton overlay live → `count_reps`, `correct_posture` → `generate_progress_chart`.  
**Flow:** Patient does squat → canvas skeleton → agent: "knee valgus, try 10° out" — human fixes instantly.  
**Tools (6):** `start_camera(consent)`, `assess_form(exercise)`, `count_rep`, `feedback_posture`, `log_session`, `share_with_physio`  
**Sponsor:** Vercel AI SDK + Chrome on-device.

### 9. AnesthesiaPlan — Pre-Op Risk Planner
**Target:** Anesthesiologist pre-op clinic.  
**Flow:** Imports FHIR problem list → agent `calculate_ASA()`, `predict_airway_difficulty`, `suggest_plan` → human adjusts → `generate_checklist`.  
**Tools (7):** `parse_fhir`, `calc_asa`, `assess_airway`, `check_medication_hold`, `suggest_anesthesia_type`, `flag_contraindication`, `export_plan`

### 10. ScribeAssist — Point-of-Care EHR That Respects Clicks
**Problem:** Doctors spend 49% on EHR. Ambient scribes send audio to cloud.  
**Why WebMCP wins:** Agent + doctor share same encounter-note canvas. Agent tools: `transcribe_encounter(localWhisper)`, `extract_SOAPlines`, `suggest_code(ICD10)` → inserts as *uncommitted suggestion* (human must accept). Reuses page's existing note logic.  
**Tools (8):** `start_ambient(consent)`, `propose_soap`, `link_to_problem(problemId)`, `suggest_code`, `check_documentation_gap`, `accept_suggestion(id)`, `reject_suggestion`  
**Privacy:** Audio never leaves tab — judge Justin Rushing (OpenAI Browser Lead) loves this.

---

## CATEGORY C: CAREGIVER & COORDINATION

### 11. CareCircle — Family Handoff Board
**Problem:** ICU families get updates via hallway whispers. Caregivers burn out.  
**Flow:** Shared board: `log_meds_given`, `track_mood`, `schedule_visit`, agent `summarize_shift(nurse, family)`, `detect_caregiver_burden`, `suggest_respite_resource`.  
**Tools (8):** `add_task`, `log_event`, `summarize_for_next_caregiver`, `detect_burnout_risk`, `translate_update(language)`, `share_with_team`, `request_clarification_from_nurse`  
**Impact:** Loneliness + continuity — heartbeat story. Inspired by ExperifyHealth (Ottawa Hackathon winner).

### 12. TrialMatch — Local Clinical Trial Screener
**Problem:** 86% eligible patients never hear about trials.  
**Why WebMCP:** Parses patient's local FHIR in-browser → `match_trial` → shows *why* eligible on same page with `explain_criterion` — no upload until consent.  
**Tools (6):** `parse_eligibility`, `match_trial(zip, condition)`, `explain_match`, `pre_screen(checklist)`, `contact_site(destructiveHint)`, `save_preference`

### 13. MigraineMap — Trigger Correlation Dashboard
**Live canvas with DuckBoard pattern:** Human logs headache/mood/food/weather on calendar canvas → agent `correlate_triggers` → DuckDB-Wasm query visualization → `suggest_experiment(eliminate_trigger)` → human runs A/B.  
**Tools (7):** `log_attack`, `import_weather`, `correlate(food, weather, sleep)`, `visualize_heatmap`, `suggest_experiment`, `track_adherence`, `export_for_neurologist`

### 14. SpeechScaffold — Aphasia Communication Board
**Accessibility winner:** Stroke patient points at pictograms + types fragment → agent `expand_to_sentence(fragment, context)`, `speak_with_voice`, `predict_next_need`. Shared board with caregiver. Addresses `github.com/webmachinelearning/webmcp/issues/91` — agent as AT intermediary.  
**Tools (6):** `predict_phrase(context)`, `expand_fragment`, `speak(text)`, `log_success`, `adapt_board(usage)`, `share_board_with_family`

### 15. NICU Loop — Parent + Nurse Shared Growth Chart
**Inspired by UIC CAIDF hackathon NICU winner:** Preterm infant daily weight/feed/logs → agent `plot_fenton_curve`, `flag_growth_falter`, `translate_jargon_for_parents`, `suggest_question_for_rounds`.  
**Tools (7):** `plot_growth(weight, gestationalAge)`, `flag_falter`, `explain_in_plain_language(term)`, `suggest_question`, `share_with_parents`, `export_for_ehr`

---

## CATEGORY D: PUBLIC HEALTH & TRAINING

### 16. OutbreakLens — Community Health Worker (ASHA) PWA
**SIH winner pattern:** Offline-first (IndexedDB) + Cloudflare Workers. ASHA worker logs fever cases on village map → agent `cluster_outbreak`, `predict_spread(simple SIR)`, `suggest_referral`. Works offline, syncs when online.  
**Tools (7):** `log_case`, `cluster_geo`, `flag_outbreak_threshold`, `suggest_referral`, `generate_report_for_PHc`, `sync_when_online`

### 17. SimMan — Medical Simulation Scenario Builder
**Education:** Instructor builds manikin scenario on timeline canvas → agent `generate_vitals_trajectory(sepsis)`, `trigger_deterioration`, `quiz_student_action`, `debrief_with_timeline`.  
**Tools (7):** `set_baseline_vitals`, `trigger_event(event)`, `assess_student_action`, `branch_scenario(decision)`, `score_performance`, `generate_debrief`

### 18. ContraCheck — Contraception Decision Canvas
**Sensitive but high impact:** Human explores options privately (local-only). Agent `compare_options(preferences)`, `visualize_efficacy`, `bust_myth`, `find_clinic` → never stores. Shows WebMCP privacy edge over backend chatbots.  
**Tools (6):** `compare_options`, `visualize_efficacy`, `explain_side_effect`, `myth_bust`, `find_clinic(zip)`, `save_private_preference`

### 19. FoodLab — Allergy-Safe Meal Planner for Clinics
**Extend Sunday Table showcase:** Dietitian + renal/diabetic patient co-plan on meal canvas → agent `check_nutrient_limits`, `swap_for_restriction`, `generate_shopping_list` → pushes to cart but *clinical rules-driven*.  
**Tools (7):** `add_recipe`, `check_limits(sodium, potassium, carbs)`, `swap_ingredient(restriction)`, `scale_for_household`, `generate_shopping_list`, `export_meal_plan`

### 20. BurnCalc — Parkland Formula Interactive
**Emergency training:** Human draws burn % on body diagram canvas → agent `calc_fluid_resuscitation(weight, TBSA)`, `adjust_for_comorbidity`, `timeline_fluids`. Visual, immediate, life-saving teaching.  
**Tools (6):** `estimate_TBSA(markings)`, `calc_parkland(weight, TBSA)`, `adjust_for_renal`, `timeline_infusion`, `quiz_next_step`, `export_protocol`

---

## TOP 7 HEALTHCARE PICKS FOR WINNING

| Rank | Pick | Why Judges Will Fund It | Build in 10 Days? | WebMCP Leverage |
|---|---|---|---|---|
| **1** | **LabStory** | Emotional patient story + PostVisit.ai trend (proven winner) + local privacy. Strong `Potential Impact` `instruction.md:46` | YES: Synthea + Chart.js + PDF.js. 1 dev. | ★★★★★ 7 tools, WASM, dynamic |
| **2** | **CuraCanvas** | Zero-footprint DICOM = Cloudflare+Ilya Grigorik love + technical depth | MEDIUM: Fork OHIF viewer, add 6 tools | ★★★★★ Canvas + measurements |
| **3** | **RehabLoop** | MediaPipe in-browser = Chrome on-device AI showcase + CAIDF winner validation | YES: MediaPipe demo exists | ★★★★★ Live skeleton overlay |
| **4** | **ScribeAssist** | Solves #1 clinician burnout + perfect human-in-loop uncommitted diff pattern from spec `webmachinelearning.github.io/webmcp: edit-design batch` | YES: Mock FHIR, Whisper-tiny local | ★★★★★ Shared note canvas |
| **5** | **CareCircle** | Impact on caregivers + beats loneliness (ExperifyHealth Ottawa winner) + strong video story | YES: Simple board, no DICOM | ★★★★☆ Handoff summaries |
| **6** | **DoseCraft** | Safety double-check = unforgettable demo (judge sees math verified) + low tech risk | YES: Formula viz only | ★★★★☆ Double-check chaining |
| **7** | **SpeechScaffold** | Accessibility + WebMCP a11y debate = Sarah Drasner (Chrome) spotlight | MEDIUM: Pictogram set + TTS API | ★★★★★ A11y + AT intermediary |

**Avoid:** Generic `SymptomAI` / `MedTrack` / `DocConnect` from `placementpreparation.io` table — judges have seen 100x, zero WebMCP canvas leverage.

## Execution Checklist (Healthcare)

**1. Sponsor-aligned stack:**
- Cloudflare Workers + R2 (encrypted) for LabStory/CuraCanvas — wins Cloudflare $10k credits prize
- Vercel AI SDK + Next.js for RehabLoop/ScribeAssist — wins Vercel $4.2k credits
- Render/Netlify PWA for OutbreakLens — offline-first

**2. Implement BOTH APIs:**
- Declarative `<form tool>` for `submit_referral` / `capture_signature`
- Imperative `document.modelContext.registerTool` for `chart_interaction` / `canvas_update`
- Add `toolchange` listener so tools appear after state (e.g., after `upload_dicom`, `segment_nodule` registers). Demo this in video.

**3. Safety:**
- Annotate sensitive tools: `annotations: { readOnlyHint: false, destructiveHint: true }` → forces confirmation dialog `developer.chrome.com/docs/ai/webmcp/best-practices`
- Show banner: "Prototype with synthetic data — not a medical device"

**4. 3-min video script (40-40-40):**
- 0:00-0:40 Problem + patient quote
- 0:40-1:40 Human + agent co-editing LIVE (split screen: human drags, agent calls tool, UI updates visibly)
- 1:40-2:20 Code walk: `registerTool` snippet + `inputSchema` + reuse of client logic
- 2:20-3:00 Impact + sponsor stack + what's next

**5. Repo:** Public GitHub + LICENSE file detectable in About section `instruction.md:20`

---

## Appendix: General Novel Ideas (Non-Healthcare, from earlier research)

**For differentiation — avoid building these in healthcare track:**
- BazaarBazaar — Reverse Haggle Market (cross-iframe `exposedTo`/`fromOrigins`)
- Threadline — Non-linear narrative quilt
- BuildScope — PR Review Visualizer (Monaco + GitHub)
- TownHall — Deliberative polling Sankey
- EvidenceBoard — Investigative journalist workbench

All share same WebMCP pattern: shared canvas + uncommitted diff + confirmation.

---

*Generated for WebMCP Challenge deadline Sep 3, 2026 1pm PDT — 885 participants as of Aug 26, 2026.*
*Sources: openai.com/webmcp-challenge, webmcp.devpost.com, developer.chrome.com/docs/ai/webmcp, webmachinelearning.github.io/webmcp, learn.chatgpt.com/docs/webmcp, Chrome EPP blog Feb 10 2026.*

