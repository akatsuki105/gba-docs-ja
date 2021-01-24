# ディスプレイ制御

## 0x0400_0000 - DISPCNT - LCD制御レジスタ (R/W)

 bit  |  内容
---- | ----
0-2 | BGモード                (0-5=ビデオモード0-5, 6-7=無効)
3   | CGBモード    (0=GBA, 1=CGB BIOSオペコードで設定されます)
4   | フレーム選択   (0-1=Frame0-1) (BGモードが4,5のときのみ)
5   | HBlank中にOAMにアクセス可能か (1=可能)
6   | OBJ Character VRAM Mapping (0=2次元, 1=1次元)
7   | 強制白塗り     (1=有効 その間はVRAM,Palette,OAMに高速アクセス可能)
8   | BG0有効フラグ  (0=Off, 1=On)
9   | BG1有効フラグ  (0=Off, 1=On)
10  | BG2有効フラグ  (0=Off, 1=On)
11  | BG3有効フラグ  (0=Off, 1=On)
12  | OBJ有効フラグ  (0=Off, 1=On)
13  | Window0有効フラグ  (0=Off, 1=On)
14  | Window1有効フラグ   (0=Off, 1=On)
15  | OBJ Window有効フラグ (0=Off, 1=On)

次のテーブルは0から5までのBGモード(ビデオモード)がどのような機能を持っているか表したものです。

 モード | Rot/Scal | レイヤーサイズ | タイル | 色 | 属性 | 備考
---- | ---- | ---- | ---- | ---- | ---- | ----
0 | No | 0123 | 256x256..512x515 | 1024 | 16/16..256/1 | SFMABP |
1 | Mixed | 012- | | | | (BG0,BG1はMode0と同じ, BG2はMode2と同じ)
2 | Yes   | --23 | 128x128..1024x1024 | 256 | 256/1 | S-MABP |
3 | Yes   | --2- | 240x160            | 1   | 32768 | --MABP |
4 | Yes   | --2- | 240x160            | 2   | 256/1 | --MABP |
5 | Yes   | --2- | 160x128            | 2   | 32768 | --MABP |

属性: S)crolling, F)lip, M)osaic, A)lphaBlending, B)rightness, P)riority.

BGモード0-2はタイルモード用です。BGモード3-5はビットマップモード用で、これらのモードでは 1 つまたは 2 つのフレーム (すなわち、ビットマップ、または 'フルスクリーンタイル' ) が存在します。

2つのフレームが存在する場合は、どちらか1つを表示することができ、他の1つは背景に再描画することができます。

### 例

 STATE | HEX | Binary | 内容 
---- | ---- | ---- | ---- 
Default | 0x0080 | 0b0000_0000_1000_0000 | bit7=1:強制白塗り
Mode3 | 0x0403 | 0b0000_0100_0000_0011 | bit10:BG2の背景を使用, bit0-2:モード3
Mode0 | 0x0100 | 0b0000_0001_0000_0000 | bit8:BG0の背景を使用, bit0-2:モード0

## Blanking Bits

Setting Forced Blank (Bit 7) causes the video controller to display white lines, and all VRAM, Palette RAM, and OAM may be accessed.

"When the internal HV synchronous counter cancels a forced blank during a display period, the display begins from the beginning, following the display of two vertical lines." 

What?

Setting H-Blank Interval Free (Bit 5) allows to access OAM during H-Blank time - using this feature reduces the number of sprites that can be displayed per line.

## 有効フラグ

デフォルトではBG0-3有効フラグとOBJ有効フラグ(Bit8-12)がBGとOBJの有効無効化に使われます。

Bit13-14を通してWindow0/1を有効にすると、特殊効果が適用され、BG0-3とOBJはウィンドウによって制御されます。

## フレーム選択

BGモード4-5（ビットマップモード）では、2つのビットマップ/フレームのうちどちらか一方が表示され(Bit4で選択)、バックグラウンドでもう一方の（見えない）フレームを更新することができます。

BGモード3では、1つのフレームのみが存在します。

BGモード0-2(タイルモード) では、BGマップ や BGキャラクタデータ のベースアドレスを変更することで、同様の効果を得ることができます。

## 0x0400_0002 - Undocumented - Green Swap (R/W)

Normally, red green blue intensities for a group of two pixels is output as BGRbgr (uppercase for left pixel at even xloc, lowercase for right pixel at odd xloc). 

When the Green Swap bit is set, each pixel group is output as BgRbGr (ie. green intensity of each two pixels exchanged).

 bit  |  内容
---- | ----
0 | グリーンスワップ (0=Normal, 1=Swap)
1 | 未使用

この機能は、最終的に生成される画像(つまり別々のBGとOBJレイヤを混合した後)に適用されるようです。 

最終的には、他のディスプレイタイプ（他のピンアウトを持つ）のために意図されています。

通常のGBAハードウェアでは、面白い汚れの効果を生み出しているだけです。

NDSのDISPCNTレジスタは32ビット(0x0400_0000...0x0400_0003)なので、NDSモードではグリーンスワップは存在しませんが、GBAモードではNDSはグリーンスワップをサポートしています。

