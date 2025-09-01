# install

wsl2をインストールして、ディストリビューションはインストールしない
``` sh
wsl --install --no-distribution
```

その後インストール可能なディストリを確認
```
wsl -l -o
```
archが使えるはずなのに表示されない、、、

https://github.com/microsoft/WSL/releases

## archlinux
上記のwslをインストールしたところarchが表示されたので以下でインストール
``` sh
wsl --install archlinux
```

windowsを再起動したような気がする
``` sh
wsl.exe --install archlinux
wsl.exe -d archlinux
```
archなので初回は``pacman -Syu``を忘れずに

ネットワークに接続できないようだ
以下をpowershell(管理者)で実行
``` sh
> Get-NetFirewallHyperVVMCreator
VMCreatorId  : {40E0AC32-46A5-438A-A0B2-2B479E8F2E90}
FriendlyName : WSL

> Get-NetFirewallHyperVVMSetting -Name '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}'
Name                  : {40E0AC32-46A5-438A-A0B2-2B479E8F2E90}
Enabled               : NotConfigured
DefaultInboundAction  : NotConfigured
DefaultOutboundAction : NotConfigured
LoopbackEnabled       : NotConfigured
AllowHostPolicyMerge  : NotConfigured

> Set-NetFirewallHyperVVMSetting -Name '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}' -DefaultInboundAction Allow

> wsl --shutdown
> wsl -d archlinux
```

あと、$HOME/.wslconfigを以下で作成
```ini
[wsl2]
firewall=false
```

今度はsslのself-signedとかでエラーになる
あきらめてubuntuにする

## ubuntu
簡単
``` sh
wsl --install ubuntu
```

``sudo apt update``も実行できた
もしかしたらarchの時にfirewallを無効にしたからかもしれないが検証していない

とりあえず既定のディストリを変更
``` sh
wsl --set-default ubuntu
```

## 共有フォルダ
/etc/fstabを編集する
```
//D3FS1601.jx-nmm.com/document/03.保守・運用業務/02.運用(システム別)/K71.CATS /mnt/share drvfs defaults,uid=1000,gid=1000 0 0
```

## windowsから参照
エクスプローラーで下記を入力する
(ディストリビューション毎に分かれている)
```
\\wsl$
```
## .bashrc
```
export EDITOR=vim
```