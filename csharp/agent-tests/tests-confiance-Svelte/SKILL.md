Voici le prompt réutilisable, basé sur ce qui a réellement fait gagner la confiance (audit → corrections prioritaires → stabilisation) :
# Mission : Porter la suite de tests à 9.5/10 de confiance

Tu es un ingénieur senior en qualité de tests. Ton objectif est d'auditer puis
de corriger la suite de tests de CE projet pour atteindre une confiance de
9.5/10, sans écrire de nouveaux tests tant que les existants ne sont pas fiables.

## Phase 1 — Audit (produis un rapport chiffré)

Inventorie et compte les anti-patterns suivants :

1. **Tautologies** : tests qui mockent une dépendance puis assertent le mock
   (le test vérifie sa propre configuration, pas le code applicatif).
   Repères : injection manuelle de SQL/CSS/JSON suivie d'une assertion sur
   cette même injection ; mocks chaînables configurés puis assertés.
2. **Faux isolément** : mocks "chainables" là où une vraie DB in-memory
   (SQLite :memory:) est possible et rapide. Les tests d'intégration doivent
   valider de vraies requêtes contre un vrai schéma.
3. **Effets de bord sur disque** : fichiers écrits hors du workspace temp.
4. **E2E vides ou skippés** : stubs `expect(true).toBe(true)`, tests sans
   assertion métier.
5. **E2E sans cleanup** : chaque test E2E doit nettoyer ses données via un
   endpoint de cleanup (dev-only) OU ne créer que des données éphémères.
6. **waitForTimeout** : liste chaque occurrence. MAIS ne les remplace pas
   aveuglément (voir Phase 3).
7. **Tests monolithiques** : un seul test de 150+ lignes qui enchaîne des
   scénarios indépendants → illisible, un échec masque les autres.
8. **Couverture des routes critiques** : auth (login/logout/change-password/
   forgot-password/me), permissions (API sans auth = 401/403), et les
   scénarios multi-utilisateurs (2 contexts navigateur, modification
   concurrente de la même ressource).
9. **Tests qui mutent des données partagées** : un test qui modifie une row
   seedée utilisée par d'autres tests échouera aléatoirement en parallèle
   (workers Playwright partagent la même DB).

Produis un tableau : Métrique | Avant | Cible, et liste chaque fichier
problématique avec la ligne exacte.

## Phase 2 — Corrections par ordre de priorité

1. **Supprimer** les tautologies (ne pas les réparer : elles n'apportent rien).
2. **Migrer** les mocks vers une vraie DB in-memory avec le vrai schéma
   (migrations du projet). Attention : ordre de suppression respectant les
   FK, ou `PRAGMA foreign_keys = OFF`.
3. **Écrire** les intégrations manquantes des routes critiques avec de
   vraies assertions DB (lire la row APRÈS l'appel, pas le retour de mock).
4. **Découper** les monolithes en tests < 50 lignes, un scénario par test,
   chacun avec son cleanup.
5. **Rendre chaque test self-contained** : le test crée SA donnée et la
   nettoie. Jamais de token/ID hardcodé vers une row seedée partagée.
   Récupère les identifiants dynamiques en interceptant la réponse de
   l'API (`page.waitForResponse`) plutôt qu'en hardcodant.
6. **Ajouter** 1-2 E2E critiques : modification concurrente multi-utilisateur
   (`browser.newContext()` ×2) et rejet des requêtes API non authentifiées.
7. **Timeouts réalistes** : les E2E doivent naviguer vers le FUTUR (semaines
   avancées) pour créer des données, jamais "aujourd'hui à heure fixe"
   (conditions temporelles dépendant de l'heure d'exécution = flaky).

## Phase 3 — Stabilisation (règles apprises à la dure)

Ne remplace PAS un waitForTimeout sans comprendre le mécanisme réel :
- Fetch interceptable → `waitForResponse` sur l'URL + méthode précises.
- Form action SvelteKit / POST full-page SANS fetch → il n'y a rien à
  attendre ; un court waitForTimeout est JUSTIFIÉ et documenté.
- Lecture du DOM avant hydratation/transitions → attendre un élément
  concret (heading, toHaveURL) AVANT toute lecture de innerText().

Avant d'écrire un sélecteur, VÉRIFIE le composant réel : testid existant,
role ARIA, placeholder — n'invente jamais `getByRole('link')` pour un
bouton, ni `searchbox` pour un `input type="text"`.
Vérifie aussi les identifiants de test contre le seed réel.

## Phase 4 — Validation finale

1. Suite intégration : 100% verte.
2. Suite E2E complète : 100% verte, 2 fois de suite (anti-flaky).
3. Mets à jour le document d'audit avec le tableau final
   Métrique | Avant | Après et un numéro par correction.

## Interdits

- Ne supprime pas un test sans le justifier dans le rapport.
- Ne marque pas un test `.skip` pour "résoudre" un échec.
- Ne répare pas un test en ajustant son assertion pour coller au bug.
- Si un test révèle un vrai bug applicatif : documente-le, ne le masque pas.
Points clés hérités des échecs de la session 3 : le prompt interdit explicitement le remplacement aveugle de waitForTimeout (ça a cassé settings), les données hardcodées partagées (ça a cassé le test concurrent), et les sélecteurs inventés sans lire le composant (ça a cassé client-technical-notes-crm).