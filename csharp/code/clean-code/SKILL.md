---
name: clean-code
description: Directives et standards de bonnes pratiques de développement (Clean Code, Règle du Boy Scout, suppression des using inutilisés, factorisation DRY, client API centralisé, runtime .NET / ASP.NET Core, maintenabilité) pour les projets C#.
---

# 🧼 Clean Code, Boy Scout & .NET (`clean-code`)

Cette compétence définit les exigences de qualité du code, de propreté, de factorisation et de gestion mémoire pour les applications **C# / .NET (ASP.NET Core, EF Core)**.

---

## 📋 Directives et Standards de Code

### 1. La Règle du Boy Scout (Boy Scout Rule)
> *"Toujours laisser le code plus propre que vous ne l'avez trouvé."*

- **Nettoyage des Usings** : **OBLIGATION** de vérifier et supprimer tout `using` qui n'est plus référencé dans le fichier. Évite les avertissements de l'éditeur (lignes en gris pâle) et n'alourdit pas la compilation.
- **Code Mort & Commentaires Obsolètes** : Ne pas laisser de code commenté ou de variables/fonctions inutilisées.
- **Nettoyage des Logs** : Supprimer tous les `Console.WriteLine` / `Debug.WriteLine` temporaires avant toute soumission (les vrais logs passent par `ILogger`).

---

### 2. Lisibilité et Modularité

- **Responsabilité Unique** : Une classe ou une fonction ne doit faire qu'une seule chose.
- **Respect Strict des Limites de Taille** :
  - Fichiers / classes < **400 lignes** (**OBLIGATION** : découper immédiatement toute classe complexe en sous-services ou helpers dès que cette limite est approchée).
  - Services < **500 lignes**.
  - Fonctions / Helpers < **50 lignes**.
- **Abstraction Framework** : Ne jamais contourner les mécanismes d'ASP.NET Core (middleware, DI, model binding) par des hacks réflexifs ou des singletons manuels.

---

### 3. Validation et Typage Strict

- **Nullable Reference Types activés** sans `any` ni cast déguisé — équivalent C# : pas de `dynamic`, pas de `(T)(object)` sauvage, pas d'opérateur null-forgiving (`!`) injustifié.
- `var` réservé aux cas où le type est évident côté droit.
- Validation systématique via `dotnet build` (warnings traités comme des erreurs quand c'est configurable).

---

### 4. Factorisation & Single Source of Truth (DRY)

- **Constantes Uniques (Anti-Magic Numbers)** : **INTERDICTION** de dupliquer des constantes système, tailles de lots (`ChunkSize`), quotas ou limites de caractères entre le client et le serveur (API). Toute constante partagée doit impérativement résider dans un fichier unique (ex: `Shared/Constants.cs` ou un projet `Shared`/`Contracts` dédié).
- **Détection des Doublons** : Dès qu'une logique ou un composant (champ de formulaire complexe, carte, modal, badge, endpoint de pagination, etc.) est répété ou utilisé dans au moins deux endroits différents de l'application, il **DOIT** être immédiatement extrait sous la forme d'un service, d'un helper ou d'un composant réutilisable (dans `Features/.../` ou `Shared/`).
- **Proposition Proactive** : Lors de toute modification ou création de fonctionnalité, identifier et proposer systématiquement la factorisation si une structure identique existe déjà.

---

### 5. Traitement de Données, Fichiers & Performance Mémoire

- **Parsing Robuste de Fichiers (CSV / vCard)** : Ne pas réinventer de parseurs ad-hoc basiques. Utiliser des librairies éprouvées (CsvHelper, etc.) et gérer impérativement les cas limites (sauts de ligne dans les champs entre guillemets, encodages UTF-8 BOM, etc.).
- **Sanitisation Prudente** : La désinfection contre les injections de formules CSV/Excel ne doit JAMAIS dénaturer les données légitimes d'un utilisateur (ex: préserver des noms commençant par un symbole ou valider explicitement l'intention).
- **Gestion Mémoire & Traitement par Lots** : Ne pas instancier de tableaux géants vides (`new T[n]`) pour découper des flux ou paquets. Préférer `IAsyncEnumerable<T>`, le streaming (`Stream`, `JsonSerializer.DeserializeAsyncEnumerable`) et le traitement itératif pour limiter l'allocation et la pression sur le Garbage Collector.
- **Asynchronisme** : Ne jamais bloquer le thread (`Thread.Sleep`, `.Result`, `.Wait()`). Utiliser `await Task.Delay(...)` et le motif async de bout en bout.

---

### 6. Intercepteurs et Communication API Centralisée (`HttpClient` typé)

- **Méthode Privilégiée** : L'utilisation d'un **client API centralisé typé via `IHttpClientFactory`** (clients nommés ou typés) est la méthode à privilégier obligatoirement pour toutes les communications HTTP sortantes de l'application.
- **Gestion des Erreurs Unifiée** : Ne jamais écrire de requêtes `HttpClient` brutes éparpillées sans passer par la couche d'API ou le client typé. La couche centralisée garantit l'encapsulation systématique des statuts HTTP (`EnsureSuccessStatusCode` + handler dédié), l'interception automatique des erreurs 5xx, la résilience (Polly : retry, circuit breaker, timeout) et la propagation du `CancellationToken`.

---

### 7. Bonnes Pratiques Runtime .NET & ASP.NET Core

- **Zero État Global Mutable** : Ne jamais stocker d'état relatif à une requête dans des champs statiques ou des singletons (risque de fuite de données inter-requêtes). Les données par requête vivent dans les services `Scoped`.
- **Gestion des Tasks & Processus Arrière-Plan** : Toute Task doit être `await`, retournée ou transmise via un mécanisme d'arrière-plan dédié (`IHostedService` / `BackgroundService`) pour éviter les *floating promises* abandonnées ou silencieusement avalées.
- **Durées de Vie DI** : Respecter scrupuleusement Singleton/Scoped/Transient ; jamais de dépendance `Scoped` capturée dans un `Singleton`.
- **Requêtes EF Core Paramétrées & Ownership** : TOUTES les lectures/écritures de données utilisateur doivent vérifier l'ownership côté serveur (jamais d'ID de ressource accepté aveuglément du client).
- **Limites de Mémoire** : Ne jamais charger en mémoire des payloads non bornés dans les handlers de serveur (pas de `ReadToEndAsync` aveugle). Utiliser le streaming et respecter `MaxBulkBatchSize` / limites configurées.
- **Sécurité Cryptographique & Comparaison Temporelle** : Utiliser `RandomNumberGenerator` / `CryptographicOperations.FixedTimeEquals` (jamais `Random` pour la génération de clés/jetons) et `FixedTimeEquals` pour vérifier les jetons sensibles sans attaque par canal auxiliaire.
- **Configuration & Secrets** : Utiliser le pattern Options (`IOptions<T>`) et la configuration typée ; secrets dans User Secrets (dev) et variables d'environnement / coffre (prod) — jamais dans `appsettings.json` versionné.

---

## 📋 Checklist de Validation Clean Code

- [ ] Les using inutilisés ont été retirés du fichier.
- [ ] Aucune constante magique n'est dupliquée entre client et serveur (`Shared/Constants.cs`).
- [ ] Le client API centralisé (`IHttpClientFactory` / client typé) est utilisé pour les requêtes HTTP.
- [ ] Aucune donnée utilisateur n'est lue/écrite sans vérification d'ownership côté serveur.
- [ ] La classe ou la fonction respecte les limites de taille (< 400L fichier, < 50L fonction).
