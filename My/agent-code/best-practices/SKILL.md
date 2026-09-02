---
name: best-practices
description: Utiliser lors du développement Svelte ou SvelteKit afin de garantir un code moderne, performant, maintenable et conforme aux meilleures pratiques Svelte 5 / SvelteKit 2. À déclencher pour revue de code, optimisation ou architecture.
---

# My Skill

Ce skill fournit un cadre clair et moderne pour construire des applications **Svelte 5** et **SvelteKit 2** de haute qualité.

Objectifs principaux :
- Lisibilité et maintenabilité
- Performance réelle (SSR, streaming, bundle size)
- Architecture claire et évolutive
- Compatibilité SSR / Edge / Cloud
- Alignement avec les patterns Svelte 5 (Runes)

---

## Quand utiliser ce skill

- Création ou refactorisation de composants Svelte
- Revue de code Svelte / SvelteKit
- Optimisation des performances
- Structuration d'un projet SvelteKit
- Diagnostic SSR / hydration / bundle

---

## Doctrine Svelte 5 (IMPORTANT)

### Règles fondamentales

- `$state`, `$derived`, `$effect` sont la norme
- Les stores classiques (`writable`, `derived`) sont réservés à :
  - interop legacy
  - API publique stable
  - état transverse complexe
- Ne pas mélanger runes et stores pour un même domaine fonctionnel
- L'état local doit rester local autant que possible

---

## Hiérarchie des règles

### 🔴 OBLIGATOIRE
- Aucune mutation directe
- Aucune manipulation DOM hors actions / `onMount`
- Aucun accès navigateur en SSR
- Clés obligatoires dans les listes
- Aucun effet secondaire dans `$derived`

### 🟠 FORTEMENT RECOMMANDÉ
- `$state` / `$derived` pour l'état local
- SSR activé par défaut
- Imports dynamiques pour les librairies lourdes
- Requêtes parallèles dans `load()`

### 🟢 OPTIONNEL
- Streaming de données lentes
- Virtual scrolling
- Animations avancées
- Edge runtime

---

## 1. Conception des composants

### Principes
- Un composant = une responsabilité
- ≤ 200 lignes par composant
- Extraire la logique réutilisable (actions, utils)

```svelte
<script>
  let count = $state(0);
  function increment() {
    count++;
  }
</script>

<button onclick={increment}>
  Compteur : {count}
</button>
```

### Styles
- CSS scopé par défaut
- `:global()` uniquement au layout racine
- Favoriser Tailwind ou variables CSS

---

## 2. Réactivité (Runes Svelte 5)

```svelte
<script>
  let count = $state(0);
  let doubled = $derived(count * 2);

  $effect(() => {
    console.log(count);
  });
</script>
```

### Bonnes pratiques
- `$derived` = calcul pur uniquement
- `$effect` = effets secondaires uniquement
- Découper les calculs complexes en plusieurs dérivés
- Éviter toute dépendance implicite

---

## 3. Gestion de l'état

### État local (par défaut)

```svelte
<script>
  let form = $state({
    email: '',
    age: 0
  });
</script>
```

### Stores (cas spécifiques seulement)

```javascript
import { writable } from 'svelte/store';

export const utilisateur = writable(null);
```

---

## 4. DOM & effets secondaires

### ❌ À éviter

```javascript
document.querySelector('#el').style.display = 'none';
```

### ✅ Actions Svelte

```svelte
<script>
  function clickOutside(node, callback) {
    function handle(event) {
      if (!node.contains(event.target)) callback();
    }

    document.addEventListener('click', handle);

    return {
      destroy() {
        document.removeEventListener('click', handle);
      }
    };
  }
</script>
```

---

## 5. Performance

### Bundle
- Cible réaliste : < 50KB gzip initial
- Éviter toute dépendance > 15KB non critique

### Code splitting

```svelte
<script>
  import { onMount } from 'svelte';

  let Chart;

  onMount(async () => {
    Chart = (await import('./Chart.svelte')).default;
  });
</script>
```

### Images
- `loading="lazy"`
- Formats WebP / AVIF
- `@sveltejs/enhanced-img` recommandé

---

## 6. Architecture SvelteKit

### Modes de rendu
- **SSR** par défaut
- **CSR** pour dashboards
- **SSG** pour contenu statique
- **Hybride** recommandé

```javascript
export const ssr = true;
export const prerender = false;
```

### Requêtes parallèles (éviter les cascades)

```javascript
export async function load({ fetch }) {
  const [user, posts] = await Promise.all([
    fetch('/api/user').then(r => r.json()),
    fetch('/api/posts').then(r => r.json())
  ]);

  return { user, posts };
}
```

### Streaming des données lentes

```javascript
export async function load({ fetch }) {
  return {
    rapide: await fetch('/api/rapide').then(r => r.json()),
    lent: fetch('/api/lent').then(r => r.json())
  };
}
```

---

## 7. Formulaires & progressive enhancement

```svelte
<script>
  import { enhance } from '$app/forms';
</script>

<form method="POST" use:enhance>
  <input name="email" required />
  <button type="submit">Envoyer</button>
</form>
```

---

## 8. TypeScript

```typescript
import type { PageServerLoad } from './$types';

export const load: PageServerLoad = async () => {
  return { ok: true };
};
```

---

## 9. Tests & qualité

### Outils recommandés :
- `vitest`
- `@testing-library/svelte`
- `eslint-plugin-svelte`

```javascript
expect(screen.getByText('Compteur : 1')).toBeInTheDocument();
```

---

## 10. Organisation du projet

```
src/
├── lib/
│   ├── components/
│   ├── stores/
│   ├── actions/
│   ├── utils/
│   └── types/
├── routes/
│   ├── +layout.svelte
│   ├── +page.svelte
│   └── api/
```

---

## Anti-patterns Svelte 5 (CRITIQUE)

- ❌ `$derived` avec effets secondaires
- ❌ `$effect` avec dépendances implicites
- ❌ `$state` global non encapsulé
- ❌ Accès `window` hors environnement navigateur
- ❌ Routes API quand `load()` suffit

---

## IA & Web moderne

- Appels IA **server-only**
- Clés API **jamais côté client**
- Streaming recommandé pour réponses longues
- `enhance()` idéal pour chat IA
- Edge runtime pour réduire la latence

---

## Checklist finale

- [ ] SSR actif
- [ ] Aucune mutation directe
- [ ] Imports dynamiques pour libs lourdes
- [ ] Images optimisées
- [ ] Lighthouse > 90
- [ ] Accessibilité validée
- [ ] États de chargement visibles
- [ ] Gestion des erreurs