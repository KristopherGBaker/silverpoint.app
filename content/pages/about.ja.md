---
title: "概要とプライバシー"
description: "Silverpoint とは何か、そして何をどこへ送るか。"
summary: "Silverpoint とは何か、そして何をどこへ送るか。"
layout: article
---

## 概要

Silverpoint は、AI デザイナーを内蔵した Mac ネイティブのデザインキャンバスです。ベータの間は無料です。

## 作っている人

Silverpoint を作っているのは [Kristopher Baker](https://krisbaker.com/ja/) です。iOS を土台にしてきたプロダクトエンジニアで、東京の南、藤沢市に住んでいます。これまで Wolt（DoorDash）、SmartNews、Bodybuilding.com でコンシューマ向けアプリを作ってきました。Silverpoint は一人で作っているプロジェクトで、AI エージェントの力を借り、そのために作ったビルドハーネス [Sparra](https://krisbaker.com/ja/building/sparra/) で確かめながら開発しています。詳しくは [krisbaker.com](https://krisbaker.com/ja/) へ。

## アプリが送るもの

- **選んだモデルへ。** メッセージを送ると、依頼とエージェントが必要とするドキュメントの部分が、設定で選んだモデルに送られます。お使いの Claude Code や Codex（それぞれ自身のサービスと通信します）、API キーのプロバイダ、あるいは Apple のオンデバイスモデルやローカルサーバーなら外部には何も送られません。
- **設定 ▸ 一般 ▸ 画像で設定した画像プロバイダへ。** あなたやエージェントが画像を生成するときだけです。
- **Figma へ。** Figma ファイルをリンクから読み込むときだけです。
- **silverpoint.app へ。** 1 日 1 回、新しいビルドがあるかをこのサイトに確認します。設定 ▸ 一般 ▸ アップデートで止められます。

Silverpoint にはアナリティクスもアカウントもありません。API キーは Mac のキーチェーンに、ドキュメントはあなたのディスクに残ります。

## このサイト

このサイトは、自前で運用している [Umami](https://umami.is)（analytics.krisbaker.com）でページの閲覧数とダウンロードのクリック数を数えています。Cookie は使わず、ほかのサイトをまたいで追跡することもなく、ブラウザがトラッキングしないよう求めている場合は何も記録しません。ホスティングは GitHub Pages で、GitHub がサーバーログを保持します。
