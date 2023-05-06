# Bootstrap5 + VSCode + LiveSassCompiler settings

## ※ git clone とその後の操作

```shell
git clone git@github.com:keijiek/bootstrap__vscode__live_sass_compiler.git
cd bootstrap__vscode__live_sass_compiler
npm i
```

---

## 1. 前提

1. node を導入しておく。特定バージョンを直にインストールするのではなく、nvm や volta 等のバージョン管理ツールを使うことをおすすめします。
1. VSCode に、拡張 LiveSassCompiler を導入。WSL や サーバーを使っているなら、そのリモート接続先に導入。

---

## 2. 目指すツリー構造

この通りじゃなくてもいいですが、そうする場合、後述する LiveSassCompilre の設定上のパスを変えてください。

```shell
tree -F --dirsfirst -a -I "\node_modules|\.git|\.vscode|\.DS_Store"
./
├── assets/
│   ├── css/
│   │   ├── styles.css
│   │   └── styles.css.map
│   ├── img/
│   ├── js/
│   │   └── bootstrap.bundle.min.js
│   ├── scss/
│   │   ├── _bootstrap.scss
│   │   └── styles.scss
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

### 3.1. bootstrap の導入

#### 3.1.a. npm を使う

```shell
npm init -y
npm i -D bootstrap
```

#### 3.1.b. 公式からダウンロードする

<https://getbootstrap.jp/> の Download から、ソースファイルをダウンロードし、プロジェクト内に配置。

---

### 3.2. ディレクトリやファイルの用意

```shell
# ディレクトリ作成
mkdir {./assets,./assets/scss,./assets/css,./assets/ts,./assets/js,./assets/img}

# bootstrap の js ファイルをコピー。bootstrap 導入方法によりコピー元のパスが異なるので注意)
# この時、bootstrap.bundle.min.js は、popper.js が含まれたもの。bootstrap.min.js は含まれないもの。いずれも jquery は別途必要。
cp ./node_modules/bootstrap/dist/js/bootstrap.bundle.min.js ./assets/js/

# scss ファイルを作成。bootstrap 導入方法により、@import のパスが異なるので注意。
echo -e "@import \"./bootstrap\";" > ./assets/scss/styles.scss
echo -e "@import \"../../node_modules/bootstrap/scss/bootstrap\";" > ./assets/scss/_bootstrap.scss
```

echo の引数:
- -e : この後の文字列でエスケープを有効にする。

※ mkdir の{}内はスペースを空けないこと。

---

## ファイルの中身

### /assets/scss/styles.scss

- _bootstrap.scss をインポートする。

```scss
@import "./bootstrap_custom";
// この後に書きたい scss 文を書いてもよい。
```

### /assets/scss/_bootstrap.scss

- bootstrap のカスタマイズはこのファイルで行う。

```scss
@import "../../node_modules/bootstrap/scss/bootstrap";
// これ以降、かつ次の @import を記述する直前までに、カスタマイズ用の記述をしなければならない。
```

### *.code-workspace に liveSassCompire の設定を記述

```json
{
  // root > settings 内に記述
  "settings": {
    // (略)
    // autoprefixer がカバーするブラウザの範囲を定義。この設定が無い場合 autoprefixer は機能しない。
    "liveSassCompile.settings.autoprefix": [
      "> 1%",
      "last 2 versions"
    ],
    // 無視するディレクトリの一覧。上2つは規定値。.git/** を足した。
    "liveSassCompile.settings.excludeList": [
      "/**/node_modules/**",
      "/.vscode/**",
      "/.git/**"
    ],
    /**
     * 設定の内容：
     * format : expanded / compressed
     * extensionName : .css / .min.css
     * savePath : css の出力先パス。ディレクトリが無いなら作る。パスの起点はプロジェクトルート。
     */
    "liveSassCompile.settings.formats": [
      {
        "format": "expanded",
        "extensionName": ".css",
        "savePath": "/assets/css"
      }
    ],
    // 参照すべきscssファイルを直接、指定できる。また、配列内で複数を指定できる。
    "liveSassCompile.settings.includeItems": [
      "/assets/scss/styles.scss"
    ]
  }
  // (略)
}

```
