---
name: research
description: Recherche approfondie et itérative sur un sujet technique, stratégique ou de décision d'architecture. À déclencher pour évaluer et comparer des options (frameworks, bibliothèques, architectures, patterns) avant implémentation.
---

# 🧪 Recherche Technique & Décision d'Architecture (`research`)

## 🎯 Objectif
Cette compétence effectue une **recherche approfondie, méthodique et comparative** sur des sujets techniques ou stratégiques de code et d'architecture.

Contrairement au développement ou à la revue de code, `research` répond à la question : **"Quelle est la meilleure option technique ou architecturale pour notre besoin ?"**

---

## 🧠 Méthodologie en 4 Phases

L'analyse suit un processus rigoureux pour éviter le biais de confirmation :

| Phase | Rôle | Objectif | Actions Clés |
| :--- | :--- | :--- | :--- |
| **1. Exploration Expansive** | Explorateur Curieusement Neutre | Ratisser le plus large possible sans filtrage prématuré. | Collecte de données, benchmark de solutions, étude des patterns existants. |
| **2. Remise en Question Critique** | Avocat du Diable | Mettre à l'épreuve chaque option sélectionnée. | Traque des limitations cachées, contraintes runtime/licences, coûts et failles potentielles. |
| **3. Synthèse Multi-Perspectives** | Synthétiseur Technique | Analyser sous 5 angles. | Perspectives : Développeur, Architecte, SÉCURITÉ/OWASP, Performance, Pragmatique. |
| **4. Cristallisation & Action** | Conseiller Stratégique | Recommandation concrète et motivée. | Niveaux de confiance, compromis (trade-offs), avis contraire (*Devil's Advocate*) et plan d'action. |

---

## 🎯 Quand utiliser ce skill au sein d'Agent Code ?

- **Choix de Librairie / Dépendance** : Comparer deux bibliothèques pour C# / .NET (licence, maintenance, perf).
- **Décision d'Architecture Base de Données / Accès Données** : Évaluer la meilleure structure de données ou stratégie de mise en cache (EF Core vs Dapper, cache mémoire vs distribué).
- **Optimisation / Benchmarking** : Comparer différentes approches de performance (ex: streaming vs batching).
- **Validation de Pattern** : Vérifier la faisabilité technique d'un composant complexe avant le premier coup de code.

---

## 📋 Format de Restitution Attendu

1. **Synthèse & Recommandation Principale** (avec indice de confiance : 🟢 Élevé / 🟡 Moyen / 🔴 Faible).
2. **Tableau Comparatif des Options** (Avantages, Inconvénients, Impact Performance/Maintenabilité, Alignement Projet).
3. **Avis Contraire & Risques Identified** (*Devil's Advocate*).
4. **Prochaines Étapes Concrètes** (Orientation vers `brainstorming` pour le design UI ou `csharp-standards` pour l'implémentation).
