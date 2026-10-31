---
title: "インストールできるデザインシステムパッケージ"
description: "デザインシステムパッケージは、コンポーネント、変数、ガイドをまとめた .silverpoint ドキュメントに名前、セマンティックバージョン、追加の要素を付けたものです。ドキュメントはファイルパスではなく、id とバージョン要件でリンクできます。"
summary: "デザインシステムパッケージは、コンポーネント、変数、ガイドをまとめた .silverpoint ドキュメントに名前、セマンティックバージョン、追加の要素を付けたものです。ドキュメントはファイルパスではなく、id とバージョン要件でリンクできます。"
layout: doc
toc: true
---

デザインシステムとは、ほかのドキュメントがリンクする、コンポーネント、変数、ガイドをまとめた `.silverpoint` ドキュメントです。**デザインシステムパッケージ** は、それに名前、セマンティックバージョン、任意の追加要素（スキル、スタイル、フォントペアリング、プロダクトタイプとパレット、リントルール）を付けて包んだもので、ドキュメントはファイルパスではなく **id とバージョン要件** でリンクできます。パッケージはユーザーごとのフォルダにインストールされ、複数のバージョンを並べて置けます。`com.acme.ui` を `^1.2` でリンクしたドキュメントは、インストールされている 1.2 以上の `1.x` のうち最も新しいものを使い、より新しいものがインストールされるとすぐにそれを使います。

パス形式（`imports`、パスでリンクするファイル）は変わらず、これまでどおり使えます。パッケージはその隣に並ぶ 2 つ目の形式です。

## パッケージの形式: `.silverpointds`

パッケージは、名前が `.silverpointds` で終わる **フォルダ** です（Finder ではパッケージとして扱われ、ドキュメントのようにダブルクリックで開けます）。

```
com.acme.ui-1.2.0.silverpointds/
  manifest.json          required
  library.silverpoint/   required: the design system, a normal .silverpoint package
  skills/*.md            optional: skills, in the user-skills format (docs/skills.md)
  styles/*.md            optional: style presets, in the style-preset format (docs/templates-and-styles.md)
  pairings/*.md          optional: font pairings, in the font-pairing format (docs/templates-and-styles.md)
  products/*.md          optional: product type tables, in the bundled products table's format (docs/templates-and-styles.md)
  palettes/*.md          optional: product family palettes, one <family>.md each (docs/templates-and-styles.md)
  lint/*.json            optional: custom lint rules, in the lint-rules format (docs/lint-rules.md)
  README.md              optional: for people reading the package
```

このフォルダの zip アーカイブには、まだ **対応していません**。フォルダそのものが形式です（zip はバックログの「残り」に載っています）。

### `manifest.json`

```json
{
  "id": "com.acme.ui",
  "name": "Acme UI",
  "version": "1.2.0",
  "description": "Acme's buttons, inputs and tokens",
  "author": "Acme Design",
  "homepage": "https://acme.example/ui",
  "minimumSilverpointVersion": "0.1.0"
}
```

| フィールド | 必須 | ルール |
| --- | --- | --- |
| `id` | はい | 逆 DNS 形式かスラッグ。小文字の英字と数字をドットかダッシュでつなぎます（`[a-z0-9]+([.-][a-z0-9]+)*`）。例: `com.acme.ui`、`acme-ui`。 |
| `name` | はい | アプリに表示されるテキスト。 |
| `version` | はい | 厳密なセマンティックバージョン。`MAJOR.MINOR.PATCH` に、任意で `-prerelease` と `+build` を付けます。例: `1.2.0`、`2.0.0-beta.1`。`1.2` や `v1.2.0` は受け付けません。 |
| `description` | はい | 1 行。 |
| `author`、`homepage` | いいえ | テキスト。 |
| `minimumSilverpointVersion` | はい | それを使える最も古い Silverpoint（セマンティックバージョン）。実行中のものより新しい Silverpoint を必要とするパッケージは、インストール時に拒否されます。 |

検証エラーはどれも、そのフィールドを名指しします（`manifest.json: missing “version”`）。

## インストール、一覧、削除

インストールしたパッケージは `~/Library/Application Support/Silverpoint/Design Systems/<id>/<version>/` に置かれます。別のフォルダを使うには `SILVERPOINT_DESIGN_SYSTEMS_DIR` を設定します（アプリ、CLI、MCP サーバーはすべて同じフォルダを読みます）。

- **アプリで:** `.silverpointds` をダブルクリックします（または ファイル ▸ 開く…）。Silverpoint は名前、id、バージョンを表示し、インストールする前に確認します。キャンセルすると何も書き込みません。**設定 ▸ 一般 ▸ パッケージ** には、インストールされているすべてのバージョンが、id、名前、デフォルトのバッジとともに一覧されます。**使用中**（そのバージョンにリンクしている開いたデザインの数。ヘルプにはその名前が出ます。閉じているドキュメントは確認しません）と、開いているデザインがインストール済みのものより古いバージョンを使っているときは **新しいバージョンがインストール済み** も表示されます（情報のみで、リンクし直しはしません）。**インストール…** は選んだ `.silverpointds` をコピーします（選ぶことが同意になります。同じ id とバージョンをもう一度インストールしようとすると拒否されます）。**削除…** は先に確認し、開いているデザインがそのバージョンを使っていれば警告します。**フォルダを表示** はパッケージのフォルダを開き、**再読み込み** はそれを読み直します。各パッケージのガイドと **デフォルトとして使用** は、引き続き「デザインシステムを管理…」にあります（設定 ▸ 一般 ▸ デフォルトのデザインシステム ▸ **管理…**）。このセクションと同じ一覧を共有しているので、一方での変更はもう一方にも表示されます。
- **コマンドラインで:**

```
silverpoint ds install <file.silverpointds>        installed <id> <version>
silverpoint ds list [--json]                       <id><TAB><version><TAB><name>, sorted by id, then version
silverpoint ds remove <id> [<version>]             removed <id> <version>, once per version removed
silverpoint ds outdated <file.silverpoint>         the document's package links with a newer installed version
silverpoint ds pack <library.silverpoint> --id <id> --version <semver>
                [--name <n>] [--description <d>] [--author <a>] [--homepage <url>] [--min-version <semver>]
                [--skills <dir>] [--styles <dir>] [--pairings <dir>] [--products <dir>] [--lint <dir>] [--readme <file>]
                [--out <path>]
```

同じ id とバージョンを 2 回インストールすると失敗し（`already installed`）、何も変わりません。1 つの id の異なるバージョンは並べて置けます。失敗すると終了コード 1、使い方の誤りでは終了コード 2 で終わります。

## パッケージにまとめる

`silverpoint ds pack` は、ライブラリと、指定した追加要素のフォルダ（`*.md` のスキル、スタイル、フォントペアリング、プロダクトタイプ、`*.json` のリントルール）からパッケージのフォルダを作り、書き出した場所を表示します（デフォルトは `./<id>-<version>.silverpointds`）。上書きは決してしません。`--products <dir>` には、ユーザーのプロダクトフォルダと同じ構成のフォルダを渡します。その `*.md` の表は `products/` へ、`palettes/` サブフォルダの `*.md` のパレットは `palettes/` へ入るので、`--products ~/Library/Application\ Support/Silverpoint/Products` とすれば手元のものをそのまままとめられます（別の `--palettes` はありません）。フラグで指定したフォルダが存在しなければエラーになり、何も書き込みません。アプリでは、種類がデザインシステムのドキュメントに対して **ファイル ▸ デザインシステムパッケージとして書き出す…** が同じことをします。id、バージョン、名前を尋ね、続いて保存場所を尋ねます。（ファイル ▸ デザインシステムのトークンとコードを書き出す… は別の書き出しで、トークンとコンポーネントのコードをファイルとして書き出します。）

## パッケージでリンクする

ドキュメントは、パッケージのリンクを `imports` の隣の `packages` に保存します。

```json
"packages": { "acme": { "id": "com.acme.ui", "version": "^1.2" } }
```

キーはパスでのリンクと同じくエイリアスで、コンポーネントは `acme:btn` になります。エイリアスは `imports` と `packages` を通して一意です。パスでのリンクしかないドキュメントには `packages` キーがなく、これまでとバイト単位で同じに保存されます。

- **リンクする:** エージェントと MCP のツール `link_design_system` は `{"package": "com.acme.ui", "version": "^1.2", "alias": "acme"}` を受け取ります（`path` と `package` のどちらか一方だけを指定します。`version` がなければ、インストールされている最も新しいバージョンに `^` を付けてリンクします。インストールされていないパッケージや、要件を満たすバージョンがインストールされていないパッケージはエラーになり、ドキュメントは変わりません）。アプリでは、インスペクタのデザインシステムの **+** メニューにインストール済みのパッケージが一覧されます。設定で、新しいドキュメントのデフォルトにするパッケージを指定できます（`com.acme.ui ^1.2`）。
- **読む:** `read_design_system` にエイリアスを渡すと、パッケージの id と使用中のバージョンが表示されます。

### バージョン要件

| 要件 | 受け付けるもの |
| --- | --- |
| `1.2.3` または `=1.2.3` | ちょうどそのバージョン（**ピン留め**） |
| `^1.2` | `>=1.2.0 <2.0.0`。`^0.2` は `>=0.2.0 <0.3.0`（左端のゼロでない部分は変わりません） |
| `~1.2.3` | `>=1.2.3 <1.3.0`。`~1` は `>=1.0.0 <2.0.0` |
| `>=1.2 <2.0` | 比較演算子 `>=`、`>`、`<=`、`<`、`=` をスペースでつないだもの。すべてを満たす必要があります |
| `1.2`、`1.x` | その系列の任意のパッチ（またはマイナー） |
| `*` | 任意のリリース |

要件の中のバージョンは一部だけでもかまいません。プレリリース（`1.3.0-beta.1`）が要件を満たすのは、要件が同じ `major.minor.patch` のプレリリースを指定しているときだけです。形式の誤った要件は、そのリンクの問題として扱われ、ドキュメントを止めるエラーにはなりません。

### 解決とアップグレード

リンクは、**要件を満たすインストール済みの最も新しいバージョン** に解決されます。ドキュメントを開いたときと、リンクやインストール済みのパッケージが変わるたびに解決し直します（アプリは数秒ごとに確認し、CLI はドキュメントを開くたびに解決します）。`1.3.0` をインストールすると `^1.2` のドキュメントはそれを使い、削除すると `1.2.0` に戻ります。完全に一致するピン留めはそのバージョンのままです。`silverpoint ds outdated <doc>` は、インストール済みの最新バージョンが要件の外にあるリンクごとに `alias<TAB>id<TAB>requirement<TAB>resolved<TAB>latest` を表示し、インスペクタはパッケージの id と解決されたバージョンの下に同じことを `update available: 2.0.0` として表示します。

### パッケージがないとき

インストールされていないパッケージや、要件を満たすバージョンがインストールされていないパッケージは、**致命的ではない問題** です。ドキュメントは開き、そのライブラリのインスタンスは、パスでリンクしたライブラリがないときとまったく同じように描画されます。問題には id、要件、理由（`not installed`、または `no installed version satisfies the requirement (installed: …)`）が示されます。これはインスペクタに表示され、エージェントのデザインシステムのコンテキストに一覧され、`read_design_system` は失敗してこれを返し、`silverpoint render`、`call`、`mcp` は標準エラーに `silverpoint: …` の行として出力します（読み込めないパスのリンクも同じように報告されます）。それ以外に終了ステータスへの影響はありません。

## 追加要素の働き

ドキュメントがパッケージにリンクしている間（その間だけ）、パッケージの追加要素は独自の `package` レイヤーとして、解決済みの一覧に加わります。

- **スキル:** バンドル < **パッケージ** < ユーザー < ドキュメント。バンドルと同じスラッグのパッケージのスキルはそれを上書きし（`silverpoint skills <doc>` は `package (overrides bundled)` と表示します）、同じスラッグのユーザーのスキルはパッケージのものに優先します。
- **スタイル:** バンドル < **パッケージ** < ユーザー。
- **フォントペアリング:** バンドル < **パッケージ** < ユーザー。
- **プロダクトタイプとパレット:** バンドル < **パッケージ** < ユーザー。プロダクトはスラッグごと、パレットはファミリーごとです。`get_style {product}` からも見えます。
- **リントルール:** パッケージの `lint/*.json` はライブラリのルールとして数えられ、ソースは `library:<alias>` になります。

パッケージのリンクを外すと、これらはなくなります。インストールするだけでは何も加わりません。パッケージの README は人が読むためのもので、Silverpoint は読みません。

## まだできないこと

共有のレジストリやリモートからのインストール（URL やカタログ）、パッケージの署名と検証、zip アーカイブは残っています。`docs/BACKLOG.md` を参照してください。
