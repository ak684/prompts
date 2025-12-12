# OpenHands SDK - Bitbucket Pipelines Demo

This repository contains demo scripts showcasing the [OpenHands SDK](https://github.com/OpenHands/software-agent-sdk) integrated with Bitbucket Pipelines. These pipelines use AI agents to automate repository analysis and security tasks.

### Network Requirements

These pipelines make outbound connections to:

| Domain | Purpose |
|--------|---------|
| `github.com` | OpenHands SDK installation |
| `raw.githubusercontent.com` | syft/grype installer scripts |
| `toolbox-data.anchore.io` | grype vulnerability database |
| LLM API endpoint | Configurable via `LLM_BASE_URL` (e.g., `bedrock-runtime.*.amazonaws.com` or `llm-proxy.app.all-hands.dev` for OpenHands Cloud) |

## Included Pipelines

| Pipeline | Script | Purpose | Modifies Code? |
|----------|--------|---------|----------------|
| `ai-readiness-report` | `ai_readiness_report.py` | Scores your repo's AI-readiness (README, agent guidelines, tests, etc.) | No |
| `sbom-report` | `sbom_report.py` | Generates Software Bill of Materials with license & CVE analysis | No |
| `cve-scanner` | `cve_report.py` | Scans for CVEs, auto-applies dependency fixes, generates report | **Yes** |

### When to Use Each

- **SBOM Report** - Run weekly/monthly for visibility, compliance audits, and license reviews. Safe to run anytime (read-only).
- **CVE Scanner** - Run when you want the agent to actually fix vulnerabilities. Creates commits with dependency version bumps.
- **AI Readiness Report** - Run to understand how AI-ready your codebase is and get recommendations.

---

## Prerequisites

- A Bitbucket repository with **Pipelines enabled** (Repository Settings → Pipelines → Enable Pipelines)
- An LLM API key (e.g., from [Anthropic](https://console.anthropic.com/))

> **Note:** The pipeline is fully self-contained. It uses a Python 3.12 Docker image and automatically installs the OpenHands SDK and all required tools (syft, grype) at runtime. No local setup required.

---

## Quick Start

### 1. Add Repository Variables

Go to **Repository Settings → Repository variables** and add these variables:

| Variable | Required | Secured | Description |
|----------|----------|---------|-------------|
| `LLM_API_KEY` | Yes | **Yes** | Your LLM provider API key (e.g., Anthropic API key) |
| `LLM_MODEL` | No | No | Model to use. Default: `anthropic/claude-sonnet-4-5-20250929` |
| `LLM_BASE_URL` | No | No | Custom LLM API endpoint (e.g., `https://bedrock-runtime.us-east-1.amazonaws.com` for AWS Bedrock). Not required when using the `openhands/` provider—the SDK auto-configures this. |

### 2. Copy Files to Your Repository

```
your-repo/
├── bitbucket-pipelines.yml    # Copy from this demo
└── scripts/
    ├── ai_readiness_report.py
    ├── sbom_report.py
    └── cve_report.py
```

> **Note:** Out of the box, the pipeline expects scripts in a `scripts/` folder and outputs reports to a `reports/` folder (created automatically if it doesn't exist). To change either location, simply modify the paths in the scripts.

### 3. Run a Pipeline

Pipelines are configured as **custom pipelines** (manual trigger):

1. Go to **Pipelines** in your repository
2. Click **Run pipeline**
3. Select branch: `main`
4. Select pipeline: `custom: ai-readiness-report` (or `sbom-report`, `cve-scanner`)
5. Click **Run**

### 4. (Optional) Schedule Pipelines

For automated runs:

1. Go to **Repository Settings → Pipelines → Schedules**
2. Click **Add schedule**
3. Configure:
   - Branch: `main`
   - Pipeline: `custom: cve-scanner` (recommended weekly)
   - Schedule: Your preferred time

---

## Using OpenHands Cloud Credits

If you have an [OpenHands Cloud](https://app.all-hands.dev) account, you can use your OpenHands credits to run these demos instead of providing your own LLM API key.

### Configuration

The OpenHands SDK automatically configures the correct API endpoint when you use the `openhands/` model prefix. You only need to set two variables:

| Variable | Value |
|----------|-------|
| `LLM_API_KEY` | Your OpenHands API key (copy from [OpenHands Cloud Settings → API Keys](https://app.all-hands.dev/settings/api-keys)) |
| `LLM_MODEL` | Any model from the [supported models list](https://docs.all-hands.dev/usage/llms/openhands-llms) with `openhands/` prefix (e.g., `openhands/claude-sonnet-4-20250514`) |

The `openhands/` prefix tells the SDK to automatically route requests to `https://llm-proxy.app.all-hands.dev`.

### Example

```bash
export LLM_API_KEY="your-openhands-api-key"
export LLM_MODEL="openhands/claude-sonnet-4-20250514"
python scripts/sbom_report.py
```

> **Network Note:** If using OpenHands Cloud, ensure the `all-hands.dev` domain is allowed in your network/firewall settings. The pipeline will need outbound access to `llm-proxy.app.all-hands.dev`.

---

## Running in Other Environments

These scripts are portable Python scripts - they work anywhere, not just Bitbucket Pipelines. Use them in GitHub Actions, GitLab CI, Jenkins, CircleCI, or locally.

**Requirements:**
- Python 3.10+
- OpenHands SDK: `pip install "openhands-sdk @ git+https://github.com/OpenHands/software-agent-sdk.git#subdirectory=openhands-sdk"`
- OpenHands Tools: `pip install "openhands-tools @ git+https://github.com/OpenHands/software-agent-sdk.git#subdirectory=openhands-tools"`
- Environment variable: `LLM_API_KEY`

**Example (local):**
```bash
export LLM_API_KEY="your-api-key"
python scripts/sbom_report.py
```

**Example (GitHub Actions):**
```yaml
- name: Run SBOM Report
  env:
    LLM_API_KEY: ${{ secrets.LLM_API_KEY }}
  run: python scripts/sbom_report.py
```

The scripts auto-install `syft` and `grype` at runtime if not present.

---

## Demo Notice

These scripts are **demos** intended to showcase OpenHands SDK capabilities. Feel free to modify them for your specific needs:

- Adjust scoring weights in `ai_readiness_report.py`
- Customize CVE fix strategies in `cve_report.py`
- Add additional analysis to `sbom_report.py`
- Modify report formats and output locations

---

## Extending: Open Pull Requests Instead of Direct Commits

By default, these pipelines commit results directly to `main`. For production use, you may want to open Pull Requests instead.

### Authentication Options

Bitbucket offers several authentication methods for API access:

| Method | Scope | Best For |
|--------|-------|----------|
| **Repository Access Token** | Single repository | CI/CD pipelines (recommended) |
| **API Token** | User-level, workspace-scoped | User-based automation |
| **App Password** | User-level (legacy) | Being phased out |

**Recommendation:** Use **Repository Access Tokens** for CI/CD. They're scoped to a single repository, making them more secure than user-level tokens.

### Creating a Repository Access Token

1. Go to **Repository Settings → Security → Access tokens**
2. Click **Create access token**
3. Name it (e.g., "OpenHands Pipeline")
4. Select permissions:
   - `repository:write` - Push commits
   - `pullrequest:write` - Create and manage PRs
5. Click **Create** and copy the token immediately (it won't be shown again)
6. Add the token as a secured repository variable (e.g., `BITBUCKET_TOKEN`)

### Example: Creating a PR via API

To modify the pipelines to create PRs instead of direct commits:

```python
import requests

# Create a PR using Bitbucket REST API
def create_pull_request(workspace, repo_slug, source_branch, title, description):
    url = f"https://api.bitbucket.org/2.0/repositories/{workspace}/{repo_slug}/pullrequests"

    headers = {
        "Authorization": f"Bearer {os.getenv('BITBUCKET_TOKEN')}",
        "Content-Type": "application/json"
    }

    payload = {
        "title": title,
        "description": description,
        "source": {"branch": {"name": source_branch}},
        "destination": {"branch": {"name": "main"}},
        "close_source_branch": True
    }

    response = requests.post(url, json=payload, headers=headers)
    return response.json()
```

You would also need to modify the pipeline to:
1. Create a feature branch instead of committing to `main`
2. Push changes to the feature branch
3. Call the Bitbucket API to create a PR

---

## Reports Output

All pipelines generate reports in the `reports/` directory:

| Pipeline | Output File |
|----------|-------------|
| AI Readiness | `reports/ai-readiness-YYYY-MM-DD.md` |
| SBOM | `reports/sbom-YYYY-MM-DD.md` + `reports/sbom-raw.json` |
| CVE Scanner | `reports/cve-YYYY-MM-DD.md` + `reports/cve-raw.json` |

---

## Questions?

For questions about the OpenHands SDK, visit: https://github.com/OpenHands/software-agent-sdk
