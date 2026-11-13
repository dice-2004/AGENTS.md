# Codex のルール・設定 テンプレート

<!--
使い方（このコメントは成果物から消す）
- AGENTS.md のルールは助言で、守られる保証はない。破られては困る操作だけを、ここで仕組みとして止める。
- [ ] を埋め、使わない行は消す。
- 書式は公式ドキュメントで確かめた（2026-11-13）。ルール機能は「実験的で、変わることがある」と明記されているので、作るたびに確かめ直す。
  - 承認とサンドボックス：https://learn.chatgpt.com/docs/agent-approvals-security
  - ルール：https://learn.chatgpt.com/docs/llms-full.txt の「Rules」の節
-->

## 1. 最初から効いている守り（設定しなくてよい）

Codex をローカルで使うとき、Git で管理されたフォルダでは次が既定です。

- 書き込めるのは作業フォルダの中だけ（`workspace-write`）。
- ネットワークへの接続は切られている。
- 作業フォルダの中でも、`.git`・`.codex`・`.agents` は書き込めない。そのため、コミットなどの Git 操作には承認が要る。
- サンドボックスの外に出る操作は、実行の前に確認が出る（`on-request`）。

AGENTS.md の Git 運用が A（任せない）なら、この既定のままで足ります。`danger-full-access`（サンドボックスなし）は使いません。

## 2. コマンドのルール（対象リポジトリの `.codex/rules/[例: project].rules`）

特定のコマンドを、確認つきにするか、禁止します。

```starlark
prefix_rule(
    pattern = ["git", "push"],
    decision = "prompt",
    justification = "push は確認してから行う（AGENTS.md の Git の節）",
    match = ["git push origin feature-x"],
)

prefix_rule(
    pattern = ["git", "push", "--force"],
    decision = "forbidden",
    justification = "force push は禁止（AGENTS.md の Git の節）",
    match = ["git push --force origin feature-x"],
)

prefix_rule(
    pattern = ["[例: terraform]", "[例: apply]"],
    decision = "forbidden",
    justification = "[例: 本番環境への反映は人が行う]",
)
```

埋めるときの決まり：
- `pattern` は、コマンドの先頭からの単語の並び。先頭が一致したコマンドに効く。
- `decision` は `allow`（確認なしで実行）、`prompt`（毎回確認）、`forbidden`（確認なしで止める）のどれか。複数のルールが一致したら、いちばん厳しいものが効く。
- `match` と `not_match` に例を書くと、Codex が読み込むときに、ルールが例のとおりに働くかを確かめる。
- リポジトリの中のルール（`<リポジトリ>/.codex/rules/`）は、そのプロジェクトを信頼済みにしたときだけ読み込まれる。
- 置いたら、次のコマンドで動作を確かめる。

```bash
codex execpolicy check --pretty --rules .codex/rules/[例: project].rules -- git push --force origin feature-x
```

限界（渡すときに伝える）：
- ルールは、コマンドの先頭の並びに一致させている。`git -C . push` のような別の書き方や、`--force` を後ろに置いた書き方（`git push origin feature-x --force`）は一致しない。確実に守りたいブランチは、GitHub のブランチ保護でも守る。
- クラウドの Codex で、これらのルールが読まれるかは確かめていない【不明】。
- このテンプレートのルールは、Codex を実際に動かして試してはいない（書式は公式の例に合わせた）。初回に、上の確認用コマンドで試す。
