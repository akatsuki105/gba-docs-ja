# ALU

## 命令のフォーマット

### 命令のフォーマット(31-12bit)

 bit  |  内容
---- | ----
31-28 | 条件
27-26 | ALU命令では必ず0
25 | I - 2個目のオペランドにレジスタを使うか即値を使うか (0=レジスタ, 1=即値)
24-21 | オペコード（後述） (0x0-0xf)
20 | S - Set Condition Codes (0=No, 1=Yes) (Must be 1 for opcode 8-B)
19-16 | Rn - 第一オペランドのレジスタ番号 (R0..R15) MOV/MVNのときは必ず0になる
15-12 | Rd - 演算結果を格納するレジスタ番号 (R0..R15) CMP/CMN/TST/TEQ{P}のときは必ず0(または15)になる

### 命令のフォーマット(12-0)

#### bit25(I)が0 つまり 2個目のオペランドがレジスタの場合

 bit | 内容
---- | ----
11-7 | 後述
6-5 | シフトの種類 (0=LSL, 1=LSR, 2=ASR, 3=ROR)
4 | R - シフト量を示すのにレジスタを使うか即値を使うか (0=即値, 1=レジスタ)
3-0 | Rm - 2個目のオペランドとなるレジスタの番号

11-7bitは

- Rが0のとき: シフト量を表す即値(1-31, 0は特別な処理を行います)
- Rが1のとき: 11-8はシフト量が入っているレジスタ、 7は必ず0になります

#### bit25(I)が1 つまり 2個目のオペランドが即値の場合

 bit  |  内容
---- | ----
11-8 | Is - nnをRORシフトする量(0-30)
7-0 | nn - 2個目のオペランドとなる8bitの即値

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
0x0f | MVN{cond}{S} Rd,Op2 | Rd = NOT Op2

## 2個目のオペランド(Op2)

25bitや11-0bitを見るとわかるように2個目のオペランドは基本的に、シフトされたレジスタ値かシフトされた即値となる

- シフトされないレジスタ: Op2は`Rm`と表記されます、アセンブラでは`Rm,LSL#0`と表されます。
- シフトされるレジスタ: `Rm,SSS#Is` または `Rm,SSS Rs`と表されます (SSS=LSL/LSR/ASR/ROR)
- 即値: Specify as 32bit value, for example: "#000NN000h", assembler should automatically convert into "#0NNh,ROR#0ssh" as far as possible (ie. as far as a section of not more than 8bits of the immediate is non-zero).

## シフト量が0 (シフト量が即値の0で表されるとき)

```
 LSL#0: シフトは行われません。 つまり、Op2=Rmとなりキャリーの値は不変です。
 LSR#0: LSR#32として解釈されます。つまり、Op2が0になったとき、キャリーはRmの31bitが入ります。
 ASR#0: ASR#32として解釈されます。つまり、Op2とキャリーはRmの31bitが入ります。
 ROR#0: ROR#1 と同様に RRX#1 (RCR) と解釈されますが、Op2の31bitは古いキャリーの値になります。
```

## R15(PC)の使用

When using R15 as Destination (Rd), note below CPSR description and Execution time description.

When using R15 as operand (Rm or Rn), the returned value depends on the instruction: PC+12 if I=0,R=1 (shift by register), otherwise PC+8 (shift by immediate).

## Returned CPSR Flags

If S=1, Rd<>R15, logical operations (AND,EOR,TST,TEQ,ORR,MOV,BIC,MVN):
  V=not affected
  C=carryflag of shift operation (not affected if LSL#0 or Rs=00h)
  Z=zeroflag of result
  N=signflag of result (result bit 31)
If S=1, Rd<>R15, arithmetic operations (SUB,RSB,ADD,ADC,SBC,RSC,CMP,CMN):
  V=overflowflag of result
  C=carryflag of result
  Z=zeroflag of result
  N=signflag of result (result bit 31)
IF S=1, with unused Rd bits=1111b, {P} opcodes (CMPP/CMNP/TSTP/TEQP):
  R15=result  ;modify PSR bits in R15, ARMv2 and below only.
  In user mode only N,Z,C,V bits of R15 can be changed.
  In other modes additionally I,F,M1,M0 can be changed.
  The PC bits in R15 are left unchanged in all modes.
If S=1, Rd=R15; should not be used in user mode:
  CPSR = SPSR_${current mode}
  PC = result
  For example: MOVS PC,R14  ;return from SWI (PC=R14_svc, CPSR=SPSR_svc).
If S=0: Flags are not affected (not allowed for CMP,CMN,TEQ,TST).

The instruction "MOV R0,R0" is used as "NOP" opcode in 32bit ARM state.
Execution Time: (1+p)S+rI+pN. Whereas r=1 if I=0 and R=1 (ie. shift by register); otherwise r=0. And p=1 if Rd=R15; otherwise p=0.
