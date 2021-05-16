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
