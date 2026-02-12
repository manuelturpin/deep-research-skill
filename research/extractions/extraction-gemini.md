# Extraction — Gemini Research Doc (gemini.md)

Source: "Architecture et ingénierie des systèmes de recherche profonde : vers un framework universel et agnostique pour l'acquisition de connaissances par les agents intelligents"

---

## 1. Key Frameworks & Systems Referenced

### STORM (Stanford)
- Full name: **Synthesis of Topic Outlines through Retrieval and Multi-perspective Question Asking**
- Developed by Stanford University; one of the most complete academic references
- Models the **pre-writing** step by discovering diverse perspectives on a topic
- Simulates **conversations between a writer and a domain expert**, the expert being grounded in trusted Internet sources
- Generates **structured questions** (direct, perspective-guided, or conversational) to enrich depth and breadth of research
- Surpasses direct prompting methods that produce superficial results on niche topics
- Referenced again in the final architecture: its **perspective discovery** approach is recommended for high-depth levels

### MS-Agent (ModelScope-Agent)
- Evolution timeline: **August 2023 → February 2026** — from a simple API repository to a complex system
- Now supports: **MCP protocol** (Model Context Protocol), **DAG-based** (Directed Acyclic Graph) execution, sandboxed code execution environments
- Key innovation in **v1.6**: **Researcher/Reporter architecture** — decouples sub-agents specialized in research from those dedicated to writing
- Final reports are anchored in an **indexed and verified evidence base**
- Uses Qwen and diverse models; orchestrated via Ray

### UDR (NVIDIA — Universal Deep Research)
- Accepts a research prompt and a **natural-language strategy**
- Compiles them into **executable and auditable orchestration code**
- Key design principle: **separation between research logic and technical execution**
- Guarantees **transparency and modularity**

### Step-DeepResearch
- 32B parameter model from StepFun AI (report published January 25, 2026)
- **Internalizes** planning, information search, reflection, and professional report generation within a **single agent** (no multi-agent overhead)
- Achieves **61.42% rubric conformity** on certain rubric scales
- Cost: approximately **10x cheaper** than proprietary giants while rivaling their performance
- Built around **atomic capabilities** — specialized training on specific sub-tasks
- Technical report: arXiv 2512.20491v4

### Industrial Systems Comparison Table

| System | Reasoning Model | Architecture | Workflow Type |
|---|---|---|---|
| OpenAI Deep Research | o3-mini | Agentic asynchronous | Dynamic / Multi-step |
| Gemini Deep Research | Gemini 2.0 Flash Thinking | Plan-and-Execute | Interactive / Blueprint |
| Perplexity Pro | Sonar API / diverse LLMs | Iterative RAG | Speed-focused / Citations |
| Claude Research | Claude 3.5/4.x | Multi-Agent | Multi-angle exploration |
| MS-Agent (ModelScope) | Qwen / diverse | Orchestration via Ray | Hybrid / DAG-based |

Key architectural distinction: **Plan-and-Execute** (Gemini — generates a blueprint upfront, then executes) vs. **Iterative RAG** (Perplexity — speed-focused, citation-centric) vs. **Agentic asynchronous** (OpenAI — multi-step dynamic exploration).

---

## 2. Architecture Patterns

### Meta-Prompting Pipeline
- Core technique: use an LLM to **generate, critique, or improve other prompts**
- Cyclical/self-correcting process
- Example: if "impact of AI" is too vague, the meta-prompt generates sub-questions on health, education, finance sectors
- OpenAI uses high-intelligence models (o1-preview) to optimize prompts for lighter models (gpt-4o) → **reasoning hierarchy** where the superior model defines strategy and the inferior model executes research

### Five Structural Elements of a Universal Research Prompt

1. **Persona Assignment** — Calibrates tone and expertise (e.g., "Act as a senior technology investment analyst"). Described as "indispensable."
2. **Chain-of-Thought (CoT)** — Forces the model to decompose reasoning step-by-step before formulating search queries
3. **XML Delimiters** — Tags like `<context>`, `<instructions>`, `<search_queries>` help distinguish instructions from data, avoid contextual confusion. Anthropic explicitly recommends `<instructions>`, `<example>`, `<formatting>`. Described as "quasi-universal syntax in 2025."
4. **Multi-Perspective Simulation (MPS)** — Ask the model to identify 4-5 distinct perspectives on a problem to avoid simplistic dichotomies and enrich the search field
5. **Chain-of-Verification (CoVe)** — The AI generates verification questions on its own preliminary answers to identify weak points or unsupported assumptions

### Prompt Blueprints (Model-Agnostic Architecture)
- Separate **prompt structure** (tags, role instructions) from **model-specific parameters** (model ID, temperature, top-p)
- Include a **"model note"** at the top of the prompt for dynamic behavior adjustment without altering the main strategy body
- Fallback mechanisms: if a complex prompt fails on a specific model, the system switches to a simpler version or a different model using .env config files for API keys and base URLs

### Model-Specific Syntax Adaptations

| Model | Preferred Structure Format | Research Strengths | Prompting Recommendation |
|---|---|---|---|
| Claude | XML tags (`<tags>`) | Long documents, reflection | Contextualize early, ask for hypotheses |
| GPT | Markdown / JSON schemas | Strict formatting, code | Define schema first, add examples |
| Gemini | Scope + Sources | Multimodality, source links | Specify time range and region |

### Plan-and-Execute vs. Iterative RAG Distinction
- **Plan-and-Execute** (Gemini): Uses dynamic research plans ("blueprints") and interactive refinement. Leverages massive context window (up to **1 million tokens**) to maintain complete research history memory.
- **Iterative RAG** (Perplexity): Speed-centered, citation-focused. Quick retrieval and synthesis without extensive upfront planning.
- **Agentic Asynchronous** (OpenAI): o3-mini + multi-step web exploration + asynchronous task management
- **Multi-Agent** (Claude): Exploration from multiple angles simultaneously
- **Hybrid/DAG-based** (MS-Agent): DAG execution graph with Ray orchestration

### Progressive Pipelines
- Instead of generating all keywords at the start, the system generates queries **adaptively based on initial discoveries**
- Avoids information redundancy
- Gemini Deep Research example: recalls data from **50+ different sources** including scientific databases (arXiv) and real-time news feeds

### Researcher/Reporter Decoupling (MS-Agent v1.6)
- **Researcher sub-agents**: specialized in searching, collecting, verifying
- **Reporter sub-agents**: specialized in writing, synthesizing
- Reports anchored in an **indexed and verified evidence base**

---

## 3. Level System Design

### Source Framework
- Based on the **ResearchRubrics** benchmark
- Three original complexity axes: conceptual breadth, logical nesting, exploration level
- Extended to four axes in the matrice with the addition of **output objective**

### Complexity Matrix — Four Levels

| Level | Conceptual Breadth | Logical Nesting | Exploration | Output Objective |
|---|---|---|---|---|
| **Simple** | Single domain | One-step inference | Fully specified | Summary of 200-300 words |
| **Advanced** | 2-5 related sub-topics | 2-3 dependent steps | Moderately open | One-page report (500 words) |
| **Ultra** | >5 disjoint domains | 4+ hierarchical steps | Exploratory / Ambiguous | Study of 3-5 pages (1500-2500 words) |
| **Super Ultra** | Transdisciplinary synthesis | Recursive planning | Highly under-specified | In-depth analysis (>2500 words) |

### Level Characteristics
- **Simple**: Direct factual research (e.g., "What is the capital of X?"), single search operation
- **Super Ultra**: Requires "implicit reasoning" and "massive inter-document synthesis" — failure rates of **25% to 50%** according to benchmarks
- For high levels: system must not only collect facts but **identify contradictions between sources** and **propose resolutions or hypotheses based on cross-referenced evidence**

### Output Format Adaptation
- Simple levels → bullet lists and concise summaries
- Ultra/Super Ultra → professional report structures with dedicated sections for **hypotheses, unknowns, and trade-offs**
- Dynamic format switching between **Markdown** (human readability) and **JSON** (automated data pipeline integration)

---

## 4. Benchmark Data

### DRB-II (Deep Research Bench II) — January 2026

| Model | Info Recall (%) | Analysis (%) | Presentation (%) | Total Score (%) |
|---|---|---|---|---|
| **OpenAI-GPT-o3 DR** | 39.98 | 49.85 | 89.16 | **45.40** |
| **Gemini-3-Pro DR** | 39.09 | 48.94 | 91.85 | **44.60** |
| **Doubao Deep Research** | 34.83 | 49.43 | 83.51 | **40.99** |
| **Perplexity Research** | 33.05 | 44.47 | 79.34 | **38.58** |

**Key observations:**
- All top systems score **below 50% total** — significant gap between agents and human experts
- **Information Recall** is the weakest dimension, plateauing around **40%** — indicates persistent limitations in source coverage and precise data retrieval
- **Presentation** is consistently strong: **>85%** for top systems (Gemini-3-Pro leads at 91.85%)
- Deep analysis and complete evidence extraction remain the primary challenge for the coming year

### API Performance Comparison (Benchmarks 2024-2025)

| Tool | Average Latency | Agentic Quality Score | Key Technical Strength |
|---|---|---|---|
| **Brave Search** | 980 ms | 14.89 | Speed and index freshness |
| **Tavily** | 998 ms | 13.67 | Structured extraction and authorized sources |
| **Exa AI** | 2.9 s | 13.50 | Semantic understanding and "highlights" |
| **Perplexity** | 11+ s | 12.96 | Synthesized answers and factual verification |

### Step-DeepResearch Performance
- Model size: **32B parameters**
- Achieves **61.42% rubric conformity** on certain rubric scales
- Cost: approximately **10x cheaper** than proprietary giants
- Trained on **atomic capabilities** (specific sub-tasks)

### ResearchRubrics Benchmark
- Used **2,800+ hours** of human work
- Created **2,500 expert rubrics**
- Evaluates: **factual grounding, reasoning soundness, clarity**
- One of the most rigorous evaluation standards for deep research systems

### General Failure Rates
- Super Ultra-level tasks: **25%-50% failure rate** across current agents
- Information Recall across all systems: **capped at ~40%**

---

## 5. API & Tool Ecosystem

### Exa (formerly Metaphor)
- **Pure semantic approach** — uses neural models to identify knowledge connections
- Does NOT rank by popularity/SEO
- Particularly performant for **multi-step queries and academic research**
- Latency: 2.9s | Quality score: 13.50
- Provides "highlights" feature
- **Recommended for**: Ultra/Super Ultra depth levels where semantic understanding matters

### Tavily
- Designed as a **"source-first"** search tool
- Built-in **guardrails** — privileges credible and citable sources
- **Structured extraction** of data
- Preferred choice for **enterprise RAG systems**
- Latency: 998ms | Quality score: 13.67
- **Recommended for**: Simple to Advanced levels where credible sourcing is priority

### Brave Search
- **Fastest** API in the comparison
- Fresh index — good for current events
- Latency: 980ms | Quality score: 14.89 (highest)
- **Recommended for**: Simple level, real-time data needs

### Perplexity
- Returns **synthesized answers** (not raw search results)
- Built-in **factual verification**
- Slowest: 11+ seconds latency
- Quality score: 12.96 (lowest of the four)
- **Recommended for**: When pre-synthesized verified answers are needed

### Multi-Source Aggregation
- For Ultra/Super Ultra levels: combined use of **multiple sources** (multi-source aggregation) is necessary
- Gemini Deep Research recalls data from **50+ different sources** including arXiv and real-time news feeds

### Depth-Level to API Mapping
- **Simple**: Fast API (Brave or Tavily) sufficient
- **Advanced**: Tavily or Exa for structured/semantic results
- **Ultra / Super Ultra**: Multi-source aggregation required — combine multiple APIs

---

## 6. Anti-Hallucination & Verification Strategies

### Three Types of Factual Hallucinations in Deep Research
1. **Plausible but incorrect information**
2. **Errors in long-duration generations**
3. **Errors in complex lists**

"Red herrings" (false leads) are more frequent than creative hallucinations — the system establishes fragile links or incorporates irrelevant content due to lexical proximity with the subject.

### Five Mitigation Strategies

#### 1. Relevance Filtering
- Before extracting content from a web page, call a model to judge the page's **actual utility** (binary yes/no judgment)
- Prevents polluting the context with irrelevant text

#### 2. Clean Context Delivery
- Instead of injecting raw web pages, agents must perform **deduplication, reranking, and refinement** at each research turn
- Retain only structured **"knowledge"** (not raw text)

#### 3. Global Memory Management
- Use global memory modules that **iteratively record discoveries**
- Prevents the model from "forgetting" facts found early in the research session during final writing

#### 4. Confidence Calibration
- Agent must assign an **explicit certainty level** to each major claim
- **"Virtually certain" (>95%)**: direct evidence must be provided
- **"Speculative"**: model must identify what missing information would increase confidence

#### 5. Uncertainty Analysis
- Define how the model should proceed when encountering **contradictory information**
- Options: present all perspectives, or ask the user for clarification

### StepBack-Prompting
- Helps the agent **abstract from specific instances** to identify fundamental principles
- Produces better-grounded responses because reasoning relies on **global understanding rather than statistical co-occurrences of terms**
- Linked to data freshness: RAG framework bypasses the static nature of parametric model memory

---

## 7. Unique Insights (Not Found in Other Docs)

### ResearchRubrics Benchmark (Detailed)
- Created by University of Chicago (based on citation URL)
- **2,800+ hours of human work**, **2,500 expert rubrics**
- Three evaluation axes: factual grounding, reasoning soundness, clarity
- Became one of the most rigorous standards for deep research evaluation
- arXiv: 2511.07685v1

### Progressive Pipelines
- Alternative to "generate all keywords upfront" approach
- System generates search queries **adaptively based on initial discoveries**
- Avoids redundancy — queries evolve as the research context grows
- Distinguishes deep research systems from simple multi-query RAG

### Fan-Out Concept
- "Fan-out conceptuel" — the broad and intensive exploration of concepts during the research process
- The research prompt must integrate **task decomposition and meta-planning** components to manage fan-out
- This is what elevates research from linear retrieval to true investigative exploration

### Researcher/Reporter Decoupling (MS-Agent v1.6)
- A structural innovation: sub-agents are explicitly split into two roles
- **Researcher agents**: search, collect, verify evidence, build indexed evidence base
- **Reporter agents**: synthesize, write, structure final output
- Reports anchored in a verified, indexed evidence base
- This decoupling ensures writing quality doesn't compromise research thoroughness and vice versa

### Prompt Blueprints for Model Agnosticism
- Blueprints separate **structure** (tags, role instructions) from **model-specific parameters** (model ID, temperature, top-p)
- Enables switching between GPT-4o, Claude 3.5, Gemini 2.0, Llama 3 without rewriting research logic
- Strategic value: switch based on cost, performance, or privacy needs
- "Model note" at top of prompt for dynamic behavior adjustment

### Gemini Deep Research Specific Architecture
- Uses **dynamic research plans ("blueprints")** and interactive refinement
- Context window: up to **1 million tokens** — maintains complete research history memory
- Model: Gemini 2.0 Flash Thinking
- Recalls data from **50+ different sources**

### Output Format Dynamism
- The system must dynamically switch between Markdown (human readability) and JSON (automated pipeline integration)
- Ultra/Super Ultra levels require professional report structures with sections for **hypotheses, unknowns, and trade-offs**

### Complexity-Adaptive Architecture (Final System Design)
- Subject reception → **Complexity analysis module** (evaluates level based on query ambiguity + domain breadth)
- Level determined → **Meta-prompting orchestrator** generates optimal strategy
- High levels include **perspective discovery** (STORM-inspired) and **asynchronous research missions**
- Agnosticism maintained via abstraction layer: XML for Claude, JSON schemas for GPT
- Integrated **reflection and verification loop** (Step-DeepResearch inspired)

---

## 8. Full Citation List (30 Sources)

1. Step-DeepResearch Technical Report — arXiv, https://arxiv.org/html/2512.20491v4
2. Stanford STORM Research Project, https://storm-project.stanford.edu/research/storm/
3. stanford-oval/storm: An LLM-powered knowledge curation — GitHub, https://github.com/stanford-oval/storm
4. STORM: A New Framework for Teaching LLMs How to Prewrite Like a Researcher — Reddit, https://www.reddit.com/r/LLMDevs/comments/1lpbcnq/storm_a_new_framework_for_teaching_llms_how_to/
5. In-Depth Analysis of the Latest Deep Research Technology: Cutting... — HuggingFace, https://huggingface.co/blog/exploding-gradients/deepresearch-survey
6. Characterizing Deep Research: A Benchmark and Formal Definition — OpenReview, https://openreview.net/pdf?id=4IjL5CHKT7
7. Meta-Prompting: Unlocking AI's Power to Self-Improve — Accredian/Medium, https://medium.com/accredian/meta-prompting-unlocking-ais-power-to-self-improve-f1ee40974a4a
8. Enhance your prompts with meta prompting — OpenAI Cookbook, https://developers.openai.com/cookbook/examples/enhance_your_prompts_with_meta_prompting/
9. Advanced Prompt Engineering Techniques: Examples & Best Practices — Patronus AI, https://www.patronus.ai/llm-testing/advanced-prompt-engineering-techniques
10. Prompt Engineering in 2025: A Guide to Crafting Powerful ChatGPT Prompts — Medium, https://medium.com/@cpjrinfoandnews/prompt-engineering-in-2025-a-guide-to-crafting-powerful-chatgpt-prompts-549e9ebb3e48
11. 8 Chain-of-Thought Techniques To Fix Your AI Reasoning — Galileo, https://galileo.ai/blog/chain-of-thought-prompting-techniques
12. Use XML tags to structure your prompts — Claude API Docs, https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/use-xml-tags
13. Advanced Prompt Engineering Techniques for 2025: Beyond Basic Instructions — Reddit, https://www.reddit.com/r/PromptEngineering/comments/1k7jrt7/advanced_prompt_engineering_techniques_for_2025/
14. Implement Chain-of-Verification to Improve AI Accuracy — Relevance AI, https://relevanceai.com/prompt-engineering/implement-chain-of-verification-to-improve-ai-accuracy
15. Chain-of-Verification (CoVe): Reduce LLM Hallucinations — Learn Prompting, https://learnprompting.org/docs/advanced/self_criticism/chain_of_verification
16. Universal Deep Research — Research at NVIDIA, https://research.nvidia.com/labs/lpr/udr/
17. ResearchRubrics: A Benchmark of Prompts and Rubrics For Evaluating Deep Research Agents — arXiv, https://arxiv.org/html/2511.07685v1
18. ResearchRubrics — alphaXiv, https://www.alphaxiv.org/benchmarks/university-of-chicago/researchrubrics
19. Deep Research Bench II Evaluation — Emergent Mind, https://www.emergentmind.com/topics/deep-research-bench-ii
20. Claude vs ChatGPT vs Gemini Prompting: Best Practices (2025) — PromptBuilder, https://promptbuilder.cc/blog/claude-vs-chatgpt-vs-gemini-best-prompt-engineering-practices-2025
21. Structured Prompting Techniques: The Complete Guide to XML & JSON — Code Conductor, https://codeconductor.ai/blog/structured-prompting-techniques-xml-json/
22. Model Agnostic Prompts: Future-Proof AI Applications — PromptLayer, https://blog.promptlayer.com/model-agnostic/
23. Comparing 10 AI-Native Search APIs and Crawlers for LLM Agents — Medium/TowardsDev, https://medium.com/towardsdev/comparing-10-ai-native-search-apis-and-crawlers-for-llm-agents-ed4130d22c67
24. AI Deep Research for Content Generation — ALwrity, https://www.alwrity.com/post/ai-deep-researchers-for-content-generation
25. ResearchRubrics: A Benchmark of Prompts and Rubrics For Evaluating Deep Research Agents — ResearchGate, https://www.researchgate.net/publication/397521620_ResearchRubrics_A_Benchmark_of_Prompts_and_Rubrics_For_Evaluating_Deep_Research_Agents
26. DeepResearch Bench II: Diagnosing Deep Research Agents via Rubrics from Expert Report — arXiv, https://arxiv.org/html/2601.08536v2
27. Step-DeepResearch Technical Report — alphaXiv, https://www.alphaxiv.org/overview/2512.20491
28. A Metaprompt to improve Deep Search on almost all platforms — Reddit, https://www.reddit.com/r/PromptEngineering/comments/1kmy4tr/a_metaprompt_to_improve_deep_search_on_almost_all/
29. Retrieval Augmented Generation (RAG) for LLMs — Prompt Engineering Guide, https://www.promptingguide.ai/research/rag
30. StepFun AI Introduce Step-DeepResearch: A Cost-Effective Deep Research Agent Model Built Around Atomic Capabilities — MarkTechPost, https://www.marktechpost.com/2026/01/25/stepfun-ai-introduce-step-deepresearch-a-cost-effective-deep-research-agent-model-built-around-atomic-capabilities/
