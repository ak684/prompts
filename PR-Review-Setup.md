

The following examples are for GitHub but can also work in GitLab.

**Approach 1: OpenHands GitHub App**

Best for: Teams who want a no-code, managed solution out of the box

<img width="600" alt="Screenshot 2025-12-19 at 12 52 09 PM" src="https://github.com/user-attachments/assets/f62f66fa-3e16-4147-8711-27b221cee18e" />

1. Sign up at https://app.all-hands.dev with GitHub
2. Install the GitHub App
   - Click **+ Add GitHub Repos** → select repos to authorize → **Install & Authorize**
3. Trigger code review on any PR by mentioning OpenHands in a comment
   - `@OpenHands /codereview` (or `/codereview-roasted` for brutal mode), or mention OpenHands with custom instructions
   - The PR must be to AND from an authorized repo

Documentation: https://docs.openhands.dev/openhands/usage/cloud/github-installation

**Approach 2: SDK GitHub Workflow** (requires an LLM API key)

Best for: Teams who want full control over a PR code review AI agent

1. Copy the workflow file to your repo:
   ```bash
   mkdir -p .github/workflows
   curl -o .github/workflows/pr-review-by-openhands.yml \
     https://raw.githubusercontent.com/OpenHands/software-agent-sdk/main/examples/03_github_workflows/02_pr_review/workflow.yml
   git add .github/workflows/pr-review-by-openhands.yml
   git commit -m "Add OpenHands PR review workflow"
   git push
   ```

2. Set up GitHub Secrets
   - Go to your repo → **Settings** → **Secrets and variables** → **Actions**
   - Add `LLM_API_KEY` with your OpenHands LLM API key (or alternative provider key)

3. Trigger PR reviews
   - **Option A:** Add a `review-this` label to any PR (create this label first)
   - **Option B (recommended):** Request `openhands-agent` as a reviewer on the PR

Documentation: https://github.com/OpenHands/software-agent-sdk/tree/main/examples/03_github_workflows/02_pr_review
