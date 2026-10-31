---
title: "コマンドプラグイン"
description: "コマンドプラグインは、Swift を書かずにアプリをビルドし直すこともなく、Silverpoint にコードターゲット、エクスポーター、インポーターを追加します。plugin.json マニフェストとコマンドを入れたフォルダで、Silverpoint は JSON のリクエストを 1 つ stdin に渡し、JSON の答えを 1 つ stdout から読み取ります。"
summary: "コマンドプラグインは、Swift を書かずにアプリをビルドし直すこともなく、Silverpoint にコードターゲット、エクスポーター、インポーターを追加します。plugin.json マニフェストとコマンドを入れたフォルダで、Silverpoint は JSON のリクエストを 1 つ stdin に渡し、JSON の答えを 1 つ stdout から読み取ります。"
layout: doc
toc: true
---

コマンドプラグインは、Swift を書かずにアプリをビルドし直すこともなく、Silverpoint にコードターゲット、エクスポーター、インポーターを追加します。
中身は `plugin.json` マニフェストとコマンドを入れたフォルダです。Silverpoint はコマンドを実行して JSON のリクエストを 1 つ stdin に渡し、
JSON の答えを 1 つ stdout から読み取ります（プロトコル 1、後述）。コードターゲットは、Flutter や独自のコンポーネント
マークアップのようなプラットフォームを `export_code`、コードの書き出しパネル、ハンドオフに追加します。エクスポーターは **ファイル ▸ 別の形式で書き出す** に項目を追加し、インポーターは
**ファイル ▸ 読み込む** に項目を追加します。`import_file` ツールが実行するのもインポーターです。

プラグインはあなたの Mac でコードを実行するので、**プラグインはあなたが承認してから初めて実行され**、変更されるたびにもう一度承認が必要です
（[信頼](#trust-and-approval)を参照）。

ロジックは SilverpointKit のもの（`SilverpointCodegen/Plugins`）です。`PluginManifest`、`PluginFolder`、`PluginModelJSON`、
`PluginHost`（プラグインを実行する唯一の場所）、`PluginPermissions` と `PluginEnvironment`（権限によって渡せるもの）、
`CommandCodeTarget`、`PluginExport`、`PluginImport`、`PluginMenu`、`PluginCheck` があります。`import_file` は `DesignTools` にあり、CLI は `silverpoint plugins` です。

## プラグインの置き場所

プラグインフォルダの中に、プラグインごとに 1 つのフォルダを置きます。`$SILVERPOINT_PLUGINS_DIR` が設定されていればそこ（`ProcessEnvironment` 経由で読み、
改名前の `VELLUM_PLUGINS_DIR` をフォールバックにします）、なければ `~/Library/Application Support/Silverpoint/Plugins/<id>/` です。インストールは
**設定 ▸ 一般 ▸ プラグイン ▸ インストール…**（プラグインのフォルダを尋ねます）、`silverpoint plugins install <folder>`、または
フォルダをコピーして入れる方法があります。インストールしたプラグインは、あなたが承認するまで承認されていません。隠しフォルダはスキップします。マニフェストが
検証を通らないフォルダや、同じ id を宣言する 2 つのフォルダは無効として一覧に出て（`silverpoint plugins list`）、実行されることはありません。

ライブラリのコードには既定のフォルダがありません（`PluginSetup.none`）。標準のフォルダを渡すのはエントリーポイント（アプリ、`silverpoint mcp`、`call`、`tools`）だけなので、
テストは常に一時フォルダを使います。

コードターゲットのプラグインは、アプリの起動時に一度だけ登録されます（`silverpoint mcp`、`call`、`tools` の起動時も同じです）。インストール
や削除をしたら、再起動するとターゲットの一覧に反映されます。エクスポーターとインポーターは、メニューを開くたびと
`import_file` を呼ぶたびに読み直します。実行のたびにプラグインをフォルダから読み直すので、起動後に編集したプラグインは
今の状態でチェック（そして実行）されます。

## マニフェスト

`plugin.json`:

```json
{
  "protocol": 1,
  "id": "flutter",
  "name": "Flutter (skeleton)",
  "version": "0.1.0",
  "kind": "codeTarget",
  "command": ["/usr/bin/python3", "generate.py"],
  "timeout": 30,
  "fileExtensions": ["dart"],
  "permissions": ["library:read"],
  "capabilities": {"dropShadow": "full", "imageFill": "approximated", "innerShadow": "unsupported"}
}
```

| フィールド | 種類 | 意味 |
| --- | --- | --- |
| `protocol` | すべて | `1` でなければなりません。 |
| `id` | すべて | 英小文字、数字、`-` で、英字か数字で始まります。Silverpoint でのコードターゲットの id は `plugin:<id>` なので、組み込みのものを隠すことはありません。 |
| `name`、`version` | すべて | 空でない文字列です。名前はメニューやターゲットの一覧に表示されます。 |
| `kind` | すべて | `codeTarget`、`exporter`、`importer` のいずれかです。 |
| `command` | すべて | 空でない argv です。`command[0]` は絶対パスでなければプラグインフォルダを基準に解決されます（`./run.sh`、`/usr/bin/python3`）。ほかの引数はそのまま渡されます。コマンドはプラグインフォルダを作業ディレクトリとし、[権限](#permissions)が許す環境変数で実行されます。シェルは介しません。 |
| `timeout` | すべて | 秒数（0 より大きく、小数も可）。これより長くかかった実行は停止されます。 |
| `capabilities` | codeTarget（必須、`{}` でも可） | `CodeFeature` 名 → `full`、`approximated`、`unsupported`。組み込みのターゲットが宣言するのと同じ機能の一覧です（後述）。 |
| `fileExtensions` | codeTarget（必須）、exporter | ドットなしのファイル拡張子です。コードターゲットでは最初のものがメインです。 |
| `fileTypes` | importer（必須） | 読み取る拡張子です（大文字小文字を区別せずに照合します）。 |
| `renders` | exporter | `true` にすると、トップレベルのフレームごとの PNG が送られます。 |
| `permissions` | すべて（省略可） | プラグインが必要とするスコープです。`library:read`、`comments:read`、`library:write`、`environment`（[権限](#permissions)を参照）。省略するとレガシーのプラグインになり、すべてを受け取ります。 |

検証は厳密で、エラーにはフィールド名が入ります（`capabilities.sparkles: "sparkles" is not a code feature`）。`id` がない、または
形式が正しくない、`kind` が 3 つ以外、`command` が空、`timeout` が 0 以下、`protocol` が 1 以外、未知の
機能（またはサポートレベル）、`fileExtensions` のないコードターゲット、`fileTypes` のないインポーターは、どれも拒否されます。
`permissions` が文字列の配列でない場合や、未知のスコープ、重複したスコープ、その種類では宣言できないスコープを含む場合も
同じです（`permissions[1]: "comments:read" is not valid for a code target; a code target can declare library:read, environment`）。

コードの機能（`CodeFeature`）: `linearGradient`、`radialGradient`、`angularGradient`、`imageFill`、`dropShadow`、`shadowSpread`、
`innerShadow`、`layerBlur`、`backgroundBlur`、`strokeWidthProfile`、`nonSolidStroke`、`textRuns`、`textPath`、`strokeDash`、
`strokeAlign`、`contentStroke`、`multipleFills`、`multipleStrokeFills`、`nonSolidShapeFill`、`gradientOpacity`、`imageOpacity`、
`iconEffects`、`vectorEffects`、`unmappedIcon`、`alphaMask`、`luminanceMask`、`vectorMask`、`layerBlendMode`、`paintBlendMode`、
`maskInLayout`、`wrapFill`、`rowAlignment`、`wrap`、`link`、`textCase`、`paragraphSpacing`、`paragraphIndent`、`textList`、
`leadingTrim`、`fontVariations`。

<h2 id="permissions">権限</h2>

プラグインは必要なものを `permissions` で宣言し、Silverpoint はそれだけを送ります。スコープは次のとおりです。

| スコープ | 種類 | 効果 |
| --- | --- | --- |
| `library:read` | コードターゲット、エクスポーター | コードターゲットにはモデルの `tokens`（変数、テーマ、テキストスタイル）が、エクスポーターにはドキュメントの `variables`、`themes`、`textStyles`、`layerStyles`、`imports`、`packages` が送られます。 |
| `comments:read` | エクスポーター | エクスポーターにドキュメントの `comments` が送られます。 |
| `library:write` | インポーター | インポーターのフラグメントで `variables`、`textStyles`、`layerStyles`、`themes` を追加できます。これがないと、それらを含むフラグメントは拒否されます。実行はプラグインと `library:write` を示して失敗し、何も読み込まれません。 |
| `environment` | すべての種類 | コマンドは API キーを含め、Silverpoint の環境変数をすべて引き継ぎます。これがないと、下の許可リストだけを受け取ります。 |

- **`permissions` キーがない場合: レガシー。** スコープができる前と同じく、プラグインはすべてを受け取ります。リクエスト全体、Silverpoint の
  環境変数すべて、そしてインポーターのフラグメントはライブラリごと統合されます。どの画面でもそう表示されます:
  「権限の宣言なし：環境変数を含め、すべてを受け取ります」。
- **`[]`** は 4 つのどれも宣言しません（「追加の権限なし」）。それでもコードターゲットは画面とコンポーネントを、エクスポーターは
  ノードを受け取り、インポーターはノードを追加できます。
- **一覧を宣言したエクスポーターが受け取るもの。** 常に: `version`、`children`（ノードを選択しているときは選択中のノード）、
  `kind`、`startScreen`、`guide`、`features`、そして `renders` フラグに応じた `renders`。`library:read` があれば: `variables`、
  `themes`、`textStyles`、`layerStyles`、`imports`、`packages`。`comments:read` があれば: `comments`。何を宣言しても受け取らないもの:
  `hooks`、`skills`、`lintRules`、`disabledLintRules`、`origins`（デザインの内容ではなく Silverpoint 自身の設定）、そして
  今後の Silverpoint が追加するトップレベルのキー。
- **`library:read` のないコードターゲット** は、`tokens` を `{"variables": [], "themes": {}, "textStyles": [], "unresolvedColors": []}` として受け取るので、
  モデルはプロトコル 1 の形を保ちます。ノードは変わりません。変数から来た色や値には変数名が残るので
  （`{"token": "brand"}`）、送られていないトークンをコードから参照できます。
- **インポーターのリクエスト**（`path`、`bytes`）は、スコープの有無にかかわらず同じです。答えに何を追加できるかは `library:write` で決まります。
- **環境変数の許可リスト**（`environment` がない場合）: `PATH`、`HOME`、`USER`、`LOGNAME`、`SHELL`、`TMPDIR`、`LANG`、`TERM`
  とすべての `LC_*`。それぞれ Silverpoint が持っているときだけ値をコピーし、何も追加しません。そのため `ANTHROPIC_API_KEY`、
  `OPENAI_API_KEY`、`SILVERPOINT_*` など、Silverpoint が引き継いだほかのものがプラグインに届くことはありません。
- **適用する場所は 1 つ。** `PluginHost` が、アプリ、エージェント、MCP、`call`、`plugins check` からのすべての実行で、
  リクエスト（`PluginPermissions`）と環境変数（`PluginEnvironment`）を絞り込みます。使うのは実行時にフォルダから読み直したマニフェストです。
  `plugin.json` は承認のハッシュに含まれるので、**`permissions` を編集するとプラグインは `changed` になり、もう一度承認を求めます**。
- 承認のアラート、**設定 ▸ 一般 ▸ プラグイン ▸ 確認…**、`silverpoint plugins list` と `check` は、スコープを
  言葉で 1 行ずつ表示します: 「変数、スタイル、テーマの読み取り」「コメントの読み取り」「変数、スタイル、テーマの追加」
  「環境変数の使用（API キーを含む）」。
- スコープが変えるのは送る内容で、プロトコルではありません。プロトコルはバージョン 1 のままで、レガシーのプラグインのリクエストは以前とバイト単位で同じです。

## プロトコル（バージョン 1）

実行はどれも 1 つのプロセスです。Silverpoint は JSON オブジェクトを 1 つ stdin に書いて閉じ、プロセスが終了するまで
stdout から JSON オブジェクトを 1 つ読み取ります。stderr はエラーメッセージ用に保持します（最後の数行）。JSON はキーをソートし、スラッシュをエスケープせずに書きます。

### コードターゲット

stdin:

```json
{"protocol": 1, "kind": "codeTarget", "model": <the UIModel JSON>}
```

stdout:

```json
{
  "files": [{"path": "lib/home.dart", "contents": "…"}],
  "diagnostics": [{"feature": "innerShadow", "nodeID": "card", "message": "left out", "severity": "warning"}]
}
```

- `files` は必須です（空でも構いません）。各 `path` は呼び出し側が書き込む場所（`export_code` のパスのフォルダ、
  コードの書き出しパネルのフォルダ、ハンドオフの `code/`）からの相対パスです。空のパス、絶対パス（または `~` で始まるもの）、`..`
  を含むパス、2 回出てくるパスがあると実行全体が失敗し、何も書き込まれません。
- `diagnostics` は省略できます。`feature` は `CodeFeature` 名、`nodeID` はデザインのノード（`UINode.sourceID`、モデルの
  `id`）、`message` はターゲットが代わりにしたことです。`severity` は `info`、`warning`（既定）、`error` のいずれかです。これらは
  `export_code` から `{nodeId, feature, action}` として返り、メッセージが action になります。
- **Silverpoint は独自の診断を追加します**: マニフェストが `full` や `approximated` と宣言していない機能を使うノードには、
  どれも 1 つ付きます（`not declared by the … plugin` または `… does not support it`）。ただし、そのノードと機能をプラグイン自身が
  報告した場合は除きます。そのため、何も宣言しないプラグインでも正直な診断が出ます。
- モデルに `annotate: true` があるとき（`export_code` の `annotate`）、ターゲットはコードにノード id を入れるようにします
  （`// silverpoint:id=<id>`）。プラグインを通してコードを読み戻すことはサポートしていません。

### エクスポーター

stdin:

```json
{"protocol": 1, "kind": "exporter", "document": <document JSON>, "renders": [{"frameID": "home", "png": "<base64>"}]}
```

`document` は Silverpoint の JSON スキーマで表したドキュメント（`.silverpoint` パッケージの `document.json`）です。ノードを
選択しているときは、そのノードだけを `children` にした同じドキュメントになります。[権限](#permissions)を宣言している場合は、
それが許すキーだけが残ります。`renders` はマニフェストで `"renders": true` を設定したときだけ入ります。トップレベルのフレーム
（選択があるときはその中の）ごとに 2× の PNG が 1 つずつ、読む順に並びます。

stdout: `{"files": [{"path", "contents"}]}`。上と同じようにチェックされ、ユーザーが選んだフォルダの下に書き込まれます（ファイル ▸ 別の形式で書き出す
がフォルダを尋ねます）。内容はテキストです。

### インポーター

stdin:

```json
{"protocol": 1, "kind": "importer", "path": "/abs/path/to/input.json", "bytes": "<the file, base64>"}
```

stdout:

```json
{"fragment": {"children": [{"type": "frame", "name": "Card", "width": 320, "height": 200, "children": []}]}, "report": {"frames": 1}}
```

- `fragment` は Silverpoint の JSON スキーマ（`schema` スキル）で表したドキュメントです。その `children` が新しいトップレベルのノードになり、
  マニフェストが `library:write` を宣言している（または `permissions` がまったくない）ときは、`variables`、`textStyles`、`layerStyles`、`themes`
  を含めることもできます（`library:write` のない一覧を宣言している場合、そうしたフラグメントは拒否されます）。デコードにはドキュメントと同じデコーダ
  （`DesignDocument.decode`）を使います。デコーダが受け付けないフラグメントは、プラグインと問題点を示して実行が失敗し、何も変わりません。
  空のフラグメント（ノード、変数、テキストスタイルがない）も拒否されます。
- フラグメントは **1 回の取り消しステップ** として追加されます。トップレベルのノードはドキュメントのノードのあとに入り（すでに使われているノード id には新しい
  id が付き、それを指す `ref` も追従します）、変数、スタイル、テーマの値は、ドキュメントにまだないものが追加されます
  （既存のものはそのまま残ります）。
- `report`（省略可、オブジェクト）は、そのまま `import_file` の結果に渡されます。

### UIModel JSON

`model` は中立的な `UIModel`（`PluginModelJSON`）で、組み込みかどうかにかかわらず、すべてのコードターゲットがここからコードを生成します。プロトコル 1 の
間は安定しています。キーの名前が変わったり削除されたりすることはなく、省略可能な値は `null` と書かずに省き、数値は小数点以下
6 桁に丸め、バイト列はゴールデンファイルで固定しています（`Tests/SilverpointCodegenTests/Fixtures/Plugin/codeexport.uimodel.json`、
`PluginUIModelJSONTests`）。トップレベルのキー:

| キー | 値 |
| --- | --- |
| `protocol` | `1`。 |
| `title` | 最初に要求されたノードの名前、または `Silverpoint export`。 |
| `annotate` | コードに `silverpoint:id` コメントを入れるかどうか。 |
| `screens` | 要求されたノード（指定がなければすべてのトップレベルのノード）をノードとして。 |
| `components` | それらが使うコンポーネント（依存するものが先）: `{id, name, variant, parameters: [{name, kind (text, boolean, slot), defaultValue, component?}], root: <node>}`。 |
| `tokens` | `{variables: [{name, type, values: [{value, theme, alias?, expression?}]}], themes: {axis: [values, default first]}, textStyles: [{name, fontFamily?, fontSize?, fontWeight?, italic?, letterSpacing?, lineHeight?, fill?}], unresolvedColors: [names]}`。別の変数のエイリアスである値（`"$blue500"`）は、`alias` にその変数名を、`value` にその値のテーマでチェーンが解決される値を持ちます（チェーンが壊れていれば null）。ほかの変数の式である数値（`"=$space * 2"`）は、`expression` に保存されたままの式のテキストを、`value` にその値のテーマで計算される数値を持ちます（計算できなければ null）。 |
| `assets` | `{fonts: [families], images: [the image URLs fills name], icons: [SF Symbol names]}`。 |

ノード:

| キー | 値 |
| --- | --- |
| `id` | デザインのノードのレンダー id（インスタンスの中では `instanceID/childID`）。 |
| `kind` | `frame`、`group`、`rectangle`、`ellipse`、`polygon`、`line`、`path`、`boolean`、`text`、`icon`、`instance`（コンポーネントが見つからないもの）のいずれか。 |
| `name`、`codeName` | レイヤーリストの名前と、解決されたノード名。 |
| `visible`、`isComponent`、`absolute`、`clip` | ブール値。 |
| `frame` | レイアウト後の `{x, y, width, height}`（親の座標系）。 |
| `opacity`、`rotation`、`innerRadius`、`subtreeSize` | 数値。 |
| `layout` | `{direction (free, horizontal, vertical, grid), gap, padding: {top, right, bottom, left}, justify, align, wrap, rowGap?, alignContent, gridColumns?, gridRows?, gridAreas?}`。グリッドのトラックは `"1fr"`、`"auto"`、`"25%"`、固定値、またはデザインに書かれたとおりの関数（`"minmax(120, 1fr)"`、`"repeat(2, 100, $colW)"`、`"repeat(auto-fill, minmax(160, 1fr))"`）です。`gridAreas` はグリッドの名前付きエリアで、`["head head", "nav main"]` のような CSS の `grid-template-areas` の行です（`.` は空のセル）。 |
| `gridColumn`、`gridRow` | グリッドの子で: デザインに保存されたとおりの `{start?, span?}`。ない場合は自動配置です。 |
| `gridArea` | グリッドの子で: そのノードが覆う、グリッドの `gridAreas` のエリアの名前（その軸では明示的な `gridColumn`/`gridRow` が優先されます）。名前を指定していなければありません。 |
| `width`、`height` | ノードが求めたもの: `{"fixed": <value>}`、`"fit"`、`"fill"`。何も求めていなければ省きます。 |
| `minWidth`、`maxWidth`、`minHeight`、`maxHeight` | 設定されているときの値。 |
| `fills` | ペイント: `{type: "solid", color}`、`{type: "gradient", gradient (linear, radial, angular), stops: [{color, position}], rotation, opacity}`、`{type: "image", url, fit, opacity}`。それぞれ省略可能な `blendMode` を持ちます。 |
| `stroke` | `{fills, thickness, align (inside, center, outside), dash?, widthProfile?: [{t, width}]}`。 |
| `corners` | `{topLeft, topRight, bottomRight, bottomLeft}` の値。 |
| `effects` | `{type: "shadow", inner, x, y, blur, spread, color}`、`{type: "blur", radius}`、`{type: "backgroundBlur", radius}`。 |
| `mask` | 上にある兄弟ノードをマスクするノードの `{type (alpha, luminance, vector), opacity, fill?}`（このノード自体は描画されません）。 |
| `blendMode` | 設定されているときのレイヤーのブレンド（CSS の名前）。 |
| `text` | `{content, parameter?, fontFamily, fontSize, fontWeight, italic, underline, strikethrough, letterSpacing, lineHeight?, align, growth, style?, followsPath, hasRuns, runs: [{start, end, …, link?}], maxLines?, textCase?, paragraphSpacing?, paragraphIndent?, listItems?: [{paragraph, kind, level}], leadingTrim?, fontVariations?}`。`leadingTrim`（設定されているときだけ）は `"capHeight"` で、ボックスは最初の行のキャップハイトから最後の行のベースラインまでになります。`fontVariations`（設定されているときだけ）は `{tag: number}` で、`{"wght": 550, "wdth": 87.5}` のような可変フォントの軸の値です。`fontWeight` はすでに `wght` の値を持っています。`listItems`（設定されているときだけ）は `bullet` または `number` のリスト項目として表示される段落で、`level` は 0〜3 です。ランの `link`（設定されているときだけ）はその文字のリンク先で、`set_link` の形です。`hasRuns` はランが何かにスタイルを付けているかどうかです（リンクだけのランは含みません）。`maxLines` はデザインされたとおりの行数の上限です（キャンバスが適用するのは、パスに沿わない固定幅と固定サイズのテキストだけです）。`textCase` は `upper`、`lower`、`title` のいずれかで、`content` は保存されたままです。 |
| `icon` | `{name?}`（SF Symbol）。 |
| `path` | ノード自身の座標系でのアウトラインの要素: `{op: move/line, to}`、`{op: quad, control, to}`、`{op: curve, control1, control2, to}`、`{op: close}`。 |
| `call` | パラメータを設定するだけのインスタンス: `{componentID, componentName, variant, fillsWidth, fillsHeight, arguments: [{name, value, kind, component?}]}`。 |
| `interactions` | `{trigger, action, target?, url?, variant?, variable?, value?, toggle?, if?}`（最後の 4 つは設定されているときだけ: `set_variable` の変数とその値または `toggle`、そしてインタラクションの条件）。 |
| `source` | インポーターが読めなかったコードのプレースホルダー: `{file, line, text}`。 |
| `visibleParameter`、`slotParameter` | そのノードを表示・非表示にする、またはそのノードが表示するコンポーネントのパラメータ。 |
| `children` | ノード。 |

値は `{"value": <number>, "token": "<variable>"}` で、トークンは値が変数から来たときだけ入ります。色は
`{"hex": "#RRGGBB"}`（または `#RRGGBBAA`）か `{"token": "<variable>"}` です。組み込みのターゲットが必要に応じてレンダリングする画像（アイコンの
PNG、マスクの SVG）はモデルに含まれません。

## 失敗

次の場合、実行はプラグインを示すエラー（`plugin "flutter" …`、1 行）で失敗し、何も変わりません。

- プラグインが承認されていない、承認後に変更された（後述）、または「許可しない」がクリックされた。
- フォルダが検証を通らなくなった、または id や種類が変わった（インストールし直してください）。
- コマンドを起動できない（`command[0]` がない、または実行可能でない）。
- 0 以外のステータスで終了した（メッセージには stderr の最後の数行が入ります）、またはシグナルで終了した。
- `timeout` を超えて実行された。SIGTERM が送られ、その 1 秒後に SIGKILL が、プラグインが起動したものすべてに送られます（プラグインは
  自身のプロセスグループを率います）。
- stdout が JSON でない、オブジェクトでない、上の形でない（文字列の `path` と `contents` のない `files` の項目、未知の
  診断の機能）、または 256 MiB を超える。
- ファイルのパスが空、絶対パス、または `..` で外に出る。
- インポーターのフラグメントが有効な Silverpoint ドキュメントでない、または `library:write` なしで変数、スタイル、テーマを含む。

stdin を読まずに終了するプラグインでも問題ありません（Silverpoint がそれで SIGPIPE を受けることはありません）。CLI では、失敗したプラグインは終了
ステータス 1 と stderr の 1 行になります。MCP ではツールのエラー、アプリではアラートになります。

<h2 id="trust-and-approval">信頼と承認</h2>

プラグインはプログラムなので、**まさにそのバージョン** をあなたが承認して初めて実行されます。

- 承認は、[フック](../hooks/)が使うのと同じ承認ファイル（`$SILVERPOINT_HOOK_APPROVALS_FILE`、なければ
  `~/Library/Application Support/Silverpoint/hook-approvals.json`）に記録されます。プラグインごとに `plugin:<id>` という対象で 1 つの項目があり、
  ドキュメントのフックの項目と並びます（各項目は個別に承認、置き換え、削除されます）。
- 承認されるのはコンテンツのハッシュです。プラグインフォルダ内のすべての通常ファイル（マニフェストを含み、`.DS_Store`
  とシンボリックリンクは除く）に対する、相対パスごとの SHA-256 です。どのファイルでも編集、追加、削除、名前の変更をするとプラグインは `changed` になり、
  もう一度承認を求めます。
- **アプリでは**、プラグインの最初の実行（と、変更後の最初の実行）で、名前、id、バージョン、種類、
  フォルダ、**実行する正確なコマンド**、[権限](#permissions)を示すアラートが出ます。**許可** はハッシュを記録して実行し、**許可しない** は何も実行しません（次回
  また尋ねます）。アプリ内のエージェントによる実行も同じように尋ねます。
- **MCP と CLI では何も尋ねません**。承認されていない、または変更されたプラグインは、プラグインを示して承認されていないと伝える
  エラーになります。これはアプリ自身の MCP サーバーでも同じです。ほかのプログラムが、あなたの前にダイアログを出してコマンドを承認させることはできません。
  プラグインの承認は **設定 ▸ 一般 ▸ プラグイン**（そのスイッチ、または **確認…** から **承認**）、アプリでの最初の使用時の
  アラート、またはあなた自身のターミナルでの `silverpoint plugins install <folder> --approve`（コマンドを表示します）で行います。
- `silverpoint plugins check <folder>` は、指定したフォルダを承認なしで一度だけ実行します。チェックすることがあなたの明示的な操作だからです。
- `silverpoint plugins remove <id>` はフォルダを削除し、そのプラグインの承認だけを取り消します。

**設定 ▸ 一般 ▸ プラグイン** には、インストールされたプラグインが名前、id、種類、バージョン、状態（**承認済み**、
**承認が必要**、**承認後に変更あり**）とともに並び、使えるプラグインでないフォルダは理由とともに並びます。各プラグインの
スイッチがその承認です。オンにすると **確認…** が開き、正確なコマンド、権限、実行するフォルダ、SHA-256、
そのハッシュが対象とするすべてのファイルが表示されます。承認できるのはそこでの **承認** だけです（その時点のディスク上の状態で承認します。確認…
を開いてから変更されていれば何も承認せず、プラグインをもう一度表示します）。スイッチをオフにすると、そのプラグインの承認だけが取り消され、ほかの項目は変わりません。
**インストール…** はプラグインフォルダをコピーして入れ、承認はしません（その id の承認が残っていれば削除します）。**削除…** は確認してから
フォルダとその承認を削除します。**フォルダを表示** はプラグインフォルダを Finder で表示し、**再読み込み** はフォルダを読み直します（このセクションは
ウインドウが開いたときや前面に来たときにも読み直します）。コードターゲットのインストールや削除は、Silverpoint を
再起動すると反映されます。エクスポーターとインポーターはすぐにメニューに表示されます。ここ、アラート、`--approve` のどの承認も
`PluginHost.approve` を通ります。エージェントや MCP クライアントが触れられるものでプラグインを承認できるものはありません。

承認のほかに制限となるのは [権限](#permissions)だけです。承認されたプラグインはあなたとして実行され、
`environment` を宣言する（または `permissions` がまったくない）プラグインは、API キーを含めあなたの環境変数をすべて受け取ります。信頼できるプラグインだけを承認してください。

## コマンドライン

    silverpoint plugins list [--json]
    silverpoint plugins install <folder> [--approve]
    silverpoint plugins remove <id>
    silverpoint plugins check <folder> [--input <file>]

- `list` はプラグインごとに 1 行、`id<TAB>kind<TAB>version<TAB>status<TAB>name<TAB>permissions` を表示します。状態は `approved`、
  `needs approval`、`changed` のいずれかで、権限はスコープの id をカンマでつないだもの（`library:write,environment`）、`[]` なら `none`、
  マニフェストが何も宣言していなければ `legacy` です。続いて、検証を通らないフォルダごとに `invalid<TAB><folder><TAB><reason>` を表示します。
  プラグインフォルダがなければ何も表示せず、0 で終了します。`--json` は `{plugins: [{id, name, kind, version, status, folder,
  command, permissions, permissionLines}], invalid: [{folder, reason}]}` を返します。`permissions` はスコープの id の配列（レガシーの
  プラグインでは `null`）、`permissionLines` は下の行です。
- `install` はフォルダを検証し、`<plugins>/<id>` にコピーします。無効なマニフェストと、すでにインストールされている id は
  拒否します（終了ステータス 1、何もコピーしません）。`--approve` はコマンドを表示し、承認を記録します。
- `remove` はインストールされたプラグインとその承認を削除します。未知の id では 1 で終了します。
- `check` はマニフェストを検証し、`command:` 行のあとに `permissions: <ids, comma-separated | none | legacy>`
  と、各スコープの行を 2 スペース字下げして（`  Read your variables, styles and themes`）表示します。そのうえで、コードターゲットは組み込みのサンプルモデル（2 つの画面、1 つのコンポーネント、トークン）で、
  エクスポーターはサンプルドキュメント（マニフェストが求めればそのレンダーも）で、インポーターは `--input <file>` で実行し、
  答えをプロトコルに照らしてチェックします。返ってきたものを表示し、問題がなければ 0 で終了します。プロトコルのエラーは stderr の 1 行と
  終了ステータス 1 になります。`--input` のないインポーターはマニフェストだけをチェックします（`manifest only: …`、終了ステータス 0）。何も書き込みません。
- `silverpoint mcp`、`call`、`tools` は、インストールされたコードターゲットを先に登録します。そのため `tools --json` は `export_code` の `format` に
  `plugin:<id>` を挙げ、`call <file> export_code '{"format":"plugin:flutter"}'` で（承認後に）実行できます。
  `call <file> import_file '{"path": "/abs/file.json"}'` はインポーターを実行します。

## アプリでは

- コードターゲットは、**ファイル ▸ コードまたはトークンを書き出す…**、ハンドオフのターゲットの選択、`export_code` で組み込みのものと並びます。
- **ファイル ▸ 別の形式で書き出す** には、エクスポーターごとに 1 つの項目（マニフェストの `name`）があります。フォルダを尋ね、そこにファイルを書き込みます。
- **ファイル ▸ 読み込む** には **Figma から…**、**Mermaid から…**、**Excalidraw から…**、**draw.io から…**（あとの 3 つはプラグインではなく組み込み）が並び、続いてインポーターごとに 1 つの項目があります。
  そのオープンパネルではインポーターの `fileTypes` を選べます。読み込みは 1 回の取り消しステップです。
- `import_file`（`path`、省略可能な `plugin`。省略すると、`fileTypes` にファイルの拡張子を持つ最初のインポーターを id 順で選びます）
  は `{plugin, created, report}` を返します。`created` は新しいトップレベルのノードの id です。

## サンプル

リポジトリの `Examples/plugins/` に 2 つのサンプルプラグインがあります（何もインストールしません。テストのフィクスチャも兼ねています）。
どちらもシステムの `/usr/bin/python3` とその標準ライブラリだけを使います。

- **`flutter-skeleton`**（id `flutter`、コードターゲット、`.dart`）: 画面とコンポーネントごとに `lib/` の下に 1 つのファイルを作ります。それぞれ
  `package:flutter/material.dart` を import し、ルートにちなんだ名前の `StatelessWidget` を持ちます（フレームは `Column`、`Row`、`Stack` に、
  テキストは `Text` に、残りはサイズ付きの `Container` になります）。宣言している機能はわずかで（ドロップシャドウは full、画像の塗りと複数の
  塗りは approximated、内側のシャドウは unsupported で、これは自分で報告します）、残りは Silverpoint が報告します。モデルの `tokens` で塗りの色の変数を探して値を書くので、
  `library:read` を宣言しています。

      silverpoint plugins check Examples/plugins/flutter-skeleton
      silverpoint plugins install Examples/plugins/flutter-skeleton --approve
      silverpoint call design.silverpoint export_code '{"format": "plugin:flutter"}'

- **`shapes-json`**（id `shapes-json`、インポーター、`.json`）: `{"frames": [{"name", "width", "height", "rects": [{"x", "y",
  "width", "height", "color"}]}]}` を読み、長方形を入れたフレームとして横に並べ、数とスキップしたものを報告します。
  `sample.json` には `Sample Card`（長方形 3 つ）と `Sample Banner`（2 つ）の 2 つのフレームがあります。ノードしか追加しないので、
  `"permissions": []` を宣言しています。

      silverpoint plugins check Examples/plugins/shapes-json --input Examples/plugins/shapes-json/sample.json
      silverpoint plugins install Examples/plugins/shapes-json --approve
      silverpoint call design.silverpoint import_file "{\"path\": \"$PWD/Examples/plugins/shapes-json/sample.json\"}"

## まだできないこと

エクスポーターが返せるのはテキストファイルだけです（バイナリの `contents` は不可）。プラグインでコードをデザインに読み戻すことはできません（`parse` は
未サポートのままです）。承認と権限のほかにサンドボックスはなく（ネットワークやファイルの制限はありません）、マーケットプレイスやアップデーターもありません。
