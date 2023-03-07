# 分岐命令

## THUMB.16: 条件付き分岐

ビット | 内容
-- | --
15-12 | 0b1101
11-8 | オペコード
7-0 | 符号付きオフセット(2バイトごと)

オペコード(bit11-8):

```
0: BEQ label  ; Z=1 EQual
1: BNE label  ; Z=0 NotEqual
2: BCS label  ; C=1 CarrySet
3: BCC label  ; C=0 CarryClear
4: BMI label  ; N=1 MInus
5: BPL label  ; N=0 PLus
6: BVS label  ; V=1 VSet
7: BVC label  ; V=1 VClear
8: BHI label  ; C=1 && Z=0
9: BLS label  ; C=0 || Z=1
a: BGE label  ; N=V GreaterEqual
b: BLT label  ; N=V LessThan
c: BGT label  ; N=V GreaterThan
d: BLE label  ; N=V LessEqual
e: 未使用
f: SWIのために予約
```

フラグ: 不変

実行時間: 

- true: 2S+1N
- false: 1S

## THUMB.17: ソフトウェア割り込み \& ブレーク

SWIはOSの機能を呼び出す際に使用されます。 

BKPTはプリフェッチアボートによってアボートモードに入るのに使います。デバッグのための命令です。

ビット | 内容
-- | -- 
15-8 | オペコード - 0b1101_1111(SWI nn) or 0b1011_1110(BKPT nn)
7-0 | nn - コメントフィールド

SWIのコメントフィールドは例外ハンドラの処理を分岐させるのに使用します。例外ハンドラは`[R14_svc-2]`に格納されたSWI命令をパースすることによってこのコメントフィールドを得ます。

もしARMモードでSWIを使用する可能性があるなら、SWIハンドラはSPSR_svcのTビットをみて、ARMモードで起きたSWIかを判定して、そうだとしたら`[R14_svc-4]`の24-32bitからパースを行う必要があります。

SWIからの復帰には `MOVS PC,R14`を使用しますが、この命令はPCとCPSRの両方をリストア(つまりPC=R14_svc、CPSR=SPSR_svc)し、（THUMBモードから呼び出された場合は）THUMBモードへの復帰もおこないます。

ネストしたSWIを呼び出す場合は、SWIを呼び出す前またはIRQを有効にする前に現在の`SPSR_svc`と`R14_svc`をスタックに退避して呼び出す必要があります。

実行時間: 2S+1N

## THUMB.18: 無条件分岐

`B label`のこと

ビット | 内容
-- | --
15-11 | 0b11100
10-0 | ジャンプ先(PCからの相対オフセット、2バイトごと)

フラグ: 不変

実行時間: 2S+1N

## THUMB.19: サブルーチンコール

この命令はリターン先がR14になっているサブルーチンのコール(or ジャンプ)に利用されます。

THUMBモードの命令は16bitですが、この命令は32bit長です。<sup>[1](#exception)</sup> <sup>[2](#bl)</sup>

**1つ目(0-15)**

LR = PC+4+(nn SHL 12)

ビット | 内容
-- | --
15-11 | 0b11110
10-0 | nn - ターゲットアドレスの上位11bit

**2つ目(16-31)**

PC = LR + (nn SHL 1), and LR = PC+2 OR 1 (and BLX: T=0)

ビット | 内容
-- | --
15-11 | オペコード - 0b11111(BL label) or 0b11101(BLX label)
10-0 | nn - ターゲットアドレスの下位11bit

この命令では `PC±0x40_0000`の範囲に飛ぶことが可能です。

フラグ: 不変

実行時間: 3S+1N (1つ目: 1S, 2つ目: 2S+1N)

<sup id="exception">1: 1つ目のオペコードと2つ目のオペコードの間に例外が起きるかどうかは未定義です。</sup>  
<sup id="bl">2: `BL LR+imm`として2つ目のオペコードだけを使うことも可能です。</sup>

## BX

[THUMB.5: Hi register operations/branch exchange](./register.md#thumb5-hi-register-operationsbranch-exchange)を参照
