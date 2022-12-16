# システム制御

## 0x0400_0204 - WAITCNT - Waitstate制御レジスタ (R/W)

このレジスタは、カートリッジのアクセスタイミングを設定するために使用されます。

カートリッジのROMは `0x0800_0000`, `0x0a00_0000`, `0x0c00_0000` の3つのアドレス領域にミラーリングされており、これらの領域にCPUがアクセスするのに要する時間を WaitState0, WaitState1, WaitState2 と呼びます。

それぞれの領域に異なるアクセス時間を割り当てることができます。

WaitStateについては[こちら](arm7tdmi/cycle.md)を参照してください。

 bit  |  内容
----- | -----
0-1   | SRAM Wait Control          (0..3 = 4,3,2,8 cycles)
2-3   | N0 - WaitState0 ファーストアクセス  (0..3 = 4,3,2,8 cycles)
4     | S0 - WaitState0 セカンドアクセス (0..1 = 2,1 cycles)
5-6   | N1 - WaitState1 ファーストアクセス  (0..3 = 4,3,2,8 cycles)
7     | S1 - WaitState1 セカンドアクセス (0..1 = 4,1 cycles)
8-9   | N2 - WaitState2 ファーストアクセス  (0..3 = 4,3,2,8 cycles)
10    | S2 - WaitState2 セカンドアクセス (0..1 = 8,1 cycles)
11-12 | PHI Terminal Output        (0..3 = Disable, 4.19MHz, 8.38MHz, 16.78MHz)
13    | 未使用
14    | カートリッジのプリフェッチが有効かどうか (0=無効, 1=有効)
15    | Game Pak Type Flag  (Read Only) (0=GBA, 1=CGB) (IN35 signal)
16-31 | 未使用

WAITCNTは起動時には0x0000_0000です。

そしてその後、現在製造されているカートリッジでは `WAITCNT=0x0000_4317=0b0000_0000_0000_0000_0100_0011_0001_0111` となります。これは

```
  SRAM:        8サイクル
  N0, N1, N2:  3, 4, 8サイクル
  S0, S1, S2:  1, 4, 8サイクル
```

となります。 実際のカートリッジのアクセスにはこれに1サイクル加えたものがアクセスに要するサイクルになります。(SRAMはそのまま8サイクル)

また、カートリッジのデータバスは16bitなので、32bitアクセスは2回の16bitアクセスに分割されて行われます。 最初の16bitがノンシーケンシャルであっても、2回目の16bitは常にシーケンシャルです

注意: GBAはカートリッジROMの各128Kブロックの先頭にアクセスする際には強制的にノンシーケンシャルアクセスします。 またPHI端子出力(Gamepak BusのPHIピン)は無効にしてください。

## 0x0400_0300 - POSTFLG - BYTE - Undocumented - Post Boot / Debug Control (R/W)

最初のリセット後、BIOSはこのレジスタを0x01に初期化しリセットベクタ(0x000000)をさらに実行します。

このときレジスタがまだ0x01に設定されていることを感知すると、デバッグベクタ(0x0000_001c)に制御を渡します。

 bit  |  内容
----- | -----
0 | Undocumented. First Boot Flag  (0=First, 1=Further)
1-7 | Undocumented. Not used.

通常、デバッグハンドラは、カートリッジヘッダでデバッグフラグを検出しない限り制御を受け付けませんが、その場合は、(任天堂ロゴとブート遅延を省略することで) 簡略化されたブート処理へとリダイレクトします 

しかし、このレジスタを自動的にリセットせずにGBAを外部からリセットすることは可能なのかどうかはわかりません。

## 0x0400_0301 - HALTCNT - BYTE - Undocumented - 省電力モード制御レジスタ (W)

このレジスタに書き込みを行うことでGBAは省電力モードに移行します。

Haltモードでは、`IE & IF = 0`の限りCPUが停止し続けます。 これはCPUが割り込みを待つ間に電力消費を抑えるために使われます。

省電力モードは、サウンドや画面を含めたほとんどのハードウェアが停止します。

 bit  |  内容
----- | -----
0-6 | 未使用
7   | 省電力モード  (0=Halt, 1=Stop)

**基本的に、このレジスタに直接書き込むよりも BIOS関数 SWI2(Halt) または SWI3(Stop) を使用することが一般的に推奨されます。**

## 0x0400_0410 - Undocumented - 用途不明(1byte) (W)

BIOSはこのアドレスに0xFFを書き込みますが目的は不明です。

おそらくBIOSのバグでしょう。

## 4000800h - 32bit - Undocumented - Internal Memory Control (R/W)

ハードウェアによって`0x0d00_0020`で初期化されます。

他のIOレジスタと違って、このレジスタはIOレジスタ用のメモリ領域にミラーされます。(ここから0x0001_0000刻み。 つまり0x0400_0800, 0x0401_0800, 0x0402_0800, ..., 0x04FF_0800)

 bit  |  内容
----- | -----
0     | Disable 32K+256K WRAM (0=Normal, 1=Disable) (when off: empty/prefetch)
1-3   | Unknown          (Read/Write-able)
4     | Unknown          (Always zero, not used or write only)
5     | Enable 256K WRAM (0=Disable, 1=Normal) (when off: mirror of 32K WRAM)
6-23  | Unknown          (Always zero, not used or write only)
24-27 | Wait Control WRAM 256K (0-14 = 15..1 Waitstates, 15=Lockup)
28-31 | Unknown          (Read/Write-able)

bit24-27にはデフォルトで0x0dが入っていてWaitstateが2になるようになっています。 つまり 8/16/32bitアクセスに要するサイクルは 3/3/6となります。

bit24-27を0x0eにせっていすればWaitStateは1なので、8/16/32bitアクセスに要するサイクルは 2/2/4 となり最も高速にアクセスできます。 この設定はGBAとGBASPでは可能ですがGBA Microではこの値は固定なので変更不可能です。
