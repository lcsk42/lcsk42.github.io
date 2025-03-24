---
title: '软件列表'
slug: 'software-list'
description: 自用软件列表
date: 2025-03-20T14:24:22+08:00
categories:
  - Ops
tags:
  - Software List
---

<!--more-->

## Software List

为了方便新电脑和重装系统时（心理洁癖，当我觉得电脑特别乱时，就特别想把所有清空重置一下，干净的电脑可以给我一种干净的感觉？）可以快速恢复环境，记录一下自己使用到的软件。

### CLI

- [brew](https://gitee.com/cunkai/HomebrewCN): The Missing Package Manager for macOS (or Linux)

```shell
# The following commands need to be executed one by one.
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

#### Brew

- [act](https://github.com/nektos/act): Run your GitHub Actions locally
- [axel](https://github.com/axel-download-accelerator/axel): Light UNIX download accelerator
- [bat](https://github.com/sharkdp/bat): Clone of cat(1) with syntax highlighting and Git integration
- [docker](https://www.docker.com/): Pack, ship and run any application as a lightweight container
- [docker-compose](https://docs.docker.com/compose/): Isolated development environments using Docker
- [docker-completion](https://www.docker.com/): Bash, Zsh and Fish completion for Docker
- [fzf](https://github.com/junegunn/fzf): Command-line fuzzy finder written in Go
- [hugo](https://gohugo.io/): Configurable static site generator
- [jenv](https://www.jenv.be/): Manage your Java environment
- [jq](https://jqlang.github.io/jq/): Lightweight and flexible command-line JSON processor
- [just](https://github.com/casey/just): Handy way to save and run project-specific commands
- [lima](https://lima-vm.io/): Linux virtual machines
- [nvm](https://github.com/nvm-sh/nvm): Manage multiple Node.js versions
- [openjdk@17](https://openjdk.java.net/): Development kit for the Java programming language
- [openjdk@21](https://openjdk.java.net/): Development kit for the Java programming language
- [mvn](https://maven.apache.org/): Java-based project management
- [tmux](https://tmux.github.io/): Terminal multiplexer

```shell
brew install \
  act  \
  axel \
  bat \
  docker \
  docker-compose \
  docker-completion \
  fzf \
  hugo \
  jenv \
  jq \
  just \
  lima \
  nvm \
  openjdk@17 \
  openjdk@21 \
  mvn \
  tmux
```

#### CLI Configuration

##### .gitconfig

```shell
cat > ~/.gitconfig << EOF
[push]
	default = current
	autoSetupRemote = true
[user]
	name = lcsk42
	email = lcsk42@gmail.com
[core]
	pager = diff-so-fancy | less --tabs=4 -RFX
[interactive]
	diffFilter = diff-so-fancy --patch
[color]
	ui = true
[color "diff-highlight"]
	oldNormal = red bold
	oldHighlight = red bold 52
	newNormal = green bold
	newHighlight = green bold 22
[color "diff"]
	meta = 11
	frag = magenta bold
	func = 146 bold
	commit = yellow bold
	old = red bold
	new = green bold
	whitespace = red reverse
[init]
	defaultBranch = main
EOF
```

##### .gitignore_global

```shell
cat ~/.gitignore_global << EOF
*~
.DS_Store
.idea
.vscode
.nvm
target/
*.iml
/out/
EOF
```

##### .justfile

```shell
cat > ~/.justfile << EOF
default:
  @echo "\033[31mAlready at user home directory; no recipes available.\033[0m"
EOF
```

##### .tmux.conf

```shell
cat > ~/.tmux.conf << EOF
# -- general -------------------------------------------------------------------
set -g default-terminal "screen-256color" # colors!
setw -g xterm-keys on
set -s escape-time 10 # faster command sequences
set -sg repeat-time 600 # increase repeat timeout
set -q -g status-utf8 on # expect UTF-8 (tmux < 2.2)
setw -q -g utf8 on
set -g history-limit 5000 # boost history
# -- display -------------------------------------------------------------------
set -g base-index 1 # start windows numbering at 1
setw -g pane-base-index 1 # make pane numbering consistent with windows
setw -g automatic-rename on # rename window to reflect current program
set -g renumber-windows on # renumber windows when a window is closed
set -g set-titles on # set terminal title
set -g display-panes-time 800 # slightly longer pane indicators display time
set -g display-time 1000 # slightly longer status messages display time
set -g status-interval 10 # redraw status line every 10 seconds
# clear both screen and history
bind -n C-l send-keys C-l \; run 'sleep 0.1' \; clear-history
# activity
set -g monitor-activity on
set -g visual-activity off
# -- nvigation ----------------------------------------------------------------
# create session
bind C-c new-session
# find session
bind C-f command-prompt -p find-session 'switch-client -t %%'
# split current window horizontally
bind - split-window -v
# split current window vertically
bind _ split-window -h
# pane navigation
bind -r h select-pane -L # move left
bind -r j select-pane -D # move down
bind -r k select-pane -U # move up
bind -r l select-pane -R # move right
bind > swap-pane -D # swap current pane with the next one
bind < swap-pane -U # swap current pane with the previous one
#pane resizing
bind -r H resize-pane -L 2
bind -r J resize-pane -D 2
bind -r K resize-pane -U 2
bind -r L resize-pane -R 2
# window navigation
unbind n
unbind p
bind -r C-h previous-window # select previous window
bind -r C-l next-window # select next window
bind Tab last-window # move to last active window
EOF
```

##### .vimrc

```shell
cat > ~/.vimrc << EOF
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
" Open file type detection
filetype on
" Load the corresponding plugin according to the detected different types
filetype plugin on
" Always show the status bar
set laststatus=2
" Display the current position of the cursor
set ruler
" Turn on line number display
set number
" Highlight current row/column
set cursorline
" set cursorcolumn
" Highlight search results
set hlsearch
" Prohibition of folding
set nowrap
" Turn on syntax highlighting
syntax enable
" Allows replacement of default schemes with the specified syntax highlighting scheme
syntax on
" Adaptive smart indentation in different languages
filetype indent on
" Extend tabs to spaces
set expandtab
" Set tabs to occupy spaces when editing
set tabstop=2
" Set the number of spaces occupied by tabs when formatting
set shiftwidth=2
" Let vim treat a continuous number of spaces as a tab
set softtabstop=2
" Code folding based on indentation or grammar
"set foldmethod=indent
set foldmethod=syntax
" Turn off folding code when starting vim
set nofoldenable
EOF
```

### GUI

#### App Store

- [Bark](https://bark.day.app/#/): Bark is a push notification tool app
- [KeepingYouAwake](https://keepingyouawake.app/): Prevents your Mac from going to sleep
- [RunCat](https://kyome.io/runcat/?lang=en): The cat tells you the CPU usage of Mac by running speed

#### Brew Cask

- [Raycast](https://raycast.com/): Control your tools with a few keystrokes
- [iTerm2](https://iterm2.com/): Terminal emulator as alternative to Apple's Terminal app
- [Google Chrome](https://www.google.com/chrome/): Web browser
- [Wireshark](https://www.wireshark.org): Network analyzer and capture tool - without graphical user interface
- [IINA](https://iina.io/): Free and open-source media player
- [Input Source Pro](https://inputsource.pro/): Tool for multi-language users
- [Mos](https://mos.caldis.me/): Smooths scrolling and set mouse scroll directions independently
- [WeChat](https://mac.weixin.qq.com/): Free messaging and calling application
- [Discord](https://discord.com/): Voice and text chat software
- [Telegram](https://macos.telegram.org/): Messaging app with a focus on speed and security
- [Karabiner-Elements](https://karabiner-elements.pqrs.org/)Keyboard customiser
- [OBS](https://obsproject.com/): Open-source software for live streaming and screen recording
- [Visual Studio Code](https://code.visualstudio.com/): Open-source code editor

```shell
brew install --cask \
  iterm2 \
  google-chrome \
  wireshark \
  iina \
  input-source-pro \
  mos \
  wechat \
  telegram \
  discord \
  karabiner-elements \
  obs \
  visual-studio-code

```

#### GUI Configuration

##### karabiner-elements's Complex Modifications

F1:

```config
{
    "description": "Change right_command key to command+control+option+shift. (Post f19 key when pressed alone)",
    "manipulators": [
        {
            "from": {
                "key_code": "right_command",
                "modifiers": { "optional": ["any"] }
            },
            "to": [
                {
                    "key_code": "left_shift",
                    "modifiers": ["left_command", "left_control", "left_option"]
                }
            ],
            "to_if_alone": [{ "key_code": "f19" }],
            "type": "basic"
        }
    ]
}
```

F2:

```config
{
    "description": "Press caps_lock to activate as CTRL",
    "manipulators": [
        {
            "from": {
                "key_code": "caps_lock",
                "modifiers": { "optional": ["any"] }
            },
            "to": [{ "key_code": "left_control" }],
            "type": "basic"
        }
    ]
}
```
