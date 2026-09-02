---
name: security
description: Skill expert à utiliser lors du développement C# / .NET (ASP.NET Core) afin de prévenir les vulnérabilités de sécurité côté client et serveur. À déclencher pour audit de sécurité, owasp, revue de code, conception d'authentification, protection des données, gestion des données sensibles et déploiement sécurisé.
---

# Securite Skill

Ce skill définit les **meilleures pratiques de sécurité modernes** pour les applications **C# / .NET (ASP.NET Core)**, couvrant :
- Sécurité client et serveur
- Authentification et autorisation
- Protection contre XSS, CSRF, injections, fuites de données
- Bonnes pratiques cloud / déploiement

Objectif : **réduire la surface d'attaque** sans sacrifier la performance ou la DX.

---

## Quand utiliser ce skill
- Utiliser ce skill lorsque l'utilisateur demande une nouvelle fonctionnalité
- Audit de sécurité ASP.NET Core / .NET
- Revue de code avant mise en production
- Implémentation d'authentification / autorisation
- Manipulation de données sensibles
- Déploiement cloud / serveur
- Intégration API externes ou IA

---

## 📚 Consultation Ciblée des Fichiers de Référence (`data/`)

À **chaque fois** que ce skill est activé ou utilisé, l'agent doit effectuer une **lecture ciblée et économique** (pour préserver le budget de tokens) :
1. **Consulter l'index ci-dessous** pour identifier la catégorie liée au besoin.
2. **Lire uniquement** les **1 à 3 fichiers Markdown** pertinents situés dans `./security/data/cheatsheetseries.owasp.org/`, **SANS JAMAIS tout charger**.
3. **Appliquer les préconisations** pour la revue, l'audit ou l'implémentation.

### 🗂️ Index Thématique des Cheatsheets (`data/cheatsheetseries.owasp.org/`)

- **🔑 Authentification, Mots de passe & Sessions** :
  - `Authentication_Cheat_Sheet.md`
  - `Session_Management_Cheat_Sheet.md`
  - `Multifactor_Authentication_Cheat_Sheet.md`
  - `Forgot_Password_Cheat_Sheet.md`
  - `Credential_Stuffing_Prevention_Cheat_Sheet.md`
  - `Password_Storage_Cheat_Sheet.md`

- **🛡️ Autorisation & Contrôle d'Accès** :
  - `Authorization_Cheat_Sheet.md`
  - `Access_Control_Cheat_Sheet.md`
  - `Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.md`

- **💉 Injections & Base de Données** :
  - `SQL_Injection_Prevention_Cheat_Sheet.md`
  - `Query_Parameterization_Cheat_Sheet.md`
  - `Injection_Prevention_Cheat_Sheet.md`
  - `Database_Security_Cheat_Sheet.md`
  - `OS_Command_Injection_Defense_Cheat_Sheet.md`
  - `DotNet_Security_Cheat_Sheet.md`

- **🌐 UI, Frontend & XSS** :
  - `Cross_Site_Scripting_Prevention_Cheat_Sheet.md`
  - `DOM_based_XSS_Prevention_Cheat_Sheet.md`
  - `Content_Security_Policy_Cheat_Sheet.md`
  - `HTML5_Security_Cheat_Sheet.md`
  - `Securing_Cascading_Style_Sheets_Cheat_Sheet.md`

- **🔒 En-têtes, Cookies & CSRF** :
  - `Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.md`
  - `HTTP_Headers_Cheat_Sheet.md`
  - `Clickjacking_Defense_Cheat_Sheet.md`
  - `Cookie_Theft_Mitigation_Cheat_Sheet.md`

- **🔌 API, JWT, OAuth & Microservices** :
  - `REST_Security_Cheat_Sheet.md`
  - `JSON_Web_Token_Cheat_Sheet.md`
  - `OAuth2_Cheat_Sheet.md`
  - `GraphQL_Cheat_Sheet.md`
  - `Microservices_Security_Cheat_Sheet.md`

- **📥 Validation des Entrées & Gestion des Fichiers** :
  - `Input_Validation_Cheat_Sheet.md`
  - `File_Upload_Cheat_Sheet.md`
  - `Deserialization_Cheat_Sheet.md`
  - `Error_Handling_Cheat_Sheet.md`

- **⚙️ Serveurs, Cloud & Dépendances** :
  - `Docker_Security_Cheat_Sheet.md`
  - `Secure_Cloud_Architecture_Cheat_Sheet.md`
  - `Vulnerable_Dependency_Management_Cheat_Sheet.md`
  - `CI_CD_Security_Cheat_Sheet.md`
  - `Kubernetes_Security_Cheat_Sheet.md`

- **🤖 IA, Prompts & Agents AI** :
  - `AI_Agent_Security_Cheat_Sheet.md`
  - `LLM_Prompt_Injection_Prevention_Cheat_Sheet.md`
  - `MCP_Security_Cheat_Sheet.md`
  - `Secure_Coding_with_AI_Cheat_Sheet.md`
  - `RAG_Security_Cheat_Sheet.md`

---

## Doctrine de Sécurité (CRITIQUE)

### Principes fondamentaux

- **Tout input est malveillant par défaut**
- **Le client n'est jamais digne de confiance**
- **Le serveur est l'unique source de vérité**
- **Aucune clé secrète côté client**
- **Fail fast, fail closed**

---

## Hiérarchie des règles

### 🔴 OBLIGATOIRE
- ALWAYS check OWASP top 10+ vulnerabilities.
- Consultation ciblée obligatoire des cheatsheets Markdown pertinentes dans `data/` (`./security/data/cheatsheetseries.owasp.org/`).
- Aucune donnée sensible dans le client
- Validation serveur systématique
- Protection XSS active (encodage Razor par défaut, jamais de `Html.Raw` sur input utilisateur)
- `[Authorize]` explicite sur tout endpoint non public (ou politique globale + exceptions explicites)
- ALWAYS run `dotnet list package --vulnerable` / audit des dépendances NuGet

### 🟠 FORTEMENT RECOMMANDÉ
- Cookies `httpOnly` + `secure` + `sameSite`
- CSP stricte
- Rate limiting (`AddRateLimiter`)
- Logs d'erreurs centralisés (sans données sensibles)

### 🟢 OPTIONNEL
- Honeypots formulaires
- Détection d'abus avancée
- Headers de sécurité étendus (HSTS, Permissions-Policy)

---

## 1. XSS (Cross-Site Scripting)

### ❌ Anti-pattern critique

```cshtml
@* Razor — Html.Raw désactive l'encodage automatique *@
@Html.Raw(contenuUtilisateur)
```

### ✅ Alternative sécurisée

**S'appuyer sur l'encodage automatique de Razor** (par défaut) :

```cshtml
@contenuUtilisateur   @* encodé automatiquement *@
```

**Si du HTML est vraiment nécessaire, assainir côté serveur avec une librairie éprouvée** (HtmlSanitizer) :

```csharp
var sanitizer = new Ganss.Xss.HtmlSanitizer();
var safeHtml = sanitizer.Sanitize(input);
```

### Règles
- `Html.Raw` = interdit par défaut
- Jamais avec du contenu utilisateur brut
- Encodage adapté au contexte (HTML, attribut, JS, URL)

---

## 2. Gestion des entrées utilisateur

### Validation serveur obligatoire

```csharp
public sealed class RegisterRequest
{
    [Required, EmailAddress, MaxLength(256)]
    public required string Email { get; init; }

    [Range(18, 120)]
    public int Age { get; init; }
}
```

Ou avec FluentValidation pour les règles complexes. À activer systématiquement :

```csharp
builder.Services.AddFluentValidationAutoValidation();
```

### Bonnes pratiques
- Ne jamais faire confiance à la validation client
- Valider avant toute action métier
- Messages d'erreur génériques
- `[ApiController]` assure la validation automatique du modèle (400)

---

## 3. Secrets & Configuration

### ❌ Erreur critique

```csharp
var apiKey = configuration["ApiKey"]; // clé en clair dans appsettings.json versionné
```

### ✅ Sécurisé

```csharp
// User Secrets en dev : dotnet user-secrets set "Api:Key" "..."
// Variables d'environnement / coffre en prod
var apiKey = builder.Configuration["Api:Key"];
```

### Règles
- Pattern Options typé + `ValidateOnStart` (fail fast si le secret manque)
- Jamais de secret dans le code source, les logs ou les messages d'exception
- Rotation des clés ; scopes minimaux

---

## 4. Authentification

### Principes
- Auth server-side uniquement
- ASP.NET Core Identity pour la gestion des comptes (hashing Argon2/PBKDF2 intégré)
- Sessions/JWT gérés par le framework — ne jamais réimplémenter le hashing ou la génération de tokens

### Cookies sécurisés

```csharp
builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.HttpOnly = true;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.SameSite = SameSiteMode.Strict;
    options.ExpireTimeSpan = TimeSpan.FromHours(8);
    options.SlidingExpiration = true;
});
```

### ❌ À éviter
- Tokens en `localStorage` (XSS = vol de token) — cookie httpOnly ou stockage sécurisé
- Auth logique côté client
- Vérification des rôles côté client

---

## 5. Autorisation (RBAC / Policy-based)

### Toujours côté serveur

```csharp
// Politique déclarative
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("AdminOnly", p => p.RequireRole("admin"));

// Endpoint protégé
app.MapGet("/admin/users", GetUsers)
   .RequireAuthorization("AdminOnly");
```

### Ownership des ressources
- Vérifier que la ressource appartient à l'utilisateur authentifié avant lecture/modification/suppression (jamais de confiance à l'ID fourni par le client seul)

### Bonnes pratiques
- Vérifier l'autorisation dans chaque endpoint
- Jamais se fier au client
- Centraliser les règles d'accès (policies, handlers)

---

## 6. CSRF / Antiforgery

### Protection recommandée
- Antiforgery d'ASP.NET Core activé pour les formulaires POST :

```csharp
builder.Services.AddAntiforgery(options =>
{
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.HeaderName = "X-CSRF-TOKEN";
});
```

- Vérifier `Origin`/`Referer` pour les endpoints sensibles hors formulaire
- Éviter GET pour actions mutables

---

## 7. API & Endpoints

### Sécuriser les endpoints

```csharp
app.MapPost("/orders", [Authorize] async (CreateOrderRequest req, IOrderService orders, CancellationToken ct) =>
{
    var result = await orders.CreateAsync(req, ct);
    return Results.Created($"/orders/{result.Id}", result);
});
```

### Règles
- Toujours vérifier auth + autorisation
- Limiter les méthodes HTTP
- Réponses minimales (pas d'info inutile, pas de stack trace)
- Rate limiting sur les endpoints sensibles (login, mot de passe oublié)

---

## 8. Données sensibles

### ❌ Interdit
- Clés API dans le client
- Secrets dans les fichiers de vue
- Logs contenant tokens / PII

### ✅ Bonnes pratiques
- Configuration serveur (User Secrets / variables d'environnement / coffre)
- Rotation des clés
- Scopes minimaux
- **Data Protection API** pour chiffrer les données sensibles au repos applicatif

---

## 9. Sécurité des dépendances

### Règles
- Auditer régulièrement : `dotnet list package --vulnerable --include-transitive`
- Éviter les packages non maintenus
- `packages.lock.json` verrouillé et commité (`RestorePackagesWithLockFile`)

### ❌ Risque
- Sérialisation binaire legacy (`BinaryFormatter` = interdit absolument)
- Sanitization côté client uniquement

---

## 10. CSP & Headers HTTP

### CSP recommandée

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self';
  style-src 'self';
  img-src 'self' data:;
```

### Autres headers
- `X-Frame-Options: DENY`
- `X-Content-Type-Options: nosniff`
- `Referrer-Policy: strict-origin`
- HSTS activé (`UseHsts` en production)

---

## 11. Déploiement & Serveur

### Bonnes pratiques
- Pas de secrets dans le bundle ou l'image Docker
- Timeout stricts (reverse proxy + `CancellationToken`)
- Logs côté serveur uniquement
- Mode production : `app.UseExceptionHandler` générique, pas de détails d'exception au client

---

## 12. IA & Sécurité

### Règles critiques
- Appels IA **server-only**
- Nettoyer prompts utilisateur
- Ne jamais loguer prompts sensibles
- Limiter la taille des entrées

---

## Anti-patterns critiques

- ❌ `Html.Raw` avec input utilisateur
- ❌ Tokens dans `localStorage`
- ❌ Validation client uniquement
- ❌ Logique de sécurité côté client
- ❌ Secrets exposés au build
- ❌ `BinaryFormatter` / désérialisation non typée de données non fiables

---

## Checklist sécurité finale

- [ ] Aucune clé côté client
- [ ] Validation serveur complète
- [ ] Cookies `httpOnly` + `secure` + `sameSite`
- [ ] CSP active
- [ ] Auth + autorisation côté serveur sur tous les endpoints
- [ ] Logs nettoyés (pas de tokens/PII)
- [ ] Dépendances auditées (`dotnet list package --vulnerable`)
- [ ] Gestion d'erreurs générique (pas de fuite d'interne)


**Données Sources :** Situées dans `./security/data/` (Markdown).
