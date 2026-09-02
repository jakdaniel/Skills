# Brain — {Nom de la feature}

> Document vivant de l'**état actuel** de la feature. Source de vérité sur son fonctionnement détaillé et ses décisions. À mettre à jour à CHAQUE décision, clarification ou correction fonctionnelle — en **éditant** la section concernée, jamais en accumulant un journal daté.

## But & valeur

(2-4 phrases : à quoi sert cette feature, pour qui, quel problème elle résout, dans quel contexte du produit)

## Vue d'ensemble / déroulé

(Les grandes étapes ordonnées, de l'entrée à la sortie. Une liste courte — la "story" de la feature.)

1. ...
2. ...
3. ...

## Détail des étapes

(Par étape : ce qu'elle fait, comment, où (fichiers/routes), et les subtilités qui comptent.)

### Étape 1 — {titre}

- **Ce qui se passe** : ...
- **Comment** : ... (`fichier`, route)
- **Subtil** : ...

### Étape 2 — {titre}

...

## Décisions & cas limites

(Tableau : cas | comportement retenu | pourquoi les alternatives ont été écartées. L'état final, pas un historique.)

| Cas | Comportement retenu | Pourquoi (alternative écartée) |
| :--- | :--- | :--- |
| {cas limite} | {comportement} | {raison} |

## Pièges & contraintes

(Ce à quoi faire attention : erreurs types à ne pas refaire, invariants (formules, totaux, verrous), règles dures. Non daté.)

- {invariant ou règle dure} — {pourquoi}

## Contrats & interfaces

(Signatures, endpoints, DTOs, constantes, routes, règles de validation qui doivent rester stables car d'autres parties du code en dépendent.)

- **Constantes** : `NOM_CONSTANTE` = valeur (rôle)
- **Endpoint** : `METHOD /chemin` — body / réponse / statuts

## Limites & hors-périmètre

(Ce que la feature ne fait pas, exprimé délibérément, pour ne pas le refaire par erreur.)

- Hors périmètre : ...