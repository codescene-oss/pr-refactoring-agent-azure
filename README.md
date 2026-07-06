# CodeScene PR Refactoring Agent for Azure DevOps

An Azure Pipelines template that enables CodeScene's PR Refactoring Agent so reviewers can trigger Code Health-guided refactoring directly from pull requests.

It keeps refactoring inside the normal review flow while giving teams a consistent way to improve maintainability and prevent regressions.

## Features

- **Automatic code health analysis** — Identifies technical debt and code smells
- **AI-guided refactoring** — Uses state-of-the-art LLMs to suggest and apply improvements
- **CodeScene integration** — Leverages CodeScene's battle-tested code health metrics
- **PR-driven workflow** — Trigger refactorings directly from pull request pipelines
- **Skill-based execution** — Pre-built refactoring skills for common scenarios

## Quick Start

1. Configure pipeline variables for CodeScene and at least one supported AI provider.
2. Add the template reference and job definitions to your `azure-pipelines.yml`.
3. Go to **Pipelines → your pipeline → Run pipeline**, enter the PR ID and target branch, and click Run.

## Required: Configure pipeline variables

In your Azure DevOps project, go to **Pipelines → Library** and create a variable group (or configure variables directly on the pipeline) with:

- `CS_ACCESS_TOKEN` — Get it using the option that matches your setup:
  - CodeScene Cloud PAT: create it at [Create a Personal Access Token](https://codescene.io/users/me/pat).
  - CodeScene on-prem PAT: log in to your CodeScene instance, open `Configuration`, go to `Authentication`, then create a token under `Personal Access Tokens`.
- `AZURE_TOKEN` — A Personal Access Token with **Code (Read & Write)** and **Pull Request Threads (Read & Write)** scopes. Go to **User Settings → Personal Access Tokens** to create one.

**At least one AI provider:**
- `ANTHROPIC_API_KEY` — Get from [Anthropic](https://console.anthropic.com)
- `OPENAI_API_KEY` — Get from [OpenAI](https://platform.openai.com)
- `GOOGLE_API_KEY` — Get from [Google AI Studio](https://aistudio.google.com)
- `OPENCODE_AUTH_JSON` — OpenCode auth JSON

## Required: Add the template to your pipeline

First, create a GitHub service connection in **Project Settings → Service connections**, then add this to your `azure-pipelines.yml`:

```yaml
resources:
  repositories:
    - repository: cs-agent
      type: github
      name: codescene-oss/pr-refactoring-agent-azure
      ref: refs/tags/v1.0.0
      endpoint: <your-github-service-connection>

parameters:
  - name: pr_id
    type: string
    displayName: "Pull Request ID"
    default: ""
  - name: target_branch
    type: string
    displayName: "Target branch (e.g. main)"
    default: "main"
  - name: print_logs
    type: boolean
    displayName: "Print agent logs"
    default: false

trigger: none
pr: none

variables:
  - group: <your-variable-group>

pool:
  vmImage: ubuntu-latest

jobs:
  - job: fix_code_health_degradations
    displayName: "CodeScene — Fix Code Health Degradations"
    steps:
      - checkout: self
        persistCredentials: true
      - template: refactoring-agent.yml@cs-agent
        parameters:
          command: "skill:fix-code-health-degradations"
          model: "anthropic/claude-sonnet-4-20250514"
          pr_id: ${{ parameters.pr_id }}
          target_branch: ${{ parameters.target_branch }}
          print_logs: ${{ parameters.print_logs }}
```

Then run the job from the pipeline UI on any pull request.

The template automatically:
- Downloads the refactoring agent binary
- Configures git
- Runs the refactoring
- Pushes changes to the PR branch
- Posts a comment thread with the result

## The quality of the agent depends on the model

The refactoring quality depends heavily on the strength of the backing LLM, so use one of the strongest available models from a supported provider.

The refactoring agent supports models from Anthropic, OpenAI, and Google.

## Available Skills

The agent includes two pre-built refactoring skills:

- `skill:fix-code-health-degradations` — Fix only the Code Health regressions introduced by the PR, without touching pre-existing debt
- `skill:uplift-code-health` — Raise Code Health for selected files toward a target score, in measurable incremental steps

## Variables

| Variable | Description | Required |
|---|---|---|
| `CS_ACCESS_TOKEN` | CodeScene API access token | Yes |
| `AZURE_TOKEN` | Azure DevOps PAT | Yes |
| `ANTHROPIC_API_KEY` | Anthropic API key | No |
| `OPENAI_API_KEY` | OpenAI API key | No |
| `GOOGLE_API_KEY` | Google API key | No |
| `OPENCODE_AUTH_JSON` | OpenCode auth JSON | No |
| `CS_ONPREM_URL` | CodeScene on-prem instance URL | No |
| `REFACTORING_AGENT_VERSION` | Version of the agent to use | No |

Template parameters (`command`, `model`, `version`) are set per job in your pipeline YAML.

## Platform Support

The template supports Linux runners (amd64, aarch64). Use `vmImage: ubuntu-latest` or a self-hosted Linux agent.

## How It Works

1. **Download** — The template downloads the appropriate pre-built binary for your platform
2. **Analyze** — CodeScene analyzes your code for health issues and technical debt
3. **Refactor** — The AI model generates and applies improvements based on CodeScene's guidance
4. **Commit** — Changes are automatically committed and pushed to your branch

## License

Copyright CodeScene AB. See [LICENSE](LICENSE) for terms.

## Support

- [GitHub Issues](https://github.com/codescene-oss/pr-refactoring-agent-azure/issues)
- [Documentation](https://codescene.com/docs)
