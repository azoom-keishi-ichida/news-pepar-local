<!--
概要: Cursor のスラッシュコマンドから日次トレンド収集を、段階付きで Agent に実行させる。
主な仕様: `.agents/skills/neta-trend-daily/SKILL.md` と `.cursor/rules/daily-trend.mdc` を正とし、`ideas/daily/YYYY-MM-DD-trend.md` へ書き出す。
制限事項: ネットワークと shell（curl/jq）が使える環境であること。取得失敗は推測で埋めない。
-->

# 日次トレンド収集ワークフロー（Cursor）

このコマンドは **Agent モード** で実行すること。以下のフェーズを **順番どおり** 完了し、各フェーズの終わりに 1〜3 行で状況を報告すること。

## フェーズ0: 正本の読み込み

次を読み、以降の判断の根拠にすること。

- `.agents/skills/neta-trend-daily/SKILL.md`
- `.cursor/rules/daily-trend.mdc`

## フェーズ1: 対象日の確定

- ユーザーがチャットで日付（`YYYY-MM-DD`）を指定していればそれを `targetDate` とする。
- 指定がなければ shell で次を実行し、標準出力を `targetDate` とする。

```bash
TZ=Asia/Tokyo date +%F
```

## フェーズ2: 出力ファイルの衝突確認

- 出力パスは `ideas/daily/${targetDate}-trend.md`。
- 既にファイルが存在する場合は **上書きしてよいかユーザーに確認** し、許可があるまでフェーズ4 に進まない。

## フェーズ3: データ収集と整形

- SKILL に記載の収集元から、可能な限り `curl`（必要なら `jq`）で取得する。
- SKILL の取得ルール・評価ルール・Markdown 構造に従い、本文を組み立てる。
- 取得できなかったソースは本文末尾に短い注記で列挙する。数値や URL を推測で補完しない。

## フェーズ4: ファイルへ書き出し

- `ideas/daily/` を確保し、`ideas/daily/${targetDate}-trend.md` に **完成した Markdown 全文** を保存する。
- 保存後、見出し `# トレンドネタ: ${targetDate}` と、はてブ / Hacker News / Reddit の各 `##` セクションが存在するか目視で確認する。

## フェーズ5: 完了報告

- 保存したパス、参照した主なソース、取得できなかったソースの有無を簡潔にまとめる。
