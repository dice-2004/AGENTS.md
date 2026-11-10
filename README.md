# AGENTS.md工房 テンプレート集

AI エージェントを制御する指示のテンプレートを置くリポジトリです。ここにはテンプレートだけを置きます。プロジェクトごと・リポジトリごとの完成版は、このテンプレートをもとに作り、それぞれの置き場所（対象リポジトリ、claude.ai の設定欄）に置きます。

## リポジトリ用（`templates/agents-md/`）

| ファイル | 内容 | 完成版の置き場所 |
|---|---|---|
| `AGENTS-template-jp.md` | AGENTS.md テンプレート v1（日本語） | 対象リポジトリ直下の `AGENTS.md` |
| `AGENTS-template.md` | 同 v1（英語） | 同上 |
| `AGENTS-template-v2-jp.md` | AGENTS.md テンプレート v2（たたき台・短縮版） | 同上 |
| `CLAUDE-template.md` | Claude Code に AGENTS.md を読ませる `CLAUDE.md` | 対象リポジトリ直下の `CLAUDE.md` |

## claude.ai 用（`templates/claude-ai/`）

| ファイル | 内容 | 貼る場所 |
|---|---|---|
| `personal-instructions.md` | 個人設定 | 設定 → Claudeへの指示 |
| `project-template.md` | プロジェクトの2欄の雛形 | プロジェクト →「何に取り組みますか？」「プロジェクトの指示」 |
| `presets/spec-writing.md` | 仕様書作成プロジェクト用の、埋め済みの2欄 | 同上 |
