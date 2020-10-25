# AWS FPGA Hardware Development Kit (HDK)の使い方

## FPGAの構成

AWS EC2 F1インスタンスのFPGA部分は、大きく以下の2つの部分に分かれています。

* シェル(Shell: SH) - AWSプラットフォームによって実装されたFPGAの周辺回路(PCIe、DRAM、DMA、割込み)
* 独自回路(Custom Logic: CL) - FPGA開発ユーザによって生成された独自の加速回路

開発の最終段階で、ShellとCLが結合されてAmazon FPGA Image(AFI)が作成されます。AFIはF1インスタンスにロードされます。

## アプリケーション・ソフトウェアとShell(SH)とCustom Logic(CL)の関係

アプリケーション・ソフトウェアとSHとCLの関係は以下のようになります。

![FPGA利用時のソフトウェア構成](https://github.com/aws/aws-fpga/blob/master/hdk/docs/images/AWS_FPGA_Software_Overview.jpg)

上図のようにソフトウェアは、SHを経由してCLの機能を利用します。SHには以下の2つのPF(Physical Function)があり、このPFをソフトウェアが利用します：

* Management PF(MgmtPF): 管理用の機能(AFIのロード等)を提供します。
* Application PF(AppPF): CLの機能を利用するためのものです。

MgmtPFを利用するには、Aのshellコマンドや、Bのライブラリ、CのOpenCLのライブラリを使います。
AppPFを利用するには、CのOpenCLのライブラリや、DやEやFのライブラリを使います。

次に、SHとCLは以下のように接続されています。

![SHとCL](https://github.com/aws/aws-fpga/blob/master/hdk/docs/images/AWS_Shell_CL_overview.jpg)

F1インスタンスとFPGA部分はPCIeインタフェース(I/F)で接続されています。PCIeはSHと接続されています。SHとCL間は、ARM社が開発したAXI-4 I/Fで接続されています(つまりSHはPCIeとAXIの変換を行っています)。
