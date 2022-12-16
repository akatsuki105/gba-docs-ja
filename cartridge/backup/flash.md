# 📸 FLASH

```
  512Kb Flash ROM: 64 KB
    セーブ機能の寿命: セクタあたり10,000回の書き込み
    例: ソニックアドバンス
  1Mb Flash ROM:   128 KB
    セーブ機能の寿命: セクタあたりの書き込み回数上限があるはずだが、具体的な回数は不明
    例: ポケットモンスターエメラルド
```

## チップの種類

任天堂は市販のゲームカートリッジに様々な種類のFLASHチップを搭載しています。エミュレータ開発者はすべてのチップタイプを検出し、サポートする必要があります。

Atmel製チップの場合、ソフトウェアで4KBセクタをシミュレートすることが推奨されますが、任天堂は後期のゲームではAtmelチップを使用していないと言われています。

```
  ID     Name       Size  Sectors  AverageTimings  Timeouts/ms   Waits(w,r)
  D4BFh  SST        64K   16x4K    20us?,?,?       10,  40, 200  3,2
  1CC2h  Macronix   64K   16x4K    ?,?,?           10,2000,2000  8,3
  1B32h  Panasonic  64K   16x4K    ?,?,?           10, 500, 500  4,2
  3D1Fh  Atmel      64K   512x128  ?,?,?           ...40..,  40  8,8
  1362h  Sanyo      128K  ?        ?,?,?           ?    ?    ?    ?
  09C2h  Macronix   128K  ?        ?,?,?           ?    ?    ?    ?
```

IDの MSBはデバイスの種類、LSBは製造元を表しています。

FLASHチップの種類（および存在）を検出するためには後述のチップの検出コマンドを行います。

## アクセス

FLASHメモリは，`0xE000000..E00FFFF`の SRAMエリアに配置されており、(`WAITCNT.0-1 = 3`の場合)アクセスには8サイクルを要します。

特定のデバイスタイプが検出された場合、書き込み・消去には、短い待機時間ですみ、生の読み取りにはさらに小さい待機時間で済むことがあります。

データバスは8bitに制限されています。よって`LDRB/STRB`オペコードでしかアクセスできません。

また、FLASHメモリからの読み出しは、WRAMで実行されたコードのみが行うことができます。(ROMで実行されたコードは不可。) 書き込みについては、そのような制限はありません。

## Verify

書き込み・消去の完了を知らせる信号が出ていても、変更されたメモリ領域の内容をソフトウェアで読み込んで確認(Verify)することが推奨されています。

実際には、任天堂のソフトウェアの場合は、書き込み・消去でエラーが発生した場合に最大3回まで動作を繰り返すのが一般的です。<sup>[1](#sst)</sup>

<sup id="sst">1: SST製チップのみ、消去コマンドを最大80回繰り返し、リトライが不要な場合はさらに1回、それ以外の場合はさらに6回消去コマンドを繰り返します。</sup>

## コマンド

### チップの検出

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=90h  (enter ID mode)
  dev=[E000001h], man=[E000000h]                  (get device & manufacturer)
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=F0h  (terminate ID mode)
```

IDは `dev << 8 | man` で得られます。

### (複数バイト単位の)データの読み出し

```
  dat=[E00xxxxh]                                  (read byte from address xxxx)
```

### データの消去

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=80h  (erase command)
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=10h  (erase entire chip)
  wait until [E000000h]=FFh (or timeout)
```

チップ内の全メモリを消去し、消去されたメモリは`0xFF`で埋められます。

### 4KBセクタ単位のデータ消去 (Atmel以外)

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=80h  (erase command)
  [E005555h]=AAh, [E002AAAh]=55h, [E00n000h]=30h  (erase sector n)
  wait until [E00n000h]=FFh (or timeout)
```

`E00n000h...E00nFFFh`のメモリを消去し、消去されたメモリは0xFFで埋められます。

### 128Bセクタ単位の消去と書き込み (Atmelのみ)

```
  old=IME, IME=0                                  (disable interrupts)
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=A0h  (erase/write sector command)
  [E00xxxxh+00h..7Fh]=dat[00h..7Fh]               (write 128 bytes)
  IME=old                                         (restore old IME state)
  wait until [E00xxxxh+7Fh]=dat[7Fh] (or timeout)
```

コマンドフェーズ・ライトフェーズでは、割り込み（およびDMA）を無効にする必要があります。ターゲットアドレスは、0x80の倍数でなければなりません。

### 1バイトの書き込み (Atmel以外)

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=A0h  (write byte command)
  [E00xxxxh]=dat                                  (write byte to address xxxx)
  wait until [E00xxxxh]=dat (or timeout)
```

対象となるメモリアドレスは、事前に消去されている必要があります。

**Terminate Command after Timeout (only Macronix devices, ID=1CC2h)**

```
  [E005555h]=F0h                            (force end of write/erase command)
```

Macronixチップのみ、"wait until"期間中にタイムアウトが発生した場合に使用します。

### バンクの切り替え (64KBより大きいチップのみ)

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=B0h  (select bank command)
  [E000000h]=bnk                                  (write bank number 0..1)
```

読み出し/書き込み/消去 操作の対象の64Kバンク番号を指定します。カートリッジのFLASH/SRAMのアドレスバスが16bit幅に制限されているため必要です。
