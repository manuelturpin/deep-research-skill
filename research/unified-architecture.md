# Unified Architecture: LLM-Agnostic Multi-Level Deep Research Skill

Version: 1.0 | Synthesized: 2026-02-12
Sources: Perplexity extraction, Claude extraction, Gemini extraction

---

## 1. Meta-Prompting Pipeline

### Design Rationale

The meta-prompting pipeline is the core engine of the skill. It does NOT answer the user's question --- it generates an optimized research prompt that will be sent to an LLM for execution. This "task vs answer" separation is a foundational design principle: the skill produces prompts that instruct the LLM to **research and plan**, never to answer directly.

Three independent research threads converge on the same insight: LLMs are better prompt engineers than humans (APE: LLM outperforms humans on 24/24 tasks, IQM 0.810 vs 0.749), and prompt quality explains more performance variance than model choice (Anthropic: token usage explains 80% of research quality variance). The meta-prompting pipeline sits at the highest-leverage point in the entire research stack.

### Unified 7-Stage Pipeline

The pipeline merges Perplexity's 7-step sequential flow with Claude's 3-stage DETECT/DECOMPOSE/GENERATE framework and Gemini's complexity-adaptive architecture. The result is a 7-stage pipeline grouped into 3 phases.

#### PHASE A --- DETECT (Stages 1-3)

**Stage 1: Raw Subject Ingestion**
- Input: free-text query from user
- Optional inputs: preferred language, time horizon, desired depth level (otherwise auto-calculated), target audience
- The skill captures the raw query verbatim --- no transformation yet

**Stage 2: Subject Analysis**
- Intent classification: informational, decisional, comparative, critical, systematic, exploratory
- Complexity estimation across 4 axes (see Section 2: Automatic Depth Calibration)
- Implicit constraint detection: temporal markers ("2024-2025"), rigor markers ("academic studies", "peer-reviewed"), speed markers ("quick", "brief"), scope markers ("comprehensive", "exhaustive")
- Entity count, comparison operators ("vs", "compare"), question type (factual vs analytical vs evaluative)
- Maps to Bloom's taxonomy levels and Webb's DOK (see Section 2)

**Stage 3: Depth Level Determination**
- Apply the 4-axis complexity matrix (Section 2) to produce a composite score
- Map score to one of 4 depth levels (Simple, Advanced, Ultra, Super Ultra)
- If user explicitly chose a level, respect it but allow upward auto-escalation with warning if query is manifestly more complex
- Never auto-downgrade a user-selected level

#### PHASE B --- DECOMPOSE (Stages 4-5)

**Stage 4: Hierarchical Sub-question Decomposition**
- Level 1: No decomposition needed, or 1-3 clarifying sub-questions max
- Level 2: 3-5 sub-questions organized by thematic axes (conceptual, empirical, comparative)
- Level 3: STORM-style perspective discovery --- identify 4-6 distinct expert perspectives, generate 2-4 specific questions per perspective, include a contrarian perspective. Hierarchical outline with H2/H3 structure.
- Level 4: Full STORM perspective discovery + simulated expert conversations. Multi-phase research plan (reconnaissance, deep investigation, synthesis, verification, gap analysis). 8-12 investigation areas, each with 3 priority questions.
- At levels 2-4: explicit dependency notation between sub-questions ("answer B after exploring A") creating a directed acyclic graph of research activities, not a flat list
- Fan-out management: the decomposition must balance breadth of exploration against focus, using task decomposition and meta-planning to control conceptual fan-out (Gemini insight)

**Stage 5: Source Criteria & Verification Protocol Specification**
- Source types: academic papers, official documentation, expert blogs, GitHub repos, technical reports, primary data --- scaled by level
- Quality: priority to peer-reviewed, official docs, recognized conferences
- Freshness: explicit temporal filters ("prioritize 2024-2025, include older if foundational")
- Diversity: require divergent viewpoints; mandate at least N contrarian sources (scaled by level)
- Verification protocol: scaled from simple fact-checking (Level 1) to full CoVe + self-consistency + steelmanning (Level 4) --- see Section 5
- Cross-validation: "For each important fact, find at least N independent sources; flag facts confirmed by only one source"

#### PHASE C --- GENERATE (Stages 6-7)

**Stage 6: Research Output Format Definition**
- Level-dependent structure (see Section 3 for exact specifications per level)
- Always explicit: named sections, approximate word count, expected tables, citation format
- Dynamic format capability: Markdown for human consumption, JSON/YAML for automated pipelines
- Ultra/Super Ultra: professional report structures with dedicated sections for hypotheses, unknowns, and trade-offs

**Stage 7: Internal Meta-prompting Assembly (Generate-Critique-Revise)**
- This is the critical self-reasoning step, architecturally baked into the pipeline (not optional)
- Sub-steps:
  - 7a: Reformulate the user's objective in precise research terms
  - 7b: Review the decomposition for completeness, redundancy, and dependency correctness
  - 7c: Verify source and validation criteria match the depth level
  - 7d: Assemble the complete prompt from the 5 essential components (see below)
  - 7e: Self-critique the assembled prompt (check: clarity, completeness, specificity, level-appropriateness, anti-hallucination coverage)
  - 7f: Self-revise based on critique findings
  - 7g: Wrap in model-specific adapter layer (see Section 4)
- The critique dimensions (clarity, completeness, specificity) are drawn from APO/MAPO research on textual gradient feedback

### The 5 Essential Components of Every Generated Prompt

Every research prompt produced by the pipeline MUST contain these 5 components, in order. This merges Perplexity's 7-section template with Claude's 5-component architecture and Gemini's 5 structural elements.

| # | Component | Purpose | Scaling Dimension |
|---|-----------|---------|-------------------|
| 1 | **Role and Context Frame** | Define persona (domain expert, research analyst), provide topic context, set audience and tone. Anthropic data: Claude matches tone/register of the prompt itself. | Persona complexity scales with level: "concise research assistant" (L1) to "principal research scientist leading a research team" (L4) |
| 2 | **Task Definition with Anti-Confusion Guard** | Explicitly state: "Your task is to research and synthesize, NOT to directly answer." Include intent and depth level description. | Task scope and methodology requirements scale with level |
| 3 | **Decomposition Scaffold** | Numbered sub-questions grouped by axes, with dependencies noted. Uses Self-Ask pattern (L1-2) or STORM perspective discovery (L3-4). Multi-Perspective Simulation: identify 4-5 distinct perspectives to avoid simplistic dichotomies. | Number and depth of sub-questions scale with level |
| 4 | **Source Criteria and Verification Protocol** | Acceptable source types, recency, cross-validation instructions, CoVe steps, confidence ratings. Disagreement synthesis blocks mandatory at all levels. | Source count, type requirements, and verification rigor scale with level |
| 5 | **Output Format and Constraint Boundaries** | Exact structure, length, citation format, temporal bounds, topical limits, anti-hallucination instructions. Include current date. Specify behavior under uncertainty: "flag rather than guess." | Structure complexity and word count scale with level |

### Data Flow Diagram

```
User Query
    |
    v
[Stage 1: Ingestion] --- capture raw query + optional params
    |
    v
[Stage 2: Analysis] --- intent + complexity + implicit constraints
    |
    v
[Stage 3: Level Determination] --- 4-axis matrix -> level assignment (+ auto-escalation logic)
    |
    v
[Stage 4: Decomposition] --- sub-questions, perspectives, dependencies (DAG)
    |
    v
[Stage 5: Source & Verification Criteria] --- types, freshness, diversity, CoVe protocol
    |
    v
[Stage 6: Output Format] --- structure, length, citation format per level
    |
    v
[Stage 7: Meta-prompting Assembly]
    |--- 7a: Reformulate objective
    |--- 7b: Review decomposition
    |--- 7c: Verify criteria
    |--- 7d: Assemble 5 components
    |--- 7e: Self-critique
    |--- 7f: Self-revise
    |--- 7g: Apply model adapter
    |
    v
Final Research Prompt (ready for any LLM)
```

---

## 2. Automatic Depth Calibration

### Design Rationale

No consumer tool currently offers automatic granular depth calibration --- users are either forced to choose a mode (Perplexity) or get a one-size-fits-all response. This is a key competitive advantage. The system should analyze the query and auto-select the appropriate depth, while allowing user override with bidirectional escalation awareness.

### The 4-Axis Complexity Matrix

Merges Claude's 4-dimension classifier (breadth, depth, ambiguity, temporality), Gemini's ResearchRubrics-based matrix (conceptual breadth, logical nesting, exploration, output objective), and Perplexity's 5-dimension criteria.

| Axis | What It Measures | Scoring (1-4) | Signal Sources |
|------|-----------------|----------------|----------------|
| **Breadth** | Number of distinct sub-topics/domains | 1: single domain, 2: 2-5 related, 3: >5 disjoint, 4: transdisciplinary | Entity count, topic classification, presence of "and/or/vs" |
| **Depth** | Logical nesting and reasoning hops required | 1: one-step inference, 2: 2-3 dependent steps, 3: 4+ hierarchical steps, 4: recursive planning | Question type (factual/analytical/evaluative), causal chains, "why/how" depth |
| **Ambiguity** | Open-endedness and specification level | 1: fully specified, 2: moderately open, 3: exploratory/ambiguous, 4: highly under-specified | Presence/absence of concrete constraints, number of possible interpretations |
| **Temporality** | Time-sensitivity of information | 1: stable/historical, 2: slowly evolving, 3: actively changing, 4: real-time/breaking | Temporal qualifiers ("recent", "evolution", "2024"), domain volatility |

**Composite Score Calculation:**
- Sum of 4 axes = 4-16
- Level 1 (Simple): 4-6
- Level 2 (Advanced): 7-9
- Level 3 (Ultra): 10-12
- Level 4 (Super Ultra): 13-16

### Signal Types for Complexity Detection

Three categories of signals feed the matrix scoring:

**1. Structural Signals (from query text itself)**
- Entity count: <2 entities = low breadth, 4+ = high
- Comparison operators: "vs", "compare", "difference between" = higher breadth
- Temporal qualifiers: "evolution", "recent", "trend" = higher temporality
- Scope indicators: "comprehensive", "exhaustive" = higher breadth+depth; "brief", "quick" = lower
- Question type mapping to Bloom's taxonomy:
  - "What is" = Remember (L1)
  - "How does / Why" = Understand/Apply (L2)
  - "Compare / Evaluate / Analyze" = Analyze/Evaluate (L3)
  - "Design / Synthesize / Create framework" = Create (L4)
- Implicit constraint detection: mentions of "academic studies", "peer-reviewed" push depth up; "quick tutorial" pushes depth down

**2. Retrieval-based Signals (from initial probe)**
- Number of documents needed (multi-hop indicator)
- Diversity of relevant source types
- Contradiction rate across initial sources --- high contradiction = deeper analysis needed
- Topic obscurity: hallucination risk is inversely proportional to topic popularity (6% fabrication for well-studied vs 28-29% for niche topics) --- niche topics push verification requirements up

**3. LLM-based Assessment (meta-cognitive)**
- Prompt LLM to estimate complexity before generating research prompt
- Count sub-questions during decomposition --- more = higher complexity
- First-token probability and entropy correlate with question difficulty (Zotos et al., 2024-2025)

### Webb's DOK + Bloom's Taxonomy Mapping

| Level | Webb's DOK | Bloom's Taxonomy | Research Equivalent |
|-------|-----------|-----------------|---------------------|
| 1 - Simple | Level 1: Recall & Reproduction | Remember / Understand | Factual lookup, definition, quick synthesis |
| 2 - Advanced | Level 2-3: Skills/Concepts + Strategic Thinking | Apply / Analyze | Structured comparison, mechanism explanation, multi-facet analysis |
| 3 - Ultra | Level 3-4: Strategic + Extended Thinking | Analyze / Evaluate | Critical evaluation, cross-domain synthesis, literature review |
| 4 - Super Ultra | Level 4: Extended Thinking | Evaluate / Create | Systematic review, meta-analysis, novel framework creation |

### Auto-Escalation Protocol

1. If user selected a level AND detected complexity is 2+ levels higher: auto-escalate with explicit warning in generated prompt ("Note: this query's complexity suggests Level N treatment; escalated from user-requested Level M")
2. If user selected a level AND detected complexity is 1 level higher: respect user choice but add advisory note
3. If user selected a level AND detected complexity is lower: respect user choice (never auto-downgrade)
4. If no level selected: auto-assign based on composite score

### Calibration Mechanism

Intent-Based Prompt Calibration (2024) provides a mechanism for the skill to self-calibrate thresholds over time using synthetic boundary cases. Rather than relying on hardcoded score ranges, the system can generate edge-case queries that sit at level boundaries and use them to refine the mapping function. This is an advanced capability for future iterations.

---

## 3. The 4-Level System

### Level 1 --- Simple: Quick Factual Synthesis

**Trigger Criteria:**
- Composite score: 4-6
- Single-entity queries, factual lookups, definitions, "what is" questions
- <2 entities, no comparison operators, no analytical verbs
- Webb's DOK 1 / Bloom's Remember-Understand
- Equivalent to: Perplexity Quick Search, standard ChatGPT response

**Source Requirements:**
- Count: 1-5 sources
- Types: generalist sources, official documentation, clear summaries
- Freshness: include current date, prioritize recent but not strictly filtered
- Diversity: not required beyond basic cross-check

**Decomposition:**
- None needed, or 1-3 implicit clarifying sub-questions
- No hierarchical structure
- Single research pass

**Verification Protocol:**
- "Confirm key facts from at least one additional source. Flag uncertainty."
- "If information is uncertain or conflicting, state this explicitly."
- No formal CoVe protocol
- Confidence rating: not required

**Output Format:**
- Direct answer in 1-3 paragraphs (100-500 words)
- OR: 1 context paragraph + 3-5 "key points" bullet points
- 1-5 inline citations with URLs
- No sections, no tables unless comparing 2-3 items
- Summary of 200-300 words (Gemini matrix)

**Complete Prompt Template:**

```xml
<role>
You are a research assistant. Answer the following question concisely
and accurately using current web sources. Provide 1-5 citations.
</role>

<topic>[USER TOPIC]</topic>

<task>
Provide a direct, factual answer to the question above.
This is a Level 1 (Simple) research task: prioritize accuracy and
conciseness over exhaustiveness.
</task>

<source_criteria>
- Use 1-5 credible sources (official documentation, authoritative references)
- Prioritize recent sources where relevant
- Today's date: [DATE]
</source_criteria>

<output_format>
- Direct answer in 1-3 paragraphs (under 500 words)
- Cite all sources with URLs inline
- If information is uncertain or conflicting, state this explicitly
- Do NOT fabricate citations. If you cannot verify a source exists, omit it.
</output_format>

<verification>
Confirm the answer from at least one additional source. If you find
conflicting information, present both perspectives and flag the
discrepancy.
</verification>
```

---

### Level 2 --- Advanced: Structured Analytical Overview

**Trigger Criteria:**
- Composite score: 7-9
- Multi-facet queries requiring comparison, mechanism explanation, moderate analysis
- 2-4 entities, comparison operators ("vs", "compare"), or "how/why" questions requiring multiple perspectives
- Webb's DOK 2-3 / Bloom's Apply-Analyze
- Equivalent to: Perplexity Pro Search, You.com Research mode

**Source Requirements:**
- Count: 5-15 sources (mixed types)
- Types: expert blogs, official documentation, some recent academic papers, technical reports
- Freshness: prioritize 2024-2025; include foundational older work
- Diversity: require divergent viewpoints; cross-reference key claims across at least 2 independent sources

**Decomposition:**
- 3-5 sub-questions organized by thematic axes (conceptual, empirical, comparative)
- Essentially parallel (minimal dependencies)
- Multiple research queries

**Verification Protocol:**
- "Cross-reference key claims across 2+ sources."
- Rate confidence HIGH / MEDIUM / LOW for each major finding
- "Identify at least 2-3 sources or arguments that contradict or nuance the dominant thesis."
- Flag any information contested or uncertain

**Output Format:**
- Executive summary (100 words, 3-5 sentences)
- 3-5 themed sections with descriptive H2 headers
- Comparison tables where relevant (e.g., 3-4 approaches or tools)
- Key takeaways section
- Total: 1,000-3,000 words
- Inline citations with URLs throughout
- Short bibliography (6-10 sources)

**Complete Prompt Template:**

```xml
<role>
You are an expert [DOMAIN] analyst conducting a structured research
overview. Your analysis should be balanced, well-sourced, and clearly
organized.
</role>

<topic>[USER TOPIC]</topic>

<task>
Conduct a Level 2 (Advanced) structured analytical overview of the
topic above. Your goal is to understand the main axes, variants, and
active debates. Do NOT provide a superficial summary --- investigate
the key dimensions with analytical rigor.
</task>

<research_plan>
Break this topic into 3-5 key sub-questions covering distinct
analytical axes:
1. [Auto-generated: conceptual/definitional axis]
2. [Auto-generated: comparative/alternatives axis]
3. [Auto-generated: practical/implementation axis]
4. [Auto-generated: limitations/risks axis]
5. [Auto-generated: future direction axis, if relevant]
</research_plan>

<source_criteria>
- Use 5-15 diverse sources (academic papers, official documentation,
  expert analysis, technical reports)
- Prioritize sources from [CURRENT_YEAR-1]-[CURRENT_YEAR]
- Cross-reference key claims across at least 2 independent sources
- Include at least 2 sources that present alternative or contrarian views
- Today's date: [DATE]
</source_criteria>

<output_format>
- Executive summary (100 words)
- 3-5 analytical sections with descriptive H2 headers
- Comparison tables where relevant
- Key takeaways section
- Total length: 1,000-3,000 words
- Inline citations with URLs
- Rate confidence HIGH/MEDIUM/LOW for each major finding
</output_format>

<verification>
For each major claim:
1. Verify against a second independent source
2. Flag any information that is contested or uncertain
3. Identify at least 2 arguments that contradict or nuance the
   dominant position, and summarize them fairly
Do NOT fabricate citations. If you cannot verify a source, omit it
and note the gap.
</verification>

Today's date: [DATE].
```

---

### Level 3 --- Ultra: Comprehensive Analytical Report

**Trigger Criteria:**
- Composite score: 10-12
- Complex multi-dimensional queries requiring critical evaluation, cross-domain synthesis, extensive source analysis
- 4+ entities, cross-domain analysis, scope indicators like "comprehensive", "in-depth", "analyze"
- Webb's DOK 3-4 / Bloom's Analyze-Evaluate
- Equivalent to: OpenAI Deep Research, Gemini Deep Research output

**Source Requirements:**
- Count: 15-50 sources (preference for primary and peer-reviewed)
- Types: peer-reviewed papers (last 3-5 years), official documentation, technical reports, expert analyses, primary data
- Freshness: explicit period filter; flag anything older than 2 years for time-sensitive claims
- Diversity: at least 3 sources that challenge the dominant narrative; check for geographic, linguistic, cultural coverage gaps
- Non-English sources where relevant

**Decomposition:**
- STORM-style perspective discovery: identify 4-6 distinct expert perspectives relevant to the topic
- For each perspective: generate 2-3 specific research questions
- Include a "contrarian" perspective that challenges mainstream assumptions
- Hierarchical decomposition: macro-questions -> dimensions -> specific sub-questions
- Explicit dependency notation between sub-questions
- Multi-pass research: initial broad sweep, then targeted deep dives

**Verification Protocol:**
- Full Chain-of-Verification (CoVe) protocol:
  1. Draft response
  2. Generate verification questions for each factual claim
  3. Answer verification questions independently (without referencing draft)
  4. If contradicted, revise or flag as uncertain
- Steelmanning: construct the strongest version of each competing position before evaluating
- Temporal verification: flag time-sensitive claims with source dates
- Confidence ratings: HIGH / MEDIUM / LOW for each key finding
- Citation integrity: "Only cite sources you can verify exist. If you cannot confirm a reference, omit it and note the gap."
- Bias awareness: identify the perspective the analysis takes, present strongest counter-arguments, note underrepresented perspectives

**Output Format:**
- Executive summary (200-300 words)
- Methodology and scope note (how research was conducted)
- 5-8 analytical sections with H2/H3 hierarchy
- Embedded tables for comparative data
- Critical assessment section: gaps, limitations, contradictions
- Conclusion: key findings and novel insights
- Total: 3,000-8,000 words
- Full markdown with H2/H3 hierarchy
- Inline citations with URLs
- Bold key findings for scannability
- 1-2 structured comparison tables

**Complete Prompt Template:**

```xml
<role>
You are a senior research analyst conducting a comprehensive
investigation. Approach this with the rigor of an academic literature
review. Your analysis must be thorough, critically evaluated, and
transparent about its limitations.
</role>

<topic>[USER TOPIC]</topic>

<task>
Conduct a Level 3 (Ultra) comprehensive analytical report on the topic
above. This requires multi-pass research, critical evaluation of
sources, identification of contradictions, and synthesis across
multiple expert perspectives. Your output must meet the standard of a
professional research report.
</task>

<perspective_discovery>
Before researching, identify 4-6 distinct expert perspectives relevant
to this topic. For each perspective, generate 2-3 specific research
questions. Include at least one "contrarian" perspective that
challenges mainstream assumptions.

Perspectives to consider:
- [Domain expert perspective]
- [Practitioner/implementer perspective]
- [Critic/skeptic perspective]
- [Adjacent-field perspective]
- [End-user/affected-party perspective]
- [Contrarian/dissenting perspective]
</perspective_discovery>

<decomposition>
Organize your research into these major areas:
[Auto-generated hierarchical outline based on topic analysis]

For each area, investigate:
- Current state and key data points
- Leading positions and their evidence base
- Contradictions or active debates in the field
- Recent developments ([CURRENT_YEAR-1]-[CURRENT_YEAR])
- Connections to other research areas in this investigation
</decomposition>

<source_criteria>
- Minimum 15 sources, target 30-50
- Prioritize: peer-reviewed papers, official documentation, primary
  sources, expert analyses, technical reports
- Include at least 3 sources that challenge the dominant narrative
- All sources must be dated; flag anything older than 2 years for
  time-sensitive claims
- Include non-English sources where relevant
- Today's date: [DATE]
</source_criteria>

<verification_protocol>
Apply full Chain-of-Verification (CoVe):
1. For each major factual claim, generate a verification question
2. Answer the verification question independently (without referencing
   your draft)
3. If the verification contradicts your claim, revise or flag as
   uncertain
4. Rate confidence for each key finding: HIGH / MEDIUM / LOW
5. Only cite sources you can verify exist. Remove any you cannot
   confirm and note the gap.

Additional verification:
- Steelmanning: for contested points, construct the strongest version
  of each position before evaluating
- Temporal check: flag time-sensitive claims with source dates
- Distinguish between: established facts unlikely to change, current
  data that may have been updated, evolving situations requiring the
  latest information
</verification_protocol>

<output_format>
Structure as a research report:
1. Executive Summary (200-300 words)
2. Methodology and Scope
3-7. [Analytical sections with descriptive H2/H3 headers]
8. Critical Assessment: gaps, limitations, contradictions
9. Conclusion: key findings and novel insights
- Total: 3,000-8,000 words
- Use markdown with H2/H3 hierarchy
- Inline citations with URLs
- Tables for comparative data
- Bold key findings for scannability
</output_format>

<bias_awareness>
Identify the perspective your analysis takes. Present the strongest
counter-arguments with equal analytical rigor. Note perspectives that
may be underrepresented due to language, access, geographic, or
cultural barriers. Flag where your sources are dominated by a single
viewpoint.
</bias_awareness>
```

---

### Level 4 --- Super Ultra: Exhaustive Multi-Phase Research Study

**Trigger Criteria:**
- Composite score: 13-16
- Maximum-depth queries: systematic evidence gathering, meta-analysis, comprehensive field mapping
- Explicit depth indicators: "exhaustive", "systematic review", "all available evidence"
- Naturally requires 50+ sources and multi-domain synthesis
- Webb's DOK 4 / Bloom's Evaluate-Create
- Equivalent to: commissioned research report, systematic literature review
- Expected failure rates: 25-50% across current agents (Gemini benchmark data) --- the prompt must be maximally robust

**Source Requirements:**
- Count: 50-200+ sources across peer-reviewed papers, official documentation, technical reports, expert analyses, primary data
- Explicit inclusion/exclusion criteria (keywords, databases, periods)
- High proportion of academic articles; include foundational seminal works
- Non-English sources where relevant
- Systematically document search strategy
- Freshness: prioritize current year; include foundational earlier work with explicit dating

**Decomposition:**
- Full STORM-style perspective discovery + simulated expert conversations
- Multi-phase research plan:
  - Phase 1 (Reconnaissance): map full landscape --- all major sub-domains, stakeholders, perspectives, debates, breakthroughs, historical evolution
  - Phase 2 (Deep Investigation): exhaustive search per investigation area, document claims with evidence quality ratings, identify contradictions and root causes, seek non-obvious cross-domain connections
  - Phase 3 (Verification): full CoVe + self-consistency + steelmanning + temporal + citation integrity + coverage audit
  - Phase 4 (Synthesis): emergent patterns, novel frameworks/taxonomies, consensus mapping, frontier questions, actionable recommendations
- 8-12 investigation areas, each with 3 priority questions
- Dependency graph between phases and investigation areas

**Verification Protocol:**
- Full CoVe protocol (draft -> verification questions -> independent answers -> revise)
- Self-consistency: approach key questions from multiple framings and check for convergent conclusions
- Steelmanning: for every contested point, construct the strongest version of each position (minimum 3 competing perspectives)
- Temporal verification: flag all time-sensitive claims with source dates
- Citation integrity: verify every reference exists; remove unverifiable ones and note gaps
- Coverage audit: "After completing research, identify what important perspectives or information you may have missed"
- Availability bias check: "What perspectives are likely underrepresented in your training data or accessible sources?"
- Confidence calibration: rate each section's conclusions as HIGH / MEDIUM / LOW
- Research methodology transparency: document keywords, databases, periods, inclusion/exclusion criteria

**Output Format:**
- Academic survey structure:
  1. Executive Summary (500 words)
  2. Introduction: scope, research questions, significance
  3. Methodology: search strategy, inclusion criteria, analytical approach
  4-11. Analytical sections, each with sub-sections (H2/H3/H4 hierarchy)
  12. Cross-cutting synthesis: emergent patterns and novel frameworks
  13. Gap analysis: what remains unknown or under-researched
  14. Limitations of this study
  15. Conclusion: key findings, actionable insights, future directions
  16. Annotated key resources (15-25 most important sources with commentary)
- Total: 8,000-20,000+ words
- Full markdown with H2/H3/H4 hierarchy
- Embedded tables, comparison matrices, framework diagrams (described)
- Bold key findings throughout
- Inline citations with URLs
- Possible annexes: sources classified by type and date, glossary

**Complete Prompt Template:**

```xml
<role>
You are a principal research scientist leading an exhaustive
investigation. This should meet the rigor of a systematic literature
review or commissioned research study. Your analysis must be
comprehensive, methodologically transparent, critically evaluated,
and produce novel analytical insights.
</role>

<topic>[USER TOPIC]</topic>

<task>
Conduct a Level 4 (Super Ultra) exhaustive research study on the topic
above. This is the maximum depth level: systematic evidence gathering,
multi-domain synthesis, identification of all major perspectives,
critical evaluation of the full evidence base, and construction of
novel analytical frameworks. Your output must meet the standard of a
published systematic review or commissioned expert report.
</task>

<phase_1_reconnaissance>
Map the full landscape of this topic:
1. Identify ALL major sub-domains, stakeholders, and perspectives
2. Discover the key debates, open questions, and recent breakthroughs
3. Chart the historical evolution and current trajectory
4. Generate a comprehensive research plan with 8-12 investigation areas
5. For each area, identify:
   - The 3 most important questions to answer
   - Expected source types (academic, industry, government, expert)
   - Known gaps in available evidence
6. Identify dependencies between investigation areas
</phase_1_reconnaissance>

<phase_2_deep_investigation>
For each investigation area:
1. Conduct exhaustive search across academic databases, official
   documentation, technical reports, and expert commentary
2. For each finding, document:
   - The claim
   - Supporting evidence
   - Source quality (peer-reviewed / primary / secondary / opinion)
   - Publication date
   - Confidence level (HIGH / MEDIUM / LOW)
3. Identify contradictions between sources and investigate root causes
4. Include quantitative data wherever available
5. Seek out non-obvious connections between sub-domains
6. Track what sources you consulted that proved unhelpful (for
   methodology transparency)
</phase_2_deep_investigation>

<phase_3_verification>
Apply full verification protocol:
1. Chain-of-Verification: for every major claim, generate a
   verification question and answer it independently (without
   referencing your draft). If contradicted, revise or flag.
2. Self-consistency: where possible, triangulate from 3+ independent
   sources. Approach key questions from multiple framings.
3. Steelmanning: for every contested point, construct the strongest
   version of at least 3 competing positions before evaluating
4. Temporal check: flag all time-sensitive claims with source dates.
   Distinguish between: established facts, current data that may have
   been updated, and evolving situations.
5. Citation integrity: verify every reference exists. Remove any you
   cannot confirm and note the gap explicitly.
6. Coverage audit: after completing research, identify what important
   perspectives or information you may have missed.
7. Availability bias check: identify perspectives likely
   underrepresented in accessible sources.
8. Confidence calibration: rate each section's conclusions as
   HIGH / MEDIUM / LOW with justification.
</phase_3_verification>

<phase_4_synthesis>
1. Identify emergent patterns across investigation areas
2. Construct a novel analytical framework or taxonomy if the evidence
   supports one
3. Map: current consensus, active debates, and frontier questions
4. Produce actionable recommendations grounded in evidence
5. Identify implications for different stakeholder groups
</phase_4_synthesis>

<source_criteria>
- Target 50-200+ sources across: peer-reviewed papers, official
  documentation, technical reports, expert analyses, primary data
- Prioritize [CURRENT_YEAR-1]-[CURRENT_YEAR] publications; include
  foundational earlier work with explicit dating
- Include non-English sources where relevant
- Document inclusion/exclusion criteria explicitly
- Systematically document your search strategy (keywords, databases,
  periods)
- Do NOT fabricate any citation. If you cannot verify a source exists,
  omit it and explicitly note what information gap this creates.
- Today's date: [DATE]
</source_criteria>

<output_format>
Structure as a comprehensive research study:
1. Executive Summary (500 words)
2. Introduction: scope, research questions, significance
3. Methodology: search strategy, inclusion/exclusion criteria,
   analytical approach
4-11. [Analytical sections, each with H2/H3/H4 sub-sections]
12. Cross-cutting synthesis: emergent patterns and novel frameworks
13. Gap analysis: what remains unknown or under-researched
14. Limitations of this study
15. Conclusion: key findings, actionable insights, future directions
16. Annotated key resources (15-25 most important sources with
    one-paragraph commentary each)
- Total: 8,000-20,000+ words
- Full markdown with H2/H3/H4 hierarchy
- Embedded tables, comparison matrices
- Bold key findings throughout
- Inline citations with URLs
</output_format>

<bias_and_coverage>
- Present at least 3 competing perspectives for every contested point
- Identify what perspectives your analysis underweights and why
- Check for geographic, linguistic, and cultural coverage gaps
- Note where your sources are dominated by a single viewpoint
- Flag assumptions embedded in your analytical framework
</bias_and_coverage>
```

---

## 4. LLM Agnosticity Layer

### Design Philosophy

"Prompts overfit to models the same way models overfit to data" (Max Leiter, Vercel/v0 team). Perfect portability is currently impossible, but the core-plus-adapter pattern achieves practical agnosticism.

### Core-Plus-Adapter Architecture

**Two-layer design:**

| Layer | Contents | Changes When... |
|-------|----------|-----------------|
| **Model-Agnostic Core** | Research task, decomposition scaffold, source criteria, verification protocol, output format, anti-hallucination instructions | Topic or depth level changes |
| **Model-Specific Adapter** | Structural formatting, model-specific features (extended thinking, planning instructions), parameter configuration (temperature, token limits), position bias mitigation | Target model changes |

The core contains 100% of the research logic. The adapter contains 0% research logic --- only formatting and model-behavioral adjustments.

### XML as Universal Structural Format

- XML tags are the **only structural format explicitly endorsed by all three major providers** (Anthropic, OpenAI, Google)
- Prompt formatting affects performance by **up to 40%** depending on model (arXiv:2411.10541)
- Claude: specifically trained with XML in its data
- OpenAI (o1, o3): explicitly recommend "XML tags and section headings to clearly indicate distinct parts"
- Gemini: processes XML effectively
- Llama-3.1 405B: XML **consistently outperformed** other formats for complex prompts (even without XML-specific training)
- Described as "quasi-universal syntax in 2025" (Gemini doc)
- Token cost trade-off: XML consumes **40-60% more tokens** than JSON for equivalent structures, but higher accuracy requiring fewer iterations often means net cost favors XML
- **Decision**: Use XML tags for structural elements (`<role>`, `<task>`, `<topic>`, `<source_criteria>`, `<output_format>`, `<verification>`) and Markdown within those tags for content formatting (headers, tables, bold, lists)

### 7 Universal Prompt Techniques

These work reliably across Claude, GPT-4/o3, Gemini, Llama, and Mistral --- they form the portable core of every generated prompt:

| # | Technique | Implementation in Generated Prompts |
|---|-----------|-------------------------------------|
| 1 | Clear, explicit instructions | Every `<task>` tag states the exact research objective unambiguously |
| 2 | Few-shot examples | Included at Level 3-4 for complex output formats; optional at Level 1-2 |
| 3 | Role/persona assignment | Every prompt opens with `<role>You are a [role]</role>` |
| 4 | Structured output requests | `<output_format>` specifies exact structure (headers, tables, lists, length) |
| 5 | Chain-of-thought | Embedded in decomposition scaffold ("Break this into sub-questions, then investigate each"); treated as optional enhancement, not dependency --- marginal value decreasing with newer reasoning models |
| 6 | Context separation | XML tags delimit all sections; Markdown within tags for content |
| 7 | Constraint specification | Length, source count, temporal bounds, citation format, confidence ratings all explicitly stated |

### Model-Specific Adaptation Table

| Model Family | Preferred Format | Adapter Adjustments | Position Bias | Special Features |
|--------------|-----------------|---------------------|---------------|------------------|
| **Claude** (Anthropic) | XML tags (`<tags>`) | Put detailed instructions in user prompt body, minimal system prompt. Use `<thinking>` tags for extended reasoning at L3-4. Academic phrasing produces academic output (tone matching). Contextualize early, ask for hypotheses. | Performs better with relevant context **toward the top** | Extended thinking, long document analysis, strong reflection |
| **GPT models** (OpenAI) | Markdown / JSON schemas | Add explicit planning instructions ("Before responding, create a plan"). Define schema first, add examples. Use markdown headers and formatting. | Performs better with relevant examples **first** | Strict formatting compliance, strong code analysis |
| **Gemini** (Google) | Scope + Sources / Persona-Task-Context-Format | Specify time range and geographic scope explicitly. Leverage massive context window (1M+ tokens). | Varies by version | Multimodality, source linking, long-context optimization, plan-approve-execute pattern |
| **Llama** (Meta) | Match specific chat template | Adapt to model-specific chat template format. XML works well for complex prompts even without XML-specific training. | Performs better with relevant context **toward the top** | Open-source, customizable |
| **Qwen** (Alibaba) | Follow chat template | Standard XML/Markdown works. | Performs better with relevant context **toward the end** | Good multilingual support |
| **Mistral** | Match chat template | Standard XML/Markdown works. | Test per version | European model, good multilingual |

### Position Bias Mitigation

- Different models retrieve information from different positions more reliably
- **Universal mitigation**: Duplicate critical instructions at both beginning and end of generated prompts
- Front-load the most important research directives
- Place source material / context in the middle (least critical position)
- The "lost-in-the-middle" phenomenon persists across all models --- models retrieve information from beginning and end more reliably than middle

### Agnosticity Through Natural Language Constraints

Rather than adapting to each LLM's API parameters, the skill expresses all requirements as natural language constraints and delegates parameter mapping to the orchestrator:

| Instead of... | Express as... |
|---------------|---------------|
| `search_depth: "advanced"` | "Prioritize research depth over speed" |
| `max_results: 10` | "Do not exceed 10 sources" |
| `temperature: 0.3` | "Be precise and conservative in your claims" |
| `tools: ["web_search"]` | "Perform multiple research queries as needed" |

This is an elegant inversion: instead of the skill knowing about APIs, the APIs must interpret the skill's natural language.

### "Model Note" Pattern (from Gemini doc)

Include a brief model-adaptation note at the top of the prompt for dynamic behavior adjustment without altering the main strategy body:

```xml
<model_note>
[Model-specific instructions inserted by adapter layer.
Example for Claude: "Use extended thinking for verification steps."
Example for GPT: "Create an explicit plan before beginning research."
Example for Gemini: "Leverage your full context window for source retention."]
</model_note>
```

### Fallback Mechanisms

If a complex prompt fails on a specific model:
1. The system can switch to a simpler prompt version (reduce decomposition complexity)
2. Or switch to a different model via .env configuration
3. This degradation should be graceful and logged

### Portability Libraries Reference

For implementation, these libraries solve specific portability problems:

| Library | Solves | Downloads |
|---------|--------|-----------|
| **DSPy** | Programmatic, model-agnostic prompt optimization. Signatures define `input -> output`; compiler optimizes per model. Most promising path to true portability. | --- |
| **LiteLLM** | Unified API interface across 100+ providers. Write once, deploy anywhere. | --- |
| **Instructor** | Output portability via Pydantic schema enforcement. | 3M+/month, 15+ providers |
| **Outlines** | Output validity by modifying logits per-token. | --- |

---

## 5. Anti-Hallucination Master Protocol

### Hallucination Taxonomy

| Category | Description | Severity | Prevalence |
|----------|-------------|----------|------------|
| **Source/Citation Fabrication** | Inventing plausible articles, authors, URLs, DOIs | Critical | 100+ NeurIPS 2025 papers had hallucinated citations (GPTZero). 6% for well-studied topics, 28-29% for niche topics (JMIR). Accelerating trend since early 2025. |
| **Confirmation Bias / Sycophancy** | Only searching aligned sources; agreeing with user's implicit hypothesis | High | "General behavior of SOTA AI assistants" (Sharma et al.). Up to 100% initial compliance with illogical requests across 5 frontier models. NOT correlated with model size. |
| **Temporal Blindness** | Hallucinating about events beyond training period; presenting outdated info as current | High | GPT-4 achieves only 14% accuracy on fast-changing questions (FreshQA). |
| **Shallow Coverage / False Depth** | Repeating same information from multiple angles rather than genuine investigation | Medium | "A misleading impression of the quality of the research" (Willison). Systemic across deep research tools. |
| **Task/Answer Confusion** | LLM starts "answering" instead of building research plan/prompt | Medium | Common in meta-prompting contexts without explicit guards |
| **Plausible-but-Incorrect Facts** | Establishing fragile links or incorporating irrelevant content due to lexical proximity | High | More frequent than creative hallucinations ("red herrings") |
| **Long-Generation Degradation** | Errors accumulate in long outputs; early facts contradicted by later statements | Medium | Increases with output length; particularly relevant at L3-4 |
| **Complex List Errors** | Errors in enumerated data, comparison tables, ranked lists | Medium | Common in structured outputs |

### Mitigation Strategies Ranked by Level

#### Level 1 --- Simple

| Strategy | Implementation |
|----------|---------------|
| Basic source verification | "Confirm key facts from at least one additional source." |
| Anti-fabrication guard | "Do NOT fabricate citations. If you cannot verify a source exists, omit it." |
| Uncertainty flagging | "If information is uncertain or conflicting, state this explicitly." |
| Temporal grounding | "Today's date: [DATE]. Prioritize recent sources." |

#### Level 2 --- Advanced

All Level 1 strategies, plus:

| Strategy | Implementation |
|----------|---------------|
| Cross-reference requirement | "Cross-reference key claims across at least 2 independent sources." |
| Confidence ratings | "Rate confidence HIGH / MEDIUM / LOW for each major finding." |
| Contrarian source mandate | "Identify at least 2-3 sources or arguments that contradict or nuance the dominant thesis and summarize their arguments." |
| Disagreement synthesis | "Include a section identifying where sources disagree and why." |
| Freshness filtering | "Prioritize [CURRENT_YEAR-1]-[CURRENT_YEAR] for state of the art." |

#### Level 3 --- Ultra

All Level 1-2 strategies, plus:

| Strategy | Implementation |
|----------|---------------|
| Full CoVe protocol | Draft -> verification questions -> independent answers -> revise/flag |
| Steelmanning | "For contested points, construct the strongest version of each position before evaluating." |
| Temporal verification | "Flag time-sensitive claims with source dates. Distinguish established facts from evolving situations." |
| Citation integrity | "Only cite sources you can verify exist. Remove unverifiable ones and note gaps." |
| Bias awareness | "Identify the perspective your analysis takes. Present strongest counter-arguments. Note underrepresented perspectives." |
| Research methodology transparency | "Document how the research was conducted: keywords, databases, periods." |
| Relevance filtering | "Before incorporating a source, assess its actual utility for the specific claim." |
| Clean context delivery | "Deduplicate, rerank, and refine findings at each research pass." |

#### Level 4 --- Super Ultra

All Level 1-3 strategies, plus:

| Strategy | Implementation |
|----------|---------------|
| Self-consistency | "Approach key questions from multiple framings and check for convergent conclusions." |
| Coverage audit | "After completing research, identify what important perspectives or information you may have missed." |
| Availability bias check | "What perspectives are likely underrepresented in your training data or accessible sources?" |
| 3+ perspective mandate | "Present at least 3 competing perspectives for every contested point." |
| Global memory management | "Iteratively record discoveries. Ensure facts found early are not contradicted or forgotten during final synthesis." |
| Confidence calibration with justification | "Rate each section HIGH / MEDIUM / LOW with explicit justification for the rating." |
| Uncertainty analysis protocol | "When encountering contradictory information: present all perspectives with evidence quality ratings, identify what missing information would increase confidence." |
| StepBack-Prompting | "Abstract from specific instances to identify fundamental principles before drawing conclusions." |
| Source quality stratification | "Classify each source: peer-reviewed > primary > secondary > opinion. Weight claims accordingly." |
| Research methodology full documentation | "Document inclusion/exclusion criteria, search strategy, keywords, databases, periods, languages." |

### The CoVe Protocol (Detailed Reference)

From Meta/FAIR (ACL Findings 2024, arXiv:2309.11495). **More than doubles precision** on factual claims.

1. **Draft**: Generate initial research response
2. **Generate verification questions**: For each factual claim in the draft, formulate a specific verification question
3. **Answer independently**: Answer each verification question WITHOUT referencing the original draft (prevents confirmation bias)
4. **Revise**: If verification answer contradicts draft claim, either revise the claim or flag it as uncertain with the conflicting evidence

### Special Consideration: Topic Obscurity

Hallucination risk is inversely proportional to topic popularity in training data:
- Well-studied topics: ~6% fabrication rate
- Less-studied topics: 28-29% fabrication rate

**Implication**: Verification requirements should scale not just with depth level but also with estimated topic obscurity. For niche topics, even Level 2 should apply Level 3 verification intensity.

### Decreasing Value of Chain-of-Thought

Per Mollick et al. (Wharton, June 2025): CoT's advantage **diminishes with newer reasoning models** (o3, Claude with extended thinking). Models internalize reasoning. However, CoT retains value for research **decomposition** (structuring what to investigate), even as its value for simple reasoning tasks decreases. The skill should treat CoT as an enhancement for decomposition, not a dependency for reasoning.

---

## 6. Benchmark Reference Data

### DRB-II (Deep Research Bench II) --- January 2026

| Model | Info Recall (%) | Analysis (%) | Presentation (%) | Total Score (%) |
|-------|----------------|-------------|-------------------|----------------|
| OpenAI-GPT-o3 DR | 39.98 | 49.85 | 89.16 | **45.40** |
| Gemini-3-Pro DR | 39.09 | 48.94 | 91.85 | **44.60** |
| Doubao Deep Research | 34.83 | 49.43 | 83.51 | **40.99** |
| Perplexity Research | 33.05 | 44.47 | 79.34 | **38.58** |
| LangChain open_deep_research | --- | --- | --- | RACE: 0.4344 (#6) |

**Key observations:**
- All top systems score below 50% total --- significant gap between agents and human experts
- Information Recall is the weakest dimension, plateauing around 40%
- Presentation is consistently strong: >85% for top systems
- Deep analysis and complete evidence extraction remain the primary challenge

### ResearchRubrics Benchmark

- Created by University of Chicago
- 2,800+ hours of human work, 2,500 expert rubrics
- Evaluates: factual grounding, reasoning soundness, clarity
- arXiv: 2511.07685v1

### Commercial System Performance Comparison

| System | Architecture | Typical Speed | Multimodal | User Control |
|--------|-------------|---------------|------------|--------------|
| OpenAI Deep Research | RL-trained single agent (o3) | 5-30 min | Text, images, PDFs | Clarification questions |
| Claude Research | Multi-agent orchestrator-worker | 1-3 min | Text-focused | Thinking visible |
| Gemini Deep Research | Plan-approve-execute | 2-5 min | Text only | Plan reviewable |
| Perplexity Deep Research | Iterative retrieval loop | <3 min | Text + code | Steps visible |

### API/Tool Performance Comparison

| Tool | Average Latency | Agentic Quality Score | Key Strength |
|------|----------------|----------------------|--------------|
| Brave Search | 980 ms | 14.89 | Speed and index freshness |
| Tavily | 998 ms | 13.67 | Structured extraction, authorized sources |
| Exa AI | 2.9 s | 13.50 | Semantic understanding, "highlights" |
| Perplexity | 11+ s | 12.96 | Synthesized answers, factual verification |

### Key Quantitative Data (All Sources)

| Metric | Value | Source |
|--------|-------|--------|
| Multi-agent vs single-agent improvement | 90.2% | Anthropic |
| Token usage explains quality variance | 80% | Anthropic |
| Three factors explain variance | 95% (tokens, tool calls, model choice) | Anthropic |
| Parallel tool calling time reduction | up to 90% | Anthropic |
| STORM organization improvement | 25% | STORM paper (NAACL 2024) |
| STORM coverage improvement | 10% | STORM paper |
| Co-STORM user preference | 70% of volunteers | Co-STORM |
| CrewAI vs LangGraph speed | 5.76x faster | CrewAI benchmarks |
| LangGraph monthly downloads | 90M | LangGraph |
| OpenAI HLE score | 26.6% | Humanity's Last Exam |
| DeepSeek R1 HLE score | 9.4% | Humanity's Last Exam |
| GPT-4o HLE score | 3.3% | Humanity's Last Exam |
| Perplexity daily queries | 200M | Perplexity |
| Perplexity median latency | 358ms | Perplexity |
| APE: LLM beat human prompts | 24/24 tasks | Zhou et al., ICLR 2023 |
| APE: LLM IQM vs human IQM | 0.810 vs 0.749 | Zhou et al. |
| OPRO improvement on GSM8K | up to 8% | OPRO, ICLR 2024 |
| OPRO improvement on Big-Bench Hard | up to 50% | OPRO |
| Meta-Prompting improvement | 17.1% | Suzgun & Kalai, 2024 |
| DSPy MIPROv2 accuracy gain | 46.2% -> 64.0% | DSPy, ICLR 2024 |
| Prompt format performance impact | up to 40% | arXiv:2411.10541 |
| XML token overhead vs JSON | 40-60% more | Multiple sources |
| MCP monthly SDK downloads | 97M+ | Linux Foundation / AAIF |
| Instructor monthly downloads | 3M+ | Instructor |
| NeurIPS 2025 hallucinated citations | 100+ papers | GPTZero |
| Hallucination rate (well-studied topics) | 6% | JMIR Mental Health |
| Hallucination rate (less-studied topics) | 28-29% | JMIR Mental Health |
| Sycophancy compliance rate | up to 100% | 2025 frontier model study |
| GPT-4 FreshQA fast-changing accuracy | 14% | FreshQA |
| CoVe precision improvement | more than doubles | Meta/FAIR, ACL 2024 |
| Gemini context window | 1-2M tokens | Google |
| Claude Sonnet 4 context window | 200K-1M tokens | Anthropic |
| Step-DeepResearch rubric conformity | 61.42% | arXiv 2512.20491v4 |
| Step-DeepResearch cost vs proprietary | ~10x cheaper | StepFun AI |
| Super Ultra task failure rate | 25-50% | Gemini benchmark data |
| Info Recall ceiling (all systems) | ~40% | DRB-II |

### Depth-Level to Search API Mapping

| Level | Recommended APIs | Rationale |
|-------|-----------------|-----------|
| Simple | Brave Search or Tavily | Fast, credible, sufficient for single-pass |
| Advanced | Tavily or Exa | Structured extraction, semantic understanding |
| Ultra | Multi-source aggregation (Tavily + Exa + academic) | Breadth and depth required |
| Super Ultra | Full multi-source (Brave + Tavily + Exa + academic databases + arXiv) | Exhaustive coverage mandate |

---

## 7. Key Design Decisions

### Decision 1: Task vs Answer Separation

**Decision**: The skill generates prompts that instruct the LLM to RESEARCH, not to ANSWER.

**Rationale**: Most prompt engineering focuses on getting better answers. This skill focuses on generating better research processes. The generated prompt tells the executing LLM: "Your task is NOT to directly answer the question, but to design and execute a research strategy." This prevents the common failure mode where the LLM skips research and produces a plausible-sounding but poorly grounded response.

**Implementation**: Every generated prompt includes an explicit anti-confusion guard in the `<task>` component. At Level 1 (where a direct answer IS the output), the guard is relaxed but source requirements remain.

### Decision 2: XML + Markdown Hybrid

**Decision**: Use XML tags for structural delimitation, Markdown within tags for content formatting.

**Rationale**: XML is the only format endorsed by all three major providers and outperforms alternatives on complex prompts. Markdown provides familiar, readable content formatting. The hybrid avoids proprietary syntax while maximizing compatibility.

**Trade-off**: 40-60% more tokens than pure JSON, but higher accuracy and fewer retry iterations produce net cost savings.

### Decision 3: Core-Plus-Adapter Over Universal Prompt

**Decision**: Two-layer architecture with model-agnostic core and model-specific adapter, rather than a single "universal" prompt.

**Rationale**: Perfect portability is impossible (Leiter). The core contains all research logic; the adapter handles only formatting and model-behavioral adjustments. This means the research quality is model-independent while execution is model-optimized.

### Decision 4: Auto-Calibration Over User Selection

**Decision**: The system auto-detects depth level by default, with user override available.

**Rationale**: No consumer tool currently offers automatic granular depth calibration. Users consistently either over- or under-estimate the depth their query requires. Auto-calibration with transparent reasoning ("I detected Level 3 because...") is more useful than forcing a choice.

### Decision 5: CoVe as Mandatory at L3+

**Decision**: Chain-of-Verification is mandatory in generated prompts at Level 3 and above, not optional.

**Rationale**: CoVe more than doubles factual precision (Meta/FAIR). At L3-4 where the output is long and claims are numerous, the risk of accumulated hallucination is high. The cost (more tokens, more time) is justified by the quality improvement.

### Decision 6: Disagreement Blocks as Standard Component

**Decision**: Every generated prompt at every level includes some form of disagreement/contradiction identification.

**Rationale**: Most research frameworks focus on consensus. Explicitly requiring disagreement synthesis is unusual but produces higher-quality research by preventing confirmation bias and sycophancy. This is embedded at L1 as "flag conflicting information" and escalates to "present 3+ competing perspectives" at L4.

### Decision 7: Self-Critique as Architectural Step, Not Best Practice

**Decision**: Stage 7 (meta-prompting assembly) includes mandatory self-critique and self-revision of the generated prompt BEFORE output.

**Rationale**: PromptWizard and APO demonstrate that generate-critique-refine loops systematically improve prompt quality. This is not a suggestion --- it is Step 7e-7f of the pipeline.

### Decision 8: Progressive Pipelines Over Upfront Query Generation

**Decision**: At L3-4, the generated prompt instructs the executing LLM to generate search queries adaptively based on initial discoveries, NOT to generate all queries upfront.

**Rationale**: Upfront query generation leads to redundancy and misses relevant threads discovered during research. Progressive pipelines (Gemini insight) distinguish genuine deep research from multi-query RAG.

### Decision 9: Researcher/Reporter Conceptual Separation

**Decision**: At L3-4, the generated prompt separates the research phase from the writing phase, even within a single-agent execution.

**Rationale**: MS-Agent v1.6's Researcher/Reporter decoupling shows that separating investigation from synthesis ensures writing quality does not compromise research thoroughness and vice versa. Even without multi-agent infrastructure, the prompt can enforce this separation through phased instructions.

### Decision 10: Natural Language Constraints Over API Parameters

**Decision**: Express all requirements as natural language in the generated prompt, never as API-specific parameters.

**Rationale**: The skill does not know which API or platform will execute the prompt. Natural language constraints ("prioritize depth over speed", "do not exceed 10 sources") are universally interpretable. The orchestrator layer maps these to native parameters.

---

## 8. Gap Analysis

### Gaps Identified Across All Three Sources

#### Gap 1: No Standardized Prompt Portability Format

- MCP standardizes tool connectivity but explicitly NOT prompt formatting
- No equivalent of MCP exists for prompt structure
- The XML+Markdown hybrid is a pragmatic choice, not a standard
- **Risk**: As models evolve, the "universal" format may fragment
- **Mitigation**: The adapter layer isolates format changes from research logic

#### Gap 2: Calibration Threshold Validation

- The 4-axis complexity matrix and score ranges (4-6, 7-9, 10-12, 13-16) are theoretically grounded but not empirically validated on a large query dataset
- Intent-Based Prompt Calibration provides a mechanism for self-calibration via synthetic boundary cases, but this has not been implemented or tested for this specific use case
- **Risk**: Mis-calibration leads to over- or under-depth research
- **Mitigation**: Start with conservative thresholds, collect user feedback, iterate. Allow user override as escape valve.

#### Gap 3: Verification of Verification

- CoVe and other anti-hallucination protocols are instructions to the LLM --- there is no external validation that the LLM actually followed them
- An LLM can "perform" CoVe (generate verification questions and answers) without genuinely verifying
- **Risk**: False confidence in verified results
- **Mitigation**: At L3-4, require explicit verification logs that can be audited. Future: external fact-checking tool integration.

#### Gap 4: Information Recall Ceiling

- DRB-II shows all top systems cap at ~40% information recall
- The skill can generate excellent prompts, but the executing LLM's ability to actually find and retrieve relevant information is bounded by its search capabilities
- **Risk**: Perfect prompts hitting imperfect retrieval
- **Mitigation**: The skill is search-backend-agnostic. As retrieval improves, the same prompts will produce better results. Multi-source aggregation at L3-4 partially addresses this.

#### Gap 5: Multi-Language Research Support

- All three sources mention non-English sources as important for L3-4 coverage
- No source provides a concrete protocol for multi-language research (query translation, source evaluation across languages, synthesis of findings in different languages)
- **Risk**: Systematic bias toward English-language sources
- **Mitigation**: Include explicit instruction to "consider non-English sources" and "note language coverage gaps." Full multilingual protocol is a future enhancement.

#### Gap 6: Cost and Latency Estimation

- No source provides a cost or latency model for the different levels
- L4 prompts targeting 50-200+ sources and 8,000-20,000+ words will be expensive and slow
- **Risk**: User surprise at cost/time; no way to budget
- **Mitigation**: Include estimated token counts and time ranges per level in user-facing documentation. Consider a "budget" parameter alongside depth level.

#### Gap 7: Evaluation of Generated Prompts

- The skill generates prompts, but there is no built-in mechanism to evaluate whether a generated prompt is actually good (beyond the self-critique in Stage 7)
- ResearchRubrics evaluates research outputs, not research prompts
- **Risk**: No objective quality metric for the skill's core output
- **Mitigation**: Use ResearchRubrics on downstream outputs as a proxy. Future: create a "PromptRubrics" evaluation framework.

#### Gap 8: Real-Time / Streaming Research

- All architectures assume batch processing (query in, report out)
- No source addresses real-time or streaming research where findings are delivered incrementally
- **Risk**: Poor user experience for L3-4 which may take minutes to hours
- **Mitigation**: The phased structure of L3-4 prompts (reconnaissance -> deep investigation -> verification -> synthesis) naturally supports incremental delivery. This should be formalized.

#### Gap 9: Human-in-the-Loop Integration

- Gemini's plan-approve-execute model and Co-STORM's dynamic mind map (70% user preference) suggest significant value in human review during research
- The current architecture generates a prompt and sends it --- no intermediate review point
- **Risk**: Wasted computation if research direction is wrong
- **Mitigation**: At L3-4, the skill could generate a research plan for user approval BEFORE generating the full research prompt. This mirrors Gemini's approach and could be a future enhancement.

#### Gap 10: Handling Model Refusals and Failures

- No source addresses what happens when the executing LLM refuses part of the research (safety filters, content policies) or simply fails to follow complex instructions
- Step-DeepResearch shows 25-50% failure rates at Super Ultra level
- **Risk**: Silent failures or partial research with no warning
- **Mitigation**: Include in generated prompts: "If you are unable to research any area, explicitly state what you could not investigate and why." The adapter layer should include model-specific failure handling.

#### Gap 11: Quantitative Data Sourcing

- All sources focus on text-based research
- No protocol for incorporating quantitative data (datasets, statistics, numerical evidence) beyond what appears in text sources
- **Risk**: Research that lacks quantitative grounding
- **Mitigation**: Source criteria at L3-4 include "quantitative data wherever available" but this could be more formalized with specific data source types (government statistics, research datasets, industry reports with numerical data).

---

*End of unified architecture document. This blueprint is designed to feed directly into the skill writer for implementation.*
