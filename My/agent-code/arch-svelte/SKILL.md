---
name: arch-svelte
description: Standards de développement pour Svelte 5 et architecture multi-tenant KalySync.
---

# Architecture Svelte 5 & Multi-tenant

Ce skill définit les standards de code et d'architecture pour le projet KalySync.

## 1. Réactivité (Runes)
- **Runes exclusivement** : Utiliser `$state`, `$derived`, `$props`, `$inspect`, `$effect`.
- **Interdiction** : Ne pas utiliser les stores Svelte 4 (`writable`, `readable`, `derived`) sauf si une librairie tierce l'exige.
- **Logique métier** : Encapsuler la logique dans des classes ou des fichiers `.svelte.ts` utilisant des runes.

## 2. Isolation Multi-tenant (CRITIQUE)
- **Filtrage** : TOUTES les requêtes SQL doivent inclure un filtre `WHERE compagnie_id = ?`.
- **Validation** : Toujours valider que l'utilisateur appartient à la compagnie dont il demande les données.
- **Sécurité** : Ne jamais exposer de données d'une compagnie à une autre.

## 3. Structure des Composants
- **Taille Max** : Un fichier `.svelte` ne doit pas dépasser **400 lignes**.
- **Séparation** : Séparer l'UI de la logique complexe. Utiliser des snippets pour les parties répétitives du markup.
- **Style** : Utiliser SCSS scopé. Préférer les variables CSS globales pour la cohérence visuelle.

## 4. Performance & UX
- **Données Statiques** : Éviter les requêtes inutiles vers la DB pour les `AppSettings`. Utiliser le cache ou le contexte.
- **Design** : Respecter les standards de UI/UX moderne (Glassmorphism, animations fluides, typographie premium).
- **Images** : Ne pas utiliser de placeholders. Utiliser `generate_image` pour des démonstrations réelles.
