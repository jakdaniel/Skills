---
name: testing-unit
description: Expert en tests unitaires exhaustifs avec Vitest. Utilise ce skill dès qu'il faut écrire, améliorer ou auditer des tests unitaires pour des fonctions pures, composables, stores, validateurs, ou toute logique métier. Le skill insiste sur une couverture maximale : chaque branche logique, chaque cas limite, chaque chemin d'erreur doit être testé. Je ne veux pas juste que les situations critiques ou essentielles soient couvertes, je veux que TOUTES les situations possibles soient couvertes sans exception.
---

## Philosophie Fondamentale

> **"Un test qui ne peut pas échouer ne prouve rien."**

L'objectif n'est pas d'écrire *des* tests, mais d'écrire *tous* les tests nécessaires pour qu'un bug ne puisse pas passer inaperçu. **Chaque situation possible, aussi rare ou triviale soit-elle, doit avoir son propre cas de test.** Chaque fonction testée doit être traitée comme une boîte noire dont on essaie activement de **prouver qu'elle peut échouer**.

### Mentalité de l'Expert QA

Avant d'écrire le moindre test, poser systématiquement ces questions :

1. **Quels sont tous les chemins possibles ?** (branches `if/else`, ternaires, `||`, `&&`, `??`)
2. **Quelles entrées invalides peut-on recevoir ?** (`null`, `undefined`, `""`, `0`, `NaN`, `Infinity`, tableaux vides, objets vides)
3. **Quelles sont les valeurs limites ?** (min, max, min-1, max+1, exactement à la frontière)
4. **Comment ça peut planter ?** (exceptions attendues, erreurs réseau, rejets de promesses)
5. **L'ordre des opérations est-il important ?** (effets de bord, appels multiples, état mutable)
6. **Que retourne la fonction quand tout va bien ?** (type, forme, valeurs exactes)

---

## Configuration Vitest

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      thresholds: {
        statements: 90,
        branches: 85,
        functions: 90,
        lines: 90,
      },
      exclude: ['node_modules/', 'tests/', '*.config.ts'],
    },
  },
});
```

---

## Structure des Tests : Le Protocole d'Exhaustivité

Pour **chaque** fonction ou composable, suivre ce protocole sans exception :

### Étape 1 — Analyser la signature

```typescript
// Exemple de fonction à analyser :
function calculateDiscount(price: number, coupon?: string): number
```

Extraire mentalement :
- Paramètres requis : `price`
- Paramètres optionnels : `coupon`
- Type de retour : `number`
- Effets de bord possibles : aucun (pure) ou à identifier

### Étape 2 — Cartographier les branches

```typescript
// Exemple avec branches complexes :
function getLabel(status: string, count: number): string {
  if (!status) return 'unknown';          // Branche 1
  if (count === 0) return 'empty';        // Branche 2
  if (count === 1) return `${status}`;    // Branche 3
  return `${status} (${count})`;          // Branche 4 (default)
}
```

→ **4 branches = minimum 4 tests**, souvent plus pour les valeurs limites.

### Étape 3 — Lister les cas de test AVANT d'écrire le code

```
✅ Happy paths       : cas nominaux où tout fonctionne
✅ Edge cases        : valeurs aux frontières (0, 1, max, vide)
✅ Sad paths         : entrées invalides, mauvais types
✅ Error paths       : exceptions, rejets, timeouts
✅ Side effects      : appels de mocks, mutations d'état
✅ Async behavior    : résolution, rejet, loading states
✅ Re-entrancy       : appeler la fonction plusieurs fois
```

---

## Template de Test Exhaustif

```typescript
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest';
import { functionToTest } from './moduleToTest';

describe('ModuleName', () => {
  describe('functionToTest', () => {

    // ─── HAPPY PATH ─────────────────────────────────────────────────────
    describe('cas nominaux', () => {
      it('should return expected result with valid standard input', () => {
        expect(functionToTest('valid')).toBe('expected');
      });

      it('should handle all valid input variations', () => {
        // Tester plusieurs valeurs valides représentatives
        const cases = [
          { input: 'a', expected: 'A' },
          { input: 'hello world', expected: 'Hello World' },
        ];
        cases.forEach(({ input, expected }) => {
          expect(functionToTest(input)).toBe(expected);
        });
      });
    });

    // ─── VALEURS LIMITES ─────────────────────────────────────────────────
    describe('valeurs limites', () => {
      it('should handle minimum value', () => { /* ... */ });
      it('should handle maximum value', () => { /* ... */ });
      it('should handle value just below minimum', () => { /* ... */ });
      it('should handle value just above maximum', () => { /* ... */ });
      it('should handle zero', () => { /* ... */ });
      it('should handle single element', () => { /* ... */ });
    });

    // ─── ENTRÉES INVALIDES ───────────────────────────────────────────────
    describe('entrées invalides', () => {
      it('should handle null input', () => { /* ... */ });
      it('should handle undefined input', () => { /* ... */ });
      it('should handle empty string', () => { /* ... */ });
      it('should handle empty array', () => { /* ... */ });
      it('should handle empty object', () => { /* ... */ });
      it('should handle NaN', () => { /* ... */ });
      it('should handle Infinity', () => { /* ... */ });
      it('should handle negative values when only positive expected', () => { /* ... */ });
      it('should handle wrong type (number instead of string)', () => { /* ... */ });
    });

    // ─── CHEMINS D'ERREUR ────────────────────────────────────────────────
    describe('gestion des erreurs', () => {
      it('should throw specific error when X is missing', () => {
        expect(() => functionToTest(null)).toThrow('Expected error message');
      });

      it('should throw correct error type', () => {
        expect(() => functionToTest(null)).toThrow(TypeError);
      });
    });

    // ─── EFFETS DE BORD ──────────────────────────────────────────────────
    describe('effets de bord et interactions', () => {
      it('should call dependency exactly once', () => { /* ... */ });
      it('should call dependency with correct arguments', () => { /* ... */ });
      it('should NOT call dependency when condition is false', () => { /* ... */ });
      it('should handle multiple sequential calls correctly', () => { /* ... */ });
    });
  });
});
```

---

## Patterns d'Exhaustivité par Type

### 🔢 Fonctions Numériques

```typescript
describe('numericFunction', () => {
  // Valeurs normales
  it('should work with positive integer', () => { /* ... */ });
  it('should work with positive float', () => { /* ... */ });
  it('should work with large number', () => { /* ... */ });

  // Cas limites critiques
  it('should handle 0', () => { /* ... */ });
  it('should handle -0', () => { expect(fn(-0)).toBe(fn(0)); });
  it('should handle negative number', () => { /* ... */ });
  it('should handle NaN', () => { expect(fn(NaN)).toBeNaN(); /* ou throw */ });
  it('should handle Infinity', () => { /* ... */ });
  it('should handle -Infinity', () => { /* ... */ });
  it('should handle Number.MAX_SAFE_INTEGER', () => { /* ... */ });
  it('should handle Number.MIN_SAFE_INTEGER', () => { /* ... */ });
  it('should handle floating point precision', () => {
    expect(fn(0.1 + 0.2)).toBeCloseTo(0.3, 10);
  });
});
```

### 📝 Fonctions sur Chaînes

```typescript
describe('stringFunction', () => {
  it('should work with normal string', () => { /* ... */ });
  it('should handle empty string ""', () => { /* ... */ });
  it('should handle single character', () => { /* ... */ });
  it('should handle string with only spaces', () => { /* ... */ });
  it('should handle string with special characters (!@#$%)', () => { /* ... */ });
  it('should handle unicode characters (émojis, accents)', () => { /* ... */ });
  it('should handle very long string', () => {
    const longString = 'a'.repeat(10000);
    expect(() => fn(longString)).not.toThrow();
  });
  it('should handle string with newlines', () => { /* ... */ });
  it('should handle null', () => { /* ... */ });
  it('should handle undefined', () => { /* ... */ });
  it('should handle number passed as string type', () => { /* ... */ });
});
```

### 📦 Fonctions sur Tableaux

```typescript
describe('arrayFunction', () => {
  it('should work with normal array', () => { /* ... */ });
  it('should handle empty array []', () => { /* ... */ });
  it('should handle array with single element', () => { /* ... */ });
  it('should handle array with two elements (boundary)', () => { /* ... */ });
  it('should handle array with duplicate values', () => { /* ... */ });
  it('should handle array with null/undefined elements', () => { /* ... */ });
  it('should handle array with mixed types', () => { /* ... */ });
  it('should NOT mutate the original array', () => {
    const original = [1, 2, 3];
    const copy = [...original];
    fn(original);
    expect(original).toEqual(copy); // Vérifier l'immutabilité
  });
  it('should handle very large array', () => {
    const large = Array.from({ length: 10000 }, (_, i) => i);
    expect(() => fn(large)).not.toThrow();
  });
});
```

### 🗂️ Fonctions sur Objets

```typescript
describe('objectFunction', () => {
  it('should work with complete valid object', () => { /* ... */ });
  it('should handle empty object {}', () => { /* ... */ });
  it('should handle object with missing optional fields', () => { /* ... */ });
  it('should handle object with null values', () => { /* ... */ });
  it('should handle deeply nested object', () => { /* ... */ });
  it('should NOT mutate the input object', () => {
    const input = { a: 1, b: 2 };
    const copy = { ...input };
    fn(input);
    expect(input).toEqual(copy);
  });
  it('should handle object with extra unexpected keys', () => { /* ... */ });
});
```

### ⚡ Fonctions Asynchrones

```typescript
describe('asyncFunction', () => {
  // ─── Résolution ────────────────────────────────────────────────────
  it('should resolve with correct data on success', async () => {
    const result = await fn('valid-input');
    expect(result).toEqual({ id: 1, data: 'expected' });
  });

  it('should resolve with correct shape (all fields present)', async () => {
    const result = await fn('valid-input');
    expect(result).toHaveProperty('id');
    expect(result).toHaveProperty('data');
  });

  // ─── Rejet ─────────────────────────────────────────────────────────
  it('should reject when input is invalid', async () => {
    await expect(fn(null)).rejects.toThrow('Expected error');
  });

  it('should reject with correct error type', async () => {
    await expect(fn(null)).rejects.toBeInstanceOf(TypeError);
  });

  // ─── États intermédiaires ──────────────────────────────────────────
  it('should set loading to true while pending', async () => {
    const promise = fn('input');
    expect(store.isLoading).toBe(true);
    await promise;
    expect(store.isLoading).toBe(false);
  });

  // ─── Timeouts et race conditions ───────────────────────────────────
  it('should handle slow network (timeout simulation)', async () => {
    vi.useFakeTimers();
    const promise = fn('input');
    vi.advanceTimersByTime(5000);
    await expect(promise).rejects.toThrow('Timeout');
    vi.useRealTimers();
  });
});
```

---

## Mocking Avancé — Isoler Chaque Dépendance

### Mock Complet d'un Module

```typescript
vi.mock('./api/userService', () => ({
  fetchUser: vi.fn(),
  updateUser: vi.fn(),
  deleteUser: vi.fn(),
}));

import { fetchUser } from './api/userService';

describe('withMockedApi', () => {
  beforeEach(() => {
    vi.clearAllMocks(); // Toujours reset entre les tests !
  });

  it('should call fetchUser with correct id', async () => {
    (fetchUser as vi.Mock).mockResolvedValue({ id: 1, name: 'John' });
    await myFunction(1);
    expect(fetchUser).toHaveBeenCalledWith(1);
    expect(fetchUser).toHaveBeenCalledTimes(1);
  });

  it('should handle fetchUser rejection gracefully', async () => {
    (fetchUser as vi.Mock).mockRejectedValue(new Error('Network error'));
    await expect(myFunction(1)).rejects.toThrow('Network error');
  });

  it('should NOT call fetchUser when cache is warm', async () => {
    setCache(1, { id: 1, name: 'Cached' });
    await myFunction(1);
    expect(fetchUser).not.toHaveBeenCalled();
  });
});
```

### Mock de Date et Timers

```typescript
describe('date-dependent function', () => {
  beforeEach(() => {
    vi.useFakeTimers();
    vi.setSystemTime(new Date('2024-06-15T12:00:00Z'));
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it('should use current date for calculation', () => {
    expect(getAge('1990-06-15')).toBe(34); // Âge exact au 15 juin 2024
  });

  it('should consider day boundary correctly', () => {
    vi.setSystemTime(new Date('2024-06-14T23:59:59Z')); // Veille anniversaire
    expect(getAge('1990-06-15')).toBe(33);
  });
});
```

### Spy sur Méthodes

```typescript
it('should call console.warn when value is deprecated', () => {
  const warnSpy = vi.spyOn(console, 'warn').mockImplementation(() => {});
  deprecatedFunction();
  expect(warnSpy).toHaveBeenCalledWith(expect.stringContaining('deprecated'));
  warnSpy.mockRestore();
});
```

---

## Tests de Composables Vue — Protocole Complet

```typescript
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  let counter: ReturnType<typeof useCounter>;

  beforeEach(() => {
    counter = useCounter(); // Réinitialiser avant chaque test
  });

  // ─── État initial ──────────────────────────────────────────────────
  describe('initial state', () => {
    it('should initialize with default value of 0', () => {
      expect(counter.count.value).toBe(0);
    });

    it('should initialize with custom initial value', () => {
      const custom = useCounter(10);
      expect(custom.count.value).toBe(10);
    });

    it('should initialize with negative value when provided', () => {
      const neg = useCounter(-5);
      expect(neg.count.value).toBe(-5);
    });
  });

  // ─── Actions ───────────────────────────────────────────────────────
  describe('increment', () => {
    it('should increment by 1', () => {
      counter.increment();
      expect(counter.count.value).toBe(1);
    });

    it('should increment multiple times correctly', () => {
      counter.increment();
      counter.increment();
      counter.increment();
      expect(counter.count.value).toBe(3);
    });

    it('should not exceed max if defined', () => {
      const bounded = useCounter(0, { max: 3 });
      bounded.increment();
      bounded.increment();
      bounded.increment();
      bounded.increment(); // 4ème appel sur max=3
      expect(bounded.count.value).toBe(3);
    });
  });

  // ─── Computed / Dérivés ────────────────────────────────────────────
  describe('computed values', () => {
    it('isPositive should be false at 0', () => {
      expect(counter.isPositive.value).toBe(false);
    });

    it('isPositive should be true after increment', () => {
      counter.increment();
      expect(counter.isPositive.value).toBe(true);
    });

    it('isPositive should be false when negative', () => {
      const neg = useCounter(-1);
      expect(neg.isPositive.value).toBe(false);
    });
  });

  // ─── Isolation entre instances ─────────────────────────────────────
  describe('instance isolation', () => {
    it('should not share state between two instances', () => {
      const a = useCounter();
      const b = useCounter();
      a.increment();
      expect(b.count.value).toBe(0); // b ne doit pas être affecté
    });
  });
});
```

---

## Tests de Stores Pinia — Protocole Complet

```typescript
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { setActivePinia, createPinia } from 'pinia';
import { useAuthStore } from './authStore';

describe('authStore', () => {
  beforeEach(() => {
    setActivePinia(createPinia()); // Pinia fraîche à chaque test
    vi.clearAllMocks();
  });

  // ─── État initial ──────────────────────────────────────────────────
  describe('initial state', () => {
    it('should have null user', () => {
      expect(useAuthStore().user).toBeNull();
    });

    it('should have isAuthenticated = false', () => {
      expect(useAuthStore().isAuthenticated).toBe(false);
    });

    it('should have no token', () => {
      expect(useAuthStore().token).toBeUndefined();
    });
  });

  // ─── Actions ───────────────────────────────────────────────────────
  describe('login action', () => {
    it('should set user on successful login', async () => {
      const store = useAuthStore();
      await store.login('user@test.com', 'password123');
      expect(store.user).not.toBeNull();
    });

    it('should set isAuthenticated = true after login', async () => {
      const store = useAuthStore();
      await store.login('user@test.com', 'password123');
      expect(store.isAuthenticated).toBe(true);
    });

    it('should throw on wrong credentials', async () => {
      const store = useAuthStore();
      await expect(store.login('bad@test.com', 'wrong')).rejects.toThrow();
    });

    it('should NOT set user on failed login', async () => {
      const store = useAuthStore();
      try { await store.login('bad@test.com', 'wrong'); } catch {}
      expect(store.user).toBeNull();
    });
  });

  // ─── Getters ───────────────────────────────────────────────────────
  describe('getters', () => {
    it('fullName should return empty string when no user', () => {
      expect(useAuthStore().fullName).toBe('');
    });

    it('fullName should combine firstName and lastName', async () => {
      const store = useAuthStore();
      store.user = { firstName: 'John', lastName: 'Doe' };
      expect(store.fullName).toBe('John Doe');
    });
  });
});
```

---

## Checklist d'Exhaustivité — À Valider Pour Chaque Fonction

Avant de considérer une fonction comme "bien testée", cocher :

```
COUVERTURE DES BRANCHES
[ ] Chaque if/else a un test pour true ET pour false
[ ] Chaque ternaire (? :) est testé dans les deux états
[ ] Chaque || et && est testé avec toutes les combinaisons pertinentes
[ ] Chaque return anticipé (early return) a son propre test

ENTRÉES INVALIDES
[ ] null testé
[ ] undefined testé
[ ] Chaîne vide "" testée (si string attendu)
[ ] Tableau vide [] testé (si array attendu)
[ ] Objet vide {} testé (si object attendu)
[ ] 0 testé (si number attendu)
[ ] NaN testé (si number attendu)
[ ] Type incorrect testé (ex: string reçu quand number attendu)

VALEURS LIMITES
[ ] Valeur minimale autorisée testée
[ ] Valeur maximale autorisée testée
[ ] Valeur min - 1 testée (out of bounds)
[ ] Valeur max + 1 testée (out of bounds)
[ ] Tableau/string d'un seul élément testé
[ ] Très grande valeur testée (performance)

EFFETS DE BORD ET MOCKS
[ ] Chaque dépendance mockée est vérifiée (toHaveBeenCalledWith)
[ ] Cas où la dépendance échoue est testé
[ ] Cas où la dépendance n'est PAS appelée est vérifié
[ ] Immutabilité des inputs vérifiée (si applicable)
[ ] Appels multiples de la fonction testés

ASYNC (si applicable)
[ ] Cas de résolution testé
[ ] Cas de rejet testé
[ ] État loading/pending vérifié
[ ] Comportement en cas de timeout testé
```

---

## Commandes

```bash
# Tous les tests
npm test

# Mode watch (développement)
npm test -- --watch

# Couverture complète
npm test -- --coverage

# Un fichier spécifique
npm test -- src/utils/formatters.test.ts

# Un test par nom
npm test -- -t "should handle empty string"

# Interface graphique
npm test -- --ui
```

---

## Objectifs de Couverture (Non Négociables)

| Métrique    | Minimum | Cible Idéale |
|-------------|---------|--------------|
| Statements  | 90%     | 95%+         |
| Branches    | 85%     | 90%+         |
| Functions   | 90%     | 100%         |
| Lines       | 90%     | 95%+         |

> ⚠️ Un score de couverture élevé ne garantit pas des tests de qualité — mais un score bas garantit des zones non testées. Les deux sont nécessaires.

---

**Note** : Pour les tests d'intégration (interactions entre composants, appels API réels), voir `testing.integration.md`. Pour les parcours utilisateur complets, voir `testing.e2e.md`.