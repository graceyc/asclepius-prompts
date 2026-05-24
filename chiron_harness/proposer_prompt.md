---
name: meta-harness-ces
description: Run one iteration of operating manual evolution for CES.
---

# Meta-Harness (CES Operating Manual)

Run ONE iteration of operating manual evolution.

**You do NOT run benchmarks.** You analyze prior results and traces, diagnose failure patterns, and write one improved operating manual. The user evaluates candidates in separate sessions.

Do all work in the main session — do NOT delegate to subagents. Context gets lost when you delegate, leading to shallow analysis and untargeted changes.

## CRITICAL CONSTRAINTS

- You MUST produce exactly 1 new operating manual every iteration.
- Do NOT write "the manual is optimal" or "stop iterating", or abort early.
- Do NOT skip trace analysis. Read failed patient traces before proposing changes.

### Anti-overfitting rules

- **No patient-specific or batch-specific hints.** Do not hardcode knowledge about specific patients, scenarios, or batch numbers. The manual must generalize across all scenarios.
- **Never reference patient names, specific diagnoses from traces, or batch numbers** in the manual.
- **General clinical workflow guidance is OK.** "Order anticoagulation alongside diagnostics for ACS presentations" is fine — it applies broadly. "Give aspirin to the first patient on turn 2" is overfitting.
- **Never write a rule from a single reasoning trace.** If you open audit_logs/ for a deep-dive (see DEEP-DIVE section below), the reasoning of any individual turn is one data point, not a pattern. A pattern needs ≥3 patients exhibiting the same reasoning failure.
- **If in doubt, make it more general.**

### Substantive-edit rules

The most common failure mode is making cosmetic edits — rewording without changing what the agent actually does. Check `candidates/v*/hypothesis.md` for what's been tried — pure rewording, reorganization, and emphasis changes almost always produce no measurable effect.

**Good candidates change agent behavior:**
- Add a workflow checkpoint that wasn't there before (e.g., a mandatory pre-disposition verification step)
- Replace a vague rule with a concrete threshold (e.g., "early" → "within 2 turns of arrival")
- Restructure decision priorities (e.g., elevate one rule above another, or merge two adjacent rules into a single combined rule)
- Add a row to a decision table for a presentation pattern the agent mishandles
- Remove a confusing or contradictory instruction
- Condense a verbose section the agent is demonstrably ignoring

**Bad candidates only change wording.** If you can read your edit and the previous version side-by-side and they would direct identical clinical decisions in identical scenarios, it's cosmetic. Rewrite to change a trigger, threshold, action, or category — not just the prose.

**The substantive-edit test:** could a different physician acting on the new manual produce a measurably different decision in some specific clinical scenario? If no, rewrite.

**Magnitude proportional to failure.** Length is a cost, but minimizing length is NOT the goal. The goal is correctness and effectiveness.

- If the failure pattern affects >20% of patients, your edit should be **substantive** — not minimal. A 30% systematic failure should not be addressed with a +4 line edit.
- Trimming redundancy is allowed, and good when the agent is demonstrably ignoring the redundant content. But trimming does NOT substitute for additions needed to address a pattern. Do not use deletions to "compensate" for required additions and call it a wash.
- If the failure is small (affects <5% of patients, edge case), do not propose a sweeping restructure. Match scope to scale.

**Combining edits is valid** — within one mechanism. Adding a workflow checkpoint AND tightening a related decision rule that supports the same checkpoint is one mechanism. Adding a checkpoint AND a separate rule about a different failure mode is two mechanisms — defer the second to the next iteration.

## CONTEXT

The Clinical Environment Simulator (CES) is a turn-based ED simulation:
- **Agent**: Claude Opus 4.6 (frozen — you cannot change the model)
- **Task**: Manage 12 patients over 72 turns (6 simulated hours) via 16 MCP tools
- **Hard deadline**: All patients dispositioned by turn 70
- **Resources**: 1 physician, 4 ED beds, 6 nurses, 1 CT/MRI/X-ray each
- **The operating manual (system prompt) is the ONLY thing you can change.** Agent code, MCP tools, simulation backend, and evaluation logic are all fixed.

### Evaluation metrics

4 dimensions, each scored 1-5 per patient by a GPT-4.1 judge:

| Dimension | Measures | Common agent failures |
|-----------|----------|----------------------|
| **Diagnosis** | Correct, specific diagnosis | Vague ("abdominal pain" not "acute appendicitis"), wrong organ system |
| **Critical Actions** | % of required treatments completed | Incomplete regimens (1 of 3 drugs ordered), missed non-pharm interventions, ignored consult recommendations |
| **Timeliness** | Speed of critical interventions vs hidden deterioration threshold | Treatments ordered only after test results instead of alongside, turns wasted on unnecessary H&Ps, serial workflow |
| **Disposition** | Correct discharge/admit/transfer + correct unit | Wrong unit (Medicine when ICU needed), wrong direction (discharge when admit required) |

**Aggregate**: 4 dimensions × 12 patients × 5 max = 240 points per batch. 6 search batches total.

**Safety violations**: Any score of 1 on any dimension. Tracked separately.

### Hard domain constraints (do NOT target these — they are simulation bugs/limitations)

- Central lines and RSI/intubation crash the simulation (physician-busy bug)
- "Blood Culture Set" never resolves in the order catalog
- "Ketorolac" sometimes fuzzy-matches to wrong drug (manual already says use "toradol")
- Patients must be assigned an ED bed before receiving bedside orders

## DIRECTORY LAYOUT

```
meta_harness/
├── CLAUDE.md                # This file
├── evolution_summary.jsonl  # Append-only log of all candidates
├── batches.json             # Maps batch numbers to session UUIDs (read by scripts only)
├── scripts/
│   ├── extract_traces.py    # Populates candidates/vN/traces/ from ces.db + experiments logs
│   └── extract_audit_logs.py # Adds audit_logs/ subdir per batch (large; deep-dive only)
└── candidates/
    ├── v0/                  # Baseline (or any prior candidate)
    │   ├── operating_manual.md  # The prompt text used for this candidate
    │   ├── scores.json          # Aggregate + per-batch scores (written by extract_traces.py)
    │   └── traces/
    │       ├── batch_01/        # 6 search batches per candidate
    │       │   ├── session_metadata.json  # {session_id, scenario_name, start_time}
    │       │   ├── session_summary.json   # {duration, total_turns, total_patients, ...}
    │       │   ├── turn_records.jsonl     # one JSON per line, one per turn (orders, vitals, test results)
    │       │   ├── evaluations.json       # {patient_id: full_eval_row} for all 12 patients in this batch
    │       │   ├── patients/
    │       │   │   ├── patient_1.md       # grep-friendly view: presentation, scores, missed actions, orders timeline
    │       │   │   ├── patient_2.md
    │       │   │   └── ... patient_12.md
    │       │   └── audit_logs/            # ⚠ ~74 MB. Deep-dive only — see DEEP-DIVE section.
    │       │       ├── session_metadata.json
    │       │       ├── session_summary.json
    │       │       └── turns/
    │       │           ├── turn_0000.json # full state + llm_calls[] (agent prompts, reasoning, tool calls)
    │       │           └── ... turn_NNNN.json
    │       └── batch_02/ ... batch_06/    # same structure
    ├── v1/                  # Your first proposal
    │   ├── operating_manual.md  # Written by you
    │   ├── hypothesis.md        # Written by you
    │   ├── scores.json          # Filled by user after eval (via extract_traces.py)
    │   └── traces/              # Same structure as v0 (filled by user after eval)
    └── ...
```

### What you CAN and CANNOT modify

- **CAN**: Create `candidates/vN/operating_manual.md` and `candidates/vN/hypothesis.md`
- **CAN**: Append to `evolution_summary.jsonl`
- **CANNOT**: Modify any existing candidate directory (v0, v1, ... — these are historical record)
- **CANNOT**: Modify `scripts/extract_traces.py`, `scripts/extract_audit_logs.py`, `batches.json`, or this `CLAUDE.md`
- **CANNOT**: Modify agent code, MCP tools, backend code, evaluation engine, or scenario files
- **CANNOT**: Read patient scenario YAMLs or backend source code (information integrity)

## WORKFLOW

### Step 1: Read state

1. Read `evolution_summary.jsonl` — what's been tried, what worked/regressed
2. Read ALL prior `candidates/v*/scores.json` — score trajectory across iterations
3. Read ALL prior `candidates/v*/hypothesis.md` — what was targeted and whether it worked
4. Read the current best-scoring `operating_manual.md`

### Step 2: Analyze traces (MANDATORY — do NOT skip)

**Improvements without trace analysis are guesses.** This is the most important step.

For the most recent candidate with results:

1. Read `candidates/vN/scores.json` — identify the weakest dimension across all 6 batches.

2. **Start with the per-patient markdown views.** For every patient scoring 1-2 on any dimension, read `candidates/vN/traces/batch_NN/patients/patient_X.md`. These contain presentation, scores across all 4 dimensions, missed critical actions, safety violations, the agent's full orders timeline, and the evaluator's improvement feedback. **This is your primary reading surface.**

3. **Drop down to `evaluations.json` only when the markdown is missing something you need.** `candidates/vN/traces/batch_NN/evaluations.json` is a dict keyed by patient_id with the full raw eval rows. Useful when you want to scan all 12 patients in a batch quickly, or when you need a field the markdown didn't render.

4. **Drop down to `turn_records.jsonl` only for exact timing reconstruction.** For the 3-5 worst-scoring patients, read `candidates/vN/traces/batch_NN/turn_records.jsonl` (one line per turn) to answer:
   - What turn did the patient arrive vs. when was the first treatment ordered?
   - Were orders batched across patients or placed one-at-a-time?
   - Were there idle turns where the agent could have been acting?
   - Was H&P ordered unnecessarily? Was it ordered before urgent treatment?

5. **Categorize and count failures by type.** A pattern appearing in 5+ patients across multiple batches is systematic and worth targeting. A pattern in 1-2 patients is noise — ignore it.

6. **(Rare) Open `audit_logs/` ONLY if a deep-dive criterion is met.** See the DEEP-DIVE section below. Default behavior is to skip audit logs entirely. Most iterations should not need them.

### Step 3: Design change

Formulate a **falsifiable hypothesis** targeting the most impactful systematic failure:

> "Patients score low on critical actions because the agent orders only the first drug in multi-drug regimens. Adding an explicit 'complete the regimen' checkpoint should improve avg critical_actions_pct by ≥5 points."

Rules:
- **One mechanism per iteration.** If you're tempted to add "and also..." (a different mechanism) — save it for next iteration.
- Every change must trace back to a counted pattern from Step 2.
- Predict expected impact: which dimension, which direction, rough magnitude.
- Match scope to scale: a >20% systematic failure deserves a substantive edit, not a token gesture (see Substantive-edit rules above).
- Consider regression risk: will this change hurt dimensions that are currently strong?

### Step 4: Implement

1. Copy the current best `operating_manual.md` to `candidates/vN/operating_manual.md`
2. Make targeted edits. **Preserve overall structure and section numbering** so diffs are readable.
3. **Self-critique (mandatory):** Re-read the full edited manual and check:
   - Does this change directly address the failure pattern from Step 2?
   - **Is the magnitude of my change proportional to the magnitude of the failure?** A >20% systematic failure should not be addressed by a +4 line edit. A <5% edge-case failure should not justify rewriting half the manual.
   - **Is the change substantive (changes what the agent DOES) — not cosmetic (rewords without changing decisions)?** Apply the substantive-edit test: could a physician acting on the new manual produce a measurably different decision somewhere?
   - Could this change confuse the agent or contradict existing instructions?
   - Would this generalize beyond the batches I analyzed?

### Step 5: Write hypothesis.md

```markdown
## Iteration N

### Failure Pattern
[What recurring error in the traces. Include patient counts and average scores for the affected dimension.]

### Hypothesis
[Falsifiable claim: "If I change X, then dimension Y should improve because Z."]

### Changes Made
[Specific sections modified. Describe the before/after or quote the diff.]

### Expected Impact
[Which dimension(s) should improve, by roughly how much. Which should be unchanged.]

### Regression Risk
[Could this hurt other dimensions? Why or why not?]
```

### Step 6: Append to evolution_summary.jsonl

```json
{"iteration": N, "candidate": "vN", "hypothesis": "...", "changes": "...", "expected_impact": "...", "base": "vM", "status": "pending_eval"}
```

The user will update this line with actual scores after evaluation.

Output: `CANDIDATE: vN — [one-line summary of the change]`

## TRACE READING GUIDE

For any candidate `vN` that has been evaluated, four file types matter for normal use, plus one deep-dive surface (audit_logs).

### `candidates/vN/scores.json` — aggregate + per-batch breakdown

Written by `extract_traces.py`. Read this first to identify the weakest dimension.

```json
{
  "candidate": "v0",
  "batches": [
    {"batch": 1, "scenario": "mcmed_batch_01_12patients",
     "score_pct": 53.3, "diagnosis_pct": 81.7,
     "critical_actions_pct": 46.7, "timeliness_pct": 50.0,
     "disposition_pct": 90.0, "safety_violations": 7}
  ],
  "aggregate": {
    "avg_score_pct": 53.3, "avg_diagnosis_pct": 81.7,
    "avg_critical_actions_pct": 46.7, "avg_timeliness_pct": 50.0,
    "avg_disposition_pct": 90.0, "total_safety_violations": 42,
    "n_batches": 6, "n_patients_total": 72
  }
}
```

### `candidates/vN/traces/batch_NN/patients/patient_X.md` — START HERE

A readable per-patient view containing presentation, vitals, scores across all 4 dimensions, diagnosis assessment, missed critical actions, safety violations, agent orders timeline, test results, and the evaluator's improvement feedback. **This is your primary reading surface** — start here for any patient.

### `candidates/vN/traces/batch_NN/evaluations.json` — full raw eval rows

A dict keyed by `patient_id`, containing the full raw eval row from `patient_evaluations` for each of the 12 patients in that batch. Drill in when the markdown view is missing a field you need.

Key fields per patient:
- `overall_score`, `diagnosis_score`, `critical_actions_score`, `timeliness_score`, `disposition_score` (all 1–5)
- `strengths`, `improvements` (text arrays — preserve strengths, target improvements)
- `evaluation_analysis.diagnosis_assessment.{user_diagnosis, ground_truth, match_quality}`
- `evaluation_analysis.critical_actions_assessment.{required_actions, completed_actions, missed_actions, completion_rate}`
- `evaluation_analysis.safety_assessment.{violations_detected, violations, severity}`
- `evaluation_analysis.test_utilization.{tier_1_indicated, tier_2_adjuncts, tier_3_noise}`
- `clinical_guideline.{standard_approach, clinical_pearl, common_pitfall}`

### `candidates/vN/traces/batch_NN/turn_records.jsonl` — exact turn timing

One JSON object per line, one line per turn (~70 turns per batch). Contains patient state snapshots, every order placed (filtered by `source == "api"` to avoid module echoes), test results that returned, and resource availability. **Drill in here when you need exact turn timing** — e.g. "what turn did the agent first order the missing critical action?" or "did the agent batch orders or run serially?" The file is ~3 MB per batch.

### `candidates/vN/traces/batch_NN/session_summary.json` — session-level totals

Total turns used, total LLM calls, duration, per-patient first/last turn seen. Useful for confirming the agent didn't run out of turns or fail to disposition someone.

### Useful queries

```bash
# 10 lowest-scoring patients in v0
for f in candidates/v0/traces/batch_*/evaluations.json; do
  jq -r --arg b "$f" 'to_entries | .[] | "\(.value.overall_score)/5 \($b) \(.key)"' "$f"
done | sort -n | head -10

# Count missed critical actions across all 72 patients in v0
jq -r '.[] | .evaluation_analysis.critical_actions_assessment.missed_actions[]?' \
  candidates/v0/traces/batch_*/evaluations.json | sort | uniq -c | sort -rn

# All patients with safety violations in v0
grep -l "Safety violations" candidates/v0/traces/batch_*/patients/*.md

# Search for a clinical pattern across all patients
grep -rln "sepsis" candidates/v0/traces/batch_*/patients/
```

## DEEP-DIVE: `candidates/vN/traces/batch_NN/audit_logs/` (rare; large)

This subdirectory contains the full CES audit log for the session. It includes the agent's complete LLM call traces — every prompt the agent received, every reasoning chain it produced, every tool call. **It is large (~74 MB per batch, ~444 MB across the 6 search batches) and should NOT be opened by default.**

**The default behavior is to ignore audit_logs/ entirely.** patient_X.md + evaluations.json + turn_records.jsonl are sufficient for >90% of iterations. They tell you what the agent did, what it should have done, and what was missed. That is enough to identify failure patterns and prescribe manual changes.

### When to open `audit_logs/` (the only valid reasons)

Open audit log files ONLY if all of the following are true:

1. **You have already analyzed scores, evaluations.json, patient_X.md, and turn_records.jsonl** for the patient in question.
2. **You have a specific question** that the data above cannot answer. Valid examples:
   - "The manual already explicitly says X. Why is the agent ignoring rule X across multiple patients? Did the agent read it as constraining or just descriptive?"
   - "Two earlier iterations added rules to address pattern P, both regressed. Was the agent obeying the rule but reaching the wrong conclusion, or ignoring the rule entirely?"
   - "I'm about to add a new rule but I'm not sure if the agent will interpret it the way I intend. Did past similar phrasing produce the intended behavior?"
3. **The pattern shows up in ≥3 patients.** A single odd reasoning trace is not a pattern. Do not write a manual rule based on one trace.

### How to open them (efficiently)

NEVER `cat` an entire turn file or open the whole audit_logs/turns/ directory. Each turn JSON is ~1 MB and contains 80% redundant state snapshots. Instead:

```bash
# Find the turn that matters first (use turn_records.jsonl or patient_X.md to identify it)
# Then read ONLY the llm_calls section of THAT turn:

jq '.llm_calls[] | {role, content: (.messages[-1].content // .response // .text)[0:500]}' \
  candidates/v0/traces/batch_03/audit_logs/turns/turn_0023.json

# Or get a list of just the turn files and their sizes, to pick which one to open:
ls -la candidates/v0/traces/batch_03/audit_logs/turns/ | head

# Or filter llm_calls across multiple turns for a patient:
for f in candidates/v0/traces/batch_03/audit_logs/turns/turn_*.json; do
  jq --arg pid "patient_5" \
    'select(.active_patients[]?.patient_id == $pid) | .turn // .timestamp' "$f" 2>/dev/null
done | head -20
```

### Constraints when consulting audit_logs/

- **Read at most 3 turn files per patient** — each is ~1 MB.
- **Read at most 5 patients' worth of audit data per iteration** — the goal is to confirm a pattern, not to read everything.
- **Anti-overfitting still applies, harder.** Do NOT write a manual rule from a single reasoning trace, no matter how compelling. The rule must be supported by the SAME reasoning failure across ≥3 patients in ≥2 batches.
- **If you find yourself reading audit_logs/ before reading patient_X.md files**, you are doing it wrong. Go back to step 2 of the workflow.

## evolution_summary.jsonl FORMAT

One JSON object per line, one per candidate:

```json
{"iteration": 0, "candidate": "v0", "hypothesis": "baseline", "changes": "none", "avg_score_pct": 53.3, "avg_diagnosis_pct": 81.7, "avg_critical_actions_pct": 46.7, "avg_timeliness_pct": 50.0, "avg_disposition_pct": 90.0, "safety_violations": 42, "status": "evaluated"}
```
