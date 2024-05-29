# docker-laravel
- 参照：https://qiita.com/ucan-lab/items/56c9dc3cf2e6762672f4

## 1.コマンド
1-1. build
```
docker compose build
```

1-2. up
```
docker compose up -d
```

1-3. コンテナ一覧表示
```
docker compose ps
```

1-4. appコンテナの中に入る
```
docker compose exec app bash
```

1-5. コンテナを破壊
- docker-compose.yml を変更する際は破壊する
```
docker compose down
```

1-6. appコンテナの外に出る
```
exit
```

1-7. コンテナの停止、ネットワーク・名前付きボリューム・コンテナイメージ、未定義コンテナを削除
```
docker compose down --rmi all --volumes --remove-orphans
```

1-8. .envの「APP_KEY=」の値生成
```
php artisan key:generate
```

1-9. public/storage から storage/app/public へのシンボリックリンクを張る
```
php artisan storage:link
```

### 2.実行関係
2-1. マイグレーション実行
```
docker compose exec app bash
php artisan migrate
```

2-2. データベース表示
```
docker compose exec db bash
mysql -u $MYSQL_USER -p$MYSQL_PASSWORD $MYSQL_DATABASE
show databases;
show tables;
desc users;
SELECT * FROM users;
```

2-3. データ登録
```
docker compose exec app bash
php artisan tinker
```

```
# 下記のコードはコピペして実行可能。

$user = new App\Models\User();
$user->name = 'phper';
$user->email = 'phper@example.com';
$user->password = Hash::make('secret');
$user->save();
```

### 3.確認関係
3-1. PHPのバージョン確認(コンテナの外からコマンド実行)
```
docker compose exec app php -v
```

3-2. nginxのバージョン確認
```
docker compose exec web nginx -v
```

3-3. myaqlのバージョン確認
```
docker compose exec db mysql -V
```

3-4. ネットワークの詳細を確認
- docker network inspect 「docker network listで表示されたNAME」
```
docker network list

docker network inspect docker-laravel-handson_default
```

3-5. Laravelのログをコンテナログに表示する
- src/.env を修正する。
```env:src/.env
LOG_CHANNEL=stderr
```
- src/routes/web.php に追記する。
```php:src/routes/web.php
Route::get('/', function () {
    logger('welcome route.');
    return view('welcome');
});
```

3-6. ログウォッチ
```
docker compose logs
# -f でログウォッチ
 docker compose logs -f
# サービス名を指定してログを表示
docker compose logs -f app
```

### 5.インストール関係
***
5-1. Laravelをインストールする
```
docker compose exec app bash
composer create-project --prefer-dist "laravel/laravel=9.*" .
chmod -R 777 storage bootstrap/cache
php artisan -V

exit
```

***

## URL
1. ローカル
http://localhost:8080/

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

## sec04_createMysqlDBContainerMigration
1. infra/mysql/my.cnf を作成
- 文字コード、照合順序の設定
- タイムゾーンの設定
- ログ設定
- 文字コードと照合順序は、 utf8mb4, utf8mb4_ja_0900_as_cs_ks を選択するのが現状良いとされている
