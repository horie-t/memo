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
