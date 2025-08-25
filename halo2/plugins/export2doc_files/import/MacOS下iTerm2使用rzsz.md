# MacOS下iTerm2使用rzsz

# iTerm2使用rzsz

> iTerm2我已安装完毕，配备zsh，唯一就是rzsz命令，每次执行都会报错，且iTerm2会卡死，只能重启，于是去网上搜了一下，发现的确有解决方案，目前通过如下解决方案已经可以正常使用rzsz命令，这里记录一下。

## 安装brew

mac是使用brew作为软件包管理工具，这里我已经安装了，不再赘述。

## 安装lrzsz

使用如下命令：`brew install lrzsz`

这里需要说明一下，等待过程特别长，不知道是网络问题，还是我本机问题，毕竟我是黑苹果。

## 配置rzsz

需要进行如下配置，首先进入目录：`/usr/local/bin` 目录，创建两个文件，分别是：

- iterm2-recv-zmodem.sh
- iterm2-send-zmodem.sh

分别向两个文件里添加配置信息，如下：

1. iterm2-recv-zmodem.sh

```shell
#!/bin/bash
# Author: Matt Mastracci (matthew@mastracci.com)
# AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
# licensed under cc-wiki with attribution required 
# Remainder of script public domain

osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
    FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
else
    FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
fi

if [[ $FILE = "" ]]; then
    echo Cancelled.
    # Send ZModem cancel
    echo -e \\x18\\x18\\x18\\x18\\x18
    sleep 1
    echo
    echo \# Cancelled transfer
else
    cd "$FILE"
    /usr/local/bin/rz -E -e -b
    sleep 1
    echo
    echo
    echo \# Sent \-\> $FILE
fi
```

1. iterm2-send-zmodem.sh

```shell
#!/bin/bash
# Author: Matt Mastracci (matthew@mastracci.com)
# AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
# licensed under cc-wiki with attribution required 
# Remainder of script public domain

osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
    FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
else
    FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
fi
if [[ $FILE = "" ]]; then
    echo Cancelled.
    # Send ZModem cancel
    echo -e \\x18\\x18\\x18\\x18\\x18
    sleep 1
    echo
    echo \# Cancelled transfer
else
    /usr/local/bin/sz "$FILE" -e -b
    sleep 1
    echo
    echo \# Received $FILE
fi
```

配置完成后，需要给这两个文件增加执行权限：`chmod 777 iterm2-*`

## 配置iTerm2

点击 iTerm2 的设置界面 Perferences -> Profiles -> Default -> Advanced -> Triggers 的 Edit 按钮。

参考最后这张图片，配置如下：

```bash
Regular expression: rz waiting to receive.\*\*B0100
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-send-zmodem.sh

Regular expression: \*\*B00000000000000
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
```

ok，至此配置完成，关闭并重启iTerm2即可，可能会有错误log，查看并根据提示的问题解决即可。