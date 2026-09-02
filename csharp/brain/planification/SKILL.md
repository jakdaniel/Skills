---
name: planification
description: Utiliser cette compétence lorsque vous avez une spécification ou des prérequis pour une tâche à plusieurs étapes, AVANT de toucher au code de développement.
---

# 📝 Planification : Rédiger des plans d'implémentation (Writing Plans)

## Aperçu

L'objectif de cette compétence est d'écrire des plans d'implémentation exhaustifs, en supposant que l'ingénieur/agent a **zéro contexte** sur la base de code globale du projet. 
Vous devez documenter chaque détail dont il a besoin pour accomplir le travail :
- Quels fichiers modifier pour chaque tâche
- Le code structuré à créer ou remplacer
- Les fichiers de tests à utiliser
- Comment tester la fonctionnalité
- Quand effectuer les commits

Fournissez l'intégralité du plan sous forme de **Bite-Sized Tasks** (tâches granulaires).

**Annonce de départ :** "J'utilise la compétence Planification pour créer le plan d'implémentation."

## 🧩 La Granularité "Bite-Sized Task"

**Chaque étape est une action unique (prenant 2 à 5 minutes) :**
- "Écrire le test unitaire défaillant" - *étape*
- "Exécuter le test unitaire pour s'assurer qu'il échoue" - *étape*
- "Implémenter le code minimal pour faire passer le test" - *étape*
- "Exécuter les tests et s'assurer qu'ils passent" - *étape*
- "Cree le Commit" - *étape*

## 📄 En-tête du Document de Planification

**CHAQUE plan DOIT commencer avec cet en-tête (dans `docs/plans/` ou directement dans `task.md` pour l'agent Antigravity) :**

```markdown
# Plan d'implémentation : [Nom de la Fonctionnalité]

> **Pour l'Agent :** SUIVEZ CE PLAN ÉTAPE PAR ÉTAPE.

**Objectif :** [Une phrase décrivant ce qui est construit]

**Architecture :** [2 à 3 phrases décrivant l'approche choisie]

**Stack Technique :** [Technologies clés et bibliothèques]

---
```

## 📋 Structure des Tâches (Modèle à Suivre)

````markdown
### Tâche N: [Nom du Composant]

**Fichiers :**
- Créer : `/chemin/absolu/vers/NouveauFichier.cs`
- Modifier : `/chemin/absolu/vers/Existant.cs:123-145`
- Test : `/chemin/absolu/vers/FichierTests.cs`

**Étape 1 : Écrire le test défaillant (Failing Test)**

```csharp
[Fact]
public void Fonction_ComportementSpecifique_RetourneAttendu()
{
    var result = MaClasse.Fonction(input);
    result.ShouldBe(expected);
}
```

**Étape 2 : Exécuter le test pour vérifier l'échec**

Exécutable : `dotnet test --filter "FullyQualifiedName~Fonction_ComportementSpecifique"`
Attendu : ÉCHEC avec l'erreur "méthode introuvable / non implémentée"

**Étape 3 : Écrire l'implémentation minimale**

```csharp
public static TypeRetour Fonction(Type input)
{
    return expected;
}
```

**Étape 4 : Exécuter le test pour valider le passage (Pass)**

Exécutable : `dotnet test --filter "FullyQualifiedName~Fonction_ComportementSpecifique"`
Attendu : SUCCÈS (PASS)

**Étape 5 : Faire un Commit**

```bash
git add /chemin/absolu/vers/fichier.test.ts /chemin/absolu/vers/nouveau-fichier.ts
git commit -m "feat: ajoute la fonctionnalité spécifique"
```
````

## 🛑 À Garder à l'esprit pour réussir la planification :
- **Chemins absolus** pour les fichiers : Toujours utiliser les chemins exacts ou complets.
- **Code complet dans le plan** : N'écrivez pas "ajouter une validation ici", fournissez le bout de code exact à rajouter.
- Commandes exactes fournies avec les sorties (outputs) attendus dans les terminaux.
- DRY, YAGNI, **TDD (Test Driven Development)** constant.
- **Commits fréquents :** Ne pas tout accumuler dans un commit à la fin.
