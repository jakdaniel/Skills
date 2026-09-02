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
      /______\       2. Integration (Cœur le plus large - ROI maximal, services & API réels)
      \      /
       \    /        3. Unit (Base plus étroite - Logique & algorithmes purs isolés)
        \  /
          \/          4. Static (Socle - Nullable C#, analyzers Roslyn, dotnet build)
```

### 1. Tests Statiques (Socle)
* **But :** Détecter les fautes de frappe, erreurs de types, syntaxes invalides et non-respect des conventions dès l'écriture du code, sans exécuter la moindre suite de tests.
* **Dans un projet C# :** `dotnet build` (warnings en erreurs, nullabilité), analyzers Roslyn, `dotnet format --verify-no-changes`, `.editorconfig`.

### 2. Tests Unitaires (Base Étroite)
* **But :** Tester la logique pure isolée, les algorithmes complexes, les calculs métier et les fonctions utilitaires sans aucun effet de bord.
* **Critères d'un bon test unitaire :**
  1. La fonction contient une logique décisionnelle ou un algorithme non trivial (plusieurs branches, cas limites, calculs).
  2. Elle peut être exécutée sans simuler la base de données ou le réseau.
  3. Un échec indique sans ambiguïté un bug dans l'algorithme lui-même.
* **À éviter :** Tester des passe-plats simples, des accesseurs/mutateurs basiques, ou la structure interne des services.

### 3. Tests d'Intégration (Cœur Large - Priorité № 1)
* **But :** Vérifier que plusieurs unités (services, endpoints API, base de données, middleware) interagissent correctement ensemble du point de vue de l'utilisateur ou du contrat d'interface.
* **Pourquoi c'est le niveau le plus large ?**
  * Offre le meilleur ratio confiance / coût de maintenance.
  * Réduit le besoin de mocks artificiels : on teste avec des dépendances réelles autant que possible (ex: base de données via Testcontainers).
  * Les tests restent résilients au refactoring interne tant que le comportement observable ne change pas.
* **Dans un projet C# :** Tests d'endpoints ASP.NET Core (`WebApplicationFactory`), tests de services avec EF Core, validation du contrôle d'accès/ownership.

### 4. Tests E2E / Système (Sommet Étroit)
* **But :** Valider les parcours utilisateurs critiques de bout en bout dans un vrai navigateur.
* **Usage :** Limité aux scénarios indispensables (authentification, création de commande, flux d'onboarding, règles juridiques ou réglementaires), car ils sont plus lents et plus coûteux à maintenir.
* **Dans un projet C# :** Playwright (`dotnet test` avec Microsoft.Playwright ou suite Node `npx playwright test`).

---

## 🧭 Grille de Décision : Choisir le Bon Niveau de Test

Pour chaque fonctionnalité ou exigence, se poser ces **6 questions heuristiques** (adaptées de Michael Scotto / *The Quality Index*) :

| Question Heuristique | Si OUI ➔ Niveau Recommandé | Raisonnement & Impact |
| :--- | :--- | :--- |
| **1. La fonctionnalité implique-t-elle des exigences légales ou réglementaires ?** | **E2E / Système** | Nécessite la plus haute fidélité avec l'expérience utilisateur réelle. |
| **2. Quelle est la criticité métier (ex: risque d'incident majeur) ?** | **E2E (Parcours principal)** | Sécuriser le *Happy Path* critique de bout en bout. |
| **3. Combien de variations/combinaisons de données doivent être validées ?** | **Intégration** | Valider des dizaines de combinaisons en E2E est trop lent. Les tests d'intégration de composants ou d'API permettent de tester rapidement toutes les permutations. |
| **4. Quel est le niveau le plus bas où la fonctionnalité est observable de façon réaliste ?** | **Le niveau le plus bas suffisant** | Si la logique d'un endpoint backend peut être validée par un test API avec la DB de test, inutile d'instancier toute l'UI en E2E. |
| **5. La fonctionnalité dépend-elle spécifiquement du navigateur ?** (Cookies, historique, thèmes, observers) | **E2E (Playwright)** | Valider le comportement réel dans un véritable moteur de rendu. |
| **6. Quelles sont les compétences et le stack de l'équipe ?** | **Pragmatisme du stack** | Exploiter pleinement xUnit, NSubstitute, WebApplicationFactory, Testcontainers et Playwright sans sur-ingénierie. |

---

## 🛠️ Règles d'Écriture de Bons Tests (Best Practices)

### 1. Tester le Comportement, Pas l'Implémentation
* ❌ **Mauvais :** Vérifier l'état interne d'un service privé ou l'appel d'une fonction privée.
* ✅ **Bon :** Vérifier que lorsqu'un utilisateur remplit le formulaire et clique sur "Valider", l'élément apparaît dans le DOM ou l'API retourne HTTP 200 avec les bonnes données.

### 2. Éviter le Sur-Mocking (Mock Minimalism)
* Ne mocker que les services tiers externes hors du périmètre du projet (ex: SMTP pour les emails, Stripe pour les paiements).
* Ne PAS mocker la base de données lorsqu'une DB de test réelle (Testcontainers / SQLite) est disponible pour les tests.
* Ne PAS mocker les services internes de la slice sauf cas d'isolation extrême.

### 3. Sécurité & Contrôle d'Accès (Spécifique Projet)
* Tout test d'intégration d'API ou de base de données doit vérifier explicitement le respect de l'autorisation et de l'ownership.
* Vérifier qu'un utilisateur A ne peut en aucun cas lire, modifier ou supprimer les données de l'utilisateur B (403/404 attendu).

### 4. Sélecteurs Résilients au Refactoring
* ❌ **Éviter :** `page.locator('div > div.flex > button:nth-child(2)')`
* ✅ **Privilégier :** `screen.getByRole('button', { name: /enregistrer/i })` ou `data-testid="save-appointment-btn"`.

### 5. Interdiction Absolue des Tests en Double (Règle Anti-Duplication)
* ❌ **Interdiction des intitulés identiques :** Il est strictement interdit d'ajouter des tests (`it` ou `test`) partageant exactement le même titre au sein d'un même fichier. Chaque test doit comporter un nom unique précisant le cas spécifique ou la méthode testée (ex: `"retourne 500 et logue logger.error quand la DB throw (GET)"`).
* ❌ **Interdiction des fichiers satellites redondants :** Ne JAMAIS créer de fichiers de tests fragmentés du type `-missing-coverage`, `-missing-coverage-2`, `-robustness` ou de doublons d'intégration (ex: `appointments.int.test.ts` vs `appointments_api.int.test.ts`). Tout nouveau cas de test doit obligatoirement être intégré dans le fichier de test unitaire ou d'intégration principal du module concerné.
* ❌ **Éviter le chevauchement E2E / Intégration sur les nouveaux tests :** Ne pas recréer de nouveaux tests E2E Playwright pour des cas déjà couverts à 100 % par des tests d'intégration xUnit légers et rapides, sauf pour le *Happy Path* d'un parcours utilisateur critique.
* 📌 **Conservation des tests E2E existants :** Toujours **garder les tests E2E existants**, même s'ils sont en double avec un test d'intégration (ne jamais supprimer les tests E2E déjà écrits).
* 📌 **Conservation des tests intégration existants :** Toujours **garder les tests intégration existants**, même s'ils sont en double avec un test e2e (ne jamais supprimer les tests intégration déjà écrits).

---

## 💻 Exemples Concrets par Niveau (Stack C# / .NET)

### A. Test Unitaire (xUnit) - Logique Pure
```csharp
// tests/Features/Calendar/PricingRulesTests.cs
public class PricingRulesTests
{
    [Fact]
    public void CalculateLineTotal_DoitAppliquerLaRemise_QuantiteSuperieureAuSeuil()
    {
        var item = new OrderItem(Price: 10m, Quantity: 11); // > seuil de remise

        var total = PricingRules.CalculateLineTotal(item);

        total.ShouldBe(99m); // 10 * 11 * (1 - 0.1)
    }
}
```

### B. Test d'Intégration (xUnit + WebApplicationFactory)
```csharp
// tests/IntegrationTests/OrdersApiTests.cs
public class OrdersApiTests(WebApplicationFactory<Program> factory)
    : IClassFixture<WebApplicationFactory<Program>>
{
    [Fact]
    public async Task Post_DoitRefuserLaCreation_SiUtilisateurNonAutorise()
    {
        var client = factory.CreateClient(); // sans token d'authentification

        var response = await client.PostAsJsonAsync("/api/orders", new { });

        response.StatusCode.ShouldBe(HttpStatusCode.Unauthorized);
    }
}
```

### C. Test E2E (Playwright) - Parcours Critique
```typescript
// tests/e2e/order-flow.spec.ts
import { test, expect } from '@playwright/test';

test('Commande complète par le client', async ({ page }) => {
  await page.goto('/login');
  await page.fill('input[name="email"]', 'client@test.com');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button[type="submit"]');

  await page.click('button[data-testid="new-order-btn"]');
  await page.fill('input[name="productName"]', 'Alice Dupont');
  await page.click('button[data-testid="save-btn"]');

  await expect(page.locator('.orders-grid')).toContainText('Alice Dupont');
});
```

---

## ⚡ Commandes d'Exécution et Validation

```bash
# 1. Analyse statique (types, nullabilité, analyzers)
dotnet build
dotnet format --verify-no-changes

# 2. Exécution des tests Unitaires et d'Intégration
dotnet test --filter "Category!=Integration"
dotnet test --filter "Category=Integration"

# 3. Exécution des tests E2E
npx playwright test

# 4. Exécution de tous les tests pour validation complète
dotnet test && npx playwright test
```