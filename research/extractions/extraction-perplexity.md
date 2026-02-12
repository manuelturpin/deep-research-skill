# Extraction: Perplexity Deep Research

Source: `/tmp/research-docs/perplexity.md`
Extracted: 2026-02-12

---

## 1. Key Frameworks & Systems Referenced

### PromptWizard (Microsoft, 2024)
- **Type**: Agentique framework for task-aware prompt optimization
- **Mechanism**: LLM generates, critiques, and refines prompts with exploration/exploitation balance
- **Strengths**:
  - Highly structured pipeline (variant generation, critique, synthesis) applicable to many tasks
  - Works with few annotated data points and different models (including open-source)
  - Closest to meta-prompting needs: an agent that optimizes a prompt for a given task
- **Weaknesses**:
  - Focused on optimization from performance feedback on datasets, not on multi-level user-oriented research
  - Implementation complexity (agents, evaluation loops) and dependency on experimentation infrastructure
- **Relevance**: Directly inspirational for the meta-prompting engine — the generate-critique-refine loop is a core design pattern

### PromptAgent (2023)
- **Type**: Strategic planning framework for prompt optimization
- **Mechanism**: Models prompt optimization as a multi-step strategic planning problem — a sequence of actions (propose modification, test, analyze)
- **Strengths**:
  - Emphasis on explicit planning; useful for conceptualizing 4 depth levels as distinct "research plans"
- **Weaknesses**:
  - Oriented toward supervised automatic optimization rather than a lightweight portable skill
- **Relevance**: Provides the mental model of "research as planning" — each level is a different plan with different action sequences

### Automatic Prompt Optimization (APO, 2023) + Derivatives (MAPO)
- **Type**: Gradient-descent-inspired algorithms for iterative prompt improvement
- **Mechanism**: Uses "textual gradients" produced by an LLM to improve prompts through critique-revision loops
- **Strengths**:
  - Demonstrates that critique-revision loops can systematically improve any given prompt
  - Provides ideas for structuring feedback in meta-prompts (e.g., requesting explicit critiques on clarity, completeness, specificity)
- **Weaknesses**:
  - Requires test sets, so not directly applicable to interactive use — but inspirational for design
- **Relevance**: The critique dimensions (clarity, completeness, specificity) should be embedded in the skill's self-revision step

### Meta-prompting & PE2 ("Prompt Engineering a Prompt Engineer", 2024)
- **Type**: Research showing LLMs can become effective "prompt engineers" when given the right meta-prompt
- **Key Insights**:
  - The meta-prompt must itself integrate chain-of-thought reasoning about prompt design (identify gaps, improve constraints)
  - Synthetic boundary cases (Intent-Based Prompt Calibration) can automatically calibrate depth levels and boundaries between simple vs complex cases
- **Relevance**: DIRECTLY aligned — the skill IS a meta-prompt that engineers research prompts. The self-reasoning pattern is essential.

### Intent-Based Prompt Calibration (2024)
- **Type**: Technique for automatic prompt calibration using synthetic boundary cases
- **Mechanism**: Creates synthetic edge cases to calibrate prompt behavior based on user intent
- **Relevance**: Very pertinent for complexity detection and level selection — can be used to auto-calibrate the thresholds between levels

### The Prompt Report (2024-2025)
- **Type**: Systematic survey
- **Content**: Taxonomy of 58 prompting techniques with high-level best practices for SOTA LLMs
- **Relevance**: Theoretical foundation for selecting internal building blocks (CoT, decomposition, reflection, etc.)

### Existing Research Products (Perplexity, Tavily, LLM Search Modes)
- **Perplexity**: Differentiated modes — auto (quick), pro (richer), reasoning (detailed), deep research (multi-step). Each has implicit depth parameters (number of passes, source integration, synthesis granularity)
- **Tavily**: `search_depth` parameter (basic vs advanced), `max_results`, `include_raw_content` — materializes quantity/quality/latency trade-offs
- **LLM Generalists (Claude, GPT-4/4.1, Gemini)**: All publish guidelines converging on clarity, specificity, formatting, CoT, explicit constraints. Their "research" modes remain largely opaque.

---

## 2. Architecture Patterns

### Complete 7-Step Pipeline

**Step 1: Raw Subject Ingestion**
- Minimum input: free text from user
- Optional inputs: preferred language, time horizon (target period), desired level (otherwise auto-calculated)

**Step 2: Subject Analysis**
- Intent classification: informational, decisional, comparative, critical, systematic, etc.
- Complexity estimation: number of concepts, degree of ambiguity, probable need for academic sources, temporal dimension
- Implicit constraint detection: e.g., mentions of "2024-2025", "academic studies", "quick tutorial"

**Step 3: Depth Level Determination**
- Apply criteria grid (see Level System) combining subject complexity, intent, rigor requirements, time budget
- If user chose a level, respect it but allow upward escalation (e.g., simple → advanced if query is manifestly complex, with warning in generated prompt)

**Step 4: Hierarchical Sub-question Decomposition**
- Level 1: 3-5 sub-questions max, essentially parallel
- Levels 2-4:
  - Decomposition by axes (conceptual, empirical, technical, comparative aspects)
  - Hierarchy: macro-questions → dimensions → specific sub-questions
  - Explicit dependency notation (e.g., "answer B after exploring A") if target LLM supports multi-step reasoning

**Step 5: Source Criteria Specification**
- Type: academic, official docs, expert blogs, GitHub repos, etc.
- Quality: priority to journals, official docs, recognized conferences by domain
- Freshness: period filter, explicit mention in prompt (e.g., "prioritize 2024-2025, include older if foundational")
- Diversity: require divergent viewpoints and at least one "disagreement synthesis" block

**Step 6: Research Output Format Definition**
- Level-dependent but always structured: named sections, possibly simple JSON/YAML if target LLM handles it well
- Always very explicit: sections, approximate length, expected tables, etc.

**Step 7: Internal Meta-prompting**
- The skill must itself use a reasoning canvas to generate the final prompt:
  - Step 7a: Reformulate user's objective
  - Step 7b: Propose a decomposition
  - Step 7c: Define source and validation criteria
  - Step 7d: Assemble final prompt, then self-critique and self-revise

### Essential Components of the Produced Research Prompt (7 sections)

1. **Role/Persona**: "You are a research assistant specialized in [domain], tasked with planning and executing a structured documentary research, with precise citations."
2. **Task**: "Your task is NOT to directly answer the question, but to design and execute a web research strategy to gather the best sources on: [reformulated subject]."
3. **Intent & Depth Level**: Explicitly describe: "The objective is [understand globally / compare / produce a synthetic literature review / etc.], at [simple/advanced/ultra/super-ultra] level as defined below."
4. **Decomposition Plan**: Numbered list of sub-questions, grouped by axes (concept, state of the art, comparisons, applications, limits...)
5. **Research Criteria**: Privileged/avoided source types, time period, languages, geographic/epistemic diversity. Inline citation requirements, mention of key sources if user already provided some.
6. **Verification & Contradiction Management**: Require: explicit identification of divergence points between sources, cross-checking, mention of research limits (data gaps, uncertainties).
7. **Expected Output Format**: Level-dependent but always very explicit.

### Data Flow
```
User Query → [Ingestion] → [Analysis (intent + complexity)] → [Level Determination]
→ [Sub-question Decomposition] → [Source Criteria] → [Output Format]
→ [Meta-prompting: assemble + critique + revise] → Final Research Prompt
```

---

## 3. Level System Design

### Objective Depth Criteria (5 dimensions)
1. **Subject complexity**: number of concepts, interdisciplinarity
2. **Objective**: overview, analysis, decision, quasi-systematic review
3. **Source count/type**: blogs vs academic articles, official docs, repositories
4. **Reasoning budget**: number of research passes, multi-step, verification
5. **Output structure**: from simple summary to detailed study with standardized sections

### Level 1 — Simple
- **Objective**: Obtain a quick and reliable overview
- **Sources**: 3-5, generalist or official docs, priority to clear summaries
- **Process**: 1-2 queries, no heavy decomposition, light verification (avoid obvious contradictions)
- **Output format**: Short structured summary (5-10 sentences or 5 bullet points), 3-5 commented links
- **Example template**:
  - 1 context paragraph
  - 3-5 "key points" bullet points
  - 3-5 links with one-sentence comments

### Level 2 — Advanced
- **Objective**: Understand main axes, variants and debates
- **Sources**: 6-10, mixed (expert blogs, official docs, some recent papers)
- **Process**: Decomposition into 2-3 axes, multiple queries, explicit position comparison
- **Output format**: Thematic sections, mini comparative tables, 10-20 sentences, short bibliography (6-10 sources)
- **Example template**:
  - Sections: Context, Main Axes, Points of Vigilance, Mini-FAQ
  - 1 table (e.g., comparison of 3-4 approaches or tools)

### Level 3 — Ultra
- **Objective**: In-depth analysis, decision or technical design oriented
- **Sources**: 10-20, including academic articles from last 3-5 years, technical reports
- **Process**: Multi-pass research, focus on empirical studies or technical docs, limits and bias block
- **Output format**: Structured report (Context, Research Methods, Results, Discussion, Limits), tables, 1500-2500 words
- **Example template**:
  - Standard "synthesis article" sections: Context, Research Methodology (how web research was done), Results by axis, Discussion, Limits
  - 1-2 structured tables (criteria vs tools, techniques vs advantages/limits)

### Level 4 — Super-ultra
- **Objective**: Quasi-exhaustive study / targeted literature review
- **Sources**: 20+, explicit inclusion/exclusion criteria, high proportion of academic articles
- **Process**: Explicit research plan, iterations, possible future research recommendations, rigorous bias management
- **Output format**: Complete dossier: introduction, research methodology, thematic synthesis, gaps, implications, bibliographic annexes, >2500 words
- **Example template**:
  - Includes section explaining source selection methodology (keywords, databases, periods, inclusion/exclusion criteria)
  - Possible annexes: sources classified by type and date, glossary

### Concrete Prompt Translation Examples
- Level 1: "do not use more than 5 sources"
- Level 4: "identify at least 15 academic articles since 2022"
- Level escalation: auto-upgrade from simple to advanced if query is manifestly complex, with warning

---

## 4. Prompt Templates & Techniques

### Recommended Universal Template Structure (4 pillars)
1. **Persona**: Specialized research assistant role
2. **Task**: Research/plan task (NOT direct answering)
3. **Context**: Intent, depth level, constraints, user-provided sources
4. **Format**: Explicit output structure per level

### Meta-prompting Approach (4-step self-reasoning)
1. Reformulate the user's objective
2. Propose a decomposition
3. Define source and validation criteria
4. Assemble final prompt → self-critique → self-revise

### Key Techniques Referenced
- **Chain-of-Thought (CoT)**: Explicit step-by-step reasoning in prompts; treated as optional (not a dependency) for agnosticity
- **Decomposition**: Breaking complex queries into sub-questions by axes
- **Self-reflection / Self-critique**: LLM reviews and improves its own output
- **Few-shot examples**: In-prompt examples for complex formats
- **Intent-Based Prompt Calibration**: Using synthetic boundary cases to calibrate level thresholds
- **Generate-Critique-Refine loops**: Core pattern from PromptWizard/APO
- **Agent-based prompting**: Research planner that designs a plan then executes
- **Cross-checking directives**: Requiring multi-source verification for factual claims
- **Disagreement synthesis blocks**: Explicitly requiring identification of source disagreements
- **Temporal filtering directives**: Explicit date ranges and freshness requirements in prompts

### Prompt Delimitation
- XML-like generic tags recommended: `<role>`, `<task>`, `<steps>`, `<output_format>`
- Markdown also acceptable
- Avoid proprietary syntaxes

---

## 5. Agnosticity Principles

### Universal Elements (converged across OpenAI, Google, Anthropic guides)
- Structure: persona / task / context / output format
- Explicit, unambiguous instructions, preferably as numbered lists
- Specify language, target audience, tone, expected length
- In-prompt examples (few-shots) if needed, especially for complex formats
- Generic tags (XML-like, markdown) to delimit sections without relying on proprietary syntax

### Model-Specific Sensitivities
- **System vs User prompts**: Claude follows user message instructions more faithfully than system message — put skill details in user prompt body, minimal system for general frame
- **Explicit Chain-of-Thought**: Some models/platforms filter CoT if explicitly requested, others expose it — treat reasoning steps as optional invocation, not dependency
- **Tools/Functions**: Function calling is strongly API-tied — prefer natural language ("perform multiple research queries as needed") over "call web_search tool three times"
- **Mode parameters** (Perplexity search_depth, Tavily max_results): Express needs as constraints in natural language ("prioritize research depth over speed", "do not exceed 10 sources") — let orchestrator map constraints to native parameters

### Emerging Standards
- XML-like tag structuring for prompt sections (`<role>`, `<task>`, `<steps>`, `<output_format>`) — recommended by multiple technical guides
- Agent patterns: "research agent" or "research planner" that designs a plan first, then executes (pattern found in systems like Ask Photos with Gemini)
- Common vocabulary: CoT / decomposition / self-critique — similar terms adopted across many articles and products, facilitating cross-model prompt comprehension

### Key Agnosticity Rules
1. Avoid proprietary syntax (special API-specific tags)
2. Stay within widely adopted conventions
3. Avoid mentioning functions/tools that don't exist everywhere
4. Specify behaviors in natural language rather than API parameters
5. Use generic XML-like or markdown delimiters

---

## 6. Anti-Hallucination & Verification Strategies

### Identified Traps (5 categories)

1. **Source/citation hallucinations**: Inventing plausible articles, authors, or URLs
2. **Confirmation bias & lack of diversity**: Only searching sources aligned with an implicit hypothesis
3. **Task/answer confusion**: LLM starts "answering" the question instead of building a research prompt/plan
4. **Over-confidence in a single research pass**: One general query, no refinement, no critical feedback
5. **Redundancy & lack of hierarchization**: Overlapping sub-questions, no priority, no axis grouping

### Mitigation Strategies (5 corresponding)

1. **Against source hallucinations**:
   - Require verifiable citations (complete titles, authors, year, URL)
   - Require explicit mention if info is extrapolated or sources are weak

2. **Against confirmation bias**:
   - Explicitly demand: "Identify at least 2-3 sources or arguments that contradict or nuance the dominant thesis and summarize their arguments"

3. **To maintain research task focus**:
   - In generated prompt, specify: "Do NOT write the final answer for the user; construct ONLY the research plan and queries to execute"

4. **To reinforce validation**:
   - Include cross-checking steps: "For each important fact, try to find at least two independent sources; flag facts confirmed by only one source"

5. **For data freshness**:
   - Require temporal filters ("prioritize 2024-2025 for state of the art")
   - Demand specific section: "Temporal scope and data freshness limits"

### Additional Verification Mechanisms
- Disagreement synthesis blocks between sources (at all levels, especially 3-4)
- Explicit uncertainty zones identification
- Source quality assessment (blogs vs academic vs official)
- Research methodology transparency (how was the search conducted)

---

## 7. Unique Insights

### 1. "Task vs Answer" Separation as Core Design Principle
The research explicitly frames the fundamental distinction: the skill generates prompts that instruct the LLM to **research and plan**, NOT to answer. This separation is central and must be embedded in every generated prompt. This is a subtle but critical insight — most prompt engineering focuses on getting better answers, not on generating better research processes.

### 2. Level Escalation with Warning
The system should auto-escalate depth level when a query is manifestly too complex for the selected level, but MUST include a warning in the generated prompt. This bidirectional level awareness (user preference vs detected complexity) is rarely formalized.

### 3. Intent-Based Prompt Calibration via Synthetic Boundary Cases
Using synthetic edge cases to calibrate the boundaries between levels is a sophisticated technique borrowed from PromptAgent/PE2 literature. This provides a mechanism for the skill to self-calibrate over time rather than relying on hardcoded thresholds.

### 4. Disagreement Blocks as Mandatory Research Component
Explicitly requiring a "disagreement synthesis" section is unusual — most research frameworks focus on consensus. This anti-bias mechanism is a distinctive quality feature.

### 5. Research Methodology Transparency at Higher Levels
Levels 3-4 require the output to explain HOW the research was conducted (keywords, databases, periods, inclusion/exclusion criteria). This meta-research documentation is borrowed from systematic review methodology and is rarely seen in LLM prompt design.

### 6. Agnosticity Through Natural Language Constraints
Rather than adapting to each LLM's API, the skill expresses all requirements as natural language constraints ("prioritize depth over speed", "do not exceed 10 sources") and delegates parameter mapping to the orchestrator. This is an elegant inversion — instead of the skill knowing about APIs, the APIs must interpret the skill's natural language.

### 7. Self-Critique as Mandatory Pipeline Step
The meta-prompting step (Step 7) includes mandatory self-critique and self-revision of the generated prompt BEFORE output. This is not optional or best-practice — it's architecturally baked into the pipeline.

### 8. Implicit Constraint Detection
Step 2 includes detection of implicit constraints in the user's query (mentions of date ranges, "academic studies", "quick tutorial"). This NLP-adjacent capability adds intelligence to the level determination process.

### 9. Dependency-Aware Sub-question Ordering
At levels 2-4, sub-questions can have explicit dependencies ("answer B after exploring A"), creating a directed graph of research activities rather than a flat list.

### 10. Cross-Domain Validation Patterns
The research draws validation patterns from medical (scoping reviews), biomedical (BioASQ), and annotation literature — applying domain-specific rigor standards to general-purpose research prompt design.

---

## 8. Annotated Resource List

### Core Frameworks & Optimization
1. **The Prompt Report: A Systematic Survey of Prompt Engineering Techniques (2025)** — Taxonomy of 58 prompting techniques with general recommendations. Excellent theoretical base for selecting building blocks (CoT, decomposition, reflection). FOUNDATIONAL.
2. **Prompt Design and Engineering: Introduction and Advanced Methods (2024)** — Structured introduction to prompt engineering including CoT, reflection, agents. Useful for designing internal templates.
3. **PromptWizard: Task-Aware Prompt Optimization Framework (Microsoft, 2024)** — Concrete agentique framework for prompt optimization via critique and synthesis. MOST INSPIRATIONAL for meta-prompting engine.
4. **PromptAgent: Strategic Planning with Language Models Enables Expert-level Prompt Optimization (2023)** — Prompt optimization as strategic planning problem. Useful for structuring pipeline into stages.
5. **Automatic Prompt Optimization with "Gradient Descent" and Beam Search (APO, 2023)** — Critique-improvement loop description. Inspires how to ask the LLM to analyze and refine a research prompt.
6. **Intent-based Prompt Calibration (2024)** — Introduces synthetic boundary cases for automatic prompt calibration by intent. VERY PERTINENT for complexity detection and level selection.
7. **Prompt Engineering a Prompt Engineer (PE2, 2024)** — Demonstrates LLM can become effective "prompt engineer" via well-structured meta-prompt. DIRECTLY ALIGNED with the skill's purpose.

### Domain-Specific Applications
8. **Prompt Engineering Paradigms for Medical Applications: Scoping Review (2024)** — Medical review emphasizing prompt structure, validation, ethical considerations. Useful for quality and verification aspects.
9. **Best Practices for Text Annotation with Large Language Models (2024)** — Standards for structured prompts, validation, reproducibility in annotation — transferable to research.
10. **Using Pretrained LLMs with Prompt Engineering to Answer Biomedical Questions (BioASQ 2024)** — Concrete 2-level system (IR + QA) based on prompts. Shows how to structure research prompts + answer prompts separately.
11. **UMass-BioNLP DermPrompt (2024)** — Naive CoT for information retrieval, sophisticated for diagnosis. Illustrates importance of distinguishing research prompts vs analysis prompts.

### Vendor Guides & Platform Documentation
12. **OpenAI Prompt Engineering Guide (2025)** — Official documentation on prompting strategies (clarity, CoT, formatting). Base for agnostic part and universal patterns.
13. **Google / Gemini Prompting Guidelines & Google Research blogs (2024-2025)** — Practical advice on prompt structuring and research agents (e.g., Ask Photos). Useful for formulating agent tasks.
14. **Anthropic prompt engineering docs & talks (2024)** — Guides on system vs user role, prompt length, structuring. IMPORTANT for Claude adaptation — details in user prompt body, not system.
15. **AWS + Anthropic Prompt Engineering Best Practices (2024)** — XML tag usage, persona definition, task clarity. Pertinent for agnostic prompt template.

### Product & API References
16. **Perplexity AI – Models and Modes (DeepWiki, 2025)** — Describes auto/pro/reasoning/deep research modes, their depth and usage. REFERENCE for designing depth levels.
17. **Tavily Search API docs (2025)** — Shows how search_depth, max_results materialize research depth. Useful for mapping levels to parameters.

### Broader Research Context
18. **Prompt Engineering – a disruption in information seeking? (2024)** — Analyzes links between prompt engineering and information retrieval. Brings "information science" perspective.
19. **Retrieval Augmented Generation or Long-Context LLMs? (2024)** — Discusses dynamic routing between RAG and long context using self-reflection. Provides leads for specifying desired research type.
20. **Community synthesis: persona, task, context, format (Reddit / blogs, 2025)** — Practico-practical syntheses on the four base elements of a good prompt. Anchors design to de facto standards.

---

## Summary Statistics
- **Total frameworks analyzed**: 7 (PromptWizard, PromptAgent, APO/MAPO, Meta-prompting, PE2, Intent-Based Calibration, The Prompt Report)
- **Pipeline steps**: 7 (Ingestion → Analysis → Level → Decomposition → Source Criteria → Output Format → Meta-prompting)
- **Depth levels**: 4 (Simple, Advanced, Ultra, Super-ultra)
- **Prompt template sections**: 7 (Persona, Task, Intent/Level, Decomposition Plan, Research Criteria, Verification, Output Format)
- **Anti-hallucination strategies**: 5 traps + 5 mitigations + additional mechanisms
- **Agnosticity principles**: 5 rules + 4 model-specific sensitivities + 3 emerging standards
- **Resources catalogued**: 20 with annotations
- **Unique insights extracted**: 10
