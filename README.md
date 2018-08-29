# CakePHP Application Skeleton with Heroku

[![Build Status](https://img.shields.io/travis/cakephp/app/master.svg?style=flat-square)](https://travis-ci.org/cakephp/app)
[![Total Downloads](https://img.shields.io/packagist/dt/cakephp/app.svg?style=flat-square)](https://packagist.org/packages/cakephp/app)

A skeleton for creating applications with [CakePHP](https://cakephp.org) 3.x.

The framework source code can be found here: [cakephp/cakephp](https://github.com/cakephp/cakephp).

## Changelog

オリジナルの CakePHP スケルトンから、Heroku 用に変更が必要な点は

1. Procfile の準備
2. データベースは、Heroku Postgres を利用
3. キャッシュは、Heroku Redis を利用
4. ログの出力を Heroku 向けへ
5. 環境変数とapp.json の準備
6. スケルトンの標準画面を表示する場合の注意事項

### 1. Procfile の準備

Heroku は、プログラミング言語ごとに、起動するコマンドの標準を準備していますが、`CakePHP` 向けには構成されていないため、起動コマンドを制御する必要があります。

アプリケーションのルートディレクトリに、`Procfile` を準備し、`web:` のあとに、起動するためのコマンドを指定します。
今回は次のように`CakePHP`のデフォルトルートを指定して、`Apache` で起動するようにしています。`Apache` 以外にも `nginx`が利用可能です。

```bash:Procfile
web: vendor/bin/heroku-php-apache2 webroot/
```

### 2. データベースは、Heroku Postgres を利用

Heroku では、`PostgreSQL` を標準的なリレーショナルデータベースとして提供しています。`PostgreSQL` を利用するよう、`config/app.php` 内の定義を変更します。

`return[]` の前に `$db = parse_url(env('DATABASE_URL'));` を記述しておきます。Heroku で `PostgreSQL`を定義すると、自動的に`DATABASE_URL`が定義され、それを利用するようにアプリケーションでは構成することを推奨しています。この `DATABASE_URL`環境変数を利用するように定義を行います。

`Datasources` 内の `default` を `PostgreSQL` 用に追記・修正します。

- `driver` : `PostgreSQL` ドライバを指定します
- `host`, `port`, `username`, `password`, `database` : それぞれについて、環境変数 `DATABASE_URL` の定義を

```
            'driver' => 'Cake\Database\Driver\Postgres',
            'host' => $db['host'],
            'port' => '5432',
            'username' => $db['user'],
            'password' => $db['pass'],
            'database' => substr($db['path'], 1),
```            


### 3. キャッシュは、Heroku Redis を利用


### 4. ログの出力を Heroku 向けへ



### 5. 環境変数とapp.json の準備


### 6. スケルトンの標準画面を表示する場合の注意事項


## Heroku Button

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)
