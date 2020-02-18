# 윈도우 개발환경 설정

### 윈도우 설정

1. 제어판에서 hyper-v, WSL 설정 켜기
2. WSL ubuntu 설치
3. [windows terminal](https://www.microsoft.com/store/productId/9N0DX20HK701) 설치
4. [vscode](https://code.visualstudio.com/) 설치
5. [D2Coding](https://github.com/naver/d2codingfont) 폰트 설치
6. [Docker for windows](https://docs.docker.com/docker-for-windows/install/) 설치
7. [Notion](https://www.notion.so/desktop) 설치

### Windows terminal config

```javascript
{
    "$schema": "https://aka.ms/terminal-profiles-schema",
    "globals" : 
    {
        "alwaysShowTabs" : true,
        "defaultProfile" : "{c6eaf9f4-32a7-5fdc-b5cf-066e8a4b1e40}",
        "initialCols" : 120,
        "initialRows" : 30,
        "keybindings" : 
        [
            {
                "command" : "closeTab",
                "keys" : 
                [
                    "ctrl+w"
                ]
            },
            {
                "command" : "newTab",
                "keys" : 
                [
                    "ctrl+t"
                ]
            },
            {
                "command" : "newTabProfile0",
                "keys" : 
                [
                    "ctrl+shift+1"
                ]
            },
            {
                "command" : "newTabProfile1",
                "keys" : 
                [
                    "ctrl+shift+2"
                ]
            },
            {
                "command" : "newTabProfile2",
                "keys" : 
                [
                    "ctrl+shift+3"
                ]
            },
            {
                "command" : "newTabProfile3",
                "keys" : 
                [
                    "ctrl+shift+4"
                ]
            },
            {
                "command" : "newTabProfile4",
                "keys" : 
                [
                    "ctrl+shift+5"
                ]
            },
            {
                "command" : "newTabProfile5",
                "keys" : 
                [
                    "ctrl+shift+6"
                ]
            },
            {
                "command" : "newTabProfile6",
                "keys" : 
                [
                    "ctrl+shift+7"
                ]
            },
            {
                "command" : "newTabProfile7",
                "keys" : 
                [
                    "ctrl+shift+8"
                ]
            },
            {
                "command" : "newTabProfile8",
                "keys" : 
                [
                    "ctrl+shift+9"
                ]
            },
            {
                "command" : "nextTab",
                "keys" : 
                [
                    "ctrl+tab"
                ]
            },
            {
                "command" : "openSettings",
                "keys" : 
                [
                    "ctrl+,"
                ]
            },
            {
                "command" : "prevTab",
                "keys" : 
                [
                    "ctrl+shift+tab"
                ]
            },
            {
                "command" : "scrollDown",
                "keys" : 
                [
                    "ctrl+shift+down"
                ]
            },
            {
                "command" : "scrollDownPage",
                "keys" : 
                [
                    "ctrl+shift+pgdn"
                ]
            },
            {
                "command" : "scrollUp",
                "keys" : 
                [
                    "ctrl+shift+up"
                ]
            },
            {
                "command" : "scrollUpPage",
                "keys" : 
                [
                    "ctrl+shift+pgup"
                ]
            },
            {
                "command" : "switchToTab0",
                "keys" : 
                [
                    "alt+1"
                ]
            },
            {
                "command" : "switchToTab1",
                "keys" : 
                [
                    "alt+2"
                ]
            },
            {
                "command" : "switchToTab2",
                "keys" : 
                [
                    "alt+3"
                ]
            },
            {
                "command" : "switchToTab3",
                "keys" : 
                [
                    "alt+4"
                ]
            },
            {
                "command" : "switchToTab4",
                "keys" : 
                [
                    "alt+5"
                ]
            },
            {
                "command" : "switchToTab5",
                "keys" : 
                [
                    "alt+6"
                ]
            },
            {
                "command" : "switchToTab6",
                "keys" : 
                [
                    "alt+7"
                ]
            },
            {
                "command" : "switchToTab7",
                "keys" : 
                [
                    "alt+8"
                ]
            },
            {
                "command" : "switchToTab8",
                "keys" : 
                [
                    "alt+9"
                ]
            }
        ],
        "requestedTheme" : "system",
        "showTabsInTitlebar" : true,
        "showTerminalTitleInTitlebar" : true
    },
    "profiles" : 
    [
        {
            "guid": "{c6eaf9f4-32a7-5fdc-b5cf-066e8a4b1e40}",
            "name": "Ubuntu-18.04",

            "acrylicOpacity" : 0.5,
            "closeOnExit" : true,
            "colorScheme" : "Dracula",
            "commandline" : "wsl.exe -d Ubuntu-18.04",
            "cursorColor" : "#FFFFFF",
            "cursorShape" : "bar",
            "fontFace" : "D2Coding",
            "fontSize" : 12,
            "historySize" : 150000,
            "icon" : "ms-appx:///ProfileIcons/{9acb9455-ca41-5af7-950f-6bca1bc9722f}.png",
            "padding" : "5, 5, 0, 5",
            "snapOnInput" : true,
            "useAcrylic" : false 
        },        
        {
            "acrylicOpacity" : 0.5,
            "background" : "#012456",
            "closeOnExit" : true,
            "colorScheme" : "Dracula",
            "commandline" : "powershell.exe",
            "cursorColor" : "#FFFFFF",
            "cursorShape" : "bar",
            "fontFace" : "D2Coding",
            "fontSize" : 12,
            "guid" : "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
            "historySize" : 9001,
            "icon" : "ms-appx:///ProfileIcons/{61c54bbd-c2c6-5271-96e7-009a87ff44bf}.png",
            "name" : "Windows PowerShell",
            "padding" : "5, 5, 0, 5",
            "snapOnInput" : true,
            "startingDirectory" : "%USERPROFILE%",
            "useAcrylic" : false
        },
        {
            "acrylicOpacity" : 0.75,
            "closeOnExit" : true,
            "colorScheme" : "Dracula",
            "commandline" : "cmd.exe",
            "cursorColor" : "#FFFFFF",
            "cursorShape" : "bar",
            "fontFace" : "D2Coding",
            "fontSize" : 12,
            "guid" : "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
            "historySize" : 9001,
            "icon" : "ms-appx:///ProfileIcons/{0caa0dad-35be-5f56-a8ff-afceeeaa6101}.png",
            "name" : "cmd",
            "padding" : "5, 5, 0, 5",
            "snapOnInput" : true,
            "startingDirectory" : "%USERPROFILE%",
            "useAcrylic" : true
        },
        {
            "guid": "{b453ae62-4e3d-5e58-b989-0a998ec441b8}",
            "hidden": false,
            "name": "Azure Cloud Shell",
            "source": "Windows.Terminal.Azure"
        }
    ],
    "schemes" : 
    [
        {
            "background" : "#0C0C0C",
            "black" : "#0C0C0C",
            "blue" : "#0037DA",
            "brightBlack" : "#767676",
            "brightBlue" : "#3B78FF",
            "brightCyan" : "#61D6D6",
            "brightGreen" : "#16C60C",
            "brightPurple" : "#B4009E",
            "brightRed" : "#E74856",
            "brightWhite" : "#F2F2F2",
            "brightYellow" : "#F9F1A5",
            "cyan" : "#3A96DD",
            "foreground" : "#CCCCCC",
            "green" : "#13A10E",
            "name" : "Campbell",
            "purple" : "#881798",
            "red" : "#C50F1F",
            "white" : "#CCCCCC",
            "yellow" : "#C19C00"
        },
        {
            "background" : "#282C34",
            "black" : "#282C34",
            "blue" : "#61AFEF",
            "brightBlack" : "#5A6374",
            "brightBlue" : "#61AFEF",
            "brightCyan" : "#56B6C2",
            "brightGreen" : "#98C379",
            "brightPurple" : "#C678DD",
            "brightRed" : "#E06C75",
            "brightWhite" : "#DCDFE4",
            "brightYellow" : "#E5C07B",
            "cyan" : "#56B6C2",
            "foreground" : "#DCDFE4",
            "green" : "#98C379",
            "name" : "One Half Dark",
            "purple" : "#C678DD",
            "red" : "#E06C75",
            "white" : "#DCDFE4",
            "yellow" : "#E5C07B"
        },
        {
            "background" : "#FAFAFA",
            "black" : "#383A42",
            "blue" : "#0184BC",
            "brightBlack" : "#4F525D",
            "brightBlue" : "#61AFEF",
            "brightCyan" : "#56B5C1",
            "brightGreen" : "#98C379",
            "brightPurple" : "#C577DD",
            "brightRed" : "#DF6C75",
            "brightWhite" : "#FFFFFF",
            "brightYellow" : "#E4C07A",
            "cyan" : "#0997B3",
            "foreground" : "#383A42",
            "green" : "#50A14F",
            "name" : "One Half Light",
            "purple" : "#A626A4",
            "red" : "#E45649",
            "white" : "#FAFAFA",
            "yellow" : "#C18301"
        },
        {
            "background" : "#002B36",
            "black" : "#073642",
            "blue" : "#268BD2",
            "brightBlack" : "#002B36",
            "brightBlue" : "#839496",
            "brightCyan" : "#93A1A1",
            "brightGreen" : "#586E75",
            "brightPurple" : "#6C71C4",
            "brightRed" : "#CB4B16",
            "brightWhite" : "#FDF6E3",
            "brightYellow" : "#657B83",
            "cyan" : "#2AA198",
            "foreground" : "#839496",
            "green" : "#859900",
            "name" : "Solarized Dark",
            "purple" : "#D33682",
            "red" : "#DC322F",
            "white" : "#EEE8D5",
            "yellow" : "#B58900"
        },
        {
            "background" : "#FDF6E3",
            "black" : "#073642",
            "blue" : "#268BD2",
            "brightBlack" : "#002B36",
            "brightBlue" : "#839496",
            "brightCyan" : "#93A1A1",
            "brightGreen" : "#586E75",
            "brightPurple" : "#6C71C4",
            "brightRed" : "#CB4B16",
            "brightWhite" : "#FDF6E3",
            "brightYellow" : "#657B83",
            "cyan" : "#2AA198",
            "foreground" : "#657B83",
            "green" : "#859900",
            "name" : "Solarized Light",
            "purple" : "#D33682",
            "red" : "#DC322F",
            "white" : "#EEE8D5",
            "yellow" : "#B58900"
        },
        {
            "background" : "#282A36",
            "black" : "#21222C",
            "blue" : "#BD93F9",
            "brightBlack" : "#6272A4",
            "brightBlue" : "#D6ACFF",
            "brightCyan" : "#A4FFFF",
            "brightGreen" : "#69FF94",
            "brightPurple" : "#FF92DF",
            "brightRed" : "#FF6E6E",
            "brightWhite" : "#FFFFFF",
            "brightYellow" : "#FFFFA5",
            "cyan" : "#8BE9FD",
            "foreground" : "#F8F8F2",
            "green" : "#50FA7B",
            "name" : "Dracula",
            "purple" : "#FF79C6",
            "red" : "#FF5555",
            "white" : "#F8F8F2",
            "yellow" : "#F1FA8C"
        }
    ]
}
```

### WSL Ubuntu 설정

#### /etc/wsl.conf

```text
[automount]
enabled = true
root = /
```

#### /etc/passwd

유저 목록에서 현재 계정 찾은 다음 home path를 `/c/Users/{user name}` 으로 수정

그리고 위 두가지 수정이 모두 끝나면 admin 권한으로 실행한 PowerShell에서 `Restart-Service LxssManager` 실행하고 WSL 재시작

#### ZSH 세팅

1. [oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh) 설치
2. zsh plugin 설치
   1. [powerline10k](https://github.com/romkatv/powerlevel10k)
   2. [https://github.com/zsh-users/zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)
   3. [https://github.com/zsh-users/zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)





