<!--
概要: スラッシュコマンドから日次トレンド収集をフェーズ実行する。
主な仕様: `.agents/skills/neta-trend-daily/SKILL.md` と `.cursor/rules/daily-trend.mdc` を正とし、JST 対象日のスナップショットを `ideas/daily/YYYY-MM-DD-trend.md` に書く。
制限事項: `curl` / `jq` とネットワークが使えること。欠損は推測で埋めない。
-->

# 日次トレンド収集ワークフロー（Cursor）

**Agent モード**で実行する。各フェーズの末尾に **1〜3 行**で状況を報告する。

## フェーズ0: 正本の読み込み

- `.agents/skills/neta-trend-daily/SKILL.md`
- `.cursor/rules/daily-trend.mdc`

## フェーズ1: 対象日

- ユーザーが `YYYY-MM-DD` を指定していればそれを `targetDate` にする
- なければ次の標準出力を `targetDate` にする

```bash
TZ=Asia/Tokyo date +%F
```

## フェーズ2: 出力パスの衝突

- パス: `ideas/daily/${targetDate}-trend.md`
- 既存なら原則 **上書き可否をユーザーに確認**し、許可があるまでフェーズ3 に進まない
- **`/daily-trend-workflow` のみ**の実行は、**その日のファイルを再生成する依頼**とみなし、確認を省略して続行してよい（その旨を一行報告する）

## フェーズ3: 収集・整形

- `curl` の保存先: `mktemp -d /tmp/neta-trend.XXXXXX` 等 **リポジトリ外**。ワークスペース直下に `.neta-tmp-collect` を作らない。終了後に削除してよい
- SKILL の URL・手順に従い `curl`（必要なら `jq`）。**はてな**は無印または `targetDate` の `/YYYYMMDD` **1 日分のみ**。**Reddit**は `hot.json` 等の当日向けを優先
- SKILL の取得・評価・Markdown 構造で本文を組み立てる
- 取得できなかったソースは末尾に短く列挙。数値・URL を推測で補完しない

## フェーズ4: 書き出し

- `ideas/daily/${targetDate}-trend.md` に全文を保存する
- 目視確認: `# 🗞️ トレンドネタ:`、`*収集日*` 注記、`##` の はてな / HN / Reddit

## フェーズ5: 完了報告

- 保存パス、主な参照ソース、取得失敗の有無を簡潔にまとめる
