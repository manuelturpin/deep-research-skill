# Extraction: Building an LLM-agnostic Meta-Research Prompt Framework

Source: `/tmp/research-docs/claude.md`

---

## 1. Key Frameworks & Systems Referenced

### Open-Source Research Frameworks

**GPT Researcher**
- Architecture: Plan-and-Solve — Planner Agent generates research questions, multiple Execution Agents scrape sources in parallel via `asyncio.gather()`, Aggregation Agent compiles cited reports
- Deep research mode: tree-like recursive exploration with configurable depth and breadth
- Multi-agent variant (LangGraph): Chief Editor, Browser, Editor, Researcher, Reviewer, Reviser
- LLM support: OpenAI, Anthropic, Gemini, Ollama
- Search backends: Tavily, DuckDuckGo, Google, Exa, arXiv
- Closest existing analog to an LLM-agnostic research framework
- Repo: github.com/assafelovic/gpt-researcher

**STORM (Stanford, NAACL 2024)**
- Approach: Perspective-guided question asking (fundamentally different from direct decomposition)
- Process: Discovers diverse perspectives by analyzing similar Wikipedia articles, then simulates conversations between "Wikipedia writer" and "topic expert" grounded in internet sources
- Performance: **25% better article organization**, **10% broader coverage** than baselines
- Built on DSPy — modular architecture: Knowledge Curation -> Outline Generation -> Article Generation -> Article Polishing
- Co-STORM variant: human-in-the-loop via dynamic mind map, **70% of volunteers preferred it** over traditional search
- Repo: github.com/stanford-oval/storm, arXiv:2402.14207

**LangChain open_deep_research**
- Ranks **#6 on Deep Research Bench Leaderboard** (RACE score: 0.4344)
- Architecture: supervisor-researcher with clarification phase, research brief generation, parallel sub-topic research in dedicated context windows, synthesis
- LangGraph engine: supports 3 orchestration patterns (single agent, multi-agent, hierarchical)
- LangGraph stats: **90M monthly downloads**, enterprise adoption at Uber, JP Morgan, LinkedIn
- Repo: github.com/langchain-ai/open_deep_research

**CrewAI**
- Design: Intuitive role-based — agents assigned roles, goals, backstories
- Dual architecture: Crews (autonomous collaboration) + Flows (deterministic orchestration)
- Memory: short-term, long-term, entity, contextual
- Performance: **5.76x faster** than LangGraph on certain tasks
- Repo: github.com/crewAIInc/crewAI

### Commercial Deep Research Implementations

**OpenAI Deep Research**
- Architecture: End-to-end RL on optimized o3 model (single monolithic agent, NOT multi-agent)
- Capabilities: clarification questions, autonomous web browsing (5-30 min), analyzes text/images/PDFs
- Performance: **26.6%** on Humanity's Last Exam (vs DeepSeek R1 9.4%, GPT-4o 3.3%)

**Claude Research System (Anthropic, June 2025 blog)**
- Architecture: Explicit multi-agent — Lead Agent (Claude Opus 4) plans + spawns subagents, Subagents (Claude Sonnet 4) execute parallel info gathering with interleaved thinking, Citation Agent handles attribution
- Key finding: 3 factors explain **95% of performance variance**: token usage (80%), tool calls, model choice
- Parallel tool calling cut research time by **up to 90%**
- 8 key principles including "teach orchestrator how to delegate," "scale effort to query complexity," "start wide, then narrow down"
- Multi-agent outperforms single-agent by **90.2%**

**Gemini Deep Research**
- Architecture: Plan-approve-execute cycle (user reviews plan before execution)
- Integrates Google Workspace (Gmail, Drive, Chat, Sheets)
- Uses **1-million-token context window** + RAG for session memory

**Perplexity Deep Research**
- Architecture: Multi-stage retrieval-reasoning-refinement cycle: query decomposition -> dedicated searches per subtopic (hybrid lexical + semantic retrieval) -> partial answer synthesis -> conflict verification -> final narrative with citations + reliability notes
- Stats: **200 million daily queries**, median latency **358ms**, completes most research in <3 minutes

### Comparison Table (verbatim from source)

| Feature | OpenAI | Claude | Gemini | Perplexity |
|---------|--------|--------|--------|------------|
| Architecture | RL-trained single agent | Multi-agent orchestrator-worker | Plan-approve-execute | Iterative retrieval loop |
| Typical speed | 5-30 min | 1-3 min | 2-5 min | <3 min |
| Multimodal | Text, images, PDFs | Text-focused | Text only | Text + code |
| User control | Clarification questions | Thinking visible | Plan reviewable | Steps visible |

### Meta-Prompting & Automatic Prompt Engineering Frameworks

**APE (Zhou et al., 2023, ICLR)**
- LLMs achieve **human-level prompt engineering** on **24/24** instruction-induction tasks
- InstructGPT outperformed human prompts: IQM **0.810 vs. 0.749**
- arXiv:2211.01910

**OPRO (Google DeepMind, ICLR 2024)**
- Trajectory-based prompt optimization
- Outperforms hand-designed prompts by **up to 8% on GSM8K** and **up to 50% on Big-Bench Hard**
- arXiv:2309.03409

**Meta-Prompting (Suzgun & Kalai, 2024)**
- Conductor-expert pattern: single LLM as both orchestrator and panel of diverse experts
- **17.1% improvement** over standard prompting
- arXiv:2401.12954

**PromptBreeder (ICML 2024)**
- Self-referential evolutionary prompt optimization
- arXiv:2309.16797

**DSPy (Stanford, ICLR 2024)**
- Programmatic, model-agnostic prompt optimization
- MIPROv2 optimizer: Bayesian optimization over instructions and few-shot demonstrations
- Raised accuracy from **46.2% to 64.0%** on prompt evaluation tasks
- Signatures define `input -> output` specs; compiler optimizes per target model
- Repo: github.com/stanfordnlp/dspy, dspy.ai

### Prompting Techniques

**Self-Ask (Press et al., 2023)** — decomposes complex questions into sub-questions answerable individually with search. arXiv:2210.03350

**Skeleton-of-Thought (ICLR 2024)** — generates outline first, expands sections in parallel (directly applicable to research report generation)

**Tree of Thoughts (NeurIPS 2023)** — maintains parallel reasoning branches with backtracking capability

**Plan-and-Solve (Wang et al., 2023)** — directly inspired GPT Researcher's architecture

**Chain-of-Verification / CoVe (ACL Findings 2024)** — drafting, generating verification questions, answering them independently, revising. **More than doubles precision** on factual claims. arXiv:2309.11495

### Structured Output & Portability Libraries

**Instructor** — 3M+ monthly downloads, supports 15+ providers. github.com/instructor-ai/instructor

**Outlines** — guarantees valid output by modifying logits per-token. github.com/dottxt-ai/outlines

**LiteLLM** — unified interface across 100+ providers

### Standards

**Model Context Protocol (MCP)** — now under Linux Foundation governance via AAIF, **97M+ monthly SDK downloads**. Standardizes tool connectivity but NOT prompt formatting.

---

## 2. Architecture Patterns

### The 5 Essential Prompt Components (in order)

1. **Role and context frame** — Define who the LLM is (domain expert, research analyst) and provide research context (topic, background, audience). Anthropic's data: Claude matches tone/register of the prompt itself — academic phrasing produces academic output.

2. **Decomposition scaffold** — Structure the topic into hierarchical sub-questions using Self-Ask pattern. STORM's perspective-guided approach (diverse viewpoints first, then questions per perspective) produces **25% better organization**. Instruct LLM to identify 3-5 major facets, generate 2-4 specific questions per facet, flag follow-up questions.

3. **Source criteria and verification protocol** — Specify acceptable source types (peer-reviewed, primary, authoritative), recency requirements, cross-validation instructions. Embed CoVe steps directly. This alone **more than doubles precision** on factual claims.

4. **Output format specification** — Define exact structure, length, citation format, level of analysis. Match format to depth level. Include explicit formatting instructions (markdown headers, tables, bold for key findings).

5. **Constraint and scope boundaries** — Temporal bounds, topical limits, depth expectations, anti-hallucination instructions. Include current date for temporal grounding. Specify what LLM should do when uncertain: flag rather than guess.

### Core-Plus-Adapter Pattern

- **Rationale**: "Prompts overfit to models the same way models overfit to data" (Max Leiter, Vercel/v0 team) — perfect portability is currently impossible
- **Two-layer design**:
  - **Model-agnostic core**: research task, decomposition, source criteria, output format
  - **Model-specific adapter**: adjusts structural formatting, adds model-specific features (extended thinking for Claude, planning instructions for GPT), configures parameters (temperature, token limits)

### Three-Stage Pipeline (the recommended framework architecture)

1. **DETECT** — Analyze query complexity across breadth, depth, ambiguity, and temporality dimensions
2. **DECOMPOSE** — Use STORM-style perspective discovery and Self-Ask question generation scaled to detected level
3. **GENERATE** — Assemble optimized prompt from modular components (role, decomposition scaffold, source criteria, verification protocol, output format) wrapped in model-specific adapter layer

### Meta-Prompting Pipeline (three-step)

1. User provides a raw topic
2. Framework prompts an LLM to analyze complexity, identify perspectives, decompose the topic
3. Framework assembles an optimized prompt by selecting components matched to detected depth level

This mirrors STORM's approach but operates at the **prompt-generation level** rather than research-execution level.

### Anthropic's Prompt Engineering Hierarchy

1. Be clear and direct
2. Use multishot examples
3. Chain of thought
4. XML-style thinking tags
5. Role assignment

Key insight: **"If your prompt isn't clear or lacks examples, no amount of clever phrasing will save it."**

---

## 3. Level System Design

### Complexity Classification

A practical complexity classifier assigns a score on four dimensions:
- **Breadth** — number of distinct sub-topics
- **Depth** — logical nesting and reasoning hops required
- **Ambiguity** — open-endedness of the query
- **Temporality** — how time-sensitive the information is

The composite score maps to one of four depth levels.

### Complexity Signal Types

**Structural signals** (from query itself): entity count, comparison operators ("vs," "compare"), temporal qualifiers ("evolution," "recent"), scope indicators ("comprehensive," "brief"), question type (factual vs. analytical vs. evaluative). Maps to Bloom's taxonomy levels.

**Retrieval-based signals**: number of documents needed (multi-hop indicator), diversity of relevant source types, contradiction rate across initial sources. High contradiction = deeper analysis needed.

**LLM-based assessment**: prompt LLM to estimate complexity before generating research prompt. Count sub-questions during decomposition — more = higher complexity. First-token probability and entropy correlate with question difficulty (Zotos et al., 2024-2025).

---

### Level 1 — Simple: Quick Factual Synthesis

**Webb's DOK**: Level 1 (Recall & Reproduction)
**Bloom's**: Remember/Understand
**Trigger criteria**: Single-entity queries, factual lookups, definitions, "what is" questions. <2 entities, no comparison operators, no analytical verbs.
**Sources**: 1-5
**Word count**: 100-500 words
**Decomposition**: None needed
**Output format**: Direct answer — one paragraph context, the answer, 1-5 inline citations. No sections, no tables unless comparing 2-3 items.
**Verification**: "Confirm the answer from at least one additional source."
**Equivalent to**: Perplexity Quick Search or standard ChatGPT response
**Prompt characteristics**: Role as "concise research assistant," single-pass search instruction, direct answer format.

### Level 2 — Advanced: Structured Analytical Overview

**Webb's DOK**: Level 2-3
**Bloom's**: Apply/Analyze
**Trigger criteria**: Multi-facet queries requiring comparison, explanation of mechanisms, moderate analysis. 2-4 entities, comparison operators, or "how/why" questions requiring multiple perspectives.
**Sources**: 5-15 (mixed types)
**Word count**: 1,000-3,000 words
**Decomposition**: 3-5 sub-questions
**Output format**: Executive summary (3-5 sentences), 3-5 themed sections with H2 headers, comparison tables where relevant, conclusion with key takeaways. Inline citations throughout.
**Verification**: "Cross-reference key claims across sources." Rate confidence HIGH/MEDIUM/LOW for each major finding.
**Equivalent to**: Perplexity Pro Search or You.com Research mode
**Prompt characteristics**: Role as "expert analyst in [domain]," source diversity requirement, structured output with headers and tables.

### Level 3 — Ultra: Comprehensive Analytical Report

**Webb's DOK**: Level 3-4
**Bloom's**: Analyze/Evaluate
**Trigger criteria**: Complex multi-dimensional queries requiring critical evaluation, synthesis across domains, extensive source analysis. 4+ entities, cross-domain analysis, scope indicators like "comprehensive," "in-depth," "analyze."
**Sources**: 15-50 (preference for primary and peer-reviewed)
**Word count**: 3,000-8,000 words
**Decomposition**: STORM-style perspective discovery (4-6 distinct expert perspectives), hierarchical decomposition into major facets and sub-questions
**Output format**: Executive summary, methodology note, 5-8 analytical sections with H2/H3 hierarchy, embedded tables/data, critical assessment section (gaps + limitations), conclusion with novel insights. All claims cited.
**Verification**: Full CoVe protocol + steelmanning + temporal verification. Confidence ratings.
**Equivalent to**: OpenAI Deep Research or Gemini Deep Research output
**Prompt characteristics**: Role as "senior research lead specializing in [domain]," rigorous source requirements, full CoVe verification, methodology documentation.

### Level 4 — Super Ultra: Exhaustive Multi-Agent Research Study

**Webb's DOK**: Level 4 (Extended Thinking)
**Bloom's**: Evaluate/Create
**Trigger criteria**: Maximum-depth queries — systematic evidence gathering, meta-analysis across domains, comprehensive field mapping. Explicit depth indicators ("exhaustive," "systematic review," "all available evidence") or naturally requires 50+ sources and multi-domain synthesis.
**Sources**: 50-200+
**Word count**: 8,000-20,000+ words
**Decomposition**: Full STORM-style perspective discovery + simulated expert conversations. Multi-phase research plan.
**Output format**: Academic survey structure — abstract, introduction with research questions, methodology, 8-12 analytical sections, synthesis creating novel frameworks, gap analysis, limitations, conclusion with actionable insights, annotated source list (15-25 most important). Includes tables, comparison matrices, framework diagrams.
**Verification**: Full CoVe + self-consistency (approach key questions from multiple framings) + steelmanning + temporal verification + coverage audit + citation integrity check. Confidence calibration per section (HIGH/MEDIUM/LOW).
**Equivalent to**: Commissioned research report or systematic review
**Prompt characteristics**: Role as "principal research scientist leading a research team," multi-phase plan (reconnaissance -> deep investigation -> synthesis -> verification -> gap analysis), maximum source requirements, systematic methodology documentation.

### Scaled Verification Protocol Across Levels

- **Level 1**: "Confirm key facts from at least one additional source. Flag uncertainty."
- **Level 2**: "Cross-reference key claims across 2+ sources. Rate confidence HIGH/MEDIUM/LOW for each major finding."
- **Level 3**: Full CoVe protocol + steelmanning + temporal verification.
- **Level 4**: Full CoVe, self-consistency (approach key questions from multiple framings), steelmanning, temporal verification, coverage audit, and citation integrity check.

---

## 4. Complete Prompt Templates (VERBATIM)

### Level 1 Prompt Template

```
You are a research assistant. Answer the following question concisely
and accurately using current web sources. Provide 1-5 citations.


Question: [USER TOPIC]


Requirements:
- Direct answer in 1-3 paragraphs (under 500 words)
- Cite sources with URLs
- If information is uncertain or conflicting, state this explicitly
- Today's date: [DATE]. Prioritize recent sources.
```

### Level 2 Prompt Template

```
You are an expert [DOMAIN] analyst conducting a structured research overview.


<topic>[USER TOPIC]</topic>


<research_plan>
Break this topic into 3-5 key sub-questions:
1. [Auto-generated sub-question based on topic analysis]
2. [Auto-generated sub-question]
3. [Auto-generated sub-question]
</research_plan>


<source_criteria>
- Use 5-15 diverse sources (academic papers, official documentation, expert analysis)
- Prioritize sources from 2024-2025
- Cross-reference key claims across at least 2 independent sources
</source_criteria>


<output_format>
- Executive summary (100 words)
- 3-5 sections with descriptive headers
- Comparison tables where relevant
- Key takeaways section
- Total length: 1,000-3,000 words
- Inline citations with URLs
</output_format>


<verification>
For each major claim, verify against a second source. Flag any
information that is contested or uncertain.
</verification>


Today's date: [DATE].
```

### Level 3 Prompt Template

```
You are a senior research analyst conducting a comprehensive
investigation. Approach this with the rigor of an academic literature review.


<topic>[USER TOPIC]</topic>


<perspective_discovery>
Before researching, identify 4-6 distinct expert perspectives relevant
to this topic. For each perspective, generate 2-3 specific research
questions. Include a "contrarian" perspective that challenges mainstream
assumptions.
</perspective_discovery>


<decomposition>
Organize your research into these major areas:
[Auto-generated hierarchical outline based on topic analysis]
For each area, investigate:
- Current state and key data points
- Leading positions and their evidence
- Contradictions or debates in the field
- Recent developments (2024-2025)
</decomposition>


<source_criteria>
- Minimum 15 sources, target 30-50
- Prioritize: peer-reviewed papers, official documentation, primary
  sources, expert analyses
- Include at least 3 sources that challenge the dominant narrative
- All sources must be dated; flag anything older than 2 years for
  time-sensitive claims
- Today's date: [DATE]
</source_criteria>


<verification_protocol>
1. For each major factual claim, generate a verification question
2. Answer the verification question independently (without referencing
   your draft)
3. If the verification contradicts your claim, revise or flag as uncertain
4. Rate confidence for each key finding: HIGH / MEDIUM / LOW
5. Only cite sources you can verify exist
</verification_protocol>


<output_format>
Structure as a research report:
1. Executive Summary (200-300 words)
2. Methodology and Scope
3-7. [Analytical sections with descriptive headers]
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
counter-arguments. Note perspectives that may be underrepresented
due to language or access barriers.
</bias_awareness>
```

### Level 4 Prompt Template

```
You are a principal research scientist leading an exhaustive investigation.
This should meet the rigor of a systematic literature review.


<topic>[USER TOPIC]</topic>


<phase_1_reconnaissance>
1. Map the full landscape of this topic:
   - Identify ALL major sub-domains, stakeholders, and perspectives
   - Discover the key debates, open questions, and recent breakthroughs
   - Chart the historical evolution and current trajectory
2. Generate a comprehensive research plan with 8-12 investigation areas
3. For each area, identify:
   - The 3 most important questions to answer
   - Expected source types (academic, industry, government, expert)
   - Known gaps in available evidence
</phase_1_reconnaissance>


<phase_2_deep_investigation>
For each investigation area:
1. Conduct exhaustive search across academic databases, official
   documentation, technical reports, and expert commentary
2. For each finding, document: the claim, supporting evidence,
   source quality (peer-reviewed/primary/secondary), publication date,
   and confidence level
3. Identify contradictions between sources and investigate the root cause
4. Include quantitative data wherever available
5. Seek out non-obvious connections between sub-domains
</phase_2_deep_investigation>


<phase_3_verification>
Apply full verification protocol:
1. Chain-of-Verification: for every major claim, independently verify
2. Self-consistency: where possible, triangulate from 3+ independent sources
3. Steelmanning: for contested points, construct the strongest version
   of each position before evaluating
4. Temporal check: flag any time-sensitive claims with source dates
5. Citation integrity: verify every reference exists. Remove any you
   cannot confirm
6. Confidence calibration: rate each section's conclusions as
   HIGH / MEDIUM / LOW confidence
</phase_3_verification>


<phase_4_synthesis>
1. Identify emergent patterns across investigation areas
2. Construct a novel analytical framework or taxonomy if appropriate
3. Map the current consensus, active debates, and frontier questions
4. Produce actionable recommendations grounded in evidence
</phase_4_synthesis>


<source_criteria>
- Target 50-200+ sources across: peer-reviewed papers, official
  documentation, technical reports, expert analyses, primary data
- Prioritize 2024-2025 publications; include foundational earlier work
- Include non-English sources where relevant
- Systematically document search strategy
- Today's date: [DATE]
</source_criteria>


<output_format>
Structure as a comprehensive research study:
1. Executive Summary (500 words)
2. Introduction: scope, research questions, significance
3. Methodology: search strategy, inclusion criteria, analytical approach
4-11. [Analytical sections, each with sub-sections]
12. Cross-cutting synthesis: emergent patterns and novel frameworks
13. Gap analysis: what remains unknown or under-researched
14. Limitations of this study
15. Conclusion: key findings, actionable insights, future directions
16. Annotated key resources (15-25 most important sources)
- Total: 8,000-20,000 words
- Full markdown with H2/H3/H4 hierarchy
- Embedded tables, comparison matrices
- Bold key findings throughout
- Inline citations with URLs
</output_format>


<bias_and_coverage>
- Present at least 3 competing perspectives for every contested point
- Identify what perspectives your analysis underweights
- Check for geographic, linguistic, and cultural coverage gaps
- Note where your sources are dominated by a single viewpoint
</bias_and_coverage>
```

---

## 5. Agnosticity Principles

### XML as Universal Format

- Prompt formatting affects performance by **up to 40%** depending on model (arXiv:2411.10541)
- XML tags are the **only structural format explicitly endorsed by all three major providers** (Anthropic, OpenAI, Google)
- Claude: specifically trained with XML in its data
- OpenAI (o1, o3): explicitly recommend "XML tags and section headings to clearly indicate distinct parts"
- Gemini: processes XML effectively
- Llama-3.1 405B: XML **consistently outperformed** other formats for complex prompts (even though not XML-trained)
- Token cost trade-off: XML consumes **40-60% more tokens** than JSON for equivalent structures, but higher accuracy requiring fewer iterations often means net cost favors XML
- **Recommendation**: Use XML tags for structural elements (role, task, sources, output format, verification) and markdown within those tags for content formatting

### 7 Universal Prompt Techniques

Work reliably across Claude, GPT-4/o3, Gemini, Llama, and Mistral:

1. Clear, explicit instructions stating the exact task
2. Few-shot examples showing desired input-output pairs
3. Role/persona assignment via "You are a [role]"
4. Structured output requests (JSON, tables, lists)
5. Chain-of-thought via "think step by step" (though marginal value is decreasing)
6. Context separation using any delimiter format
7. Constraint specification for length, format, and style

These form the **portable core** of any generated research prompt.

### Model-Specific Adaptations

| Model | Best Prompt Style |
|-------|-------------------|
| Claude | XML-structured prompts with `<thinking>` tags for extended reasoning |
| GPT models | Markdown-formatted with explicit planning instructions ("Before responding, create a plan") |
| Gemini | Persona-Task-Context-Format framework, long-context optimization |
| Open-source (Llama, Mistral) | Match prompt structure to specific chat templates |

### Position Bias Data

- **Qwen**: performs better with relevant context **toward the end**
- **Llama**: performs better with relevant context **toward the top**
- **OpenAI/Anthropic models**: better with relevant examples **first**
- **Mitigation**: Duplicate critical instructions at both beginning and end of generated prompts

### Context Window Considerations

- Gemini 2.5 Pro: 1-2M tokens
- Claude Sonnet 4: 200K-1M tokens
- Most models' performance **degrades well before advertised limits**
- **"Lost-in-the-Middle" phenomenon persists** — models retrieve information from beginning and end more reliably than middle
- **Strategy**: Front-load critical instructions, place source material in the middle

### DSPy/Instructor/Outlines for Portability

- **DSPy**: Abstracts away prompts entirely. Signatures define `input -> output` specs, compiler optimizes per model. Most promising path to true portability.
- **LiteLLM**: Unified interface across 100+ providers — write once, deploy across models.
- **Instructor** (3M+ monthly downloads, 15+ providers): Solves output portability via Pydantic schema enforcement.
- **Outlines**: Guarantees valid output by modifying logits per-token — output portability even when input prompts differ.

---

## 6. Anti-Hallucination & Verification Strategies

### Hallucinated Citations

- GPTZero analysis of NeurIPS 2025: **at least 100 accepted papers with confirmed hallucinated citations** ("vibe citing")
- SPY Lab analysis of arXiv: accelerating trend in hallucinated references starting early 2025, coinciding with Deep Research tool releases
- JMIR Mental Health study: fabrication rates vary by topic — **6%** for well-studied disorders but **28-29%** for less-studied ones
- **Hallucination risk is inversely proportional to topic popularity in training data**
- **Solution**: Explicit anti-hallucination instructions: "Only cite sources you can verify exist. If you cannot confirm a reference's existence, omit it and note the gap." CoVe protocol **more than doubles precision**. Level 3-4: running verification log.

### Sycophancy

- Sharma et al. (arXiv:2310.13548): sycophancy is **"a general behavior of state-of-the-art AI assistants"** driven by RLHF training that rewards user-agreeable responses
- 2025 study: **up to 100% initial compliance** with medically illogical requests across five frontier models
- Sycophancy is **NOT correlated with model size** — bigger models are not necessarily less sycophantic
- **Solution**: Steelmanning instructions: "Identify the stance you inferred from my prompt. Now make the strongest argument for the opposite view with equal detail. Note what evidence you omitted in the first version." Higher levels: "present at least 3 competing perspectives for every contested point" + "identify what perspectives your analysis underweights."

### Temporal Blindness

- GPT-4 achieves only **14% accuracy** on fast-changing questions in FreshQA benchmark
- LLMs can hallucinate about events beyond training period while declaring overly conservative cutoff dates
- Perplexity observed presenting outdated sources as current answers
- **Solution**: Every generated prompt must include current date + explicit temporal instructions: "Today's date is [DATE]. For time-sensitive claims, cite the source date. Distinguish between established facts unlikely to change, current data that may have been updated, and evolving situations requiring the latest information."

### Shallow Coverage / False Depth

- Simon Willison: Deep Research tools provide "a misleading impression of the quality of the 'research' that took place" — essentially a "presentation layer"
- LLMs tend to repeat the same information from multiple angles rather than investigating genuinely different facets
- **Solution**: STORM's perspective-discovery approach. Instruct LLM to identify distinct perspectives, research each independently, then synthesize. Higher levels: "coverage check: after completing your research, identify what important perspectives or information you may have missed" + "availability bias check: what perspectives are likely underrepresented in your training data?"

### CoVe (Chain-of-Verification) Protocol Detail

From Meta/FAIR (ACL Findings 2024, arXiv:2309.11495):
1. Draft response
2. Generate verification questions for each factual claim
3. Answer verification questions independently (without referencing draft)
4. If contradicted, revise or flag as uncertain
- Result: **more than doubles precision** on factual claims

### Decreasing Value of Chain-of-Thought

- Ethan Mollick et al. (Wharton, June 2025, "Prompting Science Report 2")
- CoT's advantage **diminishes with newer reasoning models** (o3, Claude with extended thinking)
- Models internalize reasoning — explicit step-by-step instructions become less necessary
- Does NOT eliminate CoT's value for research decomposition, but marginal returns are decreasing

---

## 7. Unique Insights

### "Prompt quality > model choice"

**The single most non-obvious insight**: Anthropic's finding that token usage explains **80%** of research quality variance, combined with APE's demonstration that LLM-generated prompts outperform human ones, means the meta-prompt framework sits at the **highest-leverage point** in the research pipeline. A well-constructed Level 3 prompt driving a mid-tier model will consistently outperform a poorly constructed prompt driving a frontier model.

### Three Competitive Advantages Most Tools Miss

1. **Automatic depth calibration** — Rather than asking users to choose a level, the system should analyze the query and recommend/auto-select the appropriate depth. No consumer tool currently offers granular middle-depth options.
2. **Embedded verification that scales with depth** — From simple fact-checking at Level 1 to full CoVe with self-consistency at Level 4.
3. **True model agnosticism** via core-plus-adapter pattern — XML structural tags as universal scaffold with model-specific formatting at deployment time.

### 80% Variance from Tokens

- Three factors explain **95% of performance variance** in Anthropic's research system: token usage (80%), tool calls, model choice
- This means **how much the model works** (prompted by the quality of instructions) matters far more than **which model does the work**

### LLMs Are Better Prompt Engineers Than Humans

- APE: LLMs outperform humans on **24 out of 24** instruction-induction tasks
- InstructGPT IQM: 0.810 vs. human 0.749
- Implication: the framework should use LLMs to generate the research prompts themselves (meta-prompting)

### Decreasing CoT Value

- As models internalize reasoning, explicit CoT yields diminishing returns
- But CoT remains valuable for research **decomposition** (not for simple reasoning tasks)

### Hallucination Risk Inversely Proportional to Topic Popularity

- Well-studied topics: ~6% fabrication rate
- Less-studied topics: 28-29% fabrication rate
- Implication: verification requirements should scale not just with depth level but with topic obscurity

### Multi-Agent Outperforms Single-Agent by 90.2%

- From Anthropic's engineering data
- Parallel tool calling cut research time by up to 90%
- Validates the orchestrator-worker architecture for research tasks

### The Gap in the Market

"Existing tools optimize for their own model, while this framework optimizes the prompt for any model." The configurable meta-layer is the unsolved problem.

---

## 8. Annotated Resource List (All 20)

### Foundational Architectures (4)

1. **GPT Researcher** — github.com/assafelovic/gpt-researcher — Most complete open-source research agent with multi-LLM support and configurable depth.
2. **STORM** — github.com/stanford-oval/storm, arXiv:2402.14207 — Most academically rigorous approach, with perspective-guided question generation and DSPy-based modularity.
3. **LangChain open_deep_research** — github.com/langchain-ai/open_deep_research — Production-grade supervisor-researcher architecture ranked #6 on Deep Research Bench.
4. **CrewAI** — github.com/crewAIInc/crewAI — Most intuitive role-based agent orchestration.

### Meta-Prompting and Automatic Prompt Engineering (5)

5. **"Large Language Models Are Human-Level Prompt Engineers"** — Zhou et al., arXiv:2211.01910, ICLR 2023 — Established that LLMs outperform humans at prompt engineering (24/24 tasks, IQM 0.810 vs 0.749).
6. **"Meta-Prompting: Enhancing Language Models with Task-Agnostic Scaffolding"** — Suzgun & Kalai, arXiv:2401.12954 — Demonstrated conductor-expert pattern with 17.1% improvement over standard prompting.
7. **OPRO** — Yang et al., arXiv:2309.03409, ICLR 2024 — Trajectory-based prompt optimization outperforming human prompts by up to 50% on Big-Bench Hard.
8. **PromptBreeder** — arXiv:2309.16797, ICML 2024 — Self-referential evolutionary prompt optimization.
9. **DSPy** — github.com/stanfordnlp/dspy, dspy.ai — Most practical framework for programmatic, model-agnostic prompt optimization. MIPROv2 Bayesian optimizer.

### Prompting Techniques for Research (4)

10. **Self-Ask** — Press et al., arXiv:2210.03350 — Foundational pattern for compositional question decomposition with search integration.
11. **Chain-of-Verification (CoVe)** — arXiv:2309.11495, ACL Findings 2024 — More than doubles factual precision through independent claim verification.
12. **The Prompt Report** — June 2024 — Surveys 50+ text-based prompting techniques systematically.
13. **Anthropic's multi-agent research engineering blog** — June 2025 — Most detailed practitioner guide to building research agent systems. Multi-agent outperforms single-agent by 90.2%.

### Official Documentation and Expert Commentary (4)

14. **Anthropic's prompt engineering docs** — docs.anthropic.com/en/docs/build-with-claude/prompt-engineering — Authoritative vendor-specific guidance.
15. **OpenAI's GPT-4.1 prompting guide** — cookbook.openai.com — Authoritative vendor-specific guidance.
16. **Lilian Weng's "LLM Powered Autonomous Agents"** — lilianweng.github.io, June 2023 — Defined canonical agent framework (LLM + Planning + Memory + Tools).
17. **Simon Willison's blog** — simonwillison.net — Most candid and practically grounded analysis of research AI limitations.
18. **Ethan Mollick's "Prompting Science Report 2"** — June 2025, SSRN — Documents decreasing value of explicit CoT with newer models.

### Cross-Model Compatibility (3 — total is 20)

19. **Prompt formatting impact paper** — arXiv:2411.10541 — Quantifies up to 40% performance variation from format alone.
20. **Instructor** — github.com/instructor-ai/instructor — 3M+ monthly downloads, solves output portability across 15+ providers. (Outlines — github.com/dottxt-ai/outlines — also mentioned as complementary for logit-level output enforcement.)

*Note: MCP (Model Context Protocol) under Linux Foundation via AAIF, 97M+ monthly SDK downloads, is also referenced as the only successful cross-model standard for tool connectivity (but not prompt formatting).*

---

## Summary Statistics (Key Numbers Preserved)

| Metric | Value | Source |
|--------|-------|--------|
| Multi-agent vs single-agent improvement | 90.2% | Anthropic |
| Token usage explains quality variance | 80% | Anthropic |
| Three factors explain variance | 95% | Anthropic |
| Parallel tool calling time reduction | up to 90% | Anthropic |
| STORM organization improvement | 25% | STORM paper |
| STORM coverage improvement | 10% | STORM paper |
| Co-STORM user preference | 70% | Co-STORM |
| LangChain RACE score | 0.4344 (#6) | Deep Research Bench |
| CrewAI vs LangGraph speed | 5.76x faster | CrewAI benchmarks |
| LangGraph monthly downloads | 90M | LangGraph |
| OpenAI HLE score | 26.6% | Humanity's Last Exam |
| DeepSeek R1 HLE score | 9.4% | Humanity's Last Exam |
| GPT-4o HLE score | 3.3% | Humanity's Last Exam |
| Perplexity daily queries | 200M | Perplexity |
| Perplexity median latency | 358ms | Perplexity |
| APE tasks where LLM beat human | 24/24 | Zhou et al. |
| APE LLM IQM vs human IQM | 0.810 vs 0.749 | Zhou et al. |
| OPRO improvement on GSM8K | up to 8% | OPRO |
| OPRO improvement on Big-Bench Hard | up to 50% | OPRO |
| Meta-Prompting improvement | 17.1% | Suzgun & Kalai |
| DSPy MIPROv2 accuracy gain | 46.2% -> 64.0% | DSPy |
| Prompt format performance impact | up to 40% | arXiv:2411.10541 |
| XML token overhead vs JSON | 40-60% more | Source doc |
| MCP monthly SDK downloads | 97M+ | MCP/Linux Foundation |
| Instructor monthly downloads | 3M+ | Instructor |
| NeurIPS 2025 hallucinated citations | 100+ papers | GPTZero |
| Hallucination rate (well-studied topics) | 6% | JMIR Mental Health |
| Hallucination rate (less-studied topics) | 28-29% | JMIR Mental Health |
| Sycophancy compliance rate | up to 100% | 2025 study |
| GPT-4 FreshQA fast-changing accuracy | 14% | FreshQA |
| CoVe precision improvement | more than doubles | Meta/FAIR ACL 2024 |
| Gemini context window | 1-2M tokens | Source doc |
| Claude Sonnet 4 context window | 200K-1M tokens | Source doc |
