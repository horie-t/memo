# SG90サーボモータのサーボホーンを3Dプリンタで作ってみる。

サーボホーンを光造形3Dプリンタ(ELEGOO MARS2)で作ってみました。[SG90のギアを測定したサイト](https://www.robotshop.com/community/blog/show/modelling-a-servo-spline)を参考にしています。

注意: 3Dプリンタでのプリントは、設計データと1ミクロンの誤差なくプリントされるものではありません。3Dプリンタの機種や使う樹脂、印刷条件等によって出力のサイズは異なります。ここでの数値はあくまで参考値となり、各自の環境において調整をしてください。

# サーボホーンの設計

造形物の設計データは、オープンソースの3Dモデラーの[FreeCAD](https://www.freecadweb.org/?lang=ja)で作成しました。

## ギアの作成

1. FreeCADを起動し、ワークベンチを、Part Design ![Part Design](./images/part_design_icon.png) に切り替えます。
2. 新規ボタン ![新規ボタン](./images/new_document.png) をクリックし、ドキュメントを作成します。
3. 新規スケッチを作成 ![新規スケッチを作成](./images/new_scketch.png) をクリックして、スケッチを作成します。
4. フィーチャを選択ダイアログで、「XY_Plane」を選択して、「OK」をクリックします。  
   ![フィーチャを選択](./images/select_feature.png)
