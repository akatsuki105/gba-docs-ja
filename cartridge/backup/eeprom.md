# ⚡️ EEPROM

```
  EEPROM 512B
    チップ名: 9853
    セーブ機能の寿命: アドレスあたり100,000回の読み書き
    例: スーパーマリオアドバンス
  EEPROM 8KB
    チップ名: 9854
    セーブ機能の寿命: アドレスあたり100,000回の読み書き
    例: ボクらの太陽
```

## アクセス

EEPROM はデータバスのbit0と、カートリッジROMアドレスバスの上位1bit（32MBの大容量ROMの場合は上位17bit）に接続されており、チップとの通信はシリアルで行われます。

EEPROMは`0xDFFFF00..0xDFFFFFF`(WS2)にマッピングされます。(`WAITCNT=0xX3XXh`の場合)アクセスには8サイクルが必要です。

EEPROMのデータは、64bit（8バイト）単位で読み書きできます。

書き込みを行うと、それまでの64bitのデータは自動的に消去されます。

アドレッシングについて、512BのEEPROMでは、`0x0..3F`のアドレス範囲、6bitのバス幅となります。8KBのEEPROMでは、`0x0..3FF`のアドレス範囲，14bitのバス幅となります。（アドレスの下位10bitのみ使用し、上位4bitは0とします）

## コマンド

**読み込み対象のアドレス設定**

次のようなビットストリームをメモリ上に用意し、DMAを使ってEEPROMに転送します。

```
  11XXXXXX0
    0b11:     読み込みリクエストを示す2bit
    0bXXXXXX: 設定する6bitのEEPROMのアドレス (8KBのEEPROMでは14bit)
    0b0:      終端bit
```

**データの読み込み**

DMAを使ってEEPROM(`0xDFFFF00..0xDFFFFFF`)から68bitのビットストリームを読み出し、受信したデータを以下のようにデコードします。

```
  4 bits  - 無視
  64 bits - 読み出したデータ
```

**データの書き込み**

次のようなビットストリームをメモリ上に用意し、DMAを使ってEEPROMに転送します。

古いデータが消去され、新しいデータが書き込まれるまで、約108368クロックサイクル（約6.5ms）かかります。

```
  10XXXXXXDD...DDD0
    0b10:       書き込みリクエストを示す2bit
    0bXXXXXX:   書き込み先を示す6bitのEEPROMのアドレス (8KBのEEPROMでは14bit)
    0bDD...DDD: 書き込む64bitデータ
    0b0:        終端bit
```

DMA終了後、通常の`LDRH [DFFFF00h]`により、戻ってきたデータのビット0が "1"(Ready)になるまで、チップからの読み出しを続けます。

誤動作時にプログラムがロックしないように、10ms以上経ってもチップが応答しない場合はタイムアウトを発生させてください。

## DMAによるアクセス

ビットストリームのEEPROMに対する読み書きは、DMA3を介して行う必要があります。 転送中に`/CS=LOW`と`A23=HIGH`を維持しないため、LDRH/STRHによる手動転送は動作しません。

DMAを使用するためには、メモリ上のバッファを使用する必要があります。そのバッファは、通常、スタック上に一時的に割り当てられ、2バイトあたり1bitを使用します。つまりbit0のみが使用され、bit1-15は無視されます。

バッファはDMA3を使用してEEPROMへ(or から)全体として転送する必要があります（DMA0-2は外部メモリにアクセスできません）。転送モードは16bitで、ソースとデスティネーションの両方のアドレスをインクリメントします。（例：DMA3CNT=`0x80000000+length`）

優先度の高いDMAチャンネルは、転送中は無効にしてください（例：H/V-BlankやSound FIFO DMA）。また、DMAレジスタに干渉する可能性のある割り込みは、当然ながら無効にしてください。

**注意**

自動検出の仕組みがないため、ハードコードされたバス幅を使用しなければならないようです。
