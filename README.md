# AGENTS.md工房 テンプレート集

AI エージェントを制御する指示のテンプレートを置くリポジトリです。ここにはテンプレートだけを置きます。プロジェクトごと・リポジトリごとの完成版は、このテンプレートをもとに作り、それぞれの置き場所（対象リポジトリ、claude.ai の設定欄）に置きます。

完成版の作り方は [WORKFLOW.md](WORKFLOW.md) にあります。

## リポジトリ用（`templates/agents-md/`）

| ファイル | 内容 | 完成版の置き場所 |
|---|---|---|
| `AGENTS-template-jp.md` | AGENTS.md テンプレート（日本語・正本） | 対象リポジトリ直下の `AGENTS.md` |
| `AGENTS-template.md` | 同（英語。日本語版と同じ内容） | 同上 |
| `CLAUDE-template.md` | Claude Code に AGENTS.md を読ませる `CLAUDE.md` | 対象リポジトリ直下の `CLAUDE.md` |

## リポジトリの補助ファイル（`templates/repo-docs/`）

必要なものだけを使います。

| ファイル | 内容 | 完成版の置き場所 |
|---|---|---|
| `context-template.md` | 背景・用語・設計方針 | 対象リポジトリの `docs/context.md` |
| `progress-template.md` | 進捗・未決事項・決定の履歴・検証の記録 | 対象リポジトリの `docs/progress.md` |
| `troubleshooting-template.md` | よく起きる問題と対処 | 対象リポジトリの `docs/troubleshooting.md` |

## 権限設定・フック（`templates/agent-settings/`）

AGENTS.md のルールは助言なので、破られては困る操作は仕組みでも止めます。

| ファイル | 内容 | 完成版の置き場所 |
|---|---|---|
| `claude-code-settings-template.md` | Claude Code の権限設定と、push を見張るフック | 対象リポジトリの `.claude/settings.json`、`.claude/hooks/` |
| `codex-rules-template.md` | Codex の既定の守りと、コマンドのルール | 対象リポジトリの `.codex/rules/` |

## claude.ai 用（`templates/claude-ai/`）

| ファイル | 内容 | 貼る場所 |
|---|---|---|
| `personal-instructions.md` | 個人設定 | 設定 → Claudeへの指示 |
| `project-template.md` | プロジェクトの2欄の雛形 | プロジェクト →「何に取り組みますか？」「プロジェクトの指示」 |
| `presets/spec-writing.md` | 仕様書作成プロジェクト用の、埋め済みの2欄 | 同上 |

`personal-instructions.md` と `presets/` は、雛形ではなく埋め済みの文です。「テンプレートだけを置く」方針の例外として、ここを正本にしています。
