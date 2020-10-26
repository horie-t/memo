# CL_DRAM_DMA のメモ

## 実行方法

AFIの生成方法は、[AWS EC2 F1インスタンスを使ってみる](./index.html)と同じ。

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
