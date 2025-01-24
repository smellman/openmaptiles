# make版の使い方

## 前作業

```bash
sudo mkdir -p /opt/nvme/docker-data/data
sudo chown -R 777 /opt/nvme/docker-data/data
```

## インポート手順

```bash
# ソースコードのビルド
make 
# エリアのダウンロード
make download area=japan
# データのインポート
make import-data
# OSMデータのインポート
make import-osm
# Wikidataのインポート
make import-wikidata
# レイヤーのインポート
make import-sql
```