---
title: macOS使用指南之Homebrew
date: 2021-08-09 08:20:08
tags:
  - macOS
categories:
comments:
---

# what and why
打开[Homebrew官网](https://brew.sh/)，映入眼帘的第一句话是：

> The Missing Package Manager for macOS (or Linux)

即，**Homebrew是macOS上缺失的包管理器**，就像Linux中的yum、apt等。它让你能用命令行的方式集中的管理你的软件包，而不需要在AppStore和各种软件官网之间反复横跳。搜索、安装、卸载、更新等操作都可以通过命令行来统一完成。

<!-- more -->

# 核心概念

Homebrew官网的logo就是一杯酒，brew的意思是酿造，Homebrew的一些核心概念也都和酿酒有关，像是一个个比喻，大概作者比较喜欢酿酒或喝酒。

## formula(配方)
安装包的描述文件，实际上就是一个ruby文件

## celler(地窖)
所有的安装包都会安装在这个目录下

## keg(桶)
celler的子目录，地窖里有一个个的桶

## bottle(瓶子)
已经编译好的formula

## cask(木桶)
一般来说是有图形化界面的app，比如AppStore里的那些应用

## tap(水龙头)
formula的下载源

# 安装
我安装的时候是用的国内镜像，因为即使开了vpn，官网提供的安装方式也是安装失败的。由于我安装的时候没有记录过程，这篇文章是后来才写的，所以过程就不写了，运行下面的命令，反正跟着提示走就行了。

```
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

# 常用命令

```
# 安装服务类、命令行软件，如git,redis,nginx
brew install [包名]
# 安装带有图形界面的软件
brew install --cask [包名]
# 卸载软件
brew uninstall [包名]
# 搜索
brew search [包名]
# 查看
brew info [包名]
# 打开软件官网
brew home [包名]
# 更新Homebrew自身
brew update
# 查看可升级的软件
brew outdated
# 升级所有软件
brew upgrade
# 升级指定软件
brew upgrade [包名]
# 查看所有已安装的软件
brew list
# 清理所有旧版软件
brew cleanup
# 清理指定软件的旧版
brew cleanup [包名]
# 查看安装源列表
brew tap
# 添加一个新的tap
brew tap [tap名]
# 管理后台服务，如nginx、redis等
## 查看所有已注册的服务
brew services list
## 运行某个服务，并设置开机自动运行
brew services start [服务名]
## 停止某个服务
brew services stop [服务名]
## 重启某个服务
brew services restart [服务名]
## 单次运行某个服务
brew services run [服务名]
```

需要额外说一下的是，在学习Homebrew的过程中，我发现网上的教程给的命令都是`brew cask install`，但我在使用时提示`cask`命令是不存在的，官网提供的也是`--cask`参数的形式。


