---
name: responsive-design
description: Compétence spécialisée pour analyser, adapter et valider le design d'une page Svelte/CSS afin de garantir qu'elle soit parfaitement fonctionnelle, fluide et esthétique sur tous les types d'écrans (Mobile, Tablette, Ordinateur).
---

# 📱💻 Skill Responsive Design & Multi-Support

## 📌 Aperçu
Cette compétence guide l'agent Antigravity dans l'audit, l'implémentation et la validation du design adaptatif (responsive design) pour l'application **KalySync**.
Elle garantit une expérience utilisateur optimale et sans défaut d'affichage sur les 3 principaux formats d'appareils : **Smartphone (Mobile)**, **Tablette**, et **Ordinateur (Desktop)**.

---

## 📐 Breakpoints & Viewports Standards

| Appareil | Type | Largeur Viewport | Mise en Page / Pattern UI |
| :--- | :--- | :--- | :--- |
| 📱 **Mobile** | Smartphone Portrait | `375px` à `639px` | 1 Colonne, Bottom Navigation ou Sidebar Drawer, modales plein écran, cibles tactiles ≥ 44px. |
| 📱 **Tablette** | Tablette / Dual Pane | `640px` à `1023px` | 1 à 2 Colonnes, Sidebar compacte ou repliable, modales recentrées avec padding adaptatif. |
| 🖥️ **Ordinateur** | Desktop / Grand Écran | `1024px` à `1920px+` | Multi-colonnes (Grille Agenda complet), Sidebar étendue, interactions hover riches et raccourcis clavier. |

---

## 🔍 Checklist d'Audit Responsive (Par Appareil)

### 1. 📱 Smartphone / Mobile (`< 640px`)
- [ ] **Zéro Scroll Horizontal** : `html`, `body` et les conteneurs principaux n'ont aucun débordement (`overflow-x: hidden` / `max-width: 100%`).
- [ ] **Cibles Tactiles (Touch Targets)** : Tous les boutons, icônes et liens ont une zone de clic minimale de `44px × 44px`.
- [ ] **Navigation Adaptée** : Les menus latéraux complexes sont convertis en tiroir coulissant (Drawer) ou navigation inférieure.
- [ ] **Modales & Formulaires** : Les modales s'affichent en plein écran ou prennent au moins 95% de la largeur avec bouton d'action bien accessible.
- [ ] **Safe Areas (iOS / Android)** : Prise en compte du `env(safe-area-inset-bottom)` et `env(safe-area-inset-top)`.
- [ ] **Clavier Virtuel** : Les champs de saisie restent visibles lorsque le clavier mobile s'ouvre (utilisation de `100dvh` au lieu de `100vh`).

### 2. 📱 Tablette (`640px` à `1023px`)
- [ ] **Transition Fluide** : Redimensionnement propre entre le mode portrait et paysage.
- [ ] **Composants Hybrides** : Support simultané des interactions tactiles (`touch`) et de la pointeuse/souris.
- [ ] **Optimisation des Grilles** : Passage dynamique de 1 à 2 ou 3 colonnes (`grid-template-columns: repeat(auto-fit, minmax(280px, 1fr))`).

### 3. 🖥️ Ordinateur / Desktop (`≥ 1024px`)
- [ ] **Utilisation de l'Espace** : Mise en page multi-colonnes tirant parti des grands écrans sans étirer exagérément le texte (`max-width` sur les conteneurs de lecture).
- [ ] **Interactions Hover Riches** : Tooltips, menus survolés et effets visuels avancés.
- [ ] **Navigation Complète** : Sidebar dépliée avec libellés textuels et raccourcis clavier explicites.

---

## 🛠️ Directives d'Analyse du Code Svelte & CSS

1. **Interdiction des Largeurs Fixes Critiques** :
   - ❌ Éviter `width: 600px;` ou `width: 1200px;`
   - ✅ Préférer `max-width: 600px; width: 100%;`

2. **Unités Adaptatives** :
   - Utiliser `rem`, `em`, `%`, `ch` pour le texte et les espaces.
   - Utiliser `clamp(min, val, max)` pour la typographie fluide.
   - Utiliser `dvh` / `svh` (Dynamic/Small Viewport Height) pour les conteneurs plein écran sur mobile.

3. **Media Queries & CSS Modernes** :
   ```css
   /* Mobile-first par défaut */
   .container {
     display: flex;
     flex-direction: column;
     gap: 1rem;
   }

   /* Tablette */
   @media (min-width: 640px) {
     .container {
       flex-direction: row;
     }
   }

   /* Desktop */
   @media (min-width: 1024px) {
     .container {
       display: grid;
       grid-template-columns: 280px 1fr;
     }
   }
   ```

---

## 🧪 Procédure de Validation

Lorsqu'on vous demande de vérifier le design d'une page :
1. **Inspection du Code CSS/Svelte** : Vérifier les classes et media queries du composant cible.
2. **Vérification des 3 Viewports** :
   - Tester le rendu ou lancer Playwright en simulant :
     - Mobile : `375 × 667` (iPhone SE) ou `390 × 844` (iPhone 12/13/14)
     - Tablette : `768 × 1024` (iPad)
     - Desktop : `1440 × 900` (MacBook / Écran HD)
3. **Correction & Optimisation** : Ajuster les règles CSS ou les Runes Svelte 5 pour garantir une réactivité parfaite sans régression visuelle.
