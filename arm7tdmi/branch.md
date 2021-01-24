# 分岐命令

B, BL, BX, BLX, SWI, BKPTが該当します。

## 分岐命令(B, BL, BLX_imm)

Bはサブルーチンへのジャンプをおこないます。

BL, BLX_immはサブルーチンの`コール`つまりリターンアドレスをR14に退避してサブルーチンへジャンプします。

 ビット | 内容
---- | ---- 
31-28 | 条件(BLXのときは必ず0b1111)
27-25 | 必ず0b101
24 | オペコード(後述)
23-0 | nn - 符号付きオフセット(1ごとにつき4バイト)

オペコードは
- 0: `B{cond} label     ; PC=PC+8+nn*4`
- 1: `BL{cond} label    ; PC=PC+8+nn*4, LR=PC+4`
- 2: `BLX label         ; PC=PC+8+nn*4+H*2, LR=PC+4, T=1`

実行時間: 2S + 1N

フラグは全部不変です。

## ソフトウェア割り込み命令(SWI/BKPT)

SWIはOSの機能を呼び出してARMステートのSVCモードに入るための命令です。 (SWI = SoftWare Interrupt)

BKPTはプリフェッチアボートを通してARMステートのABTモードに入るための命令でデバッグ目的で使われます。(BKPT = BreaKPoinT)

### SWI

 ビット | 内容
---- | ---- 
31-28 | 条件
27-24 | 0b1111 SWI{cond} nn
23-0 | nn - コメントフィールドでプロセッサには無視されます

実行時間: 2S+1N

### BKPT

 ビット | 内容
---- | ---- 
31-28 | 0b1110
27-24 | 0b0001 BKPT      nn
23-20 | 0b0010
19-8 | nn - コメントフィールドの上位12bitでプロセッサには無視されます
7-4 | 0b0111
3-0 | nn - コメントフィールドの下位4bitでプロセッサには無視されます

実行時間: 2S+1N

例外ハンドラは`[R14_svc - 4]`に退避されたSWI命令の下位24bitのコメントフィールドを自由に使うことが可能です。

もしTHUMBステートでSWI命令を使った場合は、SWIハンドラは`SPSR_svc`のTを見て、SWI命令がTHUMBステートから使われたことを理解し、`[R14_svc - 2]`の指す16bitのSWI命令のうち、下位8bitをコメントフィールドとして利用します。

SWIからリターンするときは`MOVS PC,R14`を実行してPCとCPSRを復帰します。(`PC=R14_svc と CPSR=SPSR_svc`)

Nesting SWIs: SPSR_svc and R14_svc should be saved on stack before either invoking nested SWIs, or (if the IRQ handler uses SWIs) before enabling IRQs.

SWIがネストするときにはSWIする前に`SPSR_svc`と`R14_svc`をスタックに退避したり、(IRQハンドラがSWIを使う場合は)IRQを有効にする必要があります。

SWI/BKPT実行時の処理

```
  R14_svc=PC+4  or   R14_abt=PC+4         ; 復帰先のアドレスを退避
  SPSR_svc=CPSR  or  SPSR_abt=CPSR        ; CPSRを退避
  CPSR=\${changed} or  CPSR=\${changed}   ; ARMステートのSVC/ABTモードに入りIRQを無効にする
  PC=VVVV0008h  or   PC=VVVV000Ch         ; SWI または プリフェッチアボート のベクタにジャンプする
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
