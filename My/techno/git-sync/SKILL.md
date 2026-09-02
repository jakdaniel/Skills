---
name: git-sync
description: Workflow Git sécurisé pour KalySync, incluant la synchronisation et le respect des branches.
---

# Workflow Git & Synchronisation

Ce skill garantit l'intégrité du dépôt Git et le respect du flux de travail.

## 1. Gestion des Branches
- **Branche Secondaire** : Toujours travailler sur une branche de fonctionnalité (ex: `feat/nom-feature`).
- **Interdiction Main** : Ne JAMAIS pousser ou fusionner directement vers `main`.

## 2. Processus de Synchronisation
Avant tout push sur la branche de fonctionnalité :
1. Récupérer les derniers changements : `git checkout main`, `git pull origin main`.
2. Revenir sur la branche : `git checkout <ma-branche>`.
3. Synchroniser (merger main dans la branche) : `git merge main`.
4. Résoudre les conflits si nécessaire.

## 3. Commit & Push
- **Validation** : Attendre TOUJOURS la validation explicite de l'utilisateur avant de commit.
- **Commande** : Exécuter `commit` et `push` uniquement sur demande.
- **Review** : Présenter un résumé clair des changements effectués.

## 4. Déploiement
- Le déploiement vers la production (Cloudflare Pages) s'effectue uniquement après validation et fusion via une Pull Request (gérée côté utilisateur ou sur demande spécifique).
