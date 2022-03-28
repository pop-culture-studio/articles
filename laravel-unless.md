PHPにはunlessがないのでLaravelが用意している類似機能が地味に役に立つ
----

`if(! $test)`のように`!`での否定はLaravel内部でも普通に使われてるけど個人的にはあまり使いたくない。PHPのゆるい型変換に依存した使い方。Laravel内では`!`で反転してるだけなので実際の動作上はこだわっても意味はない。

## バージョン
- Laravel 9.x
- PHP 8.1

## Bladeの@unless
```php
@unless (Auth::check())
    You are not signed in.
@endunless
```

https://laravel.com/docs/9.x/blade#if-statements

## CollectionのdoesntContain()
```php
$collection = collect(['name' => 'Desk', 'price' => 100]);
 
if($collection->doesntContain('Table')) {

}
```

https://laravel.com/docs/9.x/collections#method-doesntcontain

Laravel内部ではcontains()を反転してるだけ。
```php
    public function doesntContain($key, $operator = null, $value = null)
    {
        return ! $this->contains(...func_get_args());
    }
```

## Conditionable Traitのwhen()/unless()
トレイトなので色々な所で使えるけどCollectionやQueryBuilderで使うことが多いかも。
```php
$collection = collect([1, 2, 3]);
 
$collection->unless(true, function ($collection) {
    return $collection->push(4);
});
```

```php
$role = $request->input('role');
 
$users = DB::table('users')
                ->when($role, function ($query, $role) {
                    $query->where('role_id', $role);
                })
                ->get();
```
これのように`when($role`とは書きたくない。
```php
->when(filled($role),
```
empty()の逆としてfilled()で確認。
```php
->unless(blank($role),
```
unlessならblank()かempty()。

https://github.com/laravel/framework/blob/9.x/src/Illuminate/Conditionable/Traits/Conditionable.php  
https://laravel.com/docs/9.x/collections#method-unless  
https://laravel.com/docs/9.x/queries#conditional-clauses

## abort_unless()
```php
abort_unless(Auth::user()->isAdmin(), 403);
```
```php
abort_if(! Auth::user()->isAdmin(), 403);
```
ドキュメントからも「!で反転よりはunless」がいいような意図が見える。

https://laravel.com/docs/9.x/helpers#method-abort-unless

```php
    function abort_unless($boolean, $code, $message = '', array $headers = [])
    {
        if (! $boolean) {
            abort($code, $message, $headers);
        }
    }
```
ここもLaravel内では!だけど「フレームワーク内のコードとユーザーランドのコードは違う」のでユーザーランドではやはり!は避けたい。
