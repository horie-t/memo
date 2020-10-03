# AWS F1インスタンスを使ってみる

Amazon EC2 F1 インスタンスは、FPGA を使用できるインスタンスです。

このページでは、まず、AWS FPGA Hardware Development Kit (HDK)のサンプルの cl_hello_world を実行してみて、その後、Chiselで書いたプロジェクトを動かしてみます。

## cl_hello_worldを実行する

### 概略

作業の流れは、以下の通りです。

1. FPGAイメージの開発用のEC2インスタンス(以降、開発用インスタンスと呼びます)をセットアップ
2. 開発用インスタンスでFPGAイメージをビルドしてS3にアップロード
3. Amazon EC2 F1 インスタンス(以降、実行用F1インスタンスと呼びます)をセットアップ
4. 実行用F1インスタンスでS3にアップロードされたFPGAイメージをロード
5. FPGAの機能を呼び出すプログラムを実行

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

次に、独自回路をビルドします。ビルドの実行自体はバックグラウンドで行われるので、すぐにプロンプトが戻ります。

```
cd $CL_DIR/build/scripts
./aws_build_dcp_from_cl.sh
```

上記のスクリプト実行時に以下の例のような内容が表示されます。

```
INFO: Output is being redirected to 20_10_03-020546.nohup.out
```

以下の例のコマンドを実行して、ビルドの進行状況を確認できます。( `Ctrl + C` キーで確認を終了できます。)
(メールでビルド完了を通知させる方法もありますが、今回は省略します)

```
tail -f 20_10_03-020546.nohup.out
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
