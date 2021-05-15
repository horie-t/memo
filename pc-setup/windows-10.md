# Windows 10

## Chrome

「Chrome ダウンロード」で検索して、ダウンロードしたexeファイルをダブルクリックしてインストール。

## Dropbox

「Dropbox インストール」で検索して、ダウンロードしたexeファイルをダブルクリックしてインストール。

## WSL2

[Windows 10 用 Windows Subsystem for Linux のインストール ガイド](https://docs.microsoft.com/ja-jp/windows/wsl/install-win10)を参考にインストール

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

## Docker

「Docker Desktop for Windows」で検索して、ダウンロードしたexeファイルをダブルクリックしてインストール。

