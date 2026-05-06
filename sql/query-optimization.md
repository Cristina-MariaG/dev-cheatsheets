# SQL — Query Optimization

> Comprendre et corriger les requêtes lentes.

---

## EXPLAIN — lire le plan d'exécution

Avant d'optimiser, il faut comprendre ce que la base fait réellement. `EXPLAIN` montre le plan d'exécution choisi par le query planner.

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 1;
-- Montre le plan prévu, sans exécuter la requête

EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 1;
-- Exécute réellement la requête et compare plan prévu vs réel

EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
-- Ajoute les informations sur les cache hits / disk reads
```

### Nœuds clés à reconnaître

| Nœud | Ce que ça signifie |
|---|---|
| `Seq Scan` | Scan complet de la table — pas d'index utilisé |
| `Index Scan` | Utilise un index, lit ensuite la table pour les colonnes manquantes |
| `Index Only Scan` | Utilise un index couvrant — ne touche pas la table |
| `Nested Loop` | Pour chaque ligne de la table externe, cherche dans l'interne — efficace pour les petits ensembles |
| `Hash Join` | Construit une hash table — bon pour les grands ensembles |
| `Merge Join` | Les deux entrées sont triées — efficace si déjà ordonnées |
| `Sort` | Tri explicite — coûteux sans index sur la colonne |

### Lire les coûts

```
Seq Scan on orders  (cost=0.00..1842.00 rows=50000 width=32)
                           ↑ démarrage   ↑ total
```

Le coût est une unité arbitraire. Ce qui compte c'est la différence relative — `cost=10` vs `cost=10000` est significatif. `rows` est l'estimation du planner — si elle est très éloignée du réel, les stats sont périmées.

---

## Patterns lents et corrections

### 1. Fonction sur une colonne filtrée — détruit l'index

```sql
-- Lent — l'index sur created_at n'est pas utilisé
WHERE DATE(created_at) = '2024-01-01'

-- Rapide — la plage utilise l'index
WHERE created_at >= '2024-01-01' AND created_at < '2024-01-02'
```

### 2. N+1 — une requête par ligne au lieu d'un JOIN

```sql
-- Mauvais : 1 requête pour les orders + 1 par order pour le user
SELECT * FROM orders;
-- puis pour chaque order : SELECT * FROM users WHERE id = ?

-- Bon : une seule requête
SELECT o.*, u.name FROM orders o JOIN users u ON o.user_id = u.id;
```

> Le N+1 est invisible dans le code mais catastrophique en prod — 1000 orders = 1001 requêtes.

### 3. SELECT * — transfère des colonnes inutiles

```sql
-- Mauvais — transfère toutes les colonnes, y compris les blobs
SELECT * FROM products;

-- Bon — uniquement ce dont on a besoin
SELECT id, name, price FROM products;
```

### 4. Sous-requête corrélée vs JOIN

Une sous-requête corrélée est réévaluée pour chaque ligne de la requête externe.

```sql
-- Plus lent — la sous-requête s'exécute une fois par utilisateur
SELECT * FROM users WHERE id IN (SELECT user_id FROM orders WHERE amount > 100);

-- Plus rapide — le JOIN est évalué une seule fois
SELECT DISTINCT u.* FROM users u JOIN orders o ON u.id = o.user_id WHERE o.amount > 100;
```

### 5. Pagination avec OFFSET élevé

```sql
-- Lent — la base scanne et jette 100 000 lignes
SELECT * FROM orders ORDER BY id LIMIT 10 OFFSET 100000;

-- Rapide — keyset pagination : partir de la dernière valeur vue
SELECT * FROM orders WHERE id > :last_seen_id ORDER BY id LIMIT 10;
```

---

## Requêtes de diagnostic (PostgreSQL)

```sql
-- Requêtes les plus lentes (nécessite l'extension pg_stat_statements)
SELECT query, calls, ROUND(mean_exec_time::numeric, 2) AS avg_ms
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Index jamais utilisés — candidats à la suppression
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0;

-- Taille des tables
SELECT tablename, pg_size_pretty(pg_total_relation_size(tablename::regclass)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(tablename::regclass) DESC;
```

---

## Common pitfalls

- **Optimiser avant de mesurer** — toujours `EXPLAIN ANALYZE` d'abord. L'intuition est souvent mauvaise
- **Ajouter un index et supposer qu'il est utilisé** — vérifier avec `EXPLAIN` après création
- **Ignorer les estimations de rows erronées** — un planner qui estime 100 rows alors qu'il y en a 1 million choisira le mauvais plan. Solution : `ANALYZE table`
