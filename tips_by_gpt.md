## 結論

- 通常のCSSなら **`.user.css`**
- Stylus記法（インデント構文など）を使うなら **`.user.styl`**
- GitHubから配布するなら、`github.com/.../blob/...` ではなく **Raw URL**
- `@-moz-document domain(...)` は、スタイルを適用するサイトを指定する条件ブロック
- `@-moz-document` 自体はWeb標準としては非推奨ですが、**StylusのUserCSSでは現在も対象サイト指定の記法として使われます**

---

# 1. どのような拡張子のファイルか

Stylusには、実質的に次の形式があります。

| 拡張子 | 内容 | 推奨用途 |
|---|---|---|
| `.user.css` | UserCSS形式の通常のCSS | 最も互換性が高い |
| `.user.styl` | Stylus言語で書いたUserCSS | Stylus記法を使う場合 |
| `.user.less` | Lessで書いたUserCSS | Less記法を使う場合 |

単なる通常のCSSなら、まずは **`.user.css` を推奨**します。

例えば次のようなファイルです。

```text
my-style.user.css
```

なお、これはブラウザ拡張機能そのもののファイルではありません。Stylusが読み込む「ユーザースタイル」のファイルです。

Stylusの公式ドキュメントでは、URLからインストールする場合、URLまたはファイル名が次のいずれかで終わる必要があると説明されています。

```text
.user.css
.user.styl
.user.less
```

参考：

- [Stylus: Writing UserCSS - Installation](https://github.com/openstyles/stylus/wiki/Writing-UserCSS#installation)
- [Stylus: UserCSS](https://github.com/openstyles/stylus/wiki/Usercss)

## UserCSSの基本形式

通常のCSSに、冒頭のメタデータを追加します。

```css
/* ==UserStyle==
@name         Example Site Style
@namespace    https://github.com/USERNAME/REPOSITORY
@version      1.0.0
@description  Example site customization
@author       Your Name
@license      MIT
@homepageURL  https://github.com/USERNAME/REPOSITORY
@supportURL   https://github.com/USERNAME/REPOSITORY/issues
@updateURL    https://raw.githubusercontent.com/USERNAME/REPOSITORY/main/example.user.css
@preprocessor default
==/UserStyle== */

@-moz-document domain("example.com") {
  body {
    background: #222 !important;
    color: #eee !important;
  }
}
```

このコメント部分はCSSコメントなので、通常のCSSパーサーからは無視されます。一方、Stylusはこの部分を読み取って、名前、バージョン、更新先、設定項目などを表示します。

### Stylusの設定項目も作れる

例えば、利用者が色を変更できるようにする場合は次のようにします。

```css
/* ==UserStyle==
@name         Example Site Style
@namespace    https://github.com/USERNAME/REPOSITORY
@version      1.0.0
@license      MIT
@preprocessor default
@var color accent "アクセント色" #448aff
==/UserStyle== */

@-moz-document domain("example.com") {
  a {
    color: var(--accent) !important;
  }
}
```

Stylusの設定画面に「アクセント色」のカラーピッカーが表示されます。

---

# 2. `@-moz-document domain(){}` の意味

例えば次のコードです。

```css
@-moz-document domain("example.com") {
  body {
    background: #222 !important;
  }
}
```

これは、

> `example.com` およびそのサブドメインに該当するページで、このCSSを適用する

という意味です。

対象になる例：

```text
https://example.com/
https://www.example.com/
https://blog.example.com/article
```

`domain()` には、プロトコルやワイルドカードを付けません。

```css
/* 正しい */
@-moz-document domain("example.com")

/* 不適切 */
@-moz-document domain("*.example.com")
@-moz-document domain("https://example.com/")
@-moz-document domain("example.com:443")
```

## 他の指定方法

```css
/* URL完全一致 */
@-moz-document url("https://example.com/about") {
}

/* URLの先頭一致 */
@-moz-document url-prefix("https://example.com/blog/") {
}

/* 正規表現 */
@-moz-document regexp("https://example\\.com/users/.*") {
}
```

Stylusの公式ドキュメントでは、`domain`、`url-prefix`、`url`、`regexp` の4種類が説明されています。

参考：

- [Stylus: Applying styles to specific sites](https://github.com/openstyles/stylus/wiki/Writing-styles#applying-styles-to-specific-sites)
- [Stylus: Valid `@-moz-document` rules](https://github.com/openstyles/stylus/wiki/Writing-styles#valid--moz-document-rules)

## Stylusの通常形式ではどうなるか

Stylusの通常のセクション形式では、対象サイトはエディターの **Applies to** 欄で管理します。

そのため、通常のStylusエディターでは、次のような記述を直接表示しない場合があります。

```css
@-moz-document domain("example.com") {
  /* CSS */
}
```

Stylus内部では対象サイト情報を管理しており、Mozilla形式としてエクスポートした場合などに、この形式が現れます。

一方、**UserCSS形式では `@-moz-document` がソースコード内に残ります**。GitHubなどで配布するなら、対象サイトを明確にできるため、UserCSS形式のほうが扱いやすいです。

## 現在は非推奨なのか

### Web標準としては非推奨

はい。`@document` は現在、次の理由からWeb標準としては推奨されません。

- 非標準のCSS機能
- CSS仕様から削除された
- Chrome、Edge、Safariなどでは通常のWebページ用CSSとして利用できない
- Firefoxでも一般のコンテンツCSSでの利用は制限されている

MDNでも、`@document` は非推奨・非標準の機能として扱われています。

- [MDN: `@document`](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@document)
- [Mozilla Bug 1851097](https://bugzilla.mozilla.org/show_bug.cgi?id=1851097)

### ただし、StylusのUserCSSでは使ってよい

ここは少しややこしいところです。

```text
Webページに通常のCSSとして読み込ませる
    → @-moz-documentに依存しない

StylusのUserCSSとして配布する
    → @-moz-documentを対象サイト指定に使う
```

StylusはブラウザにそのままCSSを解釈させるだけではなく、UserCSSを解析して対象サイトを判定し、そのページにCSSを注入します。そのため、StylusのUserCSSでは `@-moz-document` が現在も実用的な記法です。

つまり、

> Web標準としては非推奨だが、StylusのUserCSSフォーマットでは現在も利用されている

という理解が適切です。

Stylus向けに公開する場合は、`@-moz-document` を削除するより、対象サイトを明示するために残すほうがよいでしょう。削除すると、スタイルが全サイトに適用される可能性があります。

---

# 3. GitHubで共有する方法

## 推奨構成

例えば、GitHubリポジトリを次のようにします。

```text
my-user-styles/
├─ README.md
├─ LICENSE
└─ styles/
   └─ example.user.css
```

リポジトリはPublicにします。

GitHub上で見るためのURLは次のようになります。

```text
https://github.com/USERNAME/REPOSITORY/blob/main/styles/example.user.css
```

しかし、この `blob` URLはHTMLページです。Stylusにインストールさせる場合は使いません。

## インストール用のRaw URL

インストール用には次のURLを使います。

```text
https://raw.githubusercontent.com/USERNAME/REPOSITORY/main/styles/example.user.css
```

例えば、

```text
https://raw.githubusercontent.com/daizu/my-user-styles/main/styles/example.user.css
```

のような形式です。

重要なのは次の点です。

```text
GitHub閲覧用:
https://github.com/USERNAME/REPOSITORY/blob/main/example.user.css

Stylusインストール用:
https://raw.githubusercontent.com/USERNAME/REPOSITORY/main/example.user.css
```

後者のRaw URLをブラウザで開くと、Stylusがインストーラー画面を表示します。

公式ドキュメントでも、GitHubでホストする場合はRaw URLを使用するよう説明されています。

- [Stylus: Hosting a UserCSS](https://github.com/openstyles/stylus/wiki/Writing-UserCSS#hosting-a-usercss)

## READMEにインストールバッジを置く

README.mdに次のようなリンクを置けます。

```markdown
[![Stylusで直接インストール](https://img.shields.io/badge/Install%20directly%20with-Stylus-00adad.svg)](https://raw.githubusercontent.com/USERNAME/REPOSITORY/main/styles/example.user.css)
```

GitHub上では、次のようなバッジになります。

```text
[ Stylusで直接インストール ]
```

利用者がバッジをクリックすると、Raw URLに移動し、Stylusのインストール画面が表示されます。

ただし、uBlock Originの

```text
https://subscribe.adblockplus.org/?location=...
```

のような専用のインストールプロトコルは、Stylusには一般的に使われていません。

そのため、Stylusの場合は次の流れになります。

```text
READMEのバッジをクリック
    ↓
Rawの.user.cssを開く
    ↓
Stylusのインストール画面
    ↓
「Install style」をクリック
```

セキュリティ上、完全に無確認でインストールする仕組みではありません。

Stylusの設定で、`.user.css` URLを開いたときにインストーラーを表示する機能を無効にしている場合は、Stylusの設定で次の項目を有効にする必要があります。

```text
Open installer when navigating to a .user.css URL
```

---

# 自動更新の設定

UserCSSのバージョンを更新しながら管理します。

```css
@version 1.0.0
```

変更を公開したら、例えば次のようにバージョンを上げます。

```css
@version 1.1.0
```

`@updateURL`を設定しておくと、更新先を明示できます。

```css
@updateURL https://raw.githubusercontent.com/USERNAME/REPOSITORY/main/styles/example.user.css
```

### 各URLの役割

```css
@homepageURL https://github.com/USERNAME/REPOSITORY
```

これはStylusの管理画面から「ホームページ」を開くためのURLです。更新先ではありません。

```css
@supportURL https://github.com/USERNAME/REPOSITORY/issues
```

これは問題報告用のURLです。

```css
@updateURL https://raw.githubusercontent.com/USERNAME/REPOSITORY/main/styles/example.user.css
```

これは自動更新で取得するURLです。

`@updateURL`を指定しない場合、通常はインストール元のURLが更新先として使われます。

## ブランチとタグの使い分け

常に最新版を配布するなら、`main`を使います。

```text
https://raw.githubusercontent.com/USERNAME/REPOSITORY/main/styles/example.user.css
```

一方、特定バージョンを固定したい場合はタグを使えます。

```text
https://raw.githubusercontent.com/USERNAME/REPOSITORY/v1.0.0/styles/example.user.css
```

ただし、タグのURLをインストール元にすると、そのタグの内容から自動では更新されません。通常は、配布用URLを`main`にして、Gitのタグでリリース履歴を管理する方法が扱いやすいです。

---

# おすすめの実運用

最初は次の形がよいと思います。

```text
my-user-styles/
├─ README.md
├─ LICENSE
└─ styles/
   └─ example.user.css
```

`example.user.css`：

```css
/* ==UserStyle==
@name         Example Site Style
@namespace    https://github.com/USERNAME/my-user-styles
@version      1.0.0
@description  Example site customization
@author       Your Name
@license      MIT
@homepageURL  https://github.com/USERNAME/my-user-styles
@supportURL   https://github.com/USERNAME/my-user-styles/issues
@updateURL    https://raw.githubusercontent.com/USERNAME/my-user-styles/main/styles/example.user.css
@preprocessor default
==/UserStyle== */

@-moz-document domain("example.com") {
  body {
    background: #222 !important;
    color: #eee !important;
  }
}
```

`README.md`：

```markdown
# My User Styles

## インストール

[![Stylusで直接インストール](https://img.shields.io/badge/Install%20directly%20with-Stylus-00adad.svg)](https://raw.githubusercontent.com/USERNAME/my-user-styles/main/styles/example.user.css)

## 対応サイト

- example.com

## ライセンス

MIT
```

通常のCSSしか使わないのであれば、`@preprocessor default` と `.user.css` の組み合わせをおすすめします。Stylus言語のインデント構文を使う場合だけ、`.user.styl` と `@preprocessor stylus`を使うとよいでしょう。
