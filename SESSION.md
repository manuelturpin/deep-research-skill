# Deep Research Skill — Handoff

## Statut : IMPLEMENTE, TEST L1 PASS, L2+ A TESTER

## Fichiers

| Fichier | Emplacement | Statut |
|---------|-------------|--------|
| Skill | `~/.claude/commands/deep-research.md` | 254 lignes, actif |
| Plan | `~/.claude/projects/-Users-manuelturpin-openclaw-repo/memory/deep-research-skill-plan.md` | Complet |
| Rapports | Ce dossier (`deep-research-skill/`) | Vide — en attente de tests L2+ |

## Ce qui est fait

1. Plan detaille redige (5 phases, 4 niveaux, architecture outils)
2. Skill implemente : `~/.claude/commands/deep-research.md`
   - Frontmatter + persona
   - Phase 1 — Detect : scoring 4 axes, calibration auto L1-L4
   - Phase 2 — Decompose : L1 direct, L2 axes, L3 STORM, L4 full plan
   - Phase 3 — Execute : L1 WebSearch direct, L2 agents paralleles, L3 2 vagues, L4 3 vagues
   - Phase 4 — Verify : WebFetch obligatoire, cross-ref L2+, CoVe L3+
   - Phase 5 — Synthesize : L1 conversationnel, L2-L4 rapport markdown dans ce dossier
   - Guardrails + exemples calibration
3. Test L1 reussi : `/deep-research C'est quoi le protocole MCP ?`
   - Calibration correcte (score 5 → L1)
   - 2 WebSearch + 2 WebFetch executes
   - Sources verifiees avant citation
   - Reponse conversationnelle avec sources inline
4. Dossier rapports cree, skill pointe vers ce dossier
5. MEMORY.md mis a jour

## Prochaines etapes

### Test L2 (priorite)
```
/deep-research Compare Brave Search, Tavily et Exa pour un agent de recherche
```
Attendu :
- Calibration → L2 ou L3 (score ~10)
- 2-3 agents Task websearch en parallele
- Rapport markdown ecrit dans ce dossier
- Resume affiche en conversation

### Test L3 (si L2 passe)
```
/deep-research Comment le EU AI Act affecte-t-il le developpement des LLMs open-source ?
```
Attendu :
- 2 vagues d'agents (4-6 total)
- Brave MCP utilise (Temporalite >= 3)
- CoVe verification
- Rapport 3000-8000 mots

### Verifications transversales
- [ ] URLs WebFetchees (jamais de fabrication)
- [ ] Rapport ecrit dans ce dossier (pas dans le CWD du repo)
- [ ] Agents paralleles fonctionnent bien
- [ ] Brave MCP accessible via ToolSearch

## Architecture du skill (rappel)

```
Query → DETECT (scoring 4 axes → niveau L1-L4)
      → DECOMPOSE (sous-questions par niveau)
      → EXECUTE (WebSearch/WebFetch/Task agents/Brave MCP)
      → VERIFY (WebFetch obligatoire, cross-ref, CoVe)
      → SYNTHESIZE (conversation L1, rapport markdown L2+)
```

## Outils utilises

| Outil | Usage dans le skill |
|-------|---------------------|
| WebSearch | Recherche directe (tous niveaux) |
| WebFetch | Verification sources (tous niveaux) |
| Task (websearch) | Agents paralleles (L2+) |
| ToolSearch → brave_web_search | Couverture elargie (L3+) |
| ToolSearch → brave_news_search | Actualites recentes (Temporalite >= 3) |
| Write | Ecriture rapport markdown (L2+) |
