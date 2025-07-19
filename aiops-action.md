# AI Ops Action

Automate your GitHub workflow with Claude‑powered code review, changelogs, and issue sync.

[![Action Version](https://img.shields.io/github/v/release/your-org/aiops-action?label=action)](https://github.com/marketplace/actions/aiops-action)
[![CI](https://github.com/your-org/aiops-action/actions/workflows/test.yml/badge.svg)](https://github.com/your-org/aiops-action/actions)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

---

## Why?

* **AI‑powered automation** – Claude reviews PRs, generates changelogs, and manages issues.
* **Smart caching** – never re‑check the same content twice.
* **Multi‑trigger** – schedule daily tasks or respond to PR events.
* **Discord integration** – instant notifications for your team.
* **Featurebase sync** – automatically convert customer bugs to GitHub issues.
* **Custom prompts** – configure Claude for your specific workflows.
* **Copy quality** – catch typos and style issues in docs, HTML, and markdown.

---

## Installation

Create `.github/workflows/aiops.yml`:

```yaml
name: AI Ops
on:
  schedule:
    - cron: '0 12 * * *'  # 5AM PT daily
  pull_request:
    types: [opened, synchronize]
    paths:
      - '**/*.md'
      - '**/*.html'
      - '**/copy/**'

jobs:
  ai-ops:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: your-org/aiops-action@v1
        with:
          claude_api_key: ${{ secrets.CLAUDE_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          discord_webhook: ${{ secrets.DISCORD_WEBHOOK }}
```

---

## Quick Start

### Daily changelog generation

```yaml
# .github/aiops/prompts/changelog.md
Generate a changelog for commits since yesterday.
Focus on user‑facing changes.
Group by feature, fix, and docs.
```

```yaml
# .github/workflows/aiops.yml
- uses: your-org/aiops-action@v1
  with:
    command: changelog
    schedule: '0 12 * * *'  # 5AM PT
```

### PR style checking

```yaml
# triggered automatically on PR with .md/.html changes
- uses: your-org/aiops-action@v1
  with:
    command: style-check
    cache: true  # skip already-reviewed files
```

---

## Commands

| Command          | Description                              | Trigger            |
| ---------------- | ---------------------------------------- | ------------------ |
| `changelog`      | Generate daily changelog                 | Schedule           |
| `style-check`    | Review copy in PRs                       | PR opened/updated  |
| `issue-sync`     | Import Featurebase bugs                  | Schedule/webhook   |
| `custom`         | Run any prompt from `.github/aiops/`     | Any                |

---

## Configuration

### Environment variables

```yaml
env:
  CLAUDE_API_KEY: ${{ secrets.CLAUDE_API_KEY }}
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  DISCORD_WEBHOOK: ${{ secrets.DISCORD_WEBHOOK }}
  FEATUREBASE_API_KEY: ${{ secrets.FEATUREBASE_API_KEY }}
```

### Custom prompts

Place prompts in `.github/aiops/prompts/`:

```markdown
# .github/aiops/prompts/security-review.md
Review this PR for security vulnerabilities.
Check for exposed secrets, SQL injection, XSS.
Suggest fixes using OWASP best practices.
```

```yaml
# use in workflow
- uses: your-org/aiops-action@v1
  with:
    command: custom
    prompt: security-review
```

### Advanced options

| Option              | Description                        | Default          |
| ------------------- | ---------------------------------- | ---------------- |
| `model:`            | Claude model to use                | `claude-3-opus`  |
| `max_tokens:`       | Response length limit              | `4000`           |
| `temperature:`      | Creativity (0.0–1.0)               | `0.3`            |
| `cache_days:`       | Cache reviewed files               | `30`             |
| `batch_size:`       | Files per API call                 | `10`             |
| `fail_on_issues:`   | Fail workflow if issues found      | `false`          |

---

## Examples

### Automated morning report

```yaml
on:
  schedule:
    - cron: '0 13 * * *'  # 6AM PT

jobs:
  morning-report:
    runs-on: ubuntu-latest
    steps:
      - uses: your-org/aiops-action@v1
        with:
          command: custom
          prompt: |
            Generate a morning report:
            - Yesterday's merged PRs
            - Open high‑priority issues
            - Failed CI runs
            - Stale PRs (>7 days)
          discord_notify: true
```

### Featurebase to GitHub sync

```yaml
on:
  schedule:
    - cron: '0 * * * *'  # hourly
  webhook:
    types: [featurebase-bug-created]

jobs:
  sync-bugs:
    runs-on: ubuntu-latest
    steps:
      - uses: your-org/aiops-action@v1
        with:
          command: issue-sync
          source: featurebase
          labels: ['bug', 'customer-reported']
          assign: '@your-org/triage'
```

### Smart PR review

```yaml
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  ai-review:
    runs-on: ubuntu-latest
    steps:
      - uses: your-org/aiops-action@v1
        with:
          command: style-check
          paths: |
            **/*.md
            **/*.mdx
            **/copy/**
            **/*.html
          cache: true
          comment_on_pr: true
          suggest_fixes: true
```

---

## Prompt templates

Built‑in templates in `.github/aiops/templates/`:

```markdown
# style-check.md
Review for:
- Spelling and grammar
- Consistent terminology  
- Active voice
- Sentences ≤20 words
- No marketing fluff

# changelog.md  
Format: Keep a Changelog
Sections: Added, Changed, Fixed, Removed
Focus: User impact
Ignore: Dependencies, CI, tests

# issue-sync.md
Priority: P0 (critical), P1 (high), P2 (normal)
Labels: From Featurebase tags
Assignee: Based on component
```

---

## Contributing

Everyone is encouraged to improve this project. Fork, make changes, and open a pull request.

---

## License

MIT