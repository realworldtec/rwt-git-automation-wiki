# rwt-git-automation Wiki

PowerShell automation for creating and publishing GitHub repositories on Windows.

## Pages

- [Session Setup](Session-Setup) — load the tools, verify SSH, available commands
- [Usage Examples](Usage-Examples) — real commands organized by workflow
- [Environment Setup](Environment-Setup) — fresh Windows VM setup: tools, SSH key, git globals
- [Script Reference](Script-Usage-and-Examples) — full parameter reference for all commands
- [Configuration Reference](Configuration-Reference) — every key in `RWTProjectTools_config_v6.psd1`
- [Preset Reference](Preset-Reference) — built-in presets and how to use them
- [SSH and ssh-agent Setup](SSH-and-ssh-agent-Setup) — SSH key generation, config, and agent
- [Troubleshooting](Troubleshooting) — errors encountered and how they were resolved
- [VS2022 Community Setup](VS2022-Community-Setup) — installation, anti-bloat config, Copilot suppression

## Overview

These scripts wrap `git`, `gh`, and PowerShell to provide a repeatable workflow for
creating and publishing GitHub repositories from Windows. All defaults and presets
live in a single `.psd1` config file, keeping the scripts themselves generic.

The typical session looks like this:

```powershell
# 1. Load the tools — see Session Setup page
Set-ExecutionPolicy -Scope Process Bypass
. C:\Projects\NewGitProject\Register-RWTProjectTools_v6.ps1 `
    -ConfigPath C:\Projects\NewGitProject\RWTProjectTools_config_v6.psd1

# 2. Create a new project
New-RWTProject MyProject -PrivateWithWiki

# 3. Or publish an existing local repo
Publish-RWTExistingRepo -RepoPath C:\Projects\MyExistingRepo `
    -RemoteUrl git@github.com:owner/my-existing-repo.git `
    -CreateGitHubRepo
```
