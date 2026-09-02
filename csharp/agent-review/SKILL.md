---
name: agent-review
description: Orchestrateur des compétences de revue de code (revue structurée & actionnable, posture sceptique orientée risques, revue sénior exigeante, revue brutale sans concession, traque de bugs & hypothèses implicites). À utiliser pour toute analyse de code, revue de qualité, validation de Pull Request, revue avant merge, diagnostic de bug ou vérification de code généré.
---

# 🔎 Orchestrateur Agent Review (`agent-review`)

Cette compétence orchestre l'ensemble des compétences de revue de code situées dans le dossier `agent-review/`. Elle permet de choisir la **posture de review adaptée** au contexte : revue pédagogique et actionnable, analyse sceptique orientée risques, audit sénior exigeant, retour brutal sans filtre, ou chasse aux bugs.

**Règle d'activation** : dès qu'une revue ou une analyse de code est demandée, identifier l'intention (apprendre / valider / stress-tester / diagnostiquer) puis charger le sous-skill correspondant ci-dessous avant toute analyse.

---

## 🗂️ Matrice des Sous-Compétences Orchestrées

L'orchestrateur délègue aux compétences suivantes (3 situées dans le dossier [`../code/`](../code/) + `bug-finder` partagé avec `agent-code`) :

| Compétence | Fichier | Rôle & Posture de Review |
| :--- | :--- | :--- |
| **`code-review-sceptique`** | [`../code/code-review-sceptique/SKILL.md`](../code/code-review-sceptique/SKILL.md) | **Revue Sceptique Orientée Risques ("What could go wrong?")**<br>• Principe : "All tests pass" ≠ code sûr, robuste ou maintenable.<br>• Classification stricte double axe : Sévérité (Critical → Info) × Validité (Confirmed → Speculative).<br>• Table de constats obligatoire affichée en premier, évaluation de la couverture réelle des tests. |
| **`code-review-extreme`** | [`../code/code-review-extreme/SKILL.md`](../code/code-review-extreme/SKILL.md) | **Revue Brutale Sans Concession (Stress-Test)**<br>• Posture d'un développeur senior dur, critiqueur et sans filtre.<br>• Débusque sans pitié le code "à peu près", les anti-patterns et la dette.<br>• À réserver au stress-test volontaire — jamais pour un feedback utilisateur standard. |
| **`review-senior`** | [`../code/review-senior/SKILL.md`](../code/review-senior/SKILL.md) | **Revue Sénior Exigeante & Constructive (Validation Prod)**<br>• Audit rigoureux, incisif et sans concession, mais constructif (sans la toxicité extrême).<br>• Tolérance zéro sur les points bloquants : contrôle d'accès/ownership, `dynamic`, fuites de ressources, limites de lignes.<br>• Posture de référence pour valider un code prêt pour la production avant merge. |
| **`bug-finder`** | [`../code/bug-finder/SKILL.md`](../code/bug-finder/SKILL.md) | **Senior Bug Hunter — Traque de Bugs & Hypothèses Implicites**<br>• Analyse le code "comme s'il était déjà en production sous forte charge" : bugs existants, latents et futurs probables.<br>• Principe : un bug est souvent une hypothèse implicite qui s'avère fausse à l'exécution.<br>• Axes C#/.NET : async/await, LINQ différé, EF Core, ownership, edge cases.<br>• Classification `ANO-XX` (Critical → Potential) + scénarios de déclenchement. |

> ⚠️ **Note de versionnage** : une variante plus riche existe ailleurs —
> le `code-review-extreme` global (version étendue) ; privilégier la version enrichie si disponible.

> 🧹 **Historique de déduplication** :
> - `mycode-review` supprimé (copie renommée de l'ancien `agent-code/code-review`, lui-même retiré lors du déplacement des revues).
> - `review-senior` a été **déplacé dans `../code/`** (les copies d'`agent-code` ont été supprimées) : [`../code/review-senior/`](../code/review-senior/SKILL.md) en est désormais la version canonique, référencée par ce dossier.
> - `bug-finder` a été **fusionné avec la version d'`agent-code`** dans [`code/bug-finder/`](../code/bug-finder/SKILL.md) (copie locale supprimée) : unique version canonique, partagée entre les deux orchestrateurs.

---

## 🗺️ Workflow d'Orchestration

Lorsqu'une revue de code est confiée à l'agent, l'orchestrateur suit le flux ci-dessous :

```mermaid
flowchart TD
    A[Demande Utilisateur / Revue de Code] --> B{Identifier l'intention}

    B -->|Améliorer / Apprendre / Refactoring| C[Activer review-senior (posture constructive)]
    B -->|Valider avant merge / PR / Risques| D[Activer code-review-sceptique]
    B -->|Diagnostiquer un bug / Comportement inattendu| E[Activer bug-finder]
    B -->|Stress-test brutal volontaire| F[Activer code-review-extreme]
    B -->|Validation prod / Code prêt à merger| S[Activer review-senior]
    B -->|Audit complet multi-postures| G[Combinaison Séquentielle]

    C --> H1[5 axes d'analyse + Problème/Pourquoi/Solution corrigée]
    D --> H2[Table de constats Sévérité x Validité + Couverture tests réelle]
    E --> H3[Hypothèses implicites + Edge cases + Bugs latents]
    F --> H4[Anti-patterns + Dette + Zéro complaisance]
    S --> H6[Tolérance zéro bloquants + Audit exigeant constructif]
    G --> H5[bug-finder -> code-review-sceptique -> review-senior]

    H1 --> I[Restitution : constats classifiés + recommandations]
    H2 --> I
    H3 --> I
    H4 --> I
    H5 --> I
    H6 --> I
```

---

## 🎯 Combinaisons & Scénarios d'Orchestration

### Scénario 1 : Revue de Code Standard & Actionnable
* **Compétence Principale** : [`review-senior`](../code/review-senior/SKILL.md) (posture constructive)
* **Consignes** :
  1. Couvrir les axes : découpage, SOLID, Clean Architecture, bonnes pratiques C# / .NET.
  2. Pour chaque problème : explication du risque + exemple de code corrigé.
  3. Rester constructif : le but est l'amélioration, pas la culpabilisation.

### Scénario 2 : Validation Avant Merge / Analyse de Pull Request
* **Compétence Principale** : [`code-review-sceptique`](../code/code-review-sceptique/SKILL.md)
* **Compétence Complémentaire** : [`bug-finder`](../code/bug-finder/SKILL.md)
* **Consignes** :
  1. Adopter la posture "What could go wrong?" — ne jamais défendre le code.
  2. Produire la table de constats (Sévérité × Validité) en premier.
  3. Évaluer ce que les tests ne couvrent PAS (faux sentiment de sécurité).

### Scénario 3 : Diagnostic de Bug / Comportement Inattendu
* **Compétence Principale** : [`bug-finder`](../code/bug-finder/SKILL.md)
* **Consignes** :
  1. Analyser le code comme s'il était déjà en production.
  2. Lister les hypothèses implicites non prouvées (entrées, timing, état, concurrence).
  3. Documenter les scénarios d'échec et les cas limites non gérés.

### Scénario 4 : Revue Sénior Exigeante Avant Mise en Production
* **Compétence Principale** : [`review-senior`](../code/review-senior/SKILL.md)
* **Compétence Complémentaire** : [`code-review-sceptique`](../code/code-review-sceptique/SKILL.md)
* **Consignes** :
  1. Audit sans complaisance mais constructif : le code doit être prêt pour la production.
  2. Tolérance zéro sur les bloquants : contrôle d'accès/ownership serveur, typage non strict (`dynamic`, casts sauvages), fuites de ressources (`IDisposable`, `HttpClient`), limites de lignes dépassées.
  3. Chaque refus est motivé et accompagné d'une piste de correction.

### Scénario 5 : Stress-Test / Retour Brutal Volontaire
* **Compétence Principale** : [`code-review-extreme`](../code/code-review-extreme/SKILL.md)
* **Consignes** :
  1. Zéro complaisance : tout anti-pattern, toute dette et tout "à peu près" sont dénoncés.
  2. Utiliser uniquement à la demande explicite (posture agressive assumée).

### Scénario 6 : Audit Complet Multi-Postures
* **Pipeline** : [`bug-finder`](../code/bug-finder/SKILL.md) → [`code-review-sceptique`](../code/code-review-sceptique/SKILL.md) → [`review-senior`](../code/review-senior/SKILL.md)
* **Consignes** :
  1. Phase 1 (bug-finder) : identifier bugs latents et hypothèses fragiles.
  2. Phase 2 (sceptique) : classifier tous les constats par risque et validité.
  3. Phase 3 (review-senior) : fournir les solutions concrètes et le code corrigé.

---

## 📜 Invariants Transverses à Toutes les Revues

Quel que soit le sous-skill activé, les constats s'exécutent sous les standards du projet :

1. **"Tests verts" ≠ Code fiable** :
   - Toujours évaluer la couverture réelle et signaler ce qui n'est pas testé.
2. **Constats Toujours Structurés** :
   - Chaque problème est classifié (sévérité) et justifié (pourquoi c'est risqué) — jamais de vague "ça pourrait être mieux".
3. **Standards C# / .NET Non Négociables** :
   - Contrôle d'accès/ownership serveur sur toutes les données utilisateur, nullable reference types actifs (zéro `dynamic`, zéro null-forgiving injustifié), fichiers < 400 lignes, zéro placeholder, async/await de bout en bout.
4. **Le Silence N'est Jamais Acceptable** :
   - Si aucun problème n'est trouvé, expliquer explicitement ce qui a été vérifié et pourquoi le code est sain.

---

## 🌳 Arbre de Décision d'Activation

```
SI la demande concerne "review le code / améliore ce code / refactoring conseillé" :
   ➜ Charger ../code/review-senior/SKILL.md

SI la demande concerne "valide ce code avant merge / review de PR / est-ce risqué ?" :
   ➜ Charger ../code/code-review-sceptique/SKILL.md (+ ../code/bug-finder/SKILL.md si logique complexe)

SI la demande concerne "code prêt pour la prod / revue exigeante avant merge" :
   ➜ Charger ../code/review-senior/SKILL.md

SI la demande concerne "ça bug / comportement inattendu / trouve les cas limites" :
   ➜ Charger ../code/bug-finder/SKILL.md

SI la demande concerne "sois brutal / stress-test / review sans pitié" :
   ➜ Charger ../code/code-review-extreme/SKILL.md

SI la demande concerne "audit complet du code" :
   ➜ Pipeline : ../code/bug-finder -> ../code/code-review-sceptique -> ../code/review-senior
```

---

## ✅ Checklist de Vérification d'Orchestration

Avant de finaliser une revue prise en charge par `agent-review` :
- [ ] La posture de review est-elle adaptée à l'intention (pédagogique / sceptique / exigeante / brutale / diagnostic) ?
- [ ] Chaque constat est-il classifié (sévérité, validité) et justifié ?
- [ ] Les hypothèses implicites ont-elles été explicitement listées ?
- [ ] Ce que les tests ne couvrent pas a-t-il été signalé ?
- [ ] Les standards C# / .NET (ownership, nullabilité, 400L, async/await) ont-ils été vérifiés ?
- [ ] Les recommandations sont-elles actionnables (code corrigé ou étape concrète) ?
