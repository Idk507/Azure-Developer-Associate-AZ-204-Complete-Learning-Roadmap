**End-to-End Introduction to Azure DevOps and GitHub Actions (with Python Example)**

I will explain **everything** from the absolute basics in very simple words, step by step, without skipping any concept. You will understand **why** these tools exist, **what** every part does, **how** they work together, and then we will do a **complete real-world setup** with a simple Python project. We will cover:

- What DevOps is
- Full Azure DevOps explanation (all 5 services)
- Full GitHub Actions explanation (every concept: workflows, jobs, steps, triggers, etc.)
- Side-by-side comparison
- A complete Python project (app + tests)
- Exact YAML code for both tools
- Line-by-line explanation of every YAML line
- Full setup guidance (account creation to running the pipeline)

Everything is explained as if you are completely new. No jargon without explanation.

### 1. What is DevOps? (Super Simple)
DevOps is **not** a tool — it is a **way of working**.  
It joins two teams that used to fight each other:
- **Developers** (who write code)
- **Operations** (who run the code on servers)

The goal: **Ship software faster, safer, and more often**.  
Instead of waiting weeks for a new feature to reach users, you can release many times a day automatically.

The magic tools that make this possible are called **CI/CD**:
- **CI (Continuous Integration)** = Every time someone pushes code, automatically **build** it and **test** it.
- **CD (Continuous Delivery/Deployment)** = Automatically **deploy** the tested code to servers (or create a ready-to-deploy package).

Azure DevOps and GitHub Actions are two popular platforms that do CI/CD (and much more).

### 2. Azure DevOps – Full Introduction
Azure DevOps is Microsoft’s **complete cloud platform** for the entire software lifecycle. It is like a Swiss army knife with 5 integrated services that work together perfectly.

**Core Services (explained simply):**
- **Azure Boards**: Planning tool. Think of it as digital sticky notes + Kanban boards + backlogs. You create “work items” (user stories, bugs, tasks). It shows burndown charts, velocity, sprints — perfect for Agile/Scrum teams.
- **Azure Repos**: Place to store your code. Supports **Git** (most popular) or old TFVC. Unlimited private repos, pull requests, code reviews, branch policies (e.g., “no one can merge without 2 approvals”).
- **Azure Pipelines**: The **CI/CD engine**. This is the part we will use most. It automatically builds, tests, and deploys your code.
- **Azure Test Plans**: Testing tool. Manual test cases, exploratory testing, screenshots, videos, and it links tests back to your work items.
- **Azure Artifacts**: Package storage. Like a private “PyPI” or “npm” inside your company. You publish Python packages, NuGet, Maven, etc., and reuse them safely.

**How they all connect (end-to-end flow)**:
1. Plan in Boards → 2. Code in Repos → 3. Build & test in Pipelines → 4. Test more in Test Plans → 5. Store packages in Artifacts → 6. Deploy via Pipelines → 7. See everything on Dashboards.

You can use **all 5** together or just **Pipelines** if you only need CI/CD.

**Azure Pipelines** (the CI/CD part we care about most):
- Uses **YAML files** (preferred) or classic editor.
- Runs on Microsoft-hosted agents (free tier available) or your own servers.
- Works with any language (Python, .NET, Java, etc.).
- Supports matrix builds (test on Python 3.10, 3.11, 3.12 at the same time).

### 3. GitHub Actions – Full Introduction
GitHub Actions is GitHub’s **built-in CI/CD platform**. It lives right inside your GitHub repository — no separate tool needed.

**Key Concepts (explained one by one):**

- **Workflow**: The entire automated process. It is a single `.yml` file inside `.github/workflows/`. One repo can have many workflows.
- **Trigger (`on:`)**: “When should this run?” Examples: every push to main, every pull request, every day at 8 AM, or manually by clicking a button.
- **Job**: A group of steps that run on the **same runner** (virtual machine). Jobs can run in parallel.
- **Step**: One single command or action. Steps run one after another inside a job.
- **Runner**: The computer that actually runs your code. GitHub provides free Linux, Windows, macOS machines (`ubuntu-latest`, etc.). You can also use your own self-hosted runners.
- **Action**: Reusable piece of code (like a Lego brick). You can write your own or use thousands from the GitHub Marketplace (e.g., `actions/setup-python`).
- **Matrix strategy**: Run the same job on many combinations (e.g., Python 3.10 + 3.11 + 3.12) at the same time.
- **Secrets & Variables**: Store passwords/API keys safely.
- **Artifacts**: Store build outputs (ZIP files, test reports) to download later.

Everything is defined in **YAML** — very readable once you understand the structure.

### 4. Quick Comparison (2025-2026 view)
- **Azure DevOps Pipelines**: Better if you already use Microsoft tools, need enterprise compliance, or want full Boards + Test Plans in one place. More GUI options.
- **GitHub Actions**: Simpler, faster to start, huge marketplace of actions, lives inside GitHub (perfect if your code is already on GitHub). Free minutes are generous for public repos.
Both are excellent. Many teams use **both** (e.g., GitHub for code + Azure Pipelines for enterprise deployment).

### 5. Complete Python Project Example (End-to-End)
We will build a tiny Python app + tests. Then we will set up CI/CD in **both** tools.

#### Project Files (create these on your computer)

**Folder structure:**
```
my-python-app/
├── app.py
├── tests/
│   └── test_app.py
├── requirements.txt
├── azure-pipelines.yml          ← for Azure DevOps
└── .github/
    └── workflows/
        └── ci.yml               ← for GitHub Actions
```

**app.py** (the actual code)
```python
def add(a: int, b: int) -> int:
    """Simple function to add two numbers"""
    return a + b

if __name__ == "__main__":
    print("Hello from Python CI/CD demo!")
    print("2 + 3 =", add(2, 3))
```

**tests/test_app.py** (the tests using pytest)
```python
import pytest
from app import add

def test_add_positive():
    assert add(2, 3) == 5

def test_add_negative():
    assert add(-1, 1) == 0

def test_add_zero():
    assert add(0, 0) == 0
```

**requirements.txt**
```
pytest
```

That’s it! Super simple but enough to demonstrate build + test.

### 6. Azure DevOps Pipeline (YAML) – Full Code + Line-by-Line

Create file `azure-pipelines.yml` in the root:

```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

strategy:
  matrix:
    Python310:
      python.version: '3.10'
    Python311:
      python.version: '3.11'
    Python312:
      python.version: '3.12'

steps:
  - task: UsePythonVersion@0
    inputs:
      versionSpec: '$(python.version)'
    displayName: 'Use Python $(python.version)'

  - script: |
      python -m pip install --upgrade pip
      pip install -r requirements.txt
    displayName: 'Install dependencies'

  - script: |
      pip install pytest pytest-azurepipelines
      pytest --junitxml=test-results.xml
    displayName: 'Run tests with pytest'

  - task: PublishTestResults@2
    inputs:
      testResultsFormat: 'JUnit'
      testResultsFiles: '**/test-*.xml'
    condition: succeededOrFailed()

  - task: ArchiveFiles@2
    displayName: 'Archive files'
    inputs:
      rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
      includeRootFolder: false
      archiveType: zip
      archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId)-$(python.version).zip'

  - task: PublishBuildArtifacts@1
    inputs:
      PathtoPublish: '$(Build.ArtifactStagingDirectory)'
      ArtifactName: 'drop'
      publishLocation: 'Container'
```

**Line-by-line (simple words):**
- `trigger: - main` → Run automatically when you push to the `main` branch.
- `pool: vmImage: ubuntu-latest` → Use a free Linux computer from Microsoft.
- `strategy: matrix` → Run the whole pipeline **3 times in parallel**, once for each Python version.
- `UsePythonVersion@0` → Tell the computer “use this exact Python version”.
- `script: install` → Upgrade pip and install everything from requirements.txt.
- `script: pytest` → Run all tests and create a nice test report.
- `PublishTestResults@2` → Show test results beautifully in Azure DevOps UI.
- `ArchiveFiles` + `PublishBuildArtifacts` → Zip the whole project and save it as a downloadable artifact.

### 7. GitHub Actions Workflow (YAML) – Full Code + Line-by-Line

Create folder `.github/workflows/` and file `ci.yml`:

```yaml
name: Python CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]

    steps:
      - uses: actions/checkout@v5

      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: 'pip'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Test with pytest
        run: |
          pip install pytest
          pytest --junitxml=test-results.xml

      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: test-results.xml
```

**Line-by-line (simple words):**
- `name:` → Name shown in GitHub Actions tab.
- `on: push & pull_request` → Run on push to main **and** on every pull request.
- `runs-on: ubuntu-latest` → Use GitHub’s free Linux machine.
- `strategy: matrix` → Same as Azure — test on 3 Python versions in parallel.
- `actions/checkout@v5` → Download your code into the runner.
- `actions/setup-python@v5` → Install exact Python version + auto-cache dependencies.
- `run: install` → Same pip commands.
- `run: pytest` → Run tests.
- `upload-artifact` → Save test report so you can download it later.

### 8. Complete Setup Guidance (Step-by-Step)

#### **Option A: GitHub Actions (Easiest)**
1. Create free GitHub account at github.com if you don’t have one.
2. Create new repository → name it `my-python-app`.
3. Clone it: `git clone https://github.com/yourusername/my-python-app.git`
4. Copy the 5 files above into it.
5. `git add . && git commit -m "Add Python CI/CD demo" && git push`
6. Go to your repo → click **Actions** tab → you will see the workflow running automatically!
7. Click the run → see jobs, logs, test results, artifacts.

#### **Option B: Azure DevOps (Full Platform)**
1. Go to dev.azure.com → sign in with Microsoft account → **New organization** (free).
2. Create a new project.
3. Two choices:
   - **Use Azure Repos**: In the project, go to Repos → Initialize repo → push your code (same git commands, but URL from Azure).
   - **Use GitHub** (recommended for this demo): Fork the sample or push to your GitHub repo first.
4. In Azure DevOps project → left menu **Pipelines** → **New pipeline**.
5. Choose **GitHub** (or Azure Repos) → select your repo.
6. Choose **Starter pipeline** → replace the YAML with the `azure-pipelines.yml` above → **Save and run**.
7. Watch it run! Check **Tests** and **Artifacts** tabs.

**Free tier notes**:
- GitHub Actions: 2,000 minutes/month free for public repos.
- Azure DevOps: Free for up to 5 users + generous pipeline minutes.

