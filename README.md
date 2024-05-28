# docker-laravel
- 参照：https://qiita.com/ucan-lab/items/56c9dc3cf2e6762672f4

## コマンド
1. build
```
docker compose build
```

1-1. up
```
docker compose up -d
```

1-2. コンテナ一覧表示
```
docker compose ps
```

1-3. appコンテナの中に入る
```
docker compose exec app bash
```

1-4. コンテナを破壊
- docker-compose.yml を変更する際は破壊する
```
docker compose down
```

2-1. appコンテナの外に出る
```
exit
```

3. PHPのバージョン確認(コンテナの外からコマンド実行)
```
docker compose exec app php -v
```

3-1. nginxのバージョン確認
```
docker compose exec web nginx -v
```

## URL

## 内容
### ウェブサーバー(web)コンテナを作る
1. nginxウェブサーバーコンテナを作成する
- nginxのベースイメージをそのまま利用する - https://hub.docker.com/_/nginx
```
.
├── infra
│   └── nginx
│       └── default.conf # nginxの設定ファイル
├── src
│  └── public # 動作確認用に作成
│       ├── index.html # HTML動作確認用
│       └── phpinfo.php # PHP動作確認用
└─── docker-compose.yml
```

2. docker-compose.yml へ追記
```yml:docker-compose.yml
version: "3.9"
services:
  app:
    build: ./infra/php
    volumes:
      - ./src:/data

  # 追記
  web:
    image: nginx:1.20-alpine
    ports:
      - 8080:80
    volumes:
      - ./src:/data
      - ./infra/nginx/default.conf:/etc/nginx/conf.d/default.conf
    working_dir: /data
```