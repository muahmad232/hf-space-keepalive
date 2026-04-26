# hf-space-keepalive

A lightweight GitHub Actions workflow that automatically pings a [Hugging Face Space](https://huggingface.co/spaces) on a regular schedule to prevent it from going to sleep due to inactivity.

---

## Table of Contents

- [What It Does](#what-it-does)
- [How It Works](#how-it-works)
- [Key Features](#key-features)
- [Prerequisites](#prerequisites)
- [Configuration](#configuration)
- [Usage](#usage)
  - [Automatic (Scheduled)](#automatic-scheduled)
  - [Manual Trigger](#manual-trigger)
  - [Customising the Target Space](#customising-the-target-space)
- [Workflow File Reference](#workflow-file-reference)
- [Troubleshooting](#troubleshooting)
- [Security Notes](#security-notes)
- [License](#license)

---

## What It Does

Free-tier Hugging Face Spaces are paused after a period of inactivity (typically ~48 hours). This project keeps your Space awake by sending an HTTP `GET` request to its public URL every **30 minutes** via a scheduled GitHub Actions workflow.

---

## How It Works

1. GitHub Actions triggers the workflow on a cron schedule (`*/30 * * * *` — every 30 minutes).
2. The `ping` job runs on `ubuntu-latest` and executes a single `curl` command against the target Hugging Face Space URL.
3. The HTTP request is enough to register activity and reset the Space's inactivity timer, preventing it from sleeping.

```
GitHub Actions scheduler
        │
        ▼  (every 30 min)
  ubuntu-latest runner
        │
        ▼  curl <HF_SPACE_URL>
  Hugging Face Space  ←── stays awake ✔
```

---

## Key Features

| Feature | Detail |
|---|---|
| **Zero dependencies** | Uses only `curl`, which is pre-installed on every GitHub-hosted runner |
| **Fully automated** | Runs on a cron schedule — no manual intervention required |
| **Manual override** | Can be triggered at any time via the GitHub Actions UI (`workflow_dispatch`) |
| **Free to run** | Uses GitHub Actions free tier minutes (public repositories get unlimited minutes) |
| **Easy to adapt** | Change the target URL in one line to keep any HTTP-accessible Space alive |

---

## Prerequisites

| Requirement | Notes |
|---|---|
| **GitHub account** | Required to fork/clone this repository and run GitHub Actions |
| **Hugging Face Space** | The Space must be publicly accessible via HTTPS |
| **No tokens required** | The workflow pings a public URL — no Hugging Face API token is needed |

> **Note:** If your Space requires authentication, see [Configuration](#configuration) for how to pass a token securely.

---

## Configuration

### Target URL

The Space URL is currently hardcoded in `.github/workflows/keepserveralive.yml`:

```yaml
run: |
  curl https://muahmad123-dark-tower-chatbot.hf.space
```

Replace `https://muahmad123-dark-tower-chatbot.hf.space` with your own Space's URL (see [Customising the Target Space](#customising-the-target-space)).

### Schedule

The default cron expression runs every 30 minutes:

```yaml
schedule:
  - cron: "*/30 * * * *"
```

You can adjust this to any valid [POSIX cron expression](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule). For example, every hour:

```yaml
- cron: "0 * * * *"
```

### Using a Secret Token (optional)

If your Space requires a Bearer token, store it as a [GitHub Actions secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets) (e.g. `HF_TOKEN`) and reference it in the workflow:

```yaml
- name: Ping Hugging Face Space
  env:
    HF_TOKEN: ${{ secrets.HF_TOKEN }}
  run: |
    curl -H "Authorization: Bearer $HF_TOKEN" https://<your-space>.hf.space
```

---

## Usage

### Automatic (Scheduled)

After you fork or push this repository to GitHub, the workflow runs automatically every 30 minutes. No additional setup is required.

1. Fork or clone this repository to your own GitHub account.
2. Verify that GitHub Actions is enabled for the repository (**Settings → Actions → Allow all actions**).
3. The workflow will appear under the **Actions** tab and run on schedule.

### Manual Trigger

To run the workflow immediately:

1. Go to your repository on GitHub.
2. Click the **Actions** tab.
3. Select **Keep Hugging Face Space Alive** from the left-hand list.
4. Click **Run workflow → Run workflow**.

### Customising the Target Space

1. Open `.github/workflows/keepserveralive.yml`.
2. Replace the hardcoded URL with your Space's URL:

```yaml
- name: Ping Hugging Face Space
  run: |
    curl https://<your-username>-<your-space-name>.hf.space
```

3. Commit and push the change. The next scheduled (or manual) run will ping your Space.

---

## Workflow File Reference

**File:** `.github/workflows/keepserveralive.yml`

```yaml
name: Keep Hugging Face Space Alive

on:
  schedule:
    - cron: "*/30 * * * *"   # every 30 minutes
  workflow_dispatch:           # allows manual run

jobs:
  ping:
    runs-on: ubuntu-latest

    steps:
      - name: Ping Hugging Face Space
        run: |
          curl https://muahmad123-dark-tower-chatbot.hf.space
```

| Field | Value | Description |
|---|---|---|
| `on.schedule.cron` | `*/30 * * * *` | Triggers every 30 minutes |
| `on.workflow_dispatch` | — | Enables manual runs from the GitHub UI |
| `jobs.ping.runs-on` | `ubuntu-latest` | GitHub-hosted runner |
| `curl` target | `https://muahmad123-dark-tower-chatbot.hf.space` | The Space URL to ping |

---

## Troubleshooting

| Problem | Possible Cause | Solution |
|---|---|---|
| Workflow does not run on schedule | GitHub Actions may delay scheduled jobs on inactive repos | Trigger a manual run to re-activate; consider pushing a commit periodically |
| `curl` returns a non-2xx status | The Space URL is incorrect or the Space is private | Double-check the URL; if private, add an `Authorization` header with a token |
| Workflow is not visible under Actions | Actions may be disabled for the repository | Go to **Settings → Actions** and enable them |
| Space still goes to sleep | 30-minute interval may be too infrequent for your Space tier | Reduce the cron interval (e.g. `*/15 * * * *` for every 15 minutes) |
| Rate-limiting / bot protection | Some Spaces may reject automated requests | Add a `User-Agent` header: `curl -A "Mozilla/5.0" <url>` |

---

## Security Notes

- **No secrets are required** by default — the workflow only pings a public URL.
- If you add a `HF_TOKEN` or any other credential, **always** store it as a [GitHub Actions encrypted secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets) — never hardcode tokens in the workflow YAML.
- Regularly rotate any tokens you use and grant them the minimum required permissions.
- Be aware that the target URL is visible in the workflow file (and therefore in your public repository). If you need to keep the URL private, store it as a secret:

  ```yaml
  env:
    SPACE_URL: ${{ secrets.SPACE_URL }}
  run: |
    curl "$SPACE_URL"
  ```

---

## License

This repository does not currently include a license file. All rights are reserved by the repository owner unless a license is explicitly added. If you wish to use, modify, or distribute this project, please contact the repository owner or open an issue to request a license.
