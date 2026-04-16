# Usage Examples

Real commands used in production, organized by workflow.
All examples assume the tools are loaded for the session — see [Session Setup](Session-Setup.md).

---

## Session setup

Run once per PowerShell session before using any commands.

```powershell
Set-ExecutionPolicy -Scope Process Bypass

. C:\Projects\NewGitProject\Register-RWTProjectTools_v6.ps1 `
    -ConfigPath C:\Projects\NewGitProject\RWTProjectTools_config_v6.psd1
```

Verify the load:

```powershell
Get-RWTProjectToolConfig | Format-List
Get-Command Publish-RWTExistingRepo
```

---

## New project workflows

### Private repo with public wiki docs (standard RWT pattern)

```powershell
New-RWTProject MyProject -PrivateWithWiki
```

### PowerShell module

```powershell
New-RWTProject MyModule -PowerShellModule
```

### .NET Web API with controllers and xUnit tests

```powershell
New-RWTProject MyApi -DotNetService
```

### .NET class library with xUnit tests, pushed to GitHub

```powershell
New-RWTProject MyLibrary `
    -ProjectType DotNet `
    -DotNetTemplate classlib `
    -DotNetFramework net8.0 `
    -CreateDotNetTests `
    -DotNetTestTemplate xunit `
    -CreateGitHubRepo `
    -GitHubVisibility private `
    -PushInitialCommit `
    -LaunchEditor
```

### Node project in VS Code

```powershell
New-RWTProject MyNodeTool `
    -ProjectType Node `
    -Editor VSCode `
    -CreateStarterFiles `
    -CreateGitHubRepo `
    -PushInitialCommit `
    -LaunchEditor
```

### Local scratch repo — no remote, no commit, no push

```powershell
New-RWTProject Scratch -GenericScratch
```

### Local only — no remote operations

```powershell
New-RWTProject MyProject -LocalOnly
```

---

## Publishing an existing repo

### Step 1 — Validate locally (dry run, nothing pushed)

Run this first to confirm the repo passes structure and ignore hygiene checks.

```powershell
Publish-RWTExistingRepo `
    -RepoPath "C:\Projects\finance-audit-parsers" `
    -NoCommit `
    -NoTag `
    -NoPush `
    -CleanTemporaryWorkDirs `
    -Force
```

### Step 2 — Commit and tag locally, no push yet

```powershell
Publish-RWTExistingRepo `
    -RepoPath "C:\Projects\finance-audit-parsers" `
    -NoPush `
    -CleanTemporaryWorkDirs
```

### Step 3 — Create GitHub repo and push everything

```powershell
Publish-RWTExistingRepo `
    -RepoPath     "C:\Projects\finance-audit-parsers" `
    -RemoteUrl    "git@github.com:realworldtec/finance-audit-parsers.git" `
    -GitHubRepo   "finance-audit-parsers" `
    -GitHubOwner  "realworldtec" `
    -CreateGitHubRepo `
    -CleanTemporaryWorkDirs `
    -Force
```

### Re-run after the repo already exists — push tags only

```powershell
Publish-RWTExistingRepo `
    -RepoPath  "C:\Projects\finance-audit-parsers" `
    -RemoteUrl "git@github.com:realworldtec/finance-audit-parsers.git" `
    -NoCommit `
    -CleanTemporaryWorkDirs `
    -Force
```

---

## Publishing a scripts/tools repo with wiki and public docs

### Full one-shot: create private repo, push private wiki, create public docs repo

```powershell
Publish-RWTExistingRepo `
    -RepoPath             "C:\Projects\NewGitProject" `
    -RemoteUrl            "git@github.com:realworldtec/rwt-git-automation.git" `
    -GitHubRepo           "rwt-git-automation" `
    -GitHubOwner          "realworldtec" `
    -CreateGitHubRepo `
    -GitHubVisibility     private `
    -EnableWiki `
    -WikiFilesPath        "C:\Projects\NewGitProject-wiki" `
    -CreatePublicDocsRepo `
    -DocsFilesPath        "C:\Projects\NewGitProject-wiki" `
    -CommitMessage        "Initial release: rwt-git-automation v6" `
    -NoTag `
    -SkipStructureCheck `
    -Force
```

### Update wiki and public docs after content changes (repo already exists)

```powershell
Publish-RWTExistingRepo `
    -RepoPath             "C:\Projects\NewGitProject" `
    -RemoteUrl            "git@github.com:realworldtec/rwt-git-automation.git" `
    -GitHubRepo           "rwt-git-automation" `
    -GitHubOwner          "realworldtec" `
    -EnableWiki `
    -WikiFilesPath        "C:\Projects\NewGitProject-wiki" `
    -CreatePublicDocsRepo `
    -DocsFilesPath        "C:\Projects\NewGitProject-wiki" `
    -NoCommit `
    -NoTag `
    -SkipStructureCheck `
    -Force
```

---

## Post-baseline git workflow

After the initial baseline push, always work on a branch.

```powershell
Set-Location "C:\Projects\finance-audit-parsers"

# Create a working branch
git checkout -b post-baseline-hardening

# Check status before committing
git status
git diff --cached --name-only
```

---

## Checking repo state before a push

```powershell
Set-Location "C:\Projects\finance-audit-parsers"
git status
git remote -v
git log --oneline --decorate --max-count=5
git tag
```

---

## SSH and auth verification

Run these to confirm SSH and GitHub CLI auth are working before any push operation.

```powershell
# Confirm SSH key is loaded in agent
ssh-add -l

# Test GitHub SSH authentication
ssh -T git@github.com

# Confirm gh CLI authenticated account
gh auth status
gh api user --jq .login
```

If `ssh-add -l` returns "The agent has no identities", the key needs to be re-added:

```powershell
ssh-add C:\Users\YourName\.ssh\id_mygithubkey
```

---

## Recovering from common push errors

### Remote rejected — fetch first

Occurs when the remote has commits the local repo does not have
(e.g. after creating a Wiki page in the browser).

```powershell
Set-Location "C:\Projects\NewGitProject"
git pull
# Then re-run the publish command
```

### Repo created with wrong name

```powershell
# Add delete_repo scope if not already granted
gh auth refresh -h github.com -s delete_repo

# Delete the wrongly-named repo
gh repo delete realworldtec/WrongName --yes
```

### Wiki git repo does not exist yet

The GitHub wiki git repo is not created until the first page is created through
the browser. The script detects this and prints instructions:

1. Open `https://github.com/owner/repo/wiki`
2. Click **Create the first page**, type anything, click **Save Page**
3. Re-run the publish command — the wiki push completes automatically

---

## Config inspection and runtime updates

```powershell
# Inspect loaded config
Get-RWTProjectToolConfig | Format-List

# Change default owner at runtime without reloading
Set-RWTProjectToolConfig -DefaultGitHubOwner "newowner"

# Reload from a different config file
Set-RWTProjectToolConfig -ConfigPath "C:\Projects\NewGitProject\alt_config.psd1"
```

---

## Useful git one-liners

```powershell
# Show all local tags
git tag

# Show recent commits with decorations
git log --oneline --decorate --max-count=10

# Show what's staged
git diff --cached --name-only

# Show remotes
git remote -v

# Pull latest and prune deleted remote branches
git pull --prune

# Delete a local tag
git tag -d tag-name

# Push a specific tag
git push origin tag-name
```
