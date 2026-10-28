# bench-pr-reviewer

An OpenHands automation plugin that reviews pull requests in
[`Nexusdeveloper902/Bench`](https://github.com/Nexusdeveloper902/Bench) and posts
findings as a **comment-only** GitHub review.

Derived from the `github-pr-reviewer` plugin in
[OpenHands/extensions](https://github.com/OpenHands/extensions) (MIT). The changes
here are limited to the review policy Bench needs:

- **Never approves.** The upstream prompt asks the model to publish an `APPROVE`
  review when it finds no material issues. This fork publishes `event: COMMENT`
  in every case — it never approves, requests changes, or merges. The clean-code
  verdict is `✅ NO BLOCKING FINDINGS` instead of `✅ APPROVED`.
- **Prompt-injection hardening.** The review prompt states that the PR title,
  body, comments, commit messages, diffs, and repository files are untrusted data,
  never instructions, and that only the prompt is authoritative.
- **Actionable findings.** The prompt requires a severity (critical/high/medium/low),
  a file and line, what is wrong, why it matters, and a suggested fix for every
  finding, grounded in the surrounding code rather than the diff alone.
- **Least privilege.** The automation uses a dedicated agent profile
  (`bench-pr-reviewer`) with only the `terminal` and `file_editor` tools.

Everything else — the scheduled scan, the head-SHA dedupe, the check gate, the
one-review-per-head rule — is upstream behaviour.

## How it reviews

A scheduled scan (every 15 minutes) lists open, non-draft pull requests, skips
heads it has already reviewed, waits for the head's required checks, then starts
one review conversation per eligible head. That covers a PR being opened,
reopened, or pushed to with new commits. A new head SHA is eligible again; an
unchanged head is never reviewed twice.

## Configuration

The plugin-spec automation supplies `config`:

| Key | Value |
|---|---|
| `repos` | `["Nexusdeveloper902/Bench"]` |
| `review_tone` | `thorough` |
| `github_token_secret` | `GITHUB_PERSONAL_ACCESS_TOKEN` |
| `trigger_label` | `openhands-review` (unused by the scheduled scan) |
| `max_new_per_run` | `2` |
