# Git Policy

- Commit messages are concise, English, imperative mood ("Add user card component", not
  "Added" or "Adding").
- Never add `Co-authored-by`, `Generated-by`, AI attribution, or assistant attribution lines
  to commit messages.
- Never claim co-authorship for Claude, Codex, ChatGPT, or any other AI tool.
- Before any command that could discard uncommitted work (`git checkout`/`restore`/`reset`/
  `clean`), run `git status` first and stash or commit anything found.
- Before staging broadly (`git add .`/`git add -A`), review `git status` output — don't
  stage files you haven't looked at, especially anything that could hold secrets (`.env`,
  credentials, keys) even if the filename looks innocuous.

## Husky Hooks

Husky wires the quality gate from `testing-and-quality.md` directly into git, so a bad
commit/push can't land even if the agent forgets to run checks manually.

- **pre-commit**: runs `lint-staged` — ESLint `--fix` and Prettier `--write`, but only on
  the files actually staged for this commit (not the whole repo). Fast, and only touches
  what you're already changing. If `lint-staged` can't auto-fix a lint error, the commit is
  blocked until it's fixed by hand. Also runs the non-English content check (see
  `testing-and-quality.md`) — a stray Cyrillic character in a staged file blocks the commit.
- **pre-push**: runs the full gate — `{{PACKAGE_MANAGER}} run lint`, then
  `{{PACKAGE_MANAGER}} run format:check`, then the configured test suite(s), all
  repo-wide. This is the same list as "Before every push" in `testing-and-quality.md`;
  Husky just makes it non-optional.
- Do not bypass hooks with `--no-verify` to get an unrelated commit through. If a hook is
  blocking on something unrelated to your change, fix or report it — don't skip it.
- `package.json` has a `"prepare": "husky"` script. This is what actually installs the
  hooks — without it, hooks only exist on the machine that ran `husky init` once and
  silently don't exist after a fresh clone + install anywhere else. Never remove it.
- Husky runs hook scripts with `sh -e`. A `grep` (or any command) that returns non-zero
  because it found *no match* will abort the whole script under `set -e` if it sits
  unguarded inside a pipeline, `&&` chain, or command substitution — even when "no match"
  is the success case you actually wanted. Guard it explicitly:
  `if grep -q pattern file; then echo "found"; exit 1; fi` — not
  `grep pattern file && exit 1`. This bites specifically because it only shows up when the
  check legitimately finds nothing, which is the common case in CI.
- After creating `.husky/pre-commit` and `.husky/pre-push`, make sure they're actually
  executable in git's index, not just on disk — a checkout on a filesystem that doesn't
  track the executable bit (e.g. some Windows setups) can leave the hook non-executable
  even though `chmod +x` "worked" locally. Verify/fix with:
  `git update-index --chmod=+x .husky/pre-commit .husky/pre-push`.

## Commit & push authority

{{GIT_POLICY}}
<!--
  SCAFFOLD: replace the line above with exactly one of:
  - "The agent may commit freely without asking. Pushing to the remote still requires
    explicit user confirmation each time."
  - "The agent may commit and push freely without asking, as authorized by the user during
    project setup."
  - "Every commit and every push requires explicit user confirmation before it happens."
  Pick the one matching the questionnaire answer. Do not leave the placeholder in place.
-->

## Review Checklist

- [ ] No AI attribution anywhere in the message.
- [ ] Message is English, concise, describes the *why* when it isn't obvious from the diff.
- [ ] Commit/push authority above matches what was actually done.
