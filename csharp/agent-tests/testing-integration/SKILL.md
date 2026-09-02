---
name: testing-integration
description: Spécialiste des tests d'intégration pour valider la communication entre modules, services et API (ASP.NET Core, EF Core, WebApplicationFactory, Testcontainers).
---

# Integration Testing - Spécialiste Intégration
## Vue d'ensemble
Cette skill guide la création de **tests d'intégration**, qui vérifient que différents modules, services ou couches de l'application fonctionnent correctement ensemble. L'objectif est d'atteindre une couverture **absolue**. Je ne veux pas juste que les situations critiques ou essentielles soient couvertes, je veux que TOUTES les situations possibles d'interaction entre modules soient couvertes sans exception.


## Quand Utiliser Cette Skill
### ✅ Utilise les Tests d'Intégration Pour :
- Valider la communication entre un endpoint et son service métier.
- Tester les endpoints d'API (requête → logique → base de données).
- Vérifier les effets secondaires complexes (ex: envoi d'email après inscription).
- S'assurer que les migrations EF Core fonctionnent avec le code.
- Tester l'intégration avec des services tiers (via des mocks ou versions locales).
### ❌ N'Utilise PAS les Tests d'Intégration Pour :
- La logique purement algorithmique → [`testing-unit`](./testing-unit/SKILL.md).
- Les parcours utilisateurs complexes dans le navigateur → [`testing-e2e`](./testing-e2e/SKILL.md).
## Stratégie d'Intégration
### 1. API Testing (Backend)
Vérifier qu'un appel HTTP produit le bon changement d'état et la bonne réponse.
- **Outils :** xUnit + `WebApplicationFactory<Program>`, `HttpClient`.
- **Focus :** Status codes, structure JSON, persistance en DB.
### 2. Service/Repository Testing
Vérifier la logique métier qui interagit avec la base de données.
- **Focus :** Requêtes LINQ/EF Core, transactions, contraintes d'intégrité, concurrence.
### 3. Component Integration (Couche Application)
Vérifier l'interaction entre les services et le contexte de persistance.
## Structure d'un Test API (Exemple)
```csharp
public class ClientsApiTests(WebApplicationFactory<Program> factory)
    : IClassFixture<WebApplicationFactory<Program>>
{
    [Fact]
    public async Task Post_DoitCreerUnNouveauClient_EtRenvoyer201()
    {
        // Arrange : DB de test fraîche (fixture) + client HTTP
        var client = factory.CreateClient();
        var payload = new { name = "Client Test", email = "test@example.com" };

        // Act
        var response = await client.PostAsJsonAsync("/api/clients", payload);
        var data = await response.Content.ReadFromJsonAsync<ClientDto>();

        // Assert : réponse HTTP
        response.StatusCode.ShouldBe(HttpStatusCode.Created);
        data!.Name.ShouldBe("Client Test");

        // Assert : persistance en base de données (DbContext de test)
        using var scope = factory.Services.CreateScope();
        var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
        var clientInDb = await db.Clients
            .SingleOrDefaultAsync(c => c.Email == payload.email);
        clientInDb.ShouldNotBeNull();
    }
}
```
## Gestion des Dépendances Externes
### Base de Données
- Utilisez une base de données de test réelle : **Testcontainers** (conteneur PostgreSQL/SQL Server éphémère) pour la fidélité maximale, ou SQLite in-memory pour la rapidité (avec le caveat des différences de dialecte SQL).
- **Règle d'or :** Toujours nettoyer la base avant ou après chaque test (`Respawn` pour le reset des tables, ou rollback de transaction / DbContext scoping par test).
### Services Tiers (Emails, SMS, APIs Externes)
- Ne faites jamais d'appels réels en test d'intégration.
- Utilisez des **substituts** (NSubstitute / Moq) injectés via la DI du `WebApplicationFactory`, ou des services de test comme MailHog/Mailtrap.
## Patterns Recommandés
### Pattern Repository / Service Isolation
Séparez la logique de données de la logique métier pour faciliter le test des interactions.
### Pattern Endpoint/Handler
Testez que l'endpoint appelle correctement les services et traduit correctement les résultats (code de statut, Problem Details).
## Audit & Rapport de Santé (Crucial)
Un test qui passe au "vert" n'est pas forcément un bon test. Après chaque exécution, vous DEVEZ auditer les sorties console et la structure pour identifier :
### 1. Le Bruit (Logs Parasites)
- Identifiez les erreurs `HttpRequestException` ou `UriFormatException` (souvent dues à des `HttpClient` mal configurés dans l'environnement de test).
- Signalez les `Console.Error` ou warnings du logger qui polluent le rapport même si les tests réussissent.
### 2. La Fragilité des Substituts
- Vérifiez si les substituts retournent des valeurs statiques (ex: `Id = 123`) qui pourraient causer des erreurs `UNIQUE constraint failed` lors de tests parallèles ou répétés.
- Préférez des générateurs d'identifiants (GUID ou compteurs) dans les substituts.
### 3. Les Effets de Bord
- Assurez-vous que chaque test nettoie TOUTES les tables qu'il a modifiées.
- Vérifiez que les états globaux de test (fixtures partagées, caches statiques) sont réinitialisés (`IAsyncLifetime` / constructeur xUnit).

## Structure du Rapport Post-Exécution
Après avoir exécuté des tests, présentez toujours un bref résumé à l'utilisateur :
1. **Statut** : (Réussite/Échec)
2. **Qualité de l'isolation** : (Les données sont-elles propres ?)
3. **Points d'attention** : (Bruit dans les logs, erreurs 500 silencieuses dans les substituts, etc.)
4. **Améliorations suggérées** : (Nettoyage de code, renforcement des substituts).
---
**Skill liée :** `testing-diamond` pour coordonner avec les autres types de tests.
