# SSH and ssh-agent Setup

This page covers SSH key generation, the `~/.ssh/config` file, and configuring
`ssh-agent` so that VSCode, PowerShell, and `gh` can all push to GitHub without
being prompted for a passphrase in every operation.

---

## Why ssh-agent matters

A passphrase-protected SSH key is more secure than one without a passphrase, but
every process that uses the key would normally prompt for it interactively. VSCode's
git integration runs git in a non-interactive background process and cannot show a
passphrase prompt — it fails silently with `CreateProcessW failed error:193` or
`Permission denied (publickey)`.

`ssh-agent` solves this by holding the decrypted key in memory for the session.
Any process — PowerShell, VSCode, `gh` — can use it without a prompt.

---

## 1. Enable the ssh-agent service

Run in an **elevated** PowerShell session:

```powershell
Set-Service -Name ssh-agent -StartupType Automatic
Start-Service ssh-agent
```

Setting `StartupType Automatic` means the agent starts with Windows. You will
only need to run `ssh-add` once after each reboot.

Verify the service is running:

```powershell
Get-Service ssh-agent
```

---

## 2. Generate an SSH key

```powershell
ssh-keygen -t ed25519 -C "your@email.com" -f "$HOME\.ssh\id_mygithubkey"
```

- `-t ed25519` — modern, compact key type; preferred over RSA
- `-C` — comment embedded in the public key, typically your email
- `-f` — output filename; choose a descriptive name if you have multiple keys

You will be prompted to set a passphrase. **Use a passphrase.** The agent will
handle it so you are only prompted once per session.

This creates two files:

```
C:\Users\YourName\.ssh\id_mygithubkey        (private key — never share this)
C:\Users\YourName\.ssh\id_mygithubkey.pub    (public key — add this to GitHub)
```

---

## 3. Add the public key to GitHub

```powershell
# Copy the public key to clipboard
Get-Content "$HOME\.ssh\id_mygithubkey.pub" | Set-Clipboard
```

Then in your browser: **GitHub → Settings → SSH and GPG keys → New SSH key**

Paste the public key, give it a descriptive title (e.g. `dev-vm-2026`), and save.

Alternatively, use the CLI:

```powershell
gh ssh-key add "$HOME\.ssh\id_mygithubkey.pub" --title "dev-vm-2026"
```

---

## 4. Create the SSH config file

Create or edit `C:\Users\YourName\.ssh\config`:

```
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_mygithubkey
  IdentitiesOnly yes
```

`IdentitiesOnly yes` prevents SSH from trying other keys in the agent before this one,
which avoids confusing authentication failures when multiple keys are loaded.

---

## 5. Add the key to the agent

```powershell
ssh-add "$HOME\.ssh\id_mygithubkey"
```

You will be prompted for the passphrase **once**. After this, the key is held in
memory and all subsequent operations are passphrase-free for the rest of the session.

Verify the key is loaded:

```powershell
ssh-add -l
```

Expected output:

```
256 SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx your@email.com (ED25519)
```

---

## 6. Test the connection

```powershell
ssh -T git@github.com
```

Expected output (exit code 1 is normal — GitHub does not allow shell access):

```
Hi yourusername! You've successfully authenticated, but GitHub does not provide shell access.
```

---

## 6. Configure git to use Windows OpenSSH

```powershell
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```

This tells Git for Windows to use the Windows built-in SSH executable rather than its own
bundled copy. This is essential for VSCode integration: Git for Windows' bundled SSH cannot
reliably communicate with the Windows `ssh-agent` service, causing pushes from VSCode to fail
even when the key is loaded in the agent.

Use forward slashes in the path even on Windows.

---

## After a reboot

The agent service starts automatically, but the key must be re-added:

```powershell
ssh-add "$HOME\.ssh\id_mygithubkey"
```

To avoid this, some environments use a PowerShell profile to add the key automatically on
login — but that triggers a passphrase prompt at every shell startup, which is a matter
of preference.

---

## Multiple keys

If you have keys for multiple GitHub accounts or services, add a separate `Host` block
for each in `~/.ssh/config`:

```
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_personal
  IdentitiesOnly yes

Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_work
  IdentitiesOnly yes
```

For the work account, use `git@github-work:org/repo.git` as the remote URL instead of
`git@github.com:org/repo.git`.
