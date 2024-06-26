Laravel11 ドキュメントに書いてないbootstrap/app.phpの使い方
----

海外の質問サイトでよく質問されてるのでまとめる。

日本では全く見ない。日本のLaravelコミュニティは**ドキュメントに書いてることを質問する段階の初心者**と**質問する必要のないベテラン**の両極端に分かれている。ドキュメントにないちょっと難しいことをする段階の人がいない。

## バージョン
- Laravel 11
- PHP 8.3

Laravelの話をする時は必ずバージョンを確認する。古いバージョンにはない機能や将来のバージョンでは使い方が変わってることがよくある。

## bootstrap/app.phpより前の基本的な流れ
Laravelの入り口は  
Httpなら`public/index.php`
```php
// Bootstrap Laravel and handle the request...
(require_once __DIR__.'/../bootstrap/app.php')
    ->handleRequest(Request::capture());
```

Consoleなら`artisan`
```php
// Bootstrap Laravel and handle the command...
$status = (require_once __DIR__.'/bootstrap/app.php')
    ->handleCommand(new ArgvInput);
```

どちらからも読み込まれてる`bootstrap/app.php`がLaravel11で重要ファイルに変わった。

## bootstrap/app.phpの普通の使い方
実際のところ、普通の使い方がドキュメントにまとまってない。Laravel11のリリースノートくらい。あとはドキュメントの各所に機能ごとに書かれてる。  
https://laravel.com/docs/11.x/releases

大事な前提。**bootstrap/app.phpはLaravelの起動前**ってこと。起動後に当たり前に使えてる機能がここでは使えない。  
ここを理解してないと`bootstrap/app.php`で`config()`などを使おうとしてエラーになる。

（正確には最初に`basePath`を設定してるので`base_path()`くらいは使える。でも使うと他のヘルパーも使えると勘違いされるので使ってないと予想できる）

## 調べるコードはApplicationBuilder
`bootstrap/app.php`の`Application::configure()`が返してるのは
```php
use Illuminate\Foundation\Application;

return Application::configure(basePath: dirname(__DIR__))
```
`Illuminate\Foundation\Configuration\ApplicationBuilder`
```php
    public static function configure(?string $basePath = null)
    {
        $basePath = match (true) {
            is_string($basePath) => $basePath,
            default => static::inferBasePath(),
        };

        return (new Configuration\ApplicationBuilder(new static($basePath)))
            ->withKernels()
            ->withEvents()
            ->withCommands()
            ->withProviders();
    }
```
https://github.com/laravel/framework/blob/11.x/src/Illuminate/Foundation/Application.php

`bootstrap/app.php`はほとんどがApplicationBuilderなのでこれを調べればいい。  
https://github.com/laravel/framework/blob/11.x/src/Illuminate/Foundation/Configuration/ApplicationBuilder.php

ここまでが前ふり。

## 起動後に処理を挟むにはbooted()
`bootstrap/app.php`はLaravelの起動前だけど`booted()`だけは起動後。ここでならLaravelの機能がすべて使える。  
「bootstrap/app.phpで特殊なことをしたい」質問への答えはほとんどがこれ。

```php
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        //
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })
    ->booted(function (Application $app) {
        info($app->version());
        info(config('app.name'));
    })->create();
```

簡単な例だと分かりにくいけどこのbootedの段階でもなんでもできる。  
特殊なこととは「withMiddlewareで設定したミドルウェアをすべて無視して再度設定し直す」とか。  
なんでそんなことがしたいのか分からないけどそんな質問も実在した。

普通はしないだろうから詳細は説明しない。

```php
    ->booted(function (Application $app) {
        $kernel = $app->make(Kernel::class);

        $middleware = (new Middleware)
            ->redirectGuestsTo(fn () => route('login'));

        // $middleware->...

        $kernel->setGlobalMiddleware($middleware->getGlobalMiddleware());
        $kernel->setMiddlewareGroups($middleware->getMiddlewareGroups());
        $kernel->setMiddlewareAliases($middleware->getMiddlewareAliases());

        if ($priorities = $middleware->getMiddlewarePriority()) {
            $kernel->setMiddlewarePriority($priorities);
        }

        $app->instance(Kernel::class, $kernel);
    })
```

Laravel内部のことまで熟知してる人しか使わないのでドキュメントに書けない。

## registered()やbooting()もある
Laravelの起動処理はこの順番。`bootstrap/app.php`でも同じ対応なので処理を挟みたいタイミングに合わせて選択。タイミングごとにできることとできないことがある。

- すべてのServiceProviderの`register()`実行
- appの`registered()`
- appの`booting()`
- すべてのServiceProviderの`boot()`実行
- appの`booted()`

ミドルウェアの例も`registered()`のほうが分かりやすいけど
```php
    ->registered(function (Application $app) {
        $app->afterResolving(Kernel::class, function ($kernel) {
            $middleware = (new Middleware)
                ->redirectGuestsTo(fn () => route('login'));

            // $middleware->...

            $kernel->setGlobalMiddleware($middleware->getGlobalMiddleware());
            $kernel->setMiddlewareGroups($middleware->getMiddlewareGroups());
            $kernel->setMiddlewareAliases($middleware->getMiddlewareAliases());

            if ($priorities = $middleware->getMiddlewarePriority()) {
                $kernel->setMiddlewarePriority($priorities);
            }
        });
    })
```

現実的には`bootstrap/app.php`に長いコードを書くよりServiceProviderに書いたほうがいい。  
「すべてのServiceProviderより後に実行したい」なら`booted()`使う意味もあるけどかなり特殊な用途。

AppServiceProviderに書いても同じ。
```php
class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        info('1. AppServiceProvider register');

        $this->booting(function () {
            info('4. AppServiceProvider booting');
        });

        $this->booted(function () {
            info('5. AppServiceProvider booted');
        });

        $this->app->registered(function () {
            info('2. app registered');
        });

        $this->app->booting(function () {
            info('3. app booting');
        });

        $this->app->booted(function () {
            info('6. AppServiceProvider@register app booted');
        });
    }

    public function boot(): void
    {
        $this->app->booted(function () {
            info('7. AppServiceProvider@boot app booted');
        });
    }
}
```

## withBindings()やwithSingletons()は
ServiceProvider使えばいいので`bootstrap/app.php`で書くことは少なそう。

## withMiddleware()は
ドキュメント中にバラバラに書かれているけどほとんど説明されてるはず。  
https://laravel.com/docs/11.x/middleware

詳細まで知りたいならコードを見るしかない。  
https://github.com/laravel/framework/blob/11.x/src/Illuminate/Foundation/Configuration/Middleware.php

## withExceptions()は
ドキュメントで説明してるけどこれだけでは分からない部分もある。  
https://laravel.com/docs/11.x/errors

これもコードを見るしかない。  
https://github.com/laravel/framework/blob/11.x/src/Illuminate/Foundation/Configuration/Exceptions.php  
https://github.com/laravel/framework/blob/11.x/src/Illuminate/Foundation/Exceptions/Handler.php

- 例外の種類ごとにレスポンスを変えるなら`$exceptions->render()`
- 最終的なレスポンスを返す直前の処理を変えたいなら`$exceptions->respond()`

質問でよく見ることならこの辺りを知っておけば十分。
