# Benchmark pg-strom

## 先にやっておくこと

```bash
# ダウンロードなど
make download area=japan
# コンテナとvolumeを削除
docker compose down -v
# max_stack_depthを64MBに設定
vim my-postgres.conf
# DBを開始
make start-db
```

## Benchmarks

### make import-data

#### postgis 16-3.5

```bash
$ time make import-data
real    0m47.101s
user    0m0.250s
sys     0m0.171s
```

#### pg-strom

```bash
$ time make import-data
real    0m45.799s
user    0m0.216s
sys     0m0.179s
```

### make import-osm

#### postgis 16-3.5

```bash
$ time make import-osm
real    9m28.400s
user    0m0.282s
sys     0m0.195s
```

#### pg-strom

```bash
$ time make import-osm
real    9m21.307s
user    0m0.269s
sys     0m0.214s
```

### make import-wikidata

#### pg-strom (with cache data)

```bash
$ time make import-wikidata
real    0m25.213s
user    0m0.118s
sys     0m0.131s
```

### make import-sql

#### postgis 16-3.5

```bash
$ time make import-sql
real    25m11.382s
user    0m0.378s
sys     0m0.275s
```

#### pg-strom

```bash
$ time make import-sql
```

Then raise error.

```sql
psql:/sql/parallel/water__waterway.sql:1035: ERROR:  type "char" is not device supported
CONTEXT:  SQL expression "NOT EXISTS (
        SELECT constraint_name
        FROM information_schema.table_constraints
        WHERE table_name = 'osm_important_waterway_linestring' AND constraint_type = 'PRIMARY KEY'
    )"
PL/pgSQL function inline_code_block line 3 at IF
Time: 1.422 ms
xargs: sh: exited with status 255; aborting
make: *** [Makefile:453: import-sql] Error 124
```

### generate-tiles

#### postgis 16:3-5

```bash
$ time make generate-tiles-pg
real    143m16.959s
user    0m3.103s
sys     0m3.359s
```