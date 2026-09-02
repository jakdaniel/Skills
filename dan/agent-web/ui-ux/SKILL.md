---
name: ui-ux
description: boîte à outils d'intelligence de conception optimisée pour **KalySync** et **Svelte 5**. Elle fournit des recommandations de design premium basées sur des bases de données de styles, couleurs, et typographies, désormais accessibles via des fichiers Markdown de référence.
---


## Comment utiliser cette compétence (Sans Python)

Suivez ce workflow pour chaque tâche UI/UX :

### Étape 1 : Analyser les besoins
Identifiez le type de produit (Agenda, CRM, Dashboard Beauté) et l'ambiance recherchée (Luxueux, Minimaliste, Zen).

### Étape 2 : Consulter les Références (Action Directe)
Au lieu d'exécuter un script, lisez directement les fichiers de référence pour choisir les tokens de design :

1.  **Couleurs** : `read_file` sur `.gemini/skills/ui-ux-pro-max/docs/COLORS.md`
2.  **Styles** : `read_file` sur `.gemini/skills/ui-ux-pro-max/docs/STYLES.md`
3.  **Typographie** : `read_file` sur `.gemini/skills/ui-ux-pro-max/docs/TYPOGRAPHY.md`

### Étape 3 : Définir le Design System
Combinez les informations trouvées pour créer le Design System de la fonctionnalité.
*Exemple pour KalySync (Beauté) :*
- **Style** : Soft UI Evolution (ID 19 dans STYLES.md)
- **Couleurs** : Beauty/Spa (ID 34 dans COLORS.md)
- **Typographie** : Classic Elegant (ID 1 dans TYPOGRAPHY.md)

### Étape 4 : Implémentation Svelte 5
- Utilisez exclusivement les **Runes Svelte 5** (`$state`, `$derived`, `$props`).
- Respectez les standards de design de KalySync (variables CSS globales dans `app.css` : `var(--font-heading)`, `var(--font-body)`, `var(--font-number)`, `var(--font-size-title-page)`, `var(--font-size-title-modal)`, `var(--font-size-base)`, `var(--font-size-md)`, `var(--font-size-sm)`, `var(--font-size-xs)`).

---

## Guide de Référence Rapide (Checklist)

- [ ] **Svelte 5** : Utilisation correcte des runes.
- [ ] **Esthétique** : Pas d'emojis comme icônes (utiliser Lucide/Heroicons).
- [ ] **Interactions** : `cursor-pointer` sur les éléments cliquables + Transitions fluides.
- [ ] **Couleurs & Fonds** : Utilisation des variables globales CSS de KalySync (`var(--color-bg-card)`, `var(--color-bg-main)`).
- [ ] **Typographie & Tailles** : Utilisation stricte des variables de polices (`var(--font-heading)`, `var(--font-body)`, `var(--font-number)`) et de tailles (`var(--font-size-*)`). Zéro valeur en dur (`px`/`rem`/font name).
- [ ] **Contraste** : Texte lisible (WCAG AA).
- [ ] **Responsive** : Testé sur mobile et desktop.

---

**Données Sources :** Situées dans `.agent/skills/ui-ux/data/` (CSV) et `.agent/skills/ui-ux/docs/` (Markdown).
