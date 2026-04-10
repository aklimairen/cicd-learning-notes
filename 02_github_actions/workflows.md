
# GitHub Actions

## Overview

GitHub Actions is a CI/CD tool available in every repository. It is integrated into GitHub as a service. GitHub Actions allows you to treat your CI pipeline as code, just like other CI/CD tools. You need a YAML file to store the workflow definitions in a folder called `.github/workflows`.

Similar to Jenkins plugins, GitHub Actions has a **Marketplace** that hosts reusable actions you can use in your workflows.

---

## Core Concepts

Each workflow has the following components in order:

**Event → Runner → Jobs → Steps → Actions**

### Event

Tells the workflow *when* it should run. Events can be:

- Push to a repository
- Create a pull request
- Create a release
- A scheduled time (cron)
- Manual trigger (`workflow_dispatch`)

### Runner

Executes the jobs in the workflow. Two types:

- **Built-in runners** — GitHub-hosted virtual environments (Ubuntu, Windows, macOS)
- **Self-hosted runners** — custom environments you manage yourself

### Jobs

- A workflow can contain one or more jobs (e.g. build, test, publish, deploy)
- By default, jobs run **in parallel**
- Use `needs:` to make a job wait for another to finish (sequential execution)

### Steps

- Each job consists of a series of steps
- Steps run **sequentially** inside a job
- A step can either:
  - Run a shell command → `run:`
  - Use a pre-built action → `uses:`

### Actions

- Reusable units of code that perform a specific task inside a step
- Can be used from the **GitHub Marketplace** or written yourself
- Example: `actions/checkout@v3` checks out your repository code

---

## Basic Workflow YAML Structure

```yaml
name: CI Pipeline

on: [push, pull_request]       # Event

jobs:
  build:                        # Job
    runs-on: ubuntu-latest      # Runner
    steps:
      - uses: actions/checkout@v3          # Action
      - name: Run tests                    # Step
        run: npm test
```

---

## Other Important Points

- **Secrets & Environment Variables:** Sensitive values like API keys are stored in *Settings → Secrets* and accessed in workflows via `${{ secrets.MY_SECRET }}`
- **Artifacts:** Upload and download build artifacts between jobs using `actions/upload-artifact` and `actions/download-artifact`
- **Caching:** Use `actions/cache` to cache dependencies (e.g. `node_modules`) and speed up builds
- **Matrix Strategy:** Run the same job across multiple configurations (e.g. different Node.js versions or OS) in parallel using `strategy: matrix:`
- **Manual Triggers:** Use `workflow_dispatch` to trigger a workflow manually from the GitHub UI
- **Status Badges:** Embed a workflow status badge in your README to show if builds are passing

---

## Resources

- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [GitHub Marketplace](https://github.com/marketplace?type=actions)
