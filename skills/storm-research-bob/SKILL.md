---
name: storm-research-bob
description: Autonomous AI coding with spec-driven development for IBM Bob Shell. Implements Geoffrey Huntley's iterative bash loop methodology where Bob works through specs one at a time, outputting a completion signal only when acceptance criteria are 100% met.
argument-hint: "[topic to research]"
license: MIT
metadata:
  version: "4.0"
  repository: https://github.com/injusticescorpio/storm-research-bob
---

# Storm Research

## What this does

Turns one topic into a verified, multi-perspective HTML briefing. It simulates five expert lenses on the topic, maps where they contradict each other, synthesizes everything into a single self-contained HTML report, then adversarially peer-reviews its own output and verifies every citation against its primary source before delivering. The output is one HTML file with no blind spots and no unchecked claims.

Run the full pipeline end to end. Do not shortcut a phase. This is heavier than a quick web lookup; that is the point.

## Portability

This skill is self-contained. It depends only on `curl` (available in all sub-agent shells), file-write, and sub-agent capabilities, plus `storm-template.html` in this same skill folder. No external scripts, APIs, or paid services are required.

## Phase 1: Scope the topic

1. If `$ARGUMENTS` has the topic, use it. Otherwise ask what to research.
2. State your interpretation of the topic in one line and proceed. Only ask a clarifying question if the topic is genuinely ambiguous in a way that changes the research. Default to proceeding.
3. Identify the **reader's role** so the actionable section can target it. Infer it from the topic and any stated context; if unclear, ask in one line, or default to "a practitioner or decision-maker in this field."
4. Derive a kebab-case `topic-slug` from the topic for the filename.
5. Tell the user the pipeline is running (5 lenses, then verify). One line.

## Phase 2: Five expert lenses (parallel sub-agents)

Spawn **five sub-agents in a single batch** so they run concurrently. Each sub-agent **must use `curl` to fetch real web pages** — MCP tools are not available inside sub-agents. Instruct each to use `curl -sL "<url>"` to retrieve page content, and to discover URLs first via a DuckDuckGo search like `curl -sL "https://html.duckduckgo.com/html/?q=<query>"` to extract links from the HTML. Each sub-agent must return only claims backed by URLs it actually fetched during that run. Each gets the SAME topic framing plus its own lens. Use these exact prompts, substituting `{TOPIC}` and a one-line `{TOPIC_FRAME}` (your Phase 1 interpretation):

**1. THE PRACTITIONER** — `You are THE PRACTITIONER for: {TOPIC} ({TOPIC_FRAME}). You work with this daily. Use curl to do real web research: search DuckDuckGo with curl -sL "https://html.duckduckgo.com/html/?q=<query>" to find URLs, then fetch pages with curl -sL "<url>" to extract content. Prioritize recent sources, case studies, practitioner threads, operator data. Surface the GAP between what hands-on operators know and what academics/pundits miss, and the practical realities (workflow friction, what actually works, where it breaks) that get ignored. Return EXACTLY: 1) CORE POSITION in 2 sentences. 2) STRONGEST EVIDENCE, 3-5 bullets each with a concrete data point/case/named source + URL you actually fetched. 3) THE ONE THING only a practitioner would say. Under 400 words.`

**2. THE ACADEMIC** — `You are THE ACADEMIC for: {TOPIC} ({TOPIC_FRAME}). You care about peer-reviewed evidence and effect sizes, not anecdotes. Use curl to do real web research: search DuckDuckGo with curl -sL "https://html.duckduckgo.com/html/?q=<query>" to find URLs, then fetch pages with curl -sL "<url>". Target peer-reviewed studies, arXiv, university reports, journals. Answer: what does the rigorous evidence ACTUALLY say vs popular belief, and where does it CONTRADICT the hype. Return EXACTLY: 1) CORE POSITION in 2 sentences. 2) STRONGEST EVIDENCE, 3-5 bullets each tied to a named study/report + URL you actually fetched with the actual finding/effect size. 3) THE ONE THING only an academic would say. Flag where evidence is thin or contested, and note peer-review status (published vs preprint). Under 400 words.`

**3. THE SKEPTIC** — `You are THE SKEPTIC for: {TOPIC} ({TOPIC_FRAME}). You think the mainstream view is overstated or wrong. Build the STRONGEST steelman bear case. Use curl to do real web research: search DuckDuckGo with curl -sL "https://html.duckduckgo.com/html/?q=<query>" for backlash, failures, contradicting data, policy/regulatory changes, debunkings, then fetch pages with curl -sL "<url>". Answer: the strongest counterargument, and what proponents conveniently ignore. Return EXACTLY: 1) CORE POSITION in 2 sentences. 2) STRONGEST EVIDENCE, 3-5 bullets each with a concrete source + URL you actually fetched. 3) THE ONE THING only a skeptic would say. Be rigorous, not contrarian for sport. Under 400 words.`

**4. THE ECONOMIST** — `You are THE ECONOMIST for: {TOPIC} ({TOPIC_FRAME}). You follow the money. Use curl to do real web research: search DuckDuckGo with curl -sL "https://html.duckduckgo.com/html/?q=<query>" for revenues, valuations, market size, funding flows, unit economics, then fetch pages with curl -sL "<url>". Answer: who profits from the current narrative, and what financial incentives shape the research and hype. Return EXACTLY: 1) CORE POSITION in 2 sentences. 2) STRONGEST EVIDENCE, 3-5 bullets each with a real number (revenue/valuation/market size/funding) + named source + URL you actually fetched. 3) THE ONE THING only an economist would say (the follow-the-money insight). Under 400 words.`

**5. THE HISTORIAN** — `You are THE HISTORIAN for: {TOPIC} ({TOPIC_FRAME}). You have seen disruption cycles before and look for patterns. Use curl to do real web research: search DuckDuckGo with curl -sL "https://html.duckduckgo.com/html/?q=<query>" for genuine historical parallels, prior technologies, manias, market shifts, then fetch pages with curl -sL "<url>". Answer: what parallels actually fit, and what we learn from how they played out (who won, who lost, what stabilized). Return EXACTLY: 1) CORE POSITION in 2 sentences. 2) STRONGEST EVIDENCE, 3-5 bullets each a specific historical case with dates/outcomes + a URL you actually fetched. 3) THE ONE THING only a historian would say (the pattern no one else surfaces). Under 400 words.`

When all five sub-agents return, post a 2-3 line note in chat: which way they converge, and the sharpest disagreement. Keep the raw briefs out of chat.

## Phase 3: Map the contradictions

Working only from the five briefs, determine (do this inline, no sub-agents needed):

1. **Direct conflicts** — where two or more lenses claim opposite things. Name the specific clashing claims, not just topics.
2. **Strongest vs weakest evidence** — which lens is best-supported (rank: peer-reviewed causal > official data > anecdote/analogy) and which is weakest, with why.
3. **The resolving question** — the single empirical question that would settle the biggest contradiction.
4. **Universal agreement** — what every lens confirms, even opponents. This is the likely-true load-bearing finding.
5. **The blind spot** — what NO lens addressed. This becomes the "missing 6th lens" and feeds the Frontier Question.

This map is not a separate deliverable. It is the raw material for the report's findings (supports/challenges), hidden connection, 6th-lens box, and frontier question.

## Phase 4: Synthesize the HTML report

1. Read `storm-template.html` in this skill folder. Clone it; do not rebuild the CSS.
2. Fill every section. Mapping from the phases:
   - **60-second summary** — decision-maker-grade, nuance not headline. Lead with the settled fact, then the contested interpretation.
   - **5 key findings, ranked by reliability** — most important things now known, highest reliability first. Each carries a 1-10 confidence score (set in Phase 5) and Supported-by / Challenged-by chips drawn from the contradiction map.
   - **Hidden connection** — the non-obvious link from Phase 3 that only appears across all five lenses.
   - **Key assumption / missing 6th lens** — the blind spot from Phase 3, framed as the lens that could change the conclusions.
   - **Actionable insight** — 3-6 specific moves for the reader's role identified in Phase 1. Specific, not abstract.
   - **Claim safety guide** — assert / caveat / avoid, populated after Phase 5 verification.
   - **Frontier question** — the one question that would change everything.
   - **References** — every citation with a verification-status tag (set in Phase 5).
3. Write to `storm-reports/{topic-slug}-briefing.html` (relative to the current working directory; create the folder if needed).

## Phase 5: Adversarial peer review + verification (do not skip)

This is what separates Storm Research from a normal report. Run it before delivering.

**5a. Self-review (inline).** Score each of the 5 findings 1-10 for reliability and justify. Identify the weakest link and what would verify it. Run a bias check (which lens dominated the synthesis, what got underweighted). Name the missing 6th perspective. Assign an honest overall grade.

**5b. Verify every citation (parallel sub-agents).** Spawn one sub-agent per distinct citation cluster in a single batch (group related claims; ~4-6 sub-agents). Each verifier sub-agent **must use `curl`** to hit the actual source URL and distrust any uncited memory. Each sub-agent prompt:

`Independently verify a citation against its PRIMARY source. You MUST use curl to fetch the actual source: curl -sL "<url>". Do not rely on memory or unverified prior summaries. Be skeptical; do not trust secondary blog summaries. CLAIM: {claim + cited figure + named source + URL}. Fetch the URL with curl, read the content, and confirm or correct: exact title/authors/venue/year/URL, the real figure or effect size as published, sample/method and any author-stated limits, and peer-review status (published vs preprint). For any contested claim, search DuckDuckGo with curl -sL "https://html.duckduckgo.com/html/?q=<query>" to find the strongest credible counter-source and fetch it. Return: VERDICT = CONFIRMED / PARTIALLY CONFIRMED (list corrections) / UNVERIFIED / FALSE, then the corrected one-line citation, then 2-4 bullets of specifics with the primary URL. Under 280 words.`

**5c. Apply corrections.** Edit the report:
- Fix any wrong figures, titles, dates, or mischaracterizations.
- Downgrade confidence scores where evidence turned out thin; demote preprints and contested claims into the "Contested signal" sidebar.
- Re-attribute single-survey or commissioned stats honestly.
- Fill the verification banner (`X fabricated, Y corrected, Z demoted`) and the per-citation status tags.
- Populate the claim safety guide from the verdicts.

## Output

1. Final deliverable: `storm-reports/{topic-slug}-briefing.html` (the v2, post-verification version).
2. Give the user the file path and open it if the platform supports it.
3. In chat, give: the file path, the verification tally (`N/N checked, X fabricated, Y corrected, Z demoted`), the one universal finding, the frontier question, and the claim safety summary (what is safe to assert vs avoid). Keep it tight.

## Notes & guardrails

- **Real research only.** Every lens and every citation must trace to a real, fetched source. Sub-agents must use `curl` to fetch pages during that run. No invented studies, numbers, or URLs. If a figure can't be verified, demote or cut it; never paper over it.
- **Sub-agents use curl, not MCP tools.** MCP tools (web-search, etc.) are not available inside sub-agents. All sub-agents must fetch content via `curl -sL "<url>"` and discover URLs via DuckDuckGo HTML search (`curl -sL "https://html.duckduckgo.com/html/?q=<query>"`). Do not instruct sub-agents to call MCP tools.
- **The panel is author-built.** Always disclose this in the report. Agreement across lenses is a strong hypothesis, not independent proof. Do not present convergence as consensus of the field.
- **Verification is mandatory.** A report delivered without Phase 5 is not a Storm Research report. The verification banner must be truthful.
- **Reliability = evidence quality, not confidence.** Score on the source hierarchy: peer-reviewed causal > official policy/financial data > single commissioned survey > analogy > preprint.
- **Target the reader, not a default person.** The actionable insight and claim safety guide speak to the role identified in Phase 1. Keep them generic if no role is given.
- **Cost.** This spawns ~9-11 sub-agents per run. That is expected. Do not fan out wider than five lenses or one verifier per citation cluster.
- **Design.** Clean white and professional (Montserrat / Roboto Mono, blue accent). Keep the template CSS verbatim. Do not swap in a different visual style.
