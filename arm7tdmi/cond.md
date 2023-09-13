# CPUフラグと条件フィールド

## 条件フィールド(cond)

オペコードの末尾に`{cond}`をつけるとCPSRレジスタのC,N,Z,Vフラグに基づいて実行されるかされないかが変わってきます。

例えば、`BEQ`は `Branch if Equal` の略でZフラグが立っているときに分岐し、`MOVMI`は `Move if Singed` の略でNフラグが立っている時に実行されます。

ARMステートのとき、`{cond}`はすべてのオペコードで使うことができます。 THUMBステートでは`{cond}`は分岐命令でのみ使用できます。

 コード  |  ニーモニック | フラグ | 内容
---- | ---- | ---- | ----
0 | EQ | Z=1 | 等価、0
1 | NE | Z=0 | 値が異なる、非0
2 | CS/HS | C=1 |  unsigned higher or same (carry set)
3 | CC/LO | C=0         | unsigned lower (carry cleared)
4 | MI    | N=1         | 前の処理結果が負
5 | PL    | N=0         | 前の処理結果が0以上
6 | VS    | V=1         | signed overflow (V set)
7 | VC    | V=0         | signed no overflow (V cleared)
8 | HI    | C=1 and Z=0 | unsigned higher
9 | LS    | C=0 or Z=1  | unsigned lower or same
A | GE    | N=V         | signed greater or equal
B | LT    | N≠V        | signed less than
C | GT    | Z=0 and N=V | signed greater than
D | LE    | Z=1 or N≠V | signed less or equal
E | AL    | -           | always (the "AL" suffix can be omitted)
F | NV    | -           | never (ARMv1,v2 only) (Reserved ARMv3 and up)

実行時間

- 条件が満たされた時: それぞれのオペコードの実行時間
- 条件が満たされなかった時: 1S

## CPSR

 bit  |  内容
---- | ----
0-4 | M0-M4 - モード(後述)
5 | T - THUMBフラグ (0=ARM, 1=THUMB)
6 | F - FIQ無効フラグ (1=FIQ無効)
7 | I - IRQ無効フラグ (1=IRQ無効)
8-26 | 予約 
27 | Q - Stickyフラグ (GBAでは未使用)
28 | V - Overflowフラグ (1=Overflow)
29 | C - Carryフラグ (1=Carry/No Borrow)
30 | Z - Zeroフラグ (1=Zero)
31 | N - Signフラグ (1=Signed)

### Bit28-31(NZCV)

これらのビットは、論理命令または算術命令の結果を反映します。

ARMステートでは、ある命令がフラグを修正するかどうかは任意であることが多く、例えば、条件フラグを修正しないSUB命令を実行することも可能です。

ARMステートでは、`MOVEQ（if Z=1, MOV）`のように、フラグの設定次第で全ての命令を条件付きで実行することができます。THUMBステートでは、分岐命令（ジャンプ）のみ条件付きで実行できます。

### Bit27(Q)

>**Warning** ARMv5TE/ARMv5TExP 以降のみ有効(GBAはサポートしていません)

QADD、QSUB、QDADD、QDSUB、SMLAxy、SMLAWy でのみ使用されます。

これらのオペコードは、オーバーフローが発生した場合にQフラグをセットしますが、それ以外は変更しない。

QフラグはMSR/MRSオペコードによってのみテスト/リセットできます。

### Bit0-7 制御ビット

これらのbitは例外が起きた時に変わる可能性があります。特権モード(非ユーザモード)では手動で変更することができます。

IとFはIRQとFIQ割り込みをそれぞれ無効にするのに使用します。

TはCPUのステートを表し、決して手動で変えてはいけません。ステートを変えたい時はBX命令を使うようにしてください。

M4-M0は現在の動作モードを表します。

 bit  |  モード | 備考
 ---- | ---- | ----
0b0xx00 | Old User | 26bit Backward Compatibility modes
0b0xx01 | Old FIQ | supported only on ARMv3, except ARMv3G, and on some non-T variants of ARMv4
0b0xx10 | Old IRQ | supported only on ARMv3, except ARMv3G, and on some non-T variants of ARMv4
0b0xx11 | Old Supervisor | 
0b10000 | User | これだけ特権モードではない
0b10001 | FIQ |
0b10010 | IRQ |
0b10011 | Supervisor (SWI) |
0b10111 | Abort |
0b11011 | Undefined | 
0b11111 | System | 特権"ユーザ"モード

M0-M4を上記の値以外にすることはできません。

## SPSR (SPSR_${mode})

`Saved Program Status Registers`の略

上記のCPSRに加えて5つのSPSRが存在します: `SPSR_fiq, SPSR_svc, SPSR_abt, SPSR_irq, SPSR_und`

CPUが例外を起こすと、現在のステータスレジスタ（CPSR）が対応するSPSRレジスタにコピーされます。

各モードには1つのSPSRしかないので、同じモード内でネストした例外は、例外ハンドラがSPSRの内容をスタックに退避している場合にのみ許可されることに注意してください。

例えば、IRQ例外の場合。IRQモードが入力され、CPSRがSPSR_irqにコピーされます。割り込みハンドラがネストしたIRQを有効にしたい場合は、その前にまずSPSR_irqをプッシュしなければなりません。
