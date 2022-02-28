---
title: Mac 环境搭建及实用软件
date: 2022-01-31 21:49:15
update: 2022-02-22 15:07:41
categories: 杂记
tags:
  - Mac
  - 环境配置
description: 关于 Mac 上开发环境的配置和一些实用的软件推荐！
cover: https://help.apple.com/assets/605932B4A1B7A93F492858E8/605932C0A1B7A93F492858FF/zh_CN/a597502c5c9a69569810be3414c5db14.png
typora-copy-images-to: upload
---

# 1. 开发环境搭建

## 1.1 配置终端

新的 macOS 已经默认使用 zsh 作为 shell。查看当前可用 shells：

```shell
cat /etc/shells
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.

/bin/bash
/bin/csh
/bin/dash
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```

切换 shell：

```shell
chsh -s /bin/zsh
```

## 1.2 Homebrew

> Homebrew 是一款自由及开放源代码的软件包管理系统，用以简化 macOS 系统上的软件安装过程

Homebrew，Mac 上做开发必备神器，不多解释。

[The Missing Package Manager for macOS (or Linux) — Homebrew](https://brew.sh/)

### 安装

安装命令：

```shell
# 安装依赖工具
xcode-select --install

sh -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

注意，国内安装 Homebrew 因为 GFW 的原因会安装失败或安装的很慢，可以使用国内的镜像进行安装，下面以使用国内清华的镜像源为例：

```shell
git clone --depth=1 https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/install.git brew-install
/bin/bash brew-install/install.sh
rm -rf brew-install
```

### 换源

同样因为 GFW 的原因，使用/更新 Homebrew 会很慢（或者报 443 等网络错误），这时我们需要对 Homebrew 的源进行替换，使用国内的镜像提升访问速度。

**查看当前 Homebrew 源：**

```shell
cd "$(brew --repository)" && git remote -v # Homebrew 源代码仓库
# 例如我现在使用的是中科大的镜像
origin	https://mirrors.ustc.edu.cn/brew.git (fetch)
origin	https://mirrors.ustc.edu.cn/brew.git (push)

cd "$(brew --repository homebrew/core)" && git remote -v # Homebrew 核心软件仓库
origin	https://mirrors.ustc.edu.cn/homebrew-core.git (fetch)
origin	https://mirrors.ustc.edu.cn/homebrew-core.git (push)

cd "$(brew --repository homebrew/cask)" && git remote -v #提供 macOS 应用和大型二进制文件 
origin	https://mirrors.ustc.edu.cn/homebrew-cask.git (fetch)
origin	https://mirrors.ustc.edu.cn/homebrew-cask.git (push)
```

**使用国内镜像替换：**

以清华镜像为例：

```shell
echo 'export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"' >> ~/.zshrc
echo 'export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"' >> ~/.zshrc
source ~/.zshrc
for tap in core cask{,-fonts,-drivers,-versions} command-not-found; do
    brew tap --custom-remote --force-auto-update "homebrew/${tap}" "https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-${tap}.git"
done
brew update
```

[homebrew | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/)

[Homebrew 源使用帮助 — USTC Mirror Help 文档](https://mirrors.ustc.edu.cn/help/brew.git.html)

### 常用命令

```shell
brew help #帮助
brew update #更新 Homebrew
brew search [关键词] #搜索相关包
brew info [包名] #查看包的信息
brew list #查看已安装的包
brew upgrade [包名] #更新包，不带包名则更新所有已安装的包
brew cleanup #清理缓存和所有软件的旧版
brew uninstall [包名] #卸载包
brew services list #查看所有服务
brew services run [服务名] #单次运行某个服务
brew services start [服务名] #运行某个服务，并设置开机自动运行。
brew services stop [服务名] #停止某个服务
brew services restart [服务名] #重启某个服务
```

### 配置迁移

如果更换电脑需要迁移 Homebrew 环境，通过 Homebrew Bundle 工具可以快速实现：

```shell
brew bundle dump
```

执行命令后可以得到一个 `Brewfile` ，复制到新电脑上并执行命令即可安装：

```shell
brew bundle
```

## 1.3 iTerm2

> iTerm2 is a replacement for Terminal and the successor to iTerm.

iTerm2，替换原生终端的利器。

[iTerm2 - macOS Terminal Replacement](https://iterm2.com/index.html)

### 安装

1. 通过官方网站可以下载 zip 包：

   [Downloads - iTerm2 - macOS Terminal Replacement](https://iterm2.com/downloads.html)

   解压后复制到 Application 目录下即可

2. 或者通过 brew 安装：

   ```shell
   brew install iterm2
   ```

3. 通过命令直接安装：

   ```shell
   sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)" # github 源
   sh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)" # 国内源
   ```

## 1.4 oh-my-zsh

> Oh My Zsh is a delightful, open source, community-driven framework for managing your Zsh configuration

oh-my-zsh，对 zsh 的进一步扩展，**神器！**

[Oh My Zsh - a delightful & open source framework for Zsh](https://ohmyz.sh/)

[ohmyzsh/ohmyzsh: 🙃 A delightful community-driven  (github.com)](https://github.com/ohmyzsh/ohmyzsh)

### 安装

```shell
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 配置

用户目录下的 `.zshrc ` 文件为 oh-my-zsh 的配置文件：`~/.zshrc`

#### 主题

Oh-my-zsh 自带了很多主题，都在 `~/.oh-my-zsh/themes` 目录下，预览这些主题可以看：[Themes · ohmyzsh/ohmyzsh Wiki (github.com)](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)，如果没有想要的主题只需将想安装的主题放置在上述目录中即可。

想要选择要使用的主题有两种方法：

1. 修改配置文件

   ```shell
   vim ~/.zshrc
   ```

   找到 `ZSH_THEME="robbyrussell"` 将主题换位你想用的主题就可以，例如我的是 `ZSH_THEME="fino-time"`，效果如下：

   ![image-20220221205904858](https://s2.loli.net/2022/02/21/vQzMWJ2Hi61UBIx.png)

2. 命令

   ```shell
   omz theme set 主题名
   ```

   效果如下：

   ![image-20220221210034404](https://s2.loli.net/2022/02/21/UNvQqu4gSI6c8JT.png)

想要随机主题的设置为：`ZSH_THEME="random"` 即可，如果想要在某几个主题中随机，启用如下设置：`ZSH_THEME_RANDOM_CANDIDATES=( "robbyrussell" "agnoster" )`

#### 插件

oh-my-zsh 支持许多好用的插件，它本身自带了很多插件，也可以再 github 上找找看。

oh-my-zsh 自带的插件都在 `~/.oh-my-zsh/plugins` 目录下，和主题一样，想要自行安装插件，只需放置在此目录再启用即可

**启用插件**

1. 修改配置文件中下行：

   ```zshrc
   plugins=(git zsh-syntax-highlighting autojump z)
   ```

   括号中为想要启用的插件名

2. 或者使用命令：

   ```shell
   omz plugin enable/disable
   ```

##### autojump

目录切换神器，这个好像 oh-my-zsh 已经自带了，安装命令：

```shell
brew install autojump
```

使用实例：

```shell
# 首次进入某个目录
cd Documents/personalFile/blog
# 切换至其他目录
cd ~
# 在此进入该目录使用 j 即可
j blog
```

![image-20220221212120303](https://s2.loli.net/2022/02/21/zEb2gFe7r5Ryktf.png)

#### zsh-syntax-highlighting

输入正确命令会以绿色显示，有效检查命令语法，这个好像也自带了，安装命令：

```bash
cd ~/.oh-my-zsh/plugins
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

## 1.3 Vim

Vim 不多介绍，用 brew 安装命令如下：

```shell
brew install vim
```

配置 vim 的行号和主题色：

```shell
# 显示行号
echo 'set nu' >> ~/.vimrc
# 颜色显示方案
echo 'colorscheme desert' >> ~/.vimrc
# 打开语法高亮
echo 'syntax on' >> ~/.vimrc

# 或者只用这一条命令即可
echo 'set nu\ncolorscheme desert\nsyntax on' >> ~/.vimrc
```

vim 自带的配色方案在此目录下：`/usr/share/vim/vim*/colors`

```shell
ls /usr/share/vim/vim82/colors
```

## 1.5 git

git 其实没啥好说的，Mac 也自带，主要说一下 git 设置全局用户名和邮箱：

```shell
git config --global user.email "xxx@xxx.com" #全局设置邮箱
git config --global user.name "xxx" #全局设置用户名
```

想要在某个 repo 下单独设置去掉 `--global` 即可

如果 git 用户名和邮箱配对在 git 里会正确显示你的账户对应的头像等信息

## 1.6 jdk

**这里主要介绍配置多版本 jdk 环境**

### 安装 jdk

1. 通过 brew 可以快速配置你机器上的 jdk：

   ```shell
   brew search openjdk # 会有多个 jdk 版本可以选择
   brew install openjdk # 选择其中一个安装即可
   ```

2. 下载安装包

   [Java Downloads | Oracle](https://www.oracle.com/java/technologies/downloads/)

配置几个可以快速切换 jdk 版本的 alias，在`~/.zshrc ` 中添加如下 alias：

```shell
alias openjdk11='export JAVA_HOME=/usr/local/opt/openjdk@11/libexec/openjdk.jdk/Contents/Home'
alias openjdk8='export JAVA_HOME=/usr/local/opt/openjdk@8/libexec/openjdk.jdk/Contents/Home'
alias openjdk16='export JAVA_HOME=/Users/d3j/Library/Java/JavaVirtualMachines/openjdk-16.0.2/Contents/Home'
```

~~这里我想通过 brew 安装 openjdk8 在设置 alias 一直失败，需要搞一下。~~

同理配置 jdk16 或其他版本，使用效果如下：

![image-20220222150059650](https://s2.loli.net/2022/02/22/leg4fPEiY9aKJHG.png)

# 2. 实用软件

## 2.1 retangle

[Rectangle (rectangleapp.com)](https://rectangleapp.com/)

一款 macOS 上的分屏软件，不同于系统自带的左右分屏需要进入全屏模式，这个仍然是窗口模式，只是把软件界面的大小改为占据屏幕大小的一部分，效果如下：

![image-20220221224651574](https://s2.loli.net/2022/02/21/BkHjQPnVJcWY4CD.png)

**安装**

```shell
brew install --cask retangle
```

**快捷键**

![image-20220221224827808](https://s2.loli.net/2022/02/21/ekHRAQYFXafxToK.png)

