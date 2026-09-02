---
name: testing-e2e
description: Spécialiste des tests de bout en bout (End-to-End) avec Playwright pour les parcours utilisateurs critiques.
---

# E2E Testing - Spécialiste E2E (Playwright)
## Vue d'ensemble
Cette skill guide la création de tests **End-to-End (E2E)**. Ces tests simulent un utilisateur réel naviguant dans l'application via un navigateur (Chromium, Firefox, WebKit). L'objectif est une validation **totale et exhaustive** des parcours utilisateurs. Je ne veux pas juste que les situations critiques ou essentielles soient couvertes, je veux que TOUTES les situations possibles de navigation et d'interaction utilisateur soient couvertes sans exception.

**IMPORTANT** Utiliser conjointement avec le sous-skill [`testing-playwright`](file:///C:/Users/jakda/.agent/skills/tests/testing-playwright/SKILL.md).

## Quand Utiliser Cette Skill
### ✅ Utilise les Tests E2E Pour :
- Les parcours critiques (Smoke Tests) : Login, Inscription, Création de rendez-vous.
- Les tests de non-régression sur l'interface utilisateur.
- Vérifier que le Frontend, le Backend et la DB communiquent parfaitement dans un environnement proche de la prod.
### ❌ N'Utilise PAS les Tests E2E Pour :
- Des tests de logique métier simple (ex: calcul de prix) → [`testing-unit`](file:///C:/Users/jakda/.agent/skills/tests/testing-unit/SKILL.md).
- Couvrir 100% des cas d'erreur API → [`testing-integration`](file:///C:/Users/jakda/.agent/skills/tests/testing-integration/SKILL.md).
- Les tests E2E sont **lents et coûteux**, gardez-les pour le "Happy Path" et les bugs critiques passés.
## Structure et Conventions
### Localisation
Les tests se trouvent généralement dans le répertoire `/tests` à la racine ou tel que configuré dans `playwright.config.ts`.
### Nommage
- Fichiers : `nom-du-test.test.ts` or `nom-du-test.spec.ts`.
- Blocks : Utilisez des descriptions métier (ex: "L'utilisateur doit pouvoir modifier son mot de passe").

### ✅ Sécurité & Robustesse (100% - Automatisé)
*   [x] Gestion des conditions de course au démarrage (Attente auto-initialisation Google).
*   [x] Filtrage intelligent des logs (Réduction du bruit des bots, masquage PII).
*   [x] Stabilité des sessions (Tolérance aux décalages temporels).
*   [x] Protection CSRF et Limitation de débit (Rate Limiting) sur les endpoints sensibles (Logs).
*   [x] Purge automatique des logs (GDPR Compliance).

## Workflow de Test E2E
### 1. Sélecteurs (Locators)
Utilisez des sélecteurs robustes qui ne changent pas avec le style (évitez les classes CSS comme `.blue-button`).
- **Préféré :** `page.getByRole('button', { name: 'Valider' })`
- **Préféré :** `page.getByTestId('submit-btn')` (en ajoutant `data-testid` dans le code HTML).
- **Éviter :** `page.locator('div > span > button')`.
### 2. Assertions
Utilisez les assertions asynchrones de Playwright qui incluent des attentes automatiques (auto-wait).
```typescript
await expect(page.getByText('Succès !')).toBeVisible();
await expect(page)).toHaveURL(/.*dashboard/);
```
## Exemple de Test E2E
```typescript
import { test, expect } from '@playwright/test';
test.describe('Gestion des Rendez-vous', () => {
  
  test.beforeEach(async ({ page }) => {
    // Connexion automatique ou injection de session
    await page.goto('/login');
    await page.getByLabel('Email').fill('admin@test.com');
    await page.getByLabel('Mot de passe').fill('password123');
    await page.getByRole('button', { name: 'Se connecter' }).click();
  });
  test('Doit permettre de créer un nouveau RDV', async ({ page }) => {
    await page.goto('/calendar');
    await page.getByRole('button', { name: 'Nouveau RDV' }).click();
    
    // Remplissage du formulaire
    await page.getByLabel('Client').fill('Jean Dupont');
    await page.getByLabel('Heure').fill('14:00');
    await page.getByRole('button', { name: 'Enregistrer' }).click();
    // Vérification
    await expect(page.locator('.calendar-event')).toContainText('Jean Dupont');
    await expect(page.getByText('Rendez-vous créé avec succès')).toBeVisible();
  });
});
```
## Gestion des Flaky Tests (Tests Instables)
Les tests E2E sont souvent fragiles à cause du réseau ou du rendu.
- **Attente explicite :** Utilisez `page.waitForLoadState()` ou `page.waitForResponse()` après une action importante.
- **Isolation :** Chaque test doit être indépendant. Utilisez des données fraiches par test si possible.
- **Trace Viewer :** Utilisez `npx playwright show-trace` pour voir pourquoi un test a échoué dans la CI.
## Mise à jour de la documentation
Conformément aux règles du projet, après avoir ajouté un test E2E :
1. Mettre à jour `docs/TESTS_E2E.md`.
2. Mettre à jour la section **"État de la Couverture Fonctionnelle"**.
## Commandes Utiles
```bash
# Lancer tous les tests E2E
npx playwright test
# Lancer un test spécifique
npx playwright test tests/login.test.ts
# Lancer en mode debug (avec interface graphique)
npx playwright test --debug
```
## Checklist pour un bon test E2E
- [ ] Le test porte-t-il sur un parcours critique ?
- [ ] Les sélecteurs sont-ils robustes (`getByRole`, `data-testid`) ?
- [ ] Le test nettoie-t-il ses données ou utilise-t-il un isolat ?
- [ ] La documentation `docs/TESTS_E2E.md` est-elle à jour ?
---
**Skill liée :** `testing.pyramid.md` pour décider si un test doit être E2E ou Unitaire.