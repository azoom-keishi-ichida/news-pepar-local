---
name: neta-trend-daily
description: 技術トレンドの日次収集（JST 対象日のスナップショットのみ）。トレンド、ネタ収集、daily trend、はてブ、Hacker News、Reddit の依頼で使う。
---

<!--
概要: はてブIT・Hacker News・セキュリティブログ・Reddit から、JST の対象日（通常は当日）に相当する一覧だけを集め、発信ネタ向けのポップな Markdown に整形する。
主な仕様: 当日スナップショット、収集元 URL、取得・評価ルール、出力 `ideas/daily/YYYY-MM-DD-trend.md`、章立てと表の列を固定する。
制限事項: サイト仕様変更・レート制限で取得できないことがある。推測で数値・URL を埋めず、不足は本文末尾に注記する。取得キャッシュはリポジトリ内に置かない。
-->

# 日次トレンド収集（neta-trend-daily）

## 目的・いつ使うか

- 「今日のトレンド」「ネタ収集」「daily trend」など、**1 日分**の技術トレンド整理を依頼されたとき
- `ideas/daily/YYYY-MM-DD-trend.md` を新規作成・更新するとき

## 用語と出力

| 用語 | 意味 |
| --- | --- |
| `targetDate` | 集計の基準日。未指定なら JST 当日（`TZ=Asia/Tokyo date +%F`） |
| 当日スナップショット | その実行で各ソースが返す **「いまの一覧」** に相当するデータ。過去多日分をマージしない |
| 保存先 | `ideas/daily/{targetDate}-trend.md` |

- 既存ファイルを上書きする前にユーザーへ確認する

## ソース別の取り方（スナップショット）

| ソース | やること | やらないこと |
| --- | --- | --- |
| はてブ | 各ベース URL の **無印**ページを `curl`。`targetDate` が当日でないときだけ末尾に **`/YYYYMMDD` を 1 日分** 付与 | 過去 **複数日**のアーカイブをまとめて取得・マージしない |
| Hacker News | `v0/topstories` + `v0/item` で一覧化 | `time` で長期ウィンドウを切らない（フロントに近いトップをその日のネタ源とする） |
| Reddit | `hot.json` または `top.json?t=day` | `t=month` を大量取得してから日付だけ絞る方式は使わない（当日用途） |
| Aikido / Wiz | 一覧 HTML の **先頭付近**を確認しメモ | — |

出力 Markdown の先頭付近に必ず次を入れる（`今日のひとこと` の直後でよい）。

```markdown
> *収集日*: JST **YYYY-MM-DD**（当日スナップショット）
```

## 優先する興味領域

- AI（開発・セキュリティ応用）
- Web セキュリティ / サプライチェーン
- OSS / コミュニティ、個人開発 / SaaS
- キャリア / 実践（含: 生産性・働き方）
- JavaScript / TypeScript スタック

## 実行手順（エージェント）

1. `targetDate` を決める
2. 一時ディレクトリをリポジトリ外に作る: `OUT="$(mktemp -d /tmp/neta-trend.XXXXXX)"`（**`.neta-tmp-collect` をリポジトリ内に作らない**）。終了後 `rm -rf "$OUT"`
3. はてブ: 下記ベース URL を `curl` → HTML パース。カテゴリが複数ある場合は **URL 単位でマージ**し、同一 URL のブクマ数は **最大**を採用
4. HN: `topstories.json` と各 `item/<id>.json` を取得
5. Aikido / Wiz: ブログ一覧を `curl`
6. Reddit: 各サブレで `hot.json`（推奨）または `top.json?t=day&limit=10`
7. 興味領域を最優先に並べ替え・要約する
8. 次章「出力 Markdown 構造」に従い本文を生成し、`ideas/daily/{targetDate}-trend.md` に保存する
9. 取得は原則 `curl`（必要なら `WebFetch`）。`jq` で JSON をパースしてよい

## 収集元 URL

### はてブIT

- https://b.hatena.ne.jp/hotentry/it
- https://b.hatena.ne.jp/hotentry/it/%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0
- https://b.hatena.ne.jp/hotentry/it/AI%E3%83%BB%E6%A9%9F%E6%A2%B0%E5%AD%A6%E7%BF%92
- https://b.hatena.ne.jp/hotentry/it/%E3%81%AF%E3%81%A6%E3%81%AA%E3%83%96%E3%83%AD%E3%82%B0%EF%BC%88%E3%83%86%E3%82%AF%E3%83%8E%E3%83%AD%E3%82%B8%E3%83%BC%EF%BC%89
- https://b.hatena.ne.jp/hotentry/it/%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E6%8A%80%E8%A1%93
- https://b.hatena.ne.jp/hotentry/it/%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%8B%E3%82%A2

### Hacker News

- 入口: https://news.ycombinator.com/
- データ: Firebase `https://hacker-news.firebaseio.com/v0/topstories.json` と `v0/item/<id>.json`

### セキュリティブログ

- https://www.aikido.dev/blog
- https://www.wiz.io/blog

### Reddit（13 サブレ）

| 分類 | サブレ |
| --- | --- |
| セキュリティ | `netsec`, `cybersecurity` |
| AI | `OpenAI`, `LocalLLaMA`, `ClaudeCode` |
| コア | `programming`, `technology` |
| OSS / 個人開発 | `opensource`, `indiehackers`, `webdev`, `javascript` |
| キャリア / 実践 | `cscareerquestions`, `productivity` |

**取得 URL 例**（各サブレ）:

- `https://old.reddit.com/r/<name>/hot.json?t=day&limit=10`
- `User-Agent: neta-trend-collector/1.0 (trend analysis tool)`

## 取得ルール

- **はてブ**: タイトル、**元記事 URL**、ブックマーク数（permalink ではなく記事 URL）
- **HN**: タイトル、**日本語見出し**（表示用）、`https://news.ycombinator.com/item?id=...`、**元記事 URL**（あれば）、ポイント
- **Reddit**: タイトル、**日本語見出し**（表示用）、スレ URL、ups、コメント数、サブレ名
- すべての行に URL を含める
- 取得できなかったソースは本文末尾に短く列挙する

## 評価ルール

- 上記「優先する興味領域」との一致を最優先する
- **興味度**: `★★★` 直接 / `★★` 間接 / `★` 一般トレンド
- **カテゴリ**（表ではバッククォート）: `AI` `Security` `OSS` `個人開発` `キャリア` `JavaScript/TypeScript` `開発` `Web` `その他`
- **メモ列**: 発信ネタとしての使い道を一言

## 出力 Markdown 構造

プレビュー向けに **絵文字見出し**、**`<details>` 折りたたみ**、**短いリンクテキスト** を使う。

### 章の並び

1. `# 🗞️ トレンドネタ: YYYY-MM-DD`
2. `> **今日のひとこと**` … 1〜2 文
3. `> *収集日*: JST **YYYY-MM-DD**（当日スナップショット）`
4. `---` → `## 🔥 今日のピックアップ 3選`（表）
5. `---` → `## 🇯🇵 はてブIT` → `### ⭐ 注目トピック` → `<details><summary>📋 全エントリー</summary>` … 表
6. `---` → `## 🌍 Hacker News` … 同様
7. `---` → `## 💬 Reddit` → 注目表 → カテゴリ別 `<details>`（🛡️🤖💻🛠️🎯）
8. `---` → `## 🔬 セキュリティブログ確認メモ`（blockquote）

### 表の列

| ブロック | 列 |
| --- | --- |
| ピックアップ 3 選 | `#`, タイトル, ソース, 数値 |
| はてブ 注目 | タイトル, 🔖, 興味度, カテゴリ, メモ |
| はてブ 全件 | `#`, タイトル, 🔖 |
| HN 注目 | タイトル, ⬆️, 興味度, カテゴリ, メモ |
| HN 全件 | `#`, タイトル（日/英）, ⬆️, リンク（HN・元記事） |
| Reddit 注目 | タイトル, ⬆️, 💬, 興味度, カテゴリ, サブレ, メモ |
| Reddit カテゴリ別 | `#`, タイトル, ⬆️, 💬, サブレ |

### 表記の細則

- 注目表のタイトルは **約 40 字以内**の日本語リンクに寄せる
- HN 全件は **日本語タイトル** と _英題_ を並記、リンク列に HN と元記事
- セクション間は `---`、カテゴリ列は `` `AI` `` のようにバッククォート

## 禁止事項

- API キー・トークンをリポジトリや Markdown に書かない
- 取得失敗を隠してダミーの数値・URL を入れない
- はてなの **複数日アーカイブ**を束ねて「週次・2 週間分」の一覧を作らない
- `ideas/daily/` に `_reddit_cat_block.md` などの **中間スクラッチ**を置かない（断片は `/tmp` か最終ファイルのみ）
- リポジトリ内に `.neta-tmp-collect` や `neta-tmp-*` などの **取得キャッシュ用ディレクトリ**を作らない

## MCP

- Linear / Figma など本タスクに不要な MCP に依存しない。Web は `curl` と検索ツールを優先する。

## チャットのみでファイルに書けない場合

- 応答先頭を `ネタ収集完了。` とし、続けて ` ```markdown ` フェンスで **上記と同じ構造**の本文を返す。
