# CloudFormationを使ってAWS EC2 F1インスタンスの開発環境を構築する

【動機】週末にしか動かさないのでEBS ボリュームの維持費がもったいない。だったら、毎週インスタンスの作成から始めた方が安い気がする。とはいえ、毎回GUIでインスタンスの作成や環境構築をするのは面倒くさい。

ここでは、[AWS EC2 F1インスタンスを使ってみる](https://github.com/horie-t/memo/blob/master/aws-f1/index.html)にある環境を構築する方法を記述しています。gitリポジトリのcloneまでは実行しています。

## 環境の構築手順

**CloudFormationのyamlファイルのダウンロード**

[こちら](https://github.com/horie-t/aws-fpga/blob/master/CloudFormation/cloud_formation.yaml)のファイルをダウンロードしてください。

**CloudFormationのスタックの作成**

yamlファイルをダウンロードしたディレクトリに移動して、以下のコマンドを実行します。

```bash
aws cloudformation create-stack --stack-name F1Dev --template-body file://`pwd`/cloud_formation.yaml  --parameters ParameterKey=KeyPairParameter,ParameterValue=キーペアの名前
```

## 環境の削除手順

インスタンスが不要になったら以下のコマンドを実行します。

**スタックの削除**

```bash
aws cloudformation delete-stack --stack-name F1Dev
```
