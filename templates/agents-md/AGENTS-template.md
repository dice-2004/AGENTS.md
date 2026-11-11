# AGENTS.md Template v2

<!--
How to use (delete this comment from the finished file)
- Fill in every [ ]. Delete lines and sections that do not apply (do not leave blanks or "N/A").
- For [CHOOSE ONE], keep exactly one of the listed options (A, B, C) and delete the "[CHOOSE ONE]" line and the letters.
- Keep text outside [ ] as written, as long as it applies to the target.
- After filling in, stay under 200 lines and 32 KiB. Put procedures and detailed specs under docs/ and write only their location here.
- Write rules as checkable actions. Example: not "test carefully" but "run npm test before committing".
- When there are several stacks (such as frontend and backend), list the commands per stack. For something that does not exist yet (such as tests), keep the line and write "not set up yet" (so the AI does not search for a command that does not exist).
- For rules that must never be broken (secret files, no push, etc.), also block them with permission settings or hooks, not only with text here.
-->

# [Project name]

[One or two sentences: what this project builds and who uses it.]

## Commands
- Setup: `[e.g. npm ci]`
- Build: `[e.g. npm run build]`
- Test: `[e.g. npm test]` (single file: `[e.g. npm test -- path/to/file]`)
- Lint and format: `[e.g. npm run lint]`
- Type check: `[e.g. npm run typecheck]`
- Run commands inside [e.g. the Dev Container]. If you find you are outside that environment, report it instead of running them.

## Layout
- `[e.g. src/api/]`: [role]
- `[e.g. docs/]`: [specs to read before working]
- `[e.g. docs/context.md]`: [background, terms, and design decisions. Read before work that touches design.]
- [Outdated documents not to rely on, if any. e.g. The layout diagram in `README.md` is outdated; do not rely on it.]

## Policies
- [Decisions that fit in a few lines. e.g. Do not add a state management library. Move them to `docs/context.md` if they grow.]

## How to work
- Write replies, comments, and commit messages in [Japanese].
- When you start work on a module, read its spec (`[e.g. docs/spec/]`) if you have not read it in this session. Read it again after the conversation has been summarized or compacted.
- Ask before doing: [e.g. implementing a new feature that has no spec, adding files or modules, adding external packages, changing public interfaces, changing configuration or infrastructure]. When asking, explain which part of which file you will change, why, and how.
- Do without asking: [e.g. bug fixes inside existing functions, adding tests, fixing comments and names]. This includes fixing bugs caused by your own change that you find while verifying it.
- If the same error is not fixed after [3] attempts, stop and report what you tried, the error log, and the current state.
- Do not change files outside the scope of the request (updates to the files listed under "Records" are excepted). Report improvements you notice as proposals.
- When you are not sure about a spec or the behavior of an external API, say so and ask for confirmation instead of asserting it.
- [CHOOSE ONE] Unclear requirements
  - A: Do not fill gaps with assumptions. Collect them as questions.
  - B: For work that can be undone, proceed on an assumption and report the assumption.
- In runs where you cannot ask the user midway (such as cloud tasks), do not perform operations that need confirmation. List them as "proposed, not done" in the report or the PR description. When you reach the attempt limit, leave your changes on the branch and report the state.

## Quality and verification
- Before reporting completion, pass the test, lint, and type check commands above. Do not report completion while they fail. Do not use skip options (`[e.g. --no-verify]`).
- Separate failures that existed before your change, and checks you could not run because of the environment, from failures caused by your change. Report them as "pre-existing failure" and "not verified".
- If a code change alters the spec (inputs, outputs, design), update the spec in the same change.
- [Language and conventions. e.g. Python follows PEP 8 with required type hints. Formatter is Black.]
- [Project-specific technical rules. e.g. Access the database only through `src/db/`.]
- [Verification that commands cannot cover. e.g. After changing the UI, render the screen, check how it looks, and report which screens you checked and the result.]
- [Rules about correctness of content. e.g. For facts about the person (career, achievements), write only what was provided and do not fill gaps by guessing.]
- [Production constraints. e.g. No local absolute paths. No external network access.]

## Git
[CHOOSE ONE]
- A: Do not run Git operations that change the repository (commit, push, pull). Ask the user to run them.
- B: You may commit to the working branch. In runs where a working branch is designated (such as cloud tasks), you may also push to that branch (changes that are not pushed are lost). Otherwise, push and open a PR only when the request includes it, and ask first if it does not. Never push directly to `[main]`, force push, or rewrite history.
- C: In runs that deliver work as a branch and a PR (such as cloud tasks, or when the request names a PR or branch), follow B. Otherwise (local interactive sessions), follow A.
- [Commit message or branch naming conventions, if any]

## Protected items
- Do not edit this file (AGENTS.md) or CLAUDE.md without an explicit instruction from the user. Propose improvements instead.
- Never change: [e.g. `vendor/`, existing files under `migrations/`]
- Change only with agreement: [e.g. shared module `src/core/`]
- Never write API keys, tokens, or passwords in code, commits, logs, or chat. Read secrets from [e.g. environment variables].
- Before deleting files, stopping or restarting services, making destructive database changes (migrations, deleting records), or operating on [production or shared environments], explain the operation and get approval. Where possible, show a dry-run result first.

## Records
- [If you keep a progress record. e.g. Read `docs/progress.md` when you start. Update the current state, decisions, and verification results when a subtask is done, when a decision is made, and before you finish.]
- [If you keep a context file. e.g. Update `docs/context.md` when a design rule changes, and when you learn a lasting fact (a quirk of an external API, an approach that was tried and failed).]
- [If you keep a troubleshooting file. e.g. When an error occurs, check `docs/troubleshooting.md` first. If you solve a problem that is not listed, append its cause and fix.]
- Include updates to record files in the same commit and PR as the work. When you are not allowed to run Git operations, write the updated content in your report (in cloud runs, uncommitted changes are lost).

## Reporting
Report completion in this order.
1. What changed
2. Why
3. How it was verified and the result (write "not verified" if you did not verify)
4. Remaining issues and proposals

## Instruction priority
- When this file conflicts with an instruction in the conversation, follow the most recent explicit instruction in the conversation. The "Git" and "Protected items" sections are lifted only when the user explicitly suspends that rule.
- When the user's global settings (such as personal instructions) conflict with this file, this file takes priority for this repository.
- When a skill's instructions conflict with this file, this file takes priority.
