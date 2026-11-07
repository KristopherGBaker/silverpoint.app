---
title: "ユーザーテンプレートとスタイルプリセット"
description: "Silverpoint には、新しいドキュメントを始めるためのスターターテンプレートと、エージェントが読み込むスタイルプリセットが付属しています。どちらも、自分で管理するフォルダに独自のものを追加して拡張できます。"
summary: "Silverpoint には、新しいドキュメントを始めるためのスターターテンプレートと、エージェントが読み込むスタイルプリセットが付属しています。どちらも、自分で管理するフォルダに独自のものを追加して拡張できます。"
layout: doc
toc: true
---

Silverpoint にはスターター **テンプレート**（起動ウインドウの「開始方法」のカード、ファイル ▸ テンプレートから新規作成）と **スタイルプリセット**（エージェントの `get_style` が提供するもの）が付属しています。どちらも、自分で管理するフォルダに独自のものを追加して拡張できます。スキルも同じ仕組みです（[skills.md](../skills/) を参照）。

## テンプレート

テンプレートは、新しいドキュメントを始めるもとになるデザインです。

付属のスターターは、空白、iPhone アプリ、ウェブページ、スライド（ホームウインドウの「開始方法」の行）と、デザインシステム、ダイアグラム、サンプルカード（テンプレートセクションとファイル ▸ テンプレートから新規作成）です。**ダイアグラム**（`Resources/Templates/diagram.json`）は、`diagrams` スキルが描くとおりに準備されています。ライトとダークの値を持つ変数としての Diagram Minimal のパレット（`background`、`surface`、`ink`、`muted`、`border`、`accent`、`chart-1` … `chart-6`）、`Title`、`Node`、`Label`、`Legend` のテキストスタイル、ノードの種類ごとのコンポーネント（Process、Decision、Terminal、Data、Actor、Note、Group）と Legend、そして Checkout flow の例です。例は、接続したコネクタでつながれ `layout_diagram` で配置された 7 つのノードのインスタンスに、コールアウトとその下の凡例が付いたものです。ノードの輪郭は、プリセットの `$border` のヘアラインではなく `$muted` です。`$border` は、ボックスの縁に対するコントラストのリントの 3:1 に届かないためです。このファイルは生成されたものです。`DiagramTemplateBuilder`（SilverpointAgentTests）が実際の操作を通して作り、`DiagramTemplateTests` がコミットされたファイルがその出力と同じであることを確認し、`SILVERPOINT_BLESS=1` で書き直します。

### 置き場所

| 対象 | 場所 | 変更方法 |
| --- | --- | --- |
| テンプレート | `~/Library/Application Support/Silverpoint/Templates` | `SILVERPOINT_TEMPLATES_DIR` |

このフォルダには `*.silverpoint` パッケージと `*.json` ドキュメント（`document.json` 単体で、画像は含みません）を置きます。アプリと `silverpoint templates` は同じフォルダを読みます。隠しファイル、サブフォルダ、ほかの拡張子のファイルは無視されます。Silverpoint のドキュメントとして読めないファイルはスキップされ、警告として報告されます（`silverpoint templates` では stderr に出力）。クラッシュすることはありません。

**名前** は拡張子を除いたファイル名です（`Phone App.json` は「Phone App」）。ドキュメント自体にはタイトルのフィールドがないため、ファイル名がそのまま名前になります。テンプレートは大文字と小文字を区別せず、名前順に並びます。`Hero.silverpoint` と `Hero.json` が両方ある場合は 1 つのテンプレートとして扱われ、パッケージが優先されて `.json` は警告になります。

### 表示される場所

- **ホームウインドウ:** 「開始方法」の行に付属のカードが並び、そのあとに「あなたのテンプレート」というラベルの下にあなたのテンプレートが並びます（テンプレートがあるときだけ表示され、数が多いと行が横にスクロールします）。各カードには、「最近」のカードと同じレンダラーとキャッシュで描かれたサムネイルが付きます。サイドバーのテンプレートセクションにも表示されます。
- **ファイル ▸ テンプレートから新規作成:** 付属のテンプレートのあとに「マイテンプレート」セクションがあります。メニューは開くたびに読み込み直されます。

カードをクリックするか、メニュー項目を選ぶと、テンプレートの内容で **名称未設定の新しいドキュメント** が開きます。このドキュメントはコピーです。ファイルを持たないので保存時に保存先を尋ねられ、何をしてもテンプレートは変わりません。`.silverpoint` テンプレートは画像も一緒に持ってきます。

### テンプレートとして保存…

ファイル ▸ **テンプレートとして保存…** は名前を尋ね、現在のドキュメントを `<name>.silverpoint` としてテンプレートフォルダに保存します。選択範囲にトップレベルのフレームが含まれるときは **選択したフレームのみ** チェックボックスが表示されます。オンにすると、テンプレートにはそのフレームだけが入り、それらが依存するもの（ドキュメントのテーマ、変数、テキストスタイルとレイヤースタイル、トップレベルのコンポーネント、残したレイヤーが使う画像）も残ります。キャンバス全体に属するコメントとインポート元は含まれません。

同じ名前のテンプレートがすでにある場合（拡張子はどちらでも、大文字と小文字は区別しません）、Silverpoint は **置き換える前に確認します**。名前はファイル名として安全な形に変換されます。スラッシュ、バックスラッシュ、コロンはダッシュになり、先頭と末尾のドット、ダッシュ、スペースは取り除かれるので、テンプレートは常にテンプレートフォルダの直下に保存されます（`../Evil` は `Evil` として保存されます）。文字や数字をひとつも含まない名前は受け付けません。

### 管理

ファイル ▸ **テンプレートをFinderに表示** でフォルダを開きます（フォルダがなければ先に作成します）。カードのコンテキストメニューの **削除…** は確認したうえで、ファイルを **ゴミ箱** に移動します（完全には削除しません）。カードのメニューには「開く」と「Finderに表示」もあります。

設定 ▸ 一般 ▸ **テンプレートとスタイル**（デザインシステムのグループ内）には、テンプレートとスタイルのフォルダ、そしてフォントの組み合わせとプロダクトのフォルダが表示されます（組み合わせとパレットはスタイルの一部なので、セクションの名前はそのままです）。それぞれのパスと「Finderに表示」（フォルダがなければ作成し、プロダクトのフォルダでは `palettes/` も作成します）、あなたの項目、読み込めないファイルごとにその理由を示す行（`silverpoint templates` と `silverpoint styles` が出力するのと同じ警告）が並びます。テンプレートとスタイルでは、そのあとに組み込みの項目である「開始方法」のテンプレートと付属のすべてのスタイルプリセットが並びます。**フォントの組み合わせ** は、あなたのものと組み込みのものの数を示し（`2 yours · 35 built-in`、さらに `· 1 not loaded`）、あなたのものをスラッグで一覧にします。**パレットとプロダクトタイプ** は、`palettes/` のパレットとフォルダの表にあるプロダクトの種類について同じことを行い（表の不正な行は、ファイル名と行番号を示す独立した行になります）、あなたの各パレットの下に、そのパレットが通らない[コントラストの確認](#contrast-checks-on-your-palettes)を一覧にします。組み合わせ、モード、最小値に対する比です（`Low contrast: onAccent on accent, dark, 2.31:1 (needs 4.5:1)`）。付属のものと同じ名前のあなたの項目には、それを置き換えることが表示されます。このセクションは読み取り専用です（フォルダは Finder で編集してください）。設定ウインドウが表示されたときやキーウインドウになったとき、そして「再読み込み」で、メインアクターの外でフォルダを読み込み直します。そのモデル（`TemplatesStylesSettingsModel`）は渡されたフォルダだけを読みます。アプリは標準のフォルダを渡し（`TemplatesStylesSettingsModel.standard()`）、テストは一時フォルダを渡します。

## スタイルプリセット

スタイルプリセットは、スキルと同じ形式の Markdown ファイルです（`title:` と `summary:` のフロントマターのあとにガイドが続きます）。フォント、色、タイプスケール、形状を定めます。`get_style` で読み込みます。

付属のプリセットは 11 個です。8 個はページと画面用（`bold-brutalist`、`corporate-clean`、`dark-tech-glow`、`editorial-serif`、`luxury-noir`、`soft-pastel`、`swiss-minimal`、`warm-organic`）、3 個は図解用で、`diagrams` スキルが指定するものです。

| プリセット | 見た目 | フォント |
| --- | --- | --- |
| `diagram-minimal-light` | 温かみのある紙の上に白いノード、錆色のアクセント 1 色 | 見出しと本文に SF Pro、技術的なテキストに SF Mono |
| `diagram-minimal-dark` | ほぼ黒の上に明るめの面、琥珀色のアクセント 1 色 | 見出しと本文に SF Pro、技術的なテキストに SF Mono |
| `diagram-editorial` | クリーム色の上で図の上にセリフ体のタイトル、インクブルーのアクセント 1 色 | 見出しに Charter、本文に SF Pro、SF Mono |

**ページ用プリセットの色: すべてのロールをライトとダークで。** 8 個のページ用プリセットはどれも同じカラーロールを定義し、それぞれにそのプリセットのためにデザインしたライトとダークの値があります（反転ではありません）。

| ロール | 用途 |
| --- | --- |
| `background` | ページ |
| `surface` | カード、パネル、フィールド |
| `ink` | 面の前景色: background と surface の上の見出しと本文 |
| `muted` | 控えめな前景色: キャプション、ラベル、プレースホルダー、コントロールと入力欄の枠線 |
| `border` | ヘアラインと区切り線 |
| `accent` | プライマリボタン、リンク、ハイライト |
| `onAccent` | `accent` の上のテキストとアイコン |
| `destructive` | 削除ボタンとエラーテキスト |
| `onDestructive` | `destructive` の上のテキストとアイコン |
| `focus` | フォーカスリング |

プリセットがもともと持っていた追加のロール（`accent2`、`accent3`、`success`、soft-pastel の 4 つの `pastels`）も残り、ダークの値も付いています。色の行は `- accent: #RRGGBB / #RRGGBB` の形で、2 つの値は見出しが示す順に並びます。見出しは `## Colors (mode: light / dark; define these with set_variables)` で、もともとダークな 2 つのプリセット（`dark-tech-glow`、`luxury-noir`）では `mode: dark / light` となり、最初の値がそのプリセット本来の見た目のままです。これらは `mode` テーマ軸を持つ変数として適用します。最初の値はそのまま（デフォルトのモード）、2 つ目の値にはもう一方のモードのタグを付けます。各プリセットの Colors セクションには例が 1 行あり、`get_style` はプリセットのほかの部分と一緒にそれを返します。

```json
{"op":"set_variables","variables":{"accent":{"type":"color","value":[{"value":"#7A1E2C"},{"value":"#E07A8A","theme":{"mode":"dark"}}]}},"themes":{"mode":["light","dark"]}}
```

どの組み合わせも両方のモードでコントラストの基準を満たします（`BundledStyleDataTests` と、テーマのモードごとに確認する `contrast` リント）。background と surface の上の `ink`、background の上の `muted` と `destructive`（エラーテキストなので）、accent の上の `onAccent`、destructive の上の `onDestructive` は 4.5:1、background の上の `accent` と `focus`、そして surface の上の `focus` は 3:1 です。border は装飾なので確認しません。すでに違いのある領域の間のヘアラインには意味がないためで、コントロールの枠線には 3:1 以上を満たす `muted` を使います。save_style も同じライト / ダークの形式を受け付け、本文を渡されたとおりに書き込みます。

図解用プリセットはそれぞれ、6 つのパレット変数（`background`、`surface`、`ink`、`muted`、`border`、`accent`）を 1 つのモードで定義します（ライトとダークの見た目は別のプリセットです）。小さなタイプスケール、10 以下の角丸、4 pt の間隔単位を持ち、注記では 1 pt のヘアライン、影なし、アクセント 1 色、横並びの凡例を求めています。フォントは macOS にインストールされていて `list_fonts` が提供するものです。diagram-design が使う書体（どちらも macOS には付属しません）の代わりに、Geist と Geist Mono には SF Pro と SF Mono、Instrument Serif には Charter を使います。

| レイヤー | 場所 | 編集する人 |
| --- | --- | --- |
| 付属 | アプリ内（`Resources/Styles`） | 編集不可。同じ名前のスタイルを保存して上書きします |
| パッケージ | ドキュメントがリンクする各デザインシステムパッケージの `styles/*.md`（`docs/design-system-packages.md`） | パッケージの作者 |
| ユーザー | `~/Library/Application Support/Silverpoint/Styles/*.md`（`SILVERPOINT_STYLES_DIR`） | あなた。エージェントの `save_style` |

**優先順位:** 同じ名前なら、ユーザーのスタイルがパッケージのスタイルより、パッケージのスタイルが付属のものより優先されます（名前は `.md` を除いたファイル名を小文字にし、文字と数字以外の連続を 1 つのダッシュにしたもので、スキルと同じです）。解決されたスタイルは出どころを記録し、下のレイヤーのものを置き換えたときはそう示します。`silverpoint styles` は `user (overrides bundled)` や `package (overrides bundled)` と出力します。パッケージのスタイルがリストに入るのは、ドキュメントがそのパッケージをリンクしている間だけです（`silverpoint styles <doc>`、`get_style`、そのドキュメントに対するエージェントのプロンプト）。フォルダに同じ名前のファイルが 2 つあるときは、ファイル名のバイト順で先のものが優先され、もう一方は警告になります。

**スタイルにはドキュメントのレイヤーがなく**、無効化リストもありません。ドキュメントは見た目を変数、テーマ、テキストスタイルに持っていて、エージェントはそれをすでに読んでいます。スタイルは、まだ見た目のないデザインの出発点です。（どちらも今後の検討事項です。バックログを参照してください。）

`get_style`、その説明（解決されたすべてのスタイルの索引で、`name` の選択肢でもあります）、エージェントのシステムプロンプト（「Styles available through get_style」）、MCP クライアント、CLI はすべて、解決済みの同じリストを見ます。リストはツールの実行時に読まれるので、ターンの合間に保存したスタイルもすぐに見つかります。

### スタイルの保存: `save_style`

「この見た目を Editorial という名前のスタイルとして保存して」と頼むと、エージェントは `name`、`title`、`summary`、`body`、`confirm` を付けて `save_style` を呼びます。共有カタログのツールの 1 つで、`save_skill` と同じ確認の流れです。

- **アプリでは** 直接書き込みません。カードにテキスト（または置き換えるスタイルとの差分）が「保存」と「破棄」と一緒に表示され、「保存」で `<name>.md` をあなたのスタイルフォルダに書き込みます。
- **MCP と `silverpoint call` では**、`confirm: true` がなければプレビュー（`{written: false, slug, title, replaces, text, diff}`）を返し、何も書き込みません。付けると書き込み、`written: true` を返します。既存のスタイルを置き換えるとき、`replaces` は `user` か `bundled` です。
- 本文が空、または文字や数字をひとつも含まない名前はツールエラーで、何も書き込みません。スタイルフォルダのない構成（フォルダを渡さなかったライブラリの呼び出し元）も同様です。

別の名前のファイル（`Neon Look.md`）から来たスタイルの上に保存すると、そのファイルを書き換えるので、読まれるのは保存したほうです。

## フォントの組み合わせ

フォントの組み合わせは、見出しの書体と本文の書体（ときには等幅の書体も）の組です。スタイルのパレットに別の文字の声を合わせられるよう、スタイルプリセットとは分けてあります。`get_font_pairing` が提供します。`name` を付けるとその組み合わせの書体、タグ、書体の入手元（`fonts: installed with macOS`、またはフォールバックを付けた `fonts: library (…)`）、テキストを返し、付けないと組み合わせごとに 1 行、`slug — summary [moods; uses]` を返します。ライブラリの組み合わせの行は ` (library: <families>; fetched when added)` で終わります。これは `mood`、`use`、`fonts`（`installed` または `library`）で絞り込めます（それぞれ大文字と小文字を区別せず完全一致で、複数指定するとそのすべてで絞り込みます）。インストール済みの組み合わせが常に先に、そのあとにライブラリの組み合わせが並ぶので、デザインがネットワークに依存するのはライブラリの組み合わせを選んだときだけです。一致がないときは `No font pairing matches` で始まる行と語彙を返し、不明な名前は組み合わせの一覧を示すツールエラーになります。共有カタログにあるので、エージェント、MCP クライアント、`silverpoint call <doc> get_font_pairing '{"mood":"calm"}'` のどれでも使え、`silverpoint styles --pairings [--fonts installed|library] [--json]` で一覧にできます（コマンドラインを参照）。

それぞれ `Resources/Pairings` にある Markdown ファイルで、フロントマターは次のとおりです。

- `title:` と `summary:`。スタイルと同じです。
- `heading:` と `body:`。2 つのファミリーで、`mono:` は任意です。
- `moods:`。editorial、calm、technical、playful、luxurious、friendly、formal、bold からのカンマ区切りのリスト。
- `uses:`。app UI、marketing、long reading、data、slides からのカンマ区切りのリスト。
- `fonts:`。書体がフォントライブラリから来るときは `library`（それ以外の値や、この行がないときは `installed`）。
- ライブラリの組み合わせでは、`fallback:`（書体を取得できないときに使うインストール済みの組み合わせ）と `products:`（その組み合わせを提案するプロダクトの種類。`products.md` のスラッグ）。

そのあとのテキストは `# <title>` と書体を示す 1 行で始まり、`## Sizes and weights`（合うポイントサイズとウェイト）と `## Avoid` が続きます。付属のインストール済みの組み合わせは 14 個で、`new-york-sf-pro`（editorial）、`sf-pro-sf-mono`（technical）、`sf-pro-rounded`（friendly）などがあります。どの書体も macOS に付属するフォントです。SF Pro、SF Pro Rounded、SF Mono、New York はシステムのデザインで（SF Pro Rounded と New York は、その名前でインストールされていないときはシステムフォントの丸ゴシックとセリフのデザインで描かれます）、ほかは Didot、Avenir Next、Charter、Helvetica Neue のようなインストール済みのファミリーです。付属データに対するテストが、実行している Mac でそれぞれが解決できることを確認します。

### ライブラリの組み合わせ

付属の組み合わせには、フォントライブラリのファミリー（下の「フォントライブラリ」を参照）でできたものがさらに 21 個（`fonts: library`）あります。私たち独自の選択で、それぞれに、書体がなぜ合うのかと使うウェイトについての 1 行が付いています。

| 組み合わせ | 書体 | 用途 |
| --- | --- | --- |
| `source-serif-4-source-sans-3` | Source Serif 4, Source Sans 3, Source Code Pro | ドキュメントとヘルプ: 1 つのシステムで、プロポーションが共通 |
| `ibm-plex-serif-ibm-plex-sans` | IBM Plex Serif, IBM Plex Sans, IBM Plex Mono | レポートと金融: 1 つの組織の 3 つの声 |
| `ibm-plex-sans-ibm-plex-mono` | IBM Plex Sans, IBM Plex Mono | 数字がそろう、密度の高いダッシュボード |
| `geist-geist-mono` | Geist, Geist Mono | 開発者ツール: テキストとコマンドに 1 つのグリッド |
| `space-grotesk-inter` | Space Grotesk, Inter, JetBrains Mono | テックのローンチ: 癖のある見出しと中立的な本文 |
| `fraunces-figtree` | Fraunces, Figtree | 温かくおいしそうな食べ物と日常のアプリ |
| `playfair-display-source-sans-3` | Playfair Display, Source Sans 3 | 華やかな雑誌とファッションの見出し |
| `newsreader-public-sans` | Newsreader, Public Sans | ニュース: 報道のために作られたセリフ体と、それを囲む公共的なサンセリフ |
| `instrument-sans-literata` | Instrument Sans, Literata | 読書アプリとエディタ: モダンな UI と本の書体 |
| `instrument-serif-instrument-sans` | Instrument Serif, Instrument Sans | 現代的なエディトリアル: コンデンスのディスプレイ用セリフ体と、対になるサンセリフ |
| `cormorant-garamond-manrope` | Cormorant Garamond, Manrope | 静かなラグジュアリーとホスピタリティ |
| `dm-serif-display-dm-sans` | DM Serif Display, DM Sans | ブティックのコマース: 1 つのプロジェクトのセリフ体とサンセリフ |
| `bricolage-grotesque-dm-sans` | Bricolage Grotesque, DM Sans | 大胆で現代的なマーケティング |
| `syne-manrope` | Syne, Manrope | スタジオ、ギャラリー、カルチャーブランド |
| `bebas-neue-work-sans` | Bebas Neue, Work Sans | トレーニング、スポーツ、イベントのポスター |
| `unbounded-onest` | Unbounded, Onest | フェスティバル、音楽、ストリーミング |
| `abril-fatface-lora` | Abril Fatface, Lora | メニュー、レシピ、食の読み物 |
| `nunito-lexend` | Nunito, Lexend | 子どもの学習: 丸みのあるタイトルと、読み始めの子のために作られたサンセリフ |
| `caveat-figtree` | Caveat, Figtree | 日記と習慣のツール: 手書きのアクセント |
| `noto-serif-jp-noto-sans-jp` | Noto Serif JP, Noto Sans JP | 日本語のエディトリアル: ゴシック体の本文に明朝体の見出し |
| `m-plus-1-noto-sans-jp` | M PLUS 1, Noto Sans JP | 親しみやすい日本語のアプリ UI |

**決まり。** ライブラリの組み合わせの書体は、テキストが使う前にドキュメントに入っていなければなりません。`family` の代わりに `{"pairing": "<slug>"}` を付けた `add_library_font` は、組み合わせが指定するライブラリの書体のうちドキュメントがまだ持っていないものをすべて取得し、すべて届いたときだけ、各ファミリーの OFL ライセンスと一緒に 1 回の取り消しステップ（フォントペアリングを追加）でドキュメントに持たせます。そのあとでテキストスタイルがそれを指定できます。取得できない書体があるとき（オフライン、このコンテキストにライブラリがない、一致しないファイル）は何も追加されず、答えは `Fallback:` で始まります。理由を示し、代わりに使うインストール済みの組み合わせ（組み合わせの `fallback`、なければムードが共通するインストール済みの組み合わせ）をその詳細とともに挙げ、ライブラリのファミリーを指定しないようエージェントに伝えるので、デザインに `missing-font` の警告が残ることはありません。インストール済みの組み合わせには追加するものがありません。`design-principles` とスキーマのスキルがそう伝え、ライブラリの組み合わせについての `get_font_pairing` の詳細も、行うべき呼び出しを繰り返します。付属データに対するテスト（`LibraryPairingTests`）は、各ライブラリの組み合わせが指定するファミリーがすべて、カタログと同じ綴りで、そのファイルが持つウェイト（とイタリック）で `library.json` にあること、それぞれにインストール済みのフォールバックと実在するプロダクトがあること、そしてどのプロダクトの種類にも少なくとも 1 つが提案されることを確認します。

### 独自の組み合わせ

フォントの組み合わせはスタイルと同じ仕組みで、同じリゾルバによってレイヤー化され、スラッグ（スタイルと同じくファイル名）で照合されます。

| レイヤー | 場所 | 優先する相手 |
| --- | --- | --- |
| ユーザー | `~/Library/Application Support/Silverpoint/Pairings/*.md`（`SILVERPOINT_PAIRINGS_DIR`） | パッケージ、付属 |
| パッケージ | リンクしたデザインシステムパッケージの `pairings/*.md`（`design-system-packages.md` を参照） | 付属 |
| 付属 | `Resources/Pairings/*.md` | |

パッケージの組み合わせは、ドキュメントがそのパッケージをリンクしている間だけ使えます。リンクした 2 つのパッケージが同じスラッグを定義しているときは、先のものが優先されます。ユーザーフォルダやパッケージの `pairings/` にあるファイルは付属のものと同じフロントマターを持ち、`title`、`summary`、`heading`、`body` を指定する必要があります。そのどれかがない、フロントマターがない、または UTF-8 テキストでないファイルは、そのファイル名を示す警告とともにスキップされ（同じスラッグのファイルが 2 つあるときは、ファイル名のバイト順で先のものが優先され、もう一方は警告になります）、残りは読み込まれます。moods と uses は自由な語です。上の付属の語彙が縛るのは付属の組み合わせだけで、絞り込みは組み合わせに書かれている語とそのまま照合します。自分で書いたファイルの書体は確認されません（`save_font_pairing` は確認します）。インストール済みのファミリーを指定するか（そうでないとデザインはシステムフォントで表示されます）、`fonts: library`（と `fallback:`）を付けてフォントライブラリのファミリーを指定してください。後者は、`pairing` を付けた `add_library_font` が、付属のライブラリの組み合わせと同じようにドキュメントに持たせます。`get_font_pairing` は実行時にレイヤーを解決するので、ターンの合間にフォルダへ追加したファイルも次の呼び出しで使えます。ただし説明と `name` の選択肢は、ツール一覧を作った時点の組み合わせを示します。エージェント、MCP（`silverpoint mcp`）、`silverpoint call` はすべて同じフォルダを読みます。

### 組み合わせの保存: `save_font_pairing`

「このフォントを組み合わせとして保存して」と頼むと、エージェントは `name`、`title`、`summary`、`heading`、`body`、任意の `mono`、`moods` と `uses`（リスト）、`notes`（フロントマターのあとの Markdown。ロールごとのサイズとウェイト、避けるべきこと）、そしてライブラリの組み合わせでは `fonts: "library"`、`fallback`、合う `products` を付けて `save_font_pairing` を呼びます。付属のものと同じ形（フロントマター、`# <title>`、ノート）で `<slug>.md` をあなたの組み合わせのフォルダに書き込み、`get_font_pairing` はそれをすぐに提供します。共有カタログのツールの 1 つで（エージェント、MCP、`silverpoint call`）、`save_style` と同じ確認の流れと、既存の名前についての同じ決まりを持ちます。

- **アプリでは** 直接書き込みません。スタイルと同じカードにファイル（または置き換える組み合わせとの差分）が「保存」と「破棄」と一緒に表示され、「保存」で書き込みます。
- **MCP と `silverpoint call` では**、`confirm: true` がなければプレビュー（`{written: false, slug, title, replaces, text, diff}`）を返し、何も書き込みません。付けると書き込み、`written: true` を返します。
- **付属の名前** も、スタイルと同じく使えます。その場合、あなたのファイルがすべてのドキュメントで付属の組み合わせを置き換えます（付属のファイルには触れません）。黙って置き換えることはなく、何かを書き込む前に、プレビューとカードが差分とともに `replaces: bundled`（または `package`、`user`）と示します。別の名前のファイル（`My Pair.md`）から来たあなたの組み合わせは、そのファイルがその場で書き換えられます。
- **理由を示して拒否し、何も書き込まないもの:** 文字や数字をひとつも含まない名前、空の title、summary、heading、body、moods、uses、notes、`installed` と `library` 以外の `fonts`、この Mac にインストールされていないファミリーを指定するインストール済みの組み合わせ（フォントライブラリにあるファミリーなら `fonts: "library"` を使うよう伝えます）、フォントライブラリにもなくインストールもされていないファミリーを指定するライブラリの組み合わせ、インストール済みの組み合わせでない `fallback`、プロダクトの種類でないプロダクト、そして組み合わせのフォルダのない構成。

### ドキュメント内のフォント

ドキュメントは自分のフォントファイルを持つことができます。そのため、この Mac にインストールされていないファミリーでも、ここでも、次にそのファイルを開く Mac でも描画されます。テキストのインスペクタでは、フォントファミリーのメニューの、インストール済みのファミリーの上にある独自のセクション **この書類内** にドキュメントのフォントが並びます。**フォントファイルを追加…** は TrueType（`.ttf`）、OpenType（`.otf`）、コレクション（`.ttc`）のファイルを選ぶパネルを開き、選んだファイルをドキュメントに持たせます（1 回の取り消しステップ、フォントを追加）。ライセンスファイル（`OFL.txt`、`LICENSE.txt`、`<family>-OFL.txt` など）が隣にあればそれも一緒です。**フォントを削除** で持たせるのをやめます（フォントを削除。ほかの編集と同じように取り消せます）。受け付けるのは CoreText がフォントとして解析できる 50 MB までのファイルだけで、それ以外はメッセージとともに拒否され、ファイルの中身が実行されることはありません。フォントの追加は人が行う操作です。パスを読んでフォントを追加するエージェントや MCP のツールはありません。

各フォントは `document.json` の `fonts` の項目で（id、ファイルから読んだファミリー、スタイル、ウェイト、イタリック、PostScript 名、パッケージの `fonts/` フォルダ内のファイル名、任意のライセンスファイル、そして出どころ。出どころは、追加したファイルか、id とバージョンで示すライブラリのフォントです）、ファイルは `assets/` の隣の `fonts/` に置かれます。保存では項目が指定するファイルだけが書き込まれるので、削除したフォントのファイルは次の保存でなくなります。バージョン履歴は各バージョンが使ったファイルを保持するので、バージョンを復元するとそのフォントも戻ります。フォントを持つドキュメントは `embeddedFonts` の機能を宣言するので、古い Silverpoint はフォントを落とさずに読み取り専用で開きます。フォントのないドキュメントは変わりません。

ドキュメントを開いている間、そのフォントは Silverpoint のプロセスだけに登録されます（一時フォルダのコピーから、CoreText のプロセススコープで）。そのため、キャンバス、テキストの計測、レンダリング、スクリーンショット、PNG、PDF、SVG の書き出しでそのフォントが描かれます。あなたのフォントフォルダにインストールされることはなく、それを使う最後のドキュメントを閉じると登録が解除されます。`silverpoint render`、`score`、`call`、`mcp` も、開いたドキュメントのフォントを同じように登録します。**インストール済みのフォントが常に優先されます。** ファミリー名か PostScript 名がインストール済みのフォントと同じファイルは登録されず、インストール済みのフォントが描かれ、メニューと `list_fonts` がそう示します。`list_fonts` は、ドキュメントのフォントを先に、それぞれのファイル、出どころ、ライセンス、登録されているかどうかとともに一覧にし、そのあとにインストール済みのファミリーを挙げます。組み込みのリントルール `missing-font` は、ファミリーがインストールされておらずドキュメントにも含まれていないテキストとテキストスタイルをそれぞれ示します（描画はこれまでどおりシステムフォントにフォールバックします）。どのフォントを使うかは選択の問題なので、`fix_lint` はそれをあなたに任せます。

### フォントライブラリ

テキストのインスペクタのフォントファミリー ▸ **Google Fonts…** で、フォントライブラリが開きます。デザイナーがよく使う [Google Fonts](https://fonts.google.com) の 56 のファミリーを選んだもので、カテゴリ（sans serif、serif、display、monospace、handwriting）別に、それぞれいくつかのムードと用途のタグ（`ui`、`editorial`、`poster`、`code`、`japanese`）が付いています。Inter、Roboto Flex、DM Sans、Manrope、Space Grotesk、IBM Plex Sans、Fraunces、Source Serif 4、Playfair Display、Newsreader、EB Garamond、Bricolage Grotesque、Syne、Bebas Neue、JetBrains Mono、IBM Plex Mono、Fira Code、Caveat、日本語では Noto Sans JP（9.6 MB）、Noto Serif JP（13.6 MB）、M PLUS 1（4.2 MB）などがあります。各行は、ファイルが手元にあれば（ドキュメントの中か、キャッシュに）ファミリー名をその書体でプレビューし、未ダウンロード、ダウンロード中、ダウンロード済み（キャッシュ）、このドキュメント内のどれかを示し、**追加** があります。追加すると、ファミリーのファイルとライセンスを取得して確認し、上の仕組みでドキュメントに持たせ（1 回の取り消しステップ、フォントファミリを追加）、選択中のテキストにそのファミリーを適用します。失敗したとき（オフライン、一致しないファイル）はそう伝え、何も変えません。

カタログは同梱され、固定されています。`Packages/SilverpointKit/Sources/SilverpointAgent/Resources/Fonts/library.json` は、各ファミリーの名前、カテゴリ、タグ、`ofl/<family>/OFL.txt` のライセンス、そしてファイルごとに、コミット `6c1dbe27dcf7d817d4b62cec033ba77fa8e2a5ec` の [google/fonts](https://github.com/google/fonts) リポジトリでのパス、スタイル（roman または italic）、ウェイトの範囲と軸、バイト数、SHA-256 を挙げています。可変フォントのファイルが優先され（1 つか 2 つのファイルですべてのウェイトをまかないます）、静的なファイルしかないファミリーはそれぞれを挙げます。入っているのは `ofl/`（SIL Open Font License）の下のファミリーだけで、`apache/` や `ufl/` のものはありません。`Scripts/fonts/build-library-catalog.py` が、スクリプト内の選定リストからこれを生成します（リストやコミットを変えたら、ネットワークにつないだ状態で実行し直し、`FILTER=FontLibraryToolTests make test` でカタログの形を確認します）。

**取得するのは必要なときだけです。** 起動時やバックグラウンドで、またライブラリの一覧や検索のために何かをダウンロードすることはありません。リクエストはファミリーを追加したとき（ブラウザの **追加**、`add_library_font`、`silverpoint fonts add`）だけ、`https://raw.githubusercontent.com/google/fonts/<commit>/<path>` から直接行われ、API キーも fonts.googleapis.com もほかのリクエストも使いません。各ファイルは、使う前にサイズと SHA-256 がカタログと一致しなければならず（一致しなければエラーになり、バイト列は破棄されます）、フォントファイルは CoreText で解析できなければなりません。確認済みのファイルは `~/Library/Caches/Silverpoint/Fonts`（`SILVERPOINT_FONT_CACHE_DIR` で変更できます）の、コミットとリポジトリのパスの下にキャッシュされるので、次にそのファミリーを追加するドキュメントはディスクから読みます。キャッシュしたファイルも使う前にもう一度確認されます。`FontLibrary`（SilverpointCore）は、注入されたクライアントとキャッシュフォルダを通してこれを行い、どちらもデフォルトではありません。標準のものを渡すのはアプリのドキュメントと CLI の `call`、`mcp`、`fonts` だけで、`SILVERPOINT_FONT_LIBRARY_OFFLINE=1` はクライアントを外します（キャッシュ済みのファミリーだけを追加できます。CLI のテストはこれを設定します）。

`fonts` にあるライブラリのファミリーの項目は、出どころが `library:<family>@<commit>` で（`list_fonts` に表示されます）、ファイルは `<File>-Variable.ttf`（リポジトリの軸のリストを単純にしたもの）として、ライセンスはその隣に `<Family>-OFL.txt` として保存されます。すでにドキュメントにあるファミリーが 2 回追加されることはありません。可変フォントのファミリーは、各 `fontWeight` を `wght` 軸のそのウェイトで（ファミリーの範囲内に収めて）描くので、300 から 900 までがそれぞれそのウェイトで描かれます。

エージェント、MCP、CLI では: `library: true` を付けた `list_fonts` は、カタログをカテゴリ別に、タグ、ウェイト、各ファミリーの状態とともに一覧にします（`category` と `query` で絞り込めます）。`add_library_font` はそこにあるファミリー名を受け取り（URL やパスは受け取りません）、そのファミリーを 1 回の取り消しステップでドキュメントに持たせます。`silverpoint fonts [<query>] [--category c] [--json]` はカタログと、各ファミリーがキャッシュ済みかどうかを一覧にし（ダウンロードはしません）、`silverpoint fonts add <file.silverpoint> <family>` で 1 つ追加します。`silverpoint call <file> add_library_font '{"family":"Inter"}'` でも同じです。スキーマのスキルとシステムプロンプトは、ライブラリのファミリーは追加したあとなら使ってよく、インストールされておらず、ドキュメントにもなく、ライブラリにもないファミリーは決して使わないようエージェントに伝えます。見つからないファミリーがライブラリにあるとき、`missing-font` は `add_library_font` を案内します。

**ライセンス。** ライブラリのファミリーはすべて SIL Open Font License 1.1 の下にあります。このライセンスは、ライセンスを一緒に添える限り、フォントの使用、埋め込み、ドキュメントと一緒の再配布を認めています（フォント単体での販売は認めていません）。そのため、ファミリーの `OFL.txt` は必ず取得され、ファイルと一緒にドキュメントに保存されます。フォントの作者はそれぞれのライセンスに記されています。Google Fonts と google/fonts リポジトリは、フォントの取得元です。

### 書き出しでのフォント

ドキュメントが持つフォントを使う書き出しは、そのフォントも一緒に持ち出します。そのため、誰もフォントをインストールしなくても、結果はキャンバスと同じ見た目になります。共有の 1 つの関数（`FontExport.used`、SilverpointCore）が、書き出しの範囲で実際に使われているファイルを見つけます。書き出すフレームとコンポーネントのテキスト、それらが適用するテキストスタイル、スタイル付きのラン、インスタンスのオーバーライドを、それぞれ同じファミリー（大文字と小文字は区別しません）のドキュメントのフォントと照合します。同じ傾き、次にそのウェイトを描けるファイル（そのウェイトの静的なファイル、`wght` の範囲にそれを含む可変フォントのファイル）、次に最も近いウェイトを優先します。インストール済みのフォントとシステムフォントは照合されないので、コピーされることもありません。ドキュメントが持つフォントでも、範囲で使われていないものはコピーされません。範囲でドキュメントのフォントを使わない書き出しは、以前とバイト単位で同じです。書き出しはストアのフォントを `DesignStore.carriedFonts` を通して読みます。ライブラリのコードにはデフォルトで何も渡されません。

コピーされるファイルには必ずライセンスファイルが付きます。ドキュメントが持つもの（ライブラリのファミリーでは `<Family>-OFL.txt`、フォントファイルでは一緒に追加したライセンスファイル）か、何も渡されていなければ、そう書いた `<font>-LICENSE-NOTE.txt` です。各書き出しが書き込むもの:

- **HTML**（`export_code`、コードの書き出し）: ページの隣に `fonts/`、そしてページの `<style>` の先頭にファイルごとの `@font-face`（ファミリー、可変フォントのファイルの `wght` の範囲またはそのウェイト、スタイル、`font-display: swap`、`src: url("fonts/<file>") format("truetype" | "opentype" | "collection")`）。どの `font-family` もフォールバックのスタックを保ちます。
- **React**: `tokens.css` の隣に同じ `fonts/`、`@font-face` のルールは `tokens.css` の先頭に。
- **ウェブサイト**（ファイル ▸ ウェブサイトを公開…、`silverpoint publish inkwell`、`export_site`）: テーマの `assets/fonts/`、ルールは `assets/tokens.css` の先頭に。
- **SwiftUI と UIKit**: ファイル、そのライセンス、それらを挙げる `README.md` を入れた `Fonts/`。README には、iOS アプリの Info.plist に入れる `UIAppFonts` の項目と、Mac アプリの Info.plist に入れる `ATSApplicationFontsPath` の項目があり、各書体の PostScript 名が記されています。コードは、CoreText が報告する PostScript 名で書体を読み込みます（`Font.custom` / `UIFont(name:)`）。そのウェイトの名前付きの書体（`Fraunces-Bold`）か、名前付きの書体がないウェイトでは、`wght` 軸を設定したファイルのデフォルトの書体です（SwiftUI では生成される `Font.carried(_:size:weight:)`、UIKit では `variedFont`）。
- **Compose**: Android のリソース名（英小文字、数字、アンダースコア: `fraunces_variable.ttf`）にしたファイルを入れた `res/font/`、`licenses/` のライセンス（テキストファイルはフォントのリソースにならないため）、そして `Fonts.kt`。`Fonts.kt` には、ドキュメントが持つファミリーごとに `FontFamily` があり、テキストが使うウェイトとスタイルごとに `Font(R.font.<name>, FontWeight.…, FontStyle.…)` が入り（可変フォントのファイルでは `variationSettings` で `wght` を設定します）、テキストはそれを `fontFamily` で指定します。
- **hand_off**: 上のターゲットのファイルを、バンドルの `code/` に。`HANDOFF.md` がそれらを挙げます。
- **SVG**: SVG は画像を `data:` URL で埋め込み、テキストをライブテキストのまま持つ自己完結した 1 つのファイルなので、使われているドキュメントのフォントも同じように埋め込まれます。SVG の `<defs>` にある、`data:` URL を使った `@font-face` です。**PNG、JPEG、PDF** はこれまでどおり登録されたフォントで描きます。

ドキュメントが持つファミリーに、テキストが求めるウェイトや傾きの書体がないとき（700 の Bebas Neue やイタリック）、書き出しは最も近いファイルを使い、何を近似したかを示す `embeddedFont` の診断を報告します（残りはブラウザ、システム、Compose が合成します）。リンクしたデザインシステムのフォントは、まだ書き出されません（BACKLOG 66）。

## プロダクトの種類とパレット

プロダクトの種類は、スタイルを選ぶ前に、ある種類のプロダクトに何が合うかを示します。どのページ用プリセットとフォントの組み合わせが合うか、色のムードと文字のムード、避けるべきこと（「banking: 遊び心のある丸い見出し書体は使わない、ネオンやグローのアクセントは使わない」）です。方向性は、それぞれのプロダクトの目的をもとに私たちが独自に書いたものです。たとえば習慣トラッカーは毎日使う落ち着いた道具なので、soft-pastel か warm-organic と親しみやすい文字が割り当てられ、「ランディングページのレイアウトにしない」がアンチパターンの 1 つです。

これらは `Resources/Products/products.md` にある 1 つの表で、プロダクトごとに 1 行（全 38 行）あり、列は次のとおりです。

| 列 | 内容 |
| --- | --- |
| `slug` | `get_style` と CLI で使うプロダクトの名前（`banking`、`habit-tracker`、`developer-tool`） |
| `title` | タイトル |
| `family` | プロダクトファミリー。パレットの名前になります |
| `presets` | 合う付属のページ用プリセットを 1〜3 個、カンマ区切りで |
| `pairings` | 合うフォントの組み合わせを 1〜3 個、カンマ区切りで |
| `color mood` | 色を表す 1 つのフレーズ |
| `type mood` | 組み合わせのムードの 1 つ（editorial、calm、technical、playful、luxurious、friendly、formal、bold） |
| `avoid` | `; ` で区切ったアンチパターン 1〜3 個。理由が自明でないときはその理由も |

ファミリーは `commerce`、`developer-tools`、`editorial-media`、`enterprise`、`finance`、`health-wellness`、`kids-education`、`productivity`、`social-community`、`travel-hospitality` です。それぞれに独自のパレットが `Resources/Palettes/<family>.md` にあり、ページ用プリセットと同じ形式です。`## Colors (mode: light / dark; define these with set_variables)` の下に同じ 10 個のロールが `#LIGHT / #DARK` の行で並び、`Apply as one batch_design operation:` の例が付きます。パレットはファミリーのためにデザインされていて（どの色も Tailwind の既定の色ではなく、アクセントが重なるものはありません）、両方のモードでプリセットと同じ確認を通ります。上のコントラストの組み合わせ、反転ではなくより暗いダークモード、操作ランナーを通して適用できる例、そしてライトとダークで問題のない `contrast` リントです。

`get_style` に `product` を付けて `name` を付けないと、そのプロダクトの方向性を返します。ファミリーを含む `Product:` の行、色と文字のムード、合うプリセットと組み合わせの索引行（解決済みのスタイルと組み合わせのとおりなので、ユーザーによる上書きの概要が表示されます）、アンチパターン、そしてファミリーのパレットの Colors セクションをそのまま返します。プロダクトはゆるく照合されます（`Habit Tracker` も `HABIT_TRACKER` も `habit-tracker` です）。不明なプロダクトは、すべてのファミリーとそのプロダクトを示すツールエラーになります。`name` を付けると、`get_style` はこれまでどおりそのスタイルを返して `product` を無視し、どちらもないと両方を挙げるエラーになります。プロダクト自身の組み合わせ（すべてインストール済みで、デフォルトのままです）のあとに、方向性は `products` にそのプロダクトを挙げているライブラリの組み合わせを、`Library pairings that also fit (Google Fonts, fetched when added; the installed pairings above stay the default):` の下に、それぞれの索引行に印を付けて提案します。どのライブラリの組み合わせにも挙げられていないプロダクトには、このセクションはありません。スタイルプリセットは組み合わせを指定しないので、変わりません。`silverpoint styles --product <slug>` は同じテキストを出力し、`silverpoint styles --products` はプロダクトの種類をファミリー別に一覧にします（下のコマンドラインを参照）。`design-principles` は、スタイルを選ぶ前に `get_style {product}` を読むようエージェントに求めます。

### 独自のプロダクトの種類とパレット

プロダクトの種類とファミリーのパレットは、フォントの組み合わせと同じリゾルバでレイヤー化されます。プロダクトはスラッグで、パレットはファミリーで照合されます。

| レイヤー | 場所 | 優先する相手 |
| --- | --- | --- |
| ユーザー | `~/Library/Application Support/Silverpoint/Products/*.md` とその `palettes/<family>.md`（`SILVERPOINT_PRODUCTS_DIR`） | パッケージ、付属 |
| パッケージ | リンクしたデザインシステムパッケージの `products/*.md` と `palettes/*.md`（`design-system-packages.md` を参照） | 付属 |
| 付属 | `Resources/Products/products.md` と `Resources/Palettes/<family>.md` | |

ユーザーフォルダ（またはパッケージの `products/`）のファイルには、付属のものと同じ形式のプロダクトの表を 1 つ以上置きます。同じヘッダー行 `| slug | title | family | presets | pairings | color mood | type mood | avoid |` とその `---` の行が必要です。ファイルの数に制限はなく、ファイル名のバイト順に読まれ、行は順番どおりに読まれます。スラッグとファミリーはスラッグ化され（`Pet Care` は `pet-care`）、ファミリーのパレットは表の隣にある `palettes/<family>.md` ファイルで、付属のものと同じ形式です。`title` と `summary` のフロントマターと、10 個すべてのロールを `- role: #RRGGBB / #RRGGBB`（ライト / ダーク）で書いた `## Colors` セクションを持ちます。パッケージとユーザーフォルダのパレットは、下のレイヤーにある同じファミリーのパレットを置き換えます。パッケージのものはドキュメントがそれをリンクしている間だけ使え、リンクした 2 つのパッケージが同じスラッグやファミリーを定義しているときは先のものが優先されます。

これらのフォルダにあるものがエラーになることはありません。セルの数が違う行、slug、family、presets、pairings、avoid のどれかが空の行（title とムードは空でもかまいません）、そのレイヤーにすでにあるスラッグを持つ 2 つ目の行、空行（表の終わりになります）のあとの行、プロダクトの表がないファイルは、ファイル名と行番号を示す警告とともにスキップされます（`skipped mine.md line 5: has 4 cells, not 8`）。`## Colors` セクションがない、ロールが欠けている、またはロールが `#RRGGBB / #RRGGBB` の形で書かれていないパレットは、ファイル名とロールを示す警告とともにスキップされ、そのファミリーを引き受けないので、付属またはパッケージのパレットがそのまま使われます。警告は `silverpoint styles --product` と `--products`（stderr）と設定に表示され、エージェントと MCP は読み込めないものをスキップします。

<h4 id="contrast-checks-on-your-palettes">あなたのパレットのコントラストの確認</h4>

読み込まれたパレットは、ライトとダークで WCAG AA に照らして確認されます（`PaletteContrast`、SilverpointAgent）。比は、`contrast` リントルールと `lowContrast` のレイアウトの問題が使うのと同じもの（`RGBA.contrast(with:)`）です。組み合わせは次のとおりで、それぞれ前景色を、その下に描かれる背景色に対して確認します。

| 前景色 | 背景色 | 最小値 |
| --- | --- | --- |
| `ink`（面の前景色） | `background`、`surface` | 4.5:1 |
| `muted`（控えめな前景色） | `background`、`surface` | 4.5:1 |
| `onAccent` | `accent` | 4.5:1 |
| `onDestructive` | `destructive` | 4.5:1 |
| `destructive`（エラーテキスト） | `background` | 4.5:1 |
| `accent`（コントロールの塗り） | `background` | 3:1 |
| `focus`（フォーカスリング） | `background`、`surface` | 3:1 |

`border` は装飾なので確認しません。確認を通らなくてもパレットは読み込まれます。これは警告で、組み合わせ、モード、小数点以下 2 桁で切り捨てた比（最小値に切り上げられることはありません）として、`onAccent on accent, dark: 2.31:1 (needs 4.5:1)` のように 3 か所に表示されます。設定のパレットの行の下、`get_style` に `product` を付けたときの結果のパレットのあと（`Contrast warnings (this user palette fails WCAG AA; …)`。そのためエージェントと MCP にも見えます）、そして `silverpoint styles --product` と `--products` の stderr（`palette pets (user): …`）です。確認されるのは、あなたのパレットとリンクしたパッケージのパレットです。付属のパレットはどれもすべての確認を通ります（`PaletteContrastTests`）。

プロダクトは、どのレイヤーにもパレットがないファミリーを指定してもかまいません。その場合、方向性の最後は `Palette (<family>): no palette for this family.` になります。指定したプリセットや組み合わせが解決済みのスタイルや組み合わせにないときは一覧から外され、一覧の最後に `(not found, left out: <slug>, <slug>)` の行が 1 つ付きます。`get_style` は実行時にレイヤーを解決するので、ターンの合間に追加した行も次の呼び出しで使えます。ただし `product` の選択肢はツール一覧を作った時点のプロダクトの種類を示し、不明なプロダクトのエラーは解決済みのファミリーとそのプロダクトを示します。エージェント、MCP（`silverpoint mcp`）、`silverpoint call`、`silverpoint styles` はすべて同じフォルダを読みます。

## コマンドライン

*ベータにはまだ含まれていません。`silverpoint` コマンドラインツールはダウンロード版に入っていないため、この節はソースからビルドした場合の説明です。*

    silverpoint styles [<file.silverpoint>] [--json] [--show <name>]
    silverpoint styles --product <slug> [--json]
    silverpoint styles --products [--json]
    silverpoint styles --pairings [--fonts installed|library] [--json]
    silverpoint templates [--json]

`styles` は解決済みのスタイルを 1 行ずつ（ドキュメントを指定すると、それがリンクするパッケージのものも）`slug<TAB>source[ (overrides bundled)]<TAB>title` の形で、`--json` を付けると `{slug, title, summary, source, overrides}` の配列で一覧にします。`--show <name>` は `get_style` が返すテキストを出力します（不明な名前では終了コード 1）。`--product <slug>` は `get_style` に `product` を付けたときの結果を出力し（`--json` を付けると `{slug, title, family, body}`。不明なプロダクトではツールのエラーとともに終了コード 1）、`--products` はプロダクトの種類を `slug<TAB>family<TAB>title` の形でファミリー順、次にスラッグ順で一覧にします（`--json` を付けると `{slug, title, family}` の配列）。どちらも、一緒に使えるフラグは `--json` だけです。どちらも stderr に、プロダクトのフォルダの警告と、そのあとにあなたのパレットとリンクしたパッケージのパレットで通らなかったコントラストの確認をそれぞれ出力します（[あなたのパレットのコントラストの確認](#contrast-checks-on-your-palettes)）。`--pairings` は `get_font_pairing` が提供するフォントの組み合わせを `slug<TAB>layer<TAB>fonts<TAB>title`（fonts は `installed` または `library`）の形で、インストール済みのものを先に、次にスラッグ順で、`--json` を付けると `{slug, title, summary, source, fonts, heading, body, mono, fallback}` の配列で一覧にします。`--fonts` を付けると一方の種類だけにします。`SILVERPOINT_PAIRINGS_DIR` を読み、ダウンロードはしません。`templates` はユーザーテンプレートを 1 行に `name<TAB>absolute path` の形で、`--json` を付けると `{name, file}` の配列で一覧にします。フォルダがなければ何も表示せず終了コード 0 で終わります。警告は stderr に出力されます。どちらも読み取り専用で、`SILVERPOINT_STYLES_DIR` と `SILVERPOINT_TEMPLATES_DIR`（`--product` と `--products` はプロダクトフォルダの `SILVERPOINT_PRODUCTS_DIR`）に従い、対応していないフラグでは終了コード 2 で終わります。

これらの変数の古い `VELLUM_` の綴りも、ほかのすべての `SILVERPOINT_*` 変数と同じく引き続き使えます。
