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
toc_sticky: false
---
# Must have extentions
1. **[pg_stat_statements](https://www.crunchydata.com/blog/tentative-smarter-query-optimization-in-postgres-starts-with-pg_stat_statements)** (std contrib package) - see 'Query Performance' below
2. **[auto_explain](https://www.postgresql.org/docs/current/auto-explain.html)**(std contrib package) - custom config is necessary, see details below
3. **[Citus](https://github.com/citusdata/citus)** - turns postgres into a sharded, distributed, horizontally scalable database. 
4. **[Pg_search](https://github.com/paradedb/paradedb)** - advanced search, but needs configuration(see below) 
5. **[Pg_cron](https://github.com/citusdata/pg_cron)** 
## How to add ext (`pg_stat_statements`) on docker Postgres locally
```sh
# copy, edit, copy back
docker cp ./postgresql.conf postgresql-foo:/var/lib/postgresql/data/postgresql.conf

# OPT: fix permissions
docker exec -u 0 -it postgresql-foo chown postgres:postgres /var/lib/postgresql/data/postgresql.conf

# reload conf
docker exec -it postgresql-foo psql -U postgres -c "SELECT pg_reload_conf();"

# or reload container
docker restart postgresql-foo
```
 connect and run
```bash
psql -U postgres postgres
```
```sql
CREATE EXTENSION pg_stat_statements;
```

# Pre-warm cache

[PGdocs](https://www.postgresql.org/docs/current/pgprewarm.html)
[Useful scripts](https://github.com/robins/PrewarmRDSPostgres/tree/master)

---

# Indexing

- takeaways from reading - [Effective_Indexing_in_Postgres.](https://resources.pganalyze.com/pganalyze_Effective_Indexing_in_Postgres.pdf)
- [my sandbox](https://github.com/friendlyantz/demystifying-postgres)

> B-tree data structure does not support indexing lookups on arbitrary keys for a JSONB value.
see below
```sql
CREATE INDEX ON mytable (int_price, jsonb_description);

EXPLAIN SELECT * FROM companies WHERE price = 123 AND description ? 'osA';

                          QUERY PLAN
--------------------------------------------------------------
 Bitmap Heap Scan on companies  (cost=28.79..3123.30 rows=1 width=40)
   Recheck Cond: (price = 123)
   Filter: (description ? 'osA'::text)
   ->  Bitmap Index Scan on companies_price_description_idx  (cost=0.00..28.79 rows=1115 width=0)
         Index Cond: (price = 123)
```

## Operator classes / families

 they map a specific operator on a data type to the actual implementation.
i.e. check all operator classes/families defined for the equality operator (`=`)
```sql
SELECT am.amname AS index_method,
	opf.opfname AS opfamily_name,
	amop.amopopr::regoperator AS opfamily_operator
FROM pg_am am,
	pg_opfamily opf,
	pg_amop amop
WHERE opf.opfmethod = am.oid AND amop.amopfamily = opf.oid
AND amop.amopopr = '=(text,text)'::regoperator;



 index_method |  opfamily_name   | opfamily_operator
--------------+------------------+-------------------
 btree        | text_ops         | =(text,text)
 hash         | text_ops         | =(text,text)
 btree        | text_pattern_ops | =(text,text)
 hash         | text_pattern_ops | =(text,text)
 spgist       | text_ops         | =(text,text)
 brin         | text_minmax_ops  | =(text,text)
 brin         | text_bloom_ops   | =(text,text)
```

equality comparisons on text data types are not supported on the GIN index type by default.

 - [ ] “text_ops” (the default), and 
 - [ ] “text_pattern_ ops” -  can be useful when your database is using a non-C locale (e.g. “en_ US.UTF-8”), and you are trying to index the `~~` operator (commonly known by its alias, “LIKE”). 

```sql
CREATE INDEX ON mytable (column3 text_pattern_ops);
```

## B-tree(balanced, not binary) indexes

produce sorted output for your queries (as the leaf pages are pre- sorted in the index),

more RTFM on btrees in postgres
*https://github.com/postgres/postgres/blob/master/src/backend/access/nbtree/README*

## Hash indexes

Hash indexes fit very similar use cases compared to B-tree indexes. They are solely focused on equality operators (“=”) and do not support any other
operators

```sql
CREATE INDEX ON mytable USING hash (column1);
```

## BRIN indexes (Block Range Index)

> “Block” in Postgres terminology refers to a section of a table, typically 8 kb in size.

TLDR: **BRIN indexes help in very specific situations where the physical structure of the table correlates with the range of a value**

-  very small and imprecise
- best used with append-only tables, where the indexed values linearly increase and the rows are added at the end of the table, and you don’t delete/update values

##  GIN, GIST, SP-GIST

Generic Index Types 

- [ ] **GIN:** Inverted data structures - commonly the index entries are composite values and the kind of query the index supports is a search for one of the elements of the composite value
- [ ] **GIST:** Search trees - implements a balanced, tree- structured index that is versatile. B-trees, [R-trees](https://postgis.net/workshops/postgis-intro/indexing.html#rtree) and other data structures are a good fit for GIST.
- [ ] **SP-GIST:** Spatial search trees - implements a non-balanced data structure that repeatedly divides the search space into partitions of different sizes, usable for structures such as [quad-trees](https://en.wikipedia.org/wiki/Quadtree), [k-d trees](https://en.wikipedia.org/wiki/K-d_tree) and [radix trees](https://en.wikipedia.org/wiki/Radix_tree).

use-cases:
- GIN: [JSONB](https://www.postgresql.org/docs/13/datatype-json.html#JSON-INDEXING) (e.g. finding key/value pairs, checking whether keys exist)
- GIN / GIST: Full text search
- GIST: Geospatial data
- GIST: ltree
- GIST: Range types (e.g. checking whether values are contained in the range)

## Creating The Best Index For Your Queries


Indexing multiple columns is supported for B-tree, GIST, GIN and BRIN index types.

 one index on multiple columns vs multiple indexes on one column each, **it often is preferable to rely on a single index**
 
Things to watch out for b-tree index:
- Ordering matters - the columns in the beginning to be used by all the queries
- Postgres <=12: put high-cardinality(diverse) columns in front of the index
 - more columns in index: less likely planner will use it
 - for `Index Only Scans`(aka covering index): use the `INCLUDE` keyword to specify columns whose data is included in the index, but which are not used for filtering purposes
 - 
 
 ![[built-in index types postgresql.png]]



---

## Misc - Indexing

EXPLAIN, you will typically see one of four main types of scanning a table in Postgres:

- **Sequential Scan:**
- **Index Scan:**
- **Index Only Scan:** - result directly from the index,
- **Bitmap Index Scan** **+** **Bitmap Heap Scan:** Uses an index to generate a bitmap of which parts of the table likely contain the matching rows, and then access the actual table to get these rows using the bitmap - this is particularly useful to combine different indexes (each resulting in a bitmap) to

implement AND and OR conditions in a query

---

# Logs

## Logfile details / db stat helpers

```sh
rails db
```

```sql
SELECT  pg_current_logfile();
```
or in Debian
```
`pg_lsclusters`
```

DB stats
```sql
SELECT datname, numbackends, xact_commit, xact_rollback, tup_inserted, tup_updated, tup_deleted
FROM pg_stat_database;

```

### Modify Docker postgres config

```sh
# download postgres config from container
docker cp demystifying-postgres:/var/lib/postgresql/data/postgresql.conf ./

# set the setting, i.e.
log_checkpoints = on

# copy back similar to above
# restart container
docker restart demystifying-postgres
```

## Transaction ID (TXID) Wraparound

https://pganalyze.com/docs/log-insights/autovacuum/A61
https://pganalyze.com/docs/log-insights/autovacuum/A62

```sh
WARNING: database “template1” must be vacuumed within 938860 transactions

HINT: To avoid a database shutdown, execute a database-wide VACUUM in that database.

You might also need to commit or roll back old prepared transactions.
```
- 32bit at the moment(4 billion unique transaction IDs.), WIP for 64bit - you can run out of available transaction IDs, resulting in the database becoming unavailable.
```sql
SELECT age(datfrozenxid) FROM pg_database;
```

---

## Data Corruption

```text
WARNING: page verification failed, calculated checksum 20919 but expected 15254
```

1. make sure you have checksums enabled in Postgres - enable it, otherwise you never know when data becomes corrupted.

choices if this happen: 
1. **Fail over to an HA standby server / follower database**
2.  **Replay WAL from an old enough base backup** (Assuming you store both your base backups and all WAL (Write-Ahead Logging) files between them,)
3. **Accept that some data is lost, and let Postgres move on** https://pganalyze.com/docs/log-insights/server/S6


## Reducing I/O spikes: Tuning checkpoints

checkpoints processes responsible for writing data thats is only in shared memory and the WAL, to disk. 
Quite I/O intensive, as they need to write a lot of data to disk and run fsync to ensure crash safety..

check if it is
```sql
SHOW log_checkpoints;
```
enable it (this works on some systmes)
```sql
ALTER SYSTEM SET log_checkpoints = 'on';
SELECT pg_reload_conf();
```
or in config
```sh
log_checkpoints = on
```

this produces
```
LOG: checkpoint starting: xlog
...

...

...
LOG: checkpoint complete: wrote 111906 buffers (10.9%); 0 WAL file(s) added, 22 removed,

29 recycled; write=215.895 s, sync=0.014 s, total=216.130 s; sync files=94, longest=0.014 s,

average=0.000 s; distance=850730 kB, estimate=910977 kB
```

**Why was this checkpoint performed?**
1. `xlog` - Checkpoint that runs after a certain amount of WAL has been generated since the last checkpoint, as determined by `max_wal_size` (or `checkpoint_segments` in older Postgres versions)
2. `time` - Checkpoint that runs after a certain amount of time has passed since the last checkpoint, as determined by `checkpoint_timeout`

> The common tuning guidance is to optimize for more **time** based checkpoints

also Postgres hints you if it too frequent
```
LOG: checkpoints are occurring too frequently (18 seconds apart)
HINT: Consider increasing the configuration parameter “max_wal_size”.
```
In any scenario other than bulk data loads, you should look at tuning the settings if you see this event.
## Temporary Files (Improving performance)
By default you won’t notice this, unless you run `EXPLAIN ANALYZE` on a query to see the detailed query execution plan. 
However, you can enable the `log_temp_files`
```sql
LOG: temporary file: path “base/pgsql_tmp/pgsql_tmp15967.0”, size 200204288
STATEMENT: alter table pgbench_accounts add primary key (aid)
```
> a lot of temporary file log events --->  increasing `work_mem`.

You can set `work_mem`:
1. in config / gloabally
2. per-connection, or per-statement basis, using the `SET` command. (*You may want to combine a SET command first, together with `EXPLAIN ANALYZE`, to verify the exact temporary file creation behaviour.*)
 
 **Why?**: Postgres ships with a very low default for the work_mem setting that is used for regular operations. By default this only allows for **4 MB of memory** used for sorting and grouping operations.
 [more info](https://pganalyze.com/docs/log-insights/server/S7)
## Lock Notices & Deadlocks (Improving performance)

> you can enable the `log_lock_waits = on` setting ->  **this will log all** **lock requests that were not granted within 1 second**,

[more](https://pganalyze.com/docs/log-insights/locks/L71)
```sql
LOG: process 2078 still waiting for ShareLock on transaction 1045207414 after 1000.100 ms
DETAIL: Process holding the lock: 583. Wait queue: 2078, 456
QUERY: INSERT INTO x (y) VALUES (1)
CONTEXT: PL/pgSQL function insert_helper(text) line 5 at EXECUTE statement
STATEMENT: SELECT insert_helper($1)


....lock granted.....

LOG: process 583 acquired AccessExclusiveLock on relation 185044 of database 16384 after 2175.443 ms
STATEMENT: ALTER TABLE x ADD COLUMN y text;
```
or
```sql
LOG: process 123 detected deadlock while waiting for AccessExclusiveLock on extension of relation 666 of database 123 after 456.000 ms
ERROR: deadlock detected
DETAIL: Process 9788 waits for ShareLock on transaction 1035; blocked by process 91.
Process 91 waits for ShareLock on transaction 1045; blocked by process 98.
Process 98: INSERT INTO x (id, name, email) VALUES (1, ‘ABC’, ‘abc@example.com’) ON
CONFLICT(email) DO UPDATE SET name = excluded.name, /* truncated */
Process 91: INSERT INTO x (id, name, email) VALUES (1, ‘ABC’, ‘abc@example.com’) ON
CONFLICT(email) DO UPDATE SET name = excluded.name, /* truncated */”
HINT: See server log for query details.
CONTEXT: while inserting index tuple (1,42) in relation “x”
STATEMENT: INSERT INTO x (id, name, email) VALUES (1, ‘ABC’, ‘abc@e
```

lock issues can be difficult to analyze, but especially when looking at a pattern over time, or after an incident, it is critical to run your system with `log_lock_waits = on` for full details in the Postgres logs.

## EXPLAIN plans through `auto_explain` ext.

https://www.postgresql.org/docs/11/auto-explain.html
this ext bundled with `pg_stat_statements` in `shared_preload_libraries`
```
shared_preload_libraries = pg_stat_statements, auto_explain
```

recommended config:
```
auto_explain.log_analyze = 1
auto_explain.log_buffers = 1
auto_explain.log_timing = 0
auto_explain.log_triggers = 1
auto_explain.log_verbose = 1
auto_explain.log_format = json
auto_explain.log_min_duration = 1000
auto_explain.log_nested_statements = 1
auto_explain.sample_rate = 1
```
> 1 and 0 can be replaced with `on` and `off`
`log_timing` is expensive, only for non `prod` systems

`log_analyze` is also expensive and can be disabled on limited hardware. Though this will give you information about the actual query execution, just like EXPLAIN (ANALYZE), which is critical when understanding what actually got executed
## Security / Privacy

PII can leak into logs

```sql
ERROR: duplicate key value violates unique constraint “test_constraint”
DETAIL: Key (b, c)=(12345, mysecretdata) already exists.
STATEMENT: INSERT INTO a (b, c) VALUES ($1,$2) RETURNING id
```

---

# Vacuuming

you can do an online vacuum using `pg_repack` without locking the table
https://reorg.github.io/pg_repack/

---

# Search / Indexing

Indexed query was faster on M1 - 50x times for short string `ABCD` with 500k dataset

`ILIKE` does not leverage standard index
- with wildcard on both sides vs only on the right side:  little difference. both sides wildcard is slower 10-20%
- wildcard provided little, to no benefit to ILIKE, however if data after wildcard was more or less identical, it was magnitudes faster. i.e. `ABCD-predictable_string_replaced_by_wildcard` -> `ABCD%`
Partial index (~20%) scan 
-  provided substantial improvement too, perhaps proportional to the size of the index, but not sure.
- Also I noted hitting Partially indexed table second time with the same query (outside index range) was 100x faster, while others did not change
## Using Indexes with LIKE and ILIKE

GIN / GIST indexes together with `pg_tgrm` can sometimes be used for LIKE and ILIKE, but query performance is unpredictable when user-generated input is presented.

> **GIN** indexes are the preferred text search index type. [PG sauce](https://www.postgresql.org/docs/current/textsearch-indexes.html)

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

      disable_ddl_transaction! # disable_ddl_transaction! added when creading new migration to add new index , and set the algorithm to concurrently so that this index will be added concurrently. This is useful (or crucial) when you have large amounts of rows and want to avoid locking the table while the index being applied

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
## Model scope to order by `similary` match

```ruby
class GinIndexedCompany < ActiveRecord::Base
  scope :name_similar,
        lambda { |name|
          quoted_name = ActiveRecord::Base.connection.quote_string(name)
          where('gin_indexed_companies.name % :name', name:).order(
            Arel.sql("similarity(gin_indexed_companies.name, '#{quoted_name}') DESC")
          )
        }
end

GinIndexedCompany.name_similar('William').first
```

---
## Minimal full text search

basic and very slow search
```ruby
require 'pg_search'

class UnindexedCompany < ActiveRecord::Base
  include PgSearch::Model
  pg_search_scope :search, against: :description
end


UnindexedCompany.search('solutions').first
```

---
## The Foundations of Full Text Search

### tsvector

```sql
SELECT 
to_tsvector('english', 'the weather in Melbourne is known for its variability as well as predictability');

-----------------------------------------------------------------------
 'known':6 'melbourn':4 'predict':13 'variabl':9 'weather':2 'well':11
(1 row)
```
it converts the provided `documents`(a string) to a `tsvector`(text search vector) containing `lexemes`, stripped of [stop words(very common  words like 'a', 'the', etc.)](https://www.postgresql.org/docs/current/textsearch-dictionaries.html#TEXTSEARCH-STOPWORDS)

---
### tsquery

```sql
SELECT
to_tsquery('english', 'weather & predictable & variable');

-----------------------
 'weather' & 'predict' & 'variabl'
 ```

---

### compare using `@@`

```sql
SELECT
to_tsvector('english', 'the weather in Melbourne is known for its variability as well as predictability')
@@ 
to_tsquery('english', 'weather & predictable & variable'); -- TRUE
```

---

### ts_rank


```sql
SELECT
ts_rank(
	to_tsvector('english', 'the weather in Melbourne is known for its variability as well as predictability'),
	to_tsquery('english', 'weather & predictable & variable')
	);

------------
 0.07614762
(1 row)
```

### Combining these together

#### Single column search

```sql
SELECT
	id,
	name,
	description,
	ts_rank(
		to_tsvector('english', description),
		to_tsquery('english', 'weather & predictable')
	) AS rank
FROM unindexed_companies
WHERE 
	to_tsvector('english', description)
	@@
	to_tsquery('english', 'weather & predictable') -- TRUE
ORDER BY rank DESC
LIMIT 5
;


   id   |   name    |                                   description                                   |    rank
--------+-----------+---------------------------------------------------------------------------------+-------------
 456978 | Auckland  | the variable weather in Auckland is not as predictable as in Melbourne          | 0.085297264
 456977 | Melbourne | the weather in Melbourne is known for its variability as well as predictability | 0.029667763
(2 rows)
```


#### Searching Multiple Columns with `setweight`

Data types of tsvector can be concatenated with the `||` operator, and precedence (weight) can be given to a certain column using the setweight function.

Valid weighting options are: A, B, C or D.

```sql
SELECT
	id,
	name,
	description,
	ts_rank(
		setweight(to_tsvector('english', name), 'A')
		||
		setweight(to_tsvector('english', description), 'B')
		,
		to_tsquery('english', 'Melbourne & predictable')
	) AS rank
FROM unindexed_companies
WHERE
	setweight(to_tsvector('english', name), 'A')
	||
	setweight(to_tsvector('english', description), 'B')
	@@
	to_tsquery('english', 'Melbourne & predictable') -- TRUE
ORDER BY rank DESC
LIMIT 5
;


   id   |   name    |                                   description                                   |    rank
--------+-----------+---------------------------------------------------------------------------------+------------
 456978 | Auckland  | the variable weather in Auckland is not as predictable as in Melbourne          | 0.38943392
 456977 | Melbourne | the weather in Melbourne is known for its variability as well as predictability |   0.285989
(2 rows)
```

----
#### advanced `pg_search_scope`

```ruby
class UnindexedCompany < ActiveRecord::Base
  include PgSearch::Model
  # pg_search_scope :search, against: :description # unscoped / no weights
  pg_search_scope :search,
                  against: { name: 'A', description: 'B' }, # can be 'A', 'B', 'C' or 'D'
                  using: { tsearch: { dictionary: 'english' } }
end
```

---
# Query Performance

1. Add index
2. Reviewing Query Performance -> [enable `pg_stat_statements` ext](https://pganalyze.com/docs/install/01_enabling_pg_stat_statements?utm_source=ebook_optimizing-query-performance) - comes by default on Debian Postgres package
> One thing to note is that pg_stat_statements records its statistics from the beginning of when it was installed, or alternatively, from when you’ve last reset the statistics ---> `pg_stat_statements_reset()`

```sql
SELECT queryid, calls, mean_exec_time, substring(query for 100)
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

```sql
SELECT query FROM pg_stat_statements WHERE queryid = 823659002;
						query
--------------------------------------------------------------------
SELECT “backend_wait_events”.”wait_event” FROM “backend_wait_events”
WHERE “backend_wait_events”.”backend_id” = $1
```

$1 is normalised `id` / binding param - to find binding param for slow query use 
1. `pg_stat_activity` table, which shows the currently running
2. logs with `log_min_duration_statement = 1000`(ms or +/-) threshold instead of  `log_statement = all` - for this you need `auto_explain` ext (refer above for config)
```sql
EXPLAIN (ANALYZE, BUFFERS) YOIUR_QUERY;
```
# JIT

https://pganalyze.com/blog/postgres11-jit-compilation-auto-prewarm-sql-stored-procedures

# Finding Root cause of slow query with EXPLAIN
[pganalyze_Finding_the_root_cause_of_slow_Postgres_queries_using_EXPLAIN.](https://resources.pganalyze.com/pganalyze_Finding_the_root_cause_of_slow_Postgres_queries_using_EXPLAIN.pdf)
When Postgres receives a query, it runs through the following four phases at the high-level:
1. **Parsing:** Turning a SQL statement into a parse tree (FAST)
2. **Analysis:** Identifying which tables are referenced (FAST)
3. **Planning:** Determining how to execute the query based on 
	1. configuration settings,
	2. the table schema and 
	3. the indexes that were created.
4. **Execution:** Actually executing the queryin the

**EXPLAIN** plans are captured in two ways:
1. **EXPLAIN ANALYZE:** Planning and Execution phases
2. **EXPLAIN (without ANALYZE):** only Planning step and does not actually run the query
> Ideally we want EXPLAIN ANALYZE, or the similarly named auto_explain.log_analyze, but overhead of actually running the query too high

## Sort memory - Sorts are slower when they spill to disk (~10% performance diff)

be alarmed when `EXPLAIN ANALYZE` shows sort with external disk merge
```
Sort Method: external merge Disk: 9856kB
```
since **default value** that Postgres sets initially (4 MB), which is **often too small**.
```sql
SHOW work_mem;
 work_mem
----------
 4MB
(1 row)
```
[postgres docs link on work_mem](https://www.postgresql.org/docs/13/runtime-config-resource.html#GUC-WORK-MEM)
```sql
SET work_mem = '10MB';
```
## Add `BUFFERS`
```sql
EXPLAIN (ANALYZE, BUFFERS) YOUR_QUERY;
```
Each buffer page is 8kb of data from a table or index that is referenced in the plan node.

```sql
Buffers: shared read=87747
```
87747 x 8kb = 686Mb
# Bloat removal


https://www.depesz.com/2013/10/15/bloat-removal-without-table-swapping/

---
# Approx. table count

```ruby
ROW_COUNT_SQL = <<~SQL.squish
  SELECT reltuples AS row_count
  FROM pg_class
  WHERE relname = '%s'
SQL

# Running a COUNT(*) in PostgreSQL can be resource heavy so, if there isn't
# a need to be exact, this will be much faster
def self.approximate_row_count(model)
  sql = format(ROW_COUNT_SQL, model.table_name)
  result = ActiveRecord::Base.connection.execute(sql)
  result.first["row_count"]
end
```

---

# Size of tables
```sql
SELECT pg_size_pretty( pg_total_relation_size('tablename') );
```
or
```sql
SELECT
    c.relname AS relname,
    c.relkind AS relkind,
    pg_size_pretty(pg_total_relation_size(c.oid)) AS size
FROM 
    pg_class c
JOIN 
    pg_namespace n ON n.oid = c.relnamespace
WHERE 
    n.nspname = 'public'
    AND c.relkind IN ('r', 'i')
ORDER BY 
    pg_total_relation_size(c.oid) DESC;
```

---

# PG Stat Tuple

```sql
CREATE EXTENSION pg_stat_tuple;

\dx

%% turn on extended display, vertical table instead horizontal%%
\x
```
---

---

# [SQL fundamentals & types of DBs refresher](https://youtu.be/W2Z7fbCLSTw)

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

## Creation

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

## Insertion

> USE `'sincle quotes'`
> Note date format: YYYY-MM-DD

```sh
INSERT INTO person (name, fav_num, dob)
VALUES ('Antz', 9, date '1987-04-01');

# alternatively use a '.sql' file via psql \i command
psql
\i  /Users/somepath.sql
```

## Select
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

## Comparator

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

## Explain Analyze

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
