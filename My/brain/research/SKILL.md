---
name: research
description: "Recherche approfondie et itérative sur un sujet technique, stratégique ou de décision d'architecture, à invoquer explicitement via /research. Ne pas confondre avec la compétence 'Brainstorming', qui est un portique obligatoire de clarification/design à utiliser avant toute implémentation. /research sert à comparer des options (frameworks, approches, bonnes pratiques) ; 'Brainstorming' sert à clarifier une demande de fonctionnalité précise avant de coder."
---

## Qu'est-ce que /research ?

`/research` (anciennement `/brainstorm`) est une méthodologie de recherche qui reproduit la psychologie créative naturelle. Contrairement à une simple recherche, elle :

- Recherche, puis re-recherche, puis remet en question les résultats
- Interroge chaque hypothèse avec scepticisme
- Explore le sujet sous 5 perspectives d'experts différentes
- Produit des recommandations concrètes assorties de niveaux de confiance
- Aboutit à des conclusions éprouvées grâce à une analyse critique

**Distinction avec la compétence "Brainstorming"** : `/research` répond à des questions de type *"quelle est la meilleure option ?"* (recherche comparative, autonome, déclenchée à la demande). "Brainstorming" répond à des questions de type *"comment doit-on construire cette fonctionnalité ?"* (dialogue de clarification, obligatoire avant tout code). Si l'utilisateur demande une comparaison de technologies ou une validation stratégique → `/research`. S'il demande d'implémenter quelque chose → la compétence "Brainstorming" se déclenche d'abord automatiquement.

## Utilisation de base

```bash
/research Quel est le meilleur framework CLI pour créer des outils de développement ?
```

## Options

| Option | Nom | Description |
|---|---|---|
| `-e` | `--economy` | Mode économique : utilise des appels d'outils directs plutôt que des invocations multiples. Réduit le coût et l'usage de contexte. |
| `-f` | `--fast` | Mode rapide : saute la Phase 2 (remise en question) et condense la Phase 3 à 3 perspectives. Résultats plus rapides. |
| `--file` | Sauvegarder la session | Écrit la recherche dans `Docs/research/{slug-du-sujet}-{date}.md` |

## Exemples

**Recherche standard**
```bash
/research Quel est le meilleur framework CLI pour créer des outils de développement ?
```

**Mode économique (économise des tokens)**
```bash
/research -e Dois-je utiliser Next.js ou Remix pour mon projet ?
```

**Mode rapide (résultats rapides)**
```bash
/research -f Meilleures pratiques pour le rate limiting d'API
```

**Options combinées avec sortie fichier**
```bash
/research -e -f --file Microservices vs monolithe : les compromis
```

## Le workflow en 4 phases

| Phase | Rôle | Objectif | Actions clés |
|---|---|---|---|
| 1. Exploration expansive | EXPLORATEUR CURIEUX | Ratisser le plus large possible — aucun filtrage | Recherche dans plusieurs sources, rassemble des perspectives diverses, collecte des données brutes |
| 2. Remise en question critique | AVOCAT DU DIABLE | Mettre à l'épreuve chaque résultat | Interroge les hypothèses, cherche des contre-preuves, remet en question les opinions populaires. *Sautée en mode rapide* |
| 3. Synthèse multi-angles | SYNTHÉTISEUR | Voir sous 5 perspectives (3 en mode rapide) | Points de vue d'un expert technique, d'un stratège d'affaires, d'un utilisateur final, d'un sceptique et d'un pragmatique |
| 4. Cristallisation de l'action | CONSEILLER STRATÉGIQUE | Recommandations claires | Niveaux de confiance, compromis, avis contraire, prochaines étapes concrètes |

## Persona

L'agent de recherche opère comme un chercheur rigoureux avec les traits suivants :

- **Profondément sceptique** — tout remettre en question, ne rien prendre pour acquis
- **Intellectuellement honnête** — admettre l'incertitude, reconnaître les points faibles
- **Multi-perspective** — voir les problèmes sous tous les angles
- **Curieux sans relâche** — chaque réponse fait naître de nouvelles questions
- **Opinions fortes, tenues avec souplesse** — se forger des avis mais les faire évoluer selon les preuves

## Quand utiliser /research

Utilisez `/research` quand vous avez besoin de :

- **Décisions technologiques** — choisir entre frameworks, bibliothèques ou approches
- **Décisions d'architecture** — microservices vs monolithe, choix de base de données
- **Recherche de bonnes pratiques** — trouver des patterns éprouvés pour des problèmes précis
- **Planification stratégique** — évaluer des options sous plusieurs angles
- **Validation de décision** — tester vos hypothèses avant de vous engager

N'utilisez PAS `/research` pour clarifier une demande de fonctionnalité précise à implémenter — c'est le rôle de la compétence "Brainstorming", qui se déclenche automatiquement.

## Résultat produit

- Résultats clés de l'exploration expansive
- Hypothèses remises en question avec contre-preuves (sauf en mode rapide)
- Analyse multi-perspective sous 5 points de vue (3 en mode rapide)
- Recommandations avec niveaux de confiance et compromis
- **Avis contraire** — les arguments contre la recommandation
- **Insights actionnables** — prochaines étapes concrètes

Avec `--file`, tout le résultat est sauvegardé dans `Docs/research/{slug-du-sujet}-{date}.md` pour référence ultérieure.

---

## Installation comme commande personnalisée Gemini CLI

Gemini CLI supporte les commandes personnalisées via des fichiers `.toml` placés dans `.gemini/commands/`. Créez le fichier `.gemini/commands/research.toml` (fourni séparément), puis invoquez-le avec `/research <votre sujet>`.

Différences par rapport à la version Claude Code :
- **Subagents → invocations multiples séquentielles/parallèles** : chaque phase est exécutée comme un tour de raisonnement distinct dans le même modèle Gemini, ou via des appels séparés à l'API `generateContent` si orchestré manuellement.
- **Recherche web** : utilise l'outil de recherche Google intégré à Gemini (`googleSearch` / `google_search_retrieval`).
- **Sauvegarde fichier** : utilise l'outil de système de fichiers de Gemini CLI (`write_file`) pour écrire dans `Docs/research/`.