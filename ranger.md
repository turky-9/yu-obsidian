# install
``` sh
sudo apt update
sudo apt install ranger
```
ネットワーク上の共有フォルダはクソ遅い
共有フォルダはwindowsのexploereを使用した方が良いっぽい

## config
```sh
# ~/.config/rangerに設定ファイルが作成される
ranger --copy-config=all
```

rc.conf
```
set column_ratios 1
set preview_files false
set preview_directories false
set draw_borders both
set colorscheme solarized
set show_hidden true
```

## command
途中まで入力してtabキーで補完してくれる

| key or command | desc                     |
| -------------- | ------------------------ |
| gn or :tab-new | タブを新規作成                  |
| gt or gT       | タブを移動                    |
| ~(チルダ)         | タブを1画面にまとめて表示 or タブ表示に戻す |
| m + any         | 現在のディレクトリをbookmark  |
| '' + any         | bookmarkに移動  |

## windowsアプリで開く
$HOME/binとか作成して以下のスクリプトを作成して
パスを通しておく(.bashrcとか)

```sh
#!/bin/bash

# open-pdf (only windows)
function open_pdf {
        current_dir=$(pwd)
        if [[ "$1" != "" ]]; then
                file_name="$1"
                if [[ "$file_name" == *"/"* ]]; then
                        IFS='/' read -ra segments <<<"$file_name"
                        file_name=${segments[-1]}
                fi
                file_path_win=$(wslpath -w "$current_dir/$file_name")
                #echo $file_path_win
                cmd.exe /c start "$file_path_win"
        fi
}
open_pdf "$@"
```

オリジナルの``rifle.conf``をリネームするとかでバックアップして
下記内容で新規作成する
```
ext x?pdf?, label editor = ~/bin/opener-win.sh "$(basename -- "$@")"
ext x?xls?, label editor = ~/bin/opener-win.sh "$(basename -- "$@")"
ext x?xlsx?, label editor = ~/bin/opener-win.sh "$(basename -- "$@")"
```
拡張子にマッチしたら=の右辺のコマンドを実行するという意味
上記ではpdf, xls, xlsxの3種を登録しているが
必要に応じて増やせば良い

何にもマッチしない場合の設定
マッチしない場合はwindowsのgvimで開くようにする
```
else, flag f, label gvim  = "gvim-win" -- $(wslpath -w "$@")
```
gvim-winはwindows側のgvimへのシンボリックリンク
```
ls -l gvim-win
lrwxrwxrwx 1 uenoyoshiharuhi9 uenoyoshiharuhi9   54 Jul 30 09:26 gvim-win -> /mnt/c/ueno/prog/vim74-kaoriya-win64-20160409/gvim.exe*
```
### explorer
rc.confに以下の行を追加する
``e``でエクスプローラーが起動する
```
map e shell explorer.exe .
```