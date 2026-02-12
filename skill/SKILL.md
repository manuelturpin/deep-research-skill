---
name: deep-research
description: "Recherche approfondie multi-niveaux avec calibration automatique, meta-prompting et anti-hallucination"
user-invocable: true
disable-model-invocation: false
---

# Deep Research --- Skill de recherche approfondie

## Vue d'ensemble

Cette skill transforme toute question utilisateur en une recherche structuree de haute qualite. Elle ne repond pas directement --- elle genere et execute un processus de recherche optimise via un pipeline de meta-prompting en 7 etapes.

**Principe fondamental** : separation tache/reponse. Le LLM est instruit de RECHERCHER et SYNTHETISER, jamais de repondre directement depuis ses connaissances. La qualite du prompt de recherche explique 80% de la variance de qualite du resultat (Anthropic).

**Quand utiliser cette skill** :
- Questions necessitant des sources fiables et verifiees
- Comparaisons multi-criteres entre technologies, approches, produits
- Analyses de domaines complexes ou en evolution rapide
- Revues de litterature, etats de l'art, cartographies de champ
- Toute question ou la precision factuelle prime sur la rapidite

**4 niveaux de profondeur** (auto-detectes ou choisis par l'utilisateur) :

| Niveau | Nom | Sources | Volume | Duree estimee |
|--------|-----|---------|--------|---------------|
| 1 | Simple | 1-5 | 100-500 mots | < 1 min |
| 2 | Advanced | 5-15 | 1 000-3 000 mots | 1-3 min |
| 3 | Ultra | 15-50 | 3 000-8 000 mots | 3-10 min |
| 4 | Super Ultra | 50-200+ | 8 000-20 000+ mots | 10-60 min |

---

## Pipeline de meta-prompting --- 7 etapes

Executer ces etapes dans l'ordre pour chaque requete de recherche. Ne pas sauter d'etape.

### PHASE A --- DETECT (Etapes 1-3)

#### Etape 1 : Ingestion brute

Capturer la requete utilisateur telle quelle. Identifier les parametres optionnels :
- Langue preferee (defaut : langue de la requete)
- Horizon temporel (si mentionne : "recent", "2024-2025", "historique")
- Niveau souhaite (si explicitement demande, sinon auto-calcul)
- Public cible (si mentionne : technique, general, decideur)

Ne pas transformer la requete. La stocker verbatim.

#### Etape 2 : Analyse du sujet

Classifier la requete selon ces dimensions :

**Classification d'intention** (une seule) :
- `informational` : "Qu'est-ce que X ?"
- `decisional` : "Quel X choisir pour Y ?"
- `comparative` : "X vs Y", "Comparer X et Y"
- `critical` : "Quels sont les risques de X ?"
- `systematic` : "Etat de l'art de X", "Revue systematique de X"
- `exploratory` : "Que sait-on de X ?", questions ouvertes

**Detection de contraintes implicites** :
- Marqueurs temporels : "recent", "evolution", "2024-2025", "tendances"
- Marqueurs de rigueur : "etudes academiques", "peer-reviewed", "donnees"
- Marqueurs de rapidite : "rapide", "en bref", "resume"
- Marqueurs de scope : "exhaustif", "complet", "systematique"

**Comptage d'entites** : nombre de concepts, technologies, ou objets distincts dans la requete.

**Operateurs de comparaison** : presence de "vs", "compare", "difference entre", "avantages/inconvenients".

**Niveau Bloom** :
- "Qu'est-ce que" = Remember (L1)
- "Comment / Pourquoi" = Understand/Apply (L2)
- "Comparer / Evaluer / Analyser" = Analyze/Evaluate (L3)
- "Concevoir / Synthetiser / Creer un framework" = Create (L4)

**Niveau Webb (DOK)** (signal complementaire) :
- "Qu'est-ce que" = Rappel et reproduction (DOK 1)
- "Comment / Pourquoi" = Competences et concepts (DOK 2-3)
- "Comparer / Evaluer" = Pensee strategique (DOK 3-4)
- "Concevoir / Creer un framework" = Pensee etendue (DOK 4)

#### Etape 3 : Determination du niveau

Appliquer la matrice de complexite a 4 axes. Pour chaque axe, attribuer un score de 1 a 4 :

**Axe 1 --- Largeur** (nombre de sous-domaines distincts) :
- 1 : domaine unique
- 2 : 2-5 domaines lies
- 3 : >5 domaines disjoints
- 4 : transdisciplinaire

**Axe 2 --- Profondeur** (niveaux de raisonnement requis) :
- 1 : inference en une etape
- 2 : 2-3 etapes dependantes
- 3 : 4+ etapes hierarchiques
- 4 : planification recursive

**Axe 3 --- Ambiguite** (degre de sous-specification) :
- 1 : entierement specifie
- 2 : moderement ouvert
- 3 : exploratoire/ambigu
- 4 : fortement sous-specifie

**Axe 4 --- Temporalite** (sensibilite temporelle de l'information) :
- 1 : stable/historique
- 2 : evolution lente
- 3 : changement actif
- 4 : temps reel/actualite brulante

**Score composite** = somme des 4 axes (4-16) :

| Score | Niveau | Nom |
|-------|--------|-----|
| 4-6 | 1 | Simple |
| 7-9 | 2 | Advanced |
| 10-12 | 3 | Ultra |
| 13-16 | 4 | Super Ultra |

**Protocole d'auto-escalade** :
- Si l'utilisateur a choisi un niveau ET la complexite detectee est 2+ niveaux au-dessus : escalader automatiquement avec avertissement explicite
- Si l'utilisateur a choisi un niveau ET la complexite detectee est 1 niveau au-dessus : respecter le choix utilisateur, ajouter une note consultative
- Ne JAMAIS auto-degrader un niveau choisi par l'utilisateur
- Si aucun niveau choisi : assigner automatiquement selon le score

**Consideration : obscurite du sujet**. Le taux de fabrication est inversement proportionnel a la popularite du sujet (6% pour sujets connus, 28-29% pour sujets de niche). Pour les sujets de niche, appliquer le protocole de verification du niveau superieur meme si le score composite indique un niveau inferieur.

Apres calcul, communiquer le resultat a l'utilisateur :

```
Calibration : Niveau [N] ([Nom]) detecte
  Largeur: [score]/4 | Profondeur: [score]/4 | Ambiguite: [score]/4 | Temporalite: [score]/4
  Score composite : [total]/16
  [Si escalade : "Escalade depuis Niveau X demande --- complexite detectee justifie Niveau Y"]
```

### PHASE B --- DECOMPOSE (Etapes 4-5)

#### Etape 4 : Decomposition hierarchique

La profondeur de decomposition depend du niveau :

**Niveau 1** : Pas de decomposition formelle. 1-3 sous-questions clarificatrices implicites maximum. Recherche en passe unique.

**Niveau 2** : 3-5 sous-questions organisees par axes thematiques :
1. Axe conceptuel/definitionnel
2. Axe comparatif/alternatives
3. Axe pratique/implementation
4. Axe limites/risques
5. Axe direction future (si pertinent)

Les sous-questions sont essentiellement paralleles (dependances minimales).

**Niveau 3** : Decouverte de perspectives style STORM :
- Identifier 4-6 perspectives d'experts distinctes pertinentes pour le sujet
- Pour chaque perspective : generer 2-3 questions de recherche specifiques
- Inclure obligatoirement une perspective contrariante qui conteste les hypotheses dominantes
- Decomposition hierarchique : macro-questions -> dimensions -> sous-questions specifiques
- Notation explicite des dependances : "explorer B apres avoir resolu A"
- Recherche multi-passes : balayage initial large, puis plongees ciblees

**Niveau 4** : Decouverte STORM complete + conversations d'experts simulees + plan de recherche multi-phases :
- Phase 1 (Reconnaissance) : cartographier le paysage complet --- sous-domaines, parties prenantes, perspectives, debats, percees, evolution historique
- Phase 2 (Investigation profonde) : recherche exhaustive par zone, documenter les affirmations avec notes de qualite, identifier contradictions et causes racines, chercher connexions inter-domaines non evidentes
- Phase 3 (Verification) : CoVe complet + auto-coherence + steelmanning + temporel + integrite citations + audit de couverture
- Phase 4 (Synthese) : patterns emergents, frameworks/taxonomies originaux, cartographie du consensus, questions frontieres, recommandations actionnables
- 8-12 zones d'investigation, chacune avec 3 questions prioritaires
- Graphe de dependances entre phases et zones d'investigation

**Gestion du fan-out** : equilibrer la largeur d'exploration et la focalisation. Ne pas generer plus de sous-questions que le LLM ne peut raisonnablement traiter. Privilegier la profondeur a la largeur quand les ressources sont limitees.

**Requetes de recherche progressives** (Niveaux 3-4) : ne PAS generer toutes les requetes de recherche a l'avance. Generer les requetes adaptativement en fonction des decouvertes initiales. Cela distingue une vraie recherche approfondie d'un simple RAG multi-requetes.

#### Etape 5 : Criteres de sources et protocole de verification

**Par niveau** :

| Critere | Niveau 1 | Niveau 2 | Niveau 3 | Niveau 4 |
|---------|----------|----------|----------|----------|
| Nombre de sources | 1-5 | 5-15 | 15-50 | 50-200+ |
| Types | generalistes, docs officielles | blogs experts, docs, articles academiques recents, rapports techniques | articles peer-reviewed, docs officielles, rapports techniques, analyses d'experts, donnees primaires | tous les precedents + articles fondamentaux, donnees gouvernementales, sources non anglophones |
| Fraicheur | date du jour, priorite au recent | priorite annee courante et precedente, inclure fondamentaux anciens | filtre temporel explicite, signaler tout >2 ans sur claims temporellement sensibles | priorite annee courante, inclure travaux fondamentaux anterieurs avec datation explicite |
| Diversite | verification croisee basique | viewpoints divergents, cross-ref 2+ sources independantes | min. 3 sources contrariant le narratif dominant, verifier couverture geographique/linguistique/culturelle | documentation systematique de la strategie de recherche, criteres inclusion/exclusion explicites |
| Sources contrariantes | aucune requise | min. 2 | min. 3 | min. 5 |

### PHASE C --- GENERATE (Etapes 6-7)

#### Etape 6 : Definition du format de sortie

Le format est defini par le niveau (voir section "Format de sortie par niveau" ci-dessous). A cette etape, confirmer :
- La structure exacte (sections, hierarchie de titres)
- Le volume cible en mots
- Le format de citation (inline avec URLs)
- Les tableaux attendus
- Le format technique : Markdown

#### Etape 7 : Assemblage meta-prompting (Generer-Critiquer-Reviser)

C'est l'etape critique. Elle est OBLIGATOIRE, pas optionnelle.

**7a** : Reformuler l'objectif de l'utilisateur en termes de recherche precis.

**7b** : Revoir la decomposition pour completude, redondance et correction des dependances.

**7c** : Verifier que les criteres de sources et de validation correspondent au niveau.

**7d** : Assembler le prompt complet a partir des 5 composants essentiels, dans cet ordre :
1. **Role et cadre contextuel** (`<role>`) : persona, contexte du domaine, audience, ton
2. **Definition de tache avec garde anti-confusion** (`<task>`) : objectif de recherche explicite, mention "rechercher et synthetiser, PAS repondre directement"
3. **Scaffold de decomposition** (`<research_plan>`, `<decomposition>`, ou `<phase_N>`) : sous-questions numerotees, groupees par axes, avec dependances
4. **Criteres de sources et protocole de verification** (`<source_criteria>`, `<verification>`) : types acceptables, fraicheur, cross-validation, CoVe
5. **Format de sortie et limites** (`<output_format>`) : structure exacte, volume, citations, bornes temporelles, instructions anti-hallucination

Utiliser les templates de la section correspondante au niveau detecte.

**7e** : Auto-critiquer le prompt assemble selon ces dimensions :
- Clarte : les instructions sont-elles sans ambiguite ?
- Completude : tous les aspects de la requete sont-ils couverts ?
- Specificite : les criteres sont-ils suffisamment precis pour etre verifiables ?
- Adequation au niveau : le prompt est-il calibre pour le bon niveau ?
- Couverture anti-hallucination : les gardes sont-ils en place ?

**7f** : Auto-reviser sur la base des problemes identifies en 7e.

**7g** : Appliquer la couche d'adaptation au modele cible (voir section "Agnosticite LLM").

---

## Templates de prompts par niveau

Les templates suivants utilisent XML pour la structure et Markdown pour le contenu. Remplacer les variables entre crochets `[...]` par les valeurs calculees aux etapes precedentes.

### Template Niveau 1 --- Simple

```xml
<model_note>
[Inserer les instructions specifiques au modele cible --- voir section Agnosticite LLM]
</model_note>

<role>
Tu es un assistant de recherche. Reponds a la question suivante de maniere
concise et precise en utilisant des sources web actuelles. Fournis 1 a 5
citations.
</role>

<topic>[REQUETE UTILISATEUR VERBATIM]</topic>

<task>
Fournis une reponse directe et factuelle a la question ci-dessus.
Ceci est une tache de recherche Niveau 1 (Simple) : priorise la precision
et la concision sur l'exhaustivite.
IMPORTANT : ne reponds PAS depuis tes seules connaissances. Recherche et
cite des sources verifiables.
</task>

<source_criteria>
- Utilise 1 a 5 sources credibles (documentation officielle, references faisant autorite)
- Priorise les sources recentes quand c'est pertinent
- Date du jour : [DATE]
</source_criteria>

<output_format>
- Reponse directe en 1 a 3 paragraphes (moins de 500 mots)
- Cite toutes les sources avec URLs inline
- Si l'information est incertaine ou contradictoire, indique-le explicitement
- Ne FABRIQUE PAS de citations. Si tu ne peux pas verifier qu'une source existe, omets-la.
</output_format>

<verification>
Confirme la reponse depuis au moins une source supplementaire. Si tu trouves
des informations contradictoires, presente les deux perspectives et signale
la divergence.
</verification>
```

### Template Niveau 2 --- Advanced

```xml
<model_note>
[Inserer les instructions specifiques au modele cible --- voir section Agnosticite LLM]
</model_note>

<role>
Tu es un analyste expert en [DOMAINE] qui conduit une revue analytique
structuree. Ton analyse doit etre equilibree, bien sourcee et clairement
organisee.
</role>

<topic>[REQUETE UTILISATEUR VERBATIM]</topic>

<task>
Conduis une analyse structuree de Niveau 2 (Advanced) sur le sujet ci-dessus.
Ton objectif est de comprendre les axes principaux, les variantes et les
debats actifs. Ne fournis PAS un resume superficiel --- investigue les
dimensions cles avec rigueur analytique.
IMPORTANT : tu recherches et synthetises, tu ne reponds PAS directement
depuis tes connaissances.
</task>

<research_plan>
Decompose ce sujet en 3 a 5 sous-questions cles couvrant des axes
analytiques distincts :
1. [Axe conceptuel/definitionnel --- genere automatiquement a l'etape 4]
2. [Axe comparatif/alternatives]
3. [Axe pratique/implementation]
4. [Axe limites/risques]
5. [Axe direction future, si pertinent]
</research_plan>

<source_criteria>
- Utilise 5 a 15 sources diversifiees (articles academiques, documentation
  officielle, analyses d'experts, rapports techniques)
- Priorise les sources de [ANNEE_COURANTE-1]-[ANNEE_COURANTE]
- Cross-reference les affirmations cles sur au moins 2 sources independantes
- Inclus au moins 2 sources qui presentent des vues alternatives ou contrariantes
- Date du jour : [DATE]
</source_criteria>

<output_format>
- Resume executif (100 mots)
- 3 a 5 sections analytiques avec titres H2 descriptifs
- Tableaux de comparaison si pertinent
- Section "Points cles a retenir"
- Volume total : 1 000-3 000 mots
- Citations inline avec URLs
- Note le niveau de confiance HAUT / MOYEN / FAIBLE pour chaque finding majeur
</output_format>

<verification>
Pour chaque affirmation majeure :
1. Verifie contre une seconde source independante
2. Signale toute information contestee ou incertaine
3. Identifie au moins 2 arguments qui contredisent ou nuancent la
   position dominante, et resume-les equitablement
Ne FABRIQUE PAS de citations. Si tu ne peux pas verifier une source,
omets-la et note la lacune.
</verification>

```

### Template Niveau 3 --- Ultra

```xml
<model_note>
[Inserer les instructions specifiques au modele cible --- voir section Agnosticite LLM]
</model_note>

<role>
Tu es un analyste de recherche senior qui conduit une investigation
comprehensive. Approche cette tache avec la rigueur d'une revue de
litterature academique. Ton analyse doit etre exhaustive, critiquement
evaluee et transparente sur ses limites.
</role>

<topic>[REQUETE UTILISATEUR VERBATIM]</topic>

<task>
Conduis un rapport analytique comprehensif de Niveau 3 (Ultra) sur le
sujet ci-dessus. Cela requiert une recherche multi-passes, une evaluation
critique des sources, l'identification des contradictions et une synthese
a travers de multiples perspectives d'experts. Ta sortie doit atteindre
le standard d'un rapport de recherche professionnel.
IMPORTANT : separe mentalement la phase de recherche de la phase de
redaction. Investigue d'abord, synthetise ensuite.
</task>

<perspective_discovery>
Avant de rechercher, identifie 4 a 6 perspectives d'experts distinctes
pertinentes pour ce sujet. Pour chaque perspective, genere 2 a 3 questions
de recherche specifiques. Inclus au moins une perspective "contrariante"
qui conteste les hypotheses dominantes.

Perspectives a considerer :
- [Expert du domaine principal]
- [Praticien/implementeur]
- [Critique/sceptique]
- [Expert d'un domaine adjacent]
- [Utilisateur final/partie affectee]
- [Perspective contrariante/dissidente]
</perspective_discovery>

<decomposition>
Organise ta recherche en ces axes majeurs :
[Outline hierarchique auto-genere a l'etape 4, avec structure H2/H3]

Pour chaque axe, investigue :
- Etat actuel et donnees cles
- Positions dominantes et leur base de preuves
- Contradictions ou debats actifs dans le domaine
- Developpements recents ([ANNEE_COURANTE-1]-[ANNEE_COURANTE])
- Connexions avec les autres axes de cette investigation
</decomposition>

<source_criteria>
- Minimum 15 sources, cible 30-50
- Priorise : articles peer-reviewed, documentation officielle, sources
  primaires, analyses d'experts, rapports techniques
- Inclus au moins 3 sources qui contestent le narratif dominant
- Toutes les sources doivent etre datees ; signale tout source de plus de
  2 ans pour les affirmations temporellement sensibles
- Inclus des sources non francophones/non anglophones si pertinent
- Date du jour : [DATE]
</source_criteria>

<verification_protocol>
Applique le protocole complet Chain-of-Verification (CoVe) :
1. Pour chaque affirmation factuelle majeure, genere une question de
   verification
2. Reponds a la question de verification de maniere independante (SANS
   referencer ton brouillon)
3. Si la verification contredit ton affirmation, revise ou signale comme
   incertain
4. Note la confiance pour chaque finding cle : HAUT / MOYEN / FAIBLE
5. Ne cite QUE des sources dont tu peux verifier l'existence. Retire
   toute source non confirmable et note la lacune.

Verification additionnelle :
- Steelmanning : pour les points contestes, construis la version la plus
  forte de chaque position avant d'evaluer
- Verification temporelle : signale les affirmations temporellement
  sensibles avec les dates de sources
- Distingue entre : faits etablis peu susceptibles de changer, donnees
  actuelles potentiellement mises a jour, situations en evolution
  necessitant les dernieres informations
</verification_protocol>

<output_format>
Structure comme un rapport de recherche :
1. Resume executif (200-300 mots)
2. Methodologie et perimetre
3-7. [Sections analytiques avec titres H2/H3 descriptifs]
8. Evaluation critique : lacunes, limites, contradictions
9. Conclusion : findings cles et insights originaux
- Volume total : 3 000-8 000 mots
- Markdown avec hierarchie H2/H3
- Citations inline avec URLs
- Tableaux pour les donnees comparatives
- Findings cles en gras pour la scannabilite
</output_format>

<bias_awareness>
Identifie la perspective que prend ton analyse. Presente les contre-arguments
les plus forts avec une rigueur analytique egale. Note les perspectives qui
pourraient etre sous-representees en raison de barrieres linguistiques,
d'acces, geographiques ou culturelles. Signale ou tes sources sont dominees
par un point de vue unique.
</bias_awareness>
```

### Template Niveau 4 --- Super Ultra

```xml
<model_note>
[Inserer les instructions specifiques au modele cible --- voir section Agnosticite LLM]
</model_note>

<role>
Tu es un chercheur principal dirigeant une investigation exhaustive.
Cette etude doit atteindre la rigueur d'une revue systematique de
litterature ou d'un rapport de recherche commissionne. Ton analyse doit
etre comprehensive, methodologiquement transparente, critiquement evaluee
et produire des insights analytiques originaux.
</role>

<topic>[REQUETE UTILISATEUR VERBATIM]</topic>

<task>
Conduis une etude de recherche exhaustive de Niveau 4 (Super Ultra) sur le
sujet ci-dessus. C'est le niveau de profondeur maximal : collecte
systematique de preuves, synthese multi-domaines, identification de toutes
les perspectives majeures, evaluation critique de la base de preuves
complete et construction de frameworks analytiques originaux.
Ta sortie doit atteindre le standard d'une revue systematique publiee ou
d'un rapport d'expert commissionne.
IMPORTANT : separe strictement les 4 phases ci-dessous. Ne commence pas
a rediger tant que les phases 1-3 ne sont pas completees.
</task>

<phase_1_reconnaissance>
Cartographie le paysage complet de ce sujet :
1. Identifie TOUS les sous-domaines, parties prenantes et perspectives majeurs
2. Decouvre les debats cles, questions ouvertes et percees recentes
3. Trace l'evolution historique et la trajectoire actuelle
4. Genere un plan de recherche comprehensif avec 8-12 zones d'investigation
5. Pour chaque zone, identifie :
   - Les 3 questions les plus importantes
   - Les types de sources attendus (academique, industrie, gouvernement, expert)
   - Les lacunes connues dans les preuves disponibles
6. Identifie les dependances entre zones d'investigation
</phase_1_reconnaissance>

<phase_2_investigation_profonde>
Pour chaque zone d'investigation :
1. Conduis une recherche exhaustive a travers bases academiques,
   documentation officielle, rapports techniques et commentaires d'experts
2. Pour chaque finding, documente :
   - L'affirmation
   - Les preuves a l'appui
   - La qualite de la source (peer-reviewed / primaire / secondaire / opinion)
   - La date de publication
   - Le niveau de confiance (HAUT / MOYEN / FAIBLE)
3. Identifie les contradictions entre sources et investigue les causes racines
4. Inclus des donnees quantitatives partout ou disponibles
5. Cherche des connexions non evidentes entre sous-domaines
6. Trace quelles sources consultees se sont averees inutiles (transparence
   methodologique)
</phase_2_investigation_profonde>

<phase_3_verification>
Applique le protocole complet de verification :
1. Chain-of-Verification : pour chaque affirmation majeure, genere une
   question de verification et reponds-y de maniere independante (SANS
   referencer ton brouillon). Si contradiction, revise ou signale.
2. Auto-coherence : quand possible, triangule depuis 3+ sources
   independantes. Approche les questions cles sous plusieurs angles.
3. Steelmanning : pour chaque point conteste, construis la version la plus
   forte d'au moins 3 positions concurrentes avant d'evaluer
4. Verification temporelle : signale toutes les affirmations temporellement
   sensibles avec dates de sources. Distingue entre : faits etablis,
   donnees actuelles potentiellement mises a jour, et situations en
   evolution.
5. Integrite des citations : verifie que chaque reference existe. Retire
   toute source non confirmable et note la lacune explicitement.
6. Audit de couverture : apres avoir termine la recherche, identifie quelles
   perspectives ou informations importantes tu as pu manquer.
7. Verification du biais de disponibilite : identifie quelles perspectives
   sont probablement sous-representees dans les sources accessibles.
8. Calibration de confiance : note les conclusions de chaque section
   HAUT / MOYEN / FAIBLE avec justification explicite du rating.
</phase_3_verification>

<phase_4_synthese>
1. Identifie les patterns emergents a travers les zones d'investigation
2. Construis un framework analytique ou une taxonomie originale si les
   preuves le supportent
3. Cartographie : consensus actuel, debats actifs et questions frontieres
4. Produis des recommandations actionnables fondees sur les preuves
5. Identifie les implications pour differents groupes de parties prenantes
</phase_4_synthese>

<source_criteria>
- Cible 50-200+ sources a travers : articles peer-reviewed, documentation
  officielle, rapports techniques, analyses d'experts, donnees primaires
- Priorise les publications [ANNEE_COURANTE-1]-[ANNEE_COURANTE] ; inclus les
  travaux fondamentaux anterieurs avec datation explicite
- Inclus des sources non francophones/non anglophones si pertinent
- Documente les criteres d'inclusion/exclusion explicitement
- Documente systematiquement ta strategie de recherche (mots-cles, bases de
  donnees, periodes)
- Ne FABRIQUE aucune citation. Si tu ne peux pas verifier qu'une source
  existe, omets-la et note explicitement quelle lacune informationnelle
  cela cree.
- Date du jour : [DATE]
</source_criteria>

<output_format>
Structure comme une etude de recherche comprehensive :
1. Resume executif (500 mots)
2. Introduction : perimetre, questions de recherche, signification
3. Methodologie : strategie de recherche, criteres d'inclusion/exclusion,
   approche analytique
4-11. [Sections analytiques, chacune avec sous-sections H2/H3/H4]
12. Synthese transversale : patterns emergents et frameworks originaux
13. Analyse des lacunes : ce qui reste inconnu ou sous-etudie
14. Limites de cette etude
15. Conclusion : findings cles, insights actionnables, directions futures
16. Ressources cles annotees (15-25 sources les plus importantes avec un
    paragraphe de commentaire chacune)
- Volume total : 8 000-20 000+ mots
- Markdown complet avec hierarchie H2/H3/H4
- Tableaux integres, matrices de comparaison
- Findings cles en gras tout au long
- Citations inline avec URLs
</output_format>

<bias_and_coverage>
- Presente au moins 3 perspectives concurrentes pour chaque point conteste
- Identifie quelles perspectives ton analyse sous-represente et pourquoi
- Verifie les lacunes de couverture geographiques, linguistiques et
  culturelles
- Signale ou tes sources sont dominees par un point de vue unique
- Signale les hypotheses integrees dans ton cadre analytique
</bias_and_coverage>
```

---

## Protocole de verification gradue

Les protections anti-hallucination sont cumulatives : chaque niveau herite de toutes les protections des niveaux inferieurs.

### Niveau 1 --- Protections de base

| Protection | Instruction dans le prompt |
|------------|---------------------------|
| Verification basique | "Confirme les faits cles depuis au moins une source supplementaire." |
| Garde anti-fabrication | "Ne FABRIQUE PAS de citations. Si tu ne peux pas verifier qu'une source existe, omets-la." |
| Signalement d'incertitude | "Si l'information est incertaine ou contradictoire, indique-le explicitement." |
| Ancrage temporel | "Date du jour : [DATE]. Priorise les sources recentes." |

### Niveau 2 --- Protections etendues

Tout le Niveau 1, plus :

| Protection | Instruction dans le prompt |
|------------|---------------------------|
| Cross-reference | "Cross-reference les affirmations cles sur au moins 2 sources independantes." |
| Niveaux de confiance | "Note la confiance HAUT / MOYEN / FAIBLE pour chaque finding majeur." |
| Sources contrariantes | "Identifie au moins 2-3 sources ou arguments qui contredisent ou nuancent la these dominante." |
| Synthese des desaccords | "Inclus une section identifiant ou les sources divergent et pourquoi." |
| Filtrage de fraicheur | "Priorise [ANNEE_COURANTE-1]-[ANNEE_COURANTE] pour l'etat de l'art." |

### Niveau 3 --- Protections rigoureuses

Tous les Niveaux 1-2, plus :

| Protection | Instruction dans le prompt |
|------------|---------------------------|
| Protocole CoVe complet | Brouillon -> questions de verification -> reponses independantes -> revision/signalement |
| Steelmanning | "Pour les points contestes, construis la version la plus forte de chaque position avant d'evaluer." |
| Verification temporelle | "Signale les affirmations temporellement sensibles avec dates de sources. Distingue faits etablis vs donnees en evolution." |
| Integrite des citations | "Ne cite QUE des sources verifiees. Retire les non confirmables et note les lacunes." |
| Conscience des biais | "Identifie la perspective de ton analyse. Presente les contre-arguments les plus forts. Note les perspectives sous-representees." |
| Transparence methodologique | "Documente comment la recherche a ete conduite : mots-cles, bases, periodes." |
| Filtrage de pertinence | "Avant d'incorporer une source, evalue son utilite reelle pour l'affirmation specifique." |
| Clean context delivery | "Deduplique, re-classe et affine les decouvertes a chaque passe de recherche. Ne conserve que les connaissances structurees." |

### Niveau 4 --- Protections maximales

Tous les Niveaux 1-3, plus :

| Protection | Instruction dans le prompt |
|------------|---------------------------|
| Auto-coherence | "Approche les questions cles sous plusieurs angles et verifie la convergence des conclusions." |
| Audit de couverture | "Apres avoir termine la recherche, identifie quelles perspectives ou informations importantes tu as pu manquer." |
| Biais de disponibilite | "Quelles perspectives sont probablement sous-representees dans tes donnees d'entrainement ou sources accessibles ?" |
| 3+ perspectives obligatoires | "Presente au moins 3 perspectives concurrentes pour chaque point conteste." |
| Gestion memoire globale | "Enregistre tes decouvertes iterativement. Assure-toi que les faits trouves tot ne sont pas contredits ou oublies lors de la synthese finale." |
| Calibration de confiance justifiee | "Note chaque section HAUT / MOYEN / FAIBLE avec justification explicite du rating." |
| Protocole d'incertitude | "Devant des informations contradictoires : presente toutes les perspectives avec ratings de qualite des preuves, identifie quelle information manquante augmenterait la confiance." |
| StepBack-Prompting | "Abstrait des instances specifiques pour identifier les principes fondamentaux avant de tirer des conclusions." |
| Stratification de qualite des sources | "Classe chaque source : peer-reviewed > primaire > secondaire > opinion. Pondere les affirmations en consequence." |
| Documentation methodologique complete | "Documente criteres inclusion/exclusion, strategie de recherche, mots-cles, bases, periodes, langues." |

---

## Couche d'agnosticite LLM

### Architecture core-plus-adapter

Le prompt de recherche est compose de deux couches :

| Couche | Contenu | Change quand... |
|--------|---------|-----------------|
| **Core (agnostique)** | Tache de recherche, scaffold de decomposition, criteres de sources, protocole de verification, format de sortie, instructions anti-hallucination | Le sujet ou le niveau change |
| **Adapter (specifique)** | Formatage structurel, fonctionnalites specifiques au modele, parametres comportementaux, mitigation du biais de position | Le modele cible change |

Le core contient 100% de la logique de recherche. L'adapter contient 0% de logique de recherche.

### Instructions d'adaptation par famille de modeles

Inserer ces instructions dans la balise `<model_note>` en tete de prompt :

**Claude (Anthropic)** :
```
Utilise la reflexion etendue (extended thinking) pour les etapes de verification.
Place les instructions detaillees dans le corps du prompt utilisateur.
Le contexte pertinent doit etre place en debut de prompt.
```

**GPT (OpenAI)** :
```
Avant de commencer la recherche, cree un plan explicite de ta strategie.
Utilise les en-tetes Markdown et le formatage pour structurer clairement.
Place les exemples pertinents en debut de prompt.
```

**Gemini (Google)** :
```
Exploite toute ta fenetre de contexte pour la retention des sources.
Specifie le perimetre temporel et geographique explicitement.
Suis le pattern plan-approuve-execute.
```

**Llama / Mistral / Qwen** :
```
Le formatage XML fonctionne bien meme sans entrainement specifique XML.
Suis les instructions structurees dans l'ordre indique.
```

### Mitigation du biais de position

Les LLM retrouvent l'information plus fiablement en debut et fin de prompt qu'au milieu ("lost-in-the-middle"). Pour tout prompt genere :
- Dupliquer les instructions critiques en debut ET en fin
- Placer les directives de recherche les plus importantes en tete
- Placer le materiel source / contexte au milieu (position la moins critique)
- Reformuler la consigne principale dans la derniere balise

### Contraintes en langage naturel

Exprimer toutes les exigences en langage naturel dans le prompt, jamais en parametres d'API :

| Au lieu de... | Exprimer comme... |
|---------------|-------------------|
| `temperature: 0.3` | "Sois precis et conservateur dans tes affirmations" |
| `max_results: 10` | "Ne depasse pas 10 sources" |
| `search_depth: advanced` | "Priorise la profondeur de recherche sur la rapidite" |
| `tools: ["web_search"]` | "Effectue autant de requetes de recherche que necessaire" |

---

## Format de sortie par niveau

### Niveau 1 --- Simple

```markdown
# [Titre de la question]

[Reponse directe en 1-3 paragraphes, 100-500 mots]

**Sources** :
1. [Titre](URL)
2. [Titre](URL)
```

### Niveau 2 --- Advanced

```markdown
# [Titre du sujet]

## Resume executif
[100 mots, 3-5 phrases]

## [Section analytique 1]
[Analyse avec citations inline]
> Confiance : HAUT/MOYEN/FAIBLE

## [Section analytique 2]
[...]

## [Section analytique 3]
[...]

| Critere | Option A | Option B | Option C |
|---------|----------|----------|----------|
| ... | ... | ... | ... |

## Points cles
- [Finding 1]
- [Finding 2]
- [Finding 3]

## Sources
1. [Titre](URL) --- [type, date]
2. ...
```

### Niveau 3 --- Ultra

```markdown
# [Titre du rapport]

## Resume executif
[200-300 mots]

## Methodologie et perimetre
[Comment la recherche a ete conduite]

## [Section analytique 1]
### [Sous-section 1a]
[Analyse detaillee avec citations]
### [Sous-section 1b]
[...]

## [Sections 2-5...]

## Evaluation critique
### Lacunes identifiees
### Limites de l'analyse
### Contradictions non resolues

## Conclusion
### Findings cles
### Insights originaux

## Bibliographie
1. [Auteur, Titre, Source, Date](URL) --- [Confiance: HAUT/MOYEN/FAIBLE]
```

### Niveau 4 --- Super Ultra

```markdown
# [Titre de l'etude]

## Resume executif
[500 mots]

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

## Exemples d'utilisation

### Exemple 1 : Requete simple (auto-detecte Niveau 1)

**Requete utilisateur** : "C'est quoi le protocole MCP d'Anthropic ?"

**Calibration** :
```
Calibration : Niveau 1 (Simple) detecte
  Largeur: 1/4 | Profondeur: 1/4 | Ambiguite: 1/4 | Temporalite: 2/4
  Score composite : 5/16
```

**Raison** : question factuelle a entite unique ("Qu'est-ce que"), Bloom = Remember, pas de comparaison, pas de scope etendu. Le score 5 place la requete en Niveau 1. Le prompt genere utilise le template Simple avec 1-5 sources.

---

### Exemple 2 : Requete comparative (auto-detecte Niveau 2)

**Requete utilisateur** : "Compare Brave Search API, Tavily et Exa pour un agent de recherche autonome. Avantages, inconvenients, pricing."

**Calibration** :
```
Calibration : Niveau 2 (Advanced) detecte
  Largeur: 2/4 | Profondeur: 2/4 | Ambiguite: 1/4 | Temporalite: 3/4
  Score composite : 8/16
```

**Raison** : 3 entites a comparer, operateur "compare", criteres multiples (avantages, inconvenients, pricing), domaine en evolution rapide (APIs 2025-2026). Bloom = Analyze. Le score 8 place la requete en Niveau 2. Le prompt genere utilise le template Advanced avec tableau comparatif, 5-15 sources, et cross-reference obligatoire.

---

### Exemple 3 : Requete complexe (auto-detecte Niveau 3, avec escalade potentielle)

**Requete utilisateur** : "Fais-moi un etat de l'art complet sur les architectures multi-agents pour la recherche approfondie : STORM, Co-STORM, les approches OpenAI, Gemini et Claude. Inclus les benchmarks recents et les limitations connues."

**Calibration** :
```
Calibration : Niveau 3 (Ultra) detecte
  Largeur: 3/4 | Profondeur: 3/4 | Ambiguite: 2/4 | Temporalite: 3/4
  Score composite : 11/16
```

**Raison** : 5+ entites (STORM, Co-STORM, OpenAI, Gemini, Claude), marqueur de scope "complet" + "etat de l'art", demande de benchmarks (donnees empiriques) + limitations (evaluation critique), domaine en evolution rapide. Bloom = Analyze/Evaluate. Le score 11 place la requete en Niveau 3. Le prompt genere utilise le template Ultra avec decouverte de perspectives STORM, CoVe complet, 15-50 sources et rapport structure.

### Exemple 4 : Requete exhaustive (auto-detecte Niveau 4)

**Requete utilisateur** : "Produis une revue systematique exhaustive de l'optimisation automatique de prompts : APE, OPRO, PromptBreeder, DSPy, PromptWizard et toutes les approches significatives. Inclus l'evolution historique, les fondements theoriques, les resultats empiriques compares, les applications en production, les limitations fondamentales et les directions de recherche ouvertes. Couvre les publications academiques et les implementations open-source."

**Calibration** :
```
Calibration : Niveau 4 (Super Ultra) detecte
  Largeur: 4/4 | Profondeur: 4/4 | Ambiguite: 3/4 | Temporalite: 3/4
  Score composite : 14/16
```

**Raison** : 6+ entites nommees + "toutes les approches significatives" (scope ouvert), demande transdisciplinaire (theorie + empirique + production + open-source), marqueurs de scope maximal ("revue systematique exhaustive", "evolution historique", "fondements theoriques"), demande de synthese multi-dimensionnelle. Bloom = Create (produire un framework de comprehension). Webb DOK = 4 (Pensee etendue). Le score 14 place la requete en Niveau 4. Le prompt genere utilise le template Super Ultra avec plan 4 phases, 8-12 zones d'investigation, 50-200+ sources, CoVe complet + steelmanning + audit de couverture.

---

## Guardrails

- **Ne JAMAIS fabriquer de citations** : c'est la regle zero, appliquee a tous les niveaux. Mieux vaut une lacune documentee qu'une source inventee.
- **Ne JAMAIS presenter des connaissances du modele comme des resultats de recherche** : distinguer explicitement ce qui vient de sources verifiees et ce qui est inference du modele.
- **Transparence sur les limites** : toujours inclure une section sur ce que la recherche n'a PAS pu couvrir et pourquoi.
- **Separation recherche/redaction** : a Niveau 3+, ne pas commencer a rediger avant d'avoir termine l'investigation. Cela previent le biais de confirmation ou l'auto-satisfaction prematuree.
- **Gestion des echecs** : si le LLM ne peut pas investiguer un axe (filtres de securite, manque d'acces), il doit le declarer explicitement plutot que de produire un resultat incomplet silencieusement.
- **Sujet de niche** : si le sujet est peu couvert dans les sources accessibles, augmenter automatiquement la rigueur de verification d'un niveau.
