---
name: skill-creator
description: Compétence spécialisée pour concevoir et structurer de nouvelles compétences (Skills) Antigravity en français, en respectant les standards officiels et les bonnes pratiques. À utiliser à chaque fois qu'une nouvelle compétence doit être créée.
---

# 🛠 Créateur de Compétences (SkillCreator)

Cette compétence guide l'agent dans la création de nouvelles compétences robustes, modulaires et faciles à utiliser pour Antigravity.

## 📋 Processus de Création

### 1. Analyse et Cadrage
Avant de commencer, déterminez :
- **L'objectif unique** : Une compétence doit idéalement faire une seule chose très bien.
- **La nécessité** : La compétence apporte-t-elle une valeur ajoutée par rapport aux outils existants ?

### 2. Structure des Dossiers
Créez le dossier dans `.agent/Skills/{NomDeLaCompetence}/` avec la structure suivante :
- `SKILL.md` (Obligatoire) : Instructions et documentation.
- `scripts/` (Recommandé si besoin de logique) : Scripts Python ou Bash.
- `resources/` (Optionnel) : Templates, données, schémas.
- `examples/` (Optionnel) : Fichiers exemples d'utilisation.

### 3. Rédaction du fichier `SKILL.md`
Le fichier **doit** commencer par un bloc frontmatter YAML :
```yaml
---
name: nom-de-la-competence
description: Description précise (utilisée par l'agent pour la sélection).
---
```
Ensuite, structurez le contenu avec :
- **Aperçu** : Ce que fait la compétence.
- **Détails techniques** : Comment l'utiliser avec des exemples.
- **Arbres de décision** : "Si [condition], alors [action]".
- **Checklist de vérification** : Pour s'assurer que la tâche est bien faite.

## 💡 Bonnes Pratiques

- **Langue** : Tout doit être écrit en français.
- **Modularité** : Divisez les compétences complexes en sous-compétences si nécessaire.
- **Scripts auto-documentés** : Les scripts dans `scripts/` doivent supporter l'option `--help`.
- **Liens relatifs** : Utilisez des liens relatifs vers les ressources (`./resources/mon-fichier.md`).

## 🔍 Validation de la Compétence
Une fois la compétence créée, vérifiez :
1. Le frontmatter est valide (pas d'erreurs YAML).
2. La description est assez explicite pour être choisie par l'agent au bon moment.
3. Tous les fichiers référencés dans `SKILL.md` existent réellement.
