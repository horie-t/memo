# Ubuntu 20.04 セットアップ

PCのセットアップ時の個人的なメモ。

## Chrome

「Chrome ダウンロード」で検索して、ダウンロードしたdebファイルをダブルクリックしてインストール。

## Dropbox

「Dropbox ubuntu インストール」で検索して、ダウンロードしたdebファイルをダブルクリックしてインストール。

## 開発ツールのインストール

```bash
sudo apt install build-essential
sudo apt install git
```

## Emacs

```bash
sudo apt install emacs
```

### package: パッケージ管理ツール

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

### atomic-chrome パッケージのインストール

[GitHub](https://github.com/alpha22jp/atomic-chrome)

1. Emacsを起動
2. `M-x package-install [RET] atomic-chrome`

`~/.emacs.d/init.el` に以下を追加

```elisp
;;;; atomic-chrome
(require 'atomic-chrome)
(atomic-chrome-start-server)
```

### ブラウザ起動

`~/.emacs.d/init.el` に以下を追加

```elisp
(global-set-key "\C-cj" 'browse-url-at-point)
```

### Mew

[公式サイト](https://www.mew.org/ja/)

ソースコードの取得

```bash
cd ~/ダウンロード
wget https://www.mew.org/Release/mew-6.8.tar.gz
tar -zxf mew-6.8.tar.gz
cd mew-6.8
```

コンパイル・インストール

```bash
./configure 
make
sudo make install
sudo make install-jinfo
```

`~/.emacs.d/init.el` に以下を追加


```elisp
(add-to-list 'load-path "/usr/local/share/emacs/site-lisp/mew")

(autoload 'mew "mew" nil t)
(autoload 'mew-send "mew" nil t)

;; Optional setup (Read Mail menu):
(setq read-mail-command 'mew)

;; Optional setup (e.g. C-xm for sending a message):
(autoload 'mew-user-agent-compose "mew" nil t)
(if (boundp 'mail-user-agent)
    (setq mail-user-agent 'mew-user-agent))
(if (fboundp 'define-mail-user-agent)
    (define-mail-user-agent
      'mew-user-agent
      'mew-user-agent-compose
      'mew-draft-send-message
      'mew-draft-kill
      'mew-send-hook))
```

`~/.mew.el` を[初期設定](https://www.mew.org/ja/info/release/mew_1.html#configuration)を参考に送受信の設定を追加した後で、以下を追加。

```elisp
(setq mew-pop-delete 3) ; 3日前以上のものを削除
(setq mew-pop-size 0)   ; サイズの大きいメールも取得

(setq mew-use-cached-passwd 't) ; 起動中はパスワードを覚える

; 
(setq mew-summary-form '(type (5 date) " " (14 from) " " t (60 subj) "|" (0 body)))

; HTMLのメールをChromeで開けるようにする
(setq mew-prog-text/html-ext
      '("chrome" ("-a" "%s") t))
```
