0# Create all 3 skill directories
mkdir -p .claude/skills/ces-simulation-mechanics
mkdir -p .claude/skills/ces-clinical-guide
mkdir -p .claude/skills/ces-clinical-reasoning

Skill 1

cat > .claude/skills/ces-simulation-mechanics/SKILL.md << 'ENDOFSKILL'
---
name: ces-simulation-mechanics
description: How the CES turn-based ED simulation works — turns, action lifecycle, resource constraints, and patient deterioration. Load this skill when working in the Clinical Environment Simulator.
---

# CES Simulation Mechanics

## Turns
- 1 turn = 5 simulated minutes. Typical scenario: 72 turns (6 hours).
- Nothing happens automatically. YOU must click **"Step"** to advance each turn.
- **"Run Until Event"** advances multiple turns automatically and stops when something significant happens (patient arrival, test result, code blue, deterioration).

## Action Lifecycle
Every order follows this pipeline:

You select test/med/procedure → added to Pending Actions queue
→ You click "Step" (or "Run Until Event")
→ Backend allocates resources (nurse, physician, CT machine, etc.)
→ Action moves from Pending → In Progress (with progress bar and turn countdown)
→ After N turns → action completes → result appears in Event Log

Key implications:
- Orders do NOT execute instantly. They sit in Pending until you Step.
- Multiple orders across multiple patients can be batched in one turn — do this.
- If a resource is unavailable (e.g., CT machine occupied), the order stays in Pending and retries next turn.
- STAT orders get priority and may complete faster (~50% reduction).

## Resource Constraints
The ED has finite resources. Know the bottlenecks:
- **Physician**: 1 attending (you). Procedures, H&P, and exams consume your time — you become "busy" and cannot act until done.
- **Nurses**: ~6. Labs, meds, and bedside tasks require a nurse. Orders batch within 5-minute windows (one nurse visit per batch).
- **ED Beds**: ~4. Patients in the waiting area cannot receive bedside orders. You must room them first.
- **Imaging**: 1 CT, 1 MRI, 1 X-ray machine. Imaging queues if multiple patients need scans.
- **Procedure Room**: 1. Exclusive use — only one patient at a time.

## Patient Deterioration
Patients have hidden clinical thresholds. If you fail to complete critical actions (specific tests, meds, procedures) within a time window:
- Vitals decline (HR spikes, BP drops, O2 drops)
- Patient gets flagged with a red indicator
- Code Blue may trigger if deterioration is severe
- The simulation tracks this for your evaluation score
ENDOFSKILL



Skill 2
cat > .claude/skills/ces-clinical-guide/SKILL.md << 'ENDOFSKILL'
---
name: ces-clinical-guide
description: Clinical decision guide for ED presentations — standard management protocols, treatment completeness framework, admission unit selection, transfer destinations, and critical values. Use when diagnosing or treating patients in the Clinical Environment Simulator.
---

# Clinical Decision Guide

## Standard Management by Presentation

**COLUMN ORDER = ACTION ORDER. Treat and test in the same turn.**

| Presentation                     | TREAT (order alongside tests)                                                                                      | DIAGNOSE (order same turn)                                      | Watch For                                                          | Likely Disposition                                   |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------- | ------------------------------------------------------------------ | ---------------------------------------------------- |
| Chest pain / ACS                 | Aspirin 325mg, Nitroglycerin SL, pain management, anticoagulation if high suspicion                                | ECG, Troponin, CMP, CBC, Chest X-ray                            | Troponin elevation, ST changes                                     | Cardiology / CCU / Cath lab transfer                 |
| Sepsis / Septic shock            | IV NS 30mL/kg bolus, broad-spectrum abx WITHIN 1 HR (Piperacillin-Tazobactam), vasopressors if MAP<65 after fluids | Blood cultures (BEFORE abx), CBC, BMP, Lactate, UA, Chest X-ray | Lactate >2, WBC elevation, positive cultures, hemodynamic response | ICU (if shock) / Medicine                            |
| DKA                              | IV NS 1-2L bolus, Regular insulin drip (HUMULIN R), potassium replacement if K<5.3, bicarb monitoring              | BMP q2h, ABG, CBC, UA, Lipase                                   | pH <7.35, glucose >250, anion gap, K+ level                        | ICU / Medicine                                       |
| Appendicitis                     | Pain management, IV fluids, antibiotics if perforation suspected                                                   | CBC, BMP, Lipase, UA, CT Abdomen/Pelvis w/ contrast             | Elevated WBC, CT showing appendicitis                              | General Surgery                                      |
| DVT                              | Anticoagulation (heparin drip or LMWH), pain management, leg elevation                                             | US Lower Extremity Veins DVT, CBC, BMP, D-dimer                 | Positive DVT on ultrasound, PE symptoms                            | Anticoagulation, Medicine / Discharge with follow-up |
| Pneumonia                        | Antibiotics within 4 hours, O2 if hypoxic, IV fluids if dehydrated, antipyretics if febrile                        | Chest X-ray, CBC, BMP, Blood cultures, Procalcitonin            | Infiltrate on CXR, elevated WBC, hypoxia, severity score           | Pulmonology / Medicine / Discharge (if mild)         |
| Fracture / Trauma                | Pain management (multimodal), immobilization/splinting, IV access                                                  | X-ray or CT of affected area, CBC, BMP, Type & Screen           | Fracture on imaging, neurovascular status                          | Orthopedics / Trauma Surgery                         |
| Altered mental status            | Check glucose IMMEDIATELY, stabilize ABCs, treat reversible causes                                                 | CBC, BMP, UA, CT Head, Toxicology screen, Ammonia               | Metabolic derangement, intracranial pathology                      | Neurology / ICU / Medicine                           |
| GI bleed                         | 2 large-bore IVs, IV fluid resuscitation, PPI drip, Type & Screen early                                            | CBC (serial), BMP, Coagulation studies, Lactate                 | Dropping hemoglobin, coagulopathy, hemodynamic instability         | GI / General Surgery                                 |
| Inguinal hernia                  | Pain management, assess for incarceration/strangulation                                                            | Physical exam, CBC, BMP                                         | Incarcerated vs reducible, signs of ischemia                       | General Surgery / Discharge                          |
| Eye complaints (hordeolum, etc.) | Topical treatment as indicated, pain management if needed                                                          | Focused exam, visual acuity                                     | Red flags: vision loss, orbital signs                              | Discharge with follow-up                             |
| Laceration                       | Local anesthesia, wound irrigation, laceration repair                                                              | Wound exam, X-ray if foreign body suspected                     | Wound characteristics, tendon/nerve involvement                    | Discharge after repair                               |
| Subungual hematoma               | Pain management, nail trephination                                                                                 | X-ray of digit                                                  | Fracture on X-ray                                                  | Discharge / Orthopedics                              |
| Knee pain                        | Pain management, immobilization if unstable                                                                        | X-ray Knee, CBC, ESR/CRP; Consider joint aspiration if effusion | Fracture, effusion, crystal analysis                               | Orthopedics / Discharge                              |

## Treatment Completeness Framework

A single intervention is rarely a complete treatment plan. Before moving on from any patient, sweep through these SEVEN categories. Not all apply to every patient — but you must consciously consider each one and decide to include or skip. Skipping because you forgot is an error. Skipping because it's not indicated is correct practice.

**The 7-Category Sweep:**

**1. PRIMARY THERAPY — Am I treating the actual condition?**
   What is the standard first-line treatment? Does it involve multiple agents working through different mechanisms? If the guideline calls for A, B, and C together, ordering only A is incomplete — even if A is the "most important" one. Each agent in a multi-drug regimen exists for a reason.

**2. SYMPTOM RELIEF — Is the patient suffering right now?**
   Pain, nausea, fever, anxiety, breathlessness — these are independently treatable. For pain: one drug class is rarely optimal. Evidence supports combining agents with different mechanisms (central, peripheral, anti-inflammatory) for better relief at lower individual doses. This multimodal principle applies whenever pain is significant.

**3. FLUID & ELECTROLYTE MANAGEMENT — Which direction?**
   Before ordering any fluid, determine the correct DIRECTION. Some patients need volume in. Some need volume restricted or removed. Getting this wrong actively harms the patient. Also consider: are electrolytes deranged? Do they need correction or monitoring?

**4. SUPPORTIVE & PREVENTIVE CARE — What am I preventing?**
   Think beyond the current problem. Is this patient at risk for clots (immobility, inflammation, surgery)? Am I prescribing something that creates a secondary risk (gastric irritation, constipation, metabolic disruption)? Is there a respiratory, skin, or positioning need?

**5. REVERSAL & CORRECTION — Is something actively harmful present?**
   Is the patient on a medication that is now dangerous given the new clinical situation? Is there a toxin, overdose, or derangement requiring active reversal? Sometimes the most important treatment is stopping or reversing something.

**6. MONITORING — How will I detect change?**
   What needs continuous vs intermittent monitoring? What lab values need repeat trending? What clinical signs should trigger escalation? Is the monitoring level matched to the acuity?

**7. CONSULTATION & DISPOSITION — Who else, and what's the exit plan?**
   Does this patient need specialist input? What must be completed before disposition? What does the patient need at discharge or on the receiving unit?

**How to use this:** At initial assessment and again before disposition, run the sweep: 1-2-3-4-5-6-7. For simple cases, 2-3 categories apply. For complex cases, all 7. The ones you skip should be deliberate, not accidental.

## Admission Unit Selection
- **ICU**: Hemodynamic instability, respiratory failure, septic shock, severe DKA, active code blue recovery
- **CCU / Cardiology**: ACS, STEMI, arrhythmias, acute heart failure
- **Medicine/Hospitalist**: Stable infections, DM management, general medical admissions
- **General Surgery**: Appendicitis, cholecystitis, bowel obstruction, hernias requiring repair
- **Trauma Surgery**: Multi-system trauma, significant injury mechanisms
- **Orthopedics**: Fractures, dislocations, septic joints
- **Neurology**: Stroke, seizures, altered mental status with neuro cause
- **Pulmonology**: Severe pneumonia, COPD exacerbation, PE
- **Psychiatry**: Psychiatric emergencies, suicidal ideation, acute psychosis
- **OB/GYN**: Pregnancy complications, gynecologic emergencies

## Transfer Destinations
- **Cath lab**: STEMI, acute coronary intervention
- **Interventional radiology**: Active hemorrhage requiring embolization, abscess drainage
- **OR**: Surgical emergencies (ruptured appendix, incarcerated hernia, trauma requiring surgery)

## Critical Values — Act Immediately

| Vital | Critical Range | Action |
|---|---|---|
| HR | >120 or <50 | Assess rhythm, consider pharmacologic intervention or pacing |
| BP systolic | <90 | IV fluid bolus, vasopressors if refractory, consider sepsis/hemorrhage |
| BP systolic | >180 | Antihypertensives, assess for end-organ damage |
| SpO2 | <90% | Supplemental O2, escalate to high-flow or intubation if <85% |
| Temp | >103°F | Sepsis workup, antipyretics, cultures before antibiotics |
| Temp | <95°F | Warming measures, consider sepsis/exposure |
| RR | >24 or <10 | Respiratory distress workup, consider airway management |
| GCS | <8 | Airway protection, intubation, CT Head, Neurology consult |
ENDOFSKILL




skill 3
cat > .claude/skills/ces-clinical-reasoning/SKILL.md << 'ENDOFSKILL'
---
name: ces-clinical-reasoning
description: Clinical reasoning principles, treatment-first approach, reassessment framework, disposition verification checklist, and patterns that lead to poor scores. Use when making clinical decisions and before every disposition in the Clinical Environment Simulator.
---

# Clinical Reasoning — Every Patient, Every Encounter

Before placing ANY order, run this loop:

Stable or unstable? → Check vitals and mental status
Needs treatment right now? → Treat BEFORE or WITH diagnostic orders
What's the differential? → 3 most dangerous, 3 most likely
What workup narrows it? → Order targeted tests, not everything
Reassess after interventions/results → Better, same, or worse?
Ready for disposition? → Run the checklist below first

Rule: Treatment is not the reward for making a diagnosis. It is the first thing you do based on the presentation. If a patient has a treatable symptom (pain, nausea, hypoxia, hypotension, distress), treat it immediately — do not wait for test results.

## Treatment-First Principle

**Three Mandatory Questions Before Any Test Order**

"Is this patient in pain or distress?" → If yes, order symptom relief NOW
"Is this patient hemodynamically unstable?" → If yes, stabilize NOW
"Is there a standard initial intervention for this presentation that doesn't require test confirmation?" → If yes, start it NOW

**Completeness Rule**
Most conditions require MULTIPLE simultaneous treatments, not one. Before moving to the next patient, ask: "Does this condition have a standard multi-component regimen? Did I order ALL components or just the first one I thought of?"

**Non-Pharmacologic Interventions Exist**
Medications are not the only treatments. For every patient ask: "Is there a procedural, positional, supportive, or preventive intervention this patient needs?" (examples: immobilization, splinting, wound care, breathing exercises, fluid restriction, prophylaxis for secondary complications, oxygen supplementation)

**The "No Naked Diagnosis" Rule**
Never submit a disposition without asking: "Did I TREAT this patient, or did I only DIAGNOSE them?" A correct diagnosis with no treatment is incomplete emergency care.

## Treatment Timing Principle
Order treatments and tests in the SAME TURN.

Tests take multiple turns to result — so order them early. Correct.
Treatments help the patient immediately — so order them early. Also correct.
Conclusion: order both together. Sequencing them (tests first, treatment after results) wastes time and leaves the patient untreated during the wait.

The only treatments that should wait for results are those whose selection or dose depends on a specific test value. For everything else, ask: "Am I delaying this treatment because I genuinely need information I don't have yet, or because I defaulted to test-first thinking?" If the latter, order the treatment now.

## Dose Awareness
The acute ED dose is often different from the chronic outpatient dose. When ordering any medication:
- Match the dose to the INDICATION and ACUITY, not to the lowest available option
- The same drug may require very different doses for different clinical purposes
- Under-dosing the right drug is a treatment error — the patient gets side-effect risk without therapeutic benefit
- Adjust for patient factors (renal function, weight, age) but adjustment means calibrating, not reflexively minimizing

## Reassessment and Consult Integration

**Reassessment**
After every intervention, close the loop: "Did it work?" Check vitals after fluids. Check pain after analgesia. Check SpO2 after oxygen. Check rhythm after rate control. If not improving → escalate, don't ignore.

**Consult Responses**
When a consult returns, read the "Must-do" recommendations. These are NOT optional — order them before dispositioning. If the consultant requests information (exam findings, labs), provide it. Unaddressed must-do items = incomplete care.

**Reassessment Principle — Close Every Loop**
Every treatment creates an open loop. Before disposition, close it.

The pattern: Intervene → Wait appropriate interval → Check response → Adjust or proceed

For any hemodynamic intervention, check vitals after. For any analgesic, check pain after. For any respiratory intervention, check oxygenation and work of breathing after. For any metabolic correction, recheck the lab.

If you are dispositioning and you never verified whether your treatments worked, you are making a disposition based on hope. Always close the loop before closing the chart.

## Before Every Disposition — Mandatory Verification
Before clicking Discharge/Admit/Transfer, answer these in order. If any step fails, STOP and go back.

**Step 1 — Count:** How many distinct treatment interventions did I order for this patient? If the answer is zero or one, you almost certainly undertreated. Most acute presentations need 3-6 interventions across multiple categories.

**Step 2 — Sweep:** Run the 7-Category Sweep against THIS patient:
  1. Primary therapy — complete standard regimen, correct doses?
  2. Symptom relief — pain/nausea/fever addressed?
  3. Fluids/electrolytes — right direction, right type?
  4. Supportive/preventive — complications being prevented?
  5. Reversal/correction — anything harmful stopped or reversed?
  6. Monitoring — appropriate for acuity, trends checked?
  7. Consults/disposition — specialist input obtained, recommendations acted on?
For each: "addressed," "not applicable because [reason]," or "I missed this." Any "I missed this" → go back and fix before dispositioning.

**Step 3 — Loop Check:** For every treatment I ordered, did I verify it worked? If I gave something to change a vital sign, did it change? If I gave something for a symptom, is it better? If I don't know, I am dispositioning too early.

**Step 4 — Gut Check:** "Did I treat this patient, or did I only test and route them?"

## Patterns That Lead to Poor Scores

Diagnose-and-dispose without treating — Correct diagnosis, correct disposition, zero symptom relief or stabilization.
Serial workflow — Completing Patient A fully while B through G wait untouched.
Over-testing, under-treating — Ordering every available test but no symptom management.
Partial regimens — One drug when the standard of care requires multiple simultaneous agents.
Ignoring non-pharmacologic interventions — Forgetting procedures, precautions, and prophylaxis that aren't pills.
Ignoring consult action items — Reading consult notes but not acting on must-do recommendations.
No reassessment — Intervening and never checking if it worked.
Holding beds for perfection — Waiting for one more test on a stable patient while a sicker one sits in triage without access to treatment.

Additional execution patterns that lead to poor scores:

- The Single-Agent Trap — Ordering one drug when the standard regimen is multi-drug. Many acute conditions use agents with complementary mechanisms given simultaneously. Before leaving any patient: "Does this typically require multiple agents together? Did I order all of them, or just the first one I thought of?"

- Right Drug, Wrong Dose — Ordering the correct medication at an inappropriate dose for the acute setting. The chronic/outpatient dose is often lower than the acute/ED dose. Match dose to situation.

- Reflexive Fluid Ordering — Defaulting to IV fluids for every patient. Some conditions are worsened by volume loading. Determine the correct fluid DIRECTION (in, out, or neutral) before ordering.

- Diagnosis Without Treatment — Ordering comprehensive diagnostics but zero therapeutic interventions. A patient who receives labs and imaging but no treatment for their symptoms or condition has been studied, not treated.

- Missing the Follow-On Obligation — Many treatments create secondary requirements: monitoring obligations, side-effect prophylaxis, dose adjustments based on response. Think one step ahead: "What does starting this treatment obligate me to also do?"

- Disposition Without Verification — Sending a patient out without confirming treatments worked. If you changed something, confirm it changed.

- The Pill-Only Blind Spot — Forgetting that treatments include procedures, devices, positioning, exercises, restrictions, and precautions. For every patient: "Is there something this patient needs that isn't a medication?"
ENDOFSKILL

Verify: ls -la .claude/skills/*/SKILL.md
