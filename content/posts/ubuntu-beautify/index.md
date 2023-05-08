---
title: "Ubuntu 16.04 折腾"
date: "2018-02-14T20:34:00+08:00"
categories:
- 折腾
tags:
- Ubuntu
---

1. 卸载无用软件

    ```sh
    sudo apt-get autoremove libreoffice-common
    sudo apt-get autoremove unity-webapps-common
    sudo apt-get autoremove thunderbird totem rhythmbox
    sudo apt-get autoremove simple-scan gnome-mahjongg aisleriot
    sudo apt-get autoremove gnome-mines cheese transmission-common gnome-orca webbrowser-app gnome-sudoku
    sudo apt-get autoremove onboard deja-dup
    ```

2. 安装一些软件包

    ```sh
    sudo apt-get install git vim curl
    ```

    - vim

        vim 配置文件 `~/.vimrc`

        ```
        set nocompatible
        filetype off 
        syntax enable

        " Others
        let g:rainbow_active = 1 
        set laststatus=2
        set fileformats=unix,dos
        set nowrap
        set nobackup
        set nu
        set ts=4
        set sw=4
        set expandtab
        set autoindent
        set pastetoggle=<f3>
        au FileType python let b:delimitMate_nesting_quotes = ['"']
        ```

        `set pastetoggle=<f3>` 设置 `F3` 切换粘贴模式，粘贴文本时防止自动缩进

        <!-- more -->

3. 安装 zsh 和 oh-my-zsh

    ```sh
    sudo apt-get install zsh
    wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh
    chmod +x install.sh
    ./install.sh
    ```

4. 安装与配置 Shadowsocks 客户端

    - 安装 Shadowsocks

        ```sh
        sudo apt-get install python-pip
        sudo pip install shadowsocks
        ```

        配置文件

        ```json
        {
            "server": "server_ip",
            "server_port": 12345,
            "local_address": "127.0.0.1",
            "local_port": 1080,
            "password": "mypassword",
            "method":"chacha20"
        }
        ```

    - 安装 libsodium

        如果使用 `chacha20` 加密方式需要安装这个软件包

        ```sh
        sudo apt-get install libsodium-dev
        ```

    - 让系统管理 sslocal 的启动与停止

        systemd service 文件

        ```
        [Unit]
        Description=Shadowsocks Client
        Wants=network-online.target
        After=network.target

        [Service]
        Type=simple
        ExecStart=/usr/local/bin/sslocal -c /etc/shadowsocks/client.json

        [Install]
        WantedBy=multi-user.target
        ```

        将上述文件复制到 `/etc/systemd/system/sslocal.service`

        ```sh
        sudo systemctl daemon-reload
        sudo systemctl enable sslocal.service # 开机自启
        ```

    - 安装 proxychains

        proxychains 可以让命令行运行的程序走代理

        ```sh
        sudo apt-get install proxychains
        ```

        编辑配置文件

        ```config
        # ...
        [ProxyList]
        # add proxy here ...
        # meanwile
        # defaults set to "tor"
        socks5  127.0.0.1 1080
        ```

        测试代理

        ```sh
        proxychains curl http://ipinfo.io/ip
        ```

    - 配置系统代理

        使用 pac 文件实现非全局代理

        ```sh
        sudo pip install genpac

        genpac --proxy="SOCKS5 127.0.0.1:1080" --gfwlist-proxy="SOCKS5 127.0.0.1:1080" -o pac.txt --gfwlist-url="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"
        ```

        进入系统设置，将系统代理方法改为 Automatic (自动)

        Configuration URL 指向生成的 pac 文件

5. 美化

    - 安装 Unity Tweak Tool

        借助这个工具更改主题、图标等等

        ```sh
        sudo apt-get install unity-tweak-tool
        ```

    - 更改主题

        - flatabulous-theme

            ```sh
            sudo add-apt-repository ppa:noobslab/themes
            sudo apt-get update
            sudo apt-get install flatabulous-theme
            ```

            ![flatabulous-theme][1]

        - numix-gtk-theme

            ```sh
            sudo add-apt-repository ppa:numix/ppa
            sudo apt-get update
            sudo apt-get install numix-gtk-theme
            # 圆角图标
            sudo apt-get install numix-icon-theme-circle
            ```

            ![numix-gtk-theme][2]


[1]: ubuntu-beautify-00.png
[2]: ubuntu-beautify-01.png