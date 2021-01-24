---
layout: post
title: "ohmyzsh配置酷炫的mac终端"
categories: tools 
tags: tools ohmyzsh
---


1. 安装term2
略


2. 安装ohmyzsh
安装脚本
    
    sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"

安装错误，参照

    /etc/hosts
    199.232.96.133 raw.github.com

3. 安装完成，设置默认zsh

4. 设置样式,随机theme
.zshrc

    ZSH_THEME=random

5. 下载plugins
git

    git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

4. 设置plugins
.zshrc

    plugins=(
            git
            zsh-syntax-highlighting
            zsh-autosuggestions
    )