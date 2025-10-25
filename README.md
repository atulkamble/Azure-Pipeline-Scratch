# Azure DevOps Pipelines from Scratch — Quickstart Docs

## Prerequisites

* GitHub account (with the repo you’ll use)
* Azure DevOps account: [https://dev.azure.com/](https://dev.azure.com/)
* VS Code with:

  * GitHub Pull Requests and Issues
  * GitHub Copilot (optional)
  * GitHub Copilot Chat (optional)
  * Azure Pipelines (optional)
* Git installed and configured: `git --version`

---

## Part A — First Pipeline (import existing repo)

### 1) Fork the sample repo

* Open: `https://github.com/atulkamble/pythonhelloworld`
* Click **Fork** → choose your account.

### 2) Create an Azure DevOps Organization

* Go to **[https://dev.azure.com/](https://dev.azure.com/)**
* Sign in → create organization (e.g., `cloudnautic-org`)

### 3) Create a Project

* **New project** → Name (e.g., `pythonhelloworld-azdo`)
* Visibility: **Public** (for easy demos) or **Private** (*recommended for real work*)

### 4) Import Repo into Azure Repos (optional path A)

If you want the code in Azure Repos:

* **Repos** → **Import a repository**
* Paste the Git URL of your **fork** (from GitHub)
  e.g., `https://github.com/<your-username>/pythonhelloworld.git`
* Click **Import**

**OR (optional path B) use GitHub directly:**
You can keep code in GitHub and connect during pipeline setup.

### 5) Set up the Build (Azure Pipeline)

* Go to **Pipelines** → **Create Pipeline**
* Choose the source:

  * **Azure Repos Git** (if you imported)
  * **GitHub** (authorize and select your fork)
* When prompted, select **Python** template or **Existing Azure Pipelines YAML**.
* Use this minimal `azure-pipelines.yml` (Python sample):

```yaml
# azure-pipelines.yml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: UsePythonVersion@0
    inputs:
      versionSpec: '3.x'
      addToPath: true

  - script: |
      python --version
      pip install --upgrade pip
      if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
    displayName: 'Setup Python & install deps'

  - script: |
      python app.py || true
    displayName: 'Run app (sample)'

  - script: |
      pytest -q || true
    displayName: 'Run tests (if present)'

  - task: PublishBuildArtifacts@1
    inputs:
      PathtoPublish: '.'
      ArtifactName: 'drop'
      publishLocation: 'Container'
```

> Notes
>
> * Adjust the run/test commands to match repo structure (`app.py`, `tests/`, etc.).
> * Remove `|| true` if you want the build to **fail** on errors (recommended for real projects).

### 6) Run the Pipeline

* Commit the YAML when prompted (Azure DevOps will propose a PR/commit).
* **Run** the pipeline.

### 7) Verify Success

* In **Pipelines**, open the run → ensure a **green tick** ✅
* Check **Artifacts** → `drop` exists.

---

## Part B — Pipeline From Scratch

### 1) Create a GitHub Repo

* On GitHub: **New repository** → name (e.g., `py-azdo-demo`) → add README (optional).

### 2) Add Azure Pipeline YAML to the Repo

Create `azure-pipelines.yml` at the repo root:

```yaml
trigger:
  - main

pr:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  PYTHON_VERSION: '3.11'

steps:
  - task: UsePythonVersion@0
    inputs:
      versionSpec: '$(PYTHON_VERSION)'

  - script: |
      python -V
      pip install --upgrade pip
      if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
    displayName: 'Install dependencies'

  - script: |
      echo "Running lint (flake8 if present)"
      if command -v flake8 >/dev/null 2>&1; then flake8 .; else echo "flake8 not installed"; fi
    displayName: 'Lint (optional)'

  - script: |
      echo "Running tests (pytest if present)"
      if command -v pytest >/dev/null 2>&1; then pytest -q; else echo "pytest not installed"; fi
    displayName: 'Tests (optional)'

  - task: PublishBuildArtifacts@1
    inputs:
      PathtoPublish: '.'
      ArtifactName: 'drop'
      publishLocation: 'Container'
```

### 3) Create Azure DevOps Project

* In **[https://dev.azure.com/](https://dev.azure.com/)** → **New project** → public/private

### 4) Connect Repo

* **Pipelines** → **Create Pipeline**
* Choose **GitHub** → authorize → select your repo
* Pick **Existing Azure Pipelines YAML** (root) → **Run**

> Alternative: If you prefer Azure Repos, use **Repos → Import** and then create pipeline against Azure Repos Git.

### 5) VS Code Environment Setup (Recommended)

* Install extensions:

  * **GitHub Pull Requests and Issues**
  * **GitHub Copilot** (optional)
  * **GitHub Copilot Chat** (optional)
  * **Azure Pipelines** (optional)
* `git config --global user.name "Your Name"`
* `git config --global user.email "you@example.com"`

### 6) Clone Your GitHub Repo and Open in VS Code

```bash
git clone https://github.com/<your-username>/py-azdo-demo.git
cd py-azdo-demo
code .
```

### 7) Make Changes → Test → Push

* Edit code, add `requirements.txt` / tests as needed
* Local quick check:

  ```bash
  python -V
  pip install -r requirements.txt   # if present
  pytest -q                         # if tests exist
  ```
* Commit & push:

  ```bash
  git add .
  git commit -m "feat: initial azure pipeline"
  git push origin main
  ```
* Verify pipeline runs automatically (trigger on `main`).

---

## Git Branch Basics

### Create a Branch

```bash
git branch feature/readme-update
git checkout feature/readme-update
# or in one command:
git checkout -b feature/readme-update
```

### Switch Branches

```bash
git checkout main
git checkout feature/readme-update
```

### Merge a Branch into Main

```bash
# ensure your main is up-to-date
git checkout main
git pull origin main

# merge the feature branch
git merge feature/readme-update

# resolve conflicts if any, then:
git push origin main
```

> Tip: Use Pull Requests in GitHub/Azure Repos to review before merging:
>
> * Push your branch: `git push -u origin feature/readme-update`
> * Open PR in GitHub/Azure Repos → reviewers → complete.

---

## Optional Enhancements

### Add a Status Badge to README

* In **Pipelines** → select your pipeline → **…** → **Status badge**
* Copy Markdown and paste in your `README.md`.

### Secure GitHub Access (if using private repos)

* In Azure DevOps: **Project settings → Service connections → GitHub**
* Create a connection (OAuth or PAT) and use it in the pipeline if required.

### Cache Python Dependencies (speed up builds)

```yaml
- task: Cache@2
  inputs:
    key: 'pip | "$(Agent.OS)" | requirements.txt'
    restoreKeys: |
      pip | "$(Agent.OS)"
    path: $(PIP_CACHE_DIR)
  displayName: 'Cache pip'

- script: |
    echo "PIP_CACHE_DIR: ${PIP_CACHE_DIR:-$HOME/.cache/pip}"
    pip install --upgrade pip
    if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
  displayName: 'Install deps (cached)'
```

### Multi-Stage (Build → Test → Package)

```yaml
stages:
- stage: Build
  jobs:
  - job: BuildJob
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - script: echo "Build steps here"

- stage: Test
  dependsOn: Build
  jobs:
  - job: TestJob
    steps:
    - script: echo "Run tests here"

- stage: Publish
  dependsOn: Test
  jobs:
  - job: PublishJob
    steps:
    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '.'
        ArtifactName: 'drop'
```

---

## Troubleshooting

* **Pipeline can’t see repo (GitHub)**

  * Re-run authorization; check **Project Settings → Service connections**.
* **YAML path not found**

  * Ensure `azure-pipelines.yml` is at repo root or update the path during pipeline creation.
* **Build fails on tests**

  * Remove `|| true` from sample if you want strict failures; add proper test requirements (`pytest`) in `requirements.txt`.
* **Private project pull failure**

  * Ensure correct PAT/permissions; for Azure Repos verify you’re using the correct remote URL and credentials.

---
