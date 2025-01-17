+++
toc = true
title = "SQL"
weight = 1
+++

Since the SQL syntax is flexible and complex and distributed databases and stand-alone databases do not have identical query sceanrios, SQLs incompatible with stand-alone databases are hard to avoid.

This document has listed identified supported SQL types and unsupported SQL types, trying to avoid traps for users.

It is inevitably to have some unlisted SQLs, welcome to supplement for that. We will also try to support those unavailable SQLs in future versions.

## Supported SQL

### Route to single data node

- 100% compatible（MySQL only, we are completing other databases).

### Route to multiple data nodes

Fully support DML, DDL, DCL, TCL and some DAL. Support pagination, DISTINCT, ORDER BY, GROUP BY, aggregation and JOIN (does not support cross-database relevance). Here is an example of a most complex kind of DML:

- Main SELECT

```sql
SELECT select_expr [, select_expr ...] FROM table_reference [, table_reference ...]
[WHERE predicates]
[GROUP BY {col_name | position} [ASC | DESC], ...]
[ORDER BY {col_name | position} [ASC | DESC], ...]
[LIMIT {[offset,] row_count | row_count OFFSET offset}]
```

- select_expr

```sql
*
| [DISTINCT] COLUMN_NAME [AS] [alias]
| (MAX | MIN | SUM | AVG)(COLUMN_NAME | alias) [AS] [alias]
| COUNT(* | COLUMN_NAME | alias) [AS] [alias]
```

- table_reference

```sql
tbl_name [AS] alias] [index_hint_list]
| table_reference ([INNER] | {LEFT|RIGHT} [OUTER]) JOIN table_factor [JOIN ON conditional_expr | USING (column_list)]
```

## Unsupported SQL

### Route to multiple data nodes

Do not support CASE WHEN, HAVING and UNION (ALL) and partly available sub-query.

Support not only pagination sub-query (see [pagination](https://shardingsphere.apache.org/document/current/cn/features/sharding/usage-standard/pagination) for more details), but also sub-query with the same mode. No matter how many layers are nested, ShardingSphere can parse to the first sub-query that contains data table. Once it finds another sub-query of this kind in the sub-level nested, it will directly throw a parsing exception.

For example, the following sub-query is available:

```sql
SELECT COUNT(*) FROM (SELECT * FROM t_order o)
```

The following sub-query is unavailable:

```sql
SELECT COUNT(*) FROM (SELECT * FROM t_order o WHERE o.id IN (SELECT id FROM t_order WHERE status = ?))
```

To be simple, through sub-query, non-functional requirements are available in most cases, such as pagination, sum count and so on; but functional requirements are unavailable for now.

Due to the restriction of merger, sub-query that contains aggregation function is unavailable for now.

Do not support SQL that contains schema, for the concept of ShardingSphere is to use multiple data source as one data source, so all the SQL visits are based on one logic schema.

## Example

### Supported SQL

| SQL                                                                                         | Necessary conditions                    |
| ------------------------------------------------------------------------------------------- | --------------------------------------- |
| SELECT * FROM tbl_name                                                                      |                                         |
| SELECT * FROM tbl_name WHERE (col1 = ? or col2 = ?) and col3 = ?                            |                                         |
| SELECT * FROM tbl_name WHERE col1 = ? ORDER BY col2 DESC LIMIT ?                            |                                         |
| SELECT COUNT(*), SUM(col1), MIN(col1), MAX(col1), AVG(col1) FROM tbl_name WHERE col1 = ?    |                                         |
| SELECT COUNT(col1) FROM tbl_name WHERE col2 = ? GROUP BY col1 ORDER BY col3 DESC LIMIT ?, ? |                                         |
| INSERT INTO tbl_name (col1, col2,...) VALUES (?, ?, ....)                                   |                                         |
| INSERT INTO tbl_name VALUES (?, ?,....)                                                     |                                         |
| INSERT INTO tbl_name (col1, col2, ...) VALUES (?, ?, ....), (?, ?, ....)                    |                                         |
| UPDATE tbl_name SET col1 = ? WHERE col2 = ?                                                 |                                         |
| DELETE FROM tbl_name WHERE col1 = ?                                                         |                                         |
| CREATE TABLE tbl_name (col1 int, ...)                                                       |                                         |
| ALTER TABLE tbl_name ADD col1 varchar(10)                                                   |                                         |
| DROP TABLE tbl_name                                                                         |                                         |
| TRUNCATE TABLE tbl_name                                                                     |                                         |
| CREATE INDEX idx_name ON tbl_name                                                           |                                         |
| DROP INDEX idx_name ON tbl_name                                                             |                                         |
| DROP INDEX idx_name                                                                         |  Logic-index is configured in TableRule |
| SELECT DISTINCT * FROM tbl_name WHERE col1 = ?                                              |                                         |
| SELECT COUNT(DISTINCT col1) FROM tbl_name                                                   |                                         |

### Unsupported SQL

| SQL                                                                                        | Reason                                              |
| ------------------------------------------------------------------------------------------ | --------------------------------------------------- |
| INSERT INTO tbl_name (col1, col2, ...) VALUES(1+2, ?, ...)                                 | VALUES clause does not support operation expression |
| INSERT INTO tbl_name (col1, col2, ...) SELECT col1, col2, ... FROM tbl_name WHERE col3 = ? | INSERT .. SELECT                                    |
| SELECT COUNT(col1) as count_alias FROM tbl_name GROUP BY col1 HAVING count_alias > ?       | HAVING                                              |
| SELECT * FROM tbl_name1 UNION SELECT * FROM tbl_name2                                      | UNION                                               |
| SELECT * FROM tbl_name1 UNION ALL SELECT * FROM tbl_name2                                  | UNION ALL                                           |
| SELECT * FROM ds.tbl_name1                                                                 | Contain schema                                      |
| SELECT SUM(DISTINCT col1), SUM(col1) FROM tbl_name                                         | See DISTINCT availability detail                    |

## DISTINCT Availability Explanation

### Supported SQL

| SQL                                                           |
| ------------------------------------------------------------- |
| SELECT DISTINCT * FROM tbl_name WHERE col1 = ?                |
| SELECT DISTINCT col1 FROM tbl_name                            |
| SELECT DISTINCT col1, col2, col3 FROM tbl_name                |
| SELECT DISTINCT col1 FROM tbl_name ORDER BY col1              |
| SELECT DISTINCT col1 FROM tbl_name ORDER BY col2              |
| SELECT DISTINCT(col1) FROM tbl_name                           |
| SELECT AVG(DISTINCT col1) FROM tbl_name                       |
| SELECT SUM(DISTINCT col1) FROM tbl_name                       |
| SELECT COUNT(DISTINCT col1) FROM tbl_name                     |
| SELECT COUNT(DISTINCT col1) FROM tbl_name GROUP BY col1       |
| SELECT COUNT(DISTINCT col1 + col2) FROM tbl_name              |
| SELECT COUNT(DISTINCT col1), SUM(DISTINCT col1) FROM tbl_name |
| SELECT COUNT(DISTINCT col1), col1 FROM tbl_name GROUP BY col1 |
| SELECT col1, COUNT(DISTINCT col1) FROM tbl_name GROUP BY col1 |

### Unsupported SQL

| SQL                                                | Reason                                                                             |
| -------------------------------------------------- | ---------------------------------------------------------------------------------- |
| SELECT SUM(DISTINCT col1), SUM(col1) FROM tbl_name | Use normal aggregation function and DISTINCT aggregation function in the same time |