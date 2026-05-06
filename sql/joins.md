# SQL — Joins

> Combiner des lignes de plusieurs tables en une seule requête.

---

## Concept

Un join relie deux tables sur une condition — généralement une clé étrangère. Sans join, on ferait plusieurs requêtes séparées et on assemblerait les résultats dans le code.

```sql
-- Sans join : deux requêtes
SELECT * FROM orders WHERE user_id = 1;
SELECT * FROM users WHERE id = 1;

-- Avec join : une seule requête
SELECT u.name, o.amount FROM users u JOIN orders o ON u.id = o.user_id WHERE u.id = 1;
```

---

## INNER JOIN — seulement les correspondances

Retourne uniquement les lignes qui ont une correspondance dans les deux tables. Les lignes sans correspondance sont exclues des deux côtés.

```sql
SELECT u.name, o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;
-- Résultat : uniquement les utilisateurs qui ont au moins une commande
```

---

## LEFT JOIN — tout de la gauche

Retourne toutes les lignes de la table de gauche, et les données correspondantes de la droite. Si aucune correspondance à droite, les colonnes droites valent `NULL`.

```sql
SELECT u.name, o.amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
-- Résultat : tous les utilisateurs, même ceux sans commande (amount = NULL)
```

> C'est le join le plus courant après INNER JOIN. Utile pour "je veux tous les X avec leurs Y s'ils en ont".

---

## RIGHT JOIN — tout de la droite

Inverse du LEFT JOIN — toutes les lignes de la table de droite, correspondances ou NULL de la gauche. Rarement utilisé : on réécrit généralement en LEFT JOIN pour plus de clarté.

```sql
SELECT u.name, o.amount
FROM orders o
RIGHT JOIN users u ON u.id = o.user_id;
-- Équivalent au LEFT JOIN ci-dessus, moins lisible
```

---

## FULL OUTER JOIN — tout des deux côtés

Retourne toutes les lignes des deux tables. `NULL` là où il n'y a pas de correspondance d'un côté ou de l'autre.

```sql
SELECT u.name, o.amount
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;
-- Résultat : tous les utilisateurs + toutes les commandes, avec NULL où il manque une correspondance
```

---

## CROSS JOIN — produit cartésien

Combine chaque ligne de la première table avec chaque ligne de la seconde. Rarement voulu — une table de 100 lignes × une autre de 100 lignes = 10 000 lignes.

```sql
SELECT u.name, p.name
FROM users u
CROSS JOIN products p;
-- Utile pour générer toutes les combinaisons possibles (ex: tailles × couleurs)
```

---

## Visual reference

```
INNER JOIN         LEFT JOIN          FULL OUTER JOIN
  ┌──┬──┐           ┌──┬──┐            ┌──┬──┐
  │  │B │           │A │  │            │A │  │
  │A │B │           │A │B │            │A │B │
  └──┴──┘           │  │B │            │  │B │
  matches only      └──┴──┘            └──┴──┘
                    all left           all rows
```

---

## Self-join — joindre une table sur elle-même

Une table jointe avec elle-même, avec deux alias différents. Utile pour les hiérarchies.

```sql
-- Employés et leur manager (même table employees)
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

---

## Multiple joins — chaîner plusieurs tables

```sql
SELECT u.name, o.amount, p.name AS product
FROM users u
JOIN orders o      ON u.id = o.user_id
JOIN order_items i ON o.id = i.order_id
JOIN products p    ON i.product_id = p.id
WHERE o.status = 'paid';
```

---

## Common pitfalls

**LEFT JOIN + WHERE sur la table de droite** — le filtre dans WHERE annule l'effet du LEFT JOIN et le transforme en INNER JOIN.

```sql
-- Bug : les utilisateurs sans commande sont exclus à cause du WHERE
SELECT * FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.status = 'paid';

-- Fix : déplacer la condition dans le ON
SELECT * FROM users u
LEFT JOIN orders o ON u.id = o.user_id AND o.status = 'paid';
```

**Colonnes ambiguës** — quand deux tables ont une colonne du même nom, toujours qualifier avec l'alias (`u.id`, pas `id`).

**JOIN sans ON** — produit un cartesian product silencieux sur certaines bases. Toujours spécifier la condition de jointure.
