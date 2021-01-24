# DMA転送

GBAには4つのDMAチャンネルがあり、優先度は DMA0が最高、DMA1、DMA2と続いて、DMA3が最低となっています。優先度の低いDMAチャンネルは、優先度の高いチャンネルが完了するまで一時停止されます。

DMA転送がアクティブなときはCPUは一時停止しますが、`Sound/Blanking DMA転送`が一時停止している間はCPUは動作しています。

## それぞれのチャネルの役割

- DMA0: 優先度が最も高く、HBlank DMAなど時間にシビアな転送をしたいときにベストです
- DMA1とDMA2: can be used to feed digital sample data to the Sound FIFOs
- DMA3: Game Pak ROM/FlashROMに書き込みをする際に使われます(GamePak SRAMは除く)

加えて、全部のチャネルは汎用的なデータ転送用途にも使うことができます。

## ソースレジスタ(SAD)

DMA転送で転送するデータが始まるアドレスを指定します。

```
0x0400_00B0 & 0x0400_00B2 - DMA0SAD - DMA0ソースレジスタ (W) (内部メモリのみ)
0x0400_00BC & 0x0400_00BE - DMA1SAD - DMA1ソースレジスタ (W) 
0x0400_00C8 & 0x0400_00CA - DMA2SAD - DMA2ソースレジスタ (W) 
0x0400_00D4 & 0x0400_00D6 - DMA3SAD - DMA3ソースレジスタ (W) 
```

DMA0の場合は`0x07ff_ffff`まで、DMA1,DMA2,DMA3の場合は`0x0fff_ffff`まで指定することが可能です。

## ターゲットレジスタ(DAD)

DMA転送でデータ転送先のアドレスを指定します。

```
0x0400_00B4 & 0x0400_00B6 - DMA0DAD - DMA0ターゲットレジスタ (W) (内部メモリのみ)
0x0400_00C0 & 0x0400_00C2 - DMA1DAD - DMA1ターゲットレジスタ (W) (内部メモリのみ)
0x0400_00CC & 0x0400_00CE - DMA2DAD - DMA2ターゲットレジスタ (W) (内部メモリのみ)
0x0400_00D8 & 0x0400_00DA - DMA3DAD - DMA3ターゲットレジスタ (W)
```

DMA0, DMA1, DMA2の場合は`0x07ff_ffff`まで、DMA3の場合は`0x0fff_ffff`まで指定することが可能です。

## ワードカウントレジスタ(CNT_L)

```
0x0400_00B8 - DMA0CNT_L - DMA0ワードカウント (W) (14 bit, 0x1..0x4000)
0x0400_00C4 - DMA1CNT_L - DMA1ワードカウント (W) (14 bit, 0x1..0x4000)
0x0400_00D0 - DMA2CNT_L - DMA2ワードカウント (W) (14 bit, 0x1..0x4000)
0x0400_00DC - DMA3CNT_L - DMA3ワードカウント (W) (16 bit, 0x1..0x10000)
```

一度に転送されるデータのサイズを指定するためのレジスタです。 転送タイプによって1ワードカウントで16bit(2byte)転送するのか32bit(4byte)転送するのかが変わります。0を入れた場合最大値(0x4000 or 0x10000)として扱われます

## 制御レジスタ

```
0x0400_00BA - DMA0CNT_H - DMA0制御レジスタ (R/W)
0x0400_00C6 - DMA1CNT_H - DMA1制御レジスタ (R/W)
0x0400_00D2 - DMA2CNT_H - DMA2制御レジスタ (R/W)
0x0400_00DE - DMA3CNT_H - DMA3制御レジスタ (R/W)
```

 bit  |  内容
---- | ----
0-4 | 未使用
5-6 | ターゲットアドレス制御 (0=Increment,1=Decrement,2=Fixed,3=Increment/Reload)
7-8 | ソースアドレス制御 (0=Increment,1=Decrement,2=Fixed,3=Prohibited)
9 | DMAリピート                   (0=Off, 1=On) (Must be zero if Bit 11 set)
10 | DMA転送タイプ (0=16bit, 1=32bit)
11 | Game Pak DRQ  - DMA3 only -  (0=Normal, 1=DRQ from Game Pak, DMA3)
12-13 | DMA開始タイミング (0=即座に, 1=VBlank時, 2=HBlank時, 3=※)
14 | IRQ upon end of Word Count   (0=Disable, 1=Enable)
15 | DMA有効フラグ (0=無効, 1=有効)

※ DMAチャネルごとに異なります。 DMA0=Prohibited, DMA1/DMA2=Sound FIFO, DMA3=Video Capture

DMA有効フラグ(bit15)を0から1に変更した後、DMA関連レジスタにアクセスする前に2クロックサイクル待つ必要があります。

HBlankタイミングでOAM(0x0700_0000)やOBJ VRAM(0x0601_0000)にアクセスする場合は、DISPCNTレジスタの "H-Blank Interval Free"ビットをセットする必要があります。

## ソースレジスタ、ターゲットレジスタ、ワードカウントレジスタ

ソースレジスタ、ターゲットレジスタ、ワードカウントレジスタは初期開始アドレスと初期長を保持しています。ハードウェアは転送中または転送後にこれらのレジスタの内容を変更することはありません。

実際の転送は、内部ポインタ/カウンタレジスタを使用して行われます。初期値は以下の状況下で内部レジスタにコピーされます:

DMA有効フラグ(bit15)を0から1に変更すると、SAD, DAD, CNT_Lはリロードされます。

リピート時にはCNT_Lがリロードされ、DADは任意でリロードを行えます。(DADはIncrement+Reload)

## DMAリピートビット(CNT_Hのbit9)

リピートビットがクリアされている場合: 指定したデータ単位数を転送すると、DMA有効フラグは自動的にクリアされます。

リピートビットがセットされている場合: 転送後もDMA有効フラグはセットされたままで、スタート条件（例：HBlank、Fifo）がtrueになるたびに転送が再開されます。

転送開始時には、毎回指定されたデータ単位数を転送します。転送はソフトウェアで停止するまで永遠に繰り返されます。

## サウンドDMA (FIFO Timing Mode) (DMA1/DMA2のみ)

このモードではDMAリピートビットをセットし、転送先アドレスをFIFO_A（0x0400_00A0）またはFIFO_B（0x0400_00A4）に設定する必要があります。

サウンドコントローラからのDMA要求により、32ビットを1単位として4単位分（合計16バイト）転送します（ワードカウントレジスタとDMA転送タイプビットは無視されます）。

FIFOモードでは転送先アドレスはインクリメントされません。

優先度の高いDMAチャンネルは、サウンドDMAを途中でオフにする場合があることを覚えておいてください。例えば、64kHzのサンプルレートを使用している場合、16バイトのサウンドDMAデータが 0.25ms(4kHz)ごとに要求されます。(English: Keep in mind that DMA channels of higher priority may offhold sound DMA. For example, when using a 64 kHz sample rate, 16 bytes of sound DMA data are requested each 0.25ms (4 kHz), at this time another 16 bytes are still in the FIFO so that there's still 0.25ms time to satisfy the DMA request. )

したがって、より高い優先度を持つDMAは、0.25msよりも長い間操作されるべきではありません。(HBlank時間は16.212usに制限されているため、この問題はHBlank転送では発生しません)

## Game Pak DMA

DMA 3 のみが Game Pak ROMまたは Flash ROMとの間のデータ転送に使用されますが、Game Pak SRAMにはアクセスできません。（SRAMデータバスが8bit単位に制限されているため）

通常モードでは、ワードカウントが0になるまでDMAがリクエストされます。

Game Pack DRQビットを設定する場合、カートリッジには/DREQ信号を出力する外部回路が必要です。

DREQと/IREQのピンは1つしかないため、DRQモードを使用している間はカートリッジから/IREQが供給されない可能性があります。

## ビデオキャプチャーモード(DMA3のみ)

メモリ（または外部ハードウェア/カメラ）から VRAM にビットマップをコピーすることを目的としています。

この転送モードを使用する場合は、リピートビットを設定し、ワードカウントレジスタにスキャンラインあたりのデータ単位数を書き込みます。

キャプチャは HBlank DMA と同様に動作しますが、VCOUNT=2 で転送が開始され、スキャンラインごとに繰り返され、VCOUNT=162 で停止します。

## 転送終了

DMA有効フラグ（bit15）は、転送が完了すると自動的にクリアされます。

ユーザーは、転送を停止するためにこのビットを手動でクリアすることもできます（明らかに、これはサウンド/Blank DMAの場合のみ可能です。)

## Transfer Rate/Timing

最初のデータ単位をのぞいて、すべてのデータ単位はシーケンシャルな読み書きによって転送されます。

n個のデータ単位があるとき、DMA転送に要する時間は、2N+2(n-1)S+xI です。

このうち、1N+(n-1)SがReadサイクル、残りの1N+(n-1)SがWriteサイクルとなりますが、実際のサイクル数は転送元エリアと転送先エリアのWaitStateとバス幅に依存します(CPU命令サイクルタイムの章で説明します)。

DMA処理にかかる内部時間は 2I(通常) か 4I(転送元と転送先がGamePakメモリのとき)です。

DMA lockup when stopping while starting ???
Capture delayed, Capture Enable=AutoCleared ???
