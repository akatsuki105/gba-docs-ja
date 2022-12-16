# ALU

## 📜 命令のフォーマット

### I=0

このとき、2個目のオペランドにはレジスタを指定します

 bit  |  内容
---- | ----
31-28 | 条件
27-26 | ALU命令では必ず0
25 | 0 (I - 2個目のオペランドにレジスタ)
24-21 | オペコード（後述） (0x0-0xf)
20 | S - Set Condition Codes (0=No, 1=Yes) (Must be 1 for opcode 8-B)
19-16 | Rn - 第一オペランドのレジスタ番号 (R0..R15) MOV/MVNのときは必ず0になる
15-12 | Rd - 演算結果を格納するレジスタ番号 (R0..R15) CMP/CMN/TST/TEQ{P}のときは必ず0(または15)になる
11-7 | 後述
6-5 | シフトの種類 (0=LSL, 1=LSR, 2=ASR, 3=ROR)
4 | R - シフト量を示すのにレジスタを使うか即値を使うか (0=即値, 1=レジスタ)
3-0 | Rm - 2個目のオペランドとなるレジスタの番号

11-7bitは

- Rが0のとき: シフト量を表す即値(1-31, 0は後述)
- Rが1のとき: bit11-8はシフト量が入っているレジスタ、 bit7は必ず0になります

**アセンブリの例**

```asm
シフト量がレジスタ:
  and r1, r2, r3, lsl r4  ; r1 = r2 & (r3 << r4)

シフト量が即値:
  and r1, r2, r3, lsl #2  ; r1 = r2 & (r3 << 0x2)
```

### I=1

このとき、2個目のオペランドには即値を指定する

 bit  |  内容
---- | ----
31-28 | 条件
27-26 | ALU命令では必ず0
25 | 1 (I - 2個目のオペランドに即値)
24-21 | オペコード（後述） (0x0-0xf)
20 | S - Set Condition Codes (0=No, 1=Yes) (Must be 1 for opcode 8-B)
19-16 | Rn - 第一オペランドのレジスタ番号 (R0..R15) MOV/MVNのときは必ず0になる
15-12 | Rd - 演算結果を格納するレジスタ番号 (R0..R15) CMP/CMN/TST/TEQ{P}のときは必ず0(または15)になる
11-8 | Is - nnをRORシフトする量(0-30)
7-0 | nn - 2個目のオペランドとなる8bitの即値

**アセンブリの例**

```asm
バイナリが 03 13 02 E2 のとき:
  and r1, r2, #0xc000000 ; r1 = r2 & ROR(r3, 3)
```

### オペコード(24-21bit)

 値  |  フォーマット | 処理内容
---- | ---- | ----
0x00 | AND{cond}{S} Rd,Rn,Op2 | Rd = Rn AND Op2
0x01 | EOR{cond}{S} Rd,Rn,Op2 | Rd = Rn XOR Op2
0x02 | SUB{cond}{S} Rd,Rn,Op2 | Rd = Rn-Op2
0x03 | RSB{cond}{S} Rd,Rn,Op2 | Rd = Op2-Rn
0x04 | ADD{cond}{S} Rd,Rn,Op2 | Rd = Rn+Op2
0x05 | ADC{cond}{S} Rd,Rn,Op2 | Rd = Rn+Op2+Cy
0x06 | SBC{cond}{S} Rd,Rn,Op2 | Rd = Rn-Op2+Cy-1
0x07 | RSC{cond}{S} Rd,Rn,Op2 | Rd = Op2-Rn+Cy-1
0x08 | TST{cond}{P}    Rn,Op2 | Void = Rn AND Op2
0x09 | TEQ{cond}{P}    Rn,Op2 | Void = Rn XOR Op2
0x0a | CMP{cond}{P}    Rn,Op2 | Void = Rn-Op2
0x0b | CMN{cond}{P}    Rn,Op2 | Void = Rn+Op2
0x0c | ORR{cond}{S} Rd,Rn,Op2 | Rd = Rn OR Op2
0x0d | MOV{cond}{S} Rd,Op2    | Rd = Op2
0x0e | BIC{cond}{S} Rd,Rn,Op2 | Rd = Rn AND NOT Op2
0x0f | MVN{cond}{S} Rd,Op2    | Rd = NOT Op2

## 🔴 2個目のオペランド(Op2)

25bitや11-0bitを見るとわかるように2個目のオペランドは基本的に、レジスタ値、シフトされたレジスタ値、またはシフトされた即値 となります。

また、レジスタ値のシフト量は、レジスタ値の場合と即値の場合があります。

- レジスタ値: Op2は`Rm`と表記されます、アセンブラでは`Rm,LSL#0`と表されます。
- シフトされたレジスタ値: `Rm,SSS#Is` または `Rm,SSS Rs`と表されます (SSS=LSL/LSR/ASR/ROR)
- 即値: `#000NN000h`のように表します。(このとき `#0NNh,ROR#0ssh`), for example: "#000NN000h", assembler should automatically convert into "#0NNh,ROR#0ssh" as far as possible (ie. as far as a section of not more than 8bits of the immediate is non-zero).

## 🔴 シフト量が0のとき

2個目のオペランドがレジスタで、シフト量を即値で表す場合、シフト量が0のときは特殊な処理になります。

```
 LSL#0: シフトは行われません。 つまり、Op2=Rmとなりキャリーの値は不変です。
 LSR#0: LSR#32として解釈されます。つまり、Op2が0になったとき、キャリーはRmの31bitが入ります。
 ASR#0: ASR#32として解釈されます。つまり、Op2とキャリーはRmの31bitが入ります。
 ROR#0: ROR#1 と同様に RRX#1 (RCR) と解釈されますが、Op2の31bitは古いキャリーの値になります。
```

## Using R15 (PC)

R15をターゲットレジスタ(Rd)として使用する場合は、以下のCPSRの説明と実行時間の説明に注意してください。

R15をオペランド(Rm or Rn)として使う場合、R15の値は次のようになります。

```
I=0,R=1 のとき:
  R15 => PC+12

それ以外のとき:
  R15 => PC+8
```

## 🚩 フラグの変更

###  S=1, Rd≠R15, 論理命令 のとき:

該当するのは、AND,EOR,TST,TEQ,ORR,MOV,BIC,MVN
  
- V = 不変
- C = シフト処理の場合のみ、キャリー (LSL#0 や Rs=0x00 の場合は不変)
- Z = 演算結果がゼロ
- N = 演算結果がマイナス(bit31が1)

### S=1, Rd≠R15, 算術命令 のとき:

該当するのは、SUB,RSB,ADD,ADC,SBC,RSC,CMP,CMN

- V = 演算結果がオーバーフローしたか
- C = 演算結果のキャリー
- Z = 演算結果がゼロ
- N = 演算結果がマイナス(bit31が1)

### S=1, with unused Rd bits=1111b, オペコードが CMPP/CMNP/TSTP/TEQP のとき:

- In user mode only N,Z,C,V bits of R15 can be changed.
- In other modes additionally I,F,M1,M0 can be changed.
- The PC bits in R15 are left unchanged in all modes.

### S=1, Rd=R15 のとき:

ユーザーモード以外からユーザーモードに復帰する用途などで用いる

- CPSR = `SPSR_${current_mode}`
- PC = 処理結果

例:

```
  MOVS PC,R14  ; SWIからの復帰 (PC=R14_svc, CPSR=SPSR_svc)
```

### S=0 のとき

フラグは全部不変

## 🔎 補足

**実行時間**

実行時間: (1+p)S+rI+pN

- r: 1 (`I==0 && R==1` つまりシフト量にレジスタの値を使う場合) or 0(それ以外)
- p: 1 (`Rd==R15`) or 0(それ以外)

**NOP**

ARMステートでは`MOV R0, R0`を`NOP`命令として扱う

