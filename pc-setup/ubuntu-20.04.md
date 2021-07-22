# Ubuntu 20.04 セットアップ

PCのセットアップ時の個人的なメモ。

## Chrome

「Chrome ダウンロード」で検索して、ダウンロードしたdebファイルをダブルクリックしてインストール。

## Dropbox

「Dropbox ubuntu インストール」で検索して、ダウンロードしたdebファイルをダブルクリックしてインストール。

### Docker

[docker docs](https://docs.docker.com/engine/install/ubuntu/)を参照

```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

## 開発関連のインストール

### 基本パッケージ

```bash
sudo apt install build-essential
sudo apt install curl
sudo apt install git
```

### Node.js

[n](https://github.com/tj/n)を使ってバージョン管理します。

```bash
# 一旦、node.js npmをインストール
sudo apt install nodejs npm
# nをインストール
sudo npm install -g n
# nを使って最新のNode.jsをインストール
sudo n latest
# aptでインストールしたNode.js、npmはアンインストールする
sudo apt purge nodejs npm
sudo apt autoremove
```

シェルを再起動すること。

### AWS CLI

[AWSのユーザガイド](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2-linux.html)を参照

インストール

```bash
cd /tmp
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

セットアップ

```bash
aws configure
```

### AWS SAM

[AWSのユーザガイド](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install-linux.html)を参照

```bash
curl https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip -o aws-sam-cli-linux-x86_64.zip
cd ~/ダウンロード`
unzip aws-sam-cli-linux-x86_64.zip -d sam-installation
sudo ./sam-installation/install
```


## VSCode

[ダウンロードページ](https://code.visualstudio.com/docs/?dv=linux64_deb)からdebファイルをダウンロードしてインストール

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

### 画面のテーマ変更

1. `M-x package-install [RET] solarized-theme`
2. `M-x load-theme [RET] solarized-dark`


### パッケージのインストール

インストールするパッケージのリスト

* markdown-mode

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

