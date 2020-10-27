# CL_DRAM_DMA のメモ

## 実行方法

インスタンスの準備やリポジトリのclone、AFIの生成方法等は、[AWS EC2 F1インスタンスを使ってみる](./index.html)と同じ。

### F1インスタンスでの作業

リポジトリのcloneまで終わったら、以下の手順を実施します。

#### XOCLドライバのアンインストール

テストプログラムの実行には、XDMAドライバが必要。しかし、FPGA Developer AMIのversion 1.5.0以降では、XOCLのドライバがインストールされていて、XDMAドライバが動かない。よって、XOCLドライバを取り除いてから、XDMAドライバをインストールする。

XOCLドライバがインストールされているかをチェックする。以下のようにxoclが表示されればインストールされています。

```bash
$ lsmod | grep xocl
xocl                  805140  0 
drm                   456166  5 ttm,xocl,drm_kms_helper,cirrus
libcrc32c              12644  2 xfs,xocl
```

XOCLドライバをアンインストールします。

```bash
sudo rmmod xocl
```

#### XDMAドライバのインストール

XOCLドライバをビルド、インストールします。

ドライバのビルドに必要なツールをインストールします。

```bash
sudo yum groupinstall "Development tools"
sudo yum install kernel kernel-devel
```

カーネルがインストールされたら、再起動します。

```bash
sudo shutdown -r now
```

再起動後に、以下のようにドライバをビルドします。

```bash
cd $AWS_FPGA_REPO_DIR/sdk/linux_kernel_drivers/xdma
make
```

ドライバをインストールします。

```bash
sudo make install
```

ドライバをロードします。(OSの再起動は不要です)

```bash
sudo modprobe xdma
```

ドライバがロードされているかは以下のように確認します。

```bash
$ lsmod | grep xdma
xdma                   72503  0 
```

#### テストプログラムの実行

テストプログラムの実行は以下の通り。
```bash
cd $HDK_DIR/cl/examples/cl_hello_world
export CL_DIR=$(pwd)
cd $CL_DIR/software/runtime/
make all
sudo ./test_dram_dma
```

## テストプログラムの内容

### 初期化

* <fpga_pci_sv.h> fpga_mgmt_init(): ライブラリを初期化
* "test_dram_dma_common.h" check_slot_config(int slot_id): slot_idのFPGAをチェック

### dma_example

FPGAに載っている4つのDDR4 SDRAMに16MBのランダムなデータを書き込んで、読み出して、差がないかをチェック

### interrupt_example



### axi_mstr_example
