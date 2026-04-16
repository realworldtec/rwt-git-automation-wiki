# Troubleshooting

Errors encountered during development of these scripts and how they were resolved.

---

## `CreateProcessW failed error:193` in VSCode

**Symptom:** VSCode Source Control shows this error when attempting to push or sync.
The git log shows:

```
CreateProcessW failed error:193
ssh_askpass: posix_spawnp: Unknown error
git@github.com: Permission denied (publickey).
```

**Cause:** VSCode runs git in a non-interactive background process. When `ssh-agent`
is not running (or the key is not loaded), SSH tries to prompt for the passphrase
using `ssh_askpass`, which is a POSIX helper that cannot launch in a Windows GUI context.
The result is a process creation failure (error 193) and a permission denied.

**Fix:**

```powershell
# Run elevated
Set-Service -Name ssh-agent -StartupType Automatic
Start-Service ssh-agent

# Add your key (prompted once)
ssh-add "$HOME\.ssh\id_mygithubkey"
```

Restart VSCode after running these commands. See [SSH and ssh-agent Setup](SSH-and-ssh-agent-Setup).

---

## `Parameter set cannot be resolved` from PowerShell

**Symptom:** Calling a script with `-Force` throws:

```
Parameter set cannot be resolved using the specified named parameters.
One or more parameters issued cannot be used together or an insufficient
number of parameters were provided.
```

**Cause:** `[CmdletBinding(SupportsShouldProcess = $true)]` reserves `-Force` as part
of its own internal parameter set machinery, which conflicts with a user-declared
`[switch]$Force`. PowerShell throws this error at parameter bind time before any script
body executes.

**Fix:** Remove `SupportsShouldProcess = $true` from `[CmdletBinding()]` if the script
body never calls `$PSCmdlet.ShouldProcess(...)`. The fix in `Publish-RWTExistingRepo_v6.ps1`
was to change `[CmdletBinding(SupportsShouldProcess = $true)]` to `[CmdletBinding()]`.

---

## `Parameter set cannot be resolved` from `Split-Path`

**Symptom:** The same error as above but occurring inside the script body at a line like:

```powershell
$GitHubRepo = (Split-Path -LiteralPath $resolvedRepoPath -Leaf)
```

**Cause:** In Windows PowerShell 5.1, `-LiteralPath` and `-Leaf` belong to different
parameter sets for `Split-Path` and cannot be used together.

**Fix:** Use `-Path` instead of `-LiteralPath` when also using `-Leaf`:

```powershell
$GitHubRepo = (Split-Path -Path $resolvedRepoPath -Leaf)
```

---

## `gh repo view` GraphQL error under `Stop` error preference

**Symptom:** When checking whether a GitHub repo exists, the script throws:

```
gh.exe : GraphQL: Could not resolve to a Repository with the name 'owner/repo'. (repository)
```

**Cause:** `gh repo view` uses the GitHub GraphQL API. When the repo does not exist,
GraphQL returns an error response that `gh` writes to stderr. With
`$ErrorActionPreference = 'Stop'`, PowerShell promotes the stderr output to a terminating
error even when the exit code check is suppressed with `-IgnoreExitCode`.

**Fix:** Replace `gh repo view` with a REST API existence check wrapped in `try/catch`:

```powershell
$repoExists = $false
try {
    $null = & gh api "repos/$fullName" 2>&1
    if ($LASTEXITCODE -eq 0) { $repoExists = $true }
} catch { $repoExists = $false }
```

---

## Git informational stderr treated as terminating errors

**Symptom:** Commands like `git push` succeed (the push completes) but the script
throws an error afterwards:

```
git.exe : To github.com:owner/repo.git
...
Everything up-to-date
```

**Cause:** Git writes progress and informational output to stderr (not stdout). Capturing
with `2>&1` routes that stderr text through PowerShell's error stream. With
`$ErrorActionPreference = 'Stop'`, any text on the error stream terminates the script.

**Fix:** Do not use `2>&1` when capturing git output. Let stderr go directly to the
console as plain text, which PowerShell cannot intercept:

```powershell
$output = & git @Arguments        # not: & git @Arguments 2>&1
$exitCode = $LASTEXITCODE
if (-not $IgnoreExitCode -and $exitCode -ne 0) {
    throw "git failed: exit $exitCode"
}
```

---

## Suite tag date drift between sessions

**Symptom:** On a second run the next day, the script tries to create a new tag
`suite-baseline-2026-04-13` but `suite-baseline-2026-04-12` already exists locally,
causing:

```
fatal: ambiguous argument 'suite-baseline-2026-04-13': unknown revision or path
not in the working tree.
```

**Cause:** The `$SuiteTag` default was computed at script parse/load time (inside the
param block default expression), so it always reflected the day the session started,
not the day the function was called.

**Fix:** Default to an empty string in the param block and resolve the value at call
time, preferring any existing local `suite-baseline-*` tag:

```powershell
[string]$SuiteTag = ''
# ...
if ([string]::IsNullOrWhiteSpace($SuiteTag)) {
    $existingSuiteTag = (& git -C $RepoPath tag --list "suite-baseline-*" 2>$null |
        Select-Object -First 1)
    if (-not [string]::IsNullOrWhiteSpace($existingSuiteTag)) {
        $SuiteTag = $existingSuiteTag.Trim()
    } else {
        $SuiteTag = "suite-baseline-$((Get-Date).ToString('yyyy-MM-dd'))"
    }
}
```

---

## `GitHubOwner` parameter not found in bootstrap script

**Symptom:**

```
A parameter cannot be found that matches parameter name 'GitHubOwner'.
```

**Cause:** `New-RWTProject` in the Register wrapper uses `GitHubOwner` internally,
but `New-GitHubProject_v6.ps1` declares the parameter as `GitHubUser`. When the
wrapper splatted its `$invoke` hashtable directly to the bootstrap script, PowerShell
could not bind the `GitHubOwner` key.

**Fix:** Rename the key before splatting:

```powershell
if ($invoke.ContainsKey('GitHubOwner')) {
    $invoke['GitHubUser'] = $invoke['GitHubOwner']
    $null = $invoke.Remove('GitHubOwner')
}
```

---

## 28 unrecognized parameters causing splat errors

**Symptom:** Every call to `New-RWTProject` fails with multiple "parameter not found"
errors for keys like `ApplyAclFix`, `CreateChangelog`, `LicenseType`, `PresetName`, etc.

**Cause:** The config `Defaults` and preset hashtables contain many keys that are useful
for documentation or future use but are not declared as parameters in
`New-GitHubProject_v6.ps1`. When the full merged `$invoke` hashtable was splatted to the
bootstrap script, all unrecognized keys caused bind errors.

**Fix:** A whitelist `HashSet` of the exact parameter names `New-GitHubProject_v6.ps1`
accepts. The wrapper filters `$invoke` through this set before calling the bootstrap:

```powershell
$filteredInvoke = @{}
foreach ($key in $invoke.Keys) {
    if ($script:BootstrapParamWhitelist.Contains($key)) {
        $filteredInvoke[$key] = $invoke[$key]
    }
}
Invoke-RWTProjectBootstrap -Bound $filteredInvoke
```

---

## `Repository not found` on push

**Symptom:**

```
fatal: Could not read from remote repository.
ERROR: Repository not found.
```

**Cause:** The GitHub repository did not exist yet. The push authenticated successfully
but had nowhere to push to.

**Fix:** Add `-CreateGitHubRepo` to the `Publish-RWTExistingRepo` call. This creates
the repository via the GitHub REST API before attempting the push:

```powershell
Publish-RWTExistingRepo `
    -RepoPath "C:\Projects\my-project" `
    -RemoteUrl "git@github.com:owner/my-project.git" `
    -CreateGitHubRepo `
    -GitHubOwner "owner"
```
