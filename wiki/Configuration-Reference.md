# Configuration Reference

`RWTProjectTools_config_v6.psd1` is a PowerShell data file (`.psd1`) that drives all
defaults, presets, and tab-completion values. It is loaded once when you dot-source
`Register-RWTProjectTools_v6.ps1`.

---

## Top-level keys

| Key | Type | Purpose |
|---|---|---|
| `BootstrapScriptPath` | string | Full path to `New-GitHubProject_v6.ps1` |
| `ExistingRepoPublisherPath` | string | Full path to `Publish-RWTExistingRepo_v6.ps1` |
| `DefaultRootPath` | string | Parent folder for new projects (e.g. `C:\Projects`) |
| `DefaultGitHubOwner` | string | GitHub username or org used when none is specified |
| `Defaults` | hashtable | Default parameter values applied to every `New-RWTProject` call |
| `Presets` | hashtable | Named parameter bundles; each key is a preset name |
| `Completion` | hashtable | Tab-completion values for each parameter |

At least one of `BootstrapScriptPath` or `ExistingRepoPublisherPath` must be present.

---

## Defaults keys

Values here are merged into every `New-RWTProject` call after explicit parameters.
Explicit parameters always win over defaults.

| Key | Default value | Purpose |
|---|---|---|
| `PresetName` | `RWT` | Preset applied automatically to every call |
| `Editor` | `VSCode` | Editor to use: `Auto`, `VSCode`, `VisualStudio`, `None` |
| `ProjectType` | `PowerShell` | Project type: `Generic`, `PowerShell`, `DotNet`, `Node` |
| `GitHubVisibility` | `private` | Repo visibility: `private`, `public` |
| `LicenseType` | `PolyForm-Noncommercial-Notice` | License: `None`, `MIT`, `Apache-2.0`, `PolyForm-Noncommercial-Notice` |
| `DefaultBranch` | `main` | Default branch name |
| `WikiRepoSuffix` | `wiki` | Suffix appended to project name for companion wiki repo |
| `InitialVersion` | `0.1.0` | Starting version string |
| `CreateReadme` | `$true` | Write a `README.md` |
| `CreateGitAttributes` | `$true` | Write a `.gitattributes` file |
| `CreateEditorConfig` | `$true` | Write an `.editorconfig` file |
| `CreateIssueTemplates` | `$true` | Write GitHub issue templates |
| `CreatePullRequestTemplate` | `$true` | Write a GitHub PR template |
| `CreateGitHubWorkflows` | `$true` | Write GitHub Actions workflow stubs |
| `CreateCodeOwners` | `$false` | Write a `CODEOWNERS` file |
| `CreateDocsIndex` | `$true` | Write a `docs/` index file |
| `CreateChangelog` | `$true` | Write a `CHANGELOG.md` |
| `CreateDevContainer` | `$false` | Write a `.devcontainer` config |
| `CreateLicense` | `$false` | Write a `LICENSE` file |
| `ApplyAclFix` | `$true` | Apply ACL hardening to the project folder |
| `DisableAclInheritance` | `$true` | Disable ACL inheritance on the project folder |
| `LaunchEditor` | `$false` | Open editor after creation |
| `NoOpen` | `$false` | Suppress all editor/browser launches |
| `NoHealthChecks` | `$false` | Skip pre-run health checks |
| `NoHooks` | `$false` | Skip git hook installation |

> Keys that are not accepted by `New-GitHubProject_v6.ps1` (such as `ApplyAclFix`,
> `CreateChangelog`, `LicenseType`) are safe to include here. The bootstrap parameter
> whitelist in `Register-RWTProjectTools_v6.ps1` strips them before forwarding.

---

## Completion keys

These drive tab completion in the PowerShell session after loading.

| Key | Values |
|---|---|
| `PresetName` | `RWT`, `FinanceAuditBaseline` |
| `Editor` | `Auto`, `VSCode`, `VisualStudio`, `None` |
| `ProjectType` | `Generic`, `PowerShell`, `DotNet`, `Node` |
| `GitHubVisibility` | `private`, `public` |
| `LicenseType` | `None`, `MIT`, `Apache-2.0`, `PolyForm-Noncommercial-Notice` |
| `DotNetTemplate` | `console`, `classlib`, `webapi`, `mvc`, `worker` |
| `DotNetTestTemplate` | `xunit`, `nunit` |
| `WikiRepoSuffix` | `wiki`, `pub`, `docs` |

---

## Inspecting the loaded config at runtime

```powershell
Get-RWTProjectToolConfig | Format-List
```

## Updating config at runtime

```powershell
# Change the default owner without reloading
Set-RWTProjectToolConfig -DefaultGitHubOwner "newowner"

# Reload entirely from a different config file
Set-RWTProjectToolConfig -ConfigPath "C:\Projects\NewGitProject\alt_config.psd1"
```

`Set-RWTProjectToolConfig` accepts any combination of its parameters independently.
Changing `-ConfigPath` reloads all keys from the new file. Changing individual keys
updates only those values in the in-memory config.
