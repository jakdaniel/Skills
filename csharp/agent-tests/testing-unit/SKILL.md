---
name: testing-unit
description: Expert en tests unitaires exhaustifs avec xUnit. Utilise ce skill dès qu'il faut écrire, améliorer ou auditer des tests unitaires pour des fonctions pures, services, validateurs, ou toute logique métier. Le skill insiste sur une couverture maximale : chaque branche logique, chaque cas limite, chaque chemin d'erreur doit être testé. Je ne veux pas juste que les situations critiques ou essentielles soient couvertes, je veux que TOUTES les situations possibles soient couvertes sans exception.
---

## Philosophie Fondamentale

> **"Un test qui ne peut pas échouer ne prouve rien."**

L'objectif n'est pas d'écrire *des* tests, mais d'écrire *tous* les tests nécessaires pour qu'un bug ne puisse pas passer inaperçu. **Chaque situation possible, aussi rare ou triviale soit-elle, doit avoir son propre cas de test.** Chaque fonction testée doit être traitée comme une boîte noire dont on essaie activement de **prouver qu'elle peut échouer**.

### Mentalité de l'Expert QA

Avant d'écrire le moindre test, poser systématiquement ces questions :

1. **Quels sont tous les chemins possibles ?** (branches `if/else`, switch expressions, `??`, `?.`, pattern matching)
2. **Quelles entrées invalides peut-on recevoir ?** (`null`, `""`, `0`, `NaN`/`double.NaN`, `decimal` aux limites, collections vides, objets à propriétés null)
3. **Quelles sont les valeurs limites ?** (min, max, min-1, max+1, exactement à la frontière)
4. **Comment ça peut planter ?** (exceptions attendues, erreurs réseau, `TaskCanceledException`, exceptions de dépendances)
5. **L'ordre des opérations est-il important ?** (effets de bord, appels multiples, état mutable)
6. **Que retourne la fonction quand tout va bien ?** (type, forme, valeurs exactes)

---

## Configuration xUnit

```csharp
// TestProject.csproj
// <PackageReference Include="xunit" Version="..." />
// <PackageReference Include="xunit.runner.visualstudio" Version="..." />
// <PackageReference Include="NSubstitute" Version="..." />
// <PackageReference Include="Shouldly" Version="..." />
// <PackageReference Include="coverlet.collector" Version="..." />

// [Trait("Category", "Unit")] ou [Collection] pour l'organisation
```

---

## Structure des Tests : Le Protocole d'Exhaustivité

Pour **chaque** fonction ou service, suivre ce protocole sans exception :

### Étape 1 — Analyser la signature

```csharp
// Exemple de fonction à analyser :
public static decimal CalculateDiscount(decimal price, string? coupon = null)
```

Extraire mentalement :
- Paramètres requis : `price`
- Paramètres optionnels : `coupon`
- Type de retour : `decimal`
- Effets de bord possibles : aucun (pure) ou à identifier

### Étape 2 — Cartographier les branches

```csharp
// Exemple avec branches complexes :
public static string GetLabel(string? status, int count)
{
    if (string.IsNullOrEmpty(status)) return "unknown";   // Branche 1
    if (count == 0) return "empty";                        // Branche 2
    if (count == 1) return status;                         // Branche 3
    return $"{status} ({count})";                          // Branche 4 (default)
}
```

→ **4 branches = minimum 4 tests**, souvent plus pour les valeurs limites.

### Étape 3 — Lister les cas de test AVANT d'écrire le code

```
✅ Happy paths       : cas nominaux où tout fonctionne
✅ Edge cases        : valeurs aux frontières (0, 1, max, vide)
✅ Sad paths         : entrées invalides, mauvais types
✅ Error paths       : exceptions, cancellations, timeouts
✅ Side effects      : appels des mocks/substituts, mutations d'état
✅ Async behavior    : résolution, rejet, annulation
✅ Re-entrancy       : appeler la fonction plusieurs fois
```

---

## Template de Test Exhaustif

```csharp
public class ModuleNameTests
{
    public class FunctionToTest
    {
        // ─── HAPPY PATH ─────────────────────────────────────────────────────
        public class CasNominaux
        {
            [Fact]
            public void Should_ReturnExpectedResult_WithValidStandardInput()
            {
                var result = ModuleName.FunctionToTest("valid");
                result.ShouldBe("expected");
            }

            [Theory]
            [InlineData("a", "A")]
            [InlineData("hello world", "Hello World")]
            public void Should_HandleAllValidInputVariations(string input, string expected)
            {
                ModuleName.FunctionToTest(input).ShouldBe(expected);
            }
        }

        // ─── VALEURS LIMITES ─────────────────────────────────────────────────
        public class ValeursLimites
        {
            [Fact] public void Should_HandleMinimumValue() { /* ... */ }
            [Fact] public void Should_HandleMaximumValue() { /* ... */ }
            [Fact] public void Should_HandleValueJustBelowMinimum() { /* ... */ }
            [Fact] public void Should_HandleValueJustAboveMaximum() { /* ... */ }
            [Fact] public void Should_HandleZero() { /* ... */ }
            [Fact] public void Should_HandleSingleElement() { /* ... */ }
        }

        // ─── ENTRÉES INVALIDES ───────────────────────────────────────────────
        public class EntreesInvalides
        {
            [Fact] public void Should_Throw_WhenInputIsNull() { /* ... */ }
            [Fact] public void Should_HandleEmptyString() { /* ... */ }
            [Fact] public void Should_HandleEmptyCollection() { /* ... */ }
            [Fact] public void Should_HandleNaN() { /* ... */ }
            [Fact] public void Should_HandleNegativeValue_WhenOnlyPositiveExpected() { /* ... */ }
        }

        // ─── CHEMINS D'ERREUR ────────────────────────────────────────────────
        public class GestionDesErreurs
        {
            [Fact]
            public void Should_ThrowSpecificError_WhenDependencyIsMissing()
            {
                Should.Throw<InvalidOperationException>(
                    () => ModuleName.FunctionToTest(null!))
                    .Message.ShouldContain("Expected error message");
            }

            [Fact]
            public void Should_ThrowCorrectErrorType()
            {
                Should.Throw<ArgumentNullException>(
                    () => ModuleName.FunctionToTest(null!));
            }
        }

        // ─── EFFETS DE BORD ──────────────────────────────────────────────────
        public class EffetsDeBordEtInteractions
        {
            [Fact]
            public async Task Should_CallDependency_ExactlyOnce()
            {
                var dependency = Substitute.For<IRepository>();
                dependency.FindAsync(1).Returns(new Entity());
                await ModuleName.MyFunctionAsync(dependency, 1);
                await dependency.Received(1).FindAsync(1);
            }

            [Fact]
            public void Should_NOT_CallDependency_WhenConditionIsFalse() { /* ... */ }
        }
    }
}
```

---

## Patterns d'Exhaustivité par Type

### 🔢 Fonctions Numériques / Monétaires

```csharp
public class NumericFunctionTests
{
    // Valeurs normales
    [Theory] [InlineData(1)] [InlineData(42)] [InlineData(int.MaxValue)]
    public void Should_WorkWithNormalValues(int value) { /* ... */ }

    // Cas limites critiques
    [Fact] public void Should_HandleZero() { /* ... */ }
    [Fact] public void Should_HandleNegativeNumber() { /* ... */ }
    [Fact] public void Should_HandleMinValue() { /* ... */ }
    [Fact] public void Should_HandleMaxValue() { /* ... */ }
    [Fact] public void Should_HandleFloatingPointPrecision() { /* ... */ }

    // Monnaie : TOUJOURS decimal, vérifier les arrondis (MidpointRounding)
    [Fact]
    public void Should_RoundMoneyCorrectly()
    {
        var result = PricingRules.ApplyTax(10.005m);
        result.ShouldBe(10.01m); // préciser le mode d'arrondi attendu
    }
}
```

### 📝 Fonctions sur Chaînes

```csharp
public class StringFunctionTests
{
    [Theory]
    [InlineData("")]                       // chaîne vide
    [InlineData(" ")]                      // espaces uniquement
    [InlineData("!@#$%")]                  // caractères spéciaux
    [InlineData("émojis 🎉 accents éàç")]  // unicode
    [InlineData("ligne1\nligne2")]         // sauts de ligne
    public void Should_HandleSpecialStrings(string input) { /* ... */ }

    [Fact] public void Should_HandleNull() { /* ... */ }

    [Fact]
    public void Should_HandleVeryLongString()
    {
        var longString = new string('a', 10000);
        Should.NotThrow(() => fn(longString));
    }
}
```

### 📦 Fonctions sur Collections

```csharp
public class CollectionFunctionTests
{
    [Fact] public void Should_HandleEmptyCollection() { /* ... */ }
    [Fact] public void Should_HandleSingleElement() { /* ... */ }
    [Fact] public void Should_HandleTwoElements_Boundary() { /* ... */ }
    [Fact] public void Should_HandleDuplicateValues() { /* ... */ }
    [Fact] public void Should_HandleNullElements() { /* ... */ }

    [Fact]
    public void Should_NOT_MutateTheOriginalCollection()
    {
        var original = new List<int> { 1, 2, 3 };
        var copy = new List<int>(original);
        fn(original);
        original.ShouldBe(copy); // Vérifier l'immutabilité
    }

    [Fact]
    public void Should_HandleVeryLargeCollection()
    {
        var large = Enumerable.Range(0, 10000).ToList();
        Should.NotThrow(() => fn(large));
    }
}
```

### 🗂️ Fonctions sur Objets / Records

```csharp
public class ObjectFunctionTests
{
    [Fact] public void Should_WorkWithCompleteValidObject() { /* ... */ }
    [Fact] public void Should_HandleObjectWithMissingOptionalFields() { /* ... */ }
    [Fact] public void Should_HandleObjectWithNullValues() { /* ... */ }
    [Fact] public void Should_HandleDeeplyNestedObject() { /* ... */ }

    [Fact]
    public void Should_NOT_MutateTheInputObject()
    {
        var input = new Order { Quantity = 1 };
        fn(input);
        input.Quantity.ShouldBe(1); // Vérifier l'immutabilité
    }

    [Fact] public void Should_HandleUnexpectedPropertyValues() { /* ... */ }
}
```

### ⚡ Fonctions Asynchrones

```csharp
public class AsyncFunctionTests
{
    // ─── Résolution ────────────────────────────────────────────────────
    [Fact]
    public async Task Should_ResolveWithCorrectData_OnSuccess()
    {
        var result = await fn("valid-input");
        result.ShouldBe(new Result { Id = 1, Data = "expected" });
    }

    // ─── Rejet ─────────────────────────────────────────────────────────
    [Fact]
    public async Task Should_Throw_WhenInputIsInvalid()
    {
        await Should.ThrowAsync<ValidationException>(() => fn(null!));
    }

    // ─── Annulation ────────────────────────────────────────────────────
    [Fact]
    public async Task Should_RespectCancellationToken()
    {
        using var cts = new CancellationTokenSource();
        cts.Cancel();
        await Should.ThrowAsync<OperationCanceledException>(
            () => fn(cts.Token));
    }

    // ─── Timeouts et race conditions ───────────────────────────────────
    [Fact]
    public async Task Should_HandleSlowDependency_TimeoutSimulation()
    {
        var slow = Substitute.For<IRepository>();
        slow.FindAsync(Arg.Any<int>())
            .Returns(async _ => { await Task.Delay(5000); return null; });
        await Should.ThrowAsync<TimeoutException>(() => fn(slow));
    }
}
```

---

## Mocking Avancé — Isoler Chaque Dépendance

### Substitut d'Interface (NSubstitute)

```csharp
public class WithMockedApiTests
{
    private readonly IUserRepository _users = Substitute.For<IUserRepository>();

    [Fact]
    public async Task Should_CallRepository_WithCorrectId()
    {
        _users.FindAsync(1).Returns(new User { Id = 1, Name = "John" });
        await _sut.GetUserAsync(1);
        await _users.Received(1).FindAsync(1);
    }

    [Fact]
    public async Task Should_HandleRepositoryFailure_Gracefully()
    {
        _users.FindAsync(Arg.Any<int>())
            .Returns(Task.FromException<User>(new HttpRequestException("Network error")));
        await Should.ThrowAsync<HttpRequestException>(() => _sut.GetUserAsync(1));
    }

    [Fact]
    public async Task Should_NOT_CallRepository_WhenCacheIsWarm()
    {
        _cache.Preload(1, new User { Id = 1 });
        await _sut.GetUserAsync(1);
        await _users.DidNotReceive().FindAsync(Arg.Any<int>());
    }
}
```

### Fournisseur d'Heure (Testabilité des Dates)

```csharp
// Le code de production utilise TimeProvider (injectable) — jamais DateTime.Now directement.
public class DateDependentFunctionTests
{
    [Fact]
    public void Should_UseCurrentDate_ForCalculation()
    {
        var fakeTime = new FakeTimeProvider(
            new DateTimeOffset(2024, 6, 15, 12, 0, 0, TimeSpan.Zero));
        var sut = new AgeService(fakeTime);
        sut.GetAge(new DateOnly(1990, 6, 15)).ShouldBe(34); // Âge exact au 15 juin 2024
    }

    [Fact]
    public void Should_ConsiderDayBoundary_Correctly()
    {
        var fakeTime = new FakeTimeProvider(
            new DateTimeOffset(2024, 6, 14, 23, 59, 59, TimeSpan.Zero));
        var sut = new AgeService(fakeTime);
        sut.GetAge(new DateOnly(1990, 6, 15)).ShouldBe(33);
    }
}
```

### Spy sur Méthodes

```csharp
[Fact]
public void Should_LogWarning_WhenValueIsDeprecated()
{
    var logger = Substitute.For<ILogger<MyService>>();
    var sut = new MyService(logger);
    sut.DeprecatedFunction();
    logger.Received().Log(
        LogLevel.Warning,
        Arg.Any<EventId>(),
        Arg.Is<It.IsAnyType>((v, t) => v.ToString()!.Contains("deprecated")),
        null,
        Arg.Any<Func<It.IsAnyType, Exception?, string>>());
}
```

---

## Tests de Services — Protocole Complet

```csharp
public class OrderServiceTests
{
    private readonly IOrderRepository _orders = Substitute.For<IOrderRepository>();
    private readonly OrderService _sut;

    public OrderServiceTests() => _sut = new OrderService(_orders);

    // ─── État initial ──────────────────────────────────────────────────
    public class EtatInitial
    {
        [Fact]
        public void Should_StartWithEmptyState() { /* ... */ }
    }

    // ─── Actions ───────────────────────────────────────────────────────
    public class Actions
    {
        [Fact]
        public async Task Should_CreateOrder_WithValidInput() { /* ... */ }

        [Fact]
        public async Task Should_NotExceedMaxQuantity_IfDefined() { /* ... */ }

        [Fact]
        public async Task Should_Throw_OnWrongInput() { /* ... */ }
    }

    // ─── Valeurs dérivées ──────────────────────────────────────────────
    public class ValeursDerivees
    {
        [Fact]
        public void Total_ShouldBeZero_WhenNoItems() { /* ... */ }

        [Fact]
        public void Total_ShouldSumAllItems() { /* ... */ }
    }

    // ─── Isolation entre instances ─────────────────────────────────────
    public class Isolation
    {
        [Fact]
        public void Should_NotShareState_BetweenTwoInstances() { /* ... */ }
    }
}
```

---

## Checklist d'Exhaustivité — À Valider Pour Chaque Fonction

Avant de considérer une fonction comme "bien testée", cocher :

```
COUVERTURE DES BRANCHES
[ ] Chaque if/else a un test pour true ET pour false
[ ] Chaque switch expression couvre chaque branche (inclure UnreachableException si pertinent)
[ ] Chaque ?? et ?. est testé avec toutes les combinaisons pertinentes
[ ] Chaque return anticipé (early return / guard clause) a son propre test

ENTRÉES INVALIDES
[ ] null testé (et ArgumentNullException attendu si applicable)
[ ] Chaîne vide "" testée (si string attendu)
[ ] Collection vide [] testée (si collection attendue)
[ ] 0 testé (si number attendu)
[ ] double.NaN / decimal.MinValue testés (si nombre attendu)
[ ] Valeur hors plage testée (ArgumentOutOfRangeException attendue)

VALEURS LIMITES
[ ] Valeur minimale autorisée testée
[ ] Valeur maximale autorisée testée
[ ] Valeur min - 1 testée (out of bounds)
[ ] Valeur max + 1 testée (out of bounds)
[ ] Collection d'un seul élément testée
[ ] Très grande valeur testée (performance)

EFFETS DE BORD ET SUBSTITUTS
[ ] Chaque dépendance substituée est vérifiée (Received / DidNotReceive)
[ ] Cas où la dépendance échoue est testé
[ ] Cas où la dépendance n'est PAS appelée est vérifié
[ ] Immutabilité des inputs vérifiée (si applicable)
[ ] Appels multiples de la fonction testés

ASYNC (si applicable)
[ ] Cas de résolution testé
[ ] Cas d'exception testé
[ ] CancellationToken respecté (OperationCanceledException)
[ ] Comportement en cas de timeout testé
```

---

## Commandes

```bash
# Tous les tests
dotnet test

# Mode watch (développement)
dotnet watch test

# Couverture complète (coverlet)
dotnet test --collect:"XPlat Code Coverage"

# Un fichier/classe spécifique
dotnet test --filter "FullyQualifiedName~OrderServiceTests"

# Un test par nom
dotnet test --filter "FullyQualifiedName~Should_HandleEmptyString"

# Seulement les tests unitaires (exclure l'intégration)
dotnet test --filter "Category!=Integration"
```

---

## Objectifs de Couverture (Non Négociables)

| Métrique    | Minimum | Cible Idéale |
|-------------|---------|--------------|
| Statements  | 90%     | 95%+         |
| Branches    | 85%     | 90%+         |
| Functions   | 90%     | 100%         |
| Lines       | 90%     | 95%+         |

> ⚠️ Un score de couverture élevé ne garantit pas des tests de qualité — mais un score bas garantit des zones non testées. Les deux sont nécessaires.

---

**Note** : Pour les tests d'intégration (interactions entre services, appels API réels), voir `testing-integration`. Pour les parcours utilisateur complets, voir `testing-e2e`.
