# Asclepius Agent Prompts

This repository contains the agent prompts used in the five experimental configurations evaluated in the paper.

## Repository Structure

```
.
├── chiron_baseline/              # Configuration 1: Baseline (Chiron)
│   └── main_agent_prompt.md
├── chiron_skills/                # Configuration 2: + Skills
│   ├── main_agent_prompt.md
│   └── skills.md
├── chiron_acting_subagents/      # Configuration 3: + Acting Subagents
│   ├── main_agent_prompt.md
│   └── acting_subagent_definitions.md
├── chiron_harness/               # Configuration 4: + Harness
│   ├── main_agent_prompt.md
│   └── proposer_prompt.md
└── chiron_harness_skills_subagents/  # Configuration 5: Full Asclepius
    ├── main_agent_prompt.md
    ├── skills.md
    └── acting_subagent_definitions.md
```

## Configuration Descriptions

| # | Configuration | Components | Description |
|---|---|---|---|
| 1 | **Baseline (Chiron)** | Main agent only | Single-agent baseline with no structured scaffolding |
| 2 | **+ Skills** | Main agent + Skills library | Adds a retrieval-augmented skills library with clinical regimens |
| 3 | **+ Acting Subagents** | Main agent + Acting subagents | Adds three acting subagents (triage-prioritizer, diagnostician, treatment-checker) that execute clinical actions |
| 4 | **+ Harness** | Main agent (with harness) | Replaces the baseline main prompt with a structured operating manual that enforces workflow rules |
| 5 | **Full Asclepius** | All components | Combines the harness, skills library, and acting subagents |

## File Descriptions

- **`main_agent_prompt.md`** — The system prompt for the main orchestrating agent. In configurations 1–3, this is the baseline Chiron prompt. In configuration 4, it is the structured operating manual (harness). In configuration 5, it is the harness prompt adapted for multi-agent coordination.
- **`skills.md`** — The clinical skills library containing structured treatment regimens that the agent can retrieve during patient management.
- **`acting_subagent_definitions.md`** — Definitions and system prompts for the three acting subagents: triage-prioritizer, diagnostician, and treatment-checker.
- **`proposer_prompt.md`** — The meta-harness prompt given to the proposer agent that iteratively evolves the operating manual. The proposer analyzes evaluation traces, identifies systematic failure patterns, and produces improved operating manual candidates. Found in `chiron_harness/` since the harness is the output of this process.
