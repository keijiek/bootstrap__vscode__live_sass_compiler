# Bootstrap5 + VSCode with LiveSassCompiler, settings

vite を使って開発環境を作ろうと思いましたが、その方法だと、ソースの html を編集する時に bootstrap の class 名を拾ってくれないので、先に scss だけコンパイルする方法を採用することにしました。

## ※ git clone とその後の操作

忘れずに node modules を入れてください。

```shell
git clone git@github.com:keijiek/bootstrap__vscode__live_sass_compiler.git
cd bootstrap__vscode__live_sass_compiler
npm i
```

---

## ※ @import ではなく @use を使う場合

bootstrap 公式の解説文には @import を使う方法が記されているものの、sacc 公式から廃止されると公言されて久しいので @use を使っておきたいところ。  
そうする場合、次のように`with ()`を使うのが簡単で良さそうです。  
@import だと、パラメータ変更と import の順番に気を使うけど、@use なら同時なので、こっちの方が良い気がします。とはいえ、bootstrap 公式がこのように解説しない重大な理由があるのかも…

```shell
# assets/scss/_bootstrap_custom.scss
@use "../../node_modules/bootstrap/scss/bootstrap" with (
  $primary: rgba(0, 128, 0, 1),
  $secondary: rgba(0, 0, 255, 1)
);
```

---

## 1. 前提

1. node を導入しておく。特定バージョンを直にインストールするのではなく、nvm や volta をはじめとした、何らかのバージョン管理ツールを使うことを強くおすすめします。
1. VSCode に、拡張 LiveSassCompiler を導入。WSL や サーバーを使っているなら、そのリモート接続先に導入します。

---

## 2. 目指すツリー構造

この通りじゃなくてもいいですが、そうする場合、後述する LiveSassCompilre の設定上のパスを変えてください。

```shell
tree -F --dirsfirst -a -I "\node_modules|\.git|\.vscode|\.DS_Store"
./
├── assets/
│   ├── css/
│   │   └── styles.min.css : (コンパイル後のcss。html から読み込む)
│   ├── img/
│   ├── js/
│   │   ├── bootstrap.bundle.min.js : (bootstrap の js)
│   │   └── jquery.min.js : (bootstrap 使用の前提となる jquery。WordPress を使うなら不要。)
│   ├── scss/
│   │   ├── _bootstrap_custom.scss : (bootstrap のパラメータ変更はここに書く。)
│   │   └── styles.scss : (scss のエントリーポイント。)
│   └── ts/
├── .gitignore
├── README.md
├── bootstrap__vscode__live_sass_compiler.code-workspace
├── index.html
├── package-lock.json
└── package.json
```

なお、tree の引数の意味は次の通り。
- -F : ディレクトリの末尾に / を付ける。個人的に必須。
- --dirsfirst : ファイルより先にディレクトリを表示する。個人的に必須。
- -a : 隠しファイル(.gitignoreなど)も表示する。そのうえで表示したくないもの(.git/など)は -I で省く。
- -I : で無視(ignore)するものを「パターン」で記述。複数ある場合は | でつなぐ。

## 3. 手順

### 3.1. bootstrap, jquery の導入

#### 3.1.a. npm を使う

- ただし、WordPress を使う予定なら JQuery は不要。デフォルトで読み込んでいるらしい。

```shell
npm init -y
npm i -D bootstrap jquery
```

#### 3.1.b. 公式からダウンロードする

- <https://getbootstrap.jp/> の Download から、bootstrap ソースファイルをダウンロードし、bootstrap.bundle.min.js だけプロジェクト内に配置。
- <https://jquery.com/> の Download から、jquery の compressed な production バージョンをダウンロードし、jquery.min.js だけをプロジェクト内に配置。

---

### 3.2. ディレクトリやファイルの用意

```shell
# ディレクトリ作成
mkdir {./assets,./assets/scss,./assets/css,./assets/ts,./assets/js,./assets/img}
# 別の書き方
mkdir ./assets && mkdir ./assets/{scss,css,ts,js,img}

# bootstrap の js ファイルをコピー。bootstrap 導入方法によりコピー元のパスが異なるので注意)
# この時、bootstrap.bundle.min.js は、popper.js が含まれたもの。bootstrap.min.js は含まれず、別途必要。いずれも jquery は別途必要。
cp ./node_modules/bootstrap/dist/js/bootstrap.bundle.min.js ./assets/js/
cp ./node_modules/jquery/dist/jquery.min.js ./assets/js

# scss ファイルを作成。bootstrap 導入方法により、@import のパスが異なるので注意。
echo -e "@import \"./bootstrap\";" > ./assets/scss/styles.scss
echo -e "@import \"../../node_modules/bootstrap/scss/bootstrap\";" > ./assets/scss/_bootstrap_custom.scss

# use を使う場合
echo -e "@use  \"./bootstrap_custom\" as bootstrap_custom;" > ./assets/scss/styles.scss
echo -e "@use \"../../node_modules/bootstrap/scss/bootstrap\" with (\n  \$primary: rgba(0, 128, 0, 1),\n  \$secondary: rgba(0, 0, 255, 1)\n);" > ./assets/scss/_bootstrap_custom.scss
```

echo の引数:
- -e : この後の文字列でエスケープを有効にする。
  - `\"` : 単なる文字列としてのダブルコート
  - `\n` : 改行コード
  - `\$` : エスケープして変数扱いを防いでいる

※ mkdir の{}内はスペースを空けないこと。  

---

## ファイルの中身

- `/assets/scss/styles.scss` : scss のエントリーポイント。他に必要なスタイルがあれば、書き足したりインポートしたりする。
- `/assets/scss/_bootstrap_custom.scss` : bootstrap のカスタマイズを行うファイル。
- `/*.code-workspace` : VSCode の拡張の設定もここに書く。そうする事で「ワークスペース」を変更を限定できる。「ユーザー」や「リモート」の設定を変えてしまわないよう注意。

### /assets/scss/styles.scss

- import の場合

```scss
@import "./bootstrap_custom";
// この後に、自作のスタイルを書き足したり、他のscssをインポートしたりする。
```

- use の場合

```scss
@use  "./bootstrap_custom" as bootstrap_custom;
// 名前空間はなんでもいいはず。as * とかでも。
```

### /assets/scss/_bootstrap_custom.scss

- import の場合

```scss
// 変更を加えたいパラメータを持つインポート対象を import するより先に、パラメータと変更後の値を定義しておく。
  $primary: rgba(0, 128, 0, 1);
  $secondary: rgba(0, 0, 255, 1);
@import "../../node_modules/bootstrap/scss/bootstrap";
```

- use の場合

```scss
// 変更を加えたいパラメータを持つインポート対象に、変更を with で渡す。
@use "../../node_modules/bootstrap/scss/bootstrap" with (
  $primary: rgba(0, 128, 0, 1),
  $secondary: rgba(0, 0, 255, 1)
);
```

### *.code-workspace に liveSassCompire の設定を記述

```json
{
  // root > settings 内に記述
  "settings": {
    // (略)
    "liveSassCompile.settings.autoprefix": [
      "> 0.5%",
      "last 2 versions"
    ],
    "liveSassCompile.settings.excludeList": [
      "/**/node_modules/**",
      "/.vscode/**",
      "/.git/**"
    ],
    "liveSassCompile.settings.formats": [
      {
        "format": "expanded",
        "extensionName": ".min.css",
        "savePath": "/assets/css"
      }
    ],
    "liveSassCompile.settings.includeItems": [
      "/assets/scss/styles.scss"
    ],
    "liveSassCompile.settings.generateMap": false,
  }
}

```
### liveSassCompile.settings の設定と値

#### autoprefix

- autoprefixer がカバーするブラウザの範囲を定義。この設定が無い場合 autoprefixer は機能しない。
- 設定の値は、<https://browsersl.ist/> と同じ。

#### excludeList

- 無視するディレクトリを配列状に書く。`"/**/node_modules/**"`と`"/.vscode/**",` は規定値。

#### formats

- `format` : 出力する css のミニファイ。`expanded`=しない / `compressed`=する。
- `extensionName` : 出力する css ファイルの拡張子。参照元(html側のlink)を書き換えないことを思えば、`.css` または `.min.css `になると思う。
- `savePath` : css の出力先ディレクトリのパス。ディレクトリが無いなら自動生成。パスの起点はプロジェクトルート。

#### includeItems

- ピンポイントでコンパイルしたい scss ファイルを指定する。複数ある場合は配列の要素として追加していく。

#### generateMap

map ファイルを出力するか？ true = する

---
