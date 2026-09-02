---
name: testing-diamond
description: Spécialiste de la stratégie et de l'écriture de tests selon le Diamant/Trophée de tests (Testing Diamond / Testing Trophy) pour maximiser la confiance et le ROI des tests.
---

# Testing Diamond & Testing Trophy - Strategy & Best Practices

## 🎯 Vue d'ensemble & Philosophie

Le **Diamant de Tests** (ou **Testing Trophy** popularisé par Kent C. Dodds et approfondi par Michael Scotto) est une stratégie d'automatisation des tests axée sur la **confiance (Confidence)** et le **retour sur investissement (ROI)**, plutôt que sur la simple vitesse d'exécution ou le volume brut de code couvert.

Contrairement à la Pyramide de Tests traditionnelle qui sur-investit dans la couche unitaire (au risque de tester des détails d'implémentation internes et fragiles), le Diamant/Trophée place **l'effort principal sur les tests d'intégration**.

> **Principe directeur :** *"Write tests. Not too many. Mostly integration."* — Kent C. Dodds
> ⚡ **Directive Workflow :** Ne JAMAIS générer de plan d'exécution pour les tâches Testing Diamond. Exécuter directement les actions de correction, désambiguïsation et consolidation, puis fournir le rapport avec l'état final.

---

## 🏆 Les 4 Niveaux du Trophée de Tests

```
         /\          1. E2E / System (Sommet étroit - Parcours critiques Playwright)
        /  \
       /    \
      /______\       2. Integration (Cœur le plus large - ROI maximal, composants & API réels D1)
      \      /
       \    /        3. Unit (Base plus étroite - Logique & algorithmes purs isolés)
        \  /
         \/          4. Static (Socle - TypeScript, linters, Svelte 5 Runes check)
```

### 1. Tests Statiques (Socle)
* **But :** Détecter les fautes de frappe, erreurs de types, syntaxes invalides et non-respect des conventions dès l'écriture du code, sans exécuter la moindre suite de tests.
* **Dans KalySync :** TypeScript (`npm run check`), Svelte Check (validation des Runes Svelte 5), ESLint.

### 2. Tests Unitaires (Base Étroite)
* **But :** Tester la logique pure isolée, les algorithmes complexes, les calculs métier et les fonctions utilitaires sans aucun effet de bord.
* **Critères d'un bon test unitaire :**
  1. La fonction contient une logique décisionnelle ou un algorithme non trivial (plusieurs branches, cas limites, calculs).
  2. Elle peut être exécutée sans simuler le DOM ou la base de données.
  3. Un échec indique sans ambiguïté un bug dans l'algorithme lui-même.
* **À éviter :** Tester des passe-plats simples, des accesseurs/mutateurs basiques, ou la structure interne des composants/stores.

### 3. Tests d'Intégration (Cœur Large - Priorité № 1)
* **But :** Vérifier que plusieurs unités (composants UI, endpoints API, base de données D1, services) interagissent correctement ensemble du point de vue de l'utilisateur ou du contrat d'interface.
* **Pourquoi c'est le niveau le plus large ?**
  * Offre le meilleur ratio confiance / coût de maintenance.
  * Réduit le besoin de mocks artificiels : on teste avec des dépendances réelles autant que possible (ex: base SQLite D1 locale).
  * Les tests restent résilients au refactoring interne tant que le comportement observable ne change pas.
* **Dans KalySync :** Tests de composants Svelte (Testing Library), tests d'endpoints SvelteKit (+server.ts), validation de l'isolation multi-tenant `compagnie_id`.

### 4. Tests E2E / Système (Sommet Étroit)
* **But :** Valider les parcours utilisateurs critiques de bout en bout dans un vrai navigateur.
* **Usage :** Limité aux scénarios indispensables (authentification, création de rendez-vous, flux d'onboarding, règles juridiques ou réglementaires), car ils sont plus lents et plus coûteux à maintenir.
* **Dans KalySync :** Playwright (`npm run test:e2e`).

---

## 🧭 Grille de Décision : Choisir le Bon Niveau de Test

Pour chaque fonctionnalité ou exigence, se poser ces **6 questions heuristiques** (adaptées de Michael Scotto / *The Quality Index*) :

| Question Heuristique | Si OUI ➔ Niveau Recommandé | Raisonnement & Impact |
| :--- | :--- | :--- |
| **1. La fonctionnalité implique-t-elle des exigences légales ou réglementaires ?** | **E2E / Système** | Nécessite la plus haute fidélité avec l'expérience utilisateur réelle. |
| **2. Quelle est la criticité métier (ex: risque d'incident majeur) ?** | **E2E (Parcours principal)** | Sécuriser le *Happy Path* critique de bout en bout. |
| **3. Combien de variations/combinaisons de données doivent être validées ?** | **Intégration** | Valider des dizaines de combinaisons en E2E est trop lent. Les tests d'intégration de composants ou d'API permettent de tester rapidement toutes les permutations. |
| **4. Quel est le niveau le plus bas où la fonctionnalité est observable de façon réaliste ?** | **Le niveau le plus bas suffisant** | Si la logique d'un endpoint backend peut être validée par un test API avec D1, inutile d'instancier toute l'UI en E2E. |
| **5. La fonctionnalité dépend-elle spécifiquement du navigateur ?** (Cookies, historique, thèmes, observers) | **E2E (Playwright)** | Valider le comportement réel dans un véritable moteur de rendu. |
| **6. Quelles sont les compétences et le stack de l'équipe ?** | **Pragmatisme du stack** | Exploiter pleinement Svelte 5, Vitest, Playwright et Cloudflare D1 local sans sur-ingénierie. |

---

## 🛠️ Règles d'Écriture de Bons Tests (Best Practices)

### 1. Tester le Comportement, Pas l'Implémentation
* ❌ **Mauvais :** Vérifier la valeur d'une rune interne `$state` ou l'appel d'une fonction privée.
* ✅ **Bon :** Vérifier que lorsqu'un utilisateur remplit le formulaire et clique sur "Valider", l'élément apparaît dans le DOM ou l'API retourne HTTP 200 avec les bonnes données.

### 2. Éviter le Sur-Mocking (Mock Minimalism)
* Ne mocker que les services tiers externes hors du périmètre du projet (ex: Resend pour les emails, Stripe pour les paiements).
* Ne PAS mocker la base de données D1 lorsqu'une base D1 SQLite locale est disponible pour les tests.
* Ne PAS mocker les composants enfants Svelte sauf cas d'isolation extrême.

### 3. Sécurité & Multi-Tenant (Spécifique KalySync)
* Tout test d'intégration d'API ou de base de données doit vérifier explicitement le respect du filtre `WHERE compagnie_id = ?`.
* Vérifier qu'un utilisateur de `compagnie_A` ne peut en aucun cas lire, modifier ou supprimer les données de `compagnie_B`.

### 4. Sélecteurs Résilients au Refactoring
* ❌ **Éviter :** `page.locator('div > div.flex > button:nth-child(2)')`
* ✅ **Privilégier :** `screen.getByRole('button', { name: /enregistrer/i })` ou `data-testid="save-appointment-btn"`.

### 5. Interdiction Absolue des Tests en Double (Règle Anti-Duplication)
* ❌ **Interdiction des intitulés identiques :** Il est strictement interdit d'ajouter des tests (`it` ou `test`) partageant exactement le même titre au sein d'un même fichier. Chaque test doit comporter un nom unique précisant le cas spécifique ou la méthode testée (ex: `"retourne 500 et logue logger.error quand la DB throw (GET)"`).
* ❌ **Interdiction des fichiers satellites redondants :** Ne JAMAIS créer de fichiers de tests fragmentés du type `-missing-coverage`, `-missing-coverage-2`, `-robustness` ou de doublons d'intégration (ex: `appointments.int.test.ts` vs `appointments_api.int.test.ts`). Tout nouveau cas de test doit obligatoirement être intégré dans le fichier de test unitaire ou d'intégration principal du module concerné.
* ❌ **Éviter le chevauchement E2E / Intégration sur les nouveaux tests :** Ne pas recréer de nouveaux tests E2E Playwright pour des cas déjà couverts à 100 % par des tests d'intégration Vitest légers et rapides, sauf pour le *Happy Path* d'un parcours utilisateur critique.
* 📌 **Conservation des tests E2E existants :** Toujours **garder les tests E2E existants**, même s'ils sont en double avec un test d'intégration (ne jamais supprimer les tests E2E déjà écrits).
* 📌 **Conservation des tests intégration existants :** Toujours **garder les tests intégration existants**, même s'ils sont en double avec un test e2e (ne jamais supprimer les tests intégration déjà écrits).

---

## 💻 Exemples Concrets par Niveau (Stack KalySync)

### A. Test Unitaire (Vitest) - Logique Pure
```typescript
// src/features/calendar/logic/calendarLayout.test.ts
import { describe, it, expect } from 'vitest';
import { calculateEventPositions } from './calendarLayout';

describe('calculateEventPositions', () => {
  it('doit superposer correctement deux rendez-vous en chevauchement', () => {
    const events = [
      { id: '1', start: '09:00', end: '10:00' },
      { id: '2', start: '09:30', end: '10:30' }
    ];
    const layout = calculateEventPositions(events);
    expect(layout['1'].width).toBe(50);
    expect(layout['2'].width).toBe(50);
    expect(layout['2'].left).toBe(50);
  });
});
```

### B. Test d'Intégration (Vitest + Endpoint SvelteKit / D1)
```typescript
// src/routes/api/appointments/+server.test.ts
import { describe, it, expect } from 'vitest';
import { POST } from './+server';

describe('POST /api/appointments (Integration D1)', () => {
  it('doit refuser la création si la compagnie_id est incorrecte', async () => {
    const request = new Request('http://localhost/api/appointments', {
      method: 'POST',
      body: JSON.stringify({ title: 'Coupe', compagnie_id: 'OTHER_TENANT' })
    });
    
    const response = await POST({ request, locals: { compagnie_id: 'MY_TENANT' } } as any);
    expect(response.status).toBe(403);
  });
});
```

### C. Test E2E (Playwright) - Parcours Critique
```typescript
// tests/e2e/appointment-flow.spec.ts
import { test, expect } from '@playwright/test';

test('Réservation de rendez-vous complète par le professionnel', async ({ page }) => {
  await page.goto('/login');
  await page.fill('input[name="email"]', 'pro@kalysync.com');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button[type="submit"]');

  await page.click('button[data-testid="new-appointment-btn"]');
  await page.fill('input[name="clientName"]', 'Alice Dupont');
  await page.click('button[data-testid="save-btn"]');

  await expect(page.locator('.calendar-grid')).toContainText('Alice Dupont');
});
```

---

## ⚡ Commandes de Exécution et Validation

```bash
# 1. Analyse statique (Types & Svelte)
npm run check

# 2. Execution des tests Unitaires et d'Integration
npm run test:unit
npm run test:int

# 3. Execution des tests E2E
npm run test:e2e

# 4. Execution de tout les tests pour validation complète
npm run test:all

```