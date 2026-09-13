# Fix Bot Contributor

A GitHub Action that rewrites commits authored by a bot identity (like `github-actions[bot]`) to your own real git identity, then force-pushes the result. Use this if an automated commit accidentally added a bot as a "Contributor" on your repo and you want it gone.

> [!CAUTION]
> This action **rewrites git history and force-pushes**. That is destructive:
>
> - Every commit SHA from the first rewritten commit onward changes.
> - Anyone else with a clone, fork, or open pull request against this repo will need to re-clone or hard-reset - their existing copies will no longer match.
> - There is no built-in undo. The old history is only recoverable via the reflog/GitHub's internal retention for a limited time, and only by someone with direct repo access.
>
> Only use this on repos where you understand and accept that cost - typically a solo or small personal project, not a repo with active outside collaborators or forks you care about staying in sync.

## Usage

This action does **not** check out your repo itself - your workflow must do that first, with full history (`fetch-depth: 0`), since `git filter-repo` needs the complete commit history to rewrite it correctly.

```yaml
name: Fix Bot Contributor

on:
  workflow_dispatch:
    inputs:
      confirm:
        description: "Type REWRITE to confirm - this rewrites all history and force-pushes"
        required: true
        type: string
      new_name:
        description: "Your real git name"
        required: true
        type: string
      new_email:
        description: "Your real git email"
        required: true
        type: string

jobs:
  rewrite:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - uses: MPHONlC/fix-bot-contributor@Version-0.0.1
        with:
          confirm: ${{ inputs.confirm }}
          new_name: ${{ inputs.new_name }}
          new_email: ${{ inputs.new_email }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

Trigger it manually from the Actions tab, type `REWRITE` into the confirmation field, and fill in your real name/email. Leave `bot_login` unset unless the bot you're removing uses a different commit email than the standard `github-actions[bot]` one.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `confirm` | Yes | - | Must be exactly `REWRITE` or the action aborts without changing anything. |
| `new_name` | Yes | - | The real git name to rewrite matching commits to. |
| `new_email` | Yes | - | The real git email to rewrite matching commits to. |
| `bot_login` | No | `41898282+github-actions[bot]@users.noreply.github.com` | The bot identity's commit author/committer email to match against. |
| `github_token` | Yes | - | A token with `contents: write` on the target repo, used to force-push. `secrets.GITHUB_TOKEN` is sufficient for same-repo use. |

## Requirements

- The calling job needs `permissions: contents: write`.
- If the target branch has protection rules blocking force-pushes, you'll need to temporarily disable that rule, run the action, then re-enable it.
- The checkout step in your own workflow must use `fetch-depth: 0` (full history) - a shallow checkout will cause `git filter-repo` to only see partial history.

## License

MIT - see [LICENSE](LICENSE).
