Here’s the **next level** of your Python CI/CD pipeline: **adding deployment**.

I will cover **two practical options** you asked for:

1. **Publish to PyPI** (perfect for libraries/packages — your `add()` function becomes a reusable Python package that anyone can `pip install`).
2. **Deploy to Azure App Service** (perfect for web applications — we’ll turn your simple app into a tiny Flask web app so it can actually run in the cloud with a public URL).

I keep everything in **simple words**, with **full updated code**, **line-by-line explanations**, and **exact setup steps**.  
We build on the previous project — you only need to add a few files and make small changes.

### Option 1: Publish to PyPI (Library Deployment)
This makes your code installable via `pip install my-python-app`.

#### Step-by-step changes to your project

1. **Add these files** to make it a proper Python package:

**setup.py** (in root)
```python
from setuptools import setup, find_packages

setup(
    name="my-python-app",
    version="0.1.0",                    # Change this when you release new version
    packages=find_packages(),
    install_requires=["pytest"],        # Only for tests, not runtime
    description="Simple demo package with CI/CD",
    author="Your Name",
    python_requires=">=3.10",
)
```

**pyproject.toml** (recommended modern way — place in root)
```toml
[build-system]
requires = ["setuptools>=45", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "my-python-app"
version = "0.1.0"
description = "Simple demo package"
readme = "README.md"
requires-python = ">=3.10"
dependencies = []
```

(You can use either setup.py or pyproject.toml — both work. I’ll show both in pipelines.)

2. **Update folder structure** (move app.py to package):
```
my-python-app/
├── my_python_app/          ← new folder
│   ├── __init__.py
│   └── app.py              ← move your add function here
├── tests/
│   └── test_app.py
├── requirements.txt
├── setup.py
├── pyproject.toml
├── azure-pipelines.yml
└── .github/workflows/
    └── ci-cd.yml           ← we'll rename and expand the previous ci.yml
```

**my_python_app/__init__.py**
```python
from .app import add
```

**my_python_app/app.py** (same as before, just moved)
```python
def add(a: int, b: int) -> int:
    return a + b
```

Update **tests/test_app.py** import:
```python
from my_python_app.app import add
```

#### GitHub Actions – Full CI + Publish to PyPI (Recommended)

Replace `.github/workflows/ci.yml` with this new file (name it `ci-cd.yml`):

```yaml
name: Python CI + Publish to PyPI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  release:                     # Trigger publish only on new GitHub Release
    types: [created]

jobs:
  test:
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
          pip install -e .          # Install your package in editable mode

      - name: Test with pytest
        run: pytest

  build-and-publish:
    needs: test                  # Only run if tests pass
    if: github.event_name == 'release'   # Only on release
    runs-on: ubuntu-latest
    environment:
      name: pypi
      url: https://pypi.org/p/my-python-app   # Change to your package name

    permissions:
      id-token: write            # Required for Trusted Publishing (no secrets needed!)

    steps:
      - uses: actions/checkout@v5

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install build tools
        run: |
          python -m pip install --upgrade pip build twine

      - name: Build package
        run: python -m build

      - name: Publish to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
        # No username/password needed with Trusted Publishing!
```

**How to finish PyPI setup (one-time):**
- Go to https://pypi.org → Account → “Add a new project” or use **Trusted Publishers**.
- In PyPI project settings → “Manage publishing” → Add GitHub as trusted publisher (owner/repo + workflow name).
- Create a GitHub Release (with tag v0.1.0) → the pipeline will automatically publish to PyPI.

After this, anyone can run: `pip install my-python-app`

#### Azure DevOps – Add Publish Stage (optional)
You can add a deployment stage to `azure-pipelines.yml` using the same `python -m build` + `twine upload`, but you would need to store PyPI credentials as secrets (less secure than Trusted Publishing). GitHub Actions is cleaner for PyPI.

### Option 2: Deploy to Azure App Service (Web App Deployment)

We turn your code into a tiny **Flask web app** so it can run live on the internet.

#### Make it a Web App

Add these files:

**requirements.txt** (update)
```
flask
gunicorn
pytest
```

**app.py** (replace the old one — put it in root for simplicity)
```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "<h1>Hello from Azure App Service! Your add function: 2 + 3 = 5</h1>"

@app.route("/add/<int:a>/<int:b>")
def add_route(a, b):
    return f"<h1>{a} + {b} = {a + b}</h1>"

if __name__ == "__main__":
    app.run()
```

Create **Procfile** (for Gunicorn — tells Azure how to run it)
```
web: gunicorn app:app
```

Create **.python-version** (optional but helpful)
```
3.12
```

#### Azure App Service Setup (One-time, do this first)

1. Go to https://portal.azure.com → Search “App Services” → Create.
2. Choose:
   - Subscription: your free trial or pay-as-you-go
   - Resource Group: create new
   - Name: something unique (e.g., mypythonapp2026)
   - Publish: **Code**
   - Runtime stack: **Python 3.12**
   - Operating System: **Linux**
   - Region: closest to you
   - Pricing plan: **Free F1** (or B1 for more power)
3. Create → Wait 1-2 minutes.

**Get deployment credentials** (for both tools):
- In App Service → Deployment Center → **GitHub Actions** or **FTPS credentials** → Download publish profile (XML file).
- Copy the entire XML content.

#### GitHub Actions – Deploy to Azure App Service

Add this to your workflow (or create a new one called `deploy.yml`). Here is the **full combined CI + CD**:

```yaml
name: Build, Test & Deploy to Azure App Service

on:
  push:
    branches: [ main ]

env:
  AZURE_WEBAPP_NAME: mypythonapp2026          # ← Change to your app name

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v5

      - name: Set up Python 3.12
        uses: actions/setup-python@v5
        with:
          python-version: 3.12
          cache: 'pip'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run tests
        run: pytest

      - name: Zip artifact for deployment
        run: zip -r release.zip . -x "*.git*"

      - name: Deploy to Azure Web App
        uses: azure/webapps-deploy@v3
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: release.zip
```

**Setup secrets in GitHub:**
- Go to your repo → Settings → Secrets and variables → Actions → New repository secret.
- Name: `AZURE_WEBAPP_PUBLISH_PROFILE`
- Paste the entire publish profile XML.

Push → The app deploys automatically. Visit `https://yourappname.azurewebsites.net/`  
You will see the hello message and can test `/add/10/20`.

#### Azure DevOps – Full Pipeline with Deploy Stage

Update `azure-pipelines.yml` to have **stages** (Build → Deploy):

```yaml
trigger:
- main

variables:
  azureServiceConnectionId: 'Your-Azure-Service-Connection'  # Create this in Azure DevOps
  webAppName: 'mypythonapp2026'                              # Your app name

stages:
- stage: Build
  jobs:
  - job: BuildJob
    pool:
      vmImage: ubuntu-latest
    steps:
      - task: UsePythonVersion@0
        inputs:
          versionSpec: '3.12'

      - script: |
          pip install -r requirements.txt
          pip install pytest
        displayName: 'Install deps'

      - script: pytest
        displayName: 'Run tests'

      - task: ArchiveFiles@2
        inputs:
          rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
          includeRootFolder: false
          archiveType: zip
          archiveFile: '$(Build.ArtifactStagingDirectory)/app.zip'

      - task: PublishBuildArtifacts@1
        inputs:
          PathtoPublish: '$(Build.ArtifactStagingDirectory)'
          ArtifactName: 'drop'

- stage: Deploy
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DeployWebApp
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
            - task: AzureWebApp@1
              inputs:
                azureSubscription: '$(azureServiceConnectionId)'
                appType: 'webAppLinux'
                appName: '$(webAppName)'
                packageForLinux: '$(Pipeline.Workspace)/drop/app.zip'
                runtimeStack: 'PYTHON|3.12'
```

**Setup in Azure DevOps:**
- Project Settings → Service connections → New → Azure Resource Manager → Service principal (automatic) → Link to your subscription.
- Use the connection name in the YAML.

### Which one should you use?
- **Just a library** (`add` function) → Use **PyPI publish** (GitHub Actions is easiest).
- **Web app / API** → Use **Azure App Service** deployment.
- You can do **both** in the same repo (different jobs).

### Next Steps You Can Add Later
- Add environment variables in App Service (e.g., database connection).
- Set `SCM_DO_BUILD_DURING_DEPLOYMENT=true` in App Service Configuration → Application settings (helps with Python builds).
- Use containers (Docker) for more complex apps.
- Add manual approval gates before production deploy.

