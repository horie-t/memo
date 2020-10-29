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

FPGAのレジスタに、fpga_pci_pokeで割込み番号を指定して書き込むと、FPGA側からPCIe MSI-X(Message-Signaled Interrupt - X)割込みがインスタンス側に発生するようになっている。

一連の流れは概略は以下の通り。変数の宣言やエラー処理は省略

```c
    // pollシステムコールで受け取るファイル・ディスクリプタ
    struct pollfd fds[1];
    // PCIスロットを指定してデバイス番号を取得
    fpga_pci_get_dma_device_num(FPGA_DMA_XDMA, slot_id, &device_num)
    // 割込みを受けるデバイスファイル名を生成
    sprintf(event_file_name, "/dev/xdma%i_events_%i", device_num, interrupt_number);
    // FPGAに接続
    fpga_pci_attach(slot_id, pf_id, bar_id, fpga_attach_flags, &pci_bar_handle);
    // デバイスファイルを開いて、pollのファイル・ディスクリプタを設定
    fd = open(event_file_name, O_RDONLY))
    fds[0].fd = fd;
    fds[0].events = POLLIN;
    // FPGAのレジスタに値を書き込む
    fpga_pci_poke(pci_bar_handle, interrupt_reg_offset , 1 << interrupt_number);
    // 割込みイベントを待つ
    rd = poll(fds, num_fds, poll_timeout);
    if((rd > 0) && (fds[0].revents & POLLIN)) {
        // イベントを読み込み
        rc = pread(fd, &events_user, sizeof(events_user), 0);
	// FPGA側に割込みのクリアを指示
	fpga_pci_poke(pci_bar_handle, interrupt_reg_offset , 0x1 << (16 + interrupt_number) );
    }
```

### axi_mstr_example

## FPGA側の構成

`cl_dram_dma.sv`ファイルの`cl_dram_dma`がTopモジュール。この中にDRAMアクセスや割込みの実例のモジュールがインスタンス化されている。

### DRAMアクセス

### 割込み

`cl_int_slv.sv`ファイルの`cl_int_slv`と、`cl_int_tst.sv`ファイルの`cl_int_tst`が、F1インスタンス側に割込みを要求するモジュール。
`cl_ocl_slv.sv`ファイルの`cl_ocl_slv`モジュールが、F1インスタンスからの`0xd00`番地のレジスタ書き込みを受け付けて、cl_int_slvに割込みの発生を要求する。



