---
name: neta-trend-daily
description: 技術トレンドの日次収集。トレンド、ネタ収集、daily trend、はてブ、Hacker News、Reddit の収集依頼で使う。
---

<!--
概要: はてブIT・Hacker News・セキュリティブログ・Reddit から技術トレンドを集め、発信ネタ向けに Markdown へ整形する手順を定義する。
主な仕様: 収集元URL、取得項目、興味度・カテゴリ評価、Markdown の章立てと表形式を固定する。
制限事項: サイト仕様変更・レート制限で取得できない場合がある。推測で数値やURLを埋めず、不足は本文末尾に注記する。
-->

# 日次トレンド収集（neta-trend-daily）

## いつ使うか

- ユーザーが「今日のトレンド」「ネタ収集」「daily trend」など日次の技術トレンド整理を依頼したとき
- `ideas/daily/YYYY-MM-DD-trend.md` を新規作成または更新するとき

## 対象日と保存先

- **対象日**: 特に指定がなければ JST の当日（`TZ=Asia/Tokyo` で日付を解釈する）
- **保存先**: `ideas/daily/YYYY-MM-DD-trend.md`（`YYYY-MM-DD` は対象日）
- **既存ファイル**: 上書きする前にユーザーへ確認する

## 優先する興味領域

- AI（開発・セキュリティ応用）
- Web セキュリティ / ハッキング（OWASP、脆弱性、サプライチェーン）
- OSS / コミュニティ
- 個人開発 / SaaS（Technical SEO、グロース、収益化）
- キャリア / 人生哲学（経済的自由、外資、Build in Public）
- JavaScript / TypeScript スタック

## 収集手順

1. はてブIT の人気エントリーを収集する
2. Hacker News の人気記事を収集する
3. Aikido と Wiz のブログを軽く確認する
4. Reddit の指定サブレッドを収集する
5. 興味領域との一致度を最優先で整理する
6. 下記「出力 Markdown 構造」に沿った本文を生成する
7. 可能な限り shell の `curl` で取得する（必要なら `WebFetch` を併用してよい）

## 収集元（URL）

### はてブIT

- https://b.hatena.ne.jp/hotentry/it
- https://b.hatena.ne.jp/hotentry/it/%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0
- https://b.hatena.ne.jp/hotentry/it/AI%E3%83%BB%E6%A9%9F%E6%A2%B0%E5%AD%A6%E7%BF%92
- https://b.hatena.ne.jp/hotentry/it/%E3%81%AF%E3%81%A6%E3%81%AA%E3%83%96%E3%83%AD%E3%82%B0%EF%BC%88%E3%83%86%E3%82%AF%E3%83%8E%E3%83%AD%E3%82%B8%E3%83%BC%EF%BC%89
- https://b.hatena.ne.jp/hotentry/it/%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E6%8A%80%E8%A1%93
- https://b.hatena.ne.jp/hotentry/it/%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%8B%E3%82%A2

### Hacker News

- https://news.ycombinator.com/

### セキュリティブログ

- https://www.aikido.dev/blog
- https://www.wiz.io/blog

### Reddit（サブレッド一覧）

- セキュリティ系: `r/netsec`, `r/cybersecurity`
- AI 系: `r/OpenAI`, `r/LocalLLaMA`, `r/ClaudeCode`
- コア技術系: `r/programming`, `r/technology`
- OSS / 個人開発系: `r/opensource`, `r/indiehackers`, `r/webdev`, `r/javascript`
- キャリア / 実践系: `r/cscareerquestions`, `r/productivity`

Reddit API 取得例（各サブレッド）:

- `https://old.reddit.com/r/<subreddit>/hot.json?t=day&limit=10`
- User-Agent: `neta-trend-collector/1.0 (trend analysis tool)`
- `curl` と `jq` を使ってパースする

## 取得ルール

- はてブ: タイトル、**元記事 URL**、ブックマーク数（はてブのpermalinkではなく記事URL）
- Hacker News: タイトル、**日本語訳タイトル**（表示用）、`https://news.ycombinator.com/item?id=...` 形式のコメントページ URL、ポイント数
- Reddit: タイトル、**日本語訳タイトル**、完全なコメント URL、投票数（ups）、コメント数、サブレッド名
- すべてのエントリーに URL を含める
- 取得できなかったソースは Markdown 本文の末尾に短い注記で列挙する

## 評価ルール

- 興味領域との一致度を最優先する
- 興味度: `★★★`（直接関係）、`★★`（間接）、`★`（一般的な技術ニュース）
- カテゴリは次から選ぶ: `AI`, `Security`, `OSS`, `個人開発`, `キャリア`, `JavaScript/TypeScript`, `開発`, `Web`, `その他`
- メモ列は「発信にどう使えるか」を短く書く

## 出力 Markdown 構造

`ideas/daily/YYYY-MM-DD-trend.md` に書き出す本文は、次の章立て・表構造に従う。

- 見出し: `# トレンドネタ: YYYY-MM-DD`
- `## はてブIT（日本市場）` → `### 注目トピック`（表）→ `### 全エントリー`（番号付きリスト）
- `## Hacker News（グローバル）` → 同様
- `## Reddit（13サブレッド）` → `### 注目トピック`（表）→ `### カテゴリ別エントリー`（セキュリティ系 / AI系 / OSS・個人開発系 / キャリア・実践系）

### 表の列（Markdown 表のヘッダ名と並び）

- はてブ「注目トピック」: `タイトル` | `ブクマ数` | `興味度` | `カテゴリ` | `メモ`
- Hacker News「注目トピック」: `タイトル` | `ポイント` | `興味度` | `カテゴリ` | `メモ`
- Reddit「注目トピック」: `タイトル` | `投票数` | `コメント数` | `興味度` | `カテゴリ` | `サブレッド` | `メモ`

## 出力チャネルの違い（参考）

- **このリポジトリの主運用（Cursor Agent）**: 上記 Markdown を **`ideas/daily/YYYY-MM-DD-trend.md` に直接保存**する。
- **チャットのみでファイルに書けないツール**に合わせる場合のみ、応答先頭を `ネタ収集完了。` とし、続けて ` ```markdown ` フェンスで同じ構造の本文を返す。

## MCP の扱い

- トレンド収集では不要な MCP（例: Linear、Figma）に依存しない。Web 取得は `curl` / 検索ツールを優先する。

## 禁止事項

- API キーやトークンをリポジトリや Markdown に書かない
- 取得できなかった事実を隠してダミー URL や数値を埋めない
- `ideas/daily/` に `_reddit_cat_block.md` のような **中間用スクラッチファイルを作らない**。整形用の断片はシェルの一時領域（例: `/tmp`）に出すか、**`ideas/daily/YYYY-MM-DD-trend.md` へ直接**書き込む
