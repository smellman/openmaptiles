# Benchmark pg-strom

## 先にやっておくこと

```bash
# cacheの削除
rm -fr cache build
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
real    0m45.414s
user    0m0.242s
sys     0m0.186s
```

#### pg-strom

```bash
$ time make import-data
real    0m45.504s
user    0m0.280s
sys     0m0.139s
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
real    9m31.902s
user    0m0.301s
sys     0m0.173s
```

### make import-wikidata

#### pg-strom (with cache data)

```bash
$ time make import-wikidata
real    0m24.348s
user    0m0.123s
sys     0m0.116s
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
real    18m55.744s
user    0m0.389s
sys     0m0.209s
```

### generate-tiles

#### postgis 16:3-5

```bash
$ time make generate-tiles-pg
real    143m16.959s
user    0m3.103s
sys     0m3.359s
```

#### pg-strom

```bash
$ time make generate-tiles-pg
real    408m14.850s
user    0m8.174s
sys     0m8.436s
```