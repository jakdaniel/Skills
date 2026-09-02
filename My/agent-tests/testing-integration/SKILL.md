---
name: testing-integration
description: Spécialiste des tests d'intégration pour valider la communication entre modules, services et API.
---

# Integration Testing - Spécialiste Intégration
## Vue d'ensemble
Cette skill guide la création de **tests d'intégration**, qui vérifient que différents modules, services ou couches de l'application fonctionnent correctement ensemble. L'objectif est d'atteindre une couverture **absolue**. Je ne veux pas juste que les situations critiques ou essentielles soient couvertes, je veux que TOUTES les situations possibles d'interaction entre modules soient couvertes sans exception.


## Quand Utiliser Cette Skill
### ✅ Utilise les Tests d'Intégration Pour :
- Valider la communication entre un composant UI et son store d'état.
- Tester les endpoints d'API (requête → logique → base de données).
- Vérifier les effets secondaires complexes (ex: envoi d'email après inscription).
- S'assurer que les migrations de base de données fonctionnent avec le code.
- Tester l'intégration avec des services tiers (via des mocks ou versions locales).
### ❌ N'Utilise PAS les Tests d'Intégration Pour :
- La logique purement algorithmique → [`testing-unit`](file:///C:/Users/jakda/.agent/skills/tests/testing-unit/SKILL.md).
- Les parcours utilisateurs complexes dans le navigateur → [`testing-e2e`](file:///C:/Users/jakda/.agent/skills/tests/testing-e2e/SKILL.md).
## Stratégie d'Intégration
### 1. API Testing (Backend)
Vérifier qu'un appel HTTP produit le bon changement d'état et la bonne réponse.
- **Outils :** Vitest, Supertest (ou fetch simulé).
- **Focus :** Status codes, structure JSON, persistance en DB.
### 2. Service/Repository Testing
Vérifier la logique métier qui interagit avec la base de données.
- **Focus :** Requêtes SQL/Query Builder, transactions, contraintes d'intégrité.
### 3. Component Integration (Frontend)
Vérifier l'interaction entre les composants et les stores (Pinia/Vuex/Redux).
## Structure d'un Test API (Exemple)
```typescript
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { db } from '@/db';
import { createServer } from '@/server';
describe('API: /api/clients', () => {
  beforeAll(async () => {
    // Setup: Reset DB ou transactions
  });
  describe('POST /', () => {
    it('doit créer un nouveau client et renvoyer 201', async () => {
      const payload = { name: 'Client Test', email: 'test@example.com' };
      
      const response = await fetch('http://localhost:3000/api/clients', {
        method: 'POST',
        body: JSON.stringify(payload)
      });
      
      const data = await response.json();
      
      // Assertions
      expect(response.status).toBe(201);
      expect(data.name).toBe(payload.name);
      
      // Vérification en base de données
      const clientInDb = await db.query('SELECT * FROM clients WHERE email = ?', [payload.email]);
      expect(clientInDb).toBeDefined();
    });
  });
});
```
## Gestion des Dépendances Externes
### Base de Données
- Utilisez une base de données de test séparée (ex: SQLite en mémoire ou conteneur Docker).
- **Règle d'or :** Toujours nettoyer la base avant ou après chaque test (`TRUNCATE` ou `rollback` de transaction).
### Services Tiers (Emails, SMS, APIs Externes)
- Ne faites jamais d'appels réels en test d'intégration.
- Utilisez des **Mocks** (vi.mock) ou des services de test comme MailHog/Mailtrap.
## Patterns Recommandés
### Pattern Repository / Service Isolation
Séparez la logique de données de la logique métier pour faciliter le test des interactions.
### Pattern Page/Container (Frontend)
Testez que le "Container" (la page) appelle correctement les services/stores lors des actions utilisateur.
## Audit & Rapport de Santé (Crucial)
Un test qui passe au "vert" n'est pas forcément un bon test. Après chaque exécution, vous DEVEZ auditer les sorties console et la structure pour identifier :
### 1. Le Bruit (Logs Parasites)
- Identifiez les erreurs `TypeError: Invalid URL` (souvent dues à des appels `fetch` relatifs dans un environnement Node/Vitest).
- Signalez les `console.error` ou `console.warn` qui polluent le rapport même si les tests réussissent.
### 2. La Fragilité des Mocks
- Vérifiez si les mocks retournent des valeurs statiques (ex: `id: 123`) qui pourraient causer des erreurs `UNIQUE constraint failed` lors de tests parallèles ou répétés.
- Préférez des générateurs d'identifiants (UUID ou compteurs) dans les mocks.
### 3. Les Effets de Bord
- Assurez-vous que chaque test nettoie TOUTES les tables qu'il a modifiées.
- Vérifiez que les variables globales de test (ex: `pendingTasks`) sont réinitialisées dans `beforeEach`.

## Structure du Rapport Post-Exécution
Après avoir exécuté des tests, présentez toujours un bref résumé à l'utilisateur :
1. **Statut** : (Réussite/Échec)
2. **Qualité de l'isolation** : (Les données sont-elles propres ?)
3. **Points d'attention** : (Bruit dans les logs, erreurs 500 silencieuses dans les mocks, etc.)
4. **Améliorations suggérées** : (Nettoyage de code, renforcement des mocks).
---
**Skill liée :** `testing.pyramid.md` pour coordonner avec les autres types de tests.