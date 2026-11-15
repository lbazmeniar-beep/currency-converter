# Currency Converter (Vue 3 + Vite)

Application web qui convertit un montant d'une devise vers une autre à partir de taux fixes (pivot : l'euro).

## Lancer le projet
```bash
npm install
npm run dev
```
Puis ouvrir http://localhost:5173

## Fonctionnalités
- Deux listes de devises (USD, EUR, TND, GBP, JPY, CAD) avec `v-model` + `v-for`
- Clavier numérique (CSS Grid), touche C, un seul point décimal, 10 caractères max
- Conversion arrondie à 2 décimales, bouton d'inversion
- Message d'erreur si le montant vaut 0
- Bonus : conversion automatique (`watch`), taux réels via `fetch` dans `onMounted` avec repli sur les taux fixes, saisie au clavier physique (`keydown`)

> Pour retrouver exactement les résultats des tests (taux fixes), laisser `UTILISER_API_REEL = false` (défaut ; `true` active les taux réels de l’API) dans `CurrencyConverter.vue`.

## Tests
| Montant | Source | Cible | Résultat |
|---|---|---|---|
| 120 | USD | EUR | 102.56 |
| 100 | EUR | TND | 340.00 |
| 50 | GBP | USD | 67.24 |
| 10 | EUR | EUR | 10.00 |
