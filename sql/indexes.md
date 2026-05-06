# SQL — Indexes

> Structures de données qui accélèrent les requêtes en évitant de scanner toute la table.

---

## Concept

Sans index, la base de données lit chaque ligne pour trouver les correspondances — c'est un **full table scan**. Sur une table de 10 millions de lignes, c'est lent.

Un index est une structure séparée (généralement un B-tree) qui trie les valeurs d'une colonne et stocke des pointeurs vers les lignes. La base peut ainsi trouver une valeur en O(log n) au lieu de O(n).

**Trade-off** : les index accélèrent les lectures (`SELECT`) mais ralentissent les écritures (`INSERT`, `UPDATE`, `DELETE`) car ils doivent être mis à jour à chaque modification. Ils occupent aussi de l'espace disque.

---

## Créer et supprimer des index

```sql
CREATE INDEX idx_users_email ON users (email);                       -- colonne simple
CREATE INDEX idx_orders_user_date ON orders (user_id, created_at);  -- index composite
CREATE UNIQUE INDEX idx_users_email ON users (email);                -- garantit l'unicité
CREATE INDEX idx_users_email_lower ON users (LOWER(email));          -- index sur expression
DROP INDEX idx_users_email;
```

---

## Types d'index (PostgreSQL)

| Type | Opérateurs supportés | Use case |
|---|---|---|
| `B-tree` | `=`, `<`, `>`, `<=`, `>=`, `BETWEEN`, `LIKE 'abc%'` | Default — la quasi-totalité des cas |
| `Hash` | `=` uniquement | Recherches d'égalité pure, légèrement plus rapide que B-tree |
| `GIN` | `@>`, `&&`, `@@` | Recherche full-text, tableaux, JSONB |
| `GiST` | Types géométriques, distances | Coordonnées GPS, formes géographiques |
| `BRIN` | Plages sur données ordonnées | Très grandes tables avec données naturellement séquentielles (timestamps, IDs auto-incrémentés) |

> Dans 95% des cas, un B-tree est le bon choix.

---

## Index composite — l'ordre des colonnes compte

Un index composite couvre plusieurs colonnes. Il peut servir les requêtes qui filtrent sur le **préfixe gauche** des colonnes.

```sql
CREATE INDEX idx_orders_user_status ON orders (user_id, status);
```

Cet index est utilisé pour :
- `WHERE user_id = 1` ✅
- `WHERE user_id = 1 AND status = 'paid'` ✅

Mais **pas** pour :
- `WHERE status = 'paid'` ❌ — la colonne de tête est absente

> Règle : mettre en premier la colonne la plus sélective, ou celle utilisée en égalité avant celles utilisées en plage.

---

## Index couvrant (covering index)

Un index qui contient toutes les colonnes nécessaires à une requête. La base peut répondre **sans lire la table** — on parle d'*index only scan*.

```sql
CREATE INDEX idx_orders_covering ON orders (user_id, status, amount);

-- Cette requête est servie entièrement depuis l'index
SELECT status, amount FROM orders WHERE user_id = 1;
```

---

## Quand l'index n'est PAS utilisé

```sql
-- Fonction sur la colonne indexée — l'index sur email est ignoré
WHERE LOWER(email) = 'alice@example.com'
-- Fix : créer un index sur l'expression
CREATE INDEX ON users (LOWER(email));

-- Wildcard en début de LIKE — impossible à indexer en B-tree
WHERE name LIKE '%alice%'
-- Fix : utiliser la recherche full-text (GIN + tsvector)

-- Condition OR sur des colonnes différentes — peut ignorer les index
WHERE email = 'x' OR phone = 'y'
-- Fix : réécrire en UNION
SELECT * FROM users WHERE email = 'x'
UNION
SELECT * FROM users WHERE phone = 'y';

-- Cast de type implicite — l'index sur id (integer) n'est pas utilisé si comparé à un string
WHERE id = '123'       -- '123' est un string, id est un integer
-- Fix : WHERE id = 123
```

---

## Common pitfalls

- **Over-indexing** — chaque index ralentit les écritures. Indexer les colonnes utilisées dans `WHERE`, `JOIN`, et `ORDER BY`, pas toutes les colonnes
- **Index inutilisés** — vérifier avec `pg_stat_user_indexes` et supprimer ceux avec `idx_scan = 0`
- **Pas d'index sur les clés étrangères** — les joins sur les foreign keys sont lents sans index
