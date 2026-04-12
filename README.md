# Setup Mac

## Settings

### Dock Settings

- Automatically hide dock
- Disable Show suggested and recent apps

### Finder

Tags
- Disable tags

Sidebar
- Disable Recents & Shared
- Add home to Locations
- Disable recent tags

Advanced
- Show all filename extensions

## Terminal Setup

### Install Brew, Terminal, and Shell

#### Install

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

```shell
brew install ghostty fish zsh starship neovim
```

#### Set default shell and disable last login message

```shell
command -v fish | sudo tee -a /etc/shells
chsh -s "$(command -v fish)"
touch ~/.hushlogin
```

## Change system settings

### Disable DS_Store 

```shell
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE
defaults write com.apple.desktopservices DSDontWriteUSBStores -bool TRUE
```

### Faster dock

```shell
defaults write com.apple.dock autohide-delay -float 0
defaults write com.apple.dock autohide-time-modifier -float 0.15
killall Dock
```

## Applications

### Desktop Experience

```shell
brew install bitwarden alt-tab aldente bartender moom mos stats
```

### Development

#### Basics
```shell
bash git gnupg jetbrains-toolbox
```


#### Kubernetes
```shell
brew install argocd helm kubernetes-cli talosctl
```

#### AI
```shell
brew install claude claude-code
```

### Games
```shell
brew install discord steam
```