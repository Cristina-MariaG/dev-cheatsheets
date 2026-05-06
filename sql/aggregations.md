# SQL — Aggregations

> Résumer des données en regroupant des lignes et en calculant des statistiques.

---

## Concept

Les fonctions d'agrégation calculent une valeur à partir d'un ensemble de lignes. Sans `GROUP BY`, elles s'appliquent à toute la table et retournent une seule ligne.

```sql
SELECT COUNT(*), SUM(amount), AVG(amount) FROM orders;
-- Retourne une seule ligne avec les totaux pour toute la table
```

---

## Fonctions d'agrégation

```sql
SELECT COUNT(*)                FROM orders;   -- nombre total de lignes
SELECT COUNT(email)            FROM users;    -- lignes où email n'est pas NULL
SELECT COUNT(DISTINCT country) FROM users;    -- nombre de pays uniques
SELECT SUM(amount)             FROM orders;   -- somme
SELECT AVG(amount)             FROM orders;   -- moyenne
SELECT MIN(created_at)         FROM orders;   -- valeur minimale
SELECT MAX(amount)             FROM orders;   -- valeur maximale
```

> `COUNT(*)` compte toutes les lignes y compris celles avec des NULLs. `COUNT(colonne)` ignore les NULLs — le résultat peut être différent.

---

## GROUP BY — regrouper les lignes

`GROUP BY` divise les lignes en groupes selon une ou plusieurs colonnes, puis applique la fonction d'agrégation à chaque groupe séparément.

```sql
-- Nombre de commandes par utilisateur
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id;
```

Sans `GROUP BY` : une seule ligne avec le total global.
Avec `GROUP BY user_id` : une ligne par utilisateur avec son total.

```sql
-- Chiffre d'affaires par pays et par mois
SELECT
  country,
  DATE_TRUNC('month', created_at) AS month,
  SUM(amount) AS revenue
FROM orders
GROUP BY country, DATE_TRUNC('month', created_at)
ORDER BY month DESC;
```

> Règle : toute colonne dans `SELECT` doit être soit dans `GROUP BY`, soit dans une fonction d'agrégation. Sinon la base de données ne sait pas quelle valeur afficher pour le groupe.

---

## HAVING — filtrer les groupes

`WHERE` filtre les lignes avant le groupement. `HAVING` filtre les groupes après l'agrégation — il peut donc utiliser les résultats des fonctions d'agrégation.

```sql
-- Utilisateurs avec plus de 5 commandes
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;

-- Pays dont le panier moyen dépasse 100
SELECT country, AVG(amount) AS avg_order
FROM orders
GROUP BY country
HAVING AVG(amount) > 100
ORDER BY avg_order DESC;
```

> Utiliser `WHERE` pour filtrer les lignes brutes (avant GROUP BY) et `HAVING` pour filtrer les résultats agrégés. `WHERE` est plus efficace car il réduit les données avant le groupement.

---

## Ordre d'exécution d'une requête SQL

SQL s'exécute dans cet ordre, peu importe l'ordre d'écriture :

```
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

Conséquences pratiques :
- On ne peut pas utiliser un alias défini dans `SELECT` dans un `WHERE` ou `HAVING` — l'alias n'existe pas encore à ce moment
- `WHERE` ne peut pas filtrer sur un résultat agrégé — utiliser `HAVING`

---

## Common pitfalls

- Utiliser `WHERE` pour filtrer un agrégat — `WHERE COUNT(*) > 5` est invalide, `HAVING COUNT(*) > 5` est correct
- Sélectionner une colonne qui n'est pas dans `GROUP BY` — la plupart des bases rejettent ça (sauf MySQL en mode non-strict)
- `AVG` sur des entiers peut tronquer silencieusement en fonction de la base — caster si besoin : `AVG(amount::numeric)`
