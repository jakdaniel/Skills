---
name: svelte-standards
description: Directives et conventions de développement pour Svelte 5 et SvelteKit 2 (Runes $state, $derived, $effect, architecture de composant, SSR, streaming, progressive enhancement, performance et bundle size). À utiliser pour tout développement, refactoring ou revue de code Svelte 5.
---

# ⚡ Svelte 5 & SvelteKit 2 Standards (`svelte-standards`)

Ce skill fournit un cadre clair et moderne pour construire des applications **Svelte 5** et **SvelteKit 2** de haute qualité, parfaitement alignées sur le modèle réactif des Runes.

---

## 🎯 Quand utiliser ce skill

- Création ou refactorisation de composants Svelte 5
- Revue de code de la réactivité et des formulaires SvelteKit
- Optimisation des performances SSR, streaming et taille du bundle
- Diagnostic de l'hydratation et du code splitting

---

## ⚡ Doctrine Svelte 5 (Runes Exclusives)

### Règles Fondamentales

- `$state`, `$derived`, `$effect` et `$props` sont **la norme obligatoire**.
- Les stores Svelte 4 (`writable`, `derived`) sont **strictement réservés** à l'interop legacy ou aux librairies tierces exigeant ce format.
- Ne jamais mélanger runes et stores pour un même domaine fonctionnel.
- L'état local doit rester local autant que possible.

---

## 🚦 Hiérarchie des Règles

### 🔴 OBLIGATOIRE
- Aucune mutation directe d'état sans rune.
- Aucune manipulation DOM manuelle (`document.createElement`) hors actions / `onMount`.
- Aucun accès au navigateur (`window`, `document`) pendant le SSR.
- Clés obligatoires (`{#each items as item (item.id)}`) dans les boucles.
- Aucun effet secondaire dans un bloc `$derived`.

### 🟠 FORTEMENT RECOMMANDÉ
- `$state` / `$derived` pour l'état local et partagé via modules `.svelte.ts`.
- SSR activé par défaut.
- Imports dynamiques (`await import(...)`) pour les bibliothèques lourdes.
- Requêtes parallèles (`Promise.all`) dans la fonction `load()`.

### 🟢 OPTIONNEL
- Streaming des données lentes via SvelteKit `load()`.
- Virtual scrolling pour les listes massives.
- Edge runtime et pré-chargement dynamique.

---

## 🧩 1. Conception des Composants

### Principes
- **Un composant = Une seule responsabilité**.
- **Limite Stricte de Lignes** : ≤ 200 lignes par sous-composant, max **400 lignes** par composant `.svelte` principal.
- Extraire la logique métier dans des modules `.ts` ou des services réactifs `.svelte.ts`.

```svelte
<script lang="ts">
  type Props = { initialCount?: number };
  let { initialCount = 0 }: Props = $props();

  let count = $state(initialCount);
  let doubled = $derived(count * 2);

  function increment() {
    count++;
  }
</script>

<button onclick={increment}>
  Compteur : {count} (Doublé : {doubled})
</button>
```

---

## 🔄 2. Réactivité (Runes Svelte 5)

```svelte
<script lang="ts">
  let search = $state('');
  let results = $derived(items.filter(i => i.name.includes(search)));

  $effect(() => {
    // Effets secondaires uniquement (ex: synchronisation externe, canvas, etc.)
    console.log('Recherche mise à jour:', search);
  });
</script>
```

### Bonnes Pratiques
- `$derived` = Calcul pur sans effet secondaire.
- `$effect` = Synchronisation externe ou effets secondaires uniquement.
- Découper les calculs complexes en plusieurs dérivations `$derived`.

---

## 🚀 3. Performance & Code Splitting

### Bundle & Chargement
- Cible initiale : bundle léger avec lazy loading des vues secondaires.
- Imports dynamiques pour les bibliothèques lourdes :

```svelte
<script lang="ts">
  import { onMount } from 'svelte';

  let ChartComponent = $state<any>(null);

  onMount(async () => {
    const module = await import('./Chart.svelte');
    ChartComponent = module.default;
  });
</script>
```

---

## 🛠️ 4. Architecture SvelteKit 2

### Formulaires & Progressive Enhancement
- Toujours utiliser `use:enhance` sur les formulaires SvelteKit pour une meilleure UX :

```svelte
<script lang="ts">
  import { enhance } from '$app/forms';
</script>

<form method="POST" use:enhance>
  <input name="email" type="email" required />
  <button type="submit">Enregistrer</button>
</form>
```

### Server Load Parallelisation
```typescript
import type { PageServerLoad } from './$types';

export const load: PageServerLoad = async ({ fetch }) => {
  const [appointments, clients] = await Promise.all([
    fetch('/api/appointments').then(r => r.json()),
    fetch('/api/clients').then(r => r.json())
  ]);

  return { appointments, clients };
};
```

---

## 🚫 Anti-Patterns Svelte 5 (CRITIQUE)

- ❌ `$derived` contenant des appels de fonctions avec effets secondaires.
- ❌ `$effect` avec des dépendances circulaires ou non bornées.
- ❌ État `$state` global non encapsulé dans une classe ou un fichier `.svelte.ts`.
- ❌ Utilisation de `placeholder` sur les champs de saisie.

---

## 📋 Checklist de Validation Svelte 5

- [ ] Les runes `$state`, `$derived`, `$props` sont utilisées à la place des stores V4.
- [ ] Le composant ne dépasse pas 400 lignes.
- [ ] Les formulaires utilisent `use:enhance` et la validation typée.
- [ ] La commande `npm run check` ne remonte aucune erreur de syntaxe runes.
