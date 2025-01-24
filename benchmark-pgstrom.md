# Benchmark pg-strom

## 先にやっておくこと

```bash
# ダウンロードなど
make download area=japan
# コンテナとvolumeを削除
docker compose down -v
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
psql:/sql/parallel/transportation__transportation_name.sql:401: ERROR:  (codegen_expression_walker:2713) not a supported expression type: {NULLIFEXPR :opno 98 :opfuncid 67 :opresulttype 25 :opretset false :opcollid 100 :inputcollid 100 :args ({RELABELTYPE :arg {VAR :varno 1 :varattno 7 :vartype 1043 :vartypmod -1 :varcollid 100 :varnullingrels (b) :varlevelsup 0 :varnosyn 1 :varattnosyn 7 :location 926} :resulttype 25 :resulttypmod -1 :resultcollid 100 :relabelformat 2 :location -1} {CONST :consttype 25 :consttypmod -1 :constcollid 100 :constlen -1 :constbyval false :constisnull false :location 938 :constvalue 4 [ 16 0 0 0 ]}) :location 919}
DETAIL:  problematic expression: <>
CONTEXT:  SQL statement "INSERT INTO transportation_route_member_coalesced
    SELECT
      osm_route_member_filtered.*,
      osm_route_member_network_type(network, ref) AS network_type,
      DENSE_RANK() OVER (
          PARTITION BY member
          ORDER BY osm_route_member_network_type(network, ref), network, LENGTH(ref), ref
      ) AS concurrency_index,
      CASE
           WHEN network IN ('iwn', 'nwn', 'rwn') THEN 1
           WHEN network = 'lwn' THEN 2
           WHEN osmc_symbol || colour <> '' THEN 2
      END AS rank
    FROM (
        -- etldoc:  osm_route_member ->  osm_route_member
        -- etldoc:  gbr_route_members_view ->  osm_route_member
        -- etldoc:  ire_route_members_view ->  osm_route_member
        -- see http://wiki.openstreetmap.org/wiki/Relation:route#Road_routes
        SELECT DISTINCT ON (member, COALESCE(rel.network, ''), COALESCE(rel.ref, ''))
            rel.member,
            COALESCE(NULLIF(rel.network,''), gb_way.network, ir_way.network, '') AS network,
            COALESCE(rel.ref, '') AS ref,
            osm_id,
            role,
            type,
            name,
            osmc_symbol,
            colour,
            ref_colour
        FROM osm_route_member rel
        LEFT JOIN gbr_route_members_view gb_way ON (gb_way.member=rel.member)
        LEFT JOIN ire_route_members_view ir_way ON (ir_way.member=rel.member)
        WHERE full_update OR EXISTS(
            SELECT NULL
            FROM transportation_name.network_changes c
            WHERE c.is_old IS FALSE AND c.osm_id = rel.member
        )
    ) osm_route_member_filtered
    ON CONFLICT (member, network, ref) DO UPDATE SET osm_id = EXCLUDED.osm_id, role = EXCLUDED.role,
                                                     type = EXCLUDED.type, name = EXCLUDED.name,
                                                     osmc_symbol = EXCLUDED.osmc_symbol, colour = EXCLUDED.colour, ref_colour = EXCLUDED.ref_colour,
                                                     concurrency_index = EXCLUDED.concurrency_index,
                                                     rank = EXCLUDED.rank"
PL/pgSQL function update_osm_route_member(boolean) line 14 at SQL statement
Time: 1613.988 ms (00:01.614)
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