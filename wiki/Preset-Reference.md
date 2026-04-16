# Preset Reference

Presets are named parameter bundles defined in the `Presets` section of the config file.
They are merged into a `New-RWTProject` call after explicit parameters and config defaults,
filling in any values not already set.

Presets can be invoked two ways:

```powershell
# Via named switch (shorthand)
New-RWTProject MyProject -PrivateWithWiki

# Via -PresetName (explicit)
New-RWTProject MyProject -PresetName RWT
```

---

## Built-in presets

### RWT

The standard RWT project baseline. Applied automatically as the default preset
unless overridden. Sets up a private PowerShell project with VS Code, full docs
scaffolding, and ACL hardening.

```
RootPath                  = C:\Projects
Editor                    = VSCode
ProjectType               = PowerShell
GitHubVisibility          = private
LicenseType               = PolyForm-Noncommercial-Notice
CreateReadme              = true
CreateGitAttributes       = true
CreateEditorConfig        = true
CreateIssueTemplates      = true
CreatePullRequestTemplate = true
CreateGitHubWorkflows     = true
CreateDocsIndex           = true
CreateChangelog           = true
ApplyAclFix               = true
DisableAclInheritance     = true
```

---

### PrivateWithWiki

Creates a private GitHub repository with a companion public wiki repo. Suitable for
projects that need private code and public documentation.

Invoked with: `-PrivateWithWiki`

```
CreateGitHubRepo     = true
GitHubVisibility     = private
CreatePublicWikiRepo = true
PushInitialCommit    = true
CreateReadme         = true
CreateStarterFiles   = true
```

---

### PowerShellModule

A PowerShell module project with starter files, full docs scaffolding, and GitHub Actions.

Invoked with: `-PowerShellModule`

```
ProjectType           = PowerShell
CreateStarterFiles    = true
CreateReadme          = true
CreateGitAttributes   = true
CreateEditorConfig    = true
CreateGitHubWorkflows = true
CreateDocsIndex       = true
CreateChangelog       = true
```

---

### DotNetService

A .NET Web API project with controllers and an xUnit test project.

Invoked with: `-DotNetService`

```
ProjectType           = DotNet
DotNetTemplate        = webapi
UseControllers        = true
CreateDotNetTests     = true
CreateReadme          = true
CreateGitAttributes   = true
CreateEditorConfig    = true
CreateGitHubWorkflows = true
CreateDocsIndex       = true
CreateChangelog       = true
```

---

### GenericScratch

A local-only scratch repo. No remote, no commit, no editor launch.

Invoked with: `-GenericScratch`

```
ProjectType        = Generic
NoOpen             = true
NoPush             = true
NoCommit           = true
CreateStarterFiles = true
```

---

### LocalOnly

Suppresses all remote operations and editor launches. Useful when you want
a local project without any GitHub involvement at creation time.

Invoked with: `-LocalOnly`

```
NoOpen = true
NoPush = true
```

---

### FinanceAuditBaseline

A stripped-down preset for publishing an existing audit parser repository as a
controlled baseline. Disables all scaffolding, skips the GitHub repo creation at
`New-RWTProject` time, and applies ACL hardening.

Invoked with: `-FinanceAuditBaseline`

```
ProjectType               = Generic
GitHubVisibility          = private
DefaultBranch             = main
CreateGitHubRepo          = false
CreatePublicWikiRepo      = false
PushInitialCommit         = true
CreateReadme              = false
CreateGitAttributes       = false
CreateEditorConfig        = false
CreateIssueTemplates      = false
CreatePullRequestTemplate = false
CreateGitHubWorkflows     = false
CreateDocsIndex           = false
CreateChangelog           = false
CreateStarterFiles        = false
ApplyAclFix               = true
DisableAclInheritance     = true
LaunchEditor              = false
NoOpen                    = true
NoHealthChecks            = true
NoHooks                   = true
```

---

## Preset resolution order

When `New-RWTProject` is called, parameters are resolved in this order (later steps
fill in only what is not already set):

1. Explicit parameters passed on the command line
2. Config `Defaults` section
3. `DefaultRootPath` and `DefaultGitHubOwner` from the top-level config
4. Named preset from `-PresetName` (or the default `PresetName` from `Defaults`)
5. Switch presets (`-PrivateWithWiki`, `-DotNetService`, etc.)

Explicit parameters always win. Presets never overwrite values already set.

---

## Adding a custom preset

Add a new entry to the `Presets` hashtable in `RWTProjectTools_config_v6.psd1`:

```powershell
Presets = @{
    # ... existing presets ...

    MyCustomPreset = @{
        ProjectType      = 'PowerShell'
        GitHubVisibility = 'private'
        CreateReadme     = $true
        CreateChangelog  = $true
        NoPush           = $true
    }
}
```

Also add the name to the `Completion` section so it appears in tab completion:

```powershell
Completion = @{
    PresetName = @('RWT', 'FinanceAuditBaseline', 'MyCustomPreset')
    # ...
}
```

Then reload:

```powershell
. C:\Projects\NewGitProject\Register-RWTProjectTools_v6.ps1 `
    -ConfigPath C:\Projects\NewGitProject\RWTProjectTools_config_v6.psd1
```
