v1.0.0: Aug 24, 2017 concise 整理原稿。
v1.1.0: Feb 16, 2022 bypeng fork 至 fast-crypto-lab ，並更新資訊至 ubuntu 20.04 適用。

本篇適用情境： ubuntu 16.04 以上之 LTS 版本，其中 apt-get 於較新的 ubuntu 建議可改用 apt 指令。



===== (系統管理員 only) 伺服器端，安裝套件 =====
[若你沒有 sudo 權限但想要使用 vnc ，請聯絡並溝通系統管理員進行相關處置]

$ sudo apt-get install vnc4server xfce4 xfce4-goodies firefox                    # 適用於 ubuntu 16.04 ~ 18.04
或
$ sudo apt install tigervnc-standalone-server xfce4 xfce4-goodies firefox        # 適用於 ubuntu 20.04



===== (系統管理員 only) 伺服器端，做一些基本 Xfce 設定 =====
[若你沒有 sudo 權限但想要使用 vnc ，請聯絡並溝通系統管理員進行相關處置]

避免第一次進入 XFCE 出現 "Use default config" 和 "One empty panel" 二選一的選擇題
$ sudo ln -s /etc/xdg/xfce4/panel/default.xml /etc/xdg/xfce4/xfconf/xfce-perchannel-xml/xfce4-panel.xml

把 Super 鍵相關的快捷鍵設定清除，避免 Tab 鍵失靈
$ sudo sed -i /Super/d /etc/xdg/xfce4/xfconf/xfce-perchannel-xml/xfce4-keyboard-shortcuts.xml



===== 伺服器端，在你的家目錄下，初次使用時，要先準備一些檔案 =====

$ mkdir -p ~/.vnc

設定 vnc 預設開啟 xfce4 (放 ~/.vnc/xstartup 設定檔)
$ printf '#!/bin/sh\nstartxfce4&\n' > ~/.vnc/xstartup
$ chmod +x ~/.vnc/xstartup

[選用] 設定 vnc 密碼為六個零至 ~/.vnc/passwd 密碼檔
(若不建立該密碼檔，第一次開啟 VNC 會要求你指定一個新密碼，並 hash 之後存於該檔）
$ xxd -p -r <<< e940179b669dc61d > ~/.vnc/passwd
$ chmod 0600 ~/.vnc/passwd



===== 伺服器端，打開 VNC server 並且指定解析度 =====

$ vnc4server -localhost -nolisten tcp -geometry 1280x720         # 適用於 vnc4server ，於 ubuntu 16.04~18.04
或
$ tigervncserver -localhost yes -geometry 1280x720               # 適用於 tigerVNC ，於 ubuntu 20.04

任何可以連到你開的 VNC (TCP) server 且知道密碼的人，都可以存取你的桌面、以你的帳戶做事情。

參數 "-localhost -nolisten tcp" 或 "-localhost yes" 使這個 VNC server 只會
listen for TCP connections on loopback interface。

參數 "-geometry 1280x720" 設定解析度，
如果你的本地端螢幕夠大，你可以設定更高的解析度。

這個指令成功執行以後，會顯示 display number 像是 :1 或 :2 或 :3 之類的

預設情況下

    display :1 corresponds to TCP port 5901
    display :2 corresponds to TCP port 5902
    display :3 corresponds to TCP port 5903

也可以用以下指令去做確認，看看是不是有程序已經打開 TCP server 了

    $ ls ~/.vnc/*.pid
    $ cat ~/.vnc/*.pid
    $ netstat -nlptu

以下假設我們的 VNC display server 回報的是 :1 (即 TCP port 5901)



===== 在本地端使用 SSH 做 TCP port forwarding =====

使我們可以安全地連接到在遠端才看得見的 TCP server

$ ssh yourname@yourserver -N -L 1234:127.0.0.1:5901

See SSH(1) manual for more information
The -L and -R options of ssh are awesome



===== 現在你可以連到本地端 port 1234 的 VNC server 了 =====

使用任何你喜歡的 vnc client

在 macOS 上，你可以用內建的 Screen Sharing 這個 app：

    $ open vnc://127.0.0.1:1234

在 Ubuntu 上，你可以用 xvnc4viewer 或者 tigervnc-viewer ：

    $ sudo apt-get install xvnc4viewer
    $ xvnc4viewer 127.0.0.1:1234
    或
    $ sudo apt install tigervnc-viewer
    $ xtigervncviewer 127.0.0.1:1234

在 Windows 上，你可以用 TigerVNC (以及 PuTTY 的 port forwarding 功能)

    https://github.com/TigerVNC/tigervnc
    https://bintray.com/tigervnc/stable/tigervnc/1.8.0
    vncviewer-1.8.0.exe
        127.0.0.1:1234



===== 伺服器端，當你不再需要這個 VNC server session 時，可以把它殺掉 =====

$ vnc4server -kill :1        # ubuntu 16.04 ~ 18.04
或
$ tigervncserver -kill :1    # ubuntu 20.04



===== XFCE 其他使用筆記 =====

如果本地端是 macOS 作業系統，則按下 command 鍵的效果到遠端會是 alt 鍵

經過實測，在 xfce4-terminal 下可以正常輸入 128 個 7-bit ASCII 字元

用快捷鍵 Alt+F2 以執行檔名稱搜尋並開啟應用程式
用快捷鍵 Alt+F3 以應用程式名搜尋並開啟應用程式

可以去 Applications -> Settings -> Keyboard
在 Application Shortcuts 設定用 Ctrl+Alt+T 開啟 xfce4-terminal

如果希望 Terminal 不要有 Alt+F 等快捷鍵
去 Edit -> Preferences -> Advanced
把 Disable all menu access keys (such as Alt+f) 勾起來

如果希望修改 Terminal 背景色、前景色、十六色
去 Edit -> Preferences -> Colors 做適當的設定
或者可以選擇預先定義好的 Presets
例如 "Dark Pastels" 或 "Solarized (light)" 或 "Black on White"
