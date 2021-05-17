# Windows 10

## Chrome

「Chrome ダウンロード」で検索して、ダウンロードしたexeファイルをダブルクリックしてインストール。

## Dropbox

「Dropbox インストール」で検索して、ダウンロードしたexeファイルをダブルクリックしてインストール。

## WSL2

[Windows 10 用 Windows Subsystem for Linux のインストール ガイド](https://docs.microsoft.com/ja-jp/windows/wsl/install-win10)を参考にインストール

管理者権限で、PowerShellを起動して以下を実行。

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

[Linux カーネル更新プログラム パッケージをダウンロード](https://docs.microsoft.com/ja-jp/windows/wsl/install-win10#step-4---download-the-linux-kernel-update-package)でパッケージをダウンロードする。

```powershell
wsl --set-default-version 2
```

Microsoft StoreでUbuntuをインストールする。

その後の設定は、[Ubuntu 20.04 セットアップ](./ubuntu-20.04.html)を参考にする。

## Docker

「Docker Desktop for Windows」で検索して、ダウンロードしたexeファイルをダブルクリックしてインストール。

Dockerのsetting画面で、Resources - WSL INTEGRATION で Ubuntu との統合をenableにする。

## Windows Terminal

Microsoft Storeでインストールする。

デフォルトをWSL2に設定する。起動後に設定画面を開き、「JSONファイルを開く」をクリックして、jsonファイルを変更する

```json
{
    //(中略)
    //後述のUbuntuのguidを設定する
    "defaultProfile": "{2c4de342-38b7-51cf-b940-2309a097f518}",
    //(中略)
            {
                "guid": "{2c4de342-38b7-51cf-b940-2309a097f518}",
                "hidden": false,
                "name": "Ubuntu",
                "source": "Windows.Terminal.Wsl",
                "commandline" : "wsl.exe ~ -d Ubuntu"  // 起動時にディレクトリをUbutuのHOMEにする
            },
    //(中略)
}
```

## VcXsrv

[VcXsrv](https://sourceforge.net/projects/vcxsrv/)からダウンロードしてインストールする

### 自動起動設定

1. 「スタート」メニューから「VcXsrv」 - 「XLaunch」を選択して起動する。
2. 「Display settings」ダイアログで、「Multiple windows」を選択し、「Display number」は-1にして、「次へ」をクリックする。
3. 「Client startup」ダイアログで、「Clipboard」、「Primary Selection」、「Native opengl」にチェックして、「Additional parameters for VcXsrv」に`-ac`を入力して、「次へ」をクリックする。
4. 「Finish Configration」ダイアログで、「Save Configration」をクリックして、config.xlaunchファイルを保存して、「完了」をクリックする。
5. `「Windows」キー + r`を押して、`shell:startup`と入力して、「OK」をクリックする。
6. スタートアップフォルダが開くので、ここに保存したconfig.xlaunchファイルを移動する。

### WSLの環境設定

WSL上で以下を実行する。

```bash
echo "export DISPLAY=\$(cat /etc/resolv.conf | grep nameserver | awk '{print \$2}'):0.0" >> ~/.bashrc
echo 'export LIBGL_ALWAYS_INDIRECT=1' >> ~/.bashrc
```

### 日本語入力の設定

* [fcitxで作るWSL日本語開発環境](https://qiita.com/dozo/items/97ac6c80f4cd13b84558)
* [WSL の Ubuntu18.04 で日本語入力](https://vogel.at.webry.info/201905/article_6.html)

Windowsフォントのインストール

```bash
sudo apt -y install fontconfig
sudo ln -s /mnt/c/Windows/Fonts /usr/share/fonts/windows
sudo fc-cache -fv
```

fcitxのインストール

```bash
sudo apt -y install fcitx-mozc dbus-x11 x11-xserver-utils
sudo sh -c "sudo dbus-uuidgen > /var/lib/dbus/machine-id"
```

日本語環境のセットアップ

```bash
sudo apt install language-pack-ja
sudo update-locale LANG=ja_JP.UTF8
sudo apt install fonts-takao
```

fcitxの設定

```bash
set -o noclobber
cat << EOS >> .profile
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
export DefaultIMModule=fcitx
if [ $SHLVL = 1 ] ; then
  xset -r 49  > /dev/null 2&>1
  (fcitx-autostart > /dev/null 2&>1 &)
fi
EOS
```

ターミナルを再度開き直す。日本語入力のオンオフのキーを設定する。

```bash
fcitx-config-gtk3
```

GUI上に Mozc があるか確認し、なければ「＋」ボタンを押す。「全体の設定」タブで、"Triger Input Method"で`全角/半角`キーを押して、Zenkakuhankakuを指定します。


日本語入力ができなくなった時は以下を試してみるとよい。

```bash
sudo sh -c "sudo dbus-uuidgen > /var/lib/dbus/machine-id"
fcitx-autostart > /dev/null 2&>1 &
```

## JDK



## sbt

## IntelliJ

## Vivado
