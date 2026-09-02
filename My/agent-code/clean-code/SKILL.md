---
name: clean-code
description: Directives et standards de bonnes pratiques de développement (Clean Code, Règle du Boy Scout, suppression des imports inutilisés, factorisation DRY, apiClient, Cloudflare Workers & D1 Edge Runtime, maintenabilité) pour KalySync.
---

# 🧼 Clean Code, Boy Scout & Edge Runtime (`clean-code`)

Cette compétence définit les exigences de qualité du code, de propreté, de factorisation et de gestion mémoire pour les environnements **Cloudflare Workers & D1 Edge Runtime** dans KalySync.

---

## 📋 Directives et Standards de Code

### 1. La Règle du Boy Scout (Boy Scout Rule)
> *"Toujours laisser le code plus propre que vous ne l'avez trouvé."*

- **Nettoyage des Imports** : **OBLIGATION** de vérifier et supprimer tout import (`import { ... }`) de composant, store, type ou helper qui n'est plus référencé dans le fichier. Évite les avertissements de l'éditeur (lignes en gris pâle) et n'alourdit pas le bundle.
- **Code Mort & Commentaires Obsolètes** : Ne pas laisser de code commenté ou de variables/fonctions inutilisées.
- **Nettoyage des Logs** : Supprimer tous les `console.log` ou `debugger` temporaires avant toute soumission.

---

### 2. Lisibilité et Modularité

- **Responsabilité Unique** : Un composant ou une fonction ne doit faire qu'une seule chose.
- **Respect Strict des Limites de Taille** :
  - Composants `.svelte` < **400 lignes** (**OBLIGATION** : découper immédiatement toute vue complexe en sous-composants ou helpers dès que cette limite est approchée).
  - Stores / Services réactifs < **500 lignes**.
  - Fonctions / Helpers < **50 lignes**.
- **Abstraction DOM & Svelte** : Ne jamais manipuler manuellement le DOM (ex: `document.createElement("a")`) au sein de composants Svelte. Utiliser les bindings et primitives Svelte ou des helpers dédiés et isolés.

---

### 3. Validation et Typage Strict

- **TypeScript strict** sans `any` ni double-cast sauvage (`as unknown as T`).
- Validation systématique via `npm run check`.

---

### 4. Factorisation & Single Source of Truth (DRY)

- **Constantes Uniques (Anti-Magic Numbers)** : **INTERDICTION** de dupliquer des constantes système, tailles de lots (`CHUNK_SIZE`), quotas ou limites de caractères entre le frontend (Composant/Store) et le backend (API). Toute constante partagée doit impérativement résider dans un fichier unique (ex: `$lib/shared/utils/constants.ts`).
- **Détection des Doublons UI** : Dès qu'un sous-élément d'interface (champ de formulaire complexe avec icône/boutons, carte, modal, badge, etc.) est répété ou utilisé dans au moins deux endroits différents de l'application, il **DOIT** être immédiatement extrait sous la forme d'un composant Svelte 5 réutilisable (dans `src/lib/features/.../components/` ou `src/lib/shared/ui/`).
- **Proposition Proactive** : Lors de toute modification ou création de fonctionnalité, identifier et proposer systématiquement la factorisation en composant si une structure identique existe déjà.

---

### 5. Traitement de Données, Fichiers & Performance Mémoire

- **Parsing Robuste de Fichiers (CSV / vCard)** : Ne pas réinventer de parseurs ad-hoc basiques. Gérer impérativement les cas limites (sauts de ligne dans les champs entre guillemets, encodages UTF-8 BOM, etc.).
- **Sanitisation Prudente** : La désinfection contre les injections de formules CSV/Excel ne doit JAMAIS dénaturer les données légitimes d'un utilisateur (ex: préserver des noms commençant par un symbole ou valider explicitement l'intention).
- **Gestion Mémoire & Traitement par Lots** : Ne pas instancier de tableaux géants vides (`new Array(n)`) pour découper des flux ou paquets. Préférer le traitement itératif sur les tableaux d'origine pour soulager le Garbage Collector.
- **Asynchronisme & Boucle d'Événements** : Ne jamais utiliser de `setTimeout(resolve, 0)` sauvage dans des boucles de traitement UI. Utiliser `requestAnimationFrame` ou des découpages asynchrones propres.

---

### 6. Intercepteurs et Communication API Centralisée (`apiClient`)

- **Méthode Privilégiée** : L'utilisation d'un **client API centralisé avec intercepteur (`apiClient.ts`)** est la méthode à privilégier obligatoirement pour toutes les communications HTTP de l'application.
- **Gestion des Erreurs Unifiée** : Ne jamais écrire de requêtes `fetch` brutes éparpillées sans passer par la couche d'API ou `apiClient`. L'intercepteur garantit l'encapsulation systématique des statuts HTTP (`.status`), l'interception automatique des erreurs 5xx (serveur/base de données), la résilience hors-ligne et la déconnexion automatique sur 401.

---

### 7. Bonnes Pratiques Cloudflare Workers & D1 (Edge Runtime)

- **Zero État Global Mutable** : Ne jamais stocker d'état relatif à une requête dans des variables globales au niveau du module (risque de fuite de données inter-requêtes sur l'Edge runtime multi-tenant).
- **Gestion des Promises & Processus Arrière-Plan** : Toute promesse doit être `await`, retournée ou transmise via `platform.context.waitUntil()` pour éviter les *floating promises* abandonnées ou silencieusement coupées.
- **Bindings Natifs vs API REST** : Toujours utiliser les bindings natifs (`env.rdv_db` pour D1, KV, R2) plutôt que de faire des requêtes HTTP distantes vers l'API REST Cloudflare.
- **Requêtes D1 Préparées & Isolation Multi-Tenant** : TOUTES les requêtes SQL doivent utiliser des Prepared Statements (`db.prepare(...).bind(...)`) et inclure **impérativement** la clause `WHERE compagnie_id = ?`.
- **Limites de Mémoire Edge (128 Mo)** : Ne jamais exécuter `await response.text()` ou charger en mémoire des payloads non bornés dans les handlers de serveur. Utiliser le traitement par lots (`chunking`) et respecter `MAX_BULK_BATCH_SIZE`.
- **Sécurité Cryptographique & Comparaison Temporelle** : Utiliser `crypto.randomUUID()` et `crypto.getRandomValues()` (jamais `Math.random()` pour la génération de clés/jetons) et `crypto.subtle.timingSafeEqual` pour vérifier les jetons sensibles sans attaque par canal auxiliaire.
- **Types & Configuration Wrangler** : Utiliser `wrangler types` pour générer automatiquement l'interface `Env` et conserver la configuration centralisée dans `wrangler.jsonc`.

---

## 📋 Checklist de Validation Clean Code

- [ ] Les imports inutilisés ont été retirés du fichier.
- [ ] Aucune constante magique n'est dupliquée entre frontend et backend (`$lib/shared/utils/constants.ts`).
- [ ] Le client API centralisé (`apiClient`) est utilisé pour les requêtes HTTP.
- [ ] Aucune requête SQL D1 ne manque de filtre `WHERE compagnie_id = ?`.
- [ ] Le composant ou la fonction respecte les limites de taille (< 400L composant, < 50L fonction).
