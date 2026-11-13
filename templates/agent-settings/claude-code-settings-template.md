# Claude Code の権限設定・フック テンプレート

<!--
使い方（このコメントは成果物から消す）
- AGENTS.md のルールは助言で、守られる保証はない。破られては困る操作だけを、ここで仕組みとして止める。
- 止める対象は、聞き取りで「仕組みでも止めたい」と言われたものに絞る。増やしすぎると、必要な作業まで止まる。
- [ ] を埋め、使わない行は消す。JSON にはコメントを書けないので、説明は成果物に残さず、渡すときに伝える。
- 書式は公式ドキュメントで確かめた（2026-11-13）。Claude Code の更新で変わることがあるので、作るたびに確かめ直す。
  - 権限：https://code.claude.com/docs/en/permissions
  - フック：https://code.claude.com/docs/en/hooks
-->

## 1. 権限設定（対象リポジトリの `.claude/settings.json`）

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./[例: secrets]/**)",
      "Edit(./[例: vendor]/**)",
      "Edit(./AGENTS.md)",
      "Edit(./CLAUDE.md)"
    ],
    "ask": [
      "Bash(git push *)",
      "Bash(rm -rf *)"
    ]
  }
}
```

埋めるときの決まり：
- `deny` は実行させない。`ask` は実行の前に確認を出す。`deny` は、ほかのどの設定の `allow` よりも優先される。
- `Read(...)` の deny は、ファイルの読み取りに加えて、同じパスへの編集と新規作成も止める。機密ファイルに使う。
- `Edit(...)` の deny は、ファイルを編集するすべての組み込みツールを止める。変更禁止のファイルに使う。パスの指定には `Write(...)` ではなく `Edit(...)` を使う（`Write(パス)` は一致しない）。
- パスは gitignore と同じ書き方。`./` で始めるとリポジトリの直下から、`**` で配下すべてに一致する。
- Bash のルールは、`*` を必ずサブコマンドの後ろに置く（例：`Bash(git push *)`）。
- AGENTS.md の Git 運用が A（任せない）なら、`"Bash(git push *)"` を `deny` に移し、`"Bash(git commit *)"` も `deny` に足す。
- **クラウドの実行で push を任せる場合（Git 運用が B か C）は、`"Bash(git push *)"` を `ask` にも `deny` にも入れない。** 確認に答える人がいないので、作業が止まる。代わりに 2 のフックで、守りたいブランチへの push と force push だけを止める。

限界（渡すときに伝える）：
- Bash のルールは、コマンドの文字列に一致させているだけで、完全な防壁ではない。`git -C . push` のような別の書き方は一致しない。
- `Read` の deny は、ファイル名を指定しない読み方（`grep -r` や、スクリプトの中からの読み取り）までは止めない。確実に止めたいときは、サンドボックスを使う（https://code.claude.com/docs/en/sandboxing ）。
- クラウドの Claude Code でも、リポジトリの `.claude/settings.json` が読まれるかは確かめていない【推測：読まれる】。初回に、止めたい操作が実際に止まるかを試す。

## 2. フック（守りたいブランチへの push と force push を止める）

文字列の一致より確実に止めたいときに使う。`.claude/settings.json` に次を足す。

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/guard-git.sh"
          }
        ]
      }
    ]
  }
}
```

対象リポジトリの `.claude/hooks/guard-git.sh` に次を置き、`chmod +x` で実行できるようにする。`PROTECTED` を、守りたいブランチ名に書き換える。`jq` が必要。

```bash
#!/bin/bash
# PreToolUse フック：守りたいブランチへの push と force push を止める。
# 入力は標準入力の JSON。コマンドの文字列は .tool_input.command にある。
PROTECTED='[例: main|master]'

command=$(jq -r '.tool_input.command // empty')

# git ... push を含まなければ何もしない（通常の権限の流れに任せる）
if ! grep -Eq '(^|[^[:alnum:]_-])git([[:space:]]+[^;&|]*)?[[:space:]]+push([[:space:]]|$)' <<<"$command"; then
  exit 0
fi

if grep -Eq '(^|[[:space:]])(--force|--force-with-lease[^[:space:]]*|-f|--mirror|--delete|-d)([[:space:]]|$)' <<<"$command"; then
  echo "止めました：force push とリモートブランチの削除は、AGENTS.md で禁止しています。" >&2
  exit 2
fi

if grep -Eq "(^|[[:space:]:+])(${PROTECTED})([[:space:]]|\$)" <<<"$command"; then
  echo "止めました：${PROTECTED//|/・} への直接 push は、AGENTS.md で禁止しています。作業ブランチに push してください。" >&2
  exit 2
fi

exit 0
```

- 終了コード 2 で終わると、その操作は実行されず、標準エラーに書いた文が Claude に伝わる。終了コード 0 なら、通常の権限の流れに進む。
- フックによる停止は、`allow` のルールよりも先に効く。
- 置いたら、次のコマンドで動作を確かめる。1つ目は 0、2つ目と3つ目は 2 が出れば正しい。

```bash
echo '{"tool_input":{"command":"git push origin feature-x"}}' | .claude/hooks/guard-git.sh; echo $?
echo '{"tool_input":{"command":"git push origin main"}}' | .claude/hooks/guard-git.sh; echo $?
echo '{"tool_input":{"command":"git push --force origin feature-x"}}' | .claude/hooks/guard-git.sh; echo $?
```

- 限界：ブランチ名を含むコミットメッセージなどで、誤って止まることがある。止まった理由は Claude に伝わるので、コマンドを分けて実行し直せる。確実に守りたいブランチは、GitHub のブランチ保護でも守る。
