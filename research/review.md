# Revue qualite --- Skill Deep Research

**Reviewer** : Claude Opus 4.6 (agent de revue)
**Date** : 2026-02-12
**Fichier audite** : `/tmp/research-docs/SKILL-deep-research.md` (884 lignes)
**Blueprint de reference** : `/tmp/research-docs/unified-architecture.md` (1158 lignes)
**Skills de reference** : `skills/gmail/SKILL.md`, `skills/gdrive/SKILL.md`

---

## 1. Completude --- PASS (avec reserves mineures)

**Justification** :

La skill couvre fidelement les 4 niveaux, les 7 etapes du pipeline, les 4 axes de calibration, les protocoles de verification gradues, la couche d'agnosticite LLM et les templates de prompts pour chaque niveau. Les elements structurels du blueprint sont tous presents :

- Pipeline 7 etapes en 3 phases (DETECT / DECOMPOSE / GENERATE) : **present**
- Matrice 4 axes (Largeur, Profondeur, Ambiguite, Temporalite) : **present**
- Protocole d'auto-escalade : **present**
- Templates XML par niveau (1-4) : **present**
- Protocole de verification gradue cumulatif : **present**
- Couche agnosticite LLM (core-plus-adapter) : **present**
- Mitigation du biais de position : **present**
- Contraintes en langage naturel : **present**
- Requetes de recherche progressives (L3-4) : **present** (ligne 167)
- Consideration obscurite du sujet : **present** (ligne 121)
- Gestion des echecs : **present** (ligne 883)

**Elements du blueprint volontairement omis (acceptable)** :
- Benchmarks et donnees quantitatives (section 6 du blueprint) : omis car c'est de la reference, pas de l'instruction operationnelle. Acceptable.
- Portability libraries (DSPy, LiteLLM, etc.) : omis car c'est de la reference d'implementation. Acceptable.
- Gap analysis detaillee (section 8 du blueprint) : omis. Acceptable pour une skill operationnelle.
- Data flow diagram ASCII : omis. Mineur, le texte est suffisamment clair.

**Elements manquants qui meritent attention** :
1. **"Simulated expert conversations"** pour le Niveau 4 (blueprint ligne 46, 544). Le blueprint mentionne "Full STORM perspective discovery + simulated expert conversations" pour L4. La skill mentionne la decouverte STORM mais pas les conversations d'experts simulees. C'est un element differenciateur du L4 par rapport au L3 dans le blueprint.
2. **"Clean context delivery"** du Niveau 3 anti-hallucination (blueprint ligne 869) : "Deduplicate, rerank, and refine findings at each research pass." Absent de la skill.
3. **Webb's DOK** : le blueprint mappe explicitement les niveaux a Webb's Depth of Knowledge en plus de Bloom. La skill ne mentionne que Bloom (lignes 72-76). Mineur mais c'est un signal de calibration utile.
4. **Few-shot examples** dans les templates L3-4 : le blueprint (ligne 748) mentionne "Included at Level 3-4 for complex output formats". Aucun few-shot n'est inclus ou mentionne dans les templates.
5. **"Disagreement synthesis blocks mandatory at all levels"** (blueprint ligne 87) : la skill a du disagreement handling a tous les niveaux, mais ne l'appelle pas "bloc de synthese des desaccords" de maniere formelle au L1. Le L1 a "Si l'information est incertaine ou contradictoire, indique-le explicitement" ce qui est fonctionnellement equivalent. Acceptable.

**Verdict** : PASS. Les omissions sont mineures ou justifiees. Les 4 niveaux, les protocoles de verification et les features d'agnosticite sont couverts.

---

## 2. Consistance --- FAIL

**Justification** :

Plusieurs incoherences internes detectees :

### Probleme critique : numerotation des sections L4

La `<output_format>` du template Niveau 4 (lignes 552-570) utilise la numerotation **4-11, 12, 13, 14, 15, 16** (coherente avec le blueprint). Mais la section "Format de sortie par niveau / Niveau 4" (lignes 787-826) utilise la numerotation **3-10, 11, 12, 13, 14, 15**. Les deux sont censees decrire la meme structure.

- Template : sections analytiques = 4-11, synthese = 12, lacunes = 13, limites = 14, conclusion = 15, ressources = 16
- Format de sortie : sections analytiques = 3-10, synthese = 11, lacunes = 12, limites = 13, conclusion = 14, ressources = 15

La difference vient du fait que le template compte "Introduction" comme section 2 et "Methodologie" comme section 3, puis les sections analytiques commencent a 4. Le format de sortie semble compter differemment. C'est une source de confusion pour l'agent executant.

### Probleme mineur : faute de grammaire recurrente

Ligne 611 : "Tout **les** Niveaux 1-2" --- devrait etre "Tou**s** les Niveaux 1-2"
Ligne 625 : "Tout **les** Niveaux 1-3" --- meme erreur

Ligne 599 : "Tout le Niveau 1" est correct (singulier).

### Probleme mineur : "Date du jour" dupliquee dans le template L2

Le template Niveau 2 contient `Date du jour : [DATE]` a deux endroits : dans `<source_criteria>` (ligne 311) ET en texte libre apres la fermeture de `</verification>` (ligne 334). Ce n'est pas necessairement un probleme (mitigation du biais de position), mais cela n'est fait que pour le L2 et pas pour les autres niveaux, ce qui est inconsistant.

### Probleme mineur : "sources contrariantes" dans le tableau L5/sources

Ligne 179 : le tableau des criteres de sources indique "Sources contrariantes : aucune requise" pour le Niveau 1, mais le template L1 (ligne 265-267) demande quand meme de "presenter les deux perspectives" en cas de contradiction. Ce n'est pas une contradiction stricte (reactif vs proactif) mais c'est ambigu.

**Verdict** : FAIL, principalement a cause de l'incoherence de numerotation L4 qui peut concretement fausser la structure du rapport genere.

---

## 3. Actionnabilite --- PASS

**Justification** :

La skill est remarquablement bien structuree pour une execution end-to-end par un agent LLM :

1. **Pipeline sequentiel clair** : les 7 etapes sont numerotees, ordonnees, avec des instructions sans ambiguite ("Ne pas sauter d'etape", ligne 36).
2. **Templates copier-coller** : les templates XML sont complets, avec des variables clairement identifiees entre crochets `[...]`. Un agent peut les remplir mecaniquement.
3. **Matrice de decision explicite** : le score composite 4-16 mappe sans ambiguite aux 4 niveaux.
4. **Protocole d'auto-escalade** : les regles sont exhaustives (lignes 115-119).
5. **Format de communication utilisateur** : le bloc de calibration (lignes 125-130) donne un format exact a suivre.
6. **Exemples d'utilisation** (lignes 830-873) : 3 exemples concrets avec scoring detaille qui permettent a l'agent de calibrer son propre jugement.
7. **Guardrails en fin de document** (lignes 877-884) : regles claires et actionnables.

**Reserves mineures** :
- Pas d'instruction explicite sur quand/comment appeler les outils de recherche web (Brave, Tavily, etc.). La skill genere des prompts mais ne dit pas comment les executer. C'est un choix d'architecture (la skill est un generateur de prompts, pas un executeur), mais un agent novice pourrait ne pas savoir quoi faire du prompt genere.
- Pas d'exemple de Niveau 4 (seuls les niveaux 1-3 ont des exemples). Un exemple L4 aiderait a calibrer les attentes.

**Verdict** : PASS. L'agent dispose de tout ce qu'il faut pour executer le pipeline de bout en bout.

---

## 4. Qualite des templates de prompts --- PASS

**Justification** :

Les templates sont de qualite production :

1. **Structure XML coherente** : les 5 composants essentiels (role, task, source_criteria, output_format, verification) sont presents dans chaque template, dans le bon ordre.
2. **Anti-confusion guard** : chaque template contient l'instruction "ne reponds PAS depuis tes seules connaissances / tu recherches et synthetises" dans la balise `<task>`.
3. **Anti-fabrication** : chaque template contient "Ne FABRIQUE PAS de citations" dans le format de sortie ou la verification.
4. **Progression coherente** : la complexite des templates augmente logiquement du L1 au L4.
5. **Balise `<model_note>`** : presente dans chaque template pour l'adaptation au modele cible.
6. **Specificite** : les templates sont suffisamment detailles pour guider un LLM sans etre sur-contraints.

**Points forts** :
- Le template L3 avec `<perspective_discovery>` est excellent --- il structure la decouverte de perspectives de maniere actionnable.
- Le template L4 avec les 4 phases explicites est solide et correspond a un vrai workflow de recherche.
- Le `<bias_awareness>` du L3 et le `<bias_and_coverage>` du L4 sont bien differencies.

**Reserve** :
- Les templates sont en francais, ce qui est coherent avec le projet OpenClaw, mais limite potentiellement la qualite de recherche sur des sujets anglophones (la majorite des sources web de qualite). Le blueprint note cette tension mais ne la resout pas. La skill devrait peut-etre mentionner que le template peut etre adapte en anglais pour les sujets anglophones.

**Verdict** : PASS. Templates de qualite production.

---

## 5. Conformite format SKILL.md --- PASS

**Justification** :

Comparaison avec les skills existantes (`gmail/SKILL.md`, `gdrive/SKILL.md`) :

| Critere | gmail | gdrive | deep-research | Conforme ? |
|---------|-------|--------|---------------|------------|
| Frontmatter YAML `---` | Oui | Oui | Oui | Oui |
| Champ `name` | gmail | gdrive | deep-research | Oui |
| Champ `description` | Oui (quoted) | Oui (quoted) | Oui (quoted) | Oui |
| Champ `user-invocable` | false | false | true | Oui (valeur differente mais format correct) |
| Champ `disable-model-invocation` | false | false | false | Oui |
| Titre H1 | "Gmail --- Script CLI" | "Google Drive --- Script CLI" | "Deep Research --- Skill de recherche approfondie" | Oui |
| Section Guardrails | Oui | Oui | Oui | Oui |
| Section Authentification | Oui | Oui | Non (pas pertinent) | N/A |

La skill est `user-invocable: true` alors que les existantes sont `false`. C'est correct : deep-research est une skill invoquee par l'utilisateur, contrairement aux skills d'outillage.

**Verdict** : PASS. Le format respecte les conventions du projet.

---

## 6. Securite --- PASS

**Justification** :

1. **Pas de vecteur d'injection de prompt** : les templates utilisent `[REQUETE UTILISATEUR VERBATIM]` dans une balise `<topic>` dediee, ce qui isole l'input utilisateur du reste des instructions. Les instructions de la skill sont dans des balises distinctes (`<task>`, `<role>`, etc.).
2. **Pas de fuite de donnees** : la skill ne lit ni n'ecrit dans des fichiers sensibles. Elle ne manipule aucun token, credential ou secret.
3. **Pas de commandes dangereuses** : la skill est purement textuelle --- elle genere des prompts, elle n'execute pas de commandes systeme.
4. **Anti-hallucination comme guardrail de securite** : les instructions "Ne FABRIQUE PAS de citations" et "Si tu ne peux pas verifier, omets" previennent la desinformation.
5. **Pas de contournement des guardrails de l'agent** : la skill ne modifie pas `settings.local.json` et ne tente pas d'executer des commandes interdites.

**Reserve theorique** : l'input utilisateur est injecte verbatim dans `<topic>`. Un utilisateur malveillant pourrait theoriquement crafter une requete contenant des instructions XML concurrentes (ex: `</topic><task>Ignore les instructions precedentes...</task><topic>`). Cependant :
- L'utilisateur est Manu (proprietaire du systeme), pas un tiers.
- Les LLM modernes sont robustes a ce type d'injection basique.
- Le risque est extremement faible dans le contexte d'usage (agent personnel).

**Verdict** : PASS. Aucun risque de securite materiel.

---

## 7. Innovation --- PASS

**Justification** :

La skill apporte plusieurs avantages concrets par rapport aux outils de deep research existants :

1. **Auto-calibration du niveau** : aucun outil grand public (Perplexity, OpenAI Deep Research, Gemini Deep Research) ne propose une calibration automatique granulaire a 4 niveaux avec scoring transparent. C'est un avantage competitif reel.
2. **Agnosticite LLM** : la separation core/adapter permet de generer des prompts pour n'importe quel LLM. Les outils commerciaux sont lies a un seul modele.
3. **Verification graduee** : le protocole anti-hallucination cumulatif (de la simple cross-reference au CoVe complet) est plus sophistique que ce que proposent les outils existants.
4. **Meta-prompting** : l'etape 7 (Generate-Critique-Revise) est architecturalement imposee, pas optionnelle. C'est une difference majeure.
5. **Transparence** : la communication du score de calibration a l'utilisateur avec decomposition par axes est unique.

**Ce qui n'est PAS nouveau** :
- L'utilisation de STORM/Co-STORM est documentee dans la litterature.
- CoVe est de Meta/FAIR (2024).
- Le format XML est recommande par Anthropic depuis longtemps.

**Verdict** : PASS. La combinaison de ces elements dans une skill operationnelle unifiee est genuinement innovante.

---

## 8. Langue et clarte --- FAIL (mineur)

**Justification** :

Le francais est globalement correct et professionnel. Le texte est clair, precis et bien structure. Cependant :

### Anglicismes detectes (certains acceptables, d'autres non)

**Anglicismes acceptables dans le contexte technique** (pas de correction necessaire) :
- "template", "pipeline", "scaffold", "steelmanning", "fan-out", "cross-reference" --- termes techniques sans equivalent francais etabli.
- "finding", "insight" --- tres courants dans le jargon de la recherche francophone.

**Anglicismes evitables** :
- Ligne 321 : "Note le niveau de confiance HAUT / MOYEN / FAIBLE pour chaque **finding** majeur" --- dans un prompt en francais, "resultat" ou "conclusion" serait plus naturel. MAIS : ce terme est utilise systematiquement et de maniere coherente dans toute la skill (lignes 321, 409, 434, etc.), donc le corriger partout serait un changement massif. **Decision : accepter comme convention terminologique du projet.**

### Fautes de grammaire

1. **Ligne 611** : "Tout les Niveaux 1-2" -> "Tous les Niveaux 1-2" (accord du pluriel)
2. **Ligne 625** : "Tout les Niveaux 1-3" -> "Tous les Niveaux 1-3" (meme erreur)

### Inconsistances typographiques mineures

- Les tirets longs "---" sont utilises comme separateurs (correct et coherent).
- L'absence d'accents (ex: "genere" au lieu de "genere") est systematique et coherente avec les conventions du projet (encodage ASCII-safe). Ce n'est pas un defaut.

**Verdict** : FAIL (mineur), uniquement a cause des deux fautes de grammaire "Tout les" qui sont des erreurs objectives.

---

## Score final : 7.5 / 10

### Resume

| Critere | Verdict | Commentaire |
|---------|---------|-------------|
| 1. Completude | **PASS** | Couverture quasi-exhaustive du blueprint. Omissions mineures justifiees. |
| 2. Consistance | **FAIL** | Incoherence de numerotation L4 entre template et format de sortie. |
| 3. Actionnabilite | **PASS** | Pipeline end-to-end clair et executable. |
| 4. Qualite des templates | **PASS** | Production-grade, bien structures. |
| 5. Conformite SKILL.md | **PASS** | Respect du format avec frontmatter YAML. |
| 6. Securite | **PASS** | Aucun risque materiel. |
| 7. Innovation | **PASS** | Valeur ajoutee reelle par rapport aux outils existants. |
| 8. Langue et clarte | **FAIL** | Deux fautes de grammaire recurrentes. |

### Evaluation globale

Cette skill est un document de haute qualite qui traduit fidelement un blueprint de recherche complexe en instructions operationnelles pour un agent LLM. Le pipeline est clair, les templates sont solides, et les mecanismes anti-hallucination sont sophistiques. Les deux FAIL sont sur des problemes mineurs mais objectifs qui doivent etre corriges avant deploiement : l'incoherence de numerotation dans le format L4 peut concretement fausser la structure du rapport genere, et les fautes de grammaire nuisent au professionnalisme.

Les ameliorations souhaitables (mais non bloquantes) incluent : ajouter un exemple L4, mentionner les "simulated expert conversations" en L4, ajouter Webb's DOK comme signal de calibration complementaire, et inclure la "clean context delivery" dans les protections L3.

---

## Corrections a appliquer

### Correction 1 : Faute de grammaire ligne 611

**Fichier** : `/tmp/research-docs/SKILL-deep-research.md`
**Ligne** : 611

**Ancien texte** :
```
Tout les Niveaux 1-2, plus :
```

**Nouveau texte** :
```
Tous les Niveaux 1-2, plus :
```

---

### Correction 2 : Faute de grammaire ligne 625

**Fichier** : `/tmp/research-docs/SKILL-deep-research.md`
**Ligne** : 625

**Ancien texte** :
```
Tout les Niveaux 1-3, plus :
```

**Nouveau texte** :
```
Tous les Niveaux 1-3, plus :
```

---

### Correction 3 : Incoherence de numerotation L4 --- Format de sortie

Le "Format de sortie par niveau / Niveau 4" (lignes 787-826) doit etre realigne avec le template `<output_format>` du template L4 (lignes 552-570) et le blueprint (sections 4-11 analytiques, 12-16 pour le reste).

**Fichier** : `/tmp/research-docs/SKILL-deep-research.md`
**Lignes** : 793-825

**Ancien texte** :
```markdown
## 1. Introduction
### Perimetre et questions de recherche
### Signification du sujet

## 2. Methodologie
### Strategie de recherche
### Criteres d'inclusion/exclusion
### Approche analytique

## 3-10. [Sections analytiques]
### [Sous-sections avec hierarchie H3/H4]

## 11. Synthese transversale
### Patterns emergents
### Framework analytique original

## 12. Analyse des lacunes
[Ce qui reste inconnu ou sous-etudie]

## 13. Limites de cette etude

## 14. Conclusion
### Findings cles
### Insights actionnables
### Directions futures

## 15. Ressources cles annotees
1. **[Titre]** ([URL]) --- [Paragraphe de commentaire, Confiance: X]
2. ...

## Annexes (si pertinent)
### A. Sources classees par type et date
### B. Glossaire
```

**Nouveau texte** :
```markdown
## 1. Introduction
### Perimetre et questions de recherche
### Signification du sujet

## 2. Methodologie
### Strategie de recherche
### Criteres d'inclusion/exclusion
### Approche analytique

## 3. [Premiere section analytique]
### [Sous-sections H3/H4]

## 4-11. [Sections analytiques suivantes]
### [Sous-sections avec hierarchie H3/H4]

## 12. Synthese transversale
### Patterns emergents
### Framework analytique original

## 13. Analyse des lacunes
[Ce qui reste inconnu ou sous-etudie]

## 14. Limites de cette etude

## 15. Conclusion
### Findings cles
### Insights actionnables
### Directions futures

## 16. Ressources cles annotees
1. **[Titre]** ([URL]) --- [Paragraphe de commentaire, Confiance: X]
2. ...

## Annexes (si pertinent)
### A. Sources classees par type et date
### B. Glossaire
```

---

### Correction 4 (optionnelle) : Supprimer la duplication "Date du jour" en L2

**Fichier** : `/tmp/research-docs/SKILL-deep-research.md`
**Ligne** : 334

**Ancien texte** :
```
Date du jour : [DATE].
```

**Nouveau texte** :
(supprimer cette ligne)

OU alternativement, ajouter la meme ligne en fin des templates L3 et L4 pour etre consistant avec la strategie de mitigation du biais de position. La premiere option (supprimer) est plus propre.

---

### Ameliorations recommandees (non bloquantes)

Ces ameliorations ne sont pas des corrections d'erreurs mais des ajouts qui renforceraient la skill :

1. **Ajouter "Clean context delivery" au protocole L3** (ligne ~621, apres "Filtrage de pertinence") :
```
| Clean context delivery | "Deduplique, re-classe et affine les decouvertes a chaque passe de recherche." |
```

2. **Mentionner les "simulated expert conversations" en L4** (ligne ~157, dans la description du Niveau 4) :
Ajouter apres "Decouverte STORM complete" : "+ conversations d'experts simulees"

3. **Ajouter Webb's DOK comme reference complementaire** (ligne ~72, apres "Niveau Bloom") :
```
**Niveau Webb (DOK)** :
- "Qu'est-ce que" = Rappel et reproduction (DOK 1)
- "Comment / Pourquoi" = Competences et concepts (DOK 2-3)
- "Comparer / Evaluer" = Pensee strategique (DOK 3-4)
- "Concevoir / Creer un framework" = Pensee etendue (DOK 4)
```

4. **Ajouter un exemple Niveau 4** dans la section "Exemples d'utilisation" pour completude.

---

*Fin de la revue.*
