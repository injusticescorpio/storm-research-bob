# Storm Research — Bob Skill

A Bob skill that runs the **STORM method** on any topic and delivers a verified, multi-perspective HTML briefing. Based on Stanford's [STORM system](https://storm.genie.stanford.edu) (NAACL 2024), adapted into a fully automated, citation-verified research pipeline.

---

## What it does

Give it a topic. It spawns five expert personas in parallel (Practitioner, Academic, Skeptic, Economist, Historian), maps where they contradict each other, synthesizes a structured HTML report, then adversarially peer-reviews and verifies every citation against its primary source before delivering.

Output: a single self-contained HTML briefing file with confidence scores, a contradiction map, actionable insights, and a truthful verification banner.

---

## Installation

1. Copy the `skills/storm-research-bob/` folder into your Bob skills directory (typically `~/.agents/skills/` or your workspace `.bob/skills/`).
2. Ensure the [`web-search` MCP server](https://github.com/mrkrsl/web-search-mcp) is running — the skill requires real web fetch capability.
   - If it is not running, the skill will clone, build, and register it automatically, then prompt you to restart Bob once.
3. Restart Bob. The skill is immediately available.

---

## Usage

Trigger phrases (any of these work):

```
storm research this: <topic>
storm report on <topic>
give me a STORM briefing on <topic>
run storm research on <topic>
use the storm-research skill on <topic>
```

Bob infers your role from context and targets the actionable section accordingly. You can also state it explicitly:

```
storm report on electric vehicles — I'm a fleet manager
```

---

## Pipeline

The skill runs four phases end-to-end without shortcuts.

| Phase | What happens |
|-------|-------------|
| **0 — Scope** | Interprets the topic, identifies your role, derives a filename slug |
| **1 — Five lenses** | Spawns 5 parallel sub-agents (Practitioner · Academic · Skeptic · Economist · Historian), each doing real web research |
| **2 — Contradiction map** | Inline analysis: direct conflicts, evidence ranking, universal agreements, and the blind spot no lens covered |
| **3 — HTML report** | Synthesizes everything into a structured HTML file using the bundled template |
| **4 — Peer review + verification** | Self-scores findings 1–10, then spawns parallel citation-verification sub-agents; applies corrections before delivery |

Total sub-agents per run: ~9–11.

---

## Output

A file written to `storm-reports/{topic-slug}-briefing.html` relative to the current working directory, containing:

- **60-second executive summary** — nuance-first, not headline-first
- **5 key findings** ranked by reliability with confidence scores and Supported-by / Challenged-by chips
- **Hidden connection** — the non-obvious link that only surfaces across all five lenses
- **Missing 6th lens** — the blind spot the whole panel missed
- **Actionable insight** — 3–6 specific moves for your role
- **Claim safety guide** — what is safe to assert, what needs caveats, what to avoid
- **Frontier question** — the one question that would change the entire understanding
- **References** — every citation with a verification-status tag (CONFIRMED / PARTIALLY CONFIRMED / UNVERIFIED / FALSE)
- **Verification banner** — `N/N checked, X fabricated, Y corrected, Z demoted`

---

## Files

```
skills/storm-research-bob/
├── SKILL.md              # Skill definition and full pipeline instructions
└── storm-template.html   # Mandatory HTML output template (do not modify CSS)
```

---

## Guardrails

- **Real research only.** Every claim must trace to a fetched source. No invented URLs or figures.
- **Verification is mandatory.** A report without Phase 4 is not a Storm Research report.
- **The panel is author-built.** Convergence across lenses is a strong hypothesis, not independent field consensus. The report discloses this.
- **Reliability scoring** follows a strict hierarchy: peer-reviewed causal > official data > single commissioned survey > analogy > preprint.

---

## Background

The original STORM paper (Shao et al., NAACL 2024) showed that multi-perspective question-asking produced articles **25% more organized** and **10% broader** in coverage than single-prompt research. This skill extends that idea with mandatory citation verification and adversarial self-review — addressing the self-critique gap the Stanford authors flagged in their own system.

Live Stanford demo: [storm.genie.stanford.edu](https://storm.genie.stanford.edu)  
Original code: [github.com/stanford-oval/storm](https://github.com/stanford-oval/storm)
