---
title: PostreSQL
permalink: /postgresql/
excerpt: my findings and tricks
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - databases
  - postgres
toc_sticky: true
---

# Indexing

takeaways from reading - [Effective_Indexing_in_Postgres.](https://resources.pganalyze.com/pganalyze_Effective_Indexing_in_Postgres.pdf)
[my sandbox](https://github.com/friendlyantz/demystifying-postgres)

---

Indexed query was faster on M1 - 50x times for short string `ABCD` with 500k dataset

`ILIKE` does not leverage standard index
- with wildcard on both sides vs only on the right side:  little difference. both sides wildcard is slower 10-20%
- wildcard provided little, to no benefit to ILIKE, however if data after wildcard was more or less identical, it was magnitudes faster. i.e. `ABCD-predictable_string_replaced_by_wildcard` -> `ABCD%`
Partial index (~20%) scan 
-  provided substantial improvement too, perhaps proportional to the size of the index, but not sure.
- Also I noted hitting Partially indexed table second time with the same query (outside index range) was 100x faster, while others did not change
## Using Indexes with LIKE and ILIKE

GIN/GIST indexes together with `pg_tgrm` can sometimes be used for LIKE and ILIKE, but query performance is unpredictable when user-generated input is presented.

**Pros:**
- The user doesn‘t have to worry about the case matching
- The user can enter partial matches
- No additional Ruby gems required

**Cons:**
- Index usage is unpredictable
- Spelling must be accurate (Applo would not find Apple)

```ruby
  class EnablePgTrgmExtension < ActiveRecord::Migration[7.1]
    def change
      enable_extension 'pg_trgm'
    end
  end

  EnablePgTrgmExtension.new.migrate(:up)


  class CreateGinIndexedCompanies < ActiveRecord::Migration[7.1]
    def change
      return if ActiveRecord::Base.connection.table_exists?(:gin_indexed_companies)

      disable_ddl_transaction

      create_table :gin_indexed_companies do |t|
        t.string :exchange, null: false
        t.string :symbol, null: false
        t.string :name, null: false
        t.text :description, null: false
        t.timestamps

        t.index :name, 
	        opclass: :gin_trgm_ops, # Similarity Matches with Trigrams
	        using: :gin, 
	        algorithm: :concurrently, 
	        name: 'index_on_name_trgm'
      end
    end
  end
```
## Trigrams fo Similarity Matches

 - Apple and Applo are more similar than Apple and Opple

```sql
SELECT 
	show_trgm('Apple'), --  {"  a"," ap",app,"le ",ple,ppl} this is done for words with less than 3 letters
	show_trgm('Apllo'), -- {"  a"," ap",apl,llo,"lo ",pll}
	similarity('Apple', 'Apple'), -- 1
	similarity('Applo', 'Apple'), -- 0.5
	similarity('Opple', 'Apple'), -- 0.33333334
	
	'Apple' % 'Applo' -- true (since it is >0.3 similary)
	similarity('Odple', 'Apple'), -- 0.2
	'Odple' % 'Apple' -- false
--  we have the % operator, which gives you a boolean of whether two strings are similar. By default, Postgres uses the number 0.3 when making this decision, but you can always update this setting.)
```

**Pros:**

- Misspellings are no problemmatching
- Searches are case insensitive
- Searches can be indexed

**Cons:**

- Does not replace the need for natural language search

---
---

# [types of DBs refresher](https://youtu.be/W2Z7fbCLSTw)

1. KEY-VALUE pair only: Redis- in RAM, EXTREMELY FAST
2. WIDE COOLUM: Cassandra, HBASE - no schema, CQL language, decentralized, can scale horizontally,
	a. Weather, history
	b. Frequent rights/infrequent updates
3. DOCUMENT DB: Mongo, firestorm, couch db, dyno db
	a. Unstructured/no schema
	b. Collections
4. RELATIONAL DB: MySQL, Postges
	a. Tedd codd from IBM invented 
5. Graph db: neo4j, d graph
	a. No join tables required
6. Search
	a. Based on Apache Lucene: elastic search, solr, Aprilia, meili search
7. MULTI MODEL DB

> Pay Attention: command execution happens only after `;`-char. 'Enter' only helps with formatting on a new line

# Creation

```
psql
CREATE DATABASE sandbox;

psql sandbox
CREATE TABLE user (
    Column name + data type + constraints
    id BIGSERIAL NOT NULL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    gender VARCHAR(6) NOT NULL,
    fav_num int,
    dob TIMESTAMP / DATE NOT NULL
);
```
[data_types in postgres](https://www.postgresql.org/docs/current/datatype.html)


```sh
# do not do this
DROP DATABASE sandbox;

# connect to diff DATABASE
\c name_of_other_db

\d - describe, or \dt - w/o index / sequence
\d table_name - show column_types

```

# Insertion

> USE `'sincle quotes'`
> Note date format: YYYY-MM-DD

```sh
INSERT INTO person (name, fav_num, dob)
VALUES ('Antz', 9, date '1987-04-01');

# alternatively use a '.sql' file via psql \i command
psql
\i  /Users/somepath.sql
```

# Select
```sh
SELECT name, fav_num FROM person;
SELECT name, fav_num FROM person ORDER BY fav_num DESC;


SELECT DISTINCT fav_num FROM person;
SELECT * FROM person WHERE fav_num = 1;
SELECT * FROM person WHERE fav_num = 1 AND name = 'coke';
SELECT * FROM person WHERE fav_num >= 5 OFFSET 1 LIMIT 2
```

## LIMIT

> `LIMIT` is non-orthodox SQL format / keyword
```
SELECT * FROM person WHERE fav_num >= 5 LIMIT 2
# or using FETCH
SELECT * FROM person WHERE fav_num >= 5 FETCH FIRST 2 ROW ONLY;
```

## IN / BETWEEN

```
SELECT * FROM person WHERE fav_num = 1 OR fav_num = 5 OR fav_num = 7;
# but below is better
SELECT * FROM person WHERE fav_num IN (1, 5, 7);
# or between
SELECT * FROM person WHERE fav_num BETWEEN 5 AND 7;
```

## LIKE / ILIKE

> `%` - wildcard
> `_` - single char
```
SELECT * FROM person WHERE name LIKE 'A_t%';
# or case insensitive =-> ILIKE
SELECT * FROM person WHERE name ILIKE 'a_t%';
```

## GROUP BY

> further `SELECT DISTINCT fav_num FROM person;` to
> which only fetches 'distinct' values from that column. we can count them too using COUNT / GROUP BY

```
SELECT fav_num, COUNT(*) FROM person GROUP BY fav_num;

 fav_num | count 
---------+-------
       9 |     1
       5 |     1
      99 |     1
       7 |     1
     777 |     2
       1 |     3
(6 rows)
```
## HAVING

> `HAVING` must be before ORDER BY

```sql
SELECT fav_num, COUNT(*) FROM person GROUP BY fav_num HAVING COUNT(*) >= 2;

 fav_num | count 
---------+-------
     777 |     2
       1 |     3
(2 rows)
```

# Comparator

```sql
SELECT true = true;
 ?column? 
----------
 t
(1 row)

sandbox=# SELECT true = TRUE;
 ?column? 
----------
 t
(1 row)

sandbox=# SELECT 'la' = 'LA';
 ?column? 
----------
 f
(1 row)

```

## not eql
```
SELECT 1 <> 2;
 ?column? 
----------
 t
(1 row)

```

# Explain Analyze

```sh
EXPLAIN ANALYZE SELECT COUNT("debits"."amount") 
FROM "debits"
WHERE "debits"."account_id" = '123' 
AND "debits"."state" IN (1, 2, 3, 4, 5, 6, 8, 9, 10, 11, 12) 
AND "debits"."category" = 0 AND "debits"."matures_at" BETWEEN '2021-11-24 13:00:00' AND '2021-11-25 12:59:59.999999';
```

# Resources

My sandbox:
- https://github.com/friendlyantz/demystifying-postgres

PGAnayle Books:
- [Tuning autovacuum for best Postgres performance](https://resources.pganalyze.com/pganalyze_Tuning_autovacuum_for_best_Postgres_performance.pdf)
- [Advanced Database Programming with Rails and Postgres](https://resources.pganalyze.com/pganalyze_Advanced+Database+Programming+with+Rails.pdf)
- [Best practices for optimizing postgres query performance](https://resources.pganalyze.com/pganalyze_Best-Practices-for-Optimizing-Postgres-Query-Performance.pdf)
- [Effective_Indexing_in_Postgres.](https://resources.pganalyze.com/pganalyze_Effective_Indexing_in_Postgres.pdf)
- [pganalyze_Efficient-Search-in-Rails-with-Postgres](https://resources.pganalyze.com/pganalyze_Efficient-Search-in-Rails-with-Postgres.pdf)
- [pganalyze_Finding_the_root_cause_of_slow_Postgres_queries_using_EXPLAIN.](https://resources.pganalyze.com/pganalyze_Finding_the_root_cause_of_slow_Postgres_queries_using_EXPLAIN.pdf)
- [pganalyze_The-Most-Important-Events-To-Monitor-In-Your-Postgres-Logs.](https://resources.pganalyze.com/pganalyze_The-Most-Important-Events-To-Monitor-In-Your-Postgres-Logs.pdf)

