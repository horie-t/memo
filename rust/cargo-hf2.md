# cargo-hf2

[原文](https://crates.io/crates/cargo-hf2)の翻訳です。(ver. 0.3.1)

hf2 flashing over hid プロトコルを使用して接続された uf2 デバイスへの USB 経由のフラッシュを含むように cargo build コマンドを置き換えます。

## 前提条件

libusbを使用するhidapi-sysクレートを利用します。

## linux

ディストリビューションによっては、libusbが必要になるので、以下の通りインストールしてください。
```bash
sudo apt-get install libudev-dev libusb-1.0-0-dev
```

sudo を使いたくない場合は、udev ルールが必要です。ボードが接続され、ブートローダモードになっている状態で、lsusb を使って vendorid を確認してください（ここでは 239a となっています）。

```
Bus 001 Device 087: ID 239a:001b Adafruit Industries Feather M0
```

そして、ベンダーIDを下に入れ、/etc/udev/rules.d/99-adafruit-boards.rulesに以下のように保存してください。

```
ATTRS{idVendor}=="239a", ENV{ID_MM_DEVICE_IGNORE}="1" SUBSYSTEM=="usb", ATTRS{idVendor}=="239a", MODE="0666" SUBSYSTEM=="tty", ATTRS{idVendor}=="239a", MODE="0666"
```

その後、再起動します。

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

## mac

Macでは、Catalinaの時点でアクセス権に関するプロンプトが表示され、指示に従ってターミナルアプリケーションの「入力監視」を許可する必要があります。

## インストール

```
cargo install cargo-hf2
```

## 使用法

ファームウェアのディレクトリで、通常のcargoビルドコマンドの --example と --release が実行でき、hf2サブコマンドがbuildの代わりです。ビルドが成功すると、ハードコードされたホワイトリストを使ってUSBデバイスをオープンし、ファイルをコピーします。

```bash
$ cargo hf2 --example ferris_img --release --pid 0x003d --vid 0x239a
    Finished release [optimized + debuginfo] target(s) in 0.28s
    Flashing "./target/thumbv7em-none-eabihf/release/examples/ferris_img"
Success
    Finished in 0.037s
```

オプションとして、pidとvidを省略することができます。そうすると、bininfoパケットですべてのhidデバイスへの問い合わせを試み、最初に応答したデバイスに書き込みます。

```bash
$ cargo hf2 --example ferris_img --release
    Finished release [optimized + debuginfo] target(s) in 0.24s
no vid/pid provided..
trying "" "Apple Internal Keyboard / Trackpad"
trying "Adafruit Industries" "PyGamer"
    Flashing "./target/thumbv7em-none-eabihf/release/examples/ferris_img"
Success
    Finished in 0.034s
```

デバイスが見つからない場合は、デバイスがブートローダモードになっていることを確認してください。PyGamerでは、2つのボタンを押すと、PyGamerと書かれた青と緑の画面が表示されます。

```
$ cargo hf2 --example ferris_img --release
    Finished release [optimized + debuginfo] target(s) in 0.20s
no vid/pid provided..
trying "" "Apple Internal Keyboard / Trackpad"
trying "" "Keyboard Backlight"
trying "" "Apple Internal Keyboard / Trackpad"
trying "" "Apple Internal Keyboard / Trackpad"
thread 'main' panicked at 'Are you sure device is plugged in and in bootloader mode?', src/libcore/option.rs:1166:5
```

## トラブルシューティング

デバイスが見つからない場合は、デバイスがブートローダーモードになっていて、ファームウェアを受信できる状態になっていることを確認してください。

```
thread 'main' panicked at 'Are you sure device is plugged in and in bootloader mode?: OpenHidDeviceError', src/libcore/result.rs:1165:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.
```

PyGamerでは、2つのボタンを押すと、PyGamerと書かれた青と緑の画面が表示され、一般的にはフラッシュドライブが作成されて、見ることができるはずです（この方法は使っていませんが）。

別のエラーが発生した場合は、必ずデバッグを実行して、プロセスのどこでエラーが発生したかを確認し、報告の際にそのログを含めてください。

```
RUST_LOG=debug cargo hf2 --vid 0x239a --pid 0x003d
```
