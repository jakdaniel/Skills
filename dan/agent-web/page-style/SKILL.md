---
name: page-style
description: Guide de style visuel et d'architecture UI pour les pages KalySync. Basé sur le design élégant et moderne de la page Répertoire Clients ("Beauté & Équilibre"), ce skill définit les patrons de mise en page, cartes Bento KPI, barres de recherche glassmorphism, boutons premium, tableaux à striage zebra et fiches dossiers Hero.
---

# 🎨 Standard de Style de Page - KalySync ("Beauté & Équilibre")

Cette compétence fournit le guide de style visuel, les patterns SCSS et les composants d'interface de référence créés pour le **Répertoire Clients** de KalySync. Utilisez ce skill pour reproduire cette esthétique haut de gamme, fluide et chaleureuse dans toutes les nouvelles pages ou lors du refactoring de pages existantes.

---

## 🌟 Philosophie de Design

Le style **"Beauté & Équilibre"** associe :
- **Chaleur & Luxe** : Fonds en dégradés crème/nude, typographie sérif italique (`Playfair Display`) et touches organiques.
- **Modernité Glassmorphism** : Effets de flou d'arrière-plan (`backdrop-filter: blur(16px)`), bordures translucides et ombres portées en couches.
- **Micro-animations Vivantes** : Retours visuels au survol (élévation, rotation d'icônes, barres indicatrices latérales).
- **Architecture Bento** : Cartes KPI d'en-tête cliquables et épurées pour la visualisation rapide d'informations clés.

---

## 🚨 Règle Impérative : Centralisation des Variables CSS dans `app.css`

> **RÈGLE CRITIQUE D'ARCHITECTURE STYLE :**
> **Toutes les couleurs choisies, les styles de polices (`font-family`, `font-style`, `font-weight`) et les tailles de police (`font-size`) utilisés pour ce skill doivent IMPÉRATIVEMENT être définis ou ajoutés sous forme de variables CSS avec un nom parlant, sémantique et explicite dans le fichier `src/app.css` (ou `ThemeProvider.svelte`).**

### Principes stricts :
1. **Zéro hardcode** : Aucune valeur couleur brute (hexadécimale comme `#3d121c`, `#e8decb` ou `rgba(...)` bruts avec composantes numériques en dur), ni taille ou style de police magique ne doit figurer en dur dans les blocs `<style>` des composants Svelte.
2. **Noms explicites et parlants dans `app.css`** :
   - **Couleurs & Gradients** : `var(--color-bordeaux-title)`, `var(--color-border-warm)`, `var(--color-border-warm-hover)`, `var(--color-bg-nude-light)`, `var(--color-bg-nude-hover)`, `var(--color-bg-header-gradient)`, `var(--color-bg-page-gradient)`, `var(--color-badge-amber-bg)`, `var(--color-badge-amber-text)`, `var(--color-badge-amber-border)`, etc.
   - **Ombres & Transparences RGBA** : Utiliser `rgba(var(--color-primary-rgb), opacity)` ou les variables sémantiques `var(--shadow-card-warm)`, `var(--shadow-card-warm-hover)`, `var(--shadow-bento-chip)`, `var(--shadow-btn-primary)`, `var(--border-primary-subtle)`, etc.
   - **Polices & Tailles** : `var(--font-heading)`, `var(--font-body)`, `var(--font-number)`, `var(--font-size-title-page)`, `var(--font-size-title-modal)`, `var(--font-size-badge)`.
3. **Mise à jour systématique** : Lors de tout nouvel ajout ou ajustement visuel, la variable CSS doit d'abord être créée ou mise à jour dans `src/app.css` avec un nom parlant avant d'être utilisée dans les composants.

---

## 🎨 Palette de Couleurs & Gradients Standard (Tokens `app.css`)

| Élément | Variable CSS dans `app.css` | Valeur | Description |
| :--- | :--- | :--- | :--- |
| **Fond de Page** | `var(--color-bg-page-gradient)` | `linear-gradient(135deg, #fcfbf9 0%, #faf8f5 50%, #f7f4ee 100%)` | Fond perle/nude ultra-lumineux et doux |
| **Bordure Extérieure Page** | `var(--color-border)` | `1px solid #e5e1d8` | Délimitation fine et douce |
| **Surface Carte / Table** | `var(--color-bg-card)` | `#ffffff` | Blanc pur éclatant |
| **Bordures de Cartes** | `var(--color-border-warm)` / `var(--color-border-warm-hover)` | `#e8decb` / `#d4c4b2` | Contour net et raffiné |
| **Titre Principal** | `var(--color-bordeaux-title)` | `#3d121c` | Bordeaux riche et contrasté |
| **Font Titre** | `var(--font-heading)` | `'Playfair Display', serif` | Typographie élégante sérif |
| **En-tête de Table / Modale** | `var(--color-bg-header-gradient)` | `linear-gradient(90deg, #fcebed 0%, #faf0e6 100%)` | Fond rosé/sable pastel |
| **Texte Sombre Chaud** | `var(--color-text-dark-warm)` | `#2b1f1f` | Lisibilité reposante |
| **Ligne Zebra (Paire)** | `var(--color-bg-nude-light)` | `#fdfbf8` | Alternance crème ultra-légère |
| **Ligne Hover** | `var(--color-bg-nude-hover)` | `#fbf5ee` | Fond de survol beige rosé subtil |
| **Badge Ambre / Note** | `var(--color-badge-amber-bg)` / `var(--color-badge-amber-text)` | `#faf3e8` / `#8a5a25` | Étiquette d'information ambrée |

---

## 📐 Composants & Modèles de Structure UI

### 1. Conteneur Principal de Page (`.page-container`)
Le conteneur occupe l'espace disponible avec un padding généreux et des coins arrondis élevés.

```scss
.page-container {
    padding: 30px 40px;
    max-width: 1400px;
    margin: 0 auto;
    height: 100%;
    width: 100%;
    overflow-y: auto;
    overflow-x: hidden;
    display: flex;
    flex-direction: column;
    gap: 15px;
    box-sizing: border-box;
    background: linear-gradient(135deg, #fcfbf9 0%, #faf8f5 50%, #f7f4ee 100%);
    border-radius: var(--radius-lg, 24px);

    @media (max-width: 1024px), (max-height: 700px) {
        padding: 15px 20px;
        gap: 10px;
    }

    @media (max-width: 480px) {
        padding: 10px 15px;
        gap: 5px;
        border-radius: 0;
    }
}
```

---

### 2. En-tête de Page avec Titre Sérif & Bouton Principal

- **Titre** : `Playfair Display`, italique, gras (700), taille 2.3rem.
- **Sous-titre** : Gris atténué (`var(--color-text-muted)`), 0.95rem.
- **Bouton Premium (`.btn-primary-premium`)** : Dégradé primaire avec wrapper d'icône translucide qui tourne de 90° au survol.

```scss
.page-title, h1 {
    font-family: var(--font-heading);
    font-size: var(--font-size-title-page);
    font-weight: 700;
    color: var(--color-bordeaux-title);
    margin: 0 0 4px 0;
    line-height: 1.15;
    font-style: italic;
    letter-spacing: -0.01em;
}

.btn-primary-premium {
    background: linear-gradient(135deg, var(--color-primary) 0%, var(--color-primary-hover) 100%);
    color: #ffffff;
    border: none;
    border-radius: 14px;
    padding: 0 18px 0 10px;
    height: 42px;
    font-weight: 600;
    font-size: var(--font-size-sm, 0.9rem);
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 8px;
    transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: 0 6px 18px rgba(var(--color-primary-rgb), 0.35);

    .icon-wrapper {
        width: 26px;
        height: 26px;
        background: rgba(255, 255, 255, 0.2);
        border-radius: 8px;
        display: flex;
        align-items: center;
        justify-content: center;
        transition: transform 0.25s ease;
    }

    &:hover {
        transform: translateY(-1.5px);
        box-shadow: 0 8px 20px rgba(var(--color-primary-rgb), 0.35);

        .icon-wrapper {
            transform: scale(1.1) rotate(90deg);
        }
    }

    &:active {
        transform: translateY(0) scale(0.98);
    }
}
```

---

### 3. Grille de Cartes Bento KPI (`.bento-stats-grid`)

Permet de basculer les filtres tout en affichant les chiffres clés.

```html
<div class="bento-stats-grid">
    <div 
        class="bento-stat-card" 
        class:active={activeFilter === "all"} 
        onclick={() => activeFilter = "all"}
        role="button" 
        tabindex="0"
    >
        <div class="stat-icon-bg pink">
            <Icon name="users" size={20} />
        </div>
        <div class="stat-info">
            <span class="stat-value">{totalCount}</span>
            <span class="stat-label">Total Clients</span>
        </div>
    </div>
</div>
```

```scss
.bento-stats-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
    gap: 14px;

    .bento-stat-card {
        background: #fdfbf7;
        border: 1px solid rgba(var(--color-primary-rgb), 0.14);
        border-radius: 16px;
        padding: 14px 18px;
        display: flex;
        align-items: center;
        gap: 14px;
        cursor: pointer;
        transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
        box-shadow: 0 4px 14px rgba(128, 61, 74, 0.05);

        &:hover {
            transform: translateY(-2px);
            border-color: rgba(var(--color-primary-rgb), 0.3);
            box-shadow: 0 8px 20px rgba(128, 61, 74, 0.12);
        }

        &.active {
            background: linear-gradient(135deg, #fff1f2 0%, #fdfbf7 100%);
            border-color: var(--color-primary);
            box-shadow: 0 8px 20px rgba(128, 61, 74, 0.15);
        }

        .stat-icon-bg {
            width: 42px;
            height: 42px;
            border-radius: 12px;
            display: flex;
            align-items: center;
            justify-content: center;
            flex-shrink: 0;

            &.pink { background: rgba(var(--color-primary-rgb), 0.1); color: var(--color-primary); }
            &.amber { background: rgba(212, 163, 115, 0.15); color: #bc6c25; }
            &.sage { background: rgba(96, 108, 56, 0.12); color: #606c38; }
        }

        .stat-info {
            display: flex;
            flex-direction: column;

            .stat-value {
                font-family: var(--font-number, sans-serif);
                font-size: 1.35rem;
                font-weight: 700;
                color: var(--color-text-main);
                line-height: 1.1;
            }

            .stat-label {
                font-size: 0.78rem;
                color: var(--color-text-muted);
                font-weight: 600;
                text-transform: uppercase;
                letter-spacing: 0.04em;
            }
        }
    }
}
```

---

### 4. Barre de Recherche Flottante avec Glassmorphism

```scss
.search-bar {
    display: flex;
    align-items: center;
    background: linear-gradient(135deg, rgba(255, 255, 255, 0.94) 0%, rgba(255, 241, 242, 0.88) 100%);
    border: 1px solid rgba(var(--color-primary-rgb), 0.2);
    border-radius: var(--radius-lg, 20px);
    padding: 0 10px 0 20px;
    height: 52px;
    transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    box-shadow: 
        0 10px 28px -6px rgba(128, 61, 74, 0.12),
        0 2px 6px -1px rgba(0, 0, 0, 0.02);

    &.is-focused {
        background: #ffffff;
        border-color: var(--color-primary);
        box-shadow:
            0 0 0 4px rgba(var(--color-primary-rgb), 0.12),
            0 18px 36px -8px rgba(128, 61, 74, 0.2);
        transform: translateY(-1px);

        .search-icon {
            color: var(--color-primary);
            transform: scale(1.08) rotate(-4deg);
        }
    }
}
```

---

### 5. Table / Liste Interactive à Striage Zebra & Effet de Survol

```scss
.contacts-grid {
    background: #fdfbf7;
    border-radius: var(--radius-lg, 20px);
    border: 1px solid rgba(var(--color-primary-rgb), 0.16);
    box-shadow: 
        0 16px 38px -10px rgba(128, 61, 74, 0.14),
        0 4px 12px -2px rgba(0, 0, 0, 0.03);
    overflow: hidden;
    width: 100%;

    .list-header {
        display: grid;
        gap: 15px;
        padding: 2px 20px;
        background: linear-gradient(90deg, #fcebed 0%, #faf0e6 100%);
        border-bottom: 1.5px solid #e8decb;
        font-weight: 700;
        color: #3d121c;
        font-size: 0.78rem;
        text-transform: uppercase;
        letter-spacing: 0.06em;
    }
}

.table-row {
    display: grid;
    gap: 15px;
    padding: 12px 20px; 
    align-items: center;
    border-bottom: 1px solid #eee5db;
    transition: all 0.2s ease;
    position: relative;
    background: #ffffff;

    &:nth-child(even) {
        background: #fdfbf8;
    }

    &::before {
        content: '';
        position: absolute;
        left: 0;
        top: 0;
        bottom: 0;
        width: 3px;
        background: #803d4a;
        opacity: 0;
        transition: opacity 0.2s ease;
    }

    &:hover {
        background: #fbf5ee;
        &::before {
            opacity: 1;
        }
    }
}
```

---

### 6. En-tête Dossier Hero avec Halo Radial (`.client-header`)

Pour les vues détaillées (dossier client, détail de service, etc.), une carte Hero avec un halo radial en arrière-plan apporte une touche sophistiquée.

```scss
.client-header {
    display: flex;
    align-items: center;
    gap: 32px;
    padding: 32px 36px;
    background: linear-gradient(135deg, rgba(var(--color-primary-rgb), 0.14) 0%, #fffbf8 50%, rgba(212, 163, 115, 0.14) 100%);
    border-radius: var(--radius-lg, 24px);
    border: 1px solid rgba(var(--color-primary-rgb), 0.18);
    position: relative;
    overflow: hidden;
    box-shadow: 
        0 18px 40px -10px rgba(128, 61, 74, 0.16),
        0 4px 12px -2px rgba(0, 0, 0, 0.03);

    &::before {
        content: '';
        position: absolute;
        top: -80px;
        right: -80px;
        width: 260px;
        height: 260px;
        background: radial-gradient(circle, rgba(var(--color-primary-rgb), 0.12) 0%, transparent 70%);
        border-radius: 50%;
        pointer-events: none;
        z-index: 0;
    }

    .large-avatar {
        width: 100px;
        height: 100px;
        border-radius: 24px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 2.6rem;
        font-weight: 800;
        transform: rotate(-2deg);
        box-shadow: 0 10px 25px rgba(0, 0, 0, 0.08);
    }
}
```

---

## 🛠 Boutons Secondaires / Action Bar (`.btn-sync-premium`)

Bouton secondaire épuré avec contour fin et conteneur d'icône teinté :

```scss
.btn-sync-premium {
    background: var(--color-bg-card, #ffffff);
    color: var(--color-text-main);
    border: 1px solid var(--color-border, #e5e1d8);
    border-radius: 12px;
    padding: 0 16px 0 8px;
    height: 40px;
    font-weight: 600;
    font-size: var(--font-size-sm, 0.9rem);
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 10px;
    transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.03);

    .icon-wrapper {
        width: 28px;
        height: 28px;
        background: var(--color-primary-bg, #fff1f2);
        border-radius: 8px;
        display: flex;
        align-items: center;
        justify-content: center;
        color: var(--color-primary);
        transition: transform 0.2s ease;
    }

    &:hover {
        background: var(--color-bg-surface, #faf9f6);
        border-color: rgba(var(--color-primary-rgb), 0.3);
        transform: translateY(-1.5px);
        box-shadow: 0 6px 16px rgba(128, 61, 74, 0.1);

        .icon-wrapper {
            transform: scale(1.08);
        }
    }
}
```

---

## 📋 Checklist de Validation pour une Nouvelle Page

Lors du développement ou du refactoring d'une page :

- [ ] **Fond** : Le conteneur principal a-t-il le dégradé nude `linear-gradient(135deg, #faf6f0 0%, #fcf3f4 50%, #f5efe8 100%)` et le bon padding ?
- [ ] **Titre** : Le titre utilise-t-il la police sérif `Playfair Display` en italique avec sa couleur principale ?
- [ ] **Bento KPI** : Les statistiques clés sont-elles sous forme de cartes Bento avec icônes colorées et retours hovers ?
- [ ] **Barre de Recherche** : Si présente, possède-t-elle le verre dépoli (glassmorphism) et l'effet lumineux sur focus ?
- [ ] **Boutons** : Les actions principales utilisent-elles `.btn-primary-premium` et les actions secondaires `.btn-sync-premium` ?
- [ ] **Tableau / Liste** : Le tableau a-t-il des cartes crèmes `#fdfbf7`, un en-tête dégradé `#ede5db` / `#f3ece3`, du zebra striping (`#f4ede2`) et la barre d'indication 4px au survol ?
- [ ] **Responsive** : La page gère-t-elle les breakpoints mobile (`@media (max-width: 600px)`) en masquant si nécessaire les étiquettes et ajustant la grille ?
