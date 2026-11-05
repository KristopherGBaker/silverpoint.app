---
title: "保存されるエージェントとの会話"
description: "ドキュメントはエージェントとの会話を自分のパッケージの中に保存するので、会話はファイルと一緒に持ち運べます。ドキュメントを閉じたり終了したりしても、もう一度開けばパネルには会話がそのまま表示され、エージェントは止まったところから続けます。"
summary: "ドキュメントはエージェントとの会話を自分のパッケージの中に保存するので、会話はファイルと一緒に持ち運べます。ドキュメントを閉じたり終了したりしても、もう一度開けばパネルには会話がそのまま表示され、エージェントは止まったところから続けます。"
layout: doc
toc: true
---

ドキュメントはエージェントとの会話を自分のパッケージの中に保存するので、会話はファイルと一緒に持ち運べます。ドキュメントを閉じたり終了したりしても、もう一度開けばパネルには会話がそのまま表示され、エージェントは止まったところから続けます（BACKLOG 33）。コードは SilverpointAgent の `AgentConversationArchive`（形式）と `DesignAgent+Persistence.swift`（復元と保存）です。SilverpointCore はフォルダのバイトを運ぶだけです（`DocumentPackage.agentFolder`、`DesignStore.agentFiles`）。

## ファイル

```
Poster.silverpoint/
  document.json
  agent/
    conversation.json
    images/<sha256>.png
```

- `agent/conversation.json`: 会話です。形式バージョン 1（後述）。キーを並べ替えてエンコードするので、変わっていない会話を保存すると同じバイトが書き込まれます。
- `agent/images/<sha256-hex>.<ext>`: モデルに送った画像（スクリーンショット、選択範囲の画像、ツールが返した画像）のそれぞれです。バイトの SHA-256 で名前を付けるので、2 回送った画像も 1 回だけ保存されます（パッケージのアセットと同じ方式）。

中身のない会話は `agent/` フォルダを書き込みません。テンプレートやデザインシステムパッケージには決して作られません。会話は保存のたびに（自動保存も含め）パッケージと一緒に保存されます。これは編集ではなくドキュメントのデータなので、取り消しの手順はありません。会話の変更（ターンの終了、評価、カードの状態、待機中のメッセージや入力欄に戻されたメッセージ、新規会話、圧縮）は、`DesignStore.changedWithoutUndo` を通してドキュメントを編集済みにします。開くときの復元ではそうなりません。

## 形式 1

| フィールド | 中身 |
|---|---|
| `formatVersion` | この形式では `"formatVersion": 1`。読み手はほかのバージョンのフォルダを無視します。 |
| `provider`、`model` | 最後のターンが実行されたもの（情報のみで、これに依存するものはありません）。 |
| `items` | パネルの項目を順に並べたもの。`id`、`type`（`user`、`assistant`、`thinking`、`tool`、`notice`、`failure`、`questions`、`directions`、`skillProposal`、`lintRuleProposal`、`toolApproval`、`webSearch`、`reportedIssue`）、その種類のフィールド、そしてどの項目にも付けられる `detail` と `citations`。 |
| `feedback` | 評価。評価されたターンを終える項目の id ごと。 |
| `skillCards`、`lintRuleCards`、`approvalCards`、`resolvedCards` | 各カードの状態。項目の id ごと。 |
| `history` | 送ったとおりのモデルの履歴。`role` と `content` ブロック（後述）。 |
| `queued`、`unsent` | ターンの実行中に送られてまだ待っているメッセージと、ターンが入力欄に戻したメッセージ。 |
| `usage`、`lastRoundUsage`、`estimatedCost` | これまでの会話のトークンとコスト、そして最後のリクエストの使用量（コンテキストがどれだけ埋まっているか）。使用量には `input`、`output`、`cacheRead`、`cacheWrite`、`webSearchRequests` があり、さらにあるときだけ書き込まれる任意のキーが 2 つあります。`cacheWrite1h`（`cacheWrite` のうち 1 時間キャッシュの分）と `reportedCost`（プロバイダー自身が報告した米ドルでのコスト。OpenRouter のものなど）です。コストの `kind` は `reported`（`dollars` 付き。すべてのリクエストのコストをプロバイダーが報告したもの）、`estimated`（`dollars` 付き。料金表から。一部のリクエストだけが報告したときは部分的に）、`unknown`、`notBilled` のいずれかです。コストには、料金のわかるプロバイダーで `generate_image` が作った画像も含まれます（料金表の `images` から見積もり）。使用量に数えるのはモデルのトークンだけです。 |
| `sentFacts` | ターンの事実の「最後に送ったもの」の記録（BACKLOG 47）。各ターンが `history` のどこから始まるかと、そのターンがモデルに残したデザインシステムとコンテキストのセクション。 |

入力欄の下書きは保存されません。

履歴は KBLLM の `Message` で、その `MessageContent` には 4 つのケースがあります。それが 1 対 1 で 5 つのブロックの種類になり、さらにプレースホルダーがあります。

| ブロックの `type` | KBLLM の `MessageContent` | フィールド |
|---|---|---|
| `"text"` | `.text(String)` | `text` |
| `"image"` | `.image(data:mimeType:)` | `sha256`、`mimeType`（バイトは `agent/images/<sha256>.<ext>`） |
| `"imageOmitted"` | `.text(placeholder)` | `mimeType`（保存で省かれた画像。「画像」を参照） |
| `"toolCall"` | `.toolCall(ToolCallRecord)` | `id`、`name`、`arguments`（UTF-8 でないバイトは `argumentsBase64`） |
| `"toolResult"` | `.toolResult(callID:, ToolResult)` | `callID`、`toolName`、`isError`、`content`（または `contentBase64`） |

例（テストは実際のデコーダーでこれをデコードします）:

```json
{
  "approvalCards" : {},
  "estimatedCost" : { "dollars" : 0.0213, "kind" : "estimated" },
  "feedback" : {
    "6F1C2A1E-2B1B-4C7A-9E43-1D2F3A4B5C60" : { "rating" : "good", "reasons" : [] }
  },
  "formatVersion" : 1,
  "history" : [
    {
      "content" : [
        { "text" : "# This turn\nMode: Just do it\n\nMake a card", "type" : "text" },
        { "mimeType" : "image/png", "sha256" : "9f2c0d8e4b1a7c3e5f6a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e", "type" : "image" }
      ],
      "role" : "user"
    },
    {
      "content" : [
        { "text" : "Adding a card.", "type" : "text" },
        { "arguments" : "{\"operations\":\"[]\"}", "id" : "toolu_1", "name" : "batch_design", "type" : "toolCall" }
      ],
      "role" : "assistant"
    },
    {
      "content" : [
        { "callID" : "toolu_1", "content" : "{\"created\":[]}", "isError" : false, "toolName" : "batch_design", "type" : "toolResult" }
      ],
      "role" : "tool"
    },
    {
      "content" : [
        { "text" : "Here is the image you asked for.", "type" : "text" },
        { "mimeType" : "image/png", "type" : "imageOmitted" }
      ],
      "role" : "user"
    },
    {
      "content" : [ { "text" : "Made a card.", "type" : "text" } ],
      "role" : "assistant"
    }
  ],
  "items" : [
    { "hasImage" : true, "id" : "0D7E8C55-0F3A-4E7B-8D61-2B9A0C1D2E30", "text" : "Make a card", "type" : "user" },
    { "detail" : "arguments:\n{}", "id" : "1A2B3C4D-5E6F-4A7B-8C9D-0E1F2A3B4C50", "name" : "batch_design", "status" : "succeeded", "statusText" : "1 created", "summary" : "Card", "type" : "tool" },
    { "id" : "6F1C2A1E-2B1B-4C7A-9E43-1D2F3A4B5C60", "text" : "Made a card.", "type" : "assistant" }
  ],
  "lastRoundUsage" : { "cacheRead" : 0, "cacheWrite" : 0, "input" : 1500, "output" : 40, "webSearchRequests" : 0 },
  "lintRuleCards" : {},
  "model" : "claude-opus-5-5",
  "provider" : "anthropic",
  "queued" : [],
  "resolvedCards" : [],
  "sentFacts" : [ { "message" : 0 } ],
  "skillCards" : {},
  "unsent" : [],
  "usage" : { "cacheRead" : 0, "cacheWrite" : 0, "input" : 2500, "output" : 80, "webSearchRequests" : 0 }
}
```

## 互換性

- `agent/` フォルダのないドキュメント（会話を保存するようになる前に保存されたもの、または会話のないもの）は、これまでどおり空の会話で開き、通知も出ません。
- 読めない `conversation.json`（壊れている、またはこのバージョンが読めないフィールドがある）や、`formatVersion` が 1 より新しいものは無視されます。ドキュメントは普通に開き、保存された会話を読めなかったという通知がパネルに 1 つ表示されます。フォルダはバイト単位でそのまま保たれ、ターンが実行される（形式 1 で置き換えます）か、新規会話が確定される（フォルダを削除します）まで、どの保存でも変わりません。ドキュメントは必ず開きます。会話がそれを止めることはありません。
- ファイルがない画像や、バイトがハッシュと一致しない画像は、`imageOmitted` ブロックとして読まれます。

## 画像

ドキュメントが保持する画像は、合計で 20 MiB（20,971,520 バイト）までです。保存のたびに画像を新しいものから数え、各画像はコンテンツハッシュで 1 回だけ数えます（2 回送った画像は新しいほうの位置で数えます）。収まらない最初の画像とそれより古い画像はすべて、代わりに `imageOmitted` として保存されます。モデルにはその場所でプレースホルダーのテキスト `[Image omitted: an image was here but was dropped from the saved conversation to save space.]` が渡されるので、そこに画像があったことがわかります。古いものから省かれ、開いている会話はドキュメントを開き直すまで画像を保持します。

## 設定のスイッチ

設定 ▸ エージェント ▸ 保存された会話 に「エージェントとの会話を書類に保存」スイッチがあり、デフォルトでオンです（`AgentPreferences.keepConversation`、defaults のキー `silverpoint.agent.keepConversation`）。共有するファイルではオフにしてください。オフにすると何も書き込まれず、会話が変わってもドキュメントは編集済みにならず、`agent/` フォルダのあるドキュメントを開いても何も復元されず、開いているドキュメントは次の保存でその `agent/` フォルダが削除されます（スイッチは保存のたびに読まれます）。

## 圧縮

圧縮は、ユーザーが求めたときだけ実行されます。⋯ メニューの「会話を圧縮…」か、最後のリクエストがモデルのコンテキストの 80% 以上を読んだときに表示される通知の圧縮ボタンです（`AgentContextLimits`、バンドルされた 1 つの表）。自動でモデルを呼ぶことは決してありません。確認画面には先にコストが表示されます。送る入力（最後のリクエストの入力の合計、使用量がまだ報告されていなければ履歴のテキストの文字数 ÷ 4）と、料金表からの推定コストです（トークン単位で課金しないプロバイダーではトークン数だけ）。圧縮すると、1 回のモデル呼び出しで最後の 2 ターンを除くすべてのターンを要約します。履歴は、`# Conversation so far` で始まりその要約が続く 1 つのユーザーメッセージと、そのままの最後の 2 ターンになります。古いツールの結果と画像はなくなります。パネルはすべての項目を残し、通知を 1 つ加えます。`sentFacts` は最後の 2 ターンの開始位置をセクションなしで残すので、次のターンはすべてのセクションをもう一度まるごと送ります。履歴の先頭が変わるので、プロンプトキャッシュは 1 回ミスします。呼び出しが失敗するか要約が空なら何も変わらず、パネルに失敗が加わります。ターンの実行中、2 ターン以下のとき、CLI バックエンド（CLI は自分のコンテキストを持っています）では提供されません。

## プロバイダー

保存される履歴はプロバイダーに依存しません。KBLLM の 4 つのコンテンツのケースは、テキスト、ツール呼び出し（`toolCall`）、ツールの結果（`toolResult`）、画像で、プロバイダー固有のもの（思考の署名、暗号化された検索結果）が入ることはありません（ウェブ検索の結果はテキストとして入ります）。あるプロバイダーで保存した会話は、別のプロバイダーでも、同じプロバイダーの別のモデルでも、そのまま続けられます。次のリクエストは同じメッセージで、何も省かれず、何も加わりません。`provider` と `model` は次のターンの終わりに更新され、コンテキストの上限は現在のモデルに従います。

## CLI バックエンド

CLI バックエンド（お使いの Claude Code 経由の Claude、お使いの Codex CLI 経由の Codex）でも、パネルの項目は同じように保存されます。ただし、CLI 自身のセッションは開き直したあとには再開できません。開き直したあとの最初のターンは新しいセッションを始めます（新しいプロセスで `--resume` なし、Codex では新しいスレッド）。そのメッセージは `# Conversation so far` で始まり、復元したトランスクリプトの最後の 20,000 文字（`transcriptText`）、空行、そしてそのターンのいつものメッセージが続きます。この前置きは CLI にだけ送られ、履歴、パネル、ログの `user` 行には決して入りません。それ以降のターンと、復元されなかった会話では、いつものメッセージを送ります。

## バイトの保持

履歴は、送ったとおりにブロック単位、バイト単位で（ツールの引数と結果はそのバイトのまま）、ターンの事実の記録と一緒に保存されます。そのため、開き直した会話の次のリクエストは、保存された履歴に新しいターンを加えたものになり、ドキュメントを開いたままにしていた場合とバイト単位で同じリクエストボディになります（テストは実際の KBLLM の Anthropic のボディを比較します）。つまり、プロンプトキャッシュのプレフィックスは開き直しても残ります。ターンをまたいで、連続するリクエストは `cache_control` キーだけを除いてメッセージごとに比較されます（キャッシュのブレークポイントはリクエストごとに最新のメッセージへ移り、それ以外は変わってはいけません）。これが項目 47 の正規化です。
