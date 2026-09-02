---
name: security
description: Skill expert à utiliser lors du développement Svelte ou SvelteKit afin de prévenir les vulnérabilités de sécurité côté client et serveur. À déclencher pour audit de sécurité, owasp, revue de code, conception d'authentification, protection SSR, gestion des données sensibles et déploiement sécurisé.
---

# Securite Skill

Ce skill définit les **meilleures pratiques de sécurité modernes** pour les applications **Svelte 5** et **SvelteKit 2**, couvrant :
- Sécurité client et serveur
- SSR et Edge runtimes
- Authentification et autorisation
- Protection contre XSS, CSRF, injections, fuites de données
- Bonnes pratiques cloud / déploiement

Objectif : **réduire la surface d'attaque** sans sacrifier la performance ou la DX.

---

## Quand utiliser ce skill
- Utiliser ce skill lorsque l'utilisateur demande une nouvelle fonctionnalitée
- Audit de sécurité Svelte / SvelteKit
- Revue de code avant mise en production
- Implémentation d'authentification / autorisation
- Manipulation de données sensibles
- Déploiement Cloudflare / Vercel / Node
- Intégration API externes ou IA

---

## 📚 Consultation Ciblée des Fichiers de Référence (`data/`)

À **chaque fois** que ce skill est activé ou utilisé, l'agent doit effectuer une **lecture ciblée et économique** (pour préserver le budget de tokens) :
1. **Consulter l'index ci-dessous** pour identifier la catégorie liée au besoin.
2. **Lire uniquement (via `view_file`)** les **1 à 3 fichiers Markdown** pertinents situés dans `.agent/skills/Security/data/cheatsheetseries.owasp.org/`, **SANS JAMAIS tout charger**.
3. **Appliquer les préconisations** pour la revue, l'audit ou l'implémentation.

### 🗂️ Index Thématique des Cheatsheets (`data/cheatsheetseries.owasp.org/`)

- **🔑 Authentification, Mots de passe & Sessions** :
  - `Authentication_Cheat_Sheet.md`
  - `Session_Management_Cheat_Sheet.md`
  - `Multifactor_Authentication_Cheat_Sheet.md`
  - `Forgot_Password_Cheat_Sheet.md`
  - `Credential_Stuffing_Prevention_Cheat_Sheet.md`
  - `Password_Storage_Cheat_Sheet.md`

- **🛡️ Autorisation, Droits & Multi-Tenant** :
  - `Authorization_Cheat_Sheet.md`
  - `Access_Control_Cheat_Sheet.md`
  - `Multi_Tenant_Security_Cheat_Sheet.md`
  - `Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.md`

- **💉 Injections & Base de Données** :
  - `SQL_Injection_Prevention_Cheat_Sheet.md`
  - `Query_Parameterization_Cheat_Sheet.md`
  - `Injection_Prevention_Cheat_Sheet.md`
  - `Database_Security_Cheat_Sheet.md`
  - `OS_Command_Injection_Defense_Cheat_Sheet.md`

- **🌐 UI, Frontend & XSS** :
  - `Cross_Site_Scripting_Prevention_Cheat_Sheet.md`
  - `DOM_based_XSS_Prevention_Cheat_Sheet.md`
  - `Content_Security_Policy_Cheat_Sheet.md`
  - `HTML5_Security_Cheat_Sheet.md`
  - `Securing_Cascading_Style_Sheets_Cheat_Sheet.md`

- **🔒 En-têtes, Cookies & CSRF** :
  - `Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.md`
  - `HTTP_Headers_Cheat_Sheet.md`
  - `Clickjacking_Defense_Cheat_Sheet.md`
  - `Cookie_Theft_Mitigation_Cheat_Sheet.md`

- **🔌 API, JWT, OAuth & Microservices** :
  - `REST_Security_Cheat_Sheet.md`
  - `JSON_Web_Token_Cheat_Sheet.md`
  - `OAuth2_Cheat_Sheet.md`
  - `GraphQL_Cheat_Sheet.md`
  - `Microservices_Security_Cheat_Sheet.md`

- **📥 Validation des Entrées & Gestion des Fichiers** :
  - `Input_Validation_Cheat_Sheet.md`
  - `File_Upload_Cheat_Sheet.md`
  - `Deserialization_Cheat_Sheet.md`
  - `Error_Handling_Cheat_Sheet.md`

- **⚙️ Node.js, Cloud & Dépendances** :
  - `Nodejs_Security_Cheat_Sheet.md`
  - `NPM_Security_Cheat_Sheet.md`
  - `Docker_Security_Cheat_Sheet.md`
  - `Secure_Cloud_Architecture_Cheat_Sheet.md`
  - `Vulnerable_Dependency_Management_Cheat_Sheet.md`

- **🤖 IA, Prompts & Agents AI** :
  - `AI_Agent_Security_Cheat_Sheet.md`
  - `LLM_Prompt_Injection_Prevention_Cheat_Sheet.md`
  - `MCP_Security_Cheat_Sheet.md`
  - `Secure_Coding_with_AI_Cheat_Sheet.md`
  - `RAG_Security_Cheat_Sheet.md`

---

## Doctrine de Sécurité (CRITIQUE)

### Principes fondamentaux

- **Tout input est malveillant par défaut**
- **Le client n'est jamais digne de confiance**
- **Le serveur est l'unique source de vérité**
- **Aucune clé secrète côté client**
- **Fail fast, fail closed**

---

## Hiérarchie des règles

### 🔴 OBLIGATOIRE
- ALWAYS check OWASP top 15 vulnerabilities.
- Consultation ciblée obligatoire des cheatsheets Markdown pertinentes dans `data/` (`.agent/skills/Security/data/cheatsheetseries.owasp.org/`).
- Aucune donnée sensible dans le client
- Validation serveur systématique
- Protection XSS active
- Accès navigateur interdit en SSR
- Auth côté serveur uniquement
- ALWAYS use npm audit fix  

### 🟠 FORTEMENT RECOMMANDÉ
- Cookies `httpOnly` + `secure`
- CSP stricte
- Rate limiting
- Logs d'erreurs centralisés

### 🟢 OPTIONNEL
- Edge runtime
- Honeypots formulaires
- Détection d'abus avancée

---

## 1. XSS (Cross-Site Scripting)

### ❌ Anti-pattern critique

```svelte
{@html contenuUtilisateur}
```

### ✅ Alternative sécurisée

**Échapper le HTML**

**Utiliser un parser sécurisé (DOMPurify côté serveur)**

```javascript
import DOMPurify from 'isomorphic-dompurify';

const safeHtml = DOMPurify.sanitize(input);
```

### Règles
- `{@html}` = interdit par défaut
- Jamais avec du contenu utilisateur brut
- Jamais côté client pour données non maîtrisées

---

## 2. Gestion des entrées utilisateur

### Validation serveur obligatoire

```javascript
import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
  age: z.number().min(18)
});

schema.parse(data);
```

### Bonnes pratiques
- Ne jamais faire confiance à la validation client
- Valider avant toute action métier
- Messages d'erreur génériques

---

## 3. SSR & Environnement

### ❌ Erreur critique

```svelte
<script>
  const token = localStorage.getItem('token');
</script>
```

### ✅ Sécurisé

```javascript
import { cookies } from '@sveltejs/kit';

export async function load({ cookies }) {
  const token = cookies.get('session');
}
```

### Règles
- Aucun `window`, `document`, `localStorage` en SSR
- Vérifier `browser` systématiquement

---

## 4. Authentification

### Principes
- Auth server-side uniquement
- Sessions via cookies sécurisés
- JWT uniquement si nécessaire

### Cookies sécurisés

```javascript
cookies.set('session', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
  path: '/'
});
```

### ❌ À éviter
- Tokens en `localStorage`
- Auth logique côté client
- Vérification des rôles côté client

---

## 5. Autorisation (RBAC / ABAC)

### Toujours côté serveur

```javascript
export async function load({ locals }) {
  if (!locals.user || locals.user.role !== 'admin') {
    throw error(403);
  }
}
```

### Bonnes pratiques
- Vérifier l'autorisation dans chaque endpoint
- Jamais se fier au client
- Centraliser les règles d'accès

---

## 6. CSRF

### Protection recommandée
- Cookies `sameSite=strict`
- Tokens CSRF pour actions critiques

```javascript
if (request.headers.get('origin') !== expectedOrigin) {
  throw error(403);
}
```

### Formulaires
- Préférer `enhance()` avec POST
- Éviter GET pour actions mutables

---

## 7. API & Routes serveur

### Sécuriser les routes

```javascript
export async function POST({ request, locals }) {
  if (!locals.user) throw error(401);
}
```

### Règles
- Toujours vérifier auth + autorisation
- Limiter les méthodes HTTP
- Réponses minimales (pas d'info inutile)

---

## 8. Données sensibles

### ❌ Interdit
- Clés API dans le client
- Secrets dans `+page.svelte`
- Logs contenant tokens

### ✅ Bonnes pratiques
- Variables d'environnement serveur
- Rotation des clés
- Scopes minimaux

---

## 9. Sécurité des dépendances

### Règles
- Auditer régulièrement (`npm audit`)
- Éviter les libs non maintenues
- Lockfile obligatoire

### ❌ Risque
- Libs UI manipulant le DOM
- Sanitization côté client uniquement

---

## 10. CSP & Headers HTTP

### CSP recommandée

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self';
  style-src 'self';
  img-src 'self' data:;
```

### Autres headers
- `X-Frame-Options: DENY`
- `X-Content-Type-Options: nosniff`
- `Referrer-Policy: strict-origin`

---

## 11. Edge & Cloud

### Bonnes pratiques
- Pas de secrets dans le bundle
- Timeout strict
- Logs côté serveur uniquement

### Edge runtime
- Idéal pour auth légère
- Limiter la logique métier lourde

---

## 12. IA & Sécurité

### Règles critiques
- Appels IA **server-only**
- Nettoyer prompts utilisateur
- Ne jamais loguer prompts sensibles
- Limiter la taille des entrées

---

## Anti-patterns critiques

- ❌ `{@html}` avec input utilisateur
- ❌ Tokens dans `localStorage`
- ❌ Validation client uniquement
- ❌ Logique de sécurité côté client
- ❌ Secrets exposés au build

---

## Checklist sécurité finale

- [ ] Aucune clé côté client
- [ ] Validation serveur complète
- [ ] Cookies `httpOnly` + `secure`
- [ ] CSP active
- [ ] Auth côté serveur
- [ ] Logs nettoyés
- [ ] SSR safe
- [ ] Dépendances auditées


**Données Sources :** Situées dans ``.agent/skills/Security/data/` (Markdown).