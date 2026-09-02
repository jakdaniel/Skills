---
name: csharp-standards
description: Normes complètes de développement C# / .NET (ASP.NET Core, EF Core) — conventions de nommage, nullable reference types, async/await, records, pattern matching, LINQ, injection de dépendances, architecture applicative, configuration, endpoints Minimal API, performance et logging. À utiliser pour tout développement, refactoring, revue de code ou décision d'architecture C#.
---

# ⚡ C# / .NET Standards (`csharp-standards`)

Ce skill fournit un cadre clair et moderne pour construire des applications **C# / .NET (ASP.NET Core, EF Core)** de haute qualité : conventions du langage, architecture applicative et bonnes pratiques de plateforme.

---

## 🎯 Quand utiliser ce skill

- Création ou refactorisation de classes, services ou endpoints C#
- Revue de code de la logique métier, de l'asynchronisme .NET ou de l'architecture
- Optimisation des performances (allocation, LINQ, async, EF Core)
- Structuration d'un projet .NET (DI, configuration, organisation)
- Diagnostic de la nullabilité, du pattern matching et des anti-patterns

---

## ⚡ Doctrine C# Moderne

### Règles Fondamentales

- **Nullable Reference Types activés** (`<Nullable>enable</Nullable>`) : la nullabilité est la norme obligatoire.
- **`async`/`await`** pour toute opération I/O ; interdiction de `.Result`, `.Wait()` ou `.GetAwaiter().GetResult()` (risque de deadlock et de famine de thread pool).
- **Records** pour les DTO, messages et valeurs immuables ; classes pour les entités avec identité et comportement.
- **Immutabilité par défaut** : `readonly`, `init`, collections immuables quand c'est pertinent.
- L'état statique mutable est interdit (sauf caches thread-safe explicitement conçus).
- **DI native via constructeur** — jamais de `new` sur des services avec dépendances (ce n'est pas testable).

---

## 🚦 Hiérarchie des Règles

### 🔴 OBLIGATOIRE
- Aucun `dynamic`, aucun `object` déguisé ; génériques contraints ou interfaces.
- Aucun `catch (Exception) { }` vide — toute exception est interceptée, loguée ou relancée avec contexte.
- Toute méthode publique a un type de retour explicite (pas de `var` sur les retours publics).
- `CancellationToken` propagé dans toute la chaîne async des endpoints.
- Aucun I/O bloquant dans une méthode `async` ("async all the way").
- Validation serveur systématique des entrées.
- Aucune capture de `DbContext` dans un singleton.
- Aucun état statique mutable.

### 🟠 FORTEMENT RECOMMANDÉ
- `sealed` par défaut sur les classes non destinées à l'héritage.
- Pattern matching (`switch` expressions, `is` patterns) plutôt que des cascades de `if/else if` sur des types.
- `TimeSpan`, `DateOnly`, `TimeOnly` et `DateTimeOffset` (jamais `DateTime` naïf pour des instants) pour les domaines temporels.
- `ILogger<T>` injecté plutôt que `Console.WriteLine` / `Debug.WriteLine`.
- Minimal APIs pour les endpoints simples, contrôleurs pour les grands ensembles cohérents.
- `IHttpClientFactory` / clients typés pour tout appel HTTP sortant.
- Projections EF Core (`Select`) plutôt que de charger des entités complètes.
- Options centralisées via `Directory.Build.props`.

### 🟢 OPTIONNEL
- `Span<T>` / `Memory<T>` pour les chemins chauds de parsing.
- `System.Text.Json` source generators pour la sérialisation AOT/performance.
- `FrozenDictionary` / `FrozenSet` pour les lookups en lecture intensive.
- Native AOT pour les microservices critiques en démarrage.

---

## 🧩 1. Conventions de Nommage (Microsoft .NET Guidelines)

| Élément | Convention | Exemple |
|---|---|---|
| Classe / Record / Struct | PascalCase | `OrderService` |
| Interface | `I` + PascalCase | `IOrderRepository` |
| Méthode | PascalCase, verbe | `CalculateTotal` |
| Propriété publique | PascalCase | `OrderStatus` |
| Champ privé | `_camelCase` | `_orderRepository` |
| Constante | PascalCase | `MaxRetryCount` |
| Paramètre / variable locale | camelCase | `orderId` |
| Type générique | `T` + suffixe | `TEntity` |

- Aucune abréviation ambiguë (`btn`, `usr`, `tmp`, `val` sont interdits). Écrire `button`, `user`, `temporaryOrder`, `validatedInput`.
- Les booléens sont préfixés (`IsLoading`, `HasError`, `CanSubmit`).
- Les méthodes async se terminent par `Async` (`GetOrderAsync`).

---

## 🧱 2. Conception des Classes & Structure des Fichiers

### Principes
- **Un fichier = une responsabilité** (une classe par fichier de préférence).
- **Limite Stricte de Lignes** : fichiers / classes < **400 lignes** (**OBLIGATION** : découper immédiatement toute classe complexe en sous-services ou helpers dès que cette limite est approchée). Services < **500 lignes**. Fonctions / Helpers < **50 lignes**.
- **SOLID & séparation** : Séparer la présentation (contrôleurs/endpoints, DTO) de la logique métier (services, domaine) et de l'accès aux données (repositories / DbContext). La logique métier vit dans les services ou classes de domaine — jamais dans les contrôleurs/endpoints.

```csharp
public sealed class OrderService(IOrderRepository orders, ILogger<OrderService> logger)
{
    public async Task<Order?> GetOrderAsync(Guid id, CancellationToken ct)
        => await orders.FindAsync(id, ct);
}
```

### Styles
- `sealed` par défaut, expression-bodied members pour les one-liners, primary constructors (C# 12) pour l'injection simple.

---

## 🔄 3. Async / Await

```csharp
public async Task<Result<Order>> GetOrderAsync(Guid orderId, CancellationToken cancellationToken)
{
    var order = await _orderRepository.FindAsync(orderId, cancellationToken);
    return order is null
        ? Result<Order>.NotFound()
        : Result<Order>.Success(order);
}
```

### Bonnes Pratiques
- Retourner `Task` / `Task<T>` — jamais `async void` (sauf gestionnaires d'événements UI).
- `await` de bout en bout ; `ConfigureAwait(false)` uniquement dans les bibliothèques (pas dans l'application ASP.NET Core).
- `Task.WhenAll` pour les opérations indépendantes (jamais sur un même `DbContext` — EF Core n'est pas thread-safe).
- `ValueTask<T>` uniquement pour les chemins chauds avec hit synchrone fréquent.
- Toujours attendre la `Task` : aucune *floating promise* (analyser les warnings CS4014).

---

## 🧊 4. Nullabilité & Guard Clauses

```csharp
public Order? FindOrder(Guid id) { ... }

// Validation explicite des préconditions
ArgumentNullException.ThrowIfNull(customer);
ArgumentException.ThrowIfNullOrWhiteSpace(customer.Name);
```

### Bonnes Pratiques
- `null` est un état explicite : gérer les retours nullables (`?.`, `??`, pattern `is null`).
- Interdiction de l'opérateur null-forgiving (`!`) sauf justification documentée.
- Ne jamais retourner de collection null : retourner `[]` / `Enumerable.Empty<T>()`.

---

## 🚀 5. LINQ & Collections

```csharp
// ✅ CORRECT — requête filtrée côté SQL
var recentOrders = await _db.Orders
    .AsNoTracking()
    .Where(o => o.CreatedAt >= cutoff)
    .OrderByDescending(o => o.CreatedAt)
    .Select(o => new OrderSummary(o.Id, o.Total))
    .ToListAsync(cancellationToken);
```

### Bonnes Pratiques
- LINQ est un calcul pur : aucun effet de bord dans une chaîne LINQ.
- Attention à l'**exécution différée** : matérialiser (`ToList()`) avant d'itérer plusieurs fois ou de retourner une requête EF Core.
- `AsNoTracking()` pour toute lecture EF Core sans modification.
- Interdiction de `ToList()` suivi de `.Count() == 0` : utiliser `Any()`.

---

## 🛠️ 6. Injection de Dépendances (ASP.NET Core)

- Injecter via constructeur ; enregistrer dans `Program.cs` (ou par modules d'extension `AddXxx`).
- **Programmation contre abstractions** : Les services publics exposent des interfaces.
- Durées de vie explicites : `Singleton` (stateless/thread-safe uniquement), `Scoped` (par requête), `Transient`.
- **Interdiction** : capturer un service `Scoped` dans un `Singleton` (captive dependency) — utiliser `IServiceScopeFactory`.
- Interdiction du Service Locator (`IServiceProvider.GetService` explicite en plein code).
- `IHttpClientFactory` / clients nommés ou typés pour tout appel HTTP sortant.

---

## 🔒 7. Contrôle d'Accès & Sécurité (CRITIQUE)

- **Autorisation** : Chaque endpoint/contrôleur déclare explicitement son exigence d'autorisation (`[Authorize]`, policies).
- **Validation** : Toujours valider que l'utilisateur authentifié a le droit d'accéder à la ressource demandée (ownership côté serveur) — jamais d'ID de ressource accepté aveuglément du client.
- **Sécurité** : Ne jamais exposer de données d'un utilisateur à un autre ; zéro confiance envers le client.

---

## 🌐 8. Endpoints (Minimal APIs)

```csharp
app.MapGet("/orders/{id:guid}", async (Guid id, IOrderService orders, CancellationToken ct) =>
    await orders.GetOrderAsync(id, ct) is { } order
        ? Results.Ok(OrderDto.From(order))
        : Results.NotFound());
```

### Bonnes Pratiques
- Route groups (`MapGroup`) pour préfixer et factoriser les filtres.
- Filtres d'endpoints pour la validation/autorisation transverse.
- Typage des paramètres de route (`{id:guid}`).
- Jamais de logique métier dans l'endpoint — déléguer au service.
- Problème Details RFC 7807 (`Results.Problem`) pour les réponses d'erreur cohérentes.

---

## ⚙️ 9. Configuration & Secrets

```csharp
public sealed class SmtpOptions
{
    public const string SectionName = "Smtp";

    [Required] public required string Host { get; init; }
    [Range(1, 65535)] public int Port { get; init; } = 587;
}

// Program.cs
builder.Services.AddOptions<SmtpOptions>()
    .BindConfiguration(SmtpOptions.SectionName)
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

- Pattern Options typé (`IOptions<T>` / `IOptionsMonitor<T>`) + `ValidateOnStart` (fail fast si un secret manque).
- User Secrets en développement (`dotnet user-secrets`) ; variables d'environnement / coffre en production.
- Jamais de secret dans `appsettings.json` versionné, dans le code source, les logs ou les messages d'exception.

---

## ⚡ 10. Performance

### EF Core
- `AsNoTracking()` pour les lectures ; projections `Select` pour les listes.
- Pagination + `OrderBy` déterministe obligatoires.
- Chunked / streaming pour les gros volumes.

### Allocations
- `Span<T>` / `string.Create` pour les chemins chauds de parsing.
- `ArrayPool<T>` / `MemoryPool<T>` pour les buffers réutilisables.
- Éviter les closures allouées en boucle.

### HTTP & Cache
- Polly : timeout, retry avec backoff, circuit breaker.
- Données statiques applicatives : cache (`IMemoryCache`, `IDistributedCache`) ou pattern Options — éviter les requêtes DB répétées.

---

## 📊 11. Logging & Observabilité

```csharp
_logger.LogInformation("Order {OrderId} created for {CustomerId}", order.Id, order.CustomerId);
```

- `ILogger<T>` injecté, messages structurés avec placeholders (jamais d'interpolation dans le template).
- Aucune donnée sensible dans les logs (tokens, mots de passe, PII).
- Corrélation par requête (TraceId) activée.
- Gestion d'erreurs centralisée (`IExceptionHandler` / middleware) — jamais de try/catch par endpoint masquant l'erreur.

---

## 🏗️ 12. Composition & Organisation du Projet

```csharp
// Program.cs — composition root (orchestration uniquement)
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOrderManagement();   // module de slice
builder.Services.AddDatabase();
builder.Services.AddApplicationSecurity();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();
app.MapOrderEndpoints();

await app.RunAsync();
```

```
src/
├── Api/                  # Composition, endpoints, middleware
│   ├── Program.cs
│   └── Features/
├── Shared/               # Kernel (utils, types génériques)
└── Tests/
    ├── UnitTests/
    └── IntegrationTests/
```

- Configuration par environnement (`appsettings.{Env}.json` + variables).
- Health checks (`/health`, `/ready`) pour l'orchestration.
- `BackgroundService` / `IHostedService` pour les tâches de fond — jamais de `Task.Run` fire-and-forget dans un handler.
- `Directory.Build.props` : `<Nullable>enable</Nullable>`, `<TreatWarningsAsErrors>true</TreatWarningsAsErrors>`, `<ImplicitUsings>enable</ImplicitUsings>`.

---

## 🚫 Anti-Patterns C# / .NET (CRITIQUE)

- ❌ `.Result`, `.Wait()`, `.GetAwaiter().GetResult()` sur une Task (deadlock).
- ❌ `async void`.
- ❌ `dynamic` ou cast `object` non contrôlé.
- ❌ `catch (Exception) { }` vide ou `catch` qui avale silencieusement.
- ❌ État statique mutable partagé entre requêtes.
- ❌ Requête LINQ→SQL ré-énumérée ou `IQueryable` d'EF Core retourné hors du repository.
- ❌ `new DbContext()` ou `new HttpClient()` artisanal.
- ❌ Logique métier dans un contrôleur/endpoint.
- ❌ `DateTime.Now` pour des instants à stocker : utiliser `DateTimeOffset.UtcNow` / `TimeProvider`.
- ❌ `double` pour des montants monétaires (`decimal` obligatoire).
- ❌ Usage de `placeholder` sur les champs de saisie UI.
- ❌ Service singleton capturant un état de requête (captive dependency).

---

## 📋 Checklist de Validation C#

- [ ] Nullable Reference Types actifs, zéro warning CS86xx non traité ; `dotnet build` sans erreur ni nouveau warning.
- [ ] Toute opération I/O est asynchrone, avec `CancellationToken` propagé ; aucun `.Result` / `.Wait()` / `async void` / `dynamic`.
- [ ] Les records sont utilisés pour les DTO et valeurs immuables.
- [ ] Les endpoints vérifient auth + autorisation côté serveur, sans logique métier.
- [ ] La classe ou la fonction respecte les limites de taille (< 400L fichier, < 50L fonction).
- [ ] Configuration typée (Options) et secrets hors du dépôt.
- [ ] Gestion d'erreurs centralisée (Problem Details) et logs structurés sans données sensibles.
