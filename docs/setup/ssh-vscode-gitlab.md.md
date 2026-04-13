# SSH Key Setup - VS Code to Self-Hosted GitLab

## Overview
This guide covers setting up SSH key authentication to connect VS code on your main machine to a self-hosted GitLab instance running on a remote server. Once configured, you can edit and push to your GitLab repos directly from VS Code without entering a password.

```text
VS Code (Main Machine) -> SSH -> Server(<server-ip>) -> GitLab 
```

## Prerequisites
- VS Code installed with the <b>Remote - SSH</b> extension
- Remote server reachable on the local network
- Self-hosted GitLab running on the server with an exposed SSH port

## Step 1 - Generate SSH Key on Main Machine
Open <b>PowerShell</b> on your main machine and run:
```powershell
ssh-keygen -t ed25519 -C "your-machine-name"
```
- Accept the default path `C:\Users\<username>\.ssh\id_ed25519` 
- Skip the passphrase by pressing Enter twice

This generates two files:
- `id_ed25519` - private key, never share this
- `id_ed25519.pub` - public key, this gets copied to other machines

## Step 2 - Copy Public key to Server
```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh <user>@<server-ip> "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```
Enter your server password when prompted - this is the last time you'll need it.

<b>Verify it works: </b>
```powershell
ssh <user>@<server-ip>
# Should connect without a password prompt
```

## Step 3 - Configure VS Code Remote-SSH

Open VS Code and press `Ctrl+Shift+P` -> `Remote-SSH: Open SSH Configuration File` -> select `C:\Users\<username>\.ssh\config

Add the following to the config file:
```text
Host <server-alias>
    HostName <server-ip>
    User <user>
    IdentityFile ~/.ssh/id_ed25519
```
Save the file and connect it via `Ctrl+Shift+P` -> `Remote-SSH: Connect to Host` -> select your server alias.

## Step 4 - Generate SSH Key on Server (for GitLab access)

Open a terminal in VS Code (now connected to the server) and run:
```bash
ssh-keygen -t ed25519 -C "server"
# Accept default path: ~/.ssh/id_ed25519
```
Copy the public key
```bash
cat ~/.ssh/id_ed25519.pub
```

## Step 5 - Add SSH Keys to GitLab

Go to your GitLab instance -> <b>Profile -> Edit Profile -> Access -> SSH Keys -> Add new key</b>

Add both keys:
| Label | Source |
|---|---|
| `main` | Output of `cat $env:USERPROFILE\.ssh\id_ed25519.pub` on main machine |
| `server` | Output of `cat ~/.ssh/id_ed25519.pub` on server |

## Step 6 - Clone the Repo

> <b>Note</b>: If your GItLab SSH port is non-standard, HTTP/HTTPS clone will fail — always use the SSH URL format below

```bash
git clone ssh://git@<server-ip>:<gitlab-ssh-port>/<username>/<repo>.git
cd <repo>
```

Accept the host fingerprint prompt by typing `yes`.

## Step 7 — Configure Git Identify

```bash
git config --global user.name "Your name"
git config --global user.email "your@email.com"
```
## Troubleshooting

| Error | Cause | Fix |
| --- | --- | ---|
| `HTTP/0.9 when not allowed` | Trying to clone via HTTP on an SSH Port | Use `ssh://` URL format |
| `gnutls_handshake() failed` | Trying HTTPS on an SSH-only port | Use `ssh://` URL format |
| `Permission denied (publickey)` | SSH key not added to GitLab | Add public key in GitLab profile settings |
| `remote rejected — protected branch` | Force pushing to a protected branch | Unprotect branch in GitLab → Settings → Repository |