# Environment Setup

This page covers setting up a fresh Windows VM or machine for use with these scripts.
All commands are run in an elevated PowerShell session unless noted otherwise.

---

## 1. Install required tools

```powershell
winget install Microsoft.VisualStudioCode -e
winget install --id Git.Git -e
winget install --id GitHub.cli -e
```

### Optional: GitHub Desktop

```powershell
winget install --id GitHub.Desktop -e
```

### Optional: Visual Studio Community 2022

Install from [visualstudio.microsoft.com](https://visualstudio.microsoft.com/vs/community/).

Recommended workload: **.NET desktop development**

Recommended optional components within that workload:
- Development tools for .NET
- .NET Framework 4.8 development tools
- Just-In-Time debugger
- Blend for Visual Studio

VS Code and Visual Studio can coexist. The scripts support both via the `-Editor` parameter.

---

## 2. Create the projects folder

```powershell
mkdir C:\Projects
```

### Optional: apply ACL hardening with Fix-ProjectAcl.ps1

`Fix-ProjectAcl.ps1` sets an explicit Full Control ACL on a folder for the current
user and optionally breaks inherited permissions from the parent. This is useful on
managed or domain-joined machines where inherited permissions from `C:\` may be
overly permissive or may include accounts you do not want to have access to your
project files.

Copy `Fix-ProjectAcl.ps1` to a scripts folder (e.g. `C:\Scripts\`) and run:

```powershell
.\Fix-ProjectAcl.ps1 C:\Projects -DisableInheritance
```

What it does:

- Resolves the full path and reads the current ACL
- If `-DisableInheritance` is specified: converts all inherited rules to explicit rules
  first (so no access is silently lost), then breaks the inheritance link
- Adds an explicit Full Control rule for the current user (`$env:USERNAME` by default)
  covering the folder and all descendants
- Reinforces the grant recursively using `icacls /T /C` to catch any child items

To grant to a specific user rather than the current one:

```powershell
.\Fix-ProjectAcl.ps1 C:\Projects -UserName "DOMAIN\YourAccount" -DisableInheritance
```

On a standalone VM where you are the only user this step is optional.

---

## 3. Configure git globals

```powershell
git config --global user.name "YourName"
git config --global user.email "your@email.com"
git config --global init.defaultBranch main
git config --global fetch.prune true
git config --global pull.rebase false
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```

Verify:

```powershell
git config --global --list
```

> **Note on `core.sshCommand`:** This sets git to use Windows' built-in OpenSSH rather than
> the SSH bundled with Git for Windows. This is required for VSCode's git integration to work
> correctly with a passphrase-protected SSH key loaded into `ssh-agent`. See
> [SSH and ssh-agent Setup](SSH-and-ssh-agent-Setup) for details.

---

## 4. Set up SSH key and agent

See the dedicated [SSH and ssh-agent Setup](SSH-and-ssh-agent-Setup) page for the full process.

Summary:

```powershell
# Enable and start the agent service permanently
Set-Service -Name ssh-agent -StartupType Automatic
Start-Service ssh-agent

# Generate key
ssh-keygen -t ed25519 -C "your@email.com" -f "$HOME\.ssh\id_mygithubkey"

# Add to agent (prompted for passphrase once)
ssh-add "$HOME\.ssh\id_mygithubkey"

# Test
ssh -T git@github.com
```

---

## 5. Authenticate the GitHub CLI

```powershell
gh auth login
```

Choose: **GitHub.com → SSH → your key → Login with a web browser**

Verify:

```powershell
gh auth status
gh api user --jq .login
```

---

## 6. Place the scripts

Copy the four files to `C:\Projects\NewGitProject\`:

```
C:\Projects\NewGitProject\
    New-GitHubProject_v6.ps1
    Publish-RWTExistingRepo_v6.ps1
    Register-RWTProjectTools_v6.ps1
    RWTProjectTools_config_v6.psd1
```

Edit `RWTProjectTools_config_v6.psd1` and set `DefaultGitHubOwner` to your GitHub username or org.

---

## 7. Load the tools

```powershell
Set-ExecutionPolicy -Scope Process Bypass
. C:\Projects\NewGitProject\Register-RWTProjectTools_v6.ps1 `
    -ConfigPath C:\Projects\NewGitProject\RWTProjectTools_config_v6.psd1
```

Expected output:

```
RWT project tools loaded.
Config file              : C:\Projects\NewGitProject\RWTProjectTools_config_v6.psd1
Bootstrap script         : C:\Projects\NewGitProject\New-GitHubProject_v6.ps1
Existing repo publisher  : C:\Projects\NewGitProject\Publish-RWTExistingRepo_v6.ps1
Default root             : C:\Projects
Default owner            : yourowner
Commands                 : New-RWTProject, nrwt, nrp, rwtproj, Publish-RWTExistingRepo, pubrwtrepo
```

---

## VM provisioning script reference

The following is a minimal provisioning script suitable for automating a fresh VM:

```powershell
# Tools
winget install Microsoft.VisualStudioCode -e
winget install --id Git.Git -e
winget install --id GitHub.cli -e

# Folder
mkdir C:\Projects
# Optional ACL hardening — copy Fix-ProjectAcl.ps1 to C:\Scripts\ first
# .\Fix-ProjectAcl.ps1 C:\Projects -DisableInheritance

# SSH agent — must be done before any git/gh operations
Set-Service -Name ssh-agent -StartupType Automatic
Start-Service ssh-agent
# Note: ssh-add requires interactive passphrase entry — run manually after first login

# Git globals
git config --global user.name "YourName"
git config --global user.email "your@email.com"
git config --global init.defaultBranch main
git config --global fetch.prune true
git config --global pull.rebase false
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"

# Authenticate gh (interactive)
gh auth login
```
