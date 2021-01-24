# BG制御レジスタ

Mode0 では、タイルやマップ（タイルを並べて作った背景画面）の使い方を指定するため、BG制御レジスタを設定します。背景はBG0～BG3の4枚があり、それぞれに対応するBG制御レジスタがあります。

```
0x0400_0008 - BG0CNT - BG0制御レジスタ (R/W) (BG Modes 0,1 のみ)
0x0400_000a - BG1CNT - BG1制御レジスタ (R/W) (BG Modes 0,1 のみ)
0x0400_000c - BG2CNT - BG2制御レジスタ (R/W) (BG Modes 0,1,2 のみ)
0x0400_000e - BG3CNT - BG3制御レジスタ (R/W) (BG Modes 0,2 のみ)
```

 bit | 内容
---- | ----
0-1 | BG優先度(0~3, 0=最高)
2-3 | Character Base Block  (0-3, in units of 16 KBytes) (=BG Tile Data)
4-5   | 未使用, 0
6     | モザイク                (0=無効, 1=有効)
7     | カラーモード       (0=16/16, 1=256/1)
8-12  | Screen Base Block     (0-31, in units of 2 KBytes) (=BG Map Data)
13    | BG0/BG1: Not used (except in NDS mode: Ext Palette Slot for BG0/BG1)
13    | BG2/BG3: Display Area Overflow (0=Transparent, 1=Wraparound)
14-15 | 仮想画面サイズ (0-3)

優先度が同じBGが複数ある場合はBG0 -> BG3の順に優先されます(つまりBG0が最優先)。

## カラーモード(bit7)

カラーモード(bit7)は

- 0: 16色/16パレット
- 1: 256色/1パレット

## 仮想画面サイズ(bit14-bit15)

仮想画面サイズ と BGマップのサイズ(Byte単位)の関係は次のようになっています。

 仮想画面サイズ | テキストモードのとき | 伸縮回転モードのとき
---- | ---- | ---- 
0 | 256x256 (2K) | 128x128   (256 bytes)
1 | 512x256 (4K) | 256x256   (1K)
2 | 256x512 (4K) | 512x512   (4K)
3 | 512x512 (8K) | 1024x1024 (16K)

## 例

 STATE | HEX | Binary | 内容 
---- | ---- | ---- | ---- 
Default | 0x0000 | 0b0000_0000_0000_0000 | 仮想スクリーン32x32tile, カラー16色
例1 | 0x1c80 | 0b0001_1100_1000_0000 | 仮想スクリーン32x32tile, カラー256色, タイルブロック0を使用, マップブロック28 (11100)を使用
