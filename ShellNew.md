# 新規作成
エクスプローラーで「右クリック → 新規作成」で作成するファイルはレジストリに設定されている
HKEY_CLASS_ROOT直下の「.拡張子」以下に登録されている
「ShellNew」というキーがそれっぽい。

xlsxについてはコピー元があって、それをコピーしているだけのようだ
レジストリのキーとか設定値とかはwindowsのバージョンによって変わるっぽい
(win10とwin11だと違ってた)

win11の場合
```
HKEY_CLASS_ROOT/.xlsx/Excel.Sheet.12/ShellNew/FileName
```

####  newexcel
``` sh
#!/bin/bash

if [[ $# == 1 && -n $1 ]]; then
    cp "/mnt/c/Program Files/Microsoft Office/root/vfs/Windows/SHELLNEW/EXCEL12.XLSX" $1.xlsx
    attrib.exe -R $1.xlsx
fi
```