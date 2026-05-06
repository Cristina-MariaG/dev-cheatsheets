# SQL — Basics

> Core statements for querying and modifying data.

---

## SELECT

`SELECT` récupère des colonnes d'une table. C'est toujours la première instruction d'une requête de lecture.

```sql
SELECT *                              FROM users;         -- toutes les colonnes
SELECT id, name, email                FROM users;         -- colonnes spécifiques
SELECT DISTINCT country               FROM users;         -- supprime les doublons
SELECT name, age * 12 AS age_months   FROM users;         -- colonne calculée avec alias
```

> `SELECT *` est pratique en exploration mais coûteux en production — il transfère toutes les colonnes même celles inutilisées. Toujours nommer les colonnes explicitement dans du code.

---

## WHERE — filtrer les lignes

`WHERE` filtre les lignes avant de les retourner. Seules les lignes qui satisfont la condition sont incluses.

```sql
SELECT * FROM orders WHERE status = 'active';
SELECT * FROM orders WHERE amount > 100 AND status = 'paid';   -- les deux conditions
SELECT * FROM orders WHERE amount > 100 OR status = 'free';    -- l'une ou l'autre
SELECT * FROM orders WHERE status IN ('pending', 'active');    -- équivaut à plusieurs OR
SELECT * FROM orders WHERE status NOT IN ('cancelled');
SELECT * FROM users  WHERE name LIKE 'A%';                     -- commence par A
SELECT * FROM users  WHERE name LIKE '%alice%';                -- contient "alice"
SELECT * FROM users  WHERE name ILIKE 'a%';                    -- insensible à la casse (PostgreSQL)
SELECT * FROM users  WHERE email IS NULL;                      -- valeur absente
SELECT * FROM users  WHERE email IS NOT NULL;                  -- valeur présente
SELECT * FROM users  WHERE age BETWEEN 18 AND 65;             -- inclusif des deux bornes
```

> `NULL` ne peut pas être comparé avec `=`. `WHERE email = NULL` ne retourne jamais rien — toujours utiliser `IS NULL`.

---

## ORDER BY — trier les résultats

`ORDER BY` trie les lignes retournées. Sans lui, l'ordre est indéfini et peut changer d'une exécution à l'autre.

```sql
SELECT * FROM users ORDER BY created_at DESC;                  -- plus récent en premier
SELECT * FROM users ORDER BY last_name ASC, first_name ASC;   -- tri sur plusieurs colonnes
```

---

## LIMIT et OFFSET — paginer

`LIMIT` restreint le nombre de lignes retournées. `OFFSET` saute un certain nombre de lignes avant de commencer.

```sql
SELECT * FROM users LIMIT 10;                 -- les 10 premiers
SELECT * FROM users LIMIT 10 OFFSET 20;      -- 10 lignes à partir de la 21e (page 3)
```

> Toujours combiner `LIMIT` avec `ORDER BY` — sans tri, les lignes retournées sont arbitraires.

---

## INSERT — insérer des données

`INSERT INTO` ajoute une ou plusieurs lignes dans une table.

```sql
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');

-- Insérer plusieurs lignes en une seule requête (plus efficace)
INSERT INTO users (name, email) VALUES
  ('Alice', 'alice@example.com'),
  ('Bob',   'bob@example.com');

-- Insérer le résultat d'une requête
INSERT INTO archive SELECT * FROM orders WHERE status = 'closed';
```

---

## UPDATE — modifier des données

`UPDATE` modifie des lignes existantes. `SET` définit les nouvelles valeurs, `WHERE` cible les lignes à modifier.

```sql
UPDATE users SET status = 'inactive' WHERE last_login < '2024-01-01';
UPDATE users SET name = 'Alice', email = 'alice@new.com' WHERE id = 1;  -- plusieurs colonnes
```

> **Toujours inclure un WHERE.** `UPDATE users SET status = 'inactive'` sans condition modifie toutes les lignes de la table. Tester avec un `SELECT` sur le même `WHERE` avant d'exécuter.

---

## DELETE — supprimer des données

`DELETE FROM` supprime des lignes. Sans `WHERE`, toute la table est vidée.

```sql
DELETE FROM users WHERE id = 1;
DELETE FROM sessions WHERE expires_at < NOW();    -- NOW() = timestamp actuel
```

> Même règle qu'UPDATE — toujours un `WHERE`. Pour vider complètement une table, `TRUNCATE table` est plus rapide que `DELETE FROM table` sans condition.

---

## Common pitfalls

- `NULL` — ne jamais comparer avec `=` ou `!=`. Utiliser `IS NULL` / `IS NOT NULL`
- `LIKE '%term%'` — le `%` au début empêche l'utilisation d'un index (full scan)
- `LIMIT` sans `ORDER BY` — les résultats sont non-déterministes, ils peuvent changer
- `UPDATE` / `DELETE` sans `WHERE` — irréversible sans backup ou transaction
