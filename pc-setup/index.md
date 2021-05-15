# PCセットアップ

PCのセットアップ時の個人的なメモ。

## Ubuntu 20.04

### Chrome

「Chrome ダウンロード」で検索して、ダウンロードしたdebファイルをダブルクリックしてインストール。

### Dropbox

「Dropbox ubuntu インストール」で検索して、ダウンロードしたdebファイルをダブルクリックしてインストール。

### Emacs

```bash
sudo apt install emacs
```

#### package: パッケージ管理ツール

[説明](https://emacs-jp.github.io/packages/package)

`~/.emacs.d/init.el` に以下を追加

```elisp
;;;; パッケージ管理
(require 'package)

;; package-archivesを上書き
(setq package-archives
      '(;; ("melpa" . "https://melpa.org/packages/")
        ("melpa-stable" . "https://stable.melpa.org/packages/")
        ("org" . "https://orgmode.org/elpa/")
        ("gnu" . "https://elpa.gnu.org/packages/")))

;; 初期化
(package-initialize)
```

再起動するか、Emacsで`M-x package-refresh-contents`を実行する。

#### atomic-chrome パッケージのインストール

[GitHub](https://github.com/alpha22jp/atomic-chrome)

1. Emacsを起動
2. `M-x package-install [RET] atomic-chrome`

`~/.emacs.d/init.el` に以下を追加

```elisp
;;;; atomic-chrome
(require 'atomic-chrome)
(atomic-chrome-start-server)
```

#### Mew

```bash
cd ~/ダウンロード
wget https://www.mew.org/Release/mew-6.8.tar.gz
tar -zxf mew-6.8.tar.gz
cd mew-6.8
```



## Windows 10
