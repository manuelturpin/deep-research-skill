# Deep Research Skill

Skill de recherche approfondie multi-niveaux, LLM-agnostique, avec calibration automatique et anti-hallucination.

## Origine

Ce skill a ete construit a partir d'une analyse comparative de 3 recherches independantes sur les frameworks de recherche LLM :

| Source | Modele | Focus |
|--------|--------|-------|
| Perplexity | Sonar | Frameworks d'optimisation de prompts (PromptWizard, APO, PE2) |
| Claude | Opus 4.6 | Templates actionnable, core-plus-adapter, stats anti-hallucination |
| Gemini | Gemini 2.0 | Benchmarks (DRB-II), APIs de recherche, infrastructure |

## Pipeline de construction

Le skill a ete produit par un pipeline multi-agents (Claude Opus 4.6) :

```
Phase 1 : 3 agents lecteurs en parallele (extractions exhaustives)
Phase 2 : 1 agent synthetiseur (blueprint unifie de 1158 lignes)
Phase 3 : 1 agent ecrivain (skill initial de 884 lignes)
Phase 4 : 1 agent reviewer (audit qualite, score 7.5/10)
Phase 5 : corrections et ameliorations post-review
```

**Resultat** : 906 lignes, 39KB de skill production-ready.

## Structure du repo

```
skill/
  SKILL.md                    # Le skill final (production-ready)

research/
  sources/
    perplexity-research.md    # Recherche brute Perplexity
    claude-research.md        # Recherche brute Claude
    gemini-research.md        # Recherche brute Gemini
  extractions/
    extraction-perplexity.md  # Extraction structuree
    extraction-claude.md      # Extraction structuree
    extraction-gemini.md      # Extraction structuree
  unified-architecture.md     # Blueprint unifie (1158 lignes)
  review.md                   # Audit qualite du skill
```

## Fonctionnalites du skill

- **4 niveaux de profondeur** (Simple, Advanced, Ultra, Super Ultra) avec calibration automatique
- **Matrice de complexite 4 axes** : Largeur, Profondeur, Ambiguite, Temporalite
- **Templates XML complets** pour chaque niveau, prets a l'emploi
- **Protocole anti-hallucination gradue** : CoVe, steelmanning, audit de couverture
- **Agnosticite LLM** : pattern core-plus-adapter (Claude, GPT, Gemini, Llama, Mistral, Qwen)
- **Pipeline meta-prompting 7 etapes** avec auto-critique obligatoire

## Utilisation

Le skill est concu pour etre utilise par un agent LLM (Claude Code, OpenClaw, ou tout agent compatible SKILL.md). Il transforme toute question utilisateur en un processus de recherche structure et optimise.

## Licence

MIT
