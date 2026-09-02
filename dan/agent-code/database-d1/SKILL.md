---
name: database-d1
description: Gestion de la base de données Cloudflare D1 et intégrité des données.
---

# Gestion Database D1

Ce skill couvre l'interaction avec Cloudflare D1 et la gestion des schémas.

## 1. Requêtes SQL
- **Prepared Statements** : Utiliser impérativement `db.prepare('... = ?').bind(value)`.
- **Injection SQL** : Ne jamais concaténer de variables directement dans les chaînes SQL.
- **Filtre Tenant** : Vérifier systématiquement la présence de `compagnie_id` dans les clauses `WHERE`.

## 2. Migrations
- **Commande** : `npx wrangler d1 migrations apply rdv_db --local` pour appliquer localement.
- **Nouveaux champs** : Toujours définir des longueurs maximales pour les champs texte (conforme au standard de sécurité).
- **Validation** : Valider les schémas avec Zod côté serveur avant insertion.

## 3. Optimisation
- **Transactions** : Utiliser `db.batch()` pour les opérations multiples liées.
- **Index** : S'assurer que les colonnes utilisées fréquemment dans les filtres (comme `compagnie_id` et les dates) sont indexées.
