# 振動パック

振動パックはGBAを振動させるための周辺機器です。

振動パックを備えたカートリッジは、一般的には「振動カートリッジ」と呼ばれています。

ライセンスソフトで振動カートリッジなのは

- [スクリューブレイカー 轟振どりるれろ](https://www.nintendo.co.jp/n08/v49j/index.html)
- [まわるメイドインワリオ](https://www.nintendo.co.jp/n08/rzwj/index.html)

が該当します。

GBAの振動カートリッジには小型モーターが搭載されており、スイッチを入れたとき/入れているときに振動を発生させます。DSの振動カートリッジでは、GBAと違ってスイッチのオン・オフを繰り返すことで振動を発生させているようです。

まわるメイドインワリオでは、[GPIO](gpio.md)のbit3(データ、ディレクション)を振動機能の制御に利用して、他のbitはジャイロセンサーに利用しているようです。

Note: GPIO3 is connected to an external pulldown resistor (so the HighZ level gets dragged to Low=Off when direction is set to Input).

スクリューブレイカー 轟振どりるれろ での詳細は不明です。

## DSの振動パック

また、NDSにはGBAスロットに接続する振動パックがあり、(マルチブートゲームのようなGBAスロットを必要としないゲームなら)GBAのゲームにも使用することができます。

詳細は[こちら](https://problemkaputt.de/gbatek.htm#dscartrumblepak)を参照

## ゲームキューブの振動パック

また、ゲームボーイプレイヤーで動作するGBAゲームは、ゲームキューブコントローラーの振動機能を利用しています。

詳細は[こちら](https://problemkaputt.de/gbatek.htm#gbagameboyplayer)を参照
