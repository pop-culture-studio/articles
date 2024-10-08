Laravel 開発環境構築 Mac(Apple Silicon)版
----

[Windows版](./laravel-installation-windows.md)と違って完全に新しいPCでは作業してないけどMacでの構築方法は変わってないので影響ない。

最終更新日：2023年2月  
環境構築は「いつ」の情報かが重要なので更新日から何年も後に読んでも役に立たない。

## 更新履歴
- 2023年2月：新しいMacで実際に環境構築したのでApple Silicon用に書き直し。移行アシスタントを使ったので完全に新規ではなけどApple Siliconへの変更なので色々再インストールは必要だった。

## 対象読者
プログラミング初心者は対象じゃない。Laravelは初心者が使うものじゃない。

すでにLaravelを使っていて大量のLaravelプロジェクトを抱えてる人が別の新しいPCでも開発する場合を想定。  
職場と自宅、WindowsとMacなど複数のPCで併用する前提。  
プロが対象なので有料のツールも遠慮なく含める。

## Laravelを使う前に必須な知識
フレームワークとは「知識の下限を揃えるためのもの」。Laravelの公式ドキュメントはこのくらいは知ってるだろうという前提で進む。

- 一般以上のPCスキル
- Webに関する幅広い知識
- 素のPHP。最新バージョンまで。composerとPSR。
- git / GitHub
- フロントエンドの知識。最低限node.js/npmや「JavaScriptもビルドして使うようになった」という現代の常識は必須。html→PHPと進んできた初心者が躓くのはフロントの知識が足りてないのが主な原因。
- Linuxの知識。最低でも「`php -v`はターミナルで実行する」が分かる程度。

## Chrome
https://www.google.com/intl/ja_jp/chrome/

細かいプラグインは同期すれば勝手にインストールされるので書かない。以降のアプリも同様。

## GitHub Desktop
https://desktop.github.com/

## SourceTree
https://www.sourcetreeapp.com/

GitHub Desktopだけでは足りないことがあるのでSourceTreeもインストール。MacではSourceTreeのほうがメイン。

## VS code
https://code.visualstudio.com/download

## PhpStorm
https://www.jetbrains.com/ja-jp/phpstorm/download/

普段の開発はPhpStormを使うけどちょっとした変更程度はVS codeを使うこともあるしGitHub上で直接変更することもある。

## Sequel Ace
https://sequel-ace.com/

接続の設定はSail用に共通で一つ作ればいい。

- TCP/IP
- ホスト：`localhost` もしくは `127.0.0.1`
- ポート：3306
- ユーザーとパスワード：.envで設定したもの。デフォルトのsailとpasswordのまま。
- データベース：空

sail起動後に接続可能。データベースを指定しないことで大量のLaravelプロジェクトでも共通で使える。接続後に表示するデータベースの選択が必要。

Windowsと共通化したいならTablePlusでもいい。
https://tableplus.com/

## Docker Desktop
https://www.docker.com/products/docker-desktop/

## iTerm2もしくは他のターミナルアプリ
https://iterm2.com/

標準のターミナルのままでもいい。

ここでは書かないけどzshのカスタマイズも自分の使いやすいように行う。

## XcodeのCommand Line Tools
Laravelには関係なさそうでもHomebrewで必要だったりnode.js/npmで影響が出ることがあるのでインストール。
```shell
xcode-select --install
```

Mac App StoreからXcodeのインストールは不要なはずだけど後で問題があればインストール。

## Homebrew
https://brew.sh/index_ja

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
```shell
brew -v
```

インストール時に`.zprofile`に追記するように案内が出るけどPhpStormを使うならこれは無視する。
```shell
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/***/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```
`.zprofile`ではなく`.zshrc`に追加する。原因ははっきりしないけどインストール先が変わってPhpStormからのPHP/composerの実行に失敗することがある。`.zshrc`に追加すれば解決。
```shell
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/***/.zshrc
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Apple SiliconではHomebrewのインストール先が`/opt/homebrew/`に変わっている。旧機種から移行アシスタントで移行すると`/usr/local/`内に古いHomebrewが残っているので新しいHomebrewで再度インストールし直す。

```shell
cd ~/

# 古いbrewでBrewfileを作成
/usr/local/bin/brew bundle dump

# 新brewで再インストール
brew bundle
```

古いHomebrewでインストールしたコマンド類は削除していい。

インストール先が変わったので色々な所で設定するPATHもすべて変わってしまう。これまで当たり前のように`/usr/local/bin/php`と認識していたことが変わるので全部忘れて覚え直しが必要。

## Homebrewでいろいろインストール
「開発環境は全部Docker内に閉じ込める」なんて現実的には机上の空論。そんな使い方は不便すぎるのでDocker外の手元でもphp,composer,npmくらいは使えるようにしておく。

全部brewで最新バージョンをインストールするだけなので簡単。

### PHP
brew版は拡張も自動的にインストールされるのでLaravelで使う範囲ならほぼ困ることはないだろう。
```shell
brew install php
```
```shell
php -v
```

### Xdebug
PHPの後にpeclでインストール。

```shell
pecl install xdebug
```

PhpStormでXdebugをオンデマンドモードで使うには。  
https://pleiades.io/help/phpstorm/configuring-xdebug.html#on_demand_mode  
Laravelではテストを書くのが普通で「ステップ実行」なんて全く使わない。なのでXdebugは普段は無効化、カバレッジ付きでテストを実行する時のみ有効にすればいい。この形で使うのが一番高速。

- peclでインストール後は勝手にXdebugが有効化されるのでphp.iniを編集して無効にする。
  - `php --ini`でphp.iniの場所を確認。PHP8.2なら`/opt/homebrew/etc/php/8.2/php.ini`。1行目の`zend_extension="xdebug.so"`を削除するか`;zend_extension="xdebug.so"`で無効にする。
- PhpStormのインタープリター設定の「デバッガー拡張機能」に`xdebug.so`のパスを指定。ここはPHPのバージョンアップで変わるのでpeclでインストール後の表示を確認。例としてPHP8.2なら`/opt/homebrew/Cellar/php/8.2.0/pecl/20220829/xdebug.so`
- こういうPATHは環境やバージョンで変わるのでそのままコピペして使わない。

### composer
```shell
brew install composer
```
```shell
composer
```

`laravel/installer`をインストールするなら。最近はlaravel.build使うので入れなくてもいい。
```shell
composer global require laravel/installer

laravel
```

### node.js
```shell
brew install node
```
```shell
node -v
npm -v
```

### バージョンアップ作業
```shell
brew update
brew upgrade
```

## sailコマンドのエイリアス登録
```shell
echo "alias sail='[ -f sail ] && sh sail || sh vendor/bin/sail'" >> ~/.zshrc
```
これでsailだけで使えるようになる。
```shell
sail version
```
使える場所はcomposer install済のLaravelプロジェクト直下。

## 基本の使い方1 既存のLaravelプロジェクト
GitHubにLaravelプロジェクトがある状態から開始。

MacではWSLでの所有者の問題はないのでSourceTreeかGitHub Desktopからcloneでいい。保存先は`~/Sites/`でも`~/Documents/`でもどこでもいい。
PhpStormで開けばcomposer installやnpm installの指示が出るはず。そのままPhpStormの機能で実行してもいいし「PhpStormのターミナル」もしくは「iTerm2など」で実行してもいい。

```shell
composer install
cp .env.example .env
php artisan key:generate
# 必要なら.envを編集

npm install
npm run build

sail up -d

# migrateなどDBに接続するコマンドは必ずsail内で実行
sail art migrate
# 他のmakeコマンドなどはsail外で使ってもいい。Dockerを使わないので若干早いしsail起動前でも使える。
php artisan make:controller TestController

sail down
```

PhpStormでコードを書いて、gitはSourceTree/GitHub Desktopを使う、コマンドはターミナル、といういつもの開発体制が整う。

composerやnpmのスクリプトはPhpStormから実行。  
composer.jsonのscriptsにsailのupとdownを書いておけばsailの起動・終了もPhpStormからさっとできる。
```json
        "sail:up": "./vendor/bin/sail up -d",
        "sail:down": "./vendor/bin/sail down",
```
もう一歩進めてsail up後にide-helper:modelsも実行。DBへの接続が必要なのでup直後に毎回実行が効率的。
```json
        "sail:up": [
            "./vendor/bin/sail up -d",
            "./vendor/bin/sail art ide-helper:models -N"
        ],
```

## 基本の使い方2 Laravelプロジェクトを新規作成
Laravelのドキュメント通りなので変わった所はない。

```shell
cd ~/Sites/
curl -s "https://laravel.build/example-app" | bash
cd example-app
sail up -d
sail down
```

## テスト
テストはbrewのphpとプロジェクトにインストールしたphpunitでPhpStormの機能で実行。カバレッジ付きで実行すればコードカバレッジの表示もできる。  
毎年11月頃はPHPの新バージョンで動作確認するために「brewのPHPは8.1のままsailのPHPだけ8.2RCにしてテスト」みたいなことをするので`sail test`を使うこともある。

## ポイント
- brewでインストールしたphp, composer, npmはすべてのLaravelプロジェクト共通で使う。installやupdateに使うだけなので常に最新バージョンでいい。プロジェクトごとの分離した環境はsailで作る。DBなどは分離。
- 使うアプリも使い方も同じなのでMacとWindowsを併用しても違和感ない。
- WSLは上手く融合してるとは言えWindowsの中に強引にLinuxを入れてるから色々と気を付けることがあるけどUNIXベースになって長いMacなら何もない。

## 余談：初心者がやりがちだけどやってはいけないこと
- MAMP, phpMyAdminなどは「初心者が書いた間違った本やブログ」に騙された初心者がインストールしようとするけどLaravelユーザーが使うことはないので一切不要。
- Dockerコンテナ内で作業しない。sail登場前からDockerやHomestead(Vagrant)の内部に入ってコマンドを実行する使い方してる初心者が多い。これやると普段使ってるターミナルとは別の環境で作業することになるのでとても不便。考え方が逆。コンテナの外の使い慣れたターミナルからコンテナ内のコマンドを実行する使い方をすべき。Laravel公式のsailはこの辺り良く分かっていて`sail artisan ...`はコンテナの外で実行。コンテナ内で何か作業することは全くない。
