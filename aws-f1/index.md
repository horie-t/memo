# AWS F1インスタンスを使ってみる

Amazon EC2 F1 インスタンスは、FPGA を使用できるインスタンスです。

このページでは、まず、[AWS FPGA Hardware Development Kit (HDK)](https://github.com/aws/aws-fpga)のサンプルの cl_hello_world を実行してみて、その後、Chiselで書いたコードを動かしてみます。

## cl_hello_worldを実行する

### 概略

作業の流れは、以下の通りです。

1. FPGAイメージの開発用のEC2インスタンス(以降、開発用インスタンスと呼びます)をセットアップ
2. 開発用インスタンスでFPGAイメージの元データをビルドし、S3にアップロードしてFPGAイメージを作成
3. Amazon EC2 F1 インスタンス(以降、実行用F1インスタンスと呼びます)をセットアップ
4. 実行用F1インスタンスでS3にアップロードされたFPGAイメージをロード
5. 実行用F1インスタンスでFPGAの機能を呼び出すプログラムを実行

実行用F1インスタンスで開発用インスタンスを兼用する事もできますが、実行用F1インスタンスの料金は高いので、FPGAイメージのビルド作業は料金の安い普通のEC2インスタンス(それでも32GiB以上のメモリが必須)で行います。

インスタンスの料金(リージョンは米国西部 (オレゴン)。2020/10/2現在。F1インスタンスは4倍ぐらい違う)

| 用途 | インスタンス | 料金 |
| 開発用 | m4.2xlarge | 0.40USD/時間 |
| 実行用 | f1.2xlarge | 1.65USD/時間 |

F1 インスタンスを利用できるのは、以下のリージョンです。

* 米国東部 (バージニア北部)
* 米国西部 (オレゴン)
* 欧州 (アイルランド)
* アジアパシフィック (シドニー)

### 開発用インスタンスのセットアップ

#### インスタンスの作成

AWSのコンソール画面で以下を実施します。

1. 「EC2 ダッシュボード」で「インスタンスを起動」ボタンをクリック
2. 「AMIの選択」で、「AWS Marketplace」にある「FPGA Developer AMI」を「選択」
3. 「インスタンスタイプの選択」で「m4.2xlarge」を選択
4. 「インスタンスの設定」はデフォルトのままで、次のステップへ
5. 「ストレージの追加」では「ルート」ボリュームのサイズを「160GiB」に変更し、2つ目のボリュームは削除
6. 「タグの追加」は追加せずに、次のステップへ
7. 「セキュリティグループの設定」はデフォルトのままで、次のステップへ(本当はソースのIPを制限すべきだが...)
8. 「確認」で「起動」をクリックし、キーペアの作成等を行い、キーペアを自分のPCに配置する。
9. インスタンスが起動したら、インスタンスの「パブリックIP v4アドレス」を調べる。
10. 「インスタンスの設定」で「IAMロールを変更」
11. 「新しいIMロールを作成」
12. 「ロールの作成」
13. 「EC2」を選択して、「次のステップ」
14. 「AdministratorAccess」を選択して、「次のステップ」
15. 「タグ」は追加しない
16. 「ロール名」を設定して、「ロールの作成」
17. 「IAMロールを変更」のタブに戻ってロールを選択して保存

#### インスタンスへのログインと環境設定

SSHで、先程作成したキーペアを使って、ユーザ名:`centos`、ホスト:`インスタンスのパブリックIP v4アドレス`を指定してログインします。

sshコマンドを使う場合は以下のようにします。

```bash
ssh -i ~/.ssh/キーペアファイル名 centos@IPアドレス
```

パッケージを更新します。

```bash
sudo yum update -y
```

AWS CLIコマンドを実行できるようにします。

```bash
aws configure
```


#### HDKのインストールと環境設定

AWS FPGA HDKをインストールしてセットアップします。(最後の `source hdk_setup.sh` はログインする毎に実行する必要があります。)

```
# FPGA Developer AMIには、デフォルトでAWS_FPGA_REPO_DIR=/home/centos/src/project_data/aws-fpgaの設定がされています。
git clone https://github.com/aws/aws-fpga.git $AWS_FPGA_REPO_DIR
cd $AWS_FPGA_REPO_DIR
source hdk_setup.sh
```

### Amazon FPGAイメージ(AFI)を作成

まず、cl_hello_world のサンプルディレクトリに移動して環境変数を設定します。

```
cd $HDK_DIR/cl/examples/cl_hello_world
export CL_DIR=$(pwd)
```

次に、独自回路をビルドします。(バックグラウンドでビルドしたい場合は、「-foreground」を省略します。nohupコマンドを使うので、ログアウトしてもビルドは継続されます。)

```
cd $CL_DIR/build/scripts
./aws_build_dcp_from_cl.sh -foreground
```

ビルドが完了したら、ビルドの成果物であるデザイン・チェックポイント(DCP)のtarファイルをS3にアップロードします。

```
# アップロード先のS3バケットを作成する
aws s3 mb s3://バケット名 --region us-west-2
# DCPのtarファイルをS3にアップロードする。
aws s3 cp $CL_DIR/build/checkpoints/to_aws/*.Developer_CL.tar \
       s3://バケット名/DCPフォルダ名/
```

AFI作成時のログの出力先ディレクトリを作成します。

```
# 一時ファイルを作成
touch LOGS_FILES_GO_HERE.txt
# 一時ファイルをアップロードして、フォルダを生成する
aws s3 cp LOGS_FILES_GO_HERE.txt s3://バケット名/ログ・フォルダ名/
```

AFIを作成します。

```
aws ec2 create-fpga-image \
        --region リージョン名 \
        --name AFIイメージ名 \
        --description "AFIイメージの説明" \
        --input-storage-location Bucket=バケット名,Key=DCPのtarファイルのパス \
        --logs-storage-location Bucket=バケット名,Key=ログ・フォルダのパス
```

AFIの作成の進行状況は以下のコマンドで確認できます。AFIのIDは `create-fpga-image` の戻り値の `FpgaImageId` の値を指定します。

```
aws ec2 describe-fpga-images --fpga-image-ids AFIのID
```

戻り値の `State` の `Code` が `available` になれば作成完了です。 `failed` の場合は作成失敗です。

### 実行用F1インスタンスのセットアップ

#### インスタンスの作成

開発用インスタンスの作成と同様の手順で作成します。ただし「インスタンスタイプの選択」では「f1.2xlarge」を選択します。

#### インスタンスへのログインと環境設定

開発用インスタンスの時の同様にログインし、パッケージの更新し、AWS CLIを実行できるようにします。

#### FPGA Management toolsのインストールと環境設定

FPGA Management toolsをインストールしてセットアップします。

```
git clone https://github.com/aws/aws-fpga.git $AWS_FPGA_REPO_DIR
cd $AWS_FPGA_REPO_DIR
source sdk_setup.sh
```

#### AFIのロード

まず、FPGAのイメージを削除します。

```
sudo fpga-clear-local-image -S 0
```

以下のコマンドで、イメージの状態等を確認できます。

```
sudo fpga-describe-local-image -S 0 -H
```

イメージをロードします。AFIのグローバルIDは `create-fpga-image` の戻り値の `FpgaImageGlobalId` の値を指定します。(AFIのIDではない事に注意)

```
sudo fpga-load-local-image -S 0 -I AFIのグローバルID
```

ロードが適切に行われたかを検証します。

```
sudo fpga-describe-local-image -S 0 -R -H
```

### FPGAの機能の呼び出し

テスト用のプログラムがあるので、ビルドして実行します。

まず、cl_hello_world のサンプルディレクトリに移動します。

```
cd $HDK_DIR/cl/examples/cl_hello_world
export CL_DIR=$(pwd)
cd $CL_DIR/software/runtime/
```

テスト用プログラムをビルドします。

```
make all
```

プログラムを実行します。

```
sudo ./test_hello_world
```

実行すると以下のように表示されます。

```
$ sudo ./test_hello_world
AFI PCI  Vendor ID: 0x1d0f, Device ID 0xf000
===== Starting with peek_poke_example =====
Writing 0xefbeadde to HELLO_WORLD register (0x0000000000000500)
=====  Entering peek_poke_example =====
register: 0xdeadbeef
TEST PASSEDResulting value matched expected value 0xdeadbeef. It worked!
Developers are encouraged to modify the Virtual DIP Switch by calling the linux shell command to demonstrate how AWS FPGA Virtual DIP switches can be used to change a CustomLogic functionality:
$ fpga-set-virtual-dip-switch -S (slot-id) -D (16 digit setting)

In this example, setting a virtual DIP switch to zero clears the corresponding LED, even if the peek-poke example would set it to 1.
For instance:
# sudo fpga-set-virtual-dip-switch -S 0 -D 1111111111111111
# sudo fpga-get-virtual-led  -S 0
FPGA slot id 0 have the following Virtual LED:
1010-1101-1101-1110
# sudo fpga-set-virtual-dip-switch -S 0 -D 0000000000000000
# sudo fpga-get-virtual-led  -S 0
FPGA slot id 0 have the following Virtual LED:
0000-0000-0000-0000
```

仮想的なLEDのON/OFFとかもできるのだけど、手元で実機を動かした時のような感動はないのが残念…

## Chiselのコードを動かす

### cl_hello_worldのコア部分を抜き出す

いきなりスクラッチからコードを書くの難しいので、 cl_hello_world の一部を置き換えてみる。具体的には、`cl_hello_world.sv`ファイルの以下の部分の置き換えを目指す。

```verilog
//-------------------------------------------------
// Hello World Register
//-------------------------------------------------
// When read it, returns the byte-flipped value.

always_ff @(posedge clk_main_a0)
   if (!rst_main_n_sync) begin                    // Reset
      hello_world_q[31:0] <= 32'h0000_0000;
   end

// 中略.......

// The register contains 16 read-only bits corresponding to 16 LED's.
// For this example, the virtual LED register shadows the hello_world
// register.
// The same LED values can be read from the CL to Shell interface
// by using the linux FPGA tool: $ fpga-get-virtual-led -S 0

always_ff @(posedge clk_main_a0)
   if (!rst_main_n_sync) begin                    // Reset
      vled_q[15:0] <= 16'h0000;
   end
   else begin
      vled_q[15:0] <= hello_world_q[15:0];
   end

// The Virtual LED outputs will be masked with the Virtual DIP switches.
assign pre_cl_sh_status_vled[15:0] = vled_q[15:0] & sh_cl_status_vdip_q2[15:0];
```

上記の部分を別モジュールとして、別ファイル(`cl_hello_world_core.sv`)に抜き出す。

```varilog
module cl_hello_world_core (input         clk_main_a0,
			    input 		rst_main_n_sync,
			    input [31:0] 	wr_addr,
			    input [31:0] 	wdata,
			    input 		wready,
			    input [15:0]        sh_cl_status_vdip, 	
			    output logic [31:0] hello_world_q_byte_swapped,
			    output logic [15:0] cl_sh_status_vled,
			    output logic [15:0] vled_q);
`include "cl_common_defines.vh"      // CL Defines for all examples

   logic [31:0] 				hello_world_q;
   logic [15:0] 				sh_cl_status_vdip_q;
   logic [15:0] 				sh_cl_status_vdip_q2;
   logic [15:0] 				pre_cl_sh_status_vled;
  
   //-------------------------------------------------
   // Hello World Register
   //-------------------------------------------------
   // When read it, returns the byte-flipped value.

   always_ff @(posedge clk_main_a0)
     if (!rst_main_n_sync) begin                    // Reset
	hello_world_q[31:0] <= 32'h0000_0000;
     end
     else if (wready & (wr_addr == `HELLO_WORLD_REG_ADDR)) begin  
	hello_world_q[31:0] <= wdata[31:0];
     end
     else begin                                // Hold Value
	hello_world_q[31:0] <= hello_world_q[31:0];
     end

   assign hello_world_q_byte_swapped[31:0] = {hello_world_q[7:0],   hello_world_q[15:8],
                                              hello_world_q[23:16], hello_world_q[31:24]};

   //-------------------------------------------------
   // Virtual LED Register
   //-------------------------------------------------
   // Flop/synchronize interface signals
   always_ff @(posedge clk_main_a0)
     if (!rst_main_n_sync) begin                    // Reset
	sh_cl_status_vdip_q[15:0]  <= 16'h0000;
	sh_cl_status_vdip_q2[15:0] <= 16'h0000;
	cl_sh_status_vled[15:0]    <= 16'h0000;
     end
     else begin
	sh_cl_status_vdip_q[15:0]  <= sh_cl_status_vdip[15:0];
	sh_cl_status_vdip_q2[15:0] <= sh_cl_status_vdip_q[15:0];
	cl_sh_status_vled[15:0]    <= pre_cl_sh_status_vled[15:0];
     end

   // The register contains 16 read-only bits corresponding to 16 LED's.
   // For this example, the virtual LED register shadows the hello_world
   // register.
   // The same LED values can be read from the CL to Shell interface
   // by using the linux FPGA tool: $ fpga-get-virtual-led -S 0

   always_ff @(posedge clk_main_a0)
     if (!rst_main_n_sync) begin                    // Reset
	vled_q[15:0] <= 16'h0000;
     end
     else begin
	vled_q[15:0] <= hello_world_q[15:0];
     end

   // The Virtual LED outputs will be masked with the Virtual DIP switches.
   assign pre_cl_sh_status_vled[15:0] = vled_q[15:0] & sh_cl_status_vdip_q2[15:0];


endmodule // cl_hello_world_core
```

`cl_hello_world.sv`は以下のように修正する。

```verilog
// 前略
  //logic [15:0] pre_cl_sh_status_vled;
  //logic [15:0] sh_cl_status_vdip_q;
  //logic [15:0] sh_cl_status_vdip_q2;
  //logic [31:0] hello_world_q;

// 中略
   // 上記の抽出対象を置き換える
   cl_hello_world_core CL_HELLO_WORLD_CORE(clk_main_a0,
                                          rst_main_n_sync,
                                          wr_addr,
                                          wdata,
                                          wready,
                                          sh_cl_status_vdip,
                                          hello_world_q_byte_swapped,
                                          cl_sh_status_vled,
                                          vled_q);
// 後略
```

`build/scripts/encrypt.tcl`のファイルも修正する。真ん中の行を追加しています。

```tcl
file copy -force $CL_DIR/design/cl_hello_world.sv                     $TARGET_DIR 
file copy -force $CL_DIR/design/cl_hello_world_core.sv                $TARGET_DIR 
file copy -force $CL_DIR/../common/design/cl_common_defines.vh        $TARGET_DIR 
```

### Chiselコードの作成

前述の`cl_hello_world_core`と同等のモジュールをChiselで書くと以下のようになります。

```scala
import chisel3._
import chisel3.util._

class ClHelloWorldCore extends Module {
  val io = IO(new Bundle {
    val wrAddr = Input(UInt(32.W))
    val wData = Input(UInt(32.W))
    val wrReady = Input(Bool())
    val shClStatusVDip = Input(UInt(16.W))
    val helloWorldQByteSwapped = Output(UInt(32.W))
    val clShStatusVLed = Output(UInt(16.W))
    val vLed = Output(UInt(16.W))
  })

  val HELLO_WORLD_REG_ADDR = "h0000_0500".U(32.W)

  //-------------------------------------------------
  // Hello World Register
  //-------------------------------------------------
  // When read it, returns the byte-flipped value.
  val helloWorldQ = RegInit(0.U(32.W))
  when(io.wrReady === true.B && io.wrAddr === HELLO_WORLD_REG_ADDR) {
    helloWorldQ := io.wData
  }
  io.helloWorldQByteSwapped := Cat(helloWorldQ(7, 0), helloWorldQ(15, 8),
    helloWorldQ(23, 16), helloWorldQ(31, 24))

  //-------------------------------------------------
  // Virtual LED Register
  //-------------------------------------------------
  // Flop/synchronize interface signals
  val shClStatusVDipQ = RegNext(io.shClStatusVDip , 0.U(16.W))
  val shClStatusVDipQ2 = RegNext(shClStatusVDipQ, 0.U(16.W))

  val vLedQ = RegNext(helloWorldQ(15, 0), 0.U(16.W))
  val preClShStatusVLed = vLedQ & shClStatusVDipQ2
  val clShStatusVLed = RegNext(preClShStatusVLed, 0.U(16.W))

  io.vLed := vLedQ
  io.clShStatusVLed := clShStatusVLed
}

object ClHelloWorldCore extends App {
  chisel3.Driver.execute(args, () => new ClHelloWorldCore)
}
```

上記から生成される`ClHelloWorldCore.v`を`ClHelloWorldCore.sv`としてコピーします。

`build/scripts/encrypt.tcl`のファイルも以下のように修正します。

```tcl
file copy -force $CL_DIR/design/cl_hello_world.sv                     $TARGET_DIR
file copy -force $CL_DIR/design/ClHelloWorldCore.sv                   $TARGET_DIR
file copy -force $CL_DIR/../common/design/cl_common_defines.vh        $TARGET_DIR
```

`cl_hello_world.sv`は、Chilseのリセットは正論理なので、以下のようにリセットを反転させます。

```
   cl_hello_world_core CL_HELLO_WORLD_CORE(clk_main_a0,
                                          !rst_main_n_sync,
                                          wr_addr,
                                          wdata,
                                          wready,
                                          sh_cl_status_vdip,
                                          hello_world_q_byte_swapped,
                                          cl_sh_status_vled,
                                          vled_q);

```
