設定画面の左下の「JSONファイルを開く」より

``` json
{
    "$help": "https://aka.ms/terminal-documentation",
    "$schema": "https://aka.ms/terminal-profiles-schema",
    "actions": [],
    "copyFormatting": "none",
    "copyOnSelect": false,
    "defaultProfile": "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
    "keybindings": 
    [
        {
            "id": "Terminal.CopyToClipboard",
            "keys": "ctrl+c"
        },
        {
            "id": "Terminal.PasteFromClipboard",
            "keys": "ctrl+v"
        },
        {
            "id": "Terminal.DuplicatePaneAuto",
            "keys": "alt+shift+d"
        }
    ],
    "newTabMenu": 
    [
        {
            "type": "remainingProfiles"
        }
    ],
    "profiles": 
    {
        "defaults": 
        {
            "backgroundImage": "C:\\temp\\daizu2.jpg",
            "backgroundImageAlignment": "bottomRight",
            "backgroundImageOpacity": 0.25,
            "backgroundImageStretchMode": "none",
            "opacity": 88,
            "useAcrylic": false
        },
        "list": 
        [
            {
                "commandline": "%SystemRoot%\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
                "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
                "hidden": false,
                "name": "Windows PowerShell"
            },
            {
                "colorScheme": "Solarized Dark",
                "commandline": "%SystemRoot%\\System32\\cmd.exe",
                "guid": "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
                "hidden": false,
                "name": "\u30b3\u30de\u30f3\u30c9 \u30d7\u30ed\u30f3\u30d7\u30c8"
            },
            {
                "guid": "{b453ae62-4e3d-5e58-b989-0a998ec441b8}",
                "hidden": false,
                "name": "Azure Cloud Shell",
                "source": "Windows.Terminal.Azure"
            },
            {
                "guid": "{a377ce47-1867-5a40-8f95-c8669b940658}",
                "hidden": false,
                "name": "Developer Command Prompt for VS 2017",
                "source": "Windows.Terminal.VisualStudio"
            },
            {
                "guid": "{ce82ba71-6cf4-5e13-9945-6823643f7c76}",
                "hidden": false,
                "name": "archlinux",
                "source": "Microsoft.WSL"
            },
            {
                "guid": "{b4c89672-bde6-597c-8c48-42b0fdd51d2b}",
                "hidden": false,
                "name": "Ubuntu",
                "source": "Microsoft.WSL"
            }
        ]
    },
    "schemes": [],
    "theme": "dark",
    "themes": []
}
```