# news-paper-loacl

<!--
概要: Cursor だけで日次トレンドを `ideas/daily/` にまとめるワークスペース。
主な仕様: `.agents/skills/`、`.cursor/rules/`、`.cursor/commands/`。
制限事項: サイト仕様変更で取得できない日はスキルどおり注記する。初回実行時に `ideas/daily/` はエージェントが作成する。
注意: 毎日勝手に更新などの処理はないので、自分でコマンドを動かす。
-->

技術トレンドを日次で集め、`ideas/daily/YYYY-MM-DD-trend.md` に保存するためのローカル用リポジトリです。

## Codex スキル（`.agents/skills`）

- [`.agents/skills/neta-trend-daily/SKILL.md`](.agents/skills/neta-trend-daily/SKILL.md) … 収集元 URL、取得・評価ルール、Markdown の章立てと表の列定義

## Cursor 上のワークフロー（スラッシュコマンド）

チャットで `/` を押すと、[`.cursor/commands/`](.cursor/commands/) のコマンドを選べます。

| コマンド | ファイル | 用途 |
|---------|----------|------|
| `daily-trend-workflow` | [`daily-trend-workflow.md`](.cursor/commands/daily-trend-workflow.md) | 収集から `ideas/daily/YYYY-MM-DD-trend.md` への書き出しまで（フェーズ分割） |

**Agent モード** をオンにしてから `/daily-trend-workflow` を実行するのがおすすめです。

## Cursor Agent（自由入力）

1. このフォルダを Cursor のワークスペースとして開く
2. Agent モードで例:

```text
今日のトレンドを収集して
```
もしくは
ワークフロー
```text
/daily-trend-workflow 
```
を実行

3. `.cursor/rules/daily-trend.mdc` と `.agents/skills/neta-trend-daily/SKILL.md` に従い、`ideas/daily/YYYY-MM-DD-trend.md` を作成する

## 成果物

- `ideas/daily/YYYY-MM-DD-trend.md`（初回はディレクトリごとエージェントが作成）

## AGENTS.md

[`AGENTS.md`](AGENTS.md) にリポジトリ共通の短い方針があります。
# news-pepar-local
