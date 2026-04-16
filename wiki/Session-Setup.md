# Session Setup

The tools must be loaded once per PowerShell session before any commands are available.

---

## Load command

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
Default owner            : realworldtec
Commands                 : New-RWTProject, nrwt, nrp, rwtproj, Publish-RWTExistingRepo, pubrwtrepo
```

---

## Available commands after loading

| Command | Alias | Purpose |
|---|---|---|
| `New-RWTProject` | `nrwt`, `nrp`, `rwtproj` | Create a new project |
| `Publish-RWTExistingRepo` | `pubrwtrepo` | Publish an existing local repo |
| `Get-RWTProjectToolConfig` | — | Show the loaded configuration |
| `Set-RWTProjectToolConfig` | — | Update config values at runtime |

Tab completion is registered for all commands after loading.

---

## Why Set-ExecutionPolicy is needed

Windows restricts running unsigned scripts by default. Setting the policy to `Bypass`
for the current process scope allows the dot-source to run without permanently
changing the system policy. The restriction returns to its default when the session ends.

---

## Suppress the banner

If you load the tools from a profile or script and do not want the confirmation output:

```powershell
. C:\Projects\NewGitProject\Register-RWTProjectTools_v6.ps1 `
    -ConfigPath C:\Projects\NewGitProject\RWTProjectTools_config_v6.psd1 `
    -SuppressBanner
```

---

## Verify SSH before pushing

Always confirm SSH auth is working at the start of a session where you intend to push:

```powershell
ssh-add -l          # confirm key is loaded in agent
ssh -T git@github.com   # confirm GitHub authentication
gh auth status      # confirm gh CLI authentication
```

See [SSH and ssh-agent Setup](SSH-and-ssh-agent-Setup.md) if any of these fail.
