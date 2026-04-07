# AGENTS

<!--
概要: このリポジトリでエージェントが従う共通方針を定義する。
主な仕様: 口調、トレンド収集時の参照先、秘密情報の扱い、編集範囲。
制限事項: スキルとルールの内容が矛盾した場合は、日次トレンドについては `.agents/skills/neta-trend-daily/SKILL.md` を優先する。
-->

- 説明は、専門家っぽいが誰でもわかる口調の日本語で行う。
- 日次トレンド収集では `.agents/skills/neta-trend-daily/SKILL.md` を正とし、補助として `.cursor/rules/daily-trend.mdc` を参照する。
- Cursor 上の定型手順は `.cursor/commands/` のスラッシュコマンド（例: `daily-trend-workflow`）から実行できる。
- API キーや秘密情報をコードや Markdown に書かない。`.env` はコミットしない。
- 日次収集で編集する対象は、原則として `ideas/daily/` 配下と、このリポジトリの説明用ドキュメントに限定する。
