---
title: 'Macos Setup'
slug: 'macos-setup'
description: Mac 开发环境与娱乐软件配置
date: 2025-04-29T15:07:31+08:00
categories:
  - Ops
tags:
  - Mac
  - Software
---

<!--more-->

## 初始化配置

### 鼠标触控板

- 鼠标速度最大：Setting -> Mouse -> Tracking speed 调整到 Fast
- 轻点来点按：Setting -> Trackpad -> Tap to click 打开
- 三指拖移：Setting -> Accessibility -> Pointer Control -> Trackpad Options ->
  - Use trackpad for dragging: true
  - Dragging style: Three Finger Drag
- 关闭触发角：Setting -> Desktop & Dock -> 最下方右下角 Hot Corners,点开后设置所有为空

### Homebrew (包管理)

确实无愧于自己的 slogan - The Missing Package Manager for macOS。

软件管理必备了已经是。

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

以上命令在国内环境下因为不可抗因素会执行失败，可以尝试使用手机热点进行下载（无需代理），或者也可以直接使用 Gitee 用户提供的代理来安装：

```shell
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

### 科学上网

网络是个拦路虎，首先需要先安装代理软件，否则只能在百度中看满屏的广告了。

从 [Clash Verge Rev](https://github.com/clash-verge-rev/clash-verge-rev/releases) 中下载最新版本，进行安装。

获取最新版本下载地址之后，如果下载过慢，可以使用资源访问加速服务进行下载：[gh.lcsk42.com](https:gh.lcsk42.com)

该项目来自 [hunshcn/gh-proxy](https://github.com/hunshcn/gh-proxy)

对其前端页面进行修改之后，部署在了 [Cloudflare Workers](https://workers.cloudflare.com/) 上。

无限感恩赛博佛祖 Cloudflare。

### 按键映射

```shell
brew install --cask karabiner-elements
```

自定义按键转换，我一般常用的有两个：

F1: CapsLock 转换为 Ctrl, CapsLock 占用了那么大的位置，但是用处不多（自带的键盘作用还比较多，外接键盘好像确实没有什么用处）。

```txt
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

F2: 将右 command 键转换为 F19, 主要是方便启动器，一般会把这个键位绑定到 Raycast 上替换 自带的 spotlight。

```txt
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

### 鼠标反转

```shell
brew install --cask mos
```

将鼠标滚动方向反转，并保留触控板方向不变。

Mac 的滚轮逻辑挺反人类的，刚开始尝试适应了一段时间，结果发现并不能适应，感谢 [Mos](https://mos.caldis.me/) 救我。

## Bash 配置

### iTerm2

仿真终端，替换自带的 Terminal，增加更多的主题和功能。

[Maple Mono](https://font.subf.dev/en/) 是最近非常喜欢的一个字体，在代码中大量的看会显得有点乱，但是在终端中使用很完美（可能是因为终端中的数据都比较规整，排排站的原因？）。

```shell
brew install --cask iterm2 font-maple-mono-nf-cn
```

安装完成后设置字体：

Setting -> Profiles -> Text -> Font: Maple Mono NF CN Regular 16 100 100

### oh-my-zsh

很慢，但是目前暂时没有时间去寻找更好的替代品，所以咬咬牙，先继续使用。

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

如果上述命令执行失败，依旧可以尝试国内 Gitee 的代理：

```shell
# 进入此页面复制脚本内容，创建文件 install.sh，粘贴进文件：
https://gitee.com/mirrors/oh-my-zsh/blob/master/tools/install.sh
# 替换地址
- REMOTE=${REMOTE:-https://github.com/${REPO}.git}
+ REMOTE=${REMOTE:-https://gitee.com/mirrors/oh-my-zsh.git}
# 增加脚本权限，然后执行脚本即可
chmod +x ./install.sh
./install.sh
```

配置文件(.zshrc)：

```txt
# Alias
alias vi=vim
alias v=vim
# alias cat=bat
alias n=neofetch
alias lg=lazygit
alias ,o="open ."
alias ,c="code ."
alias sz="source ~/.zshrc"
alias cdtmp='cd `mktemp -d /tmp/lucas-XXXXXXXX`'
alias hosts="sudo vim /etc/hosts"
alias tail=tailspin
alias weather="clear & curl https://v2d.wttr.in/zhengzhou"

function mcd {
  mkdir $1 && cd $1
}

function ppgrep() {
    if [[ $1 == "" ]]; then
        FZF=fzf
    else
        FZF="fzf -q $1"
    fi
    ps aux | eval $FZF | awk '{ print $2 }'
}

function ppkill() {
    if [[ $1 =~ "^-" ]]; then
        QUERY=""            # options only
    else
        QUERY=$1            # with a query
        [[ $# > 0 ]] && shift
    fi
    ppgrep $QUERY | xargs kill $*
}

```

### p10k

也是一个用的比较习惯的主题了。

```shell
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"
```

国内可以使用以下命令替换

```shell
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"
```

打开 `~/.zshrc`,找到 `ZSH_THEME`, 替换为 `powerlevel10k/powerlevel10k` 即可。

### dotfile

#### .gitconfig

个人 git 相关配置

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

#### .gitignore_global

全局的 git 忽略类型

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

#### .tmux.conf

tmux 基础配置（类 vim）

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

## Chrome 配置

开发必备了，事实上的前端标准，无数的封装，一个用户的电脑中实际安装可能高达几十个的运行环境。

```shell
brew install --cask google-chrome
```

使用 google 账号进行数据同步。

### 插件

#### Aria2 Explorer

**地址：** <https://chromewebstore.google.com/detail/aria2-explorer/mpkodccbngfoacfalldjimigbofkhgjn?hl=en>

**功能：** 配合下载软件(aria2c)一起使用

直接使用打包好的 docker 镜像进行安装：
[Aria2 Pro - 更好用的 Aria2 Docker 容器镜像](https://p3terx.com/archives/docker-aria2-pro.html)

参照提供的 [docker-compose.yml](https://github.com/P3TERX/Aria2-Pro-Docker/blob/master/docker-compose.yml) 示例，根据需要修改如下：

```yml
services:
  Aria2-Pro:
    container_name: aria2-pro
    image: p3terx/aria2-pro
    environment:
      - PUID=65534
      - PGID=65534
      - UMASK_SET=022
      - RPC_SECRET=P3TERX
      - RPC_PORT=6800
      - LISTEN_PORT=6888
      - DISK_CACHE=128M
      - IPV6_MODE=false
      - UPDATE_TRACKERS=true
      - CUSTOM_TRACKER_URL=
      - TZ=Asia/Shanghai
    volumes:
      - ${PWD}/aria2-config:/config
      - ${HOME}/Downloads:/downloads
    network_mode: host
    restart: unless-stopped
    logging:
      driver: json-file
      options:
        max-size: 1m
```

#### Immersive Translate - Translate Web & PDF

**地址：** <https://chromewebstore.google.com/detail/immersive-translate-trans/bpoadfkcbjbfhfodiogcnhhhpibjhbnh?hl=en>

**功能：** 真 - 沉浸式翻译

#### Proxy SwitchyOmega 3 (ZeroOmega)

**地址：** <https://chromewebstore.google.com/detail/proxy-switchyomega-3-zero/pfnededegaaopdmhkdmcofjmoldfiped?hl=en>

**功能：** 处理不同网站之间的代理

**Rule List:**

```txt
https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
```

#### Memo

**地址：** <https://chromewebstore.google.com/detail/memo/chgfencjlhmjhmnnpnlnchglkkdcipii?hl=en>

**功能：** 配合 GitHub 仓库，将搜藏的页面存储到一个仓库中

#### JSON Formatter

**地址：** <https://chromewebstore.google.com/detail/json-formatter/bcjindcccaagfpapjjmafapmmgkkhgoa?hl=en>

**功能：** 格式化 JSON 数据

#### I still don't care about cookies

**地址：** <https://chromewebstore.google.com/detail/i-still-dont-care-about-c/edibdbjcniadpccecjdfdjjppcpchdlm?hl=en>

**功能：** 如题

#### BiliPlus - Bilibili 加大杯，细节从大杯做起

**地址：** <https://chromewebstore.google.com/detail/biliplus-bilibili-%E5%8A%A0%E5%A4%A7%E6%9D%AF%EF%BC%8C%E7%BB%86%E8%8A%82%E4%BB%8E/liddcdcjkpeaiblhebgihpmcaknpgbgk?hl=en>

**功能：** B 站优化

#### B 站空降助手

**地址：** <https://chromewebstore.google.com/detail/b%E7%AB%99%E7%A9%BA%E9%99%8D%E5%8A%A9%E6%89%8B/eaoelafamejbnggahofapllmfhlhajdd?hl=en>

**功能：** B 站优化

### 油猴脚本

#### AC-baidu-google_sogou_bing_RedirectRemove_favicon_adaway_TwoLine

**地址:**<https://greasyfork.org/en/scripts/14178-ac-baidu-%E9%87%8D%E5%AE%9A%E5%90%91%E4%BC%98%E5%8C%96%E7%99%BE%E5%BA%A6%E6%90%9C%E7%8B%97%E8%B0%B7%E6%AD%8C%E5%BF%85%E5%BA%94%E6%90%9C%E7%B4%A2-favicon-%E5%8F%8C%E5%88%97>

#### Endless Google

**地址:**<https://greasyfork.org/en/scripts/41041-endless-google-fork>

#### Auto Skip YouTube Ads

**地址:**<https://greasyfork.org/en/scripts/498197-auto-skip-youtube-ads>

#### Pixiv Plus

**地址：**<https://greasyfork.org/en/scripts/34153-pixiv-plus>

## VS Code 配置

### 配置文件

```json
{
  "editor.cursorBlinking": "solid",
  "editor.cursorStyle": "block",
  "workbench.colorTheme": "Default Light+",
  "editor.fontFamily": "'Hack Nerd Font Mono', Monaco, 'Courier New', monospace",
  "editor.fontSize": 18,
  "editor.fontLigatures": true,
  "editor.tabSize": 2,
  "editor.tabCompletion": "on",
  "editor.renderWhitespace": "all",
  "editor.rulers": [80],
  "editor.suggestSelection": "first",
  "editor.wordWrap": "wordWrapColumn",
  "editor.wordWrapColumn": 120,
  "editor.bracketPairColorization.independentColorPoolPerBracketType": true,
  "window.restoreWindows": "none",
  "files.autoSave": "onFocusChange",
  "files.defaultLanguage": "javascript",
  "files.trimTrailingWhitespace": true,
  "files.exclude": {
    "**/.git": true,
    "**/.svn": true,
    "**/.hg": true,
    "**/CVS": true,
    "**/.DS_Store": true,
    "**/Thumbs.db": true,
    "**/.gitignore": true,
    "**/package-lock.json": true
  },
  "files.eol": "\n",
  "terminal.external.osxExec": "iTerm.app",
  "terminal.integrated.fontSize": 14,
  "terminal.integrated.fontFamily": "Maple Mono NF CN",
  "http.proxy": "http://127.0.0.1:7890",
  "workbench.activityBar.location": "hidden"
}
```

### 插件

#### Error lens

#### Better Comments

#### Bookmarks

#### Code Spell Checker

## 小软件大提升

### Raycast

**官网地址:**
<https://www.raycast.com//>

**安装方式:**

```shell
brew install --cask raycast
```

**核心功能:**
软件启动器

**辅助功能:**
历史剪切板等

### Bark

**官网地址:**
<https://bark.day.app/#/>

**安装方式:**
App Store 直接安装

**核心功能:**
消息提醒，基于 Apple 的推送渠道，可以实现不运行软件就可以接受消息

### KeepingYouAwake

**官网地址:**
<https://keepingyouawake.app/>

**安装方式:**
App Store 直接安装

**核心功能:**
可以保持屏幕常亮。功能单一但好用。完美的遵循 Unix 哲学中的 KISS(Keep it simple, stupid) 原则

### RunCat

**官网地址:**
<https://kyome.io/runcat/?lang=en>

**安装方式:**
App Store 直接安装

**核心功能:**
小猫可爱捏。会在状态栏显示一只奔跑的小猫，系统负载越大，跑得越快。也可以点开看一些系统资源的具体使用信息

### InputSourcePro

**官网地址:**
<https://inputsource.pro/>

**安装方式:**

```shell
brew install --cask input-source-pro
```

**核心功能:**
为不同的程序设置不同的默认打开输入法

## 文件目录定义

使用 kebab-case 风格进行命名。

- \/Document
  - \/lcsk42.github.io
  - \/container
  - \/code
    - \/com.lcsk42
    - \/com.skybeyondtech
  - \/script
    - \/jebra
    - \/navicat-premium-reset-trial

## 开发相关

### Back-end

#### Java

安装 jdk

```shell
brew install jenv openjdk@21 openjdk@17 mvn
```

- jenv: java 版本管理
- jdk：开发必备
- mvn: 包管理

### Front-end

- IntelliJ IDEA: 吃饭的软件
- Navicat: 数据库连接软件
- Wireshark: 网络抓包

## 沟通娱乐

- WeChat： 国内必备
- Telegram：拓宽视野
- IINA：Mac 下最好的本地视频播放器
