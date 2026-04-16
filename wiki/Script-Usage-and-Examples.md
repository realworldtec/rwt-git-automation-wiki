# Script Usage and Examples

---

## Loading the tools

Dot-source `Register-RWTProjectTools_v6.ps1` at the start of each session:

```powershell
Set-ExecutionPolicy -Scope Process Bypass
. C:\Projects\NewGitProject\Register-RWTProjectTools_v6.ps1 `
    -ConfigPath C:\Projects\NewGitProject\RWTProjectTools_config_v6.psd1
```

`-SuppressBanner` omits the confirmation output if you prefer a quieter load.

After loading, verify the config:

```powershell
Get-RWTProjectToolConfig | Format-List
Get-Command Publish-RWTExistingRepo
```

---

## New-RWTProject

Creates a new local project folder and optionally creates a GitHub repository.
Aliases: `nrwt`, `nrp`, `rwtproj`.

### Minimal — local only, no GitHub repo

```powershell
New-RWTProject MyProject
```

### Private repo with companion public wiki

```powershell
New-RWTProject MyProject -PrivateWithWiki
```

### PowerShell module project

```powershell
New-RWTProject MyModule -PowerShellModule
```

### .NET Web API with tests

```powershell
New-RWTProject MyApi -DotNetService
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

### .NET class library with xUnit tests

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

### Scratch repo — local only, no commit, no push

```powershell
New-RWTProject Scratch -GenericScratch
```

### Explicit preset by name

```powershell
New-RWTProject MyProject -PresetName RWT
```

---

## Publish-RWTExistingRepo

Publishes an already-existing local repository to GitHub.
Alias: `pubrwtrepo`.

This script can be called directly or through the wrapper loaded by Register.

### Dry run — validate locally, no push

```powershell
Publish-RWTExistingRepo `
    -RepoPath "C:\Projects\my-project" `
    -NoCommit -NoTag -NoPush `
    -CleanTemporaryWorkDirs `
    -Force
```

Use this first to confirm the repo passes structure validation and ignore hygiene
checks before touching anything remote.

### Commit and tag locally only

```powershell
Publish-RWTExistingRepo `
    -RepoPath "C:\Projects\my-project" `
    -NoPush `
    -CleanTemporaryWorkDirs
```

### Create GitHub repo and push in one step

```powershell
Publish-RWTExistingRepo `
    -RepoPath "C:\Projects\my-project" `
    -RemoteUrl "git@github.com:yourowner/my-project.git" `
    -CreateGitHubRepo `
    -GitHubOwner "yourowner" `
    -CleanTemporaryWorkDirs `
    -Force
```

`-CreateGitHubRepo` calls the GitHub REST API to create the remote repository
before pushing. If the repository already exists on GitHub the creation step is
skipped and the push proceeds.

### Push to an already-existing GitHub repo

```powershell
Publish-RWTExistingRepo `
    -RepoPath "C:\Projects\my-project" `
    -RemoteUrl "git@github.com:yourowner/my-project.git" `
    -NoCommit `
    -CleanTemporaryWorkDirs `
    -Force
```

### Push tags only (commit and repo already exist)

```powershell
Publish-RWTExistingRepo `
    -RepoPath "C:\Projects\my-project" `
    -RemoteUrl "git@github.com:yourowner/my-project.git" `
    -NoCommit `
    -CleanTemporaryWorkDirs `
    -Force
```

### Post-baseline: create a cleanup branch

After the initial baseline push, create a working branch before making changes:

```powershell
Set-Location "C:\Projects\my-project"
git checkout -b post-baseline-hardening
```

---

## Publish-RWTExistingRepo parameters

| Parameter | Type | Default | Purpose |
|---|---|---|---|
| `-RepoPath` | string | *(required)* | Path to the local repository |
| `-RemoteUrl` | string | — | SSH remote URL; required unless `-NoPush` |
| `-RemoteName` | string | `origin` | Git remote name |
| `-DefaultBranch` | string | `main` | Branch name |
| `-CommitMessage` | string | *(baseline message)* | Commit message for initial commit |
| `-BankTag` | string | `bank-v15-baseline` | First annotated tag name |
| `-AmexTag` | string | `amex-v24-baseline` | Second annotated tag name |
| `-SuiteTag` | string | `suite-baseline-{date}` | Third annotated tag; resolved from existing local tag if present |
| `-CreateGitHubRepo` | switch | — | Create the GitHub repo via REST API before pushing |
| `-GitHubOwner` | string | — | GitHub user or org; resolved from `gh auth` if omitted |
| `-GitHubRepo` | string | — | Repo name on GitHub; defaults to folder name of `-RepoPath` |
| `-GitHubVisibility` | string | `private` | `private` or `public` |
| `-CleanTemporaryWorkDirs` | switch | — | Remove `_work_*` folders before processing |
| `-NoCommit` | switch | — | Skip the commit step |
| `-NoTag` | switch | — | Skip tag creation |
| `-NoPush` | switch | — | Skip all remote operations |
| `-Force` | switch | — | Skip ignore hygiene check |

---

## New-RWTProject parameters (selected)

| Parameter | Type | Purpose |
|---|---|---|
| `-ProjectName` | string *(required)* | Project folder name |
| `-RootPath` | string | Parent folder; defaults to `DefaultRootPath` from config |
| `-GitHubOwner` | string | GitHub user/org; defaults to `DefaultGitHubOwner` from config |
| `-PresetName` | string | Named preset to merge (e.g. `RWT`, `FinanceAuditBaseline`) |
| `-Editor` | string | `Auto`, `VSCode`, `VisualStudio` |
| `-ProjectType` | string | `Generic`, `PowerShell`, `DotNet`, `Node` |
| `-GitHubVisibility` | string | `private`, `public` |
| `-CreateGitHubRepo` | switch | Create remote repo via `gh` |
| `-PushInitialCommit` | switch | Push after creating repo |
| `-LaunchEditor` | switch | Open in editor after creation |
| `-LocalOnly` | switch | Preset shortcut: no remote, no open |
| `-PrivateWithWiki` | switch | Preset shortcut: private repo + public wiki |
| `-DotNetService` | switch | Preset shortcut: .NET Web API with tests |
| `-PowerShellModule` | switch | Preset shortcut: PowerShell module layout |
| `-GenericScratch` | switch | Preset shortcut: local-only scratch repo |
| `-FinanceAuditBaseline` | switch | Preset shortcut: finance audit parser baseline |
