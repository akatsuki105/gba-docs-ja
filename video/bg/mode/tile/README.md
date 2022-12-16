# Mode0,1,2

Mode0-2 は8x8タイル単位の描画を行うのでタイルモード(テキストモード)と呼ばれています。

## タイルモード(Mode0-2)の共通の仕様

```
  0x0600_0000..0600_FFFF:  64KBでBGマップとBGタイルデータで共有
  0x0601_0000..0601_7FFF:  32KBでOBJタイルデータが格納
```

VRAMのメモリ空間のうち、64KBを[BGタイルデータ](tiledata.md)と[BGマップ](bgmap.md)で共有します。それぞれのリソースがどのアドレスを使うかは`BGnCNT`レジスタで設定します。

## Mode0

BG0-3 の4枚のBGレイヤが使えます。各BGレイヤの性能は同じです。

```
  BG0-3:
    Color:    4bpp or 8bpp
    Size:     32x32 or 32x64 or 64x32 or 64x64 タイル
    Scalerot: 不可能
    Tile:     1024種類まで
```

## Mode1

BG0-2 の3枚のBGレイヤが使えます。

```
  BG0-1:
    Color:    4bpp or 8bpp
    Size:     32x32 or 32x64 or 64x32 or 64x64 タイル
    Scalerot: 不可能
    Tile:     1024種類まで
  BG2:
    Color:    8bpp
    Size:     16x16 ~ 128x128
    Scalerot: 可能
    Tile:     256種類まで
```

## Mode2

BG2-3 の2枚のBGレイヤが使えます。

```
  BG2-3:
    Color:    8bpp
    Size:     16x16 ~ 128x128
    Scalerot: 可能
    Tile:     256種類まで
```
