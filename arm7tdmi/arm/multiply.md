# 乗算

MUL,

## 命令のフォーマット

### ワード単位

 bit  |  内容
---- | ----
31-28 | 条件
27-25 | 0b000
24-21 | オペコード
20 | S - Set Condition Codes (0=No, 1=Yes) (Must be 0 for Halfword & UMAAL)
19-16 | Rd or RdHi - 演算結果を格納するレジスタ
15-12 | Rn or RdLo - 演算レジスタ (使わないときは0b000)
11-8 | Rs - オペランドレジスタ
7-4 | 0b1001
3-0 | Rm - オペランドレジスタ

### ハーフワード単位

 bit  |  内容
---- | ----
31-28 | 条件
27-25 | 0b000
24-21 | オペコード
20 | 0b0
19-16 | Rd or RdHi - 演算結果を格納するレジスタ
15-12 | Rn or RdLo - 演算レジスタ (使わないときは0b000)
11-8 | Rs - オペランドレジスタ
7 | 0b1
6 | y - Rsの下位16bitと上位16bitのどちらを使うか(0=下位, 1=上位)
5 | x - Rmの下位16bitと上位16bitのどちらを使うか(0=下位, 1=上位)
4 | 0b0
3-0 | Rm - オペランドレジスタ

### オペコード(24-21bit)

 値  |  フォーマット | 処理内容 | 備考
---- | ---- | ---- | ----
0b0000 | MUL{cond}{S}   Rd,Rm,Rs | Rd=Rm*Rs | 符号関係なし
0b0001 | MLA{cond}{S}   Rd,Rm,Rs,Rn | Rd=Rm*Rs+Rn | 符号関係なし
0b0010 | UMAAL{cond}    RdLo,RdHi,Rm Rs | RdHiLo=Rm*Rs+RdHi+RdLo | 未サポート
0b0100 | UMULL{cond}{S} RdLo,RdHi,Rm,Rs | RdHiLo=Rm*Rs | unsigned
0b0101 | UMLAL{cond}{S} RdLo,RdHi,Rm,Rs | RdHiLo=Rm*Rs+RdHiLo | unsigned
0b0110 | SMULL{cond}{S} RdLo,RdHi,Rm,Rs | RdHiLo=Rm*Rs |
0b0111 | SMLAL{cond}{S} RdLo,RdHi,Rm,Rs | RdHiLo=Rm*Rs+RdHiLo | 
0b1000 | SMLAxy{cond}   Rd,Rm,Rs,Rn | Rd=HalfRm*HalfRs+Rn | 
0b1001 | SMLAWy{cond}   Rd,Rm,Rs,Rn | Rd=(Rm*HalfRs)/10000h+Rn | 
0b1001 | SMULWy{cond}   Rd,Rm,Rs | Rd=(Rm*HalfRs)/10000h | 
0b1010 | SMLALxy{cond}  RdLo,RdHi,Rm,Rs | RdHiLo=RdHiLo+HalfRm*HalfRs | 
0b1011 | SMULxy{cond}   Rd,Rm,Rs | Rd=HalfRm*HalfRs | 

## 各命令について

### MUL, MLA

MULとMLAには制約があり、RdとRmは同じレジスタであり、Rd,Rn,Rs,RmのどれもR15を指していてはいけません。

注：内部では64ビットで計算され、その結果の下位32ビットのみがRdに格納されるため、符号/ゼロ拡張は不要であり、MULとMLAは符号付きと符号なしの両方の計算に使用できます。

実行時間:

- MUL: 1S+mI
- MLA: 1S+(m+1)I

一方、'm' は、Rsが指すレジスタの値によって決まります。

Rsレジスタの

- 31-8bitが全て1: m=1
- 31-16bitが全て1: m=2
- 31-24bitが全て1: m=3
- それ以外: m=4

フラグ(S=1): 

- N: 計算結果が負
- Z: 計算結果が0
- C: クリア(ARMv4) or 不変(ARMv5以降)
- V: 不変

### UMULL, UMLAL, SMULL, SMLAL

- MULL = `Multiply Long`
- MLAL = `Multiply-Accumulate Long`

頭が`U`のときは符号なし、`S`のときは符号ありです。

RdHi, RdLo, Rmはすべて異なるレジスタである必要があり、R15を指すことはできないことに注意してください。

実行時間: 

- MULL: 1S+(m+1)I
- MLAL: 1S+(m+2)I

'm' は、Rsが指すレジスタの値によって決まります。

Rsレジスタの

- 31-8bitが全て1: m=1
- 31-16bitが全て1: m=2
- 31-24bitが全て1: m=3
- それ以外: m=4

フラグ(S=1): 

- N: 計算結果が負
- Z: 計算結果が0
- C: クリア(ARMv4) or 不変(ARMv5以降)
- V: 不変

### SMLAxy, SMLAWy, SMLALxy, SMULxy, SMULWy

**GBAではサポートされていません**(ARMv5TE(DS)以降からサポート)

これらは全て、符号付きのハーフワード単位の乗算命令です。

Qフラグは32bitのSMLAxy/SMLAWy加算オーバーフローで設定されますが、QADD同様、結果は切り捨てられません。

Qフラグは64bitのSMLALxy加算オーバーフローでは変更されません。

SMULxy/SMULWy ではオーバーフローしないので，Qフラグは不変です。

NZCVフラグはハーフワード乗算のとき不変です。

実行時間:

- SMULxy,SMLAxy,SMULWx,SMLAWx: 1S+Interlock
- SMLALxy: 1S+1I+Interlock

### UMAAL

**GBA・NDSではサポートされていません**(ARMv6以降からサポート)
