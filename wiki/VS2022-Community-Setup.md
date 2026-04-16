# Visual Studio 2022 Community Setup

This page covers installing Visual Studio 2022 Community on a fresh Windows VM
with minimal bloat and Copilot suppressed, alongside VS Code.

Both editors can coexist. The scripts in this repo support both via the `-Editor` parameter
(`VSCode` or `VisualStudio`).

---

## Installation

Download the installer from [visualstudio.microsoft.com](https://visualstudio.microsoft.com/vs/community/).

Run the installer. When the **Workloads** screen appears, select only what you need.
The recommended configuration for this environment is a single workload:

### Workload: .NET desktop development

> See [images/VSCommunity.png](images/VSCommunity.png) for the exact workload and
> optional component selections.

**Included (cannot be deselected):**
- .NET desktop development tools
- .NET Framework 4.7.2 development tools
- C# and Visual Basic

**Optional — check these:**
- Development tools for .NET
- .NET Framework 4.8 development tools
- Just-In-Time debugger
- Blend for Visual Studio

**Optional — leave these unchecked:**
- Entity Framework 6 tools
- .NET profiling tools
- IntelliCode
- ML.NET Model Builder
- **GitHub Copilot** ← do not install
- **GitHub Copilot app modernization for .NET** ← do not install
- F# desktop language support
- PreEmptive Protection - Dotfuscator
- .NET Framework 4.6.2–4.7.1 development tools
- .NET Portable library targeting pack
- Windows Communication Foundation
- SQL Server Express 2019 LocalDB
- MSIX Packaging Tools
- JavaScript diagnostics
- Windows App SDK C# Templates
- Live Share
- .NET Framework 4.8.1 development tools

> **Why exclude Copilot at install time?**
> Installing the Copilot components adds background services, menu badges, and account
> prompts that appear throughout the IDE even if you never activate a subscription.
> It is significantly cleaner to exclude them at install time than to suppress them
> after the fact.

---

## Post-install configuration

Open Visual Studio, then go to **Tools → Options** for each section below.

---

### Environment → General

> See [images/environment-general.png](images/environment-general.png)

**Visual Experience settings:**

| Setting | Value |
|---|---|
| Color Theme | Dark |
| Separate font settings from color theme selection | ✓ checked |
| Use Windows High Contrast settings | ✓ checked |
| Apply title case styling to menu bar | ✓ checked |
| Use compact menu and search bar | ✓ checked |
| **Hide Copilot menu badge** | **✓ checked** |

The "Hide Copilot menu badge" option suppresses the Copilot toolbar icon and badge
that appear even when Copilot is not installed or activated.

---

### Environment → Extensions

> See [images/extensions.png](images/extensions.png)

| Setting | Value |
|---|---|
| Install updates automatically | ✓ checked |
| Allow synchronous autoload of extensions | unchecked (not recommended) |

Leave **Additional Extension Galleries** and **MCP Registries** empty unless you
have a specific need.

---

### Environment → Preview Features

> See [images/preview-features.png](images/preview-features.png)

Enable these preview features for better performance and usability:

| Feature | Value |
|---|---|
| Detect 32-bit assembly load failures for Windows Forms .NET Framework projects | ✓ |
| Enable Build Acceleration by default | ✓ |
| Enable Multi-Project Launch Profiles | ✓ |
| Extension Manager UI Refresh | ✓ |
| Load projects faster (some features may be delayed) | ✓ |
| Pull Request Comments | ✓ |
| Solution Load Cancellation | ✓ |
| Support for Multi-root Workspaces | ✓ |
| Use the preview Windows Forms out-of-process designer for .NET apps | ✓ |
| Use the project cache to speed up solution load | ✓ |

Leave these unchecked:
- Enable new .NET 9+ Mono debugger for WASM apps
- Enable new .NET Mono debugger for MAUI apps
- Initialize editor parts asynchronously during solution load
- Use previews of the .NET SDK

---

### Source Control → Plug-in Selection

> See [images/source-control.png](images/source-control.png)

| Setting | Value |
|---|---|
| Current source control plug-in | **Git** |

---

### Source Control → Git Global Settings

> See [images/global-git-settings.png](images/global-git-settings.png)

| Setting | Value |
|---|---|
| User name | RealWorldTec |
| Email | realworldtec@gmail.com |
| Default location | C:\Projects |
| Default branch name | main |
| Prune remote branches during fetch | True |
| Rebase local branch when pulling | False |
| Cryptographic network provider | Unset |
| Credential helper | Unset |
| Close open solutions not under Git when opening a repository | Yes |
| Automatically activate multiple repositories | Yes |
| Enable download of author images from 3rd party source | unchecked |
| Commit changes after merge by default | ✓ checked |
| Enable push --force-with-lease | unchecked |
| Open folder in Solution Explorer when opening a Git repository | ✓ checked |
| Automatically load the solution when opening a Git repository | ✓ checked |
| Automatically check out branches with double-click or Enter key | unchecked |
| Restore the Git Repository window on restart | unchecked |
| Diff Tool | None (use Visual Studio built-in) |
| Merge Tool | None (use Visual Studio built-in) |

> These settings mirror the git globals set at the command line during
> [Environment Setup](Environment-Setup.md). Visual Studio reads from the same
> global git config, so changes here update `~/.gitconfig` and vice versa.

---

## Developer PowerShell

> See [images/developer-powershell-redacted.png](images/developer-powershell-redacted.png)

Visual Studio includes a **Developer PowerShell** terminal (View → Terminal) that runs
inside the IDE with the VS build tools on the PATH.

SSH authentication works the same way as regular PowerShell — the `ssh-agent` service
must be running and the key must be loaded. See [SSH and ssh-agent Setup](SSH-and-ssh-agent-Setup.md).

Example session from inside the Developer PowerShell terminal:

```
git status
git remote -v
ssh -T git@github.com
git push
```

---

## Winget install reference

```powershell
# Install VS Code and Visual Studio Community side by side
winget install Microsoft.VisualStudioCode -e
winget install Microsoft.VisualStudio.2022.Community -e
```

When installing via winget, the workload selection above must be made in the VS Installer
UI that launches after the initial download completes. There is no supported winget flag
to pre-select workloads non-interactively for the Community edition.

---

## Copilot suppression checklist

If Copilot components were accidentally installed or are appearing despite the steps above:

- **Tools → Options → Environment → General:** check `Hide Copilot menu badge`
- **Extensions → Manage Extensions:** search for "GitHub Copilot" and uninstall if present
- **VS Installer → Modify:** deselect both Copilot optional components and repair the installation
- Reboot after any Copilot component removal
