# Windows Subsystem for Linux - Dotfiles

## About

- Working on Windows 10 with WSL
- Having a visually nice terminal through Windows Terminal (Preview)
- `zsh` as my main shell with `oh-my-zsh` as well
- Using Docker and Docker Compose directly from zsh
- Using VSCode (Insiders) directly from WSL 2

## Contents

- Host: Windows 11 Pro x64
  - Ubuntu via WSL 2 (Windows Subsystem for Linux)
  - Docker Desktop
- Terminal: Windows Terminal Preview
- Shell: zsh
  - git
  - docker (works with Docker Desktop)
  - docker-compose (works with Docker Desktop)
- Node.js (using `nvm`)
  - node
  - npm
  - yarn
- IDE: VSCode Insiders and the Remote WSL Extension
- WSL Bridge: allow exposing WSL 2 ports on the network

## Initial Setup

- Enable WSL 2 and update the linux kernel ([Source](https://docs.microsoft.com/en-us/windows/wsl/install-win10))

```powershell
# In PowerShell as Administrator

# Enable WSL and VirtualMachinePlatform features
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# Download and install the Linux kernel update package
$wslUpdateInstallerUrl = "https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi"
$downloadFolderPath = (New-Object -ComObject Shell.Application).NameSpace('shell:Downloads').Self.Path
$wslUpdateInstallerFilePath = "$downloadFolderPath/wsl_update_x64.msi"
$wc = New-Object System.Net.WebClient
$wc.DownloadFile($wslUpdateInstallerUrl, $wslUpdateInstallerFilePath)
Start-Process -Filepath "$wslUpdateInstallerFilePath"

# Set WSL default version to 2
wsl --set-default-version 2
```

- [Install Ubuntu from Microsoft Store](https://www.microsoft.com/fr-fr/p/ubuntu/9nblggh4msv6)


## Install Common Dependencies
---------------------------

```bash
#!/bin/bash

sudo apt update && sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common \
    git \
    make \
    tig \
    zsh
```

## Setup GPG Key

> Note: exporting already created GPG keys from windows first and then importing to WSL distro's user directory.

If you already have a GPG key, restore it. If you did not have one, you can create one.

### Export from Windows

- On windows, create a backup of a GPG key
  - `gpg --list-secret-keys`
  - `gpg --export-secret-keys {{KEY_ID}} > private.key`
- Import the key to WSL:
  - `gpg --import /mnt/c/users/<username>/private.key`
- Delete the `private.key`

### Create a New Key

- `gpg --full-generate-key`

[Read GitHub documentation about generating a new GPG key for more details](https://docs.github.com/en/github/authenticating-to-github/generating-a-new-gpg-key).

## Setup Git

```bash
#!/bin/bash

# Set username and email for next commands
email="<email>"
username="<name>"
gpgkeyid="<gpg key>"

# Configure Git
git config --global user.email "${email}"
git config --global user.name "${username}"
git config --global user.signingkey "${gpgkeyid}"
git config --global commit.gpgsign true
git config --global core.pager /usr/bin/less
git config --global core.excludesfile ~/.gitignore_global
git config --global core.attributesfile ~/.gitattributes_global
git config --global color.ui "auto"
git config --global default.protocol "ssh"
git config --global init.defaultBranch "main"

# Generate a new SSH key
ssh-keygen -t rsa -b 4096 -C "${email}"

# Start ssh-agent and add the key to it
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_rsa

# copy key to clipboard (need xclip)
sudo apt install xclip -y
cat ~/.ssh/id_rsa.pub | xclip -sel clip

# launch github to add ssh key to account
powershell.exe -Command 'start https://github.com/settings/ssh/new'
```

- [Add the generated key to GitHub](https://github.com/settings/ssh/new)

### My .gitconfig

```bash
[user]
	email = jimmy.briggs@jimbrig.com
	name = Jimmy Briggs
	signingKey = <REDACTED>
	autocrlf = input
[github]
	user = jimbrig
[default]
	protocol = ssh
[gpg]
	program = /usr/bin/gpg
[init]
	defaultBranch = main
[commit]
	gpgSign = true
[tag]
	forceSignAnnotated = true
[core]
	editor = code-insiders --wait --new-window
	excludesfile = ~/.gitignore_global
  	attributesfile = ~/.gitattributes_global
[diff]
	tool = code-insiders
	renames = copies
[difftool "code-insiders"]
	cmd = code-insiders --wait --diff $LOCAL $REMOTE
[merge]
	tool = code-insiders
	log = true
[mergetool "code-insiders"]
	cmd = code-insders --wait $MERGED
	trustexitcode = true
[color]
	ui = auto
[color "branch"]
    	current = yellow reverse
    	local = yellow
    	remote = green
[color "diff"]
    	meta = yellow bold
    	frag = magenta bold
    	old = red bold
    	new = green bold
[color "status"]
    	added = yellow
    	changed = green
    	untracked = cyan
    	branch = magenta
[help]
	autocorrect = 1
[apply]
	whitespace = fix
[rerere]
	enabled = true
[submodule]
	recurse = true
```

### Link Git Config Files back to Dotfiles

``bash
#!/bin/zsh

# move from home to dotdir (since configured above)
mv ~/.gitconfig ~/dev/wsl-dotfiles/ubuntu-commprev/home/jimbrig/.gitconfig

# link back
ln -sf ~/dev/wsl-dotfiles/ubuntu-commprev/home/jimbrig/.gitconfig ~/.gitconfig

# add links for gitignore and gitattributes (global)
ln -sf ~/dev/wsl-dotfiles/ubuntu-commprev/home/jimbrig/.gitignore_global ~/.gitignore_global
ln -sf ~/dev/wsl-dotfiles/ubuntu-commprev/home/jimbrig/.gitattributes_global ~/.gitattributes_global

# push
cd ~/dev/wsl-dotfiles
git add ubuntu-commprev/home/jimbrig/**
git commit -m "add updated gitconfig"
git push --set-upstream origin main
```

## Setup Shell with `zsh`

```bash
#!/bin/zsh

# Clone the dotfiles repository
mkdir -p ~/dev/wsl-dotfiles
git clone git@github.com:jimbrig/wsl-dotfiles.git ~/dev/dotfiles

# install zsh
sudo apt -y install zsh

# clone oh-my-zsh
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh

# Install some external plugins:
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-completions ~/.oh-my-zsh/custom/plugins/zsh-completions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.zsh/zsh-syntax-highlighting

# Set Zsh as your default shell:
chsh -s /bin/zsh

# (optional) Install Antibody Plugin Manager
curl -sfL git.io/antibody | sudo sh -s - -b /usr/local/bin

# Add plugins to ~/.zsh_plugins.zsh using antibody
antibody bundle < ~/dev/wsl-dotfiles/zsh_plugins > ~/.zsh_plugins.zsh

# Link custom dotfiles
ln -sf ~/dev/wsl-dotfiles/ubuntu-commprev/home/jimbrig/.aliases.zsh ~/.aliases.zsh
ln -sf ~/dev/wsl-dotfiles/ubuntu-commprev/home/jimbrig/.p10k.zsh ~/.p10k.zsh
ln -sf ~/dev/wsl-dotfiles/ubuntu-commprev/home/jimbrig/.zshrc ~/.zshrc

# Create .screen folder used by .zshrc
mkdir ~/.screen && chmod 700 ~/.screen

# Change default shell to zsh
chsh -s $(which zsh)
```

## Docker

### Install Docker Desktop

- [Install Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-windows)
- Make sure that the "Use the WSL 2 based engine" option is checked in Docker Desktop settings

```powershell
sudo cinst -y docker-desktop
```

### Setup Docker CLI

```zsh
#!/bin/zsh

# Add Docker to sources.list
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
versionCodename=$(cat /etc/os-release | grep VERSION_CODENAME | cut -d= -f2)
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(versionCodename) stable"

# Install tools
sudo apt update && sudo apt install -y \
    docker-ce

# Add user to docker group
sudo usermod -aG docker $USER
```

## Node.js, NPM, and NVM

```zsh
#!/bin/zsh

# Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | zsh

# install node and npm
nvm install --lts
node --version && npm --version

# Update NPM
npm install -g npm

# Login
npm login

# View stars for reference
npm stars

# install some globals
npm install -g bower create-next-app create-react-app cross-env dbdocs doctoc eslint gulp jshiny npm-check-updates npm-check vercel yarn

# doctor
npm doctor
```

## Setup Windows Terminal

- [Download and install JetBrains Mono](https://www.jetbrains.com/mono/)
- [Install Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701)

```zsh
#!/bin/zsh

windowsUserProfile=/mnt/c/Users/$(cmd.exe /c "echo %USERNAME%" 2>/dev/null | tr -d '\r')

# Copy Windows Terminal settings
cp ~/dev/dotfiles/terminal-settings.json ${windowsUserProfile}/AppData/Local/Packages/Microsoft.WindowsTerminal_8wekyb3d8bbwe/LocalState/settings.json
```

## WSL Bridge

When a port is listening from WSL 2, it cannot be reached.
You need to create port proxies for each port you want to use.
To avoid doing than manually each time I start my computer, I've made the `wslb` alias that will run the `wsl2bridge.ps1` script in an admin Powershell.

```zsh
#!/bin/zsh

windowsUserProfile=/mnt/c/Users/$(cmd.exe /c "echo %USERNAME%" 2>/dev/null | tr -d '\r')

# Get the hacky network bridge script
cp ~/dev/dotfiles/wsl2-bridge.ps1 ${windowsUserProfile}/wsl2-bridge.ps1
```

In order to allow `wsl2-bridge.ps1` script to run, you need to update your PowerShell execution policy.

```powershell
# In PowerShell as Administrator

Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
PowerShell -File $env:USERPROFILE\\wsl2-bridge.ps1
```

Then, when port forwarding does not work between WSL 2 and Windows, run `wslb` from zsh:

```zsh
#!/bin/zsh

wslb
```

Note: This is a custom alias. See [`.aliases.zsh`](.aliases.zsh) for more details


Limit WSL 2 RAM consumption
---------------------------

```zsh
#!/bin/zsh

windowsUserProfile=/mnt/c/Users/$(cmd.exe /c "echo %USERNAME%" 2>/dev/null | tr -d '\r')

# Avoid too much RAM consumption
cp ~/dev/dotfiles/.wslconfig ${windowsUserProfile}/.wslconfig
```

Note: You can adjust the RAM amount in `.wslconfig` file. Personally, I set it to 8 GB.

