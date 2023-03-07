# 分岐命令

B, BL, BX, SWIが該当します。

## 分岐命令(B, BL)

Bはサブルーチンへのジャンプをおこないます。

BLはサブルーチンの`コール`つまりリターンアドレスをR14に退避してサブルーチンへジャンプします。

 ビット | 内容
---- | ---- 
31-28 | 条件
27-25 | 必ず0b101
24 | オペコード(後述)
23-0 | nn - 符号付きオフセット(1ごとにつき4バイト)

オペコードは

```
0: B{cond} label        ; PC=PC+8+nn*4
1: BL{cond} label       ; PC=PC+8+nn*4, LR=PC+4
```

フラグ: 不変

実行時間: 2S + 1N

## 特殊な分岐命令(BX)

 ビット | 内容
---- | ---- 
31-28 | 条件
27-8 | 0b0001_0010_1111_1111_1111
7-4 | 0b0001 (`BX{cond} Rn`)
3-0 | Rn - オペランドのレジスタ番号

```
BX{cond} Rn ; PC=Rn, T=Rn.0 
```

Rnのbit0を1にするとTHUMBモードにスイッチし、ジャンプ先は`Rn-1`になります。

`BX R15` は `BX $+8` と同じことになります。

フラグ: 不変

実行時間: 2S + 1N

## ソフトウェア割り込み命令(SWI)

SWIはOSの機能を呼び出してARMステートのSVCモードに入るための命令です。 (SWI = SoftWare Interrupt)

 ビット | 内容
---- | ---- 
31-28 | 条件
27-24 | 0b1111 `SWI{cond} nn`
23-0 | nn - コメントフィールドでプロセッサには無視されます

実行時間: 2S+1N

例外ハンドラは`[R14_svc - 4]`に退避されたSWI命令の下位24bitのコメントフィールドを自由に使うことが可能です。

もしTHUMBステートでSWI命令を使った場合は、SWIハンドラは`SPSR_svc`のTを見て、SWI命令がTHUMBステートから使われたことを理解し、`[R14_svc - 2]`の指す16bitのSWI命令のうち、下位8bitをコメントフィールドとして利用します。

SWIからリターンするときは`MOVS PC,R14`を実行してPCとCPSRを復帰します。(`PC=R14_svc と CPSR=SPSR_svc`)

SWIがネストするときにはSWIする前に`SPSR_svc`と`R14_svc`をスタックに退避したり、(IRQハンドラがSWIを使う場合は)IRQを有効にする必要があります。

SWI実行時の処理

```
  R14_svc=PC+4         ; 復帰先のアドレスを退避
  SPSR_svc=CPSR        ; CPSRを退避
  CPSR=${changed}      ; ARMステートのSVCモードに入りIRQを無効にする
  PC=VVVV0008h         ; SWI のベクタにジャンプする
```

## 未定義命令例外(UND)

 ビット | 内容
---- | ---- 
31-28 | 条件
27-25 | 0b011
24-5 | 未使用(将来使用予定)
4 | 0b1
3-0 | 未使用(将来使用予定)

アセンブラでは対応するニーモニックは存在しません。

- `cond011xxxxxxxxxxxxxxxxxxxx1xxxx` - 将来使用予定
- `cond01111111xxxxxxxxxxxx1111xxxx` - ユーザーが自由に使えます

実行時間: 2S+1I+1N.