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
4. セッションは、キャッシュを利用
5. ログの出力を Heroku 向けへ
6. 環境変数とapp.json の準備
7. スケルトンの標準画面を表示する場合の注意事項

### 1. Procfile の準備

Heroku は、プログラミング言語ごとに、起動するコマンドの標準を準備していますが、`CakePHP` 向けには構成されていないため、起動コマンドを制御する必要があります。

アプリケーションのルートディレクトリに、`Procfile` を準備し、`web:` のあとに、起動するためのコマンドを指定します。
今回は次のように`CakePHP`のデフォルトルートを指定して、`Apache` で起動するようにしています。`Apache` 以外にも `nginx`が利用可能です。

```bash:Procfile
web: vendor/bin/heroku-php-apache2 webroot/
```

### 2. データベースは、Heroku Postgres を利用

Heroku では、`PostgreSQL` を標準的なリレーショナルデータベースとして提供しています。`PostgreSQL` を利用するよう、`config/app.php` 内の定義を変更します。

`return[]` の前に `$db = parse_url(env('DATABASE_URL'));` を記述しておきます。Heroku で `Heroku Postgres`をプロビジョニングすると、自動的に`DATABASE_URL`が定義され、それを利用するようにアプリケーションでは構成することを推奨しています。この `DATABASE_URL`環境変数を利用するように定義を行います。

`Datasources` 内の `default` を `PostgreSQL` 用に追記・修正します。

- `driver` : `PostgreSQL` ドライバを指定します
- `host`, `port`, `username`, `password`, `database` : それぞれについて、環境変数 `DATABASE_URL` の定義を利用するように定義

```php:config/app.php
            'driver' => 'Cake\Database\Driver\Postgres',
            'host' => $db['host'],
            'port' => $db['port'],
            'username' => $db['user'],
            'password' => $db['pass'],
            'database' => substr($db['path'], 1),
```            

### 3. キャッシュは、Heroku Redis を利用 (未テスト)

`CakePHP` のキャッシュ機構は、標準ではローカルのファイルシステムを利用します。

Heroku はファイルの永続化が行えないコンテナの仕様ですので、一般的に `Heroku Redis` や `Memcacheir` などのアドオンを利用して、キャッシュやセッション情報を制御します。`Cache` 内の `default`、`_cake_core`、`_cake_model_` を、`Heroku Redis` が利用できるよう変更します。

`return[]` の前に `$cache = parse_url(env('REDIS_URL'));` を記述しておきます。Heroku で `Heroku Redis`をプロビジョニングすると、自動的に`REDIS_URL`が定義され、それを利用するようにアプリケーションでは構成することを推奨しています。この `REDIS_URL`環境変数を利用するように定義を行います。

`Cache` 内の `default`、`_cake_core`、`_cake_model_` を、`Heroku Redis` 用に追記・修正します。

- `className` : `Redis` (または `Cake\Cache\Engine\RedisEngine`) を指定します。
- `server`, `port`, `password` : それぞれについて、環境変数 `REDIS_URL` の定義を利用するように定義

```php:config/app.php
    'Cache' => [
        'default' => [
            'className' => 'Redis',
            'server' => $cache['host'],
            'port' => $cache['port'],
            'password' => $cache['pass'],
            'path' => CACHE,
        ],
        '_cake_core_' => [
            'className' => 'Redis',
            'server' => $cache['host'],
            'port' => $cache['port'],
            'password' => $cache['pass'],
            'prefix' => 'myapp_cake_core_',
            'path' => CACHE . 'persistent/',
            'serialize' => true,
            'duration' => '+1 years',
        ],
        '_cake_model_' => [
            'className' => 'Redis',
            'server' => $cache['host'],
            'port' => $cache['port'],
            'password' => $cache['pass'],
            'prefix' => 'myapp_cake_model_',
            'path' => CACHE . 'models/',
            'serialize' => true,
            'duration' => '+1 years',
        ],
    ],
```

### 4. セッションは、キャッシュを利用

`CakePHP` は、セッション情報を `php.ini` の定義に従うよう構成しています。キャッシュと同様、`Heroku Redis` を利用するように、定義を変更します。`Session` 内の `defaults` を `php` → `cache` へ変更します。

```php:config/app.php
    'Session' => [
        'defaults' => 'cache',
    ],
```

### 5. ログの出力を Heroku 向けへ (未テスト)

Heroku は、標準出力に出力されたものを、Log stream へ出力するように構成されています。`CakePHP` では、ログファイルへ保管するよう標準定義されていますので、標準出力へ出力するよう、`Log` 内の定義を変更します。

- `className` : `Cake\Log\Engine\ConsoleLog` を指定し、コンソールへ出力するよう制御します
- `stream` : `php://stdout` を指定し、標準出力のストリームを指定します

```php:config/app.php
    'Log' => [
        'debug' => [
            'className' => 'Cake\Log\Engine\ConsoleLog',    // for Heroku
            'stream' => 'php://stdout',                     // for Heroku
            'scopes' => false,
            'levels' => ['notice', 'info', 'debug'],
        ],
        'error' => [
            'className' => 'Cake\Log\Engine\ConsoleLog',    // for Heroku
            'stream' => 'php://stdout',                     // for Heroku
            'scopes' => false,
            'levels' => ['warning', 'error', 'critical', 'alert', 'emergency'],
        ],
        // To enable this dedicated query log, you need set your datasource's log flag to true
        'queries' => [
            'className' => 'Cake\Log\Engine\ConsoleLog',    // for Heroku
            'stream' => 'php://stdout',                     // for Heroku
            'scopes' => ['queriesLog'],
        ],
    ],
```

### 6. 環境変数とapp.json の準備

DebugKit がないので、`debug` モードでは Heroku 上で実行させられません。環境変数 `DEBUG` を `false` に定義します。
SALTの値も自動生成されたものではなく、個別のものを利用したい場合は、環境変数 `SECURITY_SALT` に SALT値を定義します。

`app.json` 内で、これらの環境変数を指定しておけば、`Heroku Button` などで展開する際に自動的に定義できます。特に、`SECURITY_SALT` の値を自動生成できるので、指定しておくことを推奨します。

```javascript:app.json
{
  "name": "CakePHP3.6 TEST",
  "description": "Sample: pure CakePHP3.6.8 on Heroku with Heroku Postgres",
  "env": {
    "SECURITY_SALT": {
      "description": "SALT",
      "generator": "secret"
    },
    "DEBUG": {
      "description": "debug mode",
      "value": "false"
    }
  },
  "addons": [
    "heroku-postgresql",
    "heroku-redis"
  ]
}
```

他にも、`addons` で、プロビジョニングしておきたいアドオンを指定しておけば、これらも自動的にプロビジョニングできます。ここでは、`Heroku Postgres` と `Heroku Redis` を自動的にプロビジョニングするよう設定しています。

### 7. スケルトンの標準画面を表示する場合の注意事項

実行確認を行うために、`CakePHP` の `home.ctp` を表示させてみたい場合は、`src/Template/Pages/home.ctp` ファイル内の `if (!Configure::read('debug')) ` 行をコメントアウトしておくと手っ取り早いです。

```php:src/Template/Pages/home.ctp
/*
if (!Configure::read('debug')) :
    throw new NotFoundException(
        'Please replace src/Template/Pages/home.ctp with your own version or re-enable debug mode.'
    );
endif;
*/
```

## Heroku Button

実際の起動画面をとりあえず確認したい場合は、次のボタンをクリックして、ご自分の環境へデプロイしてみてください。

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)
