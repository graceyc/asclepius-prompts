# Asclepius Agent Prompts

This repository contains the agent prompts used in the experimental configurations evaluated in the paper.

## Repository Structure

```
.
├── chiron_baseline/                        # Configuration 1: Baseline (Chiron)
│   └── main_agent_prompt.md
├── chiron_skills/                          # Configuration 2: + Skills
│   ├── main_agent_prompt.md
│   └── skills.md
├── chiron_acting_subagents/                # Configuration 3: + Acting Subagents
│   ├── main_agent_prompt.md
│   └── acting_subagent_definitions.md
├── chiron_harness/                         # Configuration 4: + Harness
│   ├── main_agent_prompt.md
│   └── proposer_prompt.md
├── chiron_harness_skills_acting_subagents/ # Configuration 5: Full Asclepius (Acting)
│   ├── main_agent_prompt.md
│   ├── skills.md
│   └── acting_subagent_definitions.md
└── chiron_harness_skills_advisory_subagents/ # Configuration 6: Full Asclepius (Advisory)
    ├── main_agent_prompt.md
    ├── skills.md
    └── advisory_subagent_definitions.md
```

## Configuration Descriptions

| # | Configuration | Components | Description |
|---|---|---|---|
| 1 | **Baseline (Chiron)** | Main agent only | Single-agent baseline with no structured scaffolding |
| 2 | **+ Skills** | Main agent + Skills library | Adds a retrieval-augmented skills library with clinical regimens |
| 3 | **+ Acting Subagents** | Main agent + Acting subagents | Adds three acting subagents (triage-prioritizer, diagnostician, treatment-checker) that execute clinical actions directly |
| 4 | **+ Harness** | Main agent (with harness) | Replaces the baseline main prompt with a structured operating manual that enforces workflow rules |
| 5 | **Full Asclepius (Acting)** | Harness + Skills + Acting subagents | Combines the harness, skills library, and acting subagents that execute actions autonomously |
| 6 | **Full Asclepius (Advisory)** | Harness + Skills + Advisory subagents | Combines the harness, skills library, and advisory subagents that provide text-only recommendations (the main agent executes all actions) |

## File Descriptions

- **`main_agent_prompt.md`** — The system prompt for the main orchestrating agent. In configurations 1–3, this is the baseline Chiron prompt. In configuration 4, it is the structured operating manual (harness). In configurations 5–6, it is the harness prompt adapted for multi-agent coordination with either acting or advisory subagents.
- **`skills.md`** — The clinical skills library containing structured treatment regimens that the agent can retrieve during patient management.
- **`acting_subagent_definitions.md`** — Definitions and system prompts for the three acting subagents (triage-prioritizer, diagnostician, treatment-checker) that have MCP access and execute clinical actions directly.
- **`advisory_subagent_definitions.md`** — Definitions and system prompts for the three advisory subagents (triage-prioritizer, diagnostician, treatment-checker) that are text-only and return recommendations to the main agent, which executes all actions itself.
- **`proposer_prompt.md`** — The meta-harness prompt given to the proposer agent that iteratively evolves the operating manual. The proposer analyzes evaluation traces, identifies systematic failure patterns, and produces improved operating manual candidates. Found in `chiron_harness/` since the harness is the output of this process.
