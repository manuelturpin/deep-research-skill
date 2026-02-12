# Building an LLM-agnostic meta-research prompt framework


## Executive summary


**The most effective architecture for a multi-level research prompt generator combines STORM’s perspective-guided decomposition, meta-prompting’s conductor-expert pattern, and automatic complexity calibration — wrapped in XML-structured prompts for maximum cross-model portability.** This matters because today’s deep research tools (OpenAI Deep Research, Gemini Deep Research, Perplexity Pro, Claude Research) all converge on the same underlying pattern — orchestrator spawning parallel sub-researchers, synthesizing results — yet no open framework exists to generate optimized prompts that drive these systems from any LLM. The gap is clear: Anthropic’s engineering data shows multi-agent research outperforms single-agent by **90.2%**, and token usage explains **80%** of quality variance, meaning prompt design directly controls research output quality.


The research landscape has matured rapidly. GPT Researcher, STORM (Stanford), LangChain’s open_deep_research, and CrewAI provide open-source reference architectures. Meta-prompting techniques — APE (Zhou et al., 2023), OPRO (Google DeepMind), PromptBreeder, and DSPy — demonstrate that LLMs generate better prompts than humans on **24 out of 24** instruction-induction tasks. Prompt formatting impacts performance by up to **40%** across models, but XML tags have emerged as the only structural format endorsed by all three major providers (Anthropic, OpenAI, Google). The Model Context Protocol (MCP), now under Linux Foundation governance with **97M+ monthly SDK downloads**, standardizes tool connectivity but not prompt formatting itself — that remains unsolved.


For depth calibration, the framework should classify queries across four levels mapped to Webb’s Depth of Knowledge: recall (quick answer, 1–5 sources), conceptual (structured overview, 5–15 sources), strategic (analytical report, 15–50 sources), and extended thinking (exhaustive study, 50–200+ sources). Each level prescribes distinct output formats, decomposition strategies, and verification requirements. Critical pitfalls include hallucinated citations (GPTZero found **100+ fabricated references** in NeurIPS 2025 accepted papers), sycophancy (up to **100% initial compliance** with illogical requests across frontier models), and temporal blind spots (GPT-4 achieves only **14% accuracy** on fast-changing questions in FreshQA). These are mitigable through Chain-of-Verification (CoVe) prompting, self-consistency sampling, and explicit temporal grounding instructions embedded directly in the generated prompts.


-----


## The research framework landscape in 2025


### Open-source systems have converged on orchestrator-worker architectures


The most mature open-source research framework is **GPT Researcher**, which uses a Plan-and-Solve architecture where a Planner Agent generates research questions, multiple Execution Agents scrape sources in parallel via `asyncio.gather()`, and an Aggregation Agent compiles cited reports. Its deep research mode adds tree-like recursive exploration with configurable depth and breadth. A multi-agent variant built with LangGraph assigns specialized roles: Chief Editor, Browser, Editor, Researcher, Reviewer, and Reviser. It supports multiple LLM providers (OpenAI, Anthropic, Gemini, Ollama) and search backends (Tavily, DuckDuckGo, Google, Exa, arXiv), making it the closest existing analog to an LLM-agnostic research framework.


**STORM** (Stanford, NAACL 2024) takes a fundamentally different approach grounded in perspective-guided question asking. Rather than decomposing a topic directly, STORM discovers diverse perspectives by analyzing similar Wikipedia articles, then simulates conversations between a “Wikipedia writer” and “topic expert” grounded in internet sources. This produces **25% better article organization** and **10% broader coverage** than baselines. Built on DSPy, its modular architecture — Knowledge Curation → Outline Generation → Article Generation → Article Polishing — makes it the most academically rigorous framework. Co-STORM adds human-in-the-loop steering through a dynamic mind map, with **70%** of volunteer users preferring it over traditional search.


**LangChain’s open_deep_research** ranks **#6 on the Deep Research Bench Leaderboard** (RACE score: 0.4344). Its supervisor-researcher architecture includes a clarification phase, research brief generation, parallel sub-topic research in dedicated context windows, and synthesis. LangGraph, the underlying engine, supports three orchestration patterns (single agent, multi-agent, hierarchical) and has achieved **90M monthly downloads** with enterprise adoption at Uber, JP Morgan, and LinkedIn.


**CrewAI** offers an intuitive role-based design that mirrors human research teams, with agents assigned roles, goals, and backstories. Its dual architecture — Crews for autonomous collaboration plus Flows for deterministic orchestration — and sophisticated memory management (short-term, long-term, entity, contextual) make it practical for structured research workflows. Benchmarks show it running **5.76× faster** than LangGraph on certain tasks.


### How the major LLMs handle deep research internally


The four leading deep research implementations reveal distinct architectural philosophies, all of which a meta-prompt framework must accommodate.


**OpenAI Deep Research** uses end-to-end reinforcement learning on an optimized o3 model trained specifically for multi-step research planning. It asks clarification questions, browses the web autonomously for 5–30 minutes, analyzes text, images, and PDFs, and produces comprehensive cited reports. It scored **26.6%** on Humanity’s Last Exam (versus DeepSeek R1’s 9.4% and GPT-4o’s 3.3%). The architecture is opaque — a single monolithic agent rather than explicit multi-agent orchestration.


**Claude’s research system** (detailed in Anthropic’s June 2025 engineering blog) uses an explicit multi-agent architecture: a Lead Agent (Claude Opus 4) plans research and spawns subagents, Subagents (Claude Sonnet 4) execute parallel information gathering with interleaved thinking, and a Citation Agent processes attribution. Three factors explain **95% of performance variance**: token usage (80%), tool calls, and model choice. Parallel tool calling cut research time by **up to 90%**. Anthropic identified eight key principles for research agents, including “teach orchestrator how to delegate,” “scale effort to query complexity,” and “start wide, then narrow down.”


**Gemini Deep Research** generates an autonomous multi-step plan and presents it to the user for review before executing. This plan-approve-execute cycle gives users more control but can miss nuanced angles. It emphasizes breadth and speed over analytical depth, and uniquely integrates Google Workspace (Gmail, Drive, Chat, Sheets). It uses a **1-million-token context window** plus RAG for session memory.


**Perplexity Deep Research** runs a multi-stage retrieval-reasoning-refinement cycle: query decomposition into subtopics, dedicated searches per subtopic (hybrid lexical + semantic retrieval), partial answer synthesis, conflict verification, and final narrative with citations and reliability notes. With **200 million daily queries** and median latency of **358ms**, it optimizes for speed, completing most research in under 3 minutes.


|Feature      |OpenAI                 |Claude                         |Gemini              |Perplexity              |
|-------------|-----------------------|-------------------------------|--------------------|------------------------|
|Architecture |RL-trained single agent|Multi-agent orchestrator-worker|Plan-approve-execute|Iterative retrieval loop|
|Typical speed|5–30 min               |1–3 min                        |2–5 min             |<3 min                  |
|Multimodal   |Text, images, PDFs     |Text-focused                   |Text only           |Text + code             |
|User control |Clarification questions|Thinking visible               |Plan reviewable     |Steps visible           |


### Prompt engineering best practices are shifting beneath our feet


A critical finding from Ethan Mollick et al. (Wharton, June 2025) documents **the decreasing value of Chain-of-Thought prompting** as models internalize reasoning. Their “Prompting Science Report 2” shows CoT’s advantage diminishes with newer reasoning models (o3, Claude with extended thinking). This doesn’t eliminate CoT’s value for research decomposition, but it means explicit step-by-step instructions become less necessary as models improve.


The most research-relevant prompting techniques remain **Self-Ask** (Press et al., 2023), which decomposes complex questions into sub-questions answerable individually with search; **Skeleton-of-Thought** (ICLR 2024), which generates an outline first then expands sections in parallel (directly applicable to research report generation); and **Tree of Thoughts** (NeurIPS 2023), which maintains parallel reasoning branches with backtracking capability. Plan-and-Solve prompting (Wang et al., 2023) directly inspired GPT Researcher’s architecture.


Anthropic’s prompt engineering hierarchy provides the clearest practitioner framework: (1) be clear and direct, (2) use multishot examples, (3) chain of thought, (4) XML-style thinking tags, (5) role assignment. Their key insight: **“If your prompt isn’t clear or lacks examples, no amount of clever phrasing will save it.”**


-----


## Recommended architecture for the research prompt generator


### The five essential components every generated prompt must contain


Based on synthesis across Anthropic’s documentation, OpenAI’s prompting guides, STORM’s architecture, and meta-prompting research, a high-performing research prompt requires these components in order:


**1. Role and context frame.** Define who the LLM is (domain expert, research analyst) and provide the research context (topic, background, audience). Anthropic’s data shows Claude matches the tone and register of the prompt itself — academic phrasing produces academic output. This component anchors all subsequent instructions.


**2. Decomposition scaffold.** Structure the topic into hierarchical sub-questions using the Self-Ask pattern. STORM’s perspective-guided approach — discovering diverse viewpoints first, then generating questions per perspective — produces **25% better organization** than direct decomposition. The prompt should instruct the LLM to identify 3–5 major facets, generate 2–4 specific questions per facet, and flag follow-up questions as they emerge.


**3. Source criteria and verification protocol.** Specify acceptable source types (peer-reviewed, primary, authoritative), recency requirements, and cross-validation instructions. Embed Chain-of-Verification (CoVe) steps directly: “For each factual claim, generate a verification question. Answer it independently. If contradicted, revise or flag as uncertain.” This alone **more than doubles precision** on factual claims per the Meta/FAIR paper (ACL Findings 2024).


**4. Output format specification.** Define the exact structure, length, citation format, and level of analysis expected. Match format to depth level (see Level System below). Include explicit formatting instructions using markdown headers, tables, and bold for key findings. Research shows models produce dramatically better-structured output when given concrete format examples.


**5. Constraint and scope boundaries.** Define temporal bounds, topical limits, depth expectations, and explicit anti-hallucination instructions (“Only cite sources you can verify exist. If unsure, say so rather than fabricating details”). Include the current date for temporal grounding. Specify what the LLM should do when uncertain: flag rather than guess.


### How meta-prompting techniques should drive prompt generation


The framework should use the LLM itself to generate optimized research prompts — a meta-prompting approach validated by multiple papers. **APE** (Zhou et al., 2023, ICLR) demonstrated that LLMs achieve **human-level prompt engineering** on 24/24 instruction-induction tasks, with InstructGPT outperforming human prompts (IQM 0.810 vs. 0.749). **OPRO** (Google DeepMind, ICLR 2024) showed optimized prompts outperform hand-designed ones by **up to 8% on GSM8K** and **up to 50% on Big-Bench Hard**. **Meta-Prompting** (Suzgun & Kalai, 2024) achieved a **17.1% improvement** over standard prompting by treating a single LLM as both orchestrator and panel of diverse experts.


The practical implementation should follow a three-step meta-prompting pipeline: (1) the user provides a raw topic, (2) the framework prompts an LLM to analyze complexity, identify perspectives, and decompose the topic, (3) the framework assembles an optimized prompt by selecting components matched to the detected depth level. This mirrors STORM’s approach but operates at the prompt-generation level rather than the research-execution level.


**DSPy** (Stanford, ICLR 2024) offers the most promising path to automatic prompt optimization. Its MIPROv2 optimizer uses Bayesian optimization over instructions and few-shot demonstrations, raising accuracy from **46.2% to 64.0%** on prompt evaluation tasks. For the research framework, DSPy signatures could define the prompt-generation pipeline declaratively, with the compiler optimizing per target model.


### Automatic depth calibration through complexity signals


The framework must detect topic complexity to select the appropriate depth level. Based on Query Performance Prediction research and the emerging Retrieval Complexity metric (arXiv:2406.03592), practical calibration should combine three signal types.


**Structural signals** from the query itself: entity count, presence of comparison operators (“vs,” “compare”), temporal qualifiers (“evolution,” “recent”), scope indicators (“comprehensive,” “brief”), and question type (factual vs. analytical vs. evaluative). These map directly to Bloom’s taxonomy levels — “What is X?” triggers recall depth, while “Design a new approach combining X and Y” triggers creation depth.


**Retrieval-based signals**: how many documents are needed to answer (multi-hop indicator), diversity of relevant source types, and contradiction rate across initial sources. High contradiction signals a topic requiring deeper analysis.


**LLM-based assessment**: prompt the LLM to estimate query complexity before generating the research prompt. Count the number of sub-questions generated during decomposition — more sub-questions indicate higher complexity. Research on LLM uncertainty as a difficulty proxy (Zotos et al., 2024–2025) shows first-token probability and entropy correlate with question difficulty.


A practical complexity classifier assigns a score on four dimensions: **breadth** (number of distinct sub-topics), **depth** (logical nesting and reasoning hops required), **ambiguity** (open-endedness of the query), and **temporality** (how time-sensitive the information is). The composite score maps to one of four depth levels.


-----


## Four depth levels with criteria, formats, and examples


### Level 1 — Simple: quick factual synthesis


**When to trigger:** Single-entity queries, factual lookups, definitions, straightforward “what is” questions. Complexity score maps to Webb’s DOK Level 1 (Recall & Reproduction) and Bloom’s Remember/Understand. Detected when query contains fewer than 2 entities, no comparison operators, and no analytical verbs.


**Generated prompt characteristics:** Role assignment as “concise research assistant,” single-pass search instruction, 1–5 source requirement, no decomposition needed, direct answer format. Verification limited to “confirm the answer from at least one additional source.”


**Output format:** Direct answer in 100–500 words. One paragraph of context, the answer, and 1–5 inline citations. No sections, no tables unless comparing 2–3 items. Equivalent to Perplexity Quick Search or standard ChatGPT response.


**Example generated prompt:**


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


### Level 2 — Advanced: structured analytical overview


**When to trigger:** Multi-facet queries requiring comparison, explanation of mechanisms, or moderate analysis. Maps to Webb’s DOK Level 2–3 and Bloom’s Apply/Analyze. Detected when query involves 2–4 entities, comparison operators, or “how/why” questions requiring multiple perspectives.


**Generated prompt characteristics:** Role as “expert analyst in [domain],” topic decomposition into 3–5 sub-questions, source diversity requirement (5–15 sources, mixed types), structured output with headers and tables, basic verification (“cross-reference key claims across sources”).


**Output format:** Structured report of 1,000–3,000 words. Executive summary (3–5 sentences), 3–5 themed sections with H2 headers, comparison tables where relevant, and a conclusion with key takeaways. Inline citations throughout. Equivalent to Perplexity Pro Search or You.com Research mode.


**Example generated prompt:**


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


### Level 3 — Ultra: comprehensive analytical report


**When to trigger:** Complex multi-dimensional queries requiring critical evaluation, synthesis across domains, or extensive source analysis. Maps to Webb’s DOK Level 3–4 and Bloom’s Analyze/Evaluate. Detected when query involves 4+ entities, requires cross-domain analysis, or contains scope indicators like “comprehensive,” “in-depth,” or “analyze.”


**Generated prompt characteristics:** Role as “senior research lead specializing in [domain],” STORM-style perspective discovery (“identify 4–6 distinct expert perspectives on this topic”), hierarchical decomposition into major facets and sub-questions, rigorous source requirements (15–50 sources, preference for primary and peer-reviewed), full CoVe verification protocol, and structured output with methodology documentation.


**Output format:** Multi-section report of 3,000–8,000 words. Executive summary, methodology note, 5–8 analytical sections with H2/H3 hierarchy, embedded tables and data, critical assessment section identifying gaps and limitations, and conclusion with novel insights. All claims cited. Equivalent to OpenAI Deep Research or Gemini Deep Research output.


**Example generated prompt:**


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


### Level 4 — Super Ultra: exhaustive multi-agent research study


**When to trigger:** Maximum-depth queries requiring systematic evidence gathering, meta-analysis across domains, or comprehensive field mapping. Maps to Webb’s DOK Level 4 (Extended Thinking) and Bloom’s Evaluate/Create. Detected when queries contain explicit depth indicators (“exhaustive,” “systematic review,” “all available evidence”) or naturally require 50+ sources and multi-domain synthesis.


**Generated prompt characteristics:** Role as “principal research scientist leading a research team,” full STORM-style perspective discovery and simulated expert conversations, multi-phase research plan (reconnaissance → deep investigation → synthesis → verification → gap analysis), maximum source requirements (50–200+ sources), full CoVe + self-consistency verification, steelmanning of opposing positions, and systematic methodology documentation.


**Output format:** Comprehensive study of 8,000–20,000+ words. Structured as an academic survey: abstract, introduction with research questions, methodology, 8–12 analytical sections, synthesis creating novel frameworks, gap analysis, limitations, conclusion with actionable insights, and annotated source list. Includes tables, comparison matrices, and framework diagrams. Equivalent to a commissioned research report or systematic review.


**Example generated prompt:**


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


-----


## What works everywhere versus what requires adaptation


### XML structure is the universal safe bet for complex prompts


A critical finding from format-impact research (arXiv:2411.10541) shows prompt formatting affects performance by **up to 40%** depending on the model. Yet XML tags have emerged as the only structural format explicitly endorsed by Anthropic, OpenAI, and Google. Claude was specifically trained with XML in its data, OpenAI’s reasoning models (o1, o3) explicitly recommend “XML tags and section headings to clearly indicate distinct parts,” and Gemini processes XML effectively. Testing with Llama-3.1 405B showed XML **consistently outperformed** other formats for complex prompts — even though Llama wasn’t XML-trained.


The token cost trade-off is real: XML consumes **40–60% more tokens** than JSON for equivalent structures. But if XML produces higher-accuracy results requiring fewer iterations, the net cost often favors XML. For the research prompt framework, the recommendation is clear: use XML tags for structural elements (role, task, sources, output format, verification) and markdown within those tags for content formatting.


### Seven truly universal prompt techniques


These work reliably across Claude, GPT-4/o3, Gemini, Llama, and Mistral: (1) clear, explicit instructions stating the exact task, (2) few-shot examples showing desired input-output pairs, (3) role/persona assignment via “You are a [role],” (4) structured output requests (JSON, tables, lists), (5) chain-of-thought via “think step by step” (though its marginal value is decreasing), (6) context separation using any delimiter format, and (7) constraint specification for length, format, and style. These form the portable core of any generated research prompt.


### Model-specific adaptations the framework should handle


The framework should maintain a model-adaptation layer that adjusts generated prompts for the target LLM. Claude responds best to XML-structured prompts with `<thinking>` tags for extended reasoning. GPT models respond best to markdown-formatted prompts with explicit planning instructions (“Before responding, create a plan”). Gemini works best with the Persona-Task-Context-Format framework and long-context optimization. Open-source models (Llama, Mistral) require matching prompt structure to their specific chat templates.


**Position bias** varies across models and is particularly dangerous for research prompts. Qwen performs better with relevant context toward the end, Llama toward the top, and OpenAI/Anthropic models with relevant examples first. The framework should duplicate critical instructions at both the beginning and end of generated prompts to hedge against position bias.


**Context window limits** create practical constraints. While Gemini 2.5 Pro offers 1–2M tokens and Claude Sonnet 4 offers 200K–1M, most models’ performance degrades well before advertised limits. The “Lost-in-the-Middle” phenomenon persists — models retrieve information from the beginning and end of context more reliably than the middle. Generated prompts should front-load critical instructions and place source material in the middle.


### The “core plus adapter” pattern for practical portability


Perfect prompt portability is currently impossible — as Max Leiter (Vercel/v0 team) argues, **“Prompts overfit to models the same way models overfit to data.”** The practical solution is a two-layer design: a model-agnostic core prompt containing the research task, decomposition, source criteria, and output format, wrapped by a model-specific adapter that adjusts structural formatting, adds model-specific features (extended thinking for Claude, planning instructions for GPT), and configures parameters (temperature, token limits).


DSPy represents the most promising path to true portability because it abstracts away prompts entirely. Its signatures define only `input → output` specifications, and the compiler optimizes per model automatically. For production frameworks, using DSPy or LiteLLM (unified interface across 100+ providers) as the backend enables writing the research prompt generator once and deploying across models.


Structured output libraries — **Instructor** (3M+ monthly downloads, supports 15+ providers) and **Outlines** (guarantees valid output by modifying logits per-token) — solve the output portability problem even when input prompts differ. By defining output schemas with Pydantic, the framework can enforce consistent research output structure regardless of which LLM generates the content.


-----


## Pitfalls and their embedded solutions


### Hallucinated citations are the most dangerous failure mode


GPTZero’s analysis of NeurIPS 2025 found **at least 100 accepted papers with confirmed hallucinated citations** — a phenomenon they call **“vibe citing.”** SPY Lab’s analysis of arXiv papers found an accelerating trend in hallucinated references starting early 2025, coinciding with Deep Research tool releases. A JMIR Mental Health study found fabrication rates vary dramatically by topic: **6%** for well-studied disorders but **28–29%** for less-studied ones. This means hallucination risk is inversely proportional to topic popularity in training data.


**Embedded solution:** The generated prompt must include explicit anti-hallucination instructions: “Only cite sources you can verify exist. If you cannot confirm a reference’s existence, omit it and note the gap.” The CoVe protocol — drafting, generating verification questions, answering them independently, and revising — **more than doubles precision** on factual claims. At Level 3 and 4, the prompt should instruct the LLM to maintain a running verification log.


### Sycophancy silently corrupts research objectivity


Research from Sharma et al. (arXiv:2310.13548) established that sycophancy is **“a general behavior of state-of-the-art AI assistants”** driven by RLHF training that rewards user-agreeable responses. A 2025 study found **up to 100% initial compliance** with medically illogical requests across five frontier models. Sycophancy is not correlated with model size — bigger models are not necessarily less sycophantic.


**Embedded solution:** The generated prompt should include steelmanning instructions: “Identify the stance you inferred from my prompt. Now make the strongest argument for the opposite view with equal detail. Note what evidence you omitted in the first version.” At higher depth levels, include explicit instructions to “present at least 3 competing perspectives for every contested point” and to “identify what perspectives your analysis underweights.”


### Temporal blindness produces confidently outdated research


GPT-4 achieves only **14% accuracy** on fast-changing questions in the FreshQA benchmark. More troublingly, LLMs can hallucinate about events beyond their training period while declaring overly conservative cutoff dates. Perplexity has been observed presenting outdated sources as current answers.


**Embedded solution:** Every generated prompt must include the current date and explicit temporal instructions: “Today’s date is [DATE]. For time-sensitive claims, cite the source date. Distinguish between established facts unlikely to change, current data that may have been updated, and evolving situations requiring the latest information. If you cannot verify information is current as of [DATE], state this limitation explicitly.”


### Shallow coverage masquerades as thorough research


Simon Willison observes that Deep Research tools provide “a misleading impression of the quality of the ‘research’ that took place” — essentially a “presentation layer” that makes results appear more authoritative than warranted. LLMs tend to repeat the same information from multiple angles rather than investigating genuinely different facets.


**Embedded solution:** STORM’s perspective-discovery approach directly counters this. The prompt should instruct the LLM to first identify distinct perspectives, then research each independently, then synthesize — ensuring genuine breadth rather than rephrased repetition. At higher levels, include “coverage check: after completing your research, identify what important perspectives or information you may have missed” and “availability bias check: what perspectives are likely underrepresented in your training data?”


### A comprehensive verification meta-prompt to embed at all levels


Based on synthesized best practices across CoVe, self-consistency, and debiasing research, every generated research prompt should include a scaled version of this verification protocol:


At **Level 1**: “Confirm key facts from at least one additional source. Flag uncertainty.”


At **Level 2**: “Cross-reference key claims across 2+ sources. Rate confidence HIGH/MEDIUM/LOW for each major finding.”


At **Level 3**: Full CoVe protocol plus steelmanning plus temporal verification.


At **Level 4**: Full CoVe, self-consistency (approach key questions from multiple framings), steelmanning, temporal verification, coverage audit, and citation integrity check.


-----


## The 20 most valuable resources for building this framework


**Foundational architectures.** GPT Researcher (github.com/assafelovic/gpt-researcher) provides the most complete open-source research agent with multi-LLM support and configurable depth. STORM (github.com/stanford-oval/storm, arXiv:2402.14207) is the most academically rigorous approach, with perspective-guided question generation and DSPy-based modularity. LangChain’s open_deep_research (github.com/langchain-ai/open_deep_research) offers a production-grade supervisor-researcher architecture ranked #6 on Deep Research Bench. CrewAI (github.com/crewAIInc/crewAI) delivers the most intuitive role-based agent orchestration.


**Meta-prompting and automatic prompt engineering.** “Large Language Models Are Human-Level Prompt Engineers” by Zhou et al. (arXiv:2211.01910, ICLR 2023) established that LLMs outperform humans at prompt engineering. “Meta-Prompting: Enhancing Language Models with Task-Agnostic Scaffolding” by Suzgun & Kalai (arXiv:2401.12954) demonstrated the conductor-expert pattern with **17.1% improvement** over standard prompting. OPRO by Yang et al. (arXiv:2309.03409, ICLR 2024) showed trajectory-based prompt optimization outperforming human prompts by **up to 50%** on Big-Bench Hard. PromptBreeder (arXiv:2309.16797, ICML 2024) introduced self-referential evolutionary prompt optimization. DSPy (github.com/stanfordnlp/dspy, dspy.ai) offers the most practical framework for programmatic, model-agnostic prompt optimization.


**Prompting techniques for research.** Self-Ask by Press et al. (arXiv:2210.03350) provides the foundational pattern for compositional question decomposition with search integration. Chain-of-Verification (arXiv:2309.11495, ACL Findings 2024) more than doubles factual precision through independent claim verification. The Prompt Report (June 2024) surveys 50+ text-based prompting techniques systematically. Anthropic’s multi-agent research engineering blog (June 2025) provides the most detailed practitioner guide to building research agent systems, including the finding that multi-agent outperforms single-agent by 90.2%.


**Official documentation and expert commentary.** Anthropic’s prompt engineering docs (docs.anthropic.com/en/docs/build-with-claude/prompt-engineering) and OpenAI’s GPT-4.1 prompting guide (cookbook.openai.com) represent the authoritative vendor-specific guidance. Lilian Weng’s “LLM Powered Autonomous Agents” (lilianweng.github.io, June 2023) defined the canonical agent framework (LLM + Planning + Memory + Tools). Simon Willison’s blog (simonwillison.net) provides the most candid and practically grounded analysis of research AI limitations. Ethan Mollick’s “Prompting Science Report 2” (June 2025, SSRN) documents the decreasing value of explicit CoT with newer models.


**Cross-model compatibility.** The prompt formatting impact paper (arXiv:2411.10541) quantifies up to 40% performance variation from format alone. The Model Context Protocol specification (now under Linux Foundation via AAIF) represents the only successful cross-model standard, with 97M+ monthly SDK downloads. Instructor (github.com/instructor-ai/instructor) and Outlines (github.com/dottxt-ai/outlines) solve output portability across providers through schema enforcement.


-----


## Conclusion


The research prompt generator framework should operate as a three-stage pipeline: **detect** (analyze query complexity across breadth, depth, ambiguity, and temporality dimensions), **decompose** (use STORM-style perspective discovery and Self-Ask question generation scaled to the detected level), and **generate** (assemble an optimized prompt from modular components — role, decomposition scaffold, source criteria, verification protocol, and output format — wrapped in a model-specific adapter layer).


The most non-obvious insight from this research is that **prompt quality matters more than model choice**. Anthropic’s finding that token usage explains 80% of research quality variance, combined with APE’s demonstration that LLM-generated prompts outperform human ones, means the meta-prompt framework sits at the highest-leverage point in the research pipeline. A well-constructed Level 3 prompt driving a mid-tier model will consistently outperform a poorly constructed prompt driving a frontier model.


The framework’s competitive advantage lies in three design choices most existing tools miss. First, **automatic depth calibration** — rather than asking users to choose a level, the system should analyze the query and recommend (or auto-select) the appropriate depth, as no consumer tool currently offers granular middle-depth options. Second, **embedded verification that scales with depth** — from simple fact-checking at Level 1 to full CoVe with self-consistency at Level 4. Third, **true model agnosticism** via the core-plus-adapter pattern, using XML structural tags as the universal scaffold with model-specific formatting applied at deployment time. The gap in the market is precisely this configurable meta-layer — existing tools optimize for their own model, while this framework optimizes the prompt for any model.