---
name: design-tokens
description: Normalise, extrait et centralise toutes les valeurs de styles brutes (couleurs, polices, ombres, animations cubic-bezier, gradients, bordures, fallbacks inline) en tokens et variables CSS sémantiques réutilisables dans src/app.css.
---

# 🎨 Centralisation & Normalisation des Tokens Design (CSS Tokens)

Cette compétence régit la détection, l'extraction et la réorganisation de toutes les valeurs de styles brutes ou non standardisées (couleurs hex/rgba, ombres, transitions, typographies, animations `cubic-bezier`, fallbacks inline) présentes dans l'application vers le fichier central `src/app.css`.

---

## 🎯 Objectifs

1. **Zéro Valeur Brute Inline :** Éliminer les couleurs en dur (`#f43f5e`, `white`, `rgba(...)`), les ombres `box-shadow` volantes, et les fonctions de bézier `cubic-bezier(...)` éparpillées dans les composants Svelte et styles SCSS.
2. **Nettoyage des Fallbacks Inline :** Remplacer les expressions comme `var(--color-status-error, #f43f5e)` par des références directes à des variables globales unifiées comme `var(--color-status-error)`.
3. **Nomenclature Sémantique et Structurée :** Déclarer des variables CSS explicites, réutilisables et cohérentes dans `:root` (dans `src/app.css`).

---

## 📐 Catégories & Conventions de Nommage dans `src/app.css`

Toutes les nouvelles variables doivent être ajoutées au fichier `src/app.css` sous la section appropriée dans `:root`.

### 1. Animations & Courbes de Bézier (`--ease-*`, `--transition-*`)
*   **Format :** `--ease-[style]` ou `--transition-[nom]`
*   **Exemples :**
    *   `cubic-bezier(0.34, 1.56, 0.64, 1)` ➔ `--ease-spring-bounce: cubic-bezier(0.34, 1.56, 0.64, 1);`
    *   `all 0.3s cubic-bezier(0.4, 0, 0.2, 1)` ➔ `--transition-smooth: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);`

### 2. Ombres et Effets (`--shadow-*`)
*   **Format :** `--shadow-[type/usage]`
*   **Exemples :**
    *   `box-shadow: 0 4px 15px rgba(var(--color-primary-rgb), 0.2);` ➔ `--shadow-primary-medium: 0 4px 15px rgba(var(--color-primary-rgb), 0.2);`
    *   `box-shadow: 0 2px 6px rgba(0, 0, 0, 0.05);` ➔ `--shadow-subtle: 0 2px 6px rgba(0, 0, 0, 0.05);`
    *   `box-shadow: 0 15px 40px rgba(0, 0, 0, 0.12);` ➔ `--shadow-overlay-dense: 0 15px 40px rgba(0, 0, 0, 0.12);`

### 3. Transparences & Effets Glassmorphism (`--glass-*`, `--color-*-alpha-*`)
*   **Format :** `--glass-[variante]` ou `--color-[teinte]-alpha-[pourcentage]`
*   **Exemples :**
    *   `rgba(255, 255, 255, 0.95)` ➔ `--glass-bg-dense: rgba(255, 255, 255, 0.95);`
    *   `rgba(0, 0, 0, 0.12)` ➔ `--color-dark-alpha-12: rgba(0, 0, 0, 0.12);`

### 4. Couleurs Texte, Surface & Statuts (`--color-*`)
*   **Format :** `--color-[domaine]-[rôle]`
*   **Exemples :**
    *   `color: white;` ➔ `color: var(--color-text-inverse);`
    *   `var(--color-status-error, #f43f5e)` ➔ `var(--color-status-error)`
    *   `background: var(--color-status-error, #f43f5e)` ➔ `background: var(--color-status-error)`
    *   `background: rgba(var(--color-status-error-rgb), 0.02)` ➔ `background: var(--color-status-error-bg);`

---

## 🔄 Exemples de Refactorisation (Avant / Après)

| Style Brut (Avant) | Variable / Token CSS (Après) | Emplacement dans `src/app.css` |
| :--- | :--- | :--- |
| `transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);` | `transition: all 0.3s var(--ease-spring-bounce);` | `--ease-spring-bounce: cubic-bezier(0.34, 1.56, 0.64, 1);` |
| `box-shadow: 0 4px 15px rgba(var(--color-primary-rgb), 0.2);` | `box-shadow: var(--shadow-primary-medium);` | `--shadow-primary-medium: 0 4px 15px rgba(var(--color-primary-rgb), 0.2);` |
| `background: rgba(255, 255, 255, 0.95);` | `background: var(--glass-bg-dense);` | `--glass-bg-dense: rgba(255, 255, 255, 0.95);` |
| `color: var(--color-status-error, #f43f5e);` | `color: var(--color-status-error);` | `--color-status-error: #bc4749;` *(déjà défini dans ThemeProvider / app.css)* |
| `color: white;` | `color: var(--color-text-inverse);` | `--color-text-inverse: #ffffff;` |
| `box-shadow: 0 2px 6px rgba(0, 0, 0, 0.05);` | `box-shadow: var(--shadow-subtle);` | `--shadow-subtle: 0 2px 5px rgba(0, 0, 0, 0.05);` |

---

## 🛠 Procédure de Traitement (Workflow Step-by-Step)

### Étape 1 : Recherche & Audit
Lancer un audit avec `grep_search` pour repérer le style cible :
- Détecter les fonctions de bézier : `cubic-bezier(`
- Détecter les ombres brutes : `box-shadow: 0` ou `rgba(`
- Détecter les fallbacks inline : `, #` dans `var(`

### Étape 2 : Vérification dans `src/app.css`
Vérifier si un token existant répond déjà au besoin.
- Si le token existe déjà : Réutiliser le token existant.
- Si le token n'existe pas : Créer le token dans la section appropriée de `:root` dans `src/app.css`.

### Étape 3 : Remplacement dans les fichiers
Utiliser `replace_file_content` ou `multi_replace_file_content` pour mettre à jour les fichiers de composants Svelte ou styles.

### Étape 4 : Validation
Exécuter la vérification du projet :
```bash
npm run check
```

---

## 📋 Checklist de Validation

- [ ] Aucun composant ne contient de valeur hexadécimale ou rgba brute non justifiée.
- [ ] Aucun fallback inline redondant (`var(--color, #hex)`) ne subsiste dans le markup/style des composants.
- [ ] Toutes les nouvelles variables CSS ont un nom sémantique explicite en anglais/français conforme à la convention.
- [ ] `src/app.css` reste structuré et propre.
- [ ] La commande `npm run check` s'exécute sans erreur.
