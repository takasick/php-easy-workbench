php-easy-workbench

XAMPPやMAMPのようにPHPでサッと何かを動かしてみたい時の作業台です。色々なバージョンを試しやすいようにコンテナで動くようにしてあります。

# できること
- PHP(mod_php)でPHPアプリケーションを実行する
- composerを使ってPHPライブラリを管理する
- cronを使って定期的に何かを実行する
- mysqlを使ってデータベースを使う
- phpmyadminを使ってデータを操作する

## 制限
セキュリティについて全く考慮していません。作業用のコンピューター等不特定多数からのアクセスが無い環境でのみ使用してください。

# 必要なもの
Podman DesktopもしくはDocker Desktopをインストールしてください。コマンドを実行する部分はわかりやすいようにdockerに統一していますが、podmanでも動作します。

- podman: https://podman.io/
- docker: https://www.docker.com/products/docker-desktop/

# 使い方
以下のコマンドはターミナル等でこのファイルのあるディレクトリに移動して実行してください。

## 起動
```
docker compose up -d
```
http://localhost:8080/ にアクセスすると、ローカルの php/app/html に配置したファイルが表示されます。phpとして実行したい場合は拡張子を .php にしてください。

## 停止
```
docker compose down
```

## composerの実行
ローカルの php/app がコンテナ内では /var/www にマウントされています。例えば php/app の初期設定をする場合は
```
docker exec -it php-easy-workbench-php-1 composer -d /var/www init
```
で実行できます。

## cronの設定
php/crontab ファイルで設定したジョブが実行されます。実行されているか確認したい時は
```
docker logs -f php-easy-workbench-php-1
```
注: 確認できるのはcron自身のログです。ジョブのログを確認したい場合は 1>/dev/shm/some_job.log 2>&1 のようにしてください。

## mysql
初期ユーザーはrootでパスワードは空です。phpmyadmin( http://localhost:8080/phpMyAdmin/ )から操作できます。mysqlコマンド等を使いたい場合は
```
docker exec -it php-easy-workbench-mysql-1 mysql -u root
```

# トラブルシューティング
## phpコンテナが変になったので作り直したい
停止後に
```
docker compose build --no-cache
```
でビルドしてから起動してください。
注: コンテナ内のデータは全て消えます

## mysqlコンテナが変になったので作り直したい
停止後に
```
docker volume rm php-easy-workbench_mysql-vol
```
でボリュームを削除してから起動してください。
注: データベース内のデータは全て消えます

## 一回全部消してやり直したい
```
docker-compose down --rmi all --volumes --remove-orphans
```
