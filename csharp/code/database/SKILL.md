---
name: database
description: Gestion de la base de données avec Entity Framework Core (LINQ, migrations, transactions, index, intégrité des données) et intégrité des données.
---

# Gestion Database — Entity Framework Core

Ce skill couvre l'interaction avec la base de données via **Entity Framework Core** et la gestion des schémas.

## 1. Requêtes & Protection Injection
- **LINQ paramétré** : Les requêtes LINQ→SQL sont paramétrées nativement — ne jamais construire de SQL dynamique par concaténation.
- **SQL brut** : Si nécessaire, utiliser uniquement `FromSqlInterpolated` / `ExecuteSqlInterpolated` (paramétrage automatique) — `FromSqlRaw` avec paramètres explicites en dernier recours. `FromSqlRaw` avec interpolation manuelle de chaîne est **interdit**.
- **Filtre de propriété** : Toujours filtrer par les données de l'utilisateur authentifié (`CurrentUserId`) côté serveur ; jamais de lecture/écriture par identifiant fourni par le client sans vérification d'ownership.

## 2. Migrations EF Core
- **Commande** : `dotnet ef migrations add <Nom>` puis `dotnet ef database update` (générer le script idempotent `--idempotent` pour la production).
- **Nouveaux champs** : Toujours définir des longueurs maximales pour les champs texte (`HasMaxLength`, conformité au standard de sécurité) et des valeurs par défaut pour les colonnes NOT NULL.
- **Validation** : Valider les entrées avec DataAnnotations / FluentValidation côté serveur avant insertion — la validation entité ne remplace pas la validation d'entrée.
- **Migrations destructrices** : Toute perte de données potentielle (drop de colonne/table, changement de type) doit être signalée et revue avant application.

## 3. Performance & Requêtes
- **AsNoTracking** : `AsNoTracking()` pour toute lecture sans modification.
- **Projections** : Utiliser `Select` pour ne charger que les colonnes nécessaires — éviter de charger des entités complètes pour des listes/DTO.
- **N+1** : Traquer les boucles de requêtes ; utiliser `Include`/`ThenInclude` ou des projections quand c'est pertinent (attention au sur-chargement).
- **Pagination** : Toute requête de liste est paginée (`Skip`/`Take`) et ordonnée de façon déterministe — `OrderBy` obligatoire avant `Skip/Take`.

## 4. Transactions & Concurrence
- **Transactions** : Les opérations multiples liées passent par `Database.BeginTransactionAsync` ou une seule `SaveChangesAsync` (transaction implicite).
- **Concurrence optimiste** : `RowVersion`/`IsRowVersion` (token de concurrence) sur les entités modifiables par plusieurs acteurs ; gérer `DbUpdateConcurrencyException`.
- **Index** : S'assurer que les colonnes utilisées fréquemment dans les filtres (clés étrangères, dates, états) sont indexées via la configuration d'entité (`HasIndex`).

## 5. Intégrité & Modélisation
- **Contraintes** : Clés étrangères, contraintes d'unicité et `required` modélisés dans la configuration (`OnModelCreating` ou `IEntityTypeConfiguration<T>` — une configuration par entité).
- **SaveChanges centralisé** : Passer par des méthodes de repository/services dédiés ; intercepter `DbUpdateException` pour traduire les violations de contraintes en erreurs métier claires.
- **Soft delete** : Si utilisé, appliquer un *global query filter* (`HasQueryFilter`) pour que le filtrage soit automatique et non contournable.
