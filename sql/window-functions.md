# SQL — Window Functions

> Calculer des valeurs sur un ensemble de lignes liées à la ligne courante, sans les regrouper.

---

## Concept

La différence fondamentale avec `GROUP BY` : les window functions **gardent toutes les lignes** et ajoutent une colonne calculée. `GROUP BY` collapse les lignes en une par groupe.

```sql
-- GROUP BY : une ligne par utilisateur
SELECT user_id, SUM(amount) FROM orders GROUP BY user_id;

-- Window function : toutes les lignes, avec le total de l'utilisateur sur chaque ligne
SELECT user_id, amount, SUM(amount) OVER (PARTITION BY user_id) AS user_total
FROM orders;
```

---

## Syntaxe

```sql
fonction() OVER (
  PARTITION BY colonne   -- divise en groupes (optionnel — comme un GROUP BY local)
  ORDER BY colonne       -- ordre au sein de chaque groupe
  ROWS BETWEEN ...       -- fenêtre de lignes à inclure (optionnel)
)
```

---

## Fonctions de ranking

Classent les lignes selon un ordre. La différence est dans la gestion des ex-aequo.

```sql
SELECT
  name,
  salary,
  ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,     -- numéro unique, pas d'ex-aequo
  RANK()       OVER (ORDER BY salary DESC) AS rank,        -- ex-aequo partagent le rang, le suivant est sauté
  DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank   -- ex-aequo partagent le rang, pas de saut
FROM employees;
```

| name  | salary | ROW_NUMBER | RANK | DENSE_RANK |
|---|---|---|---|---|
| Alice | 9000   | 1          | 1    | 1          |
| Bob   | 8000   | 2          | 2    | 2          |
| Carol | 8000   | 3          | 2    | 2          |
| Dave  | 7000   | 4          | 4    | 3          |

Bob et Carol ont le même salaire : `RANK` leur donne 2 et saute le 3, `DENSE_RANK` leur donne 2 et continue à 3.

---

## PARTITION BY — remettre le compteur à zéro par groupe

Sans `PARTITION BY`, le ranking s'applique à toute la table. Avec, il repart à 1 pour chaque groupe.

```sql
-- Classer les employés par salaire au sein de chaque département
SELECT
  name,
  department,
  salary,
  RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;
-- Alice peut être #1 dans Marketing et Bob #1 dans Engineering simultanément
```

---

## LAG / LEAD — accéder aux lignes adjacentes

`LAG` lit la valeur de la ligne précédente. `LEAD` lit la valeur de la ligne suivante. Très utile pour comparer avec la période précédente.

```sql
SELECT
  month,
  revenue,
  LAG(revenue)  OVER (ORDER BY month) AS mois_precedent,
  LEAD(revenue) OVER (ORDER BY month) AS mois_suivant,
  revenue - LAG(revenue) OVER (ORDER BY month) AS evolution
FROM monthly_revenue;
```

---

## Totaux cumulés et moyennes glissantes

```sql
-- Total cumulé : la somme grossit à chaque ligne
SELECT
  created_at,
  amount,
  SUM(amount) OVER (ORDER BY created_at) AS total_cumule
FROM orders;

-- Moyenne glissante sur 7 jours
SELECT
  day,
  revenue,
  AVG(revenue) OVER (ORDER BY day ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moy_7j
FROM daily_revenue;
```

`ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` signifie : inclure la ligne courante et les 6 précédentes (7 au total).

---

## FIRST_VALUE / LAST_VALUE

Retourne la première ou dernière valeur dans la fenêtre — utile pour afficher une référence sur chaque ligne.

```sql
-- Salaire le plus élevé du département, affiché sur chaque ligne
SELECT
  name,
  department,
  salary,
  FIRST_VALUE(salary) OVER (PARTITION BY department ORDER BY salary DESC) AS top_salaire
FROM employees;
```

> `LAST_VALUE` nécessite un frame explicite : `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` — sinon la fenêtre s'arrête à la ligne courante et `LAST_VALUE` retourne la valeur courante.

---

## Common pitfalls

- Les window functions ne peuvent pas être utilisées dans `WHERE` ou `HAVING` — les envelopper dans une CTE ou sous-requête
- `RANK()` saute des numéros après les ex-aequo — utiliser `DENSE_RANK()` pour des rangs consécutifs
- Oublier `ORDER BY` dans `OVER()` pour `LAG`/`LEAD` ou les totaux cumulés — le résultat serait non-déterministe
