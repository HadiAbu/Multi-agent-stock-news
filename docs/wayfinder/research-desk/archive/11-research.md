# Ticket 11: Limits and constraints shaping the GitHub Action

## Recommendation (v1)

- Ship a **Docker container action** (Linux runners only) wrapping the Python engine. A composite action that installs Python is the fallback if image build time hurts.
- Trigger on plain `pull_request` with minimal permissions: `contents: read`, `pull-requests: write`. Read the diff through the API; never execute PR code.
- Support fork PRs with a **two-workflow split** (unprivileged `pull_request` produces the diff as an artifact; privileged `workflow_run` reviews it and posts). Document it as optional. Default v1 behaviour on forks: skip with a clear notice.
- Cap diff size, batch all inline comments into **one** review call, and degrade to a summary-only comment on failure.
- Publish from a dedicated public repo with `action.yml` at the root, semver releases, and a moving `v1` tag.

## 1. GITHUB_TOKEN permissions

- Posting a review (`POST .../pulls/{n}/reviews`) needs pull requests **write**. Creating an issue-style summary comment is listed under issues write, and also under pull requests write ([permissions reference](https://docs.github.com/en/rest/authentication/permissions-required-for-github-apps#repository-permissions-for-pull-requests)).
- Scope it with the `permissions` key at workflow or job level, and grant least access ([token docs](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication)). Suggested: `contents: read`, `pull-requests: write`. Whether the summary comment endpoint needs `issues: write` in addition is not confirmed here; test it, or post the summary as the review body, which avoids the question.
- Any action can read `github.token` even if not passed explicitly, so scoping matters ([same page](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication)).

## 2. Fork PRs and secrets

- On `pull_request` from a fork, secrets other than `GITHUB_TOKEN` are not passed and the token is read-only ([events docs](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#pull_request_target)). The reason: fork code is untrusted and could exfiltrate secrets, whereas a same-repo PR author has write access ([GitHub Security Lab](https://securitylab.github.com/resources/github-actions-preventing-pwn-requests/)). So the Anthropic key is unavailable on forks.
- `pull_request_target` runs in the base repository context with secrets and write access. Checking out and building PR code there "may lead to repository compromise" via build scripts, tests or install hooks. Label gating ("safe to test") is weak because attackers can push after approval. Set `persist-credentials: false` on checkout ([Security Lab](https://securitylab.github.com/resources/github-actions-preventing-pwn-requests/)).
- `workflow_run` is the safe pattern only if the privileged workflow **does not check out or execute PR code**. Artifacts are untrusted input: parse them as data, extract outside the workspace ([Security Lab](https://securitylab.github.com/resources/github-actions-preventing-pwn-requests/)).
- Fit for us: an LLM review only needs the diff text, so the privileged job can fetch the diff via API (or read an artifact) and never run PR code. Prompt injection from the diff remains a risk; the model should have no tools that act on the repo. (This last point is my inference, not from a cited source.)

## 3. Packaging

| Type | Pros | Cons |
|---|---|---|
| Docker container | Bundles OS, Python and dependencies; consistent | Linux only; slowest, since image build or pull adds latency |
| Composite | Works on Linux, macOS, Windows; bundles steps | Python and dependency setup happens per run; the docs page gives no tradeoffs |
| JavaScript | Linux, macOS, Windows; fastest | Must be pure JS, so a Python engine would need a rewrite |

Source: [about custom actions](https://docs.github.com/en/actions/sharing-automations/creating-actions/about-custom-actions). For a Python engine, Docker is the least friction; the Linux-only limit is acceptable for a review bot. Using a prebuilt registry image in `action.yml` (instead of a Dockerfile built each run) would reduce latency; that optimisation is my suggestion and was not verified in the docs I fetched.

## 4. Marketplace publishing and versioning

- Requirements: public repo, one `action.yml` at the root, a unique `name`, the Marketplace Developer Agreement accepted, 2FA enabled. There is no manual review; publishing is via a release ([publishing docs](https://docs.github.com/en/actions/sharing-automations/creating-actions/publishing-actions-in-github-marketplace)).
- Versioning: semver tags such as `v1.0.1`; repoint the major tag `v1` after each release; breaking input changes mean `v2`. Users can pin a full commit SHA for immutability ([managing custom actions](https://docs.github.com/en/actions/how-tos/create-and-publish-actions/manage-custom-actions)).
- Keep the action in its own repo, not this monorepo, because only one root metadata file is listed.

## 5. Runtime, size and rate limits

- Hosted-runner job limit: 6 hours; `GITHUB_TOKEN` API limit: 1,000 requests per hour per repository ([limits](https://docs.github.com/en/actions/reference/limits)). LLM latency, not GitHub, will bound runtime; set a lower `timeout-minutes` ourselves.
- Secondary limits: about 80 content-creating requests per minute and 500 per hour; on a limit, honour `retry-after`, else wait at least a minute, then back off exponentially ([rate limits](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api)). Posting each comment separately risks this; submit one review with a `comments` array ([create review](https://docs.github.com/en/rest/pulls/reviews#create-a-review-for-a-pull-request)).
- Inline comments use diff `position` (lines below the first `@@` hunk header, continuing across hunks) or `line`/`side`; docs give no maximum comment count or body length. A 422 can mean validation failure or spam, so fall back to a summary comment ([same page](https://docs.github.com/en/rest/pulls/reviews#create-a-review-for-a-pull-request)).
- Diff size: the list-files endpoint returns at most 3,000 files (30 per page by default, up to 100) and commits are capped at 250 ([pulls API](https://docs.github.com/en/rest/pulls/pulls#get-a-pull-request)). I could not verify a byte limit for the `.diff` media type. Model context is the practical limit anyway: set a file and token budget, skip lockfiles, generated and binary files, and say in the summary what was skipped.

## Unverified

- Exact fork-PR token behaviour on the dedicated token page (that page did not cover it; I relied on the events docs).
- Byte-size limit for diffs; whether summary comments need `issues: write`.
