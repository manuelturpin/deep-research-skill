Architecture et ingénierie des systèmes de recherche profonde : vers un framework universel et agnostique pour l’acquisition de connaissances par les agents intelligents
Le passage de la simple récupération d’informations par les systèmes de génération augmentée par la recherche (RAG) vers ce que l’industrie qualifie désormais de "Deep Research" (recherche profonde) marque un tournant technologique majeur pour la période 2024-2025. Cette évolution ne se limite pas à l'amélioration des capacités de recherche web, mais concerne une refonte totale de l'interaction entre les agents intelligents et les vastes corpus de données numériques. Le concept de recherche profonde est aujourd'hui défini comme l'aptitude d'un système d'intelligence artificielle à traiter des tâches d'acquisition d'informations ouvertes, sur un horizon long et dotées d'une complexité élevée. Cette capacité repose sur une orchestration sophistiquée de fonctions atomiques : planification adaptative, collecte d'informations, vérification croisée des sources, réflexion critique et synthèse rédactionnelle de haut niveau.
État de l’art des frameworks et des modes de recherche des agents de pointe
L’analyse des systèmes existants montre une divergence stratégique entre les approches axées sur la rapidité et celles axées sur la rigueur académique. Le framework STORM (Synthesis of Topic Outlines through Retrieval and Multi-perspective Question Asking), développé par l’Université de Stanford, demeure l’une des références académiques les plus abouties. STORM modélise l’étape de pré-écriture en découvrant diverses perspectives sur un sujet donné et en simulant des conversations entre un écrivain et un expert du domaine, ce dernier étant ancré dans des sources Internet de confiance. L'innovation de STORM réside dans sa capacité à générer des questions structurées (directes, guidées par la perspective ou conversationnelles) pour enrichir la profondeur et l’étendue de la recherche, surpassant ainsi les méthodes de prompting direct qui produisent souvent des résultats superficiels sur des sujets de niche.
Parallèlement, les acteurs majeurs du secteur comme OpenAI, Google, Anthropic et Perplexity ont déployé des solutions industrielles aux caractéristiques distinctes. OpenAI Deep Research, par exemple, combine le modèle de raisonnement o3-mini avec une exploration web multi-étapes et une gestion de tâches asynchrone. À l'opposé, Gemini Deep Research de Google privilégie l'utilisation de plans de recherche dynamiques ("blueprints") et d'un raffinement interactif, s'appuyant sur la fenêtre de contexte massive de Gemini 2.0 (jusqu'à un million de tokens) pour maintenir une mémoire complète de l'historique de recherche.
Comparaison des architectures de recherche des systèmes leaders (2024-2025)
Système
	Modèle de raisonnement
	Architecture
	Type de workflow
	OpenAI Deep Research
	o3-mini
	Agentic asynchrone
	Dynamique / Multi-étapes
	Gemini Deep Research
	Gemini 2.0 Flash Thinking
	Plan-and-Execute
	Interactif / Blueprint
	Perplexity Pro
	Sonar API / LLM divers
	Iterative RAG
	Centré sur la vitesse / Citations
	Claude Research
	Claude 3.5/4.x
	Multi-Agent
	Exploration d'angles multiples
	MS-Agent (ModelScope)
	Qwen / Modèles divers
	Orchestration par Ray
	Hybride / DAG-based
	Le framework MS-Agent (anciennement ModelScope-Agent) illustre bien cette transition technique. Entre août 2023 et février 2026, il a évolué d'un simple dépôt d'API vers un système complexe supportant le protocole MCP (Model Context Protocol), des exécutions basées sur des graphes orientés acycliques (DAG) et des environnements de "bac à sable" sécurisés pour l'exécution de code. L'introduction de l'architecture Researcher/Reporter dans sa version v1.6 permet de découpler les sous-agents spécialisés dans la recherche de ceux dédiés à la rédaction, garantissant ainsi que les rapports finaux sont ancrés dans une base de preuves indexée et vérifiée.
Architecture du prompt de recherche et mécanismes de meta-prompting
La transformation d'un sujet brut en un prompt de recherche optimisé nécessite une architecture de prompt structurée qui dépasse la simple instruction textuelle. L'ingénierie de contexte moderne identifie que l'efficacité d'un agent de recherche dépend de sa capacité à gérer le "fan-out" conceptuel, c'est-à-dire l'exploration large et intensive de concepts durant le processus de recherche. Pour atteindre ce niveau, le prompt de recherche doit intégrer des composants de décomposition de tâches et de méta-planification.
Le meta-prompting est ici la technique de base : il consiste à utiliser un LLM pour générer, critiquer ou améliorer d'autres prompts. Ce processus cyclique permet au système de s'auto-corriger. Par exemple, si une requête initiale sur "l'impact de l'IA" est jugée trop vague par le module d'analyse, le meta-prompt générera une série de sous-questions plus précises sur les secteurs de la santé, de l'éducation ou de la finance. OpenAI utilise des modèles de haute intelligence comme o1-preview pour optimiser les prompts destinés à des modèles plus légers comme gpt-4o, créant ainsi une hiérarchie de raisonnement où le modèle supérieur définit la stratégie et le modèle inférieur exécute la recherche.
Éléments structurels d'un prompt de recherche universel
Une architecture robuste de prompt de recherche pour 2025 s'appuie sur plusieurs piliers :
1. L'assignation de Persona : Indispensable pour calibrer le ton et l'expertise (ex: "Agis en tant qu'analyste senior en investissement technologique").
2. Le Chain-of-Thought (CoT) : Forcer le modèle à décomposer son raisonnement étape par étape avant de formuler ses requêtes de recherche.
3. L'utilisation de délimiteurs : L'emploi de balises XML (<context>, <instructions>, <search_queries>) permet au modèle de distinguer clairement les instructions des données et d'éviter les confusions contextuelles.
4. La simulation de perspectives multiples (MPS) : Demander au modèle d'identifier 4 à 5 perspectives distinctes sur un problème pour éviter les dichotomies simplistes et enrichir le champ de recherche.
5. Le Chain-of-Verification (CoVe) : Un mécanisme où l'IA génère des questions de vérification sur ses propres réponses préliminaires pour identifier des points faibles ou des suppositions non étayées.
Cette approche permet de passer d'un système de recherche statique à un système récursif. Comme le montre le framework UDR (Universal Deep Research) de NVIDIA, le système accepte un prompt de recherche et une stratégie en langage naturel, puis les compile en un code d'orchestration exécutable et auditable. Cette séparation entre la logique de recherche et l'exécution technique garantit la transparence et la modularité du processus.
Niveaux de profondeur et critères de distinction
Le framework doit être capable d'adapter l'intensité de son investigation à la complexité intrinsèque du sujet. Les recherches menées sur le benchmark ResearchRubrics proposent trois axes de complexité pour catégoriser les tâches : l'étendue conceptuelle, l'imbrication logique et le niveau d'exploration. Sur cette base, nous pouvons définir quatre niveaux de profondeur pour notre skill de recherche.
Matrice de définition des niveaux de profondeur de recherche
Niveau
	Étendue conceptuelle
	Imbrication logique
	Exploration
	Objectif de sortie
	Simple
	Domaine unique
	Inférence en une étape
	Entièrement spécifié
	Résumé de 200-300 mots
	Avancé
	2-5 sous-topics liés
	2-3 étapes dépendantes
	Modérément ouvert
	Rapport d'une page (500 mots)
	Ultra
	>5 domaines disjoints
	4+ étapes hiérarchiques
	Exploratoire / Ambigu
	Étude de 3-5 pages (1500-2500 mots)
	Super Ultra
	Synthèse transdisciplinaire
	Planification récursive
	Fortement sous-spécifié
	Analyse approfondie (>2500 mots)
	Le niveau "Simple" correspond à une recherche factuelle directe (ex: "Quelle est la capitale de X?"), où le modèle effectue une seule opération de recherche. À l'opposé, le niveau "Super Ultra" exige une capacité de "raisonnement implicite" et de "synthèse inter-documents" massive, des domaines où les agents actuels montrent encore des taux d'échec significatifs (entre 25 % et 50 % selon les benchmarks). Pour ces niveaux élevés, le système ne doit pas seulement collecter des faits, mais aussi identifier des contradictions entre sources et proposer des résolutions ou des hypothèses basées sur des preuves croisées.
L'adaptation du format de sortie est tout aussi cruciale. Alors que les niveaux simples privilégient les listes à puces et les résumés concis, les niveaux Ultra et Super Ultra nécessitent des structures de rapport professionnelles avec des sections dédiées aux hypothèses, aux inconnues et aux arbitrages (trade-offs). Le formatage doit être capable de basculer dynamiquement entre le Markdown pour la lisibilité humaine et le JSON pour l'intégration dans des pipelines de données automatisés.
Agnosticité LLM : Syntaxe universelle vs Spécificités des modèles
Pour que le framework soit véritablement agnostique, il doit pouvoir fonctionner avec n'importe quel modèle de langage tout en tirant parti des forces spécifiques de chacun. Cette "agnosticité" est une valeur stratégique, car elle permet de basculer entre les modèles (GPT-4o, Claude 3.5, Gemini 2.0, Llama 3) en fonction du coût, de la performance ou des besoins de confidentialité sans réécrire la logique de recherche.
L'approche recommandée consiste à utiliser des "Prompt Blueprints" (modèles de prompts standardisés). Ces blueprints séparent la structure du prompt (les balises, les instructions de rôle) des paramètres spécifiques au modèle (identifiant du modèle, température, top-p). Cependant, des ajustements mineurs restent nécessaires car les modèles n'interprètent pas les structures de la même manière.
Adaptations syntaxiques par famille de modèles
Modèle
	Format de structure favori
	Points forts de recherche
	Recommandation de prompting
	Claude
	Balises XML (<tags>)
	Documents longs, réflexion
	Contextualiser dès le début, demander les hypothèses
	GPT
	Markdown / Schémas JSON
	Formatage strict, code
	Définir le schéma d'abord, ajouter des exemples
	Gemini
	Scope + Sources
	Multimodalité, liens sources
	Spécifier la plage temporelle et la région
	L'utilisation des balises XML s'est imposée comme une syntaxe quasi-universelle en 2025. Anthropic recommande explicitement l'usage de tags comme <instructions>, <example> et <formatting> pour aider le modèle à segmenter les différentes parties du prompt. Cette pratique améliore la précision en réduisant le risque que le modèle confonde les exemples avec les instructions réelles. De plus, pour les agents de recherche, l'inclusion d'une note de modèle ("model note") au sommet du prompt permet d'ajuster dynamiquement le comportement sans altérer le corps principal de la stratégie.
Un framework agnostique doit également prévoir des mécanismes de secours (fallbacks). Si un prompt complexe échoue sur un modèle spécifique (par exemple, suite à un malentendu sur les consignes de sécurité), le système doit être capable de basculer vers une version plus simple ou vers un modèle différent en utilisant des fichiers de configuration .env pour les clés d'API et les URLs de base.
Outils de recherche et APIs : Tavily, Exa et Perplexity
L'efficacité du skill de recherche dépend intrinsèquement des outils auxquels il a accès. En 2024-2025, le paysage des APIs de recherche pour agents intelligents s'est structuré autour de solutions "AI-native" qui diffèrent des moteurs de recherche traditionnels par leur capacité à retourner des données déjà structurées et filtrées pour la consommation par des LLM.
Exa (anciennement Metaphor) se distingue par son approche sémantique pure. Au lieu de classer les pages par popularité (SEO), Exa utilise des modèles neuronaux pour identifier les connexions entre les connaissances, ce qui le rend particulièrement performant pour les requêtes multi-étapes et les recherches académiques. Tavily, quant à lui, est conçu comme un outil de recherche "source-first" avec des garde-fous intégrés, privilégiant les sources crédibles et citables, ce qui en fait un choix privilégié pour les systèmes RAG d'entreprise.
Comparaison des performances des APIs de recherche agentique (Benchmarks 2024-2025)
Outil
	Latence moyenne
	Score de qualité agentique
	Point fort technique
	Brave Search
	980 ms
	14.89
	Rapidité et fraîcheur de l'index
	Tavily
	998 ms
	13.67
	Extraction structurée et sources autorisées
	Exa AI
	2.9 s
	13.50
	Compréhension sémantique et "highlights"
	Perplexity
	11+ s
	12.96
	Réponses synthétisées et vérification factuelle
	Le choix de l'API dépend du niveau de profondeur requis. Pour un niveau "Simple", une API rapide comme Brave ou Tavily suffit. Pour les niveaux "Ultra" ou "Super Ultra", l'utilisation combinée de plusieurs sources (agrégation multi-sources) est nécessaire. Gemini Deep Research, par exemple, rappelle des données de plus de 50 sources différentes, incluant des bases de données scientifiques (arXiv) et des flux d'actualités en temps réel. L'optimisation de la recherche passe également par l'utilisation de "pipelines progressifs" : au lieu de générer tous les mots-clés au début, le système génère des requêtes de manière adaptative en fonction des premières découvertes, évitant ainsi la redondance d'informations.
Benchmarks et évaluation de la qualité de la recherche
L'évaluation des systèmes de recherche profonde a nécessité la création de nouveaux benchmarks, car les tests de QA traditionnels ne mesurent pas la capacité de synthèse à long terme. ResearchRubrics est devenu l'un des standards les plus rigoureux, utilisant plus de 2 800 heures de travail humain pour créer 2 500 rubriques expertes évaluant l'ancrage factuel, la solidité du raisonnement et la clarté.
Les résultats de 2025 montrent un écart significatif entre les agents et les experts humains. Même les systèmes les plus performants, comme OpenAI o3 ou Gemini 3-Pro, obtiennent des scores de satisfaction globale des rubriques inférieurs à 50 % sur le benchmark DRB-II (Deep Research Bench II). La faiblesse majeure reste le "rappel d'informations" (Information Recall), plafonné à environ 40 %, ce qui indique des limitations persistantes dans la couverture des sources et la récupération de données précises.
Analyse des scores de performance (Benchmark DRB-II, Janvier 2026)
Modèle
	Rappel d'info (%)
	Analyse (%)
	Présentation (%)
	Score Total (%)
	OpenAI-GPT-o3 DR
	39.98
	49.85
	89.16
	45.40
	Gemini-3-Pro DR
	39.09
	48.94
	91.85
	44.60
	Doubao Deep Research
	34.83
	49.43
	83.51
	40.99
	Perplexity Research
	33.05
	44.47
	79.34
	38.58
	Ces chiffres soulignent que si les agents sont excellents pour structurer et présenter l'information (scores de présentation > 85 %), leur capacité à produire des analyses profondes et à extraire l'intégralité des preuves pertinentes reste le défi principal de l'année à venir. Des modèles plus compacts, comme Step-DeepResearch (32B), commencent néanmoins à rivaliser avec les géants propriétaires en optimisant l'entraînement sur des capacités atomiques spécifiques, atteignant 61,42 % de conformité sur certaines échelles de rubriques à un coût dix fois moindre.
Bonnes pratiques et pièges : Hallucinations et fraîcheur des données
La recherche profonde est particulièrement vulnérable à trois types d'hallucinations factuelles : les informations plausibles mais incorrectes, les erreurs dans les générations de longue durée et les erreurs dans les listes complexes. Contrairement aux hallucinations créatives, les "red herrings" (fausses pistes) sont ici plus fréquents : le système établit des liens fragiles ou incorpore des contenus non pertinents simplement parce qu'ils partagent une proximité lexicale avec le sujet.
Stratégies d'atténuation des risques
1. Filtrage de pertinence (Relevance Filtering) : Avant d'extraire le contenu d'une page web, le système doit appeler un modèle pour juger de l'utilité réelle de la page (jugement binaire oui/non). Cela évite de polluer le contexte avec du texte non pertinent.
2. Nettoyage du contexte (Clean Context Delivery) : Au lieu d'injecter des pages web brutes, les agents doivent effectuer une déduplication, un reclassement (reranking) et un raffinement à chaque tour de recherche pour ne conserver que les "connaissances" structurées.
3. Gestion de la mémoire globale : Utiliser des modules de mémoire globale qui enregistrent itérativement les découvertes. Cela permet au modèle de ne pas "oublier" les faits trouvés au début de la session de recherche lors de la rédaction finale.
4. Calibrage de la confiance : Exiger que l'agent assigne un niveau de certitude explicite à chaque affirmation majeure. Pour les affirmations classées comme "Virtuellement certaines" (>95%), une preuve directe doit être fournie. Pour les affirmations "Spéculatives", le modèle doit identifier quelles informations manquantes aideraient à augmenter la confiance.
5. Analyse d'incertitude : Demander au modèle comment il doit procéder s'il rencontre des informations contradictoires (ex: présenter toutes les perspectives ou demander clarification à l'utilisateur).
La fraîcheur des données est assurée par l'intégration de protocoles de mise à jour en temps réel. Le framework RAG permet de contourner le caractère statique de la mémoire paramétrique des modèles. L'utilisation de techniques comme le "StepBack-prompting" aide l'agent à s'abstraire des instances spécifiques pour identifier des principes fondamentaux, ce qui conduit à des réponses mieux ancrées car le raisonnement s'appuie sur une compréhension globale plutôt que sur des cooccurrences statistiques de termes.
Vers une architecture systémique de la recherche intelligente
La conception finale de ce skill de recherche multi-niveaux doit s'articuler autour d'un flux dynamique et agnostique. La réception d'un sujet brut déclenche d'abord un module d'analyse de complexité qui évalue le niveau requis (Simple à Super Ultra) en fonction de l'ambiguïté de la requête et de l'étendue des domaines concernés.
Une fois le niveau défini, un orchestrateur de meta-prompting génère la stratégie optimale. Pour les niveaux élevés, cette stratégie inclut la découverte de perspectives (inspirée de STORM) et la planification de missions de recherche asynchrones. L'agnosticité est maintenue par une couche d'abstraction qui traduit les instructions en syntaxe XML pour Claude ou en schémas JSON pour GPT, tout en gérant les appels aux APIs de recherche comme Tavily ou Exa.
Le système doit impérativement intégrer une boucle de réflexion et de vérification. Comme le suggère le framework Step-DeepResearch, l'internalisation des capacités de planification, de recherche d'information, de réflexion et de génération de rapports professionnels au sein d'un seul agent permet d'atteindre des performances d'expert tout en minimisant les coûts opérationnels. En 2025, la réussite d'un système de recherche ne se mesure plus à la quantité de données extraites, mais à la qualité de la synthèse et à la robustesse de l'ancrage factuel, transformant ainsi l'IA d'un simple moteur de recherche en un véritable partenaire intellectuel capable de naviguer dans la complexité du savoir humain.
Sources des citations
1. Step-DeepResearch Technical Report - arXiv, https://arxiv.org/html/2512.20491v4 2. | Stanford STORM Research Project, https://storm-project.stanford.edu/research/storm/ 3. stanford-oval/storm: An LLM-powered knowledge curation ... - GitHub, https://github.com/stanford-oval/storm 4. STORM: A New Framework for Teaching LLMs How to Prewrite Like a Researcher - Reddit, https://www.reddit.com/r/LLMDevs/comments/1lpbcnq/storm_a_new_framework_for_teaching_llms_how_to/ 5. In-Depth Analysis of the Latest Deep Research Technology: Cutting ..., https://huggingface.co/blog/exploding-gradients/deepresearch-survey 6. Characterizing Deep Research: A Benchmark and Formal Definition - OpenReview, https://openreview.net/pdf?id=4IjL5CHKT7 7. Meta-Prompting: Unlocking AI's Power to Self-Improve | by Pradum Shukla | Accredian, https://medium.com/accredian/meta-prompting-unlocking-ais-power-to-self-improve-f1ee40974a4a 8. Enhance your prompts with meta prompting - OpenAI for developers, https://developers.openai.com/cookbook/examples/enhance_your_prompts_with_meta_prompting/ 9. Advanced Prompt Engineering Techniques: Examples & Best Practices - Patronus AI, https://www.patronus.ai/llm-testing/advanced-prompt-engineering-techniques 10. Prompt Engineering in 2025: A Guide to Crafting Powerful ChatGPT Prompts - Medium, https://medium.com/@cpjrinfoandnews/prompt-engineering-in-2025-a-guide-to-crafting-powerful-chatgpt-prompts-549e9ebb3e48 11. 8 Chain-of-Thought Techniques To Fix Your AI Reasoning | Galileo, https://galileo.ai/blog/chain-of-thought-prompting-techniques 12. Use XML tags to structure your prompts - Claude API Docs, https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/use-xml-tags 13. Advanced Prompt Engineering Techniques for 2025: Beyond Basic Instructions - Reddit, https://www.reddit.com/r/PromptEngineering/comments/1k7jrt7/advanced_prompt_engineering_techniques_for_2025/ 14. Implement Chain-of-Verification to Improve AI Accuracy - Relevance AI, https://relevanceai.com/prompt-engineering/implement-chain-of-verification-to-improve-ai-accuracy 15. Chain-of-Verification (CoVe): Reduce LLM Hallucinations - Learn Prompting, https://learnprompting.org/docs/advanced/self_criticism/chain_of_verification 16. Universal Deep Research - Research at NVIDIA, https://research.nvidia.com/labs/lpr/udr/ 17. ResearchRubrics: A Benchmark of Prompts and Rubrics For Evaluating Deep Research Agents - arXiv, https://arxiv.org/html/2511.07685v1 18. ResearchRubrics | alphaXiv, https://www.alphaxiv.org/benchmarks/university-of-chicago/researchrubrics 19. Deep Research Bench II Evaluation - Emergent Mind, https://www.emergentmind.com/topics/deep-research-bench-ii 20. Claude vs ChatGPT vs Gemini Prompting: Best Practices (2025 ..., https://promptbuilder.cc/blog/claude-vs-chatgpt-vs-gemini-best-prompt-engineering-practices-2025 21. Structured Prompting Techniques: The Complete Guide to XML & JSON - Code Conductor, https://codeconductor.ai/blog/structured-prompting-techniques-xml-json/ 22. Model Agnostic Prompts: Future-Proof AI Applications, https://blog.promptlayer.com/model-agnostic/ 23. Comparing 10 AI-Native Search APIs and Crawlers for LLM Agents - Medium, https://medium.com/towardsdev/comparing-10-ai-native-search-apis-and-crawlers-for-llm-agents-ed4130d22c67 24. AI Deep Research for Content Generation - ALwrity, https://www.alwrity.com/post/ai-deep-researchers-for-content-generation 25. (PDF) ResearchRubrics: A Benchmark of Prompts and Rubrics For Evaluating Deep Research Agents - ResearchGate, https://www.researchgate.net/publication/397521620_ResearchRubrics_A_Benchmark_of_Prompts_and_Rubrics_For_Evaluating_Deep_Research_Agents 26. DeepResearch Bench II: Diagnosing Deep Research Agents via Rubrics from Expert Report, https://arxiv.org/html/2601.08536v2 27. Step-DeepResearch Technical Report | alphaXiv, https://www.alphaxiv.org/overview/2512.20491 28. A Metaprompt to improve Deep Search on almost all platforms (Gemini, ChatGPT, Groke, Perplexity) - Reddit, https://www.reddit.com/r/PromptEngineering/comments/1kmy4tr/a_metaprompt_to_improve_deep_search_on_almost_all/ 29. Retrieval Augmented Generation (RAG) for LLMs - Prompt Engineering Guide, https://www.promptingguide.ai/research/rag 30. StepFun AI Introduce Step-DeepResearch: A Cost-Effective Deep Research Agent Model Built Around Atomic Capabilities - MarkTechPost, https://www.marktechpost.com/2026/01/25/stepfun-ai-introduce-step-deepresearch-a-cost-effective-deep-research-agent-model-built-around-atomic-capabilities/
