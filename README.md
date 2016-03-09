# SQL Conventions

-- Proposition inspiré d'une boîte Vendéenne

* PostgreSQL is your friend, use it

* Pas d'abbréviations des mots sauf pour des expressions bien connues et longue (e.g. "i18n")
* Pas de mots-clés réservé (par exemple `user` sur PGSQL)


* Table names in PascalCase, singular (ex: `SupplierProduct`, `Product`, `Operator`, `Role`, etc.)
  - Tables de liaison avec `_`, ex: `SupplierOrder_Product`, `Operator_Role`
  - par opposition, une table sans liaison `OperatorRole`

* Column names in camelCase, singular (ex: `name`, `motivation`, `count`)
  - foreign keys: `idProduct`, `idSupplierProduct`
  - dates: `dateCreated`, `dateDeleted`, `dateUpdated` (oui, ça choque au début)

* Limiter l'usage des alias au maximum (mais cela venait du fait que nous utilisions beaucoup les CTEs)

Exemple concret de requête :

```sql
SELECT
  *,
  SUM("SupplierOrder_Product"."qty" * "SupplierOrder_Product"."price")
    OVER (PARTITION BY "SupplierOrder"."id")                          AS "totalSupplierOrder"
FROM "SupplierOrder"
  INNER JOIN "SupplierOrder_Product" ON "SupplierOrder_Product"."idSupplierOrder" = "SupplierOrder"."id"
```

* utiliser des UUID en type de PK & FK (dépend de la volumétrie ?)
* chaque table a les champs `dateCreated`, `dateUpdated` / `dateDeleted`

* utiliser une lib de data-mapping (anorm/slick) mais pas d'ORM
* utiliser BNCF (au dessus de la 3NF) (cf normal form)

* Always set column to NOT NULL by default, never use NULL (enfin, c'était notre règle... dépend des use case)
  - when using a foreign key in a table,

* Never use ON DELETE CASCADE, set `deletedAt` to `NOW()`
  - dépend du use case et de l'importance des données

* Usage of USING (jamais utilisé car nos conventions ne le permettait pas, si la clé primaire colle à la règle de la clé étrangère, ce serait possible):

```sql
SELECT *
FROM "Table1"
  INNER JOIN "Table2" ON "Table2"."idTable1" = "Table1"."id"
```

* Utiliser les enum PG qui sont des types (nous étions old school, je n'ai pas l'argument qui peut-être était derrière)
  - Quid du casing des types ? L'idéal serait aucun, sinon guillemets obligatoires

```sql
CREATE TYPE campaign_status AS ENUM ('sent', 'ordered');

-- Usage
CREATE TABLE "Campaign" (
  -- ...
  "status" campaign_status NOT NULL DEFAULT 'sent'
);
```

* Use the right PostgreSQL types:

```sql
inet --(IP address)
timestamp with time zone
point --(2D point)
tstzrange --(time range)
interval --(duration)
daterange
numeric
```

* Utiliser les tableaux si besoin (permet de gérer la notion "d'ordre" facilement)
* Constraints should be inside your database as much as possible:
* If a constraint is not possible, you can implement a trigger to check a constraint

```sql
CREATE "Reservation" (
  id    UUID PRIMARY KEY,
  dates TSTZRANGE NOT NULL,
  EXCLUDE USING GiST (dates with &&)
);
```

* Let PostgreSQL name your constraints when possible, fallback on the standard when PG can't do it for you
* Standard names for indexes in PostgreSQL are: `{tablename}_{columnname(s)}_{suffix}` (e.g. `item_a_b_pkey`) where the suffix is one of the following:
  * Primary Key constraint: `pkey`
  * Foreign key: `fkey`
  * Unique constraint: `key`
  * Check constraint: `check`
  * Exclusion constraint: `excl`
  * Any other kind of index: `idx`

([source](http://stackoverflow.com/questions/4107915/postgresql-default-constraint-names/4108266#4108266))
