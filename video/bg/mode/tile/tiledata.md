# タイルデータ

**キャラクタデータ** や BGタイルデータ とも呼ばれます。 タイルモードにおける描画単位です。

各タイルは `8×8px` です。 

色深度(色を表すのに利用するbit数)は 4bit か 8bit のどちらかを[BGnCNT](../../control.md)で選択できます。

### 4bit (16色/16パレット)

1pxあたり4bit(0.5バイト)なので `1タイル = 64px = 64×0.5 = 32バイト`です。

最初の4バイトはタイルの最上段の行のためのものです。その次の4バイトは次の行となり、以後同様です。

各バイトは2つのドットを表し、下位4ビットが**左ドット**の色、上位4ビットが**右ドット**の色を定義します。

### 8bit (256色/1パレット)

`1タイル = 64バイト`です。

4bitと同様に最初の8バイトが最初の行に対応し、その後の行も同様です。

各バイトは、各ドットのパレットエントリを選択します。