---
name: code-validator
description: Analyse et valide de façon exhaustive tout code écrit, généré ou modifié par Claude (directement ou via un autre skill), en le confrontant point par point à la demande initiale, avant de considérer la tâche terminée. Déclenche-toi systématiquement dès que du code vient d'être produit — fonctionnalité, correction de bug, refactoring, génération via un autre skill — même sans les mots "valide" ou "revue de code". Déclenche-toi aussi sur demande explicite ("valide ce code", "est-ce complet ?"). Vérifie la conformité complète à la demande (rien d'oublié, aucun raccourci silencieux), le respect de l'architecture du projet (Clean Architecture, Vertical Slice), la robustesse (erreurs, cas limites, concurrence, nullabilité) et l'absence de code factice (TODO, stubs, NotImplementedException, tests sans assertion). Corrige directement ce qu'il trouve plutôt que de se contenter de le signaler.
---

# Validation exhaustive de code

## Pourquoi ce skill existe

Un code qui compile et qui "a l'air" de répondre à la demande n'est pas la même chose qu'un code qui répond réellement, entièrement, à la demande. Le risque le plus courant n'est pas l'erreur flagrante — c'est le raccourci silencieux : un cas limite non traité, une exigence de la demande initiale discrètement laissée de côté, une gestion d'erreur qui avale l'exception au lieu de la traiter, un `TODO` oublié dans un coin. Ce skill existe pour traquer précisément ce genre de choses, systématiquement, même quand tout semble aller bien à première lecture.

Ce skill s'exécute **après** qu'un skill de génération/écriture de code (ou toi-même sans skill particulier) ait produit du code. Il ne remplace pas ce travail, il le contrôle.

## Quand s'activer

Considère ce skill comme faisant partie intégrante de la fin de toute tâche de code, pas comme une étape optionnelle qu'on saute par manque de temps :

- **Automatiquement**, juste après avoir terminé d'écrire, générer ou modifier du code — que ce soit toi qui l'as écrit directement ou un autre skill invoqué dans la même conversation. Ne considère pas la tâche terminée tant que cette validation n'a pas eu lieu.
- **Sur demande explicite** de l'utilisateur, à tout moment — y compris pour du code qui n'a pas été produit dans la conversation en cours (un fichier existant, un commit, une PR).

Ne demande pas la permission de valider — fais-le, puis présente le résultat. En revanche, avant d'appliquer des correctifs qui changeraient un comportement observable de façon significative (pas juste combler un oubli évident), utilise ton jugement : les corrections mineures et sans ambiguïté (gérer une exception non traitée, compléter un cas manquant, retirer un stub) s'appliquent directement ; un changement qui suppose une décision de conception que l'utilisateur n'a pas tranchée se signale dans le rapport plutôt que de s'imposer.

## Processus de validation

### Étape 1 — Reconstituer la demande initiale

Avant de juger le code, remonte dans la conversation et liste explicitement ce qui a été demandé : les exigences fonctionnelles énoncées, mais aussi les contraintes implicites données par le contexte (conventions déjà en place dans le projet, architecture déjà choisie, patterns déjà utilisés ailleurs dans la base de code). Une demande comme "ajoute la validation du formulaire" porte avec elle des attentes non écrites — messages d'erreur cohérents avec le reste de l'app, mêmes conventions de nommage, etc. Ignorer ces attentes implicites parce qu'elles n'étaient "pas explicitement demandées" est exactement le genre de coin rond que ce skill doit attraper.

Écris cette liste avant de regarder le code produit, pas après — sinon tu risques de la reconstruire a posteriori pour qu'elle corresponde à ce qui a été livré.

### Étape 2 — Inventaire de ce qui a été livré

Liste tous les fichiers créés ou modifiés. Pour chacun, lis le contenu réel — ne te fie pas à un résumé donné plus tôt dans la conversation (le résumé peut lui-même avoir omis quelque chose).

### Étape 3 — Vérification point par point

Passe le code au crible des catégories suivantes. Pour chaque point, la question à te poser est toujours la même : *est-ce que je peux le prouver en pointant une ligne de code précise, ou est-ce que je suppose que c'est bon ?* Si tu ne peux pas pointer la preuve, considère que ce n'est pas vérifié.

**Complétude fonctionnelle**
- Chaque exigence de la liste de l'étape 1 est-elle couverte par du code réel (pas un commentaire disant qu'elle le sera) ?
- Y a-t-il des chemins alternatifs mentionnés dans la demande (ex. "et si l'utilisateur n'est pas connecté") qui n'ont pas de traitement ?
- Le code produit fait-il *seulement* ce qui a été demandé, ou a-t-il aussi silencieusement changé un comportement existant qui n'était pas dans le périmètre ?

**Architecture et cohérence avec le projet**
- Le code respecte-t-il la séparation de couches déjà en place (Clean Architecture : domaine indépendant de l'infrastructure ; Vertical Slice : une feature = un dossier cohérent, sans coupler des slices entre elles inutilement) ?
- Une classe de domaine référence-t-elle par erreur EF Core, un détail HTTP, ou toute autre dépendance d'infrastructure ?
- Le code réutilise-t-il les abstractions déjà existantes dans le projet, ou en réinvente-t-il une variante légèrement différente ?

**Robustesse — gestion d'erreurs et cas limites**
- Les exceptions attendues sont-elles traitées, ou seulement les cas nominaux ?
- Y a-t-il des `catch` vides ou des `catch (Exception)` génériques qui avalent l'erreur sans la logger ni la relancer ?
- Nullabilité : les `Nullable Reference Types` sont-ils respectés, ou y a-t-il des `!` (null-forgiving) posés par facilité plutôt que par certitude réelle ?
- `async`/`await` : y a-t-il des `async void` en dehors des handlers d'événements, des `.Result`/`.Wait()` risquant un deadlock, des tâches lancées sans être attendues (fire-and-forget non voulu) ?
- Cycle de vie des objets `IDisposable` : sont-ils bien libérés (`using`, ou disposal explicite) ?
- Durée de vie des services injectés (Singleton/Scoped/Transient) : y a-t-il un service Scoped injecté dans un Singleton (captive dependency) ?
- Requêtes EF Core : y a-t-il un risque de N+1, un `.ToList()` prématuré qui charge toute la table, un tracking inutile sur une requête en lecture seule ?
- Concurrence : si plusieurs accès simultanés sont plausibles, y a-t-il une protection (verrou, transaction, `RowVersion`) ou est-ce ignoré ?

**Absence de code factice**
- Recherche activement les marqueurs de raccourci : `TODO`, `FIXME`, `throw new NotImplementedException()`, valeurs codées en dur qui devraient être configurables, données de test laissées dans le code de production, méthodes qui retournent une valeur par défaut sans vraiment calculer quoi que ce soit.
- Un test qui ne fait qu'appeler la méthode sans assertion significative (ou avec un `Assert.True(true)`) compte comme du code factice.

### Étape 4 — Corriger directement

Pour chaque problème confirmé et sans ambiguïté de conception, corrige-le directement dans le code plutôt que de te contenter de le décrire — c'est ce qui a été demandé : un skill qui répare, pas seulement qui signale. Documente quand même chaque correctif dans le rapport final (voir plus bas), pour que l'utilisateur sache ce qui a changé sans avoir à relire le diff en entier.

Pour les problèmes qui impliquent une vraie décision de conception (par exemple : "cette fonctionnalité gère-t-elle le cas multi-tenant ou pas ?"), ne tranche pas à la place de l'utilisateur — signale-le clairement dans le rapport comme point ouvert.

### Étape 5 — Revalider après correctifs

Une fois les correctifs appliqués, relis-les une deuxième fois avec la même rigueur qu'à l'étape 3 — un correctif rapide introduit parfois son propre raccourci (ex. : traiter une exception en la loggant, mais sans la relancer, alors que l'appelant en avait besoin). Une seule passe de revalidation suffit ; au-delà, présente l'état actuel et laisse l'utilisateur trancher plutôt que de boucler indéfiniment.

### Étape 6 — Rapport final

Termine toujours par un rapport écrit, même quand tout est conforme — l'absence de rapport ne doit jamais être interprétée comme "tout va bien", ça doit toujours être dit explicitement et étayé.

## Format du rapport

Utilise toujours cette structure :

```markdown
## Validation du code — [nom de la fonctionnalité/tâche]

### Conformité à la demande
[Pour chaque exigence identifiée à l'étape 1 : ✅ couverte / ⚠️ partielle / ❌ manquante, avec une ligne d'explication]

### Problèmes trouvés et corrigés
[Liste des problèmes réels trouvés, classés par catégorie (architecture / robustesse / code factice), avec le fichier et la correction appliquée]

### Points ouverts (nécessitent une décision de ta part)
[Uniquement s'il y en a — sinon, omettre cette section entièrement, ne pas écrire "aucun"]

### Verdict
[Une phrase directe : conforme et robuste / conforme avec réserves / non conforme — sans enjoliver]
```

## Pièges à éviter

- Ne dis jamais "tout semble correct" sans avoir explicitement vérifié chaque exigence de l'étape 1 — "sembler correct" et "être vérifié" sont deux choses différentes.
- Ne réduis pas la validation à une relecture superficielle du diff final : relis le code exécuté dans son contexte réel (comment il est appelé, avec quelles données).
- Ne masque pas un point ouvert en le noyant dans un correctif que tu appliques toi-même sans le signaler — une décision de conception qui n'a pas été prise par l'utilisateur reste une décision non prise, même si tu as codé quelque chose pour que ça compile.
- Si la demande initiale elle-même était ambiguë, ne comble pas l'ambiguïté en silence pendant la validation — signale-la, ne fais pas semblant qu'elle n'existait pas.