# [汎用モード](https://mgba-emu.github.io/gbatek/#siogeneralpurposemode)

このモードでは，SIOは4ビットの双方向パラレルポートとして「誤使用」され，SI，SO，SC，SDの各端子が直接制御され，それぞれが入力信号（内部プルアップ付き）または出力信号として個別に宣言されます。

## レジスタ

### 4000134h - RCNT - モード選択レジスタ (R/W)

SIOCNTのbit13-12と合わせて、転送モードを設定するのに利用します。

他のモードと違って、bit0-8, bit14-15が利用されます。

ビット | 内容
---- | ---- 
0    | SCデータビット (0=Low, 1=High) 
1    | SDデータビット (0=Low, 1=High)
2    | SIデータビット (0=Low, 1=High)
3    | SOデータビット (0=Low, 1=High)
4    | SC入出力フラグ (0=Input, 1=Output)
5    | SD入出力フラグ (0=Input, 1=Output)
6    | SI入出力フラグ (0=Input, 1=Output)
7    | SO入出力フラグ (0=Input, 1=Output)
8    | SI割り込み有効化フラグ (0=無効, 1=有効)
9-13 | 不使用 (常に0,読み取り専用)
14   | 0
15   | 1

SI should be always used as Input to avoid problems with other hardware which does not expect data to be output there.

### 4000128h - SIOCNT - SIO制御レジスタ

汎用モードではこのレジスタは使われません。

That is, the separate bits of SIOCNT still exist and are read- and/or write-able in the same manner as for Normal, Multiplay, or UART mode (depending on SIOCNT Bit 12,13), but are having no effect on data being output to the link port.

