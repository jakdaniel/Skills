---
name: product-brain
description: Tient à jour la mémoire du produit — un "second cerveau" par feature dans docs/Brain/{feature}.md — qui décrit le FONCTIONNEMENT détaillé et l'état ACTUEL d'une feature (le but, les étapes précises, comment, et pourquoi tel choix). Utilise ce skill DÈS QUE tu commences ou modifies une feature (lis d'abord son doc/Brain), DÈS QU'une décision de comportement produit est prise, clarifiée, corrigée ou révisée (même une petite précision du genre "en fait non, quand X arrive il faut Y"), et DÈS QU'un cas limite ou un piège vient d'être tranché. Le doc/Brain est un DOCUMENT VIVANT : à chaque décision, on met à jour l'état actuel (on EDITE la section concernée), jamais on n'empile un journal daté de changements. Si une question porte sur "c'était censé faire quoi", "pourquoi on a fait ça comme ça", ou "comment ça marche", consulte d'abord le doc/Brain correspondant avant de répondre ou de recoder.
---

# Product Brain

Ce skill fait de `docs/Brain/` la mémoire long terme du produit : un **document vivant du fonctionnement actuel** de chaque feature — son but, ses étapes détaillées, comment elle procède, et pourquoi elle a été décidée ainsi. Ce n'est **ni** une doc technique de code, **ni** un journal de bord des changements. C'est la description fiable et durable de l'état présent, capable de faire reconstruire la feature de zéro par quelqu'un qui ne l'a jamais vue.

## Pourquoi ça existe

En vibe coding, le code avance vite mais la mémoire des décisions fines (comportements, étapes, edge cases, "pourquoi pas l'autre approche") reste seulement dans la tête de l'utilisateur ou perdue dans l'historique du chat. Résultat : on redemande, on re-décide différemment, on recasse ce qui avait été corrigé. `docs/Brain/{feature}.md` est la source de vérité qu'on consulte AVANT de coder — c'est la base de connaissance sur la feature, pas seulement un compte-rendu écrit APRÈS.

## Règle d'or

**Lire avant de coder, mettre à jour au fil de l'eau — jamais en fin de session. Et l'on édite l'état actuel, on n'accumule pas un journal.**

Un doc/Brain qui n'est mis à jour qu'à la fin d'une tâche a déjà raté la moitié de son utilité : si une décision est prise à l'étape 2 d'une tâche à 5 étapes et que tu crashes, oublies, ou changes de sujet avant la fin, elle est perdue.

## Quand lire un doc/Brain

Avant toute implémentation ou modification touchant une feature existante :

1. Vérifie si `docs/Brain/{feature-slug}.md` existe.
2. S'il existe, lis-le en entier avant d'écrire du code — en particulier "But & valeur", "Déroulé détaillé", "Décisions & cas limites" et "Pièges & contraintes".
3. Si le code que tu observes contredit le doc, **ne tranche pas silencieusement** : signale l'écart à l'utilisateur ("le doc dit X, le code fait Y — lequel est correct ?") et corrige le doc une fois la réponse obtenue. Un doc silencieusement faux est pire qu'un doc absent.
4. Si aucun doc n'existe pour une feature sur laquelle tu travailles, crée-le dès la première décision de comportement rencontrée — inutile d'attendre une feature "terminée".

## Ce que le Brain contient : l'état actuel, pas l'historique

Chaque `docs/Brain/{feature}.md` décrit **ce que la feature fait MAINTENANT** et **pourquoi elle est faite ainsi**. Structure recommandée (gabarit `feature-template.md`, dans ce même dossier) :

- **But & valeur** — à quoi sert la feature, pour qui, quelle problème elle résout (2-4 phrases).
- **Vue d'ensemble / déroulé** — les grandes étapes ordonnées de la feature, de l'entrée à la sortie.
- **Détail des étapes** — pour chaque étape : ce qu'elle fait, comment, où (fichiers/routes), et les subtilités importantes.
- **Décisions & cas limites** — tableaux : cas, comportement retenu, et pourquoi les alternatives ont été écartées.
- **Pièges & contraintes** — les erreurs types à ne pas refaire, les invariants (formules, totaux, locks), et les règles dures. Non daté, orienté "ce à quoi je dois faire attention".
- **Contrats & interfaces** — endpoints, signatures, DTOs, constantes, routes, règles de validation qui doivent rester stables parce que d'autres parties du code en dépendent.
- **Limites & hors-périmètre** — ce que la feature ne fait pas (exprimé délibérément), pour ne pas la refaire par erreur.

## Ce qui va au Brain, ce qui n'y va pas

Le signal à garder : **"si je perds cette info, est-ce que je risque de recoder différemment ou de reposer la question, OU est-ce que ça décrit le fonctionnement réel de la feature ?"** Si oui → ça va dans le Brain.

**Oui, ça va dans le Brain (comportement observable et durable) :**

- Le but de la feature et sa valeur.
- Le déroulé des étapes (l'ordre, ce qui est fait, à quel moment).
- Une décision de comportement produit (validation, edge case, règle métier, format).
- Un invariant fonctionnel (une formule, une borne, un verrou).
- Un contrat d'interface qui se stabilise (DTO, endpoint, constante partagée).
- Une alternative essayée puis abandonnée ET POURQUOI (pour ne pas la retenter).

**Non, ça n'y va PAS (correctifs de code sans impact produit) :**
- Le renommage de variables, la factorisation interne, l'indentation, le style — sauf s'ils changent le fonctionnement observable.
- L'historique daté des ANO / allers-retours de revue. On ne veut pas "quand tel fix a été appliqué", on veut l'état final.
- Les détails d'implémentation pure qui n'aident pas à reconstruire la feature.

> La règle : si c'est une correction de code (le "comment coder" pour rendre le code propre), ce n'est pas du Brain. Si ça touche au comportement fonctionnel (le "comment ça marche"), ça l'est.

## Le ritual de mise à jour : éditer, jamais journaliser

Une décision, une clarification, un fix fonctionnel → **édite la section concernée** pour refléter le nouvel état actuel :

- On modifie le texte de la section pour qu'il décrive l'état final du comportement.
- On ne cumule pas d'entrées datées ("[2026-08-15] …", "ANO-07…"). On n'empile pas de "Historique" des changements.
- On met la décision à jour là où elle appartient (ex. une règle dans "Décisions & cas limites", une étape dans "Détail des étapes", une borne dans "Contrats & interfaces").

Si une décision est révisée, on **remplace** l'ancienne mention par la nouvelle. Le doc doit refléter l'état présent, pas garder toutes les versions par lesquelles on est passé. C'est ce qui le rend court, lisible et fiable. L'historique des revues de code n'a pas sa place dans le Brain — au besoin il vit ailleurs (commit, PR, doc dev).

**Cela ne change pas l'exigence de synchronisation** : on met à jour au fil de l'eau, jamais à la fin. Simplement, "mettre à jour" = éditer l'état actuel, pas append un journal.

## Granularité d'une feature

Une feature = une tranche de comportement produit délimitée et nommable (ex. `authentification`, `panier-achat`, `export-facture`), pas une classe ou un fichier. Convention de nommage : kebab-case, aligné sur le nom du module/domaine tel que l'utilisateur en parle naturellement. En cas de doute sur la granularité (trop gros vs trop fin), demande, ou pars large et scinde plus tard si le doc devient difficile à parcourir.

## Index global

Maintiens `docs/Brain/INDEX.md` (gabarit `index-template.md`, dans ce même dossier) : une ligne par feature avec son nom, une description d'une phrase, et un lien vers son doc. Mets-le à jour dès qu'une nouvelle feature obtient son propre doc. C'est le point d'entrée pour retrouver rapidement "où est la mémoire de X".

## Réflexe anti-aller-retour

Avant d'implémenter un comportement dont tu n'es pas sûr à 100 % :

1. Cherche-le dans le doc/Brain de la feature concernée.
2. S'il n'y est pas, pose la question à l'utilisateur plutôt que de deviner.
3. Dès que la réponse arrive, mets à jour le doc — dans le même tour, avant de continuer à coder.

Ce cycle "chercher → demander si absent → éditer l'état immédiatement" est ce qui, avec le temps, réduit les questions répétées et les régressions de comportement déjà tranché.