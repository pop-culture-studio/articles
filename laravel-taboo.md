Laravelの禁忌
----

依頼されてもこんなLaravelプロジェクトとは関われない例。

## viewにjQueryを直接書いてはいけない

古いバージョンのjQueryを読み込んでview内にJSのコードを書いている。Laravel使っててもこんな使い方してる事例は本当に多い。ここに途中参加した場合、どこでなにをやってるかの調査がものすごく大変。引き継いだとしてもこれを修正するのは不可能なので受け付けられない。

```html
<html>
<head>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
</head>
<body>

<button id="btn">ボタン</button>
<script>
    $(function () {
        $("#btn").click(function () {
            alert("hello");
        });
    });
</script>

</body>
</html>
```

## サポート期限の切れたバージョンを使ってはいけない
LaravelもPHPもその他もサポート中のバージョンを使うのは必須。引き継いだ場合はまずバージョンアップ作業。

- Laravel https://laravelversions.com/ja
- PHP https://www.php.net/supported-versions.php
