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

## 1. 設定ファイル（対象リポジトリの `.claude/settings.json`）

権限とフックは、同じ1つの JSON に書きます。フックを使わないときは、`"hooks"` の部分（直前のカンマも）を消します。

```json
{
  "permissions": {
    "deny": [
      "Read(/.env)",
      "Read(/.env.*)",
      "Read(/[例: secrets]/**)",
      "Edit(/[例: vendor]/**)"
    ],
    "ask": [
      "Edit(/AGENTS.md)",
      "Edit(/CLAUDE.md)",
      "Edit(/.claude/**)",
      "Bash(git push *)",
      "Bash(rm -rf *)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR\"/.claude/hooks/guard-git.sh"
          }
        ]
      }
    ]
  }
}
```

埋めるときの決まり：
- `deny` は実行させない。`ask` は実行の前に確認を出す。`deny` は、ほかのどの設定の `allow` よりも優先される。
- `Read(...)` の deny は、ファイルの読み取りに加えて、同じパスへの編集も止める（新規作成まで止まるのは v2.1.228 以降）。機密ファイルに使う。
- `Read(/.env.*)` は `.env.example` も止める。見本のファイルを読ませたいときは、止めたいファイル名を1つずつ書く。
- `Edit(...)` は、ファイルを編集するすべての組み込みツールに効く。パスの指定には `Write(...)` ではなく `Edit(...)` を使う（`Write(パス)` は一致しない）。
- パスは gitignore と同じ書き方。`/` で始めると、リポジトリの直下（設定ファイルのあるプロジェクトの直下）からの指定になる。下の階層にある同じ名前のファイルも止めたいときは、`Read(**/.env)` のように `**/` で始める。
- AGENTS.md・CLAUDE.md・`.claude/` は `ask` に入れる（AGENTS.md の「ユーザーの明示的な指示なしに編集しない」と合わせるため。`deny` にすると、指示があっても直せなくなる）。
- Bash のルールは、`*` を必ずサブコマンドの後ろに置く（例：`Bash(git push *)`）。
- AGENTS.md の Git 運用が A（任せない）なら、`"Bash(git push *)"` を `deny` に移し、`"Bash(git commit *)"` も `deny` に足す。フックは要らない。
- **クラウドの実行で push を任せる場合（Git 運用が B か C）は、`"Bash(git push *)"` を `ask` にも `deny` にも入れない。** 確認に答える人がいないので、作業が止まる。代わりに 2 のフックで、守りたいブランチへの push と force push だけを止める。

## 2. フック（対象リポジトリの `.claude/hooks/guard-git.sh`）

守りたいブランチへの push、force push、リモートブランチの削除を止めます。`PROTECTED` を、守りたいブランチ名に書き換えます（複数あるときは `|` で区切る）。

```bash
#!/bin/bash
# PreToolUse フック：守りたいブランチへの push、force push、リモートブランチの削除を止める。
# 入力は標準入力の JSON。コマンドの文字列は .tool_input.command にある。
PROTECTED='[例: main|master]'

input=$(cat)
# jq がない・失敗したときは、入力の全文をコマンドとして調べる（素通りさせないため）
command=$(jq -r '.tool_input.command // empty' <<<"$input" 2>/dev/null)
[ -n "$command" ] || command=$input

block() {
  echo "止めました：$1" >&2
  exit 2
}

# 区切り文字（; & | 改行）でコマンドを分け、git ... push を含む部分だけを調べる
while IFS= read -r part; do
  grep -Eq '(^|[^[:alnum:]_./-])git([[:space:]]+[^[:space:]]+)*[[:space:]]+push([[:space:]]|$)' <<<"$part" || continue
  args=${part#*push}

  # force push とリモートブランチの削除
  if grep -Eq "(^|[[:space:]])(--force[^[:space:]]*|--mirror|--delete|--prune|-[[:alpha:]]*[fd][[:alpha:]]*)([[:space:]]|\$)" <<<"$args" ||
     grep -Eq "(^|[[:space:]])[\"']?[+:][^[:space:]]" <<<"$args"; then
    block "force push とリモートブランチの削除は、AGENTS.md で禁止しています。"
  fi

  # 守りたいブランチを名指しした push
  if grep -Eq "(^|[[:space:]:\"']|refs/heads/)(${PROTECTED})([[:space:]\"')]|\$)" <<<"$args"; then
    block "${PROTECTED//|/・} への直接 push は、AGENTS.md で禁止しています。作業ブランチに push してください。"
  fi

  # 押し先を書いていない push（git push、git push origin、HEAD を使う形）は、今のブランチで判定する
  names=$(tr -s '[:space:]' '\n' <<<"$args" | grep -Ev '^(-|$)' | wc -l)
  if [ "$names" -lt 2 ] || grep -Eq "(^|[[:space:]\"'])HEAD([[:space:]\"']|\$)" <<<"$args"; then
    current=$(git rev-parse --abbrev-ref HEAD 2>/dev/null)
    if grep -Eqx "(${PROTECTED})" <<<"$current"; then
      block "今のブランチ（$current）は守りたいブランチです。作業ブランチに切り替えてから push してください。"
    fi
  fi
done < <(sed -E 's/(\|\||&&|[;&|])/\n/g' <<<"$command")

exit 0
```

- 終了コード 2 で終わると、その操作は実行されず、標準エラーに書いた文が Claude に伝わる。終了コード 0 なら、通常の権限の流れに進む。
- フックによる停止は、`allow` のルールよりも先に効く。
- `jq` がない環境でも止まるようにしてある（入力の全文を調べる）。
- 置いたら、作業ブランチにいる状態で、次のコマンドで動作を確かめる。1つ目は 0、2つ目と3つ目は 2 が出れば正しい。

```bash
echo '{"tool_input":{"command":"git push origin feature-x"}}' | bash .claude/hooks/guard-git.sh; echo $?
echo '{"tool_input":{"command":"git push origin main"}}' | bash .claude/hooks/guard-git.sh; echo $?
echo '{"tool_input":{"command":"git push --force origin feature-x"}}' | bash .claude/hooks/guard-git.sh; echo $?
```

## 3. 限界（渡すときに伝える）

- 設定とフックは、Claude Code を起動したディレクトリの `.claude/` から読まれる。リポジトリの直下で起動する。下の階層で起動すると効かない。
- Bash のルールは、コマンドの文字列に一致させているだけで、完全な防壁ではない。`git -C . push` のような別の書き方は一致しない。フックはそれより多くの書き方を止めるが、シェルの変数や別名を使った書き方までは追えない。
- フックは、押し先を書かない push（`git push` だけ、など）を、今いるブランチで判定する。`git -C 別の場所 push` のように別のリポジトリを指す書き方では、判定を誤ることがある。
- フックは、ブランチ名を含む文字列（コミットメッセージなど）が同じコマンドにあると、誤って止めることがある。止まった理由は Claude に伝わるので、コマンドを分けて実行し直せる。
- `Read` の deny は、ファイル名を指定しない読み方（`grep -r` や、スクリプトの中からの読み取り）までは止めない。確実に止めたいときは、サンドボックスを使う（https://code.claude.com/docs/en/sandboxing ）。
- 確実に守りたいブランチは、GitHub のブランチ保護でも守る（フックや権限は、Claude Code の外からの push には効かない）。
- クラウドの Claude Code でも、リポジトリの `.claude/settings.json` が読まれるかは確かめていない【推測：読まれる】。初回に、止めたい操作が実際に止まるかを試す。
